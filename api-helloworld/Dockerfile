FROM golang:1.20.4-bullseye as builder

WORKDIR /srv

ADD helloworld/main.go main.go
ADD helloworld/go.mod go.mod
ADD helloworld/go.sum go.sum

RUN go mod tidy
RUN go build

FROM ubuntu:22.04

WORKDIR /srv

COPY --from=builder /srv/helloworld helloworld

RUN chmod +x helloworld

CMD ["/srv/helloworld"]
