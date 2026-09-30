# ==========================================
# ЕТАП 1: Збірка та публікація (Build Stage)
# ==========================================
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder

WORKDIR /src

# Крок 1: Кешування залежностей NuGet
# Копіюємо проектний файл і відновлюємо пакети окремим шаром
COPY src/cartservice.csproj ./src/
RUN dotnet restore src/cartservice.csproj

# Крок 2: Копіюємо решту сирцевого коду
COPY src/ ./src/

# Крок 3: Компіляція та публікація додатку
# -c Release — оптимізована збірка для продакшену
# -o /app/published — директорія для збереження зкомпільованого результату
RUN dotnet publish src/cartservice.csproj -c Release -o /app/published --no-restore

# ==========================================
# ЕТАП 2: Фінальний образ виконання (Runtime Stage)
# ==========================================
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS runtime

WORKDIR /app

# Створюємо непривілейованого користувача з міркувань безпеки
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Копіюємо лише зкомпільовані бінарні файли з етапу builder
COPY --from=builder /app/published ./

# Змінюємо власника файлів
RUN chown -R appuser:appgroup /app
USER appuser

# Вказуємо порт та змінні середовища
EXPOSE 7070
ENV LISTEN_ADDR=0.0.0.0
ENV PORT=7070

# Запуск .NET додатку через DLL-файл
ENTRYPOINT ["dotnet", "cartservice.dll"]