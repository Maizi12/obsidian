
|     | Change        | Service           | Keterangan Tambahan                                                                                                                                                    | Function          |
| --- | ------------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- |
|     | Procedure     | DB                | CASE WHEN trans_lelang_jadualn.tanggalAuctionLock IS NOT NULL THEN DATE_FORMAT(trans_lelang_jadualn.tanggalAuctionLock, '%d/%m/%Y') ELSE<br>	'' ) END AS TarikhDijual, | get_jadual_n_view |
|     | Procedure     | DB                | Parameter _kode varchar(255)                                                                                                                                           | get_jadual_n_view |
|     | Excel Akta 11 | Report & Generate |                                                                                                                                                                        |                   |
