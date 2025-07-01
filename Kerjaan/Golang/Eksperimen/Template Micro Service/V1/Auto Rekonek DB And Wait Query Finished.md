
| What                               | Why                                                                     |
| ---------------------------------- | ----------------------------------------------------------------------- |
| Auto Rekonek DB                    | Biar kalo server DB nya ke restart, gaperlu restart docker atau service |
| Query DB nunggu selesai rekonek DB | Tabrakan dengan rekonek DB kalo nggak                                   |
```go
// main.go
func startDBRefresher(safeDB *db.SafeDB) {  
    go func() {  
       for {  
          newDB := db.Connection()  
          if newDB != nil {  
             safeDB.Set(newDB)  
             //fmt.Println("DB Updated")  
          }  
          time.Sleep(2 * time.Second)  
       }  
    }()  
}

func main() {
conf.ReadEnv()
go startDBRefresher(db.SafeConn)
}

// dbconnection.go
type SafeDB struct {  
    db         *gorm.DB  
    mux        sync.RWMutex  
    timeout    time.Duration  
    refreshing bool  
    active     int32 // atomic counter for active queries
}

  
func (s *SafeDB) Set(newDB *gorm.DB) {  
    s.mux.Lock()  
    s.refreshing = true  
    s.mux.Unlock()  
  
    // Wait for in-flight queries to finish  
    for {  
       if atomic.LoadInt32(&s.active) == 0 {  
          break  
       }  
       time.Sleep(50 * time.Millisecond)  
    }  
  
    s.mux.Lock()  
    old := s.db  
    s.db = newDB  
    s.refreshing = false  
    s.mux.Unlock()  
    if old != nil {  
       if sqlDB, err := old.DB(); err == nil {  
          sqlDB.Close()  
       }  
    }  
}

func Connection() *gorm.DB {  
    newLogger := &CustomLogger{}  
  
    dbUser := conf.Konfig.DB.User  
    dbPassword := conf.Konfig.DB.Password  
    dbURL := conf.Konfig.DB.Host  
    dbName := conf.Konfig.DB.Name  
    SetIdle := conf.Konfig.DB.MaxIdleConns  
    SetOpen := conf.Konfig.DB.MaxOpenConns  
    SetLifetime := conf.Konfig.DB.MaxLifetime  
  
    addr := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&parseTime=True&loc=Local", dbUser, dbPassword, dbURL, "3306", dbName)  
    db, err := gorm.Open(grmsql.Open(addr), &gorm.Config{  
       Logger: newLogger,  
    })  
    Konek = db  
    if err != nil {  
       log.Println("err connect db main:", err)  
    }  
  
    sqlDB, errSet := db.DB()  
    if errSet != nil {  
       log.Fatal("Failed to get database object from GORM:", err)  
    }  
  
    sqlDB.SetMaxIdleConns(SetIdle)  
    sqlDB.SetMaxOpenConns(SetOpen)  
    sqlDB.SetConnMaxLifetime(time.Duration(SetLifetime))  
  
    return db  
}

```

| What                                      | Test Case                                                  | Code                                                     | Result                                                                                         |
| ----------------------------------------- | ---------------------------------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Auto Rekonek DB                           | Stop service db lokal ketika aplikasi golang berjalan      | function startDBRefresher()                              | Error konek ke db, dan ketika db aktif kembali, aplikasi berjalan normal tanpa perlu direstart |
| Rekonek DB dihold sampai Query DB selesai | Buat request concurrent 100 kali dengan total request 1000 | ab -H -n 1000 -c 100 "http://localhost:4032/v1/user/coa" | Aman, Update DB nunggu query selesai                                                           |
