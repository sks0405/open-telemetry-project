# ===============================
# Stage 1: Builder (Debian)
# ===============================
FROM golang:1.22 AS builder

WORKDIR /usr/src/app

# Install only required certs for HTTPS (Debian uses apt)
RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Ensure static, reproducible Linux binary
ENV CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# Copy dependency files first (better caching)
COPY go.mod go.sum ./
RUN go mod download

# Copy application source
COPY . .

# Build stripped static binary
RUN go build -trimpath -ldflags="-s -w" -o product-catalog .


# ===============================
# Stage 2: Runtime (Distroless)
# ===============================
FROM gcr.io/distroless/static-debian12

WORKDIR /usr/src/app

# Copy runtime data
COPY ./products ./products

# Copy CA certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy application binary
COPY --from=builder /usr/src/app/product-catalog .

# Distroless static already runs as non-root
USER nonroot:nonroot

ENV PRODUCT_CATALOG_PORT=8088

ENTRYPOINT ["/usr/src/app/product-catalog"]

