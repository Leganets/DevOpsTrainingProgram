# Шпаргалка: Неделя 5 — Docker

## Урок 17: Введение в Docker

### Что такое Docker?

**Docker** — платформа для запуска приложений в изолированных контейнерах.

**Контейнер** — лёгкий, изолированный процесс, содержащий всё необходимое для работы приложения.

---

### Основные понятия

**Image (образ)** — шаблон для создания контейнера (неизменяемый)
- Пример: `nginx:latest`, `python:3.9`, `mysql:8.0`

**Container (контейнер)** — запущенный экземпляр образа
- Создаётся из образа
- Изолированный процесс

**Аналогия:**
- Image = класс
- Container = объект (экземпляр класса)

---

### Docker vs Виртуальные машины

| Виртуальная машина | Docker контейнер |
|-------------------|------------------|
| Полная ОС (несколько GB) | Только приложение (несколько MB) |
| Медленный старт (минуты) | Быстрый старт (секунды) |
| Тяжёлая | Лёгкий |

---

### Базовые команды Docker

**Проверить версию:**
```bash
docker --version
docker version
```

**Запустить контейнер:**
```bash
docker run hello-world
docker run -d nginx                    # В фоне (detached)
docker run -d --name my-nginx nginx    # С именем
docker run -d -p 8080:80 nginx         # С пробросом порта
```

**Флаги:**
- `-d` — detached (фоновый режим)
- `--name` — задать имя контейнера
- `-p HOST:CONTAINER` — пробросить порт

---

**Список контейнеров:**
```bash
docker ps              # Запущенные
docker ps -a           # Все (включая остановленные)
```

**Список образов:**
```bash
docker images
```

---

**Управление контейнерами:**
```bash
docker stop <container>      # Остановить
docker start <container>     # Запустить остановленный
docker restart <container>   # Перезапустить
docker rm <container>        # Удалить (должен быть остановлен)
docker rm -f <container>     # Удалить принудительно
```

---

**Логи контейнера:**
```bash
docker logs <container>
docker logs -f <container>           # В реальном времени
docker logs --tail 20 <container>    # Последние 20 строк
```

---

**Выполнить команду в контейнере:**
```bash
docker exec <container> ls /app
docker exec -it <container> bash     # Интерактивный shell
```

**Флаги:**
- `-i` — interactive
- `-t` — tty (терминал)

---

**Информация о контейнере:**
```bash
docker inspect <container>    # Детальная информация (JSON)
docker stats                  # Использование ресурсов
```

---

**Удалить образ:**
```bash
docker rmi <image>
```

---

**Очистка:**
```bash
docker container prune    # Удалить все остановленные контейнеры
docker image prune        # Удалить неиспользуемые образы
docker system prune -a    # Удалить всё
docker system df          # Использование диска
```

---

### Жизненный цикл контейнера

```
docker run    → Создать и запустить
docker stop   → Остановить
docker start  → Запустить остановленный
docker restart → Перезапустить
docker rm     → Удалить
```

---

### Docker Hub

**Docker Hub** — реестр образов Docker (https://hub.docker.com)

**Поиск образов:**
```bash
docker search nginx
```

**Скачать образ:**
```bash
docker pull nginx:latest
docker pull python:3.9
docker pull python:3.9-alpine
```

**Формат:** `IMAGE:TAG`

---

## Урок 18: Dockerfile — создание своих образов

### Что такое Dockerfile?

**Dockerfile** — текстовый файл с инструкциями для создания Docker образа.

---

### Основные инструкции

#### 1. FROM — базовый образ

```dockerfile
FROM ubuntu:22.04
FROM python:3.9
FROM node:18
FROM nginx:latest
```

**Важно:** `FROM` должна быть первой инструкцией.

---

#### 2. RUN — выполнить команду при сборке

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN pip install flask
```

**Лучше объединять:**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl vim && \
    rm -rf /var/lib/apt/lists/*
```

---

#### 3. COPY — скопировать файлы

```dockerfile
COPY app.py /app/app.py
COPY . /app
COPY requirements.txt /app/
```

**Формат:** `COPY <источник> <назначение>`

---

#### 4. WORKDIR — рабочая директория

```dockerfile
WORKDIR /app
```

**Аналог:** `cd /app`

---

#### 5. CMD — команда запуска контейнера

```dockerfile
CMD ["python3", "app.py"]
CMD ["nginx", "-g", "daemon off;"]
```

**Важно:** Только один `CMD` в Dockerfile.

**Формат:**
```dockerfile
# ✅ Exec форма (рекомендуется)
CMD ["python3", "app.py"]

# ⚠️ Shell форма
CMD python3 app.py
```

---

#### 6. EXPOSE — документировать порт

```dockerfile
EXPOSE 80
EXPOSE 3000
```

**Важно:** `EXPOSE` **НЕ пробрасывает** порт! Это только документация.

---

#### 7. ENV — переменные окружения

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
```

---

#### 8. ADD — скопировать файлы (расширенная версия COPY)

```dockerfile
ADD app.tar.gz /app/              # Автоматически распаковывает
ADD https://example.com/file.txt /app/
```

**Рекомендация:** Используй `COPY` вместо `ADD`, если не нужна распаковка.

---

#### 9. ENTRYPOINT — точка входа

```dockerfile
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

---

### Пример Dockerfile

**Простой Python:**
```dockerfile
FROM python:3.9

WORKDIR /app

COPY app.py /app/

CMD ["python3", "app.py"]
```

**Flask приложение:**
```dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt /app/
RUN pip install -r requirements.txt

COPY app.py /app/

EXPOSE 5000

CMD ["python3", "app.py"]
```

---

### Команды для работы с образами

**Собрать образ:**
```bash
docker build -t myapp .
docker build -t myapp:v1.0 .
```

**Флаги:**
- `-t` — tag (имя образа)
- `.` — путь к Dockerfile

---

**Посмотреть историю образа:**
```bash
docker history myapp
```

---

### Слои (layers)

Каждая инструкция в Dockerfile создаёт **слой** (layer).

**Пример:**
```dockerfile
FROM ubuntu:22.04        # Слой 1
RUN apt-get update       # Слой 2
COPY app.py /app/        # Слой 3
```

**Важно:**
- Слои кешируются
- Слои неизменяемые (read-only)

---

### .dockerignore

Создай файл `.dockerignore`:

```
node_modules
.git
*.log
.env
__pycache__
```

**Аналог:** `.gitignore`

---

## Урок 19: CMD vs ENTRYPOINT и оптимизация

### CMD vs ENTRYPOINT

| CMD | ENTRYPOINT |
|-----|-----------|
| Можно переопределить при запуске | Нельзя переопределить (только дополнить) |
| Команда по умолчанию | Фиксированная команда |

---

### CMD — команда по умолчанию

```dockerfile
FROM ubuntu:22.04
CMD ["echo", "Hello"]
```

**Запуск:**
```bash
docker run myimage              # Выполнит: echo "Hello"
docker run myimage ls /app      # Выполнит: ls /app (CMD переопределён!)
```

---

### ENTRYPOINT — точка входа

```dockerfile
FROM ubuntu:22.04
ENTRYPOINT ["echo", "Hello"]
```

**Запуск:**
```bash
docker run myimage              # Выполнит: echo "Hello"
docker run myimage test         # Выполнит: echo "Hello" test
```

---

### Комбинация ENTRYPOINT + CMD

**Лучшая практика:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY app.py /app/

ENTRYPOINT ["python3"]
CMD ["app.py"]
```

**Запуск:**
```bash
docker run myimage              # python3 app.py
docker run myimage test.py      # python3 test.py
```

---

## Оптимизация Docker образов

### Решение 1: Объединять RUN команды

**❌ Плохо:**
```dockerfile
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y curl
```

**✅ Хорошо:**
```dockerfile
RUN apt-get update && \
    apt-get install -y python3 curl && \
    rm -rf /var/lib/apt/lists/*
```

---

### Решение 2: Использовать Alpine Linux

**Alpine Linux** — минималистичный дистрибутив (5MB вместо 80MB Ubuntu).

**Сравнение:**
```dockerfile
# Ubuntu (900MB)
FROM python:3.9

# Alpine (50MB)
FROM python:3.9-alpine
```

**В Alpine используется `apk`:**
```dockerfile
FROM alpine:3.18
RUN apk add --no-cache python3 curl
```

---

### Решение 3: Multi-stage builds

**Проблема:** Для сборки нужны инструменты, но в production они не нужны.

**Решение:**
```dockerfile
# Этап 1: Сборка
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Этап 2: Production
FROM alpine:3.18
WORKDIR /app
COPY --from=builder /app/myapp /app/
CMD ["./myapp"]
```

**Результат:** 10MB вместо 800MB.

---

### Решение 4: .dockerignore

```
node_modules
.git
*.log
.env
__pycache__
```

---

### Решение 5: Порядок инструкций (кеширование)

**❌ Плохо:**
```dockerfile
FROM python:3.9
COPY . /app                    # Любое изменение → пересборка всего
RUN pip install -r requirements.txt
```

**✅ Хорошо:**
```dockerfile
FROM python:3.9
COPY requirements.txt /app/    # Копируем только зависимости
RUN pip install -r requirements.txt  # Кеш сохраняется
COPY . /app                    # Копируем код
```

**Правило:** Копируй **сначала зависимости**, **потом код**.

---

### Оптимизированный Python Dockerfile

```dockerfile
FROM python:3.9-alpine

WORKDIR /app

COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

COPY . /app

EXPOSE 5000

CMD ["python3", "app.py"]
```

**Оптимизации:**
- ✅ Alpine вместо Ubuntu (900MB → 50MB)
- ✅ `--no-cache-dir` для pip
- ✅ Копируем `requirements.txt` отдельно (кеш)

---

## Урок 20: Docker Compose

### Что такое Docker Compose?

**Docker Compose** — инструмент для управления многоконтейнерными приложениями через YAML файл.

---

### Структура docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "80:80"
  
  app:
    build: ./app
    environment:
      - NODE_ENV=production
```

---

### Основные параметры сервиса

#### 1. image — использовать готовый образ

```yaml
services:
  web:
    image: nginx:latest
```

---

#### 2. build — собрать образ из Dockerfile

```yaml
services:
  app:
    build: .
    # ИЛИ
    build:
      context: ./app
      dockerfile: Dockerfile.prod
```

---

#### 3. ports — проброс портов

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
      - "443:443"
```

---

#### 4. environment — переменные окружения

```yaml
services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
```

**Или через файл:**
```yaml
services:
  db:
    image: postgres
    env_file:
      - .env
```

---

#### 5. volumes — постоянные данные

```yaml
services:
  db:
    image: postgres
    volumes:
      - db_data:/var/lib/postgresql/data    # Named volume
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # Bind mount

volumes:
  db_data:
```

**Типы volumes:**
- **Named volume** — управляется Docker
- **Bind mount** — папка хоста

---

#### 6. depends_on — зависимости

```yaml
services:
  app:
    image: myapp
    depends_on:
      - db
      - cache
  
  db:
    image: postgres
  
  cache:
    image: redis
```

**Важно:** `depends_on` только определяет **порядок запуска**, не ждёт готовности!

---

#### 7. restart — политика перезапуска

```yaml
services:
  app:
    image: myapp
    restart: always
```

**Варианты:**
- `no` — не перезапускать
- `always` — всегда перезапускать
- `on-failure` — только при ошибке
- `unless-stopped` — пока не остановлен вручную

---

#### 8. container_name — имя контейнера

```yaml
services:
  web:
    image: nginx
    container_name: my-nginx
```

---

### Команды Docker Compose

**Запустить все сервисы:**
```bash
docker-compose up
docker-compose up -d          # В фоне
```

**Остановить все сервисы:**
```bash
docker-compose down
docker-compose down -v        # Удалить volumes
```

**Посмотреть запущенные сервисы:**
```bash
docker-compose ps
```

**Посмотреть логи:**
```bash
docker-compose logs
docker-compose logs app       # Конкретного сервиса
docker-compose logs -f        # В реальном времени
```

**Перезапустить сервис:**
```bash
docker-compose restart app
```

**Остановить сервис:**
```bash
docker-compose stop app
```

**Выполнить команду в контейнере:**
```bash
docker-compose exec app bash
docker-compose exec db psql -U postgres
```

**Собрать образы:**
```bash
docker-compose build
docker-compose build --no-cache
```

**Масштабирование:**
```bash
docker-compose up -d --scale app=3
```

---

### Пример: Nginx + статический сайт

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

**Запуск:**
```bash
docker-compose up -d
curl http://localhost:8080
```

---

### Пример: Flask + Redis

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - redis
  
  redis:
    image: redis:alpine
```

---

### Пример: WordPress + MySQL

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
    volumes:
      - db_data:/var/lib/mysql
  
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: rootpass
    depends_on:
      - db

volumes:
  db_data:
```

---

## Типичные ошибки (не повторяй!)

1. ❌ Забыть `-d` в `docker run` (контейнер блокирует терминал)
2. ❌ Не пробросить порт (`-p`) и удивляться, что приложение недоступно
3. ❌ Использовать `CMD` в shell форме вместо exec формы
4. ❌ Копировать всё через `COPY . /app` без `.dockerignore`
5. ❌ Не объединять `RUN` команды (много слоёв)
6. ❌ Использовать Ubuntu вместо Alpine (большой размер)
7. ❌ Копировать код перед зависимостями (ломается кеш)
8. ❌ Путать имя сервиса и имя контейнера в `docker-compose logs`
9. ❌ Забыть `docker-compose down` (контейнеры продолжают работать)
10. ❌ Не использовать volumes для постоянных данных (данные теряются)

---

## Полезные команды для экзамена

```bash
# Базовые команды Docker
docker run -d -p 8080:80 --name web nginx
docker ps
docker ps -a
docker logs web
docker exec -it web bash
docker stop web
docker rm web
docker images
docker rmi nginx

# Dockerfile
docker build -t myapp .
docker build -t myapp:v1.0 .

# Docker Compose
docker-compose up -d
docker-compose down
docker-compose ps
docker-compose logs
docker-compose logs app
docker-compose exec app bash
docker-compose restart app

# Очистка
docker container prune
docker image prune
docker system prune -a
docker system df
```

---

## Сравнение размеров образов

| Образ | Размер |
|-------|--------|
| `ubuntu:22.04` | ~80MB |
| `alpine:3.18` | ~5MB |
| `python:3.9` | ~900MB |
| `python:3.9-alpine` | ~50MB |
| `node:18` | ~1GB |
| `node:18-alpine` | ~170MB |

**Вывод:** Всегда используй Alpine для production!

---

## Созданные проекты

1. **docker-bash-app** — простое bash приложение в Docker
2. **docker-optimize** — сравнение оптимизированного и неоптимизированного образа (1.58GB → 79.8MB)
3. **docker-compose-demo** — Nginx + статический сайт через Docker Compose

---

**Повтори эту шпаргалку перед экзаменом!**
