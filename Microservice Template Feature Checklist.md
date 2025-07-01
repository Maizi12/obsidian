# ✅ Microservice Template Feature Checklist

## 📦 Core Architecture
- [x] RedEnv to Variable  
  _Avoid file re-reading, safer for binary deployment without config file._
- [x] Panic Recovery Middleware  
  _Prevent panics from crashing the whole service._
- [x] Graceful Shutdown  
  _Handles `SIGINT`/`SIGTERM` and finishes in-flight requests cleanly._
- [ ] Request Timeout Middleware  
  _Avoid long-hanging requests and protect from slow downstream services._

---

## ⚙️ Configuration
- [x] Config from `.env` / JSON  
  _Environment-based config management._
- [ ] ✅ Config Schema Validation  
  _Validate fields, types, and required configs on boot._
- [ ] ✅ Feature Flags
  _Toggle features safely using flags from config or environment._

---

## 🔄 Database Handling
- [x] Custom GORM Logger  
  _Improves SQL readability and debugging._
- [x] Auto Reconnect DB  
  _No need to restart Docker if DB is restarted._
- [x] Query Wait for Reconnect  
  _Prevents conflict with reconnecting DB during ongoing queries._
- [x] Connection Pool Tuning  
  _Optimizes DB resource usage (e.g., `MaxOpenConns`, `MaxIdleConns`, `ConnMaxLifetime`)._

---

## 📑 Observability & Logging
- [x] Context-Aware Logging  
  _Structured logs with request ID or context for tracing._
- [ ] Request Timeout Logging
  _Log requests that exceed the timeout duration._
- [ ] Optional: Tracing / Prometheus
  _If you're planning to add metrics or tracing._

---

## 🛡️ Resilience & Middleware
- [x] PanicHandler as Fallback Middleware  
  _Catches any unhandled panic._
- [ ] CORS
  _Future-proof for public APIs._

---

## 🧪 Developer Experience
- [x] Docker-ready Deployment  
  _With auto-restart and volume-mapped logs/configs._
- [ ] Taskfile or Makefile  
  _Simplify testing, build, and format tasks._

