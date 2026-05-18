FROM golang:1.22-alpine AS builder

WORKDIR /src

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app ./cmd/api

FROM alpine:3.20

RUN apk add --no-cache ca-certificates wget && \
    adduser -D -u 10001 app

WORKDIR /app
COPY --from=builder --chown=app:app /app /app/app

USER app
EXPOSE 8081

ENTRYPOINT ["/app/app"]
