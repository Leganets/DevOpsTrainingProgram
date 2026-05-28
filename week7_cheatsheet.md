# Шпаргалка: Неделя 7 — Docker + GitHub Actions Integration

## Урок 27: Docker build в GitHub Actions

### Docker Actions

**docker/setup-buildx-action** — настройка Docker Buildx
```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
```

**docker/build-push-action** — сборка и push образа
```yaml
- name: Build Docker image
  uses: docker/build-push-action@v5
  with:
    context: .
    push: false
    tags: myapp:latest
```

---

### Кеширование слоёв

```yaml
- name: Build Docker image
  uses: docker/build-push-action@v5
  with:
    context: .
    push: false
    tags: myapp:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

**Что даёт:**
- Ускорение сборки в 5-10 раз
- Переиспользование слоёв между запусками
- Экономия времени CI

**Важно:** `type=gha` (не `type-gha`!)

---

### Несколько тегов

```yaml
tags: |
  myapp:latest
  myapp:v1.0.0
  myapp:${{ github.sha }}
```

**Best practice:** latest + версия + commit SHA

---

### load vs push

**load: true** — загрузить образ в локальный Docker daemon
```yaml
push: false
load: true    # Образ доступен локально
```

**push: true** — отправить образ в registry
```yaml
push: true    # Образ в registry
load: false
```

---

### Проверка тегов

```yaml
- name: Verify image tags
  run: docker images | grep myapp
```

---

## Урок 28: Push образа в GitHub Container Registry

### GitHub Container Registry (GHCR)

**URL образа:**
```
ghcr.io/<username>/<image-name>:<tag>
```

**Пример:**
```
ghcr.io/leganets/devops-training:latest
```

---

### Логин в GHCR

```yaml
- name: Login to GitHub Container Registry
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

**GITHUB_TOKEN** — автоматический токен (не нужно создавать вручную!)

---

### Permissions

```yaml
permissions:
  contents: read
  packages: write    # Право на запись в GHCR
```

**Без permissions:** push в GHCR не сработает!

---

### Lowercase username

**Проблема:** `${{ github.repository_owner }}` возвращает `Leganets` (uppercase), а GHCR требует lowercase.

**Решение:**
```yaml
- name: Set lowercase repository owner
  id: repo
  run: echo "owner=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]')" >> $GITHUB_OUTPUT

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    tags: ghcr.io/${{ steps.repo.outputs.owner }}/myapp:latest
```

---

### Build and push

```yaml
- name: Build and push Docker image
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: |
      ghcr.io/${{ steps.repo.outputs.owner }}/devops-training:latest
      ghcr.io/${{ steps.repo.outputs.owner }}/devops-training:${{ github.sha }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

---

### Pull образа из GHCR

```bash
# Публичный образ (логин не нужен)
docker pull ghcr.io/leganets/devops-training:latest

# Приватный образ (нужен логин)
echo $GITHUB_TOKEN | docker login ghcr.io -u leganets --password-stdin
docker pull ghcr.io/leganets/devops-training:latest
```

---

## Урок 29: Multi-stage builds в production

### Что такое Multi-stage builds?

**Multi-stage build** — Dockerfile с несколькими стадиями (FROM), где каждая стадия выполняет свою задачу.

**Идея:**
1. **Stage 1 (Builder)** — собрать зависимости, скомпилировать код
2. **Stage 2 (Runtime)** — скопировать только готовые файлы

**Результат:**
- ✅ Образ меньше (в 5-10 раз)
- ✅ Нет build-зависимостей в runtime
- ✅ Безопаснее

---

### Пример Multi-stage Dockerfile

```dockerfile
# Stage 1: Builder
FROM python:3.9 AS builder

WORKDIR /app

RUN apt-get update && apt-get install -y gcc

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.9-slim

WORKDIR /app

# Скопировать установленные пакеты из builder
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin

COPY app/ .

RUN useradd -m appuser
USER appuser

CMD ["python", "api.py"]
```

**Что происходит:**
1. Builder: устанавливаем gcc, собираем зависимости
2. Runtime: копируем только готовые пакеты, без gcc
3. Результат: образ содержит только runtime зависимости

---

### Alpine vs Slim

| Критерий | Alpine | Slim |
|----------|--------|------|
| Размер базового образа | ~5MB | ~40MB |
| libc | musl | glibc |
| Совместимость | Средняя | Высокая |
| Скорость установки пакетов | Медленная (компиляция) | Быстрая (prebuilt wheels) |
| Build-зависимости | Нужны часто | Нужны редко |

**Когда использовать:**
- **Alpine:** Чистый Python без C-расширений, нужен минимальный размер
- **Slim:** Python с C-расширениями (numpy, pandas, cryptography), нужна совместимость

---

### gcc — зачем нужен?

**gcc** — GNU Compiler Collection (компилятор C/C++).

**Зачем:** Многие Python пакеты содержат C-расширения (код на C для скорости). Чтобы установить такие пакеты, нужно их скомпилировать.

**Примеры пакетов с C-расширениями:**
- cryptography
- numpy
- pandas
- pillow
- psycopg2

**Почему gcc только в builder:**
- Builder: компилируем пакеты
- Runtime: пакеты уже скомпилированы, gcc не нужен

---

### python vs python3

**В Docker образах `python:3.9`:**
```bash
python   → Python 3.9  ✅
python3  → Python 3.9  ✅
```

Оба работают! Образ настроен так, что `python` = `python3`.

**Best practice:** Используй `python` (короче).

---

### Копирование между стадиями

```dockerfile
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
```

**Важно:** Копировать из `/usr/local` (системная директория), а не из `/root/.local` (недоступна для других пользователей).

---

### Сравнение размеров

**Обычный Dockerfile:**
```
python:3.9 → 1.01GB
+ gcc, build-essential → +200MB
+ pip cache → +50MB
= 1.26GB
```

**Multi-stage Dockerfile:**
```
python:3.9-slim → 125MB
+ только runtime пакеты → +50MB
= 175MB
```

**Экономия:** 1.26GB → 175MB (в 7 раз меньше!)

---

## Урок 30: Docker Compose в production

### Структура docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    image: ghcr.io/leganets/devops-training:latest
    container_name: devops-app
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      - ENV=production
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://127.0.0.1:5000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

---

### Restart policies

```yaml
restart: no              # Не перезапускать (по умолчанию)
restart: always          # Всегда перезапускать
restart: on-failure      # Перезапускать только при ошибке
restart: unless-stopped  # Перезапускать, кроме ручной остановки
```

**Для production:** `unless-stopped` (лучший выбор)

**Почему:**
- ✅ Перезапускается после падения
- ✅ Перезапускается после reboot сервера
- ✅ НЕ перезапускается после `docker-compose down`

---

### Health checks

```yaml
healthcheck:
  test: ["CMD", "wget", "--spider", "http://127.0.0.1:5000/health"]
  interval: 30s      # Проверять каждые 30 секунд
  timeout: 3s        # Таймаут 3 секунды
  start_period: 5s   # Не проверять первые 5 секунд
  retries: 3         # 3 попытки перед пометкой unhealthy
```

**Зачем:**
- ✅ Автоматический мониторинг здоровья
- ✅ Видно статус через `docker-compose ps`

---

### Environment variables

**Способ 1: Прямо в docker-compose.yml**
```yaml
environment:
  - DATABASE_URL=postgresql://user:pass@db:5432/mydb
  - DEBUG=false
```

**Способ 2: Из .env файла**
```yaml
env_file:
  - .env
```

`.env`:
```
DATABASE_URL=postgresql://user:pass@db:5432/mydb
DEBUG=false
```

**Best practice:** Используй `.env` файл (не коммитить в Git!)

---

### Depends_on

```yaml
services:
  db:
    image: postgres
  
  app:
    depends_on:
      - db    # Запустить db перед app
```

**Что делает:**
- ✅ Определяет порядок запуска
- ❌ НЕ ждёт, пока db будет готов (только запущен)

---

### Networks

**По умолчанию:** Все сервисы в одной сети, видят друг друга по имени.

```yaml
services:
  app:
    # Доступен как http://app:5000
  
  db:
    # Доступен как postgresql://db:5432
```

---

### Nginx reverse proxy

**nginx.conf:**
```nginx
upstream flask_app {
    server app:5000;
}

server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://flask_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /health {
        proxy_pass http://flask_app/health;
    }
}
```

**docker-compose.yml:**
```yaml
services:
  app:
    expose:
      - "5000"    # Не публиковать наружу
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"    # Nginx доступен снаружи
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - app
    networks:
      - app-network
```

**Зачем:**
- ✅ Nginx обрабатывает статику быстрее
- ✅ SSL termination
- ✅ Load balancing
- ✅ Скрыть Flask от прямого доступа

---

## Основные команды

### Docker Compose

```bash
# Запустить все сервисы
docker-compose up -d

# Остановить все сервисы
docker-compose down

# Проверить статус
docker-compose ps

# Посмотреть логи
docker-compose logs app
docker-compose logs --tail=20 app

# Перезапустить сервис
docker-compose restart app

# Выполнить команду в контейнере
docker-compose exec app sh

# Проверить конфигурацию
docker-compose config
```

---

### Docker inspect

```bash
# Проверить RestartCount
docker inspect devops-app --format='{{.RestartCount}}'

# Проверить время запуска
docker inspect devops-app --format='{{.State.StartedAt}}'

# Проверить статус
docker inspect devops-app --format='{{.State.Status}}'
```

---

### Docker build

```bash
# Собрать образ
docker build -t myapp:latest .

# Собрать multi-stage образ
docker build -t myapp:multistage -f Dockerfile.multistage .

# Собрать только builder stage
docker build --target builder -t myapp-builder .

# Сравнить размеры
docker images | grep myapp
```

---

## Типичные ошибки (не повторяй!)

1. ❌ `cache-from: type-gha` (дефис вместо =) → `cache-from: type=gha`
2. ❌ `container-name` (дефис) → `container_name` (подчёркивание)
3. ❌ Uppercase username в GHCR → использовать `tr '[:upper:]' '[:lower:]'`
4. ❌ Копировать из `/root/.local` для appuser → копировать из `/usr/local`
5. ❌ `apt-get install gcc` без `-y` → `apt-get install -y gcc`
6. ❌ Забыть `permissions: packages: write` для GHCR
7. ❌ Использовать `push: true` без `docker/login-action`
8. ❌ Забыть `EXPOSE` и `CMD` в multi-stage Dockerfile

---

## Best Practices

### Docker

1. ✅ Используй Alpine/Slim образы (меньше размер)
2. ✅ Копируй requirements.txt отдельно (кеш)
3. ✅ Используй `--no-cache-dir` для pip
4. ✅ Не запускай от root (создай appuser)
5. ✅ Используй .dockerignore
6. ✅ Multi-stage builds для production
7. ✅ Добавляй health checks

### GitHub Actions

1. ✅ Используй кеширование (`cache-from`, `cache-to`)
2. ✅ Несколько тегов (latest + версия + SHA)
3. ✅ Lowercase username для GHCR
4. ✅ Permissions для GITHUB_TOKEN
5. ✅ Интеграция с CI (lint → test → build)

### Docker Compose

1. ✅ `restart: unless-stopped` для production
2. ✅ Health checks для мониторинга
3. ✅ `.env` файл для секретов (не коммитить!)
4. ✅ Nginx reverse proxy перед приложением
5. ✅ `expose` вместо `ports` для внутренних сервисов
6. ✅ `depends_on` для порядка запуска

---

**Повтори эту шпаргалку перед экзаменом!**


