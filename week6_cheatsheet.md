# Шпаргалка: Неделя 6 — Git + CI/CD

## Урок 21: Git Basics

### Основные команды Git

**Инициализация репозитория:**
```bash
git init                    # Создать новый репозиторий
git clone <url>             # Клонировать существующий репозиторий
```

**Базовый workflow:**
```bash
git status                  # Показать статус файлов
git add <file>              # Добавить файл в staging area
git add .                   # Добавить все изменённые файлы
git commit -m "message"     # Создать коммит
git log                     # Показать историю коммитов
git log --oneline           # Краткая история
```

---

### Staging Area (индекс)

**Три состояния файлов:**
1. **Working Directory** — рабочая директория (изменённые файлы)
2. **Staging Area** — индекс (файлы готовые к коммиту)
3. **Repository** — репозиторий (закоммиченные файлы)

```bash
# Файл изменён → Working Directory
git add file.txt            # → Staging Area
git commit -m "message"     # → Repository
```

**Зачем нужен staging area?**
- Выборочный коммит (не все изменения сразу)
- Проверка перед коммитом
- Разделение логических изменений

---

### Ветки (branches)

**Создание и переключение:**
```bash
git branch                  # Показать все ветки
git branch feature          # Создать ветку feature
git checkout feature        # Переключиться на ветку feature
git checkout -b feature     # Создать и переключиться (одной командой)
```

**Слияние веток (merge):**
```bash
git checkout main           # Переключиться на main
git merge feature           # Слить feature в main
```

**Удаление ветки:**
```bash
git branch -d feature       # Удалить ветку (безопасно)
git branch -D feature       # Удалить ветку (принудительно)
```

---

### Просмотр изменений

```bash
git diff                    # Изменения в working directory
git diff --staged           # Изменения в staging area
git diff HEAD               # Все изменения (staged + unstaged)
git diff main..feature      # Разница между ветками
```

---

### Отмена изменений

```bash
git restore <file>          # Отменить изменения в working directory
git restore --staged <file> # Убрать файл из staging area
git checkout -- <file>      # Отменить изменения (старый синтаксис)
```

---

## Урок 22: Git Advanced

### git stash — временное сохранение

```bash
git stash                   # Сохранить изменения
git stash list              # Показать список stash
git stash apply             # Применить последний stash (не удаляет)
git stash pop               # Применить и удалить последний stash
git stash drop              # Удалить последний stash
git stash clear             # Удалить все stash
```

**Когда использовать:**
- Нужно переключиться на другую ветку, но изменения не готовы к коммиту
- Временно убрать изменения

---

### git reset — откат коммитов

**Три режима:**

**1. git reset --soft**
```bash
git reset --soft HEAD~1     # Откатить коммит, оставить изменения в staging
```
- Коммит удаляется
- Изменения остаются в staging area
- Можно сразу сделать новый коммит

**2. git reset --mixed (по умолчанию)**
```bash
git reset HEAD~1            # Откатить коммит, оставить изменения в working directory
git reset --mixed HEAD~1    # То же самое
```
- Коммит удаляется
- Изменения остаются в working directory (не в staging)
- Нужно снова сделать `git add`

**3. git reset --hard**
```bash
git reset --hard HEAD~1     # Откатить коммит, удалить все изменения
```
- Коммит удаляется
- Изменения удаляются полностью
- **ОПАСНО!** Невозможно восстановить

**Таблица:**
| Режим | Коммит | Staging | Working Directory |
|-------|--------|---------|-------------------|
| `--soft` | Удалён | Сохранён | Сохранён |
| `--mixed` | Удалён | Очищен | Сохранён |
| `--hard` | Удалён | Очищен | Очищен |

---

### git rebase — перебазирование

**Что делает:** Переносит коммиты на новую базу.

```bash
git checkout feature
git rebase main             # Перебазировать feature на main
```

**Разница rebase vs merge:**

| merge | rebase |
|-------|--------|
| Создаёт merge commit | Линейная история |
| История сохраняется | История переписывается |
| Безопасно | Опасно для публичных веток |

**Правило:** Никогда не делай rebase публичных веток (main, master)!

---

### git cherry-pick — выборочное применение коммитов

```bash
git cherry-pick <commit-sha>    # Применить конкретный коммит
```

**Когда использовать:**
- Нужен только один коммит из другой ветки
- Hotfix из одной ветки в другую

---

### git revert — безопасная отмена коммита

```bash
git revert <commit-sha>     # Создать новый коммит, отменяющий изменения
```

**Разница revert vs reset:**
- `reset` — удаляет коммит из истории (опасно)
- `revert` — создаёт новый коммит, отменяющий изменения (безопасно)

---

### git reflog — история всех действий

```bash
git reflog                  # Показать все действия (даже удалённые коммиты)
git reset --hard <commit>   # Восстановить удалённый коммит
```

**Когда использовать:**
- Случайно сделал `git reset --hard`
- Нужно восстановить удалённый коммит

---

## Урок 23: GitHub Actions + Remote

### Git Remote

**Добавление remote:**
```bash
git remote add origin <url>         # Добавить remote
git remote -v                       # Показать все remotes
git remote remove origin            # Удалить remote
```

**Push и Pull:**
```bash
git push origin main                # Отправить изменения на GitHub
git push -u origin main             # Отправить и установить upstream
git pull origin main                # Скачать изменения с GitHub
git fetch origin                    # Скачать без слияния
```

---

### GitHub Actions Basics

**Структура workflow:**
```yaml
name: Workflow Name

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  job-name:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run command
        run: echo "Hello World"
```

**Основные элементы:**
- `name` — название workflow
- `on` — триггеры (push, pull_request, workflow_dispatch)
- `jobs` — задачи
- `runs-on` — ОС для выполнения (ubuntu-latest, windows-latest, macos-latest)
- `steps` — шаги выполнения

---

### Triggers (триггеры)

```yaml
# Push в ветку
on:
  push:
    branches:
      - main
      - develop

# Pull Request
on:
  pull_request:
    branches:
      - main

# Ручной запуск
on:
  workflow_dispatch:

# По расписанию (cron)
on:
  schedule:
    - cron: '0 0 * * *'    # Каждый день в 00:00

# Несколько триггеров
on:
  push:
    branches:
      - main
  pull_request:
  workflow_dispatch:
```

---

### Actions (готовые действия)

```yaml
# Checkout кода
- uses: actions/checkout@v4

# Установка Python
- uses: actions/setup-python@v5
  with:
    python-version: '3.9'

# Установка Node.js
- uses: actions/setup-node@v4
  with:
    node-version: '18'

# Кеширование зависимостей
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
```

---

## Урок 24: CI Pipeline

### CI Pipeline структура

**Типичный CI pipeline:**
1. **Lint** — проверка стиля кода
2. **Test** — запуск тестов
3. **Build** — сборка приложения

```yaml
name: CI Pipeline

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: pip install flake8
      - name: Lint
        run: flake8 app/
  
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests
        run: pytest tests/
  
  build:
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: echo "Building application..."
```

---

### Кеширование зависимостей

```yaml
- name: Cache pip dependencies
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

**Зачем:**
- Ускорение CI (не скачивать зависимости каждый раз)
- Экономия времени

---

### Code Coverage

```yaml
- name: Run tests with coverage
  run: pytest --cov=app tests/

- name: Generate coverage report
  run: pytest --cov=app --cov-report=html tests/
```

---

## Урок 25: CD Pipeline

### Environments (окружения)

**Создание environment в GitHub:**
1. Settings → Environments
2. New environment
3. Настроить protection rules (manual approval, reviewers)

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production    # Использовать environment
    
    steps:
      - name: Deploy
        run: echo "Deploying to production..."
```

---

### Manual Approval

```yaml
name: CD - Production

on:
  workflow_dispatch:    # Ручной запуск

jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment: production    # Требует manual approval
    
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: echo "Deployed to production"
```

---

### Rollback Workflow

```yaml
name: Rollback

on:
  workflow_dispatch:
    inputs:
      commit_sha:
        description: 'Commit SHA to rollback to'
        required: true
        type: string

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          ref: ${{ inputs.commit_sha }}
          fetch-depth: 0    # Скачать всю историю
      
      - name: Rollback
        run: echo "Rolled back to ${{ inputs.commit_sha }}"
```

---

### GitHub Secrets

**Создание секретов:**
1. Settings → Secrets and variables → Actions
2. New repository secret
3. Добавить имя и значение

**Использование в workflow:**
```yaml
- name: Deploy via SSH
  uses: appleboy/ssh-action@v1.0.0
  with:
    host: ${{ secrets.SSH_HOST }}
    username: ${{ secrets.SSH_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd /var/www/app
      git pull origin main
```

---

## Урок 26: Dockerfile для Python

### Базовый Dockerfile

```dockerfile
FROM python:3.9-alpine

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/

RUN adduser -D appuser
USER appuser

CMD ["python", "app/main.py"]
```

---

### Multi-stage Build

```dockerfile
# Stage 1: Build
FROM python:3.9 AS builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.9-alpine

WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY app/ ./app/

ENV PATH=/root/.local/bin:$PATH

RUN adduser -D appuser
USER appuser

CMD ["python", "app/main.py"]
```

---

### Health Check

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://127.0.0.1:5000/health || exit 1
```

**Параметры:**
- `--interval` — интервал проверки (30s)
- `--timeout` — таймаут проверки (3s)
- `--start-period` — период запуска (5s)
- `--retries` — количество попыток (3)

---

### .dockerignore

```
__pycache__/
*.pyc
*.pyo
.git/
.github/
.env
venv/
.pytest_cache/
*.log
```

---

### Best Practices

1. ✅ Используй Alpine образы (меньше размер)
2. ✅ Копируй requirements.txt отдельно (кеш)
3. ✅ Используй `--no-cache-dir` для pip
4. ✅ Не запускай от root (создай appuser)
5. ✅ Используй .dockerignore
6. ✅ Multi-stage builds для production
7. ✅ Добавляй health checks

---

## Полезные команды для экзамена

### Git

```bash
# Базовые
git init
git add .
git commit -m "message"
git status
git log --oneline

# Ветки
git branch feature
git checkout feature
git checkout -b feature
git merge feature

# Advanced
git stash
git stash pop
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
git rebase main
git cherry-pick <sha>
git revert <sha>
git reflog

# Remote
git remote add origin <url>
git push -u origin main
git pull origin main
```

---

### Docker

```bash
# Сборка
docker build -t myapp:latest .
docker build -t myapp:v1.0 .

# Запуск
docker run myapp
docker run -d myapp
docker run -d -p 8080:80 myapp
docker run -d --name app myapp

# Управление
docker ps
docker ps -a
docker logs app
docker exec -it app bash
docker stop app
docker rm app

# Образы
docker images
docker rmi myapp
docker system prune -a
```

---

## Типичные ошибки (не повторяй!)

1. ❌ Забыть `git add` перед `git commit`
2. ❌ Делать `git reset --hard` без backup
3. ❌ Делать `git rebase` на публичных ветках
4. ❌ Забыть `-u` при первом `git push`
5. ❌ Использовать `main` вместо `master` (или наоборот)
6. ❌ Забыть `fetch-depth: 0` для rollback
7. ❌ Забыть точку в `docker build -t myapp .`
8. ❌ Опечатка `addused` вместо `adduser`
9. ❌ Использовать `localhost` вместо `127.0.0.1` в health check
10. ❌ Забыть `--no-cache-dir` для pip

---

**Повтори эту шпаргалку перед экзаменом!**
