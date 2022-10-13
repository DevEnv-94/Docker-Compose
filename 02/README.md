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

networks:
  database_network:
  web_network:

services:
  database:
    image: postgres:10
    restart: on-failure
    networks:
      - database_network
    environment:
      POSTGRES_USER: user
      POSTGRES_DB: postgres
      POSTGRES_PASSWORD: DatabasePassword

  gocalc:
    image: gocalc:latest
    networks:
      - database_network
      - web_network
    restart: unless-stopped
    command: sh -c "/wait && /app"
    depends_on:
      - database
    environment:
      POSTGRES_URI: "postgres://user:DatabasePassword@database/postgres?sslmode=disable"
      WAIT_HOSTS: "database:5432"
      WAIT_BEFORE: "5"

  nginx:
    image: nginx:stable
    networks:
      - web_network
    restart: on-failure
    ports:
      - "127.0.0.1:80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - database
      - gocalc
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
            proxy_pass http://gocalc:7000;
        }
    }

}
```
```bash
sudo docker-compose -p rbm29 up -d
```
