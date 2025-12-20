# ===============================
# Stage 1: Build
# ===============================
FROM golang:1.22-alpine AS builder

WORKDIR /usr/src/app

# Install only required certs for HTTPS
RUN apk add --no-cache ca-certificates

# Ensure static, reproducible Linux binary
ENV CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# Enable Go build & module cache (BuildKit)
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    mkdir -p /root/.cache/go-build

# Copy dependency files first (better caching)
COPY go.mod go.sum ./
RUN go mod download

# Copy application source
COPY . .

# Build stripped static binary
RUN go build -ldflags="-s -w" -o product-catalog .

# ===============================
# Stage 2: Runtime (FINAL IMAGE)
# ===============================
FROM gcr.io/distroless/static-debian12

WORKDIR /usr/src/app

# Copy runtime data
COPY ./products ./products

# Copy CA certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy application binary
COPY --from=builder /usr/src/app/product-catalog .

# Run as non-root user
USER nonroot:nonroot

ENV PRODUCT_CATALOG_PORT=8088

ENTRYPOINT ["./product-catalog"]









