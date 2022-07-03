```Dockerfile
FROM golang:1.16.5-alpine as build

ENV GO111MODULE=auto GOPATH=/go

WORKDIR /go/application/

COPY ./main.go .

RUN apk add --update git && \
    rm -rf /var/cache/apk/*/apk/* && \
    go mod init module && \
    go mod tidy && \
    go get ./... && \
    go build -o /app .


FROM alpine:3.10.3

COPY --from=build /app /app

ADD https://github.com/ufoscout/docker-compose-wait/releases/download/2.9.0/wait /wait
RUN chmod +x /wait

EXPOSE 7000

CMD  ["/app"]
```
```bash
sudo docker build . -t gocalc:latest
```
```yaml
version: "3.7"                
services:                  
  db:                    
    image: postgres:10
    restart: on-failure
    environment:
      POSTGRES_USER: user
      POSTGRES_DB: postgres
      POSTGRES_PASSWORD: DatabasePassword

  app:
    image: gocalc:latest
    restart: on-failure
    command: sh -c "/wait && /app"
    depends_on:
      - db
    environment:
      POSTGRES_URI: "postgres://user:DatabasePassword@db/postgres?sslmode=disable"
      WAIT_HOSTS: "db:5432"
      WAIT_BEFORE: "5"
  frontend:
    image: nginx:stable   
    restart: on-failure               
    ports:
      - "127.0.0.1:80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - db
      - app
```
```bash
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    tcp_nopush     on;

    keepalive_timeout  65;

    gzip  on;


    server {
        listen 80;

        location  / {
            proxy_pass http://app:7000;
        }
    }

}
```
```bash
sudo docker-compose -p rbm28 up -d
```