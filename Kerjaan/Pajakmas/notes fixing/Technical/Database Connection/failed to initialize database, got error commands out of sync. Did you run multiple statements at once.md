
``` go
[mysql] 2025/05/17 21:24:20 connection.go:49: busy buffer
[ERROR] failed to initialize database, got error commands out of sync. Did you run multiple statements at once?
2025/05/17 21:24:20 err connect db main: commands out of sync. Did you run multiple statements at once?
error ini muncul di go routine auto rekonek

func startDBRefresher(safeDB *db.SafeDB) {
	newDB := db.Connection()
	if newDB != nil {
		safeDB.Set(newDB)
		//log.Println("Database connection refreshed")
	}
	go func() {
		for {
			newDB := db.Connection()
			if newDB != nil {
				safeDB.Set(newDB)
				//log.Println("Database connection refreshed")
			}
			time.Sleep(5 * time.Second)
		}
	}()
}


var Konek *gorm.DB
var SafeConn = &SafeDB{}

type SafeDB struct {
	db         *gorm.DB
	mux        sync.RWMutex
	timeout    time.Duration
	refreshing bool
	active     int32 // atomic counter for active queries
}

func (s *SafeDB) Get(ctx context.Context, timeout time.Duration, maxRetries int) (*gorm.DB, error) {
	deadline := time.Now().Add(timeout)
	// Waiting for in-flight DB Update to finish
	for {
		if !s.refreshing {
			break
		}
		time.Sleep(50 * time.Millisecond)
	}
	for i := 0; i < maxRetries; i++ {
		s.mux.RLock()
		isRefreshing := s.refreshing
		db := s.db
		s.mux.RUnlock()

		if !isRefreshing && db != nil {
			if atomic.AddInt32(&s.active, 1) > 0 {
				if db.Error != nil {
					fmt.Println("db.Error", db.Error)
				}
				return db.WithContext(ctx), nil
			}
		}

		if time.Now().After(deadline) {
			break
		}
		time.Sleep(50 * time.Millisecond)
	}
	return nil, errors.New("database temporarily unavailable or refreshing")
}

func (s *SafeDB) Done() {
	atomic.AddInt32(&s.active, -1)
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
func (s *SafeDB) WithTimeout(ctx context.Context, timeout time.Duration) *gorm.DB {
	newCtx, cancel := context.WithTimeout(ctx, timeout)
	// Important: cancel context after query completes
	db, err := s.Get(newCtx, timeout, 20)
	if err != nil {
		cancel()
		return Konek
	}
	go func() {
		<-newCtx.Done()
		s.Done()
		cancel()
	}()
	return db
}
func Connection() *gorm.DB {
	newLogger := &customlogger.CustomLogger{}

	//newLogger :=
	//	logger.New(
	//
	//		log.New(os.Stdout, "\r\n", log.LstdFlags), // io writer
	//		logger.Config{
	//			LogLevel:                  logger.Info,
	//			IgnoreRecordNotFoundError: true,  // Ignore ErrRecordNotFound error for logger
	//			Colorful:                  false, // Enable colorful output// Log level: logger.Silent, logger.Error, logger.Warn, logger.Info
	//		},
	//	)

	dbUser := conf.Result.DB.User
	dbPassword := conf.Result.DB.Password
	dbURL := conf.Result.DB.Host
	dbName := conf.Result.DB.Name
	SetIdle := conf.Result.DB.MaxIdleConns
	SetOpen := conf.Result.DB.MaxOpenConns
	SetLifetime := conf.Result.DB.MaxLifetime

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
		return db
	}

	sqlDB.SetMaxIdleConns(SetIdle)
	sqlDB.SetMaxOpenConns(SetOpen)
	sqlDB.SetConnMaxLifetime(time.Duration(SetLifetime))
	//fmt.Println("CONNECTED")
	return db

}
```

###### ChatGPT said:

Error `commands out of sync. Did you run multiple statements at once?` in MySQL typically happens when:

> You reuse a `*sql.DB` or `*sql.Conn` or `*sql.Tx` (used internally by GORM) without properly reading all rows from a previous query, or when you open a new connection while the old one is still "busy".

### ⚠️ Root Cause in Your Case:

Your `SafeDB.Set()` method switches the database connection _while possibly reusing the old `*gorm.DB` object_, which may still have pending results or prepared statements, leading to:

arduino

CopyEdit

`busy buffer commands out of sync`

---

## ✅ How to Fix

### ✅ 1. Clone `*gorm.DB` instead of reusing it directly

When you call `safeDB.Set(newDB)`, make sure that:

- No query is running on the old one.
    
- You’re not reusing any ongoing `*gorm.DB` session (they are not safe to swap underneath).
    

Your locking mechanism already tries to wait for `active == 0`, but **GORM may still hold internal states**, especially for prepared statements.

Instead of forcibly switching the whole `gorm.DB` instance, it is safer to:

### ✅ 2. Use a connection **pool** under the hood and let `gorm.DB` point to it

If you're only updating the `sql.DB` pool, you don’t have to recreate the `gorm.DB` entirely. But in your case you're doing `gorm.Open(...)` repeatedly, which can cause:

- New `*gorm.DB` instances holding fresh `sql.DB` pools.
    
- Invalid old statements in previous instances.
    
- Memory leaks or dangling goroutines if `Set` is called too often.
    

---

## ✅ Better Solution Design

Here’s how you can refactor your `SafeDB` logic to avoid this issue entirely:

### 1. Reuse a **single `*gorm.DB` instance** as much as possible.

Only update the **underlying `*sql.DB`** pool if needed (like with `db.DB().Set...()`), but avoid closing or re-opening GORM unless there's a **real** need.

If you must recreate it:

### 2. **Ensure no query, transaction, or scan is in progress.**

You did this:

go

CopyEdit

`for {   if atomic.LoadInt32(&s.active) == 0 {     break   }   time.Sleep(50 * time.Millisecond) }`

But GORM internally might still hold open statements, so it's safer to:

- Avoid using `gorm.DB` inside long-lived goroutines (e.g., don’t persist and reuse it deeply).
    
- Add timeout on query execution (`ctx.WithTimeout` — good, you're already doing it).
    

---

## 🧪 Debug Tips

### 1. Check for forgotten `.Rows()` or `.Scan()` without `Close()`

If any query like `.Raw().Rows()` or direct `sql.Rows` is not closed, it’ll block the connection pool and cause this error.

### 2. Log and track connection stats:

go

CopyEdit

`sqlDB, _ := db.DB() fmt.Printf("OpenConns: %d, InUse: %d, Idle: %d\n",  	sqlDB.Stats().OpenConnections, 	sqlDB.Stats().InUse, 	sqlDB.Stats().Idle)`

---

## 🛠 Suggested Fix in Code

### Option A (safe): Don't reassign `*gorm.DB`, just reset underlying `*sql.DB`:

go

CopyEdit

`sqlDB, _ := db.DB() sqlDB.SetConnMaxLifetime(...) sqlDB.SetMaxOpenConns(...)`

Then use `SafeDB.SetConfig(...)` instead of replacing the entire DB.

### Option B (risky but controlled): Update `SafeDB.Set()` with safer handling

go

CopyEdit

`func (s *SafeDB) Set(newDB *gorm.DB) { 	// Wait for in-flight queries to finish 	for { 		if atomic.LoadInt32(&s.active) == 0 { 			break 		} 		time.Sleep(50 * time.Millisecond) 	}  	// Close old db safely 	s.mux.Lock() 	old := s.db 	s.db = newDB 	s.mux.Unlock()  	if old != nil { 		sqlDB, err := old.DB() 		if err == nil { 			_ = sqlDB.Close() 		} 	} }`

Also, make sure no goroutines are reusing the old `db.WithContext(...)`.

---

## Final Suggestion

> Only recreate the DB if absolutely necessary (e.g., credentials rotated or DB server changed). Otherwise, tune connection pool dynamically without replacing `gorm.DB`.

If you still want hot-swapping of `gorm.DB`, consider creating **an internal `sql.DB` pool manager**, then wrap GORM around that when needed.

Let me know if you want a template or refactor plan for that approach.

##### You said:

what's the code in SetConfig? You just refer to it while I never gave it to you

###### ChatGPT said:

You're right — you never gave a `SetConfig` method, and I mentioned it as a **suggested** alternative in case you wanted a safer way to **tune the DB without recreating** the `*gorm.DB`. Since your `SafeDB.Set()` is replacing the whole GORM instance (which can cause `commands out of sync`), a better strategy is to **just reconfigure the pool settings** instead of reconnecting.

Here's what a `SetConfig()` method could look like for your `SafeDB`:

---

### ✅ `SetConfig()` — adjust connection settings without replacing `*gorm.DB`

go

CopyEdit

`func (s *SafeDB) SetConfig(maxIdle, maxOpen int, maxLifetime time.Duration) error { 	s.mux.RLock() 	defer s.mux.RUnlock()  	if s.db == nil { 		return errors.New("no DB initialized") 	}  	sqlDB, err := s.db.DB() 	if err != nil { 		return fmt.Errorf("failed to get sql.DB: %w", err) 	}  	sqlDB.SetMaxIdleConns(maxIdle) 	sqlDB.SetMaxOpenConns(maxOpen) 	sqlDB.SetConnMaxLifetime(maxLifetime) 	return nil }`

---

### ✅ Example usage:

Instead of this:

go

CopyEdit

`safeDB.Set(db.Connection()) // risk of breaking queries`

Use:

go

CopyEdit

`safeDB.SetConfig( 	conf.Result.DB.MaxIdleConns, 	conf.Result.DB.MaxOpenConns, 	time.Duration(conf.Result.DB.MaxLifetime), )`

---

### ✅ When to use `Set()` vs `SetConfig()`

|Purpose|Use|
|---|---|
|DB credentials/server changed|`Set()` with new `gorm.DB`|
|Just want to change pooling|`SetConfig()` safely|
|Query timeout|Use `WithTimeout()`|

---

Let me know if you want `SafeDB` to support _hot DB reinitialization_ without disrupting in-flight queries — I can provide a thread-safe wrapper for that too.