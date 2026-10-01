# ==========================================
# ЕТАП 1: Збірка (Builder Stage)
# ==========================================
FROM golang:1.26-alpine AS builder

WORKDIR /src

# 1. Завантаження залежностей
COPY go.mod go.sum ./
RUN go mod download

# 2. Копіюємо сирцевий код
COPY . .

# 3. Компіляція бінарного файлу у /src/productcatalogservice
RUN CGO_ENABLED=0 GOOS=linux go build -o productcatalogservice .

# ==========================================
# ЕТАП 2: Фінальний образ (Runtime Stage)
# ==========================================
FROM alpine:3.20

WORKDIR /app

RUN apk add --no-cache ca-certificates tzdata
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Копіюємо бінарник із директорії /src першого етапу
COPY --from=builder /src/productcatalogservice /app/productcatalogservice

# Копіюємо JSON-файл з каталогом товарів з директорії /src
COPY --from=builder /src/products.json /app/products.json

RUN chown -R appuser:appgroup /app
USER appuser

ENTRYPOINT ["/app/productcatalogservice"]