# ==========================================
# ЕТАП 1: Збірка (Builder Stage)
# ==========================================
FROM golang:1.26-alpine AS builder

# Встановлюємо робочу директорію
WORKDIR /src

# Крок 1: Кешування залежностей
# Копіюємо лише файли модулів і завантажуємо залежності
COPY go.mod go.sum ./
RUN go mod download

# Крок 2: Копіюємо решту сирцевого коду
COPY . .

# Крок 3: Компіляція статичного бінарного файлу
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/checkoutservice .

# ==========================================
# ЕТАП 2: Фінальний мінімальний образ (Runtime Stage)
# ==========================================
FROM alpine:3.20

# Встановлюємо CA-сертифікати (потрібні для HTTPS/TLS та gRPC з'єднань)
RUN apk add --no-cache ca-certificates tzdata

# Створюємо непривілейованого користувача з міркувань безпеки
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Копіюємо лише скомпільований бінарний файл з етапу builder
COPY --from=builder /app/checkoutservice /app/checkoutservice



# Змінюємо власника файлів на непривілейованого користувача
RUN chown -R appuser:appgroup /app
USER appuser


# Запуск додатка
ENTRYPOINT ["/app/checkoutservice"]