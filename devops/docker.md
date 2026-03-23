# 🐳 Docker

## Уровень владения: ⭐⭐⭐⭐ Advanced

## Опыт: 3+ года

---

## 📝 Описание

Docker - основной инструмент для контейнеризации приложений. Использую для разработки, тестирования и деплоя в production.

## 🎯 Специализация

### Основные навыки
- **Dockerfile** - создание образов
- **Docker Compose** - multi-container приложения
- **Volumes** - управление данными
- **Networks** - сетевое взаимодействие
- **Multi-stage builds** - оптимизация образов

### Production
- **Health checks** - мониторинг контейнеров
- **Resource limits** - ограничение ресурсов
- **Logging** - централизованные логи
- **Security** - безопасность контейнеров

## 💼 Проекты

### 1. Telegram Bot в Docker
**Описание:** Контейнеризация Python приложения

**Dockerfile:**
```dockerfile
# Multi-stage build для оптимизации
FROM python:3.12-slim as builder

WORKDIR /app

# Установка uv для быстрой установки зависимостей
RUN pip install uv

# Копирование зависимостей
COPY pyproject.toml uv.lock ./

# Установка зависимостей
RUN uv sync --frozen

# Production stage
FROM python:3.12-slim

WORKDIR /app

# Копирование зависимостей из builder
COPY --from=builder /app/.venv /app/.venv

# Копирование кода
COPY src/ ./src/

# Создание non-root пользователя
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD python -c "import requests; requests.get('http://localhost:8000/health')"

# Запуск
CMD ["/app/.venv/bin/python", "-m", "src.main"]
```

### 2. Docker Compose для разработки
**Описание:** Multi-container окружение

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Telegram Bot
  bot:
    build:
      context: .
      dockerfile: Dockerfile
    env_file:
      - .env
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./src:/app/src  # Hot reload для разработки
    restart: unless-stopped
    networks:
      - app-network

  # PostgreSQL
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Redis
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    networks:
      - app-network

  # nginx (reverse proxy)
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - bot
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### 3. Production Dockerfile
**Описание:** Оптимизированный образ для production

**Dockerfile:**
```dockerfile
FROM python:3.12-slim

# Установка системных зависимостей
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Копирование только requirements для кеширования
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копирование кода
COPY . .

# Security: non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

# Labels для метаданных
LABEL maintainer="consttokarenko@gmail.com"
LABEL version="1.0"
LABEL description="Telegram AI Bot"

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD python -c "import sys; sys.exit(0)"

# Запуск
CMD ["python", "-m", "src.main"]
```

## 🛠️ Инструменты

### Docker CLI
```bash
# Основные команды
docker build -t myapp:latest .
docker run -d -p 8000:8000 myapp:latest
docker ps
docker logs -f container_id
docker exec -it container_id bash

# Очистка
docker system prune -a
docker volume prune
```

### Docker Compose
```bash
# Запуск
docker-compose up -d

# Просмотр логов
docker-compose logs -f

# Перезапуск сервиса
docker-compose restart bot

# Остановка
docker-compose down

# Пересборка
docker-compose up -d --build
```

## 📚 Best Practices

### 1. Оптимизация размера образа
```dockerfile
# Плохо: большой образ
FROM python:3.12
COPY . .
RUN pip install -r requirements.txt

# Хорошо: slim образ + multi-stage
FROM python:3.12-slim as builder
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.12-slim
COPY --from=builder /root/.local /root/.local
COPY src/ ./src/
```

### 2. Кеширование слоёв
```dockerfile
# Плохо: каждое изменение кода пересобирает всё
COPY . .
RUN pip install -r requirements.txt

# Хорошо: зависимости кешируются
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

### 3. .dockerignore
```
# .dockerignore
__pycache__/
*.pyc
.git/
.env
.venv/
*.log
.pytest_cache/
.mypy_cache/
```

### 4. Resource limits
```yaml
services:
  bot:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

## 🎓 Продвинутые техники

### 1. Multi-stage builds
```dockerfile
# Stage 1: Build
FROM node:18 as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```

### 2. Health checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8000/health || exit 1
```

### 3. Secrets management
```yaml
services:
  bot:
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    file: ./secrets/api_key.txt
```

## 📖 Ресурсы

- [Docker Documentation](https://docs.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)

---

**Последнее обновление:** 2026-03-23
