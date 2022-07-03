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
