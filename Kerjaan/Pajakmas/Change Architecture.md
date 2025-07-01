Untuk switch arsitektur, mulai dari yang lebih jarang dipake aja dulu atau api nya dikit atau dia pola nya berulang, kayak generate, report, user, upload, transaksi, master

User pecah jadi user dan customer, report ada yang auction, pisah aja, transaksi pecah auction bank finance bikin service sendiri atau ke service yang sesuai kategori nya

Generate-> User -> Finance -> Report -> Upload -> Transaksi -> Master

TODO Services:

Generate:
	Split the repositories, models: Finance(Settlement,Booking,BufferCabang,KasValuer,KasBank,Girocek,BankTransfer)
	Auction(Presales,Jadual,Invoice)
	Gadai(OS Collateral,OS Purity,Redemption Daily, Monthly, Transaction Monthly)


	Bikin Function Upload dan Get Link nya, akomodir S3 dan OSS
	
	Compare wkhtmltopdf sama excelize mending di docker atau service linux

User:

	Split user hanya untuk login, dashboard, dll. Customer bikin service terpisah

Master:
Konstan:Wilayah,Agama,hubungan keluarga,jenis perkawinan
