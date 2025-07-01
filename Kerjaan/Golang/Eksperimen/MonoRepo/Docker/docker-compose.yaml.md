pwd: /home/zams/go/src/Digit-platform

sudo docker-compose -f infra/docker-compose.yaml up --build -d user-service

/home/zams/go/src/Digit-platform/infra/docker-compose.yaml
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
    - "3000:3000"  
  restart: always  
  network_mode: host

context di sini penting, karena akan mempengaruhi [[Dockerfile]]. Jadi context ../ artinya Dockerfile akan dieksekusi dengan working directory nya adalah /home/zams/go/src/Digit-platform alias project root. Target adalah stage yang menjadi final image yang nanti dipakai oleh container. 