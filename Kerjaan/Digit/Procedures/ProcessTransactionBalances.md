
```mysql
CREATE DEFINER=`root`@`localhost` PROCEDURE `ProcessTransactionBalances`(
	IN `_idUser` INT,
	IN `_TanggalDari` DATE,
	IN `_TanggalSampai` DATE
)
LANGUAGE SQL
NOT DETERMINISTIC
CONTAINS SQL
SQL SECURITY DEFINER
COMMENT ''
BEGIN
    -- Declare variables
    DECLARE done INT DEFAULT FALSE;
    DECLARE trans_id INT;
    DECLARE trans_date DATE;
    DECLARE trans_desc TEXT;
    DECLARE trans_type INT;
    DECLARE user_id INT;
    
    -- Cursor for transaction headers
    DECLARE trans_cursor CURSOR FOR 
        SELECT idTransaksi, tglTransaksi, keteranganTransaksi, idJenisTransaksi, idUser
        FROM transaksi
        WHERE idUser=_idUser AND tglTransaksi>=_TanggalDari AND tglTransaksi<=_TanggalSampai
        ORDER BY tglTransaksi, idTransaksi;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    -- Create temporary balance table
    DROP TEMPORARY TABLE IF EXISTS temp_balances;
    CREATE TEMPORARY TABLE temp_balances (
        idCoa INT PRIMARY KEY,
        current_balance DECIMAL(20,2) DEFAULT 0,
        last_trans_id INT
    );
    
    OPEN trans_cursor;
    
    trans_loop: LOOP
        FETCH trans_cursor INTO trans_id, trans_date, trans_desc, trans_type, user_id;
        IF done THEN
            LEAVE trans_loop;
        END IF;
        
        -- Check if details exist, if not insert template entries
        IF NOT EXISTS (SELECT 1 FROM transaksi_detail WHERE idTransaksi = trans_id) THEN
            -- Insert debit entry (adjust logic as needed)
            INSERT INTO transaksi_detail (idTransaksi, idCoa, debitKredit, nominal)
            SELECT trans_id, idCoaDebit, debitKredit, nominal
            FROM transaksi
            WHERE idTransaksi = trans_id
            AND idCoaDebit IS NOT NULL;
            
            -- Insert credit entry (adjust logic as needed)
            INSERT INTO transaksi_detail (idTransaksi, idCoa, debitKredit, nominal)
            SELECT trans_id, idCoaKredit, 
				CASE WHEN coa.posDK='K' THEN 'D' ELSE debitKredit END AS debitKredit -- ini penting karena akan mempengaruhi transaksi antar bank
				, nominal
            FROM transaksi
            INNER JOIN coa ON coa.idCoa=idCoaKredit
            WHERE idTransaksi = trans_id
            AND idCoaKredit IS NOT NULL;
        END IF;
        
        -- Process each detail record for this transaction
        BEGIN
            DECLARE detail_done INT DEFAULT FALSE;
            DECLARE detail_id INT;
            DECLARE coa_id INT;
            DECLARE dk ENUM('D','K');
            DECLARE amount DECIMAL(20,2);
            
            DECLARE detail_cursor CURSOR FOR 
                SELECT idTransaksiDetail, idCoa, debitKredit, nominal
                FROM transaksi_detail
                WHERE idTransaksi = trans_id;
                
            DECLARE CONTINUE HANDLER FOR NOT FOUND SET detail_done = TRUE;
            
            OPEN detail_cursor;
            
            detail_loop: LOOP
                FETCH detail_cursor INTO detail_id, coa_id, dk, amount;
                IF detail_done THEN
                    LEAVE detail_loop;
                END IF;
                
                -- Get COA type from chart_of_accounts
                SET @coa_type = (SELECT 
                    posDK
                FROM coa WHERE idCoa = coa_id);
                
                -- Calculate balance change
                SET @balance_change = CASE 
                    -- WHEN  dk = 'D' THEN -amount
                    -- WHEN dk = 'K' THEN amount
                   
                    WHEN @coa_type = 'D' AND dk = 'D' THEN amount
                    WHEN @coa_type = 'D' AND dk = 'K' THEN -amount
                    WHEN @coa_type = 'K' AND dk = 'D' THEN -amount
                    WHEN @coa_type = 'K' AND dk = 'K' THEN amount
                END;
                
                -- Update or insert balance record
                INSERT INTO temp_balances (idCoa, current_balance, last_trans_id)
                VALUES (coa_id, @balance_change, trans_id)
                ON DUPLICATE KEY UPDATE 
                    current_balance = current_balance + @balance_change,
                    last_trans_id = trans_id;
                
                -- Update detail record with current balance
                UPDATE transaksi_detail d
                JOIN temp_balances b ON d.idCoa = b.idCoa
                SET d.saldoCoa = b.current_balance
                WHERE d.idTransaksiDetail = detail_id;
            END LOOP;
            
            CLOSE detail_cursor;
        END;
        
        -- Update header with summary balances
        UPDATE transaksi h
        JOIN (
            SELECT 
                idTransaksi,
                SUM(CASE WHEN debitKredit = 'D' THEN nominal ELSE 0 END) AS total_debit,
                SUM(CASE WHEN debitKredit = 'K' THEN nominal ELSE 0 END) AS total_credit
            FROM transaksi_detail
            WHERE idTransaksi = trans_id
        ) AS sums ON h.idTransaksi = sums.idTransaksi
        SET 
            h.saldoCoaDebit = sums.total_debit,
            h.saldoCoaKredit = sums.total_credit,
            h.sisaSaldo = sums.total_debit - sums.total_credit;
    END LOOP;
    
    CLOSE trans_cursor;
    
    -- Return final balances
    SELECT 
        c.kodeCoa,
        c.namaCoa,
        b.current_balance AS final_balance,
        c.posDK AS account_type,
        b.last_trans_id AS last_affected_transaction
    FROM temp_balances b
    JOIN coa c ON b.idCoa = c.idCoa
    ORDER BY c.kodeCoa;
    
    -- Clean up
    DROP TEMPORARY TABLE IF EXISTS temp_balances;
END
```