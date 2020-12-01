FROM golang:latest AS builder

WORKDIR /work
COPY main.go .
RUN go build -o web .

FROM golang:latest

WORKDIR /exec
COPY --from=builder /work/web .
EXPOSE 8080
CMD ["./web"]
