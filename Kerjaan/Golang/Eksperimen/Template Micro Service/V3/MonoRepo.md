
```Dockerfile
# apps/user-service/Dockerfile
FROM golang:1.24-alpine AS builder

WORKDIR /app

# Install necessary build tools
RUN apk add --no-cache git

# Copy the whole project to the container
#COPY . .

COPY ./apps/user-service /app/apps/user-service
COPY ./apps/enkripsi-service /app/apps/enkripsi-service

#WORKDIR /app/apps/enkripsi-service
#RUN go mod tidy

COPY ./libs ./libs/

# Tidy modules in both libs and service
WORKDIR libs/shared
RUN go mod tidy
#
WORKDIR /app/apps/user-service
RUN go mod tidy
#
## Build the binary
RUN CGO_ENABLED=0 GOOS=linux go build -o user-service /app/apps/user-service/cmd/main.go

#RUN rm -rf /app/enkripsi-service
#ENTRYPOINT ["./user-service"]
#
## Final stage with minimal image
FROM bash:latest AS runner
#
WORKDIR /app
#
## Copy only the built binary
COPY --from=builder /app/apps/user-service/user-service /app/apps/user-service/
COPY --from=builder /app/libs/ ./libs/
#
WORKDIR /app/apps/user-service

## Make sure it's executable
RUN chmod +x ./user-service
#
ENTRYPOINT ["./user-service"]



```

```docker-compose 
# infra/docker-compose.yaml
  
user-service:
    container_name: user-service
    build:
      context: ../
      target: runner
      dockerfile: apps/user-service/Dockerfile

    volumes:
      - ../apps/user-service:/apps/user-service
      - ../libs:/apps/libs
    ports:
      - "3100:3100"
#    network_mode: host
    restart: always
	
```

```bash
sudo docker-compose -f infra/docker-compose.yaml up --build -d user-service
```
