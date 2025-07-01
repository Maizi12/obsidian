## 🐳 Catatan Docker (Multi-Stage Build + Docker Compose)

### 🧱 Struktur Project

Path: `/home/zams/go/src/Digit-platform`

```
Digit-platform/
├── apps/
│   ├── user-service/
│   │   └── Dockerfile
│   └── enkripsi-service/
├── libs/
│   └── shared/
└── infra/
    └── docker-compose.yaml
```


---

### 🐋 Dockerfile – Multi-stage Build

Path: `/home/zams/go/src/Digit-platform/apps/user-service/Dockerfile`

```Dockerfile
# Stage 1: Builder
FROM golang:1.24-alpine AS builder

WORKDIR /app

# Install tools
RUN apk add --no-cache git

# Copy source code (per service)
COPY ./apps/user-service /app/apps/user-service
COPY ./apps/enkripsi-service /app/apps/enkripsi-service
COPY ./libs ./libs/

# Module tidy (libs dulu baru service)
WORKDIR libs/shared
RUN go mod tidy

WORKDIR /app/apps/user-service
RUN go mod tidy

# Build binary
RUN CGO_ENABLED=0 GOOS=linux go build -o user-service /app/apps/user-service/cmd/main.go

---

# Stage 2: Runner (smaller image)
FROM bash:latest AS runner

WORKDIR /app

# Copy built binary and needed files
COPY --from=builder /app/apps/user-service/user-service /app/apps/user-service/
COPY --from=builder /app/libs/ ./libs/

WORKDIR /app/apps/user-service

# Set permission
RUN chmod +x ./user-service

# Use ENTRYPOINT to run the service
ENTRYPOINT ["./user-service"]

```

#### ✍️ Penjelasan

- **Stage 1 (`builder`)**: Compile binary dari source code pakai `golang:alpine`, lebih ringan dan cepat.
- **Stage 2 (`runner`)**: Hanya bawa file yang perlu dijalankan. Image lebih kecil & clean.
- `ENTRYPOINT` digunakan supaya saat container dijalankan, langsung eksekusi binary-nya.
- Bisa dibandingkan dengan `CMD`, tapi `ENTRYPOINT` lebih eksplisit untuk behavior utama.

---

### 🧩 docker-compose.yaml

Path: `/home/zams/go/src/Digit-platform/infra/docker-compose.yaml`

```yaml
version: "3.8"

services:
  user-service:
    container_name: user-service
    build:
      context: ../
      dockerfile: apps/user-service/Dockerfile
      target: runner  # penting: pilih stage akhir
    volumes:
      - ../apps/user-service:/apps/user-service
      - ../libs:/apps/libs
    ports:
      - "3000:3000"
    restart: always
    network_mode: host

```

#### ✍️ Penjelasan

- **`context: ../`** → Artinya Dockerfile dieksekusi dari project root (`/home/zams/go/src/Digit-platform`).
    
- **`target: runner`** → Pilih stage `runner` sebagai final image (hasil multi-stage build).
    
- **`volumes:`** → Bind mount agar update di local langsung terpantau dalam container. Cocok buat dev.
    
- **`network_mode: host`** → Container share network dengan host. Cocok kalau service jalan lokal dan butuh akses cepat (hati-hati untuk production).
    

---

### 🔧 Build & Run

```bash
# Build and run in detached mode
sudo docker-compose -f infra/docker-compose.yaml up --build -d user-service
```
---

### 🐛 Debugging Tips

- **Container restart terus?**  
    Cek pakai:
```bash
docker logs user-service
```
---
**Container nggak jalan?**  
Pastikan `ENTRYPOINT` bisa dieksekusi (`chmod +x`) dan direktori kerja benar (`WORKDIR` cocok dengan lokasi binary).
### 📌 Ringkasan Perbedaan CMD vs ENTRYPOINT

|Perbedaan|CMD|ENTRYPOINT|
|---|---|---|
|Default Run|Bisa di-overwrite via `docker run`|Tidak mudah di-override|
|Tujuan|Optional, fallback command|Menentukan _main process_ container|
|Digunakan|Untuk testing / fleksibel|Untuk container yang dedicated binary|