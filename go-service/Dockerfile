# Build stage
FROM golang:1.24-alpine AS builder

# Install build dependencies for SQLite
RUN apk add --no-cache gcc musl-dev

WORKDIR /app

# Copy go mod file
COPY go.mod ./

# Generate go.sum and download dependencies
RUN go mod download

# Copy source code
COPY . .

# Run go mod tidy to ensure everything is in sync
RUN go mod tidy

# Build with CGO enabled
RUN CGO_ENABLED=1 go build -o go-service .

# Run stage
FROM alpine:latest

# Install runtime dependencies
RUN apk add --no-cache ca-certificates

WORKDIR /app

COPY --from=builder /app/go-service .

EXPOSE 8000

CMD ["./go-service"]