Data bisa disearch, ada opsi order by terbaru terlama. Pake scopes biar reusable
Bikin pagination.
Ekstraksi JWT.
Controller jadiin buat ekstrak jtw, Json Validation aja, pengolahan data di repositories.
TableName taro di constant aja, biar gampang kalo mau diganti atau dicari.


Pake env untuk konfigurasi akses db, port router, app name.
router nya bikin grouping. Bikin grup terpisah antara login dan get user, biar settingan middleware nya bisa dibedain.
Bikin db nya bisa nampilin query yang dipake di cmd nya dengan pake:

func dbconnect(){

newLogger :=
		logger.New(

			log.New(os.Stdout, "\r\n", log.LstdFlags), // io writer
			logger.Config{
				LogLevel:                  logger.Info,
				IgnoreRecordNotFoundError: true,  // Ignore ErrRecordNotFound error for logger
				Colorful:                  false, // Enable colorful output// Log level: logger.Silent, logger.Error, logger.Warn, logger.Info
			},
		)
db, err := gorm.Open(mysql.Open(addr), &gorm.Config{
		Logger: newLogger,
	})
}

Bikin function untuk bisa ekstrak isi jwt dan pastikan hasil ekstrak nya sama dengan data yang dipake ketika create JWT nya. 

Bikin CORS
Struct untuk json input data dipisah file nya dari models.
Coba pasang middleware basic Auth sama Auth Header

Untuk variable-variable yang berulang kayak nama table, statusAktif dll jadiin constant aja