
| What                               | Why                                   |
| ---------------------------------- | ------------------------------------- |
| Query DB nunggu selesai rekonek DB | Tabrakan dengan rekonek DB kalo nggak |

```go
//dbconnection.go
func (s *SafeDB) Get(ctx context.Context, timeout time.Duration, maxRetries int) (*gorm.DB, error) {  
    deadline := time.Now().Add(timeout)  
    // Wait for in-flight Update to finish  
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

func (s *SafeDB) WithTimeout(ctx context.Context, timeout time.Duration) *gorm.DB {  
    newCtx, cancel := context.WithTimeout(ctx, timeout)  
  
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

// repositories.go
o := db.SafeConn.WithTimeout(c.Ctx.Request.Context(), 30*time.Second)

```

| What                               | Test Case                                                  | Code                                                     | Result                                  |
| ---------------------------------- | ---------------------------------------------------------- | -------------------------------------------------------- | --------------------------------------- |
| Query DB nunggu selesai rekonek DB | Buat request concurrent 100 kali dengan total request 1000 | ab -H -n 1000 -c 100 "http://localhost:4032/v1/user/coa" | Aman, query DB nunggu Update DB selesai |
