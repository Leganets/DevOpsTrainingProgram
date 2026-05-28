# Шпаргалка: Неделя 3 — Systemd углублённо

## Урок 9: Создание своих systemd сервисов

### Структура unit файла

**Расположение:** `/etc/systemd/system/`

**Базовая структура:**
```ini
[Unit]
Description=Описание сервиса
After=network.target

[Service]
Type=simple
ExecStart=/path/to/command
Restart=always

[Install]
WantedBy=multi-user.target
```

---

### Секция [Unit]

```ini
[Unit]
Description=My Application Server
Documentation=https://docs.example.com
After=network.target postgresql.service
Requires=postgresql.service
Wants=redis.service
```

**Параметры:**
- `Description` — описание сервиса
- `Documentation` — ссылка на документацию
- `After` — запускать ПОСЛЕ этих сервисов (порядок)
- `Before` — запускать ДО этих сервисов
- `Requires` — жёсткая зависимость (если упадёт, упадём и мы)
- `Wants` — мягкая зависимость (если упадёт, мы продолжим)

---

### Секция [Service]

```ini
[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/myapp
Environment="NODE_ENV=production"
Environment="PORT=3000"
ExecStart=/usr/bin/node /opt/myapp/server.js
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
```

**Type — тип сервиса:**
- `simple` — процесс запускается и работает (по умолчанию)
- `forking` — процесс создаёт дочерний процесс (nginx, apache)
- `oneshot` — процесс выполняется один раз и завершается (скрипты)
- `notify` — процесс уведомляет systemd о готовности

**ВАЖНО:** `Type=oneshot` **несовместим** с `Restart=always`!

**User/Group:**
```ini
User=www-data
Group=www-data
```

**WorkingDirectory:**
```ini
WorkingDirectory=/opt/myapp
```

**Environment:**
```ini
Environment="NODE_ENV=production"
Environment="DATABASE_URL=postgres://localhost/mydb"
```

**ExecStart — команда запуска (обязательный):**
```ini
ExecStart=/usr/bin/node /opt/myapp/server.js
ExecStart=/usr/bin/python3 /opt/myapp/app.py
ExecStart=/bin/bash /opt/scripts/monitor.sh
```

**Restart — политика перезапуска:**
- `no` — не перезапускать
- `always` — всегда перезапускать
- `on-failure` — перезапускать только при ошибке
- `on-abnormal` — перезапускать при аномальном завершении

```ini
Restart=always
RestartSec=10    # Ждать 10 секунд перед перезапуском
```

**StandardOutput/StandardError:**
```ini
StandardOutput=journal    # Логи в journalctl
StandardError=journal
```

---

### Секция [Install]

```ini
[Install]
WantedBy=multi-user.target
```

**WantedBy:**
- `multi-user.target` — обычный режим (без GUI) — используй почти всегда
- `graphical.target` — графический режим

---

### Команды для работы с сервисами

```bash
# Создать/изменить файл сервиса
sudo nano /etc/systemd/system/myapp.service

# Перечитать конфигурацию (ОБЯЗАТЕЛЬНО после изменения файла!)
sudo systemctl daemon-reload

# Запустить сервис
sudo systemctl start myapp

# Остановить сервис
sudo systemctl stop myapp

# Перезапустить сервис
sudo systemctl restart myapp

# Включить автозапуск при загрузке
sudo systemctl enable myapp

# Отключить автозапуск
sudo systemctl disable myapp

# Проверить статус
sudo systemctl status myapp

# Посмотреть логи
sudo journalctl -u myapp -f
```

---

### Примеры сервисов

**Node.js приложение:**
```ini
[Unit]
Description=Node.js API Server
After=network.target

[Service]
Type=simple
User=nodeapp
WorkingDirectory=/opt/api
Environment="NODE_ENV=production"
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=10
StandardOutput=journal

[Install]
WantedBy=multi-user.target
```

**Bash скрипт с while true:**
```ini
[Unit]
Description=Monitor Service
After=network.target

[Service]
Type=simple
ExecStart=/opt/scripts/monitor.sh
Restart=always
StandardOutput=journal

[Install]
WantedBy=multi-user.target
```

**Скрипт monitor.sh:**
```bash
#!/bin/bash
while true; do
    echo "Monitoring..."
    sleep 30
done
```

---

## Урок 10: Systemd Timers (аналог cron)

### Что такое timer

Timer = планировщик задач в systemd (аналог cron, но лучше).

**Преимущества над cron:**
- ✅ Логи через journalctl (автоматически)
- ✅ Зависимости (запускать после другого сервиса)
- ✅ Мониторинг через `systemctl list-timers`
- ✅ Persistent (запуск пропущенных задач)
- ✅ Рандомизация времени запуска

---

### Структура timer

Timer состоит из **двух файлов:**
1. **Service файл** (`.service`) — ЧТО запускать
2. **Timer файл** (`.timer`) — КОГДА запускать

**Имена должны совпадать:** `backup.service` + `backup.timer`

---

### Секция [Timer]

```ini
[Timer]
OnCalendar=daily
OnUnitActiveSec=10min
OnBootSec=5min
Persistent=true
AccuracySec=1min
RandomizedDelaySec=5min

[Install]
WantedBy=timers.target
```

**OnCalendar — запуск по календарю (как cron):**
```ini
OnCalendar=daily         # Каждый день в 00:00
OnCalendar=hourly        # Каждый час
OnCalendar=weekly        # Каждую неделю (понедельник 00:00)
OnCalendar=02:00         # Каждый день в 02:00
OnCalendar=15:30         # Каждый день в 15:30
OnCalendar=*:00          # Каждый час (в начале часа)
OnCalendar=*:0/15        # Каждые 15 минут (00, 15, 30, 45)
OnCalendar=*:0/5         # Каждые 5 минут
OnCalendar=Mon 09:00     # Каждый понедельник в 09:00
OnCalendar=Fri 17:00     # Каждую пятницу в 17:00
OnCalendar=Mon,Wed,Fri 12:00  # Пн, Ср, Пт в 12:00
OnCalendar=Mon..Fri 09:00     # Будни в 09:00
```

**OnUnitActiveSec — интервал после завершения:**
```ini
OnUnitActiveSec=30s      # Каждые 30 секунд после завершения
OnUnitActiveSec=5min     # Каждые 5 минут
OnUnitActiveSec=1h       # Каждый час
OnUnitActiveSec=1d       # Каждый день
```

**OnBootSec — запуск после загрузки:**
```ini
OnBootSec=5min           # Через 5 минут после загрузки
OnBootSec=1h             # Через 1 час после загрузки
```

**Persistent — запуск пропущенных задач:**
```ini
Persistent=true
```
Если сервер был выключен во время запланированного запуска, задача выполнится сразу после включения.

**AccuracySec — точность времени:**
```ini
AccuracySec=1min         # По умолчанию (задача может запуститься ±1 минута)
AccuracySec=1s           # Высокая точность
AccuracySec=1h           # Низкая точность (экономит ресурсы)
```

**RandomizedDelaySec — случайная задержка:**
```ini
RandomizedDelaySec=5min  # Задержка до 5 минут
```
Полезно, когда много серверов запускают задачи одновременно.

---

### Единицы времени

- `s` — секунды
- `min` — минуты
- `h` — часы
- `d` — дни
- `w` — недели
- `M` — месяцы
- `y` — годы

---

### Примеры timers

**Ежедневный бэкап в 03:00:**

**backup.service:**
```ini
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/opt/scripts/backup.sh
StandardOutput=journal
```

**backup.timer:**
```ini
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=03:00
Persistent=true

[Install]
WantedBy=timers.target
```

**Проверка здоровья каждые 10 минут:**

**healthcheck.service:**
```ini
[Unit]
Description=Health Check

[Service]
Type=oneshot
ExecStart=/root/health_check.sh
StandardOutput=journal
```

**healthcheck.timer:**
```ini
[Unit]
Description=Health Check Timer

[Timer]
OnUnitActiveSec=10min
OnBootSec=1min
Persistent=true

[Install]
WantedBy=timers.target
```

---

### Команды для работы с timers

```bash
# Включить timer (автозапуск)
sudo systemctl enable backup.timer

# Запустить timer
sudo systemctl start backup.timer

# Остановить timer
sudo systemctl stop backup.timer

# Проверить статус timer
sudo systemctl status backup.timer

# Список всех timers
systemctl list-timers

# Список всех timers (включая неактивные)
systemctl list-timers --all

# Посмотреть логи сервиса
journalctl -u backup.service

# Запустить сервис вручную (без ожидания timer)
sudo systemctl start backup.service

# Проверить выражение OnCalendar
systemd-analyze calendar "Mon 09:00"
systemd-analyze calendar "*:0/15"
```

---

### ВАЖНО: Несколько OnCalendar создают несколько расписаний

```ini
# НЕПРАВИЛЬНО (если хочешь только 03:00)
OnCalendar=daily         # 00:00
OnCalendar=03:00         # 03:00
# Результат: запустится в 00:00 И в 03:00

# ПРАВИЛЬНО
OnCalendar=03:00         # Только 03:00
```

---

## Урок 11: Логи и debugging (journalctl)

### Базовые команды

```bash
# Все логи
journalctl

# Логи конкретного сервиса
journalctl -u nginx

# Логи нескольких сервисов
journalctl -u nginx -u mysql

# Последние N строк
journalctl -n 50
journalctl -u nginx -n 20

# В реальном времени (как tail -f)
journalctl -f
journalctl -u nginx -f

# С определённого времени
journalctl --since "2026-04-29 10:00:00"
journalctl --since "1 hour ago"
journalctl --since "yesterday"
journalctl --since "today"

# До определённого времени
journalctl --until "2026-04-29 12:00:00"
journalctl --until "10 minutes ago"

# Диапазон времени
journalctl --since "2026-04-29 10:00" --until "2026-04-29 12:00"

# Логи за сегодня
journalctl --since today

# Логи за вчера
journalctl --since yesterday --until today
```

---

### Фильтрация по приоритету

**Уровни приоритета:**
- `0` — emerg (система неработоспособна)
- `1` — alert (требуется немедленное действие)
- `2` — crit (критическая ошибка)
- `3` — err (ошибка)
- `4` — warning (предупреждение)
- `5` — notice (важное уведомление)
- `6` — info (информация)
- `7` — debug (отладка)

```bash
# Только ошибки и критичнее (0-3)
journalctl -p err

# Только предупреждения и критичнее (0-4)
journalctl -p warning

# Конкретный уровень
journalctl -p 3

# Диапазон уровней
journalctl -p err..crit

# Ошибки конкретного сервиса
journalctl -u nginx -p err

# Ошибки за последний час
journalctl -p err --since "1 hour ago"
```

---

### Форматы вывода

```bash
# Короткий формат (по умолчанию)
journalctl -u nginx

# Подробный формат (с метаданными)
journalctl -u nginx -o verbose

# JSON формат
journalctl -u nginx -o json

# JSON pretty (читаемый)
journalctl -u nginx -o json-pretty

# Только сообщения (без timestamp)
journalctl -u nginx -o cat

# ISO формат времени (читаемый)
journalctl -u nginx -o short-iso
```

---

### Поиск по логам

```bash
# Grep по логам
journalctl -u nginx | grep "error"

# Grep с контекстом (3 строки до и после)
journalctl -u nginx | grep -C 3 "error"

# Поиск через journalctl (быстрее)
journalctl -u nginx -g "error"

# Case-insensitive поиск
journalctl -u nginx -g "(?i)error"
```

---

### Управление размером логов

```bash
# Показать использование диска
journalctl --disk-usage

# Очистить логи старше N дней
sudo journalctl --vacuum-time=7d

# Оставить только N GB логов
sudo journalctl --vacuum-size=1G

# Оставить только N файлов
sudo journalctl --vacuum-files=10

# Проверить целостность логов
journalctl --verify
```

---

### Экспорт логов

```bash
# Экспорт в файл
journalctl -u nginx > nginx.log

# Экспорт за период
journalctl --since "2026-04-29" --until "2026-04-30" > logs-29-04.txt

# Экспорт в JSON
journalctl -u nginx -o json > nginx.json

# Экспорт только ошибок
journalctl -u nginx -p err > nginx-errors.log
```

---

### Debugging сервисов

**Сервис не запускается:**
```bash
systemctl status myapp
journalctl -u myapp -n 50
journalctl -u myapp --since "5 minutes ago"
journalctl -u myapp -p err
```

**Сервис падает:**
```bash
journalctl -u myapp -f
sudo systemctl restart myapp &
journalctl -u myapp -f
```

**Высокая нагрузка:**
```bash
journalctl --since "10 minutes ago"
journalctl -p err --since "10 minutes ago"
```

---

### Полезные комбинации

```bash
# Последние 100 строк всех ошибок
journalctl -p err -n 100

# Ошибки nginx за последний час
journalctl -u nginx -p err --since "1 hour ago"

# Все логи сервиса с момента последней загрузки
journalctl -u nginx -b

# Логи предыдущей загрузки
journalctl -b -1

# Список всех загрузок
journalctl --list-boots

# Следить за несколькими сервисами
journalctl -u nginx -u mysql -f

# Без пейджера (для скриптов)
journalctl -u nginx --no-pager
```

---

## Урок 12: Зависимости и порядок запуска

### Типы зависимостей

**1. After / Before — порядок запуска (мягкая зависимость):**
```ini
[Unit]
After=network.target
```
- Запустить **после** network.target
- Если network.target не запустится — наш сервис всё равно запустится
- Это только **порядок**, не зависимость

```ini
[Unit]
Before=nginx.service
```
- Запустить **до** nginx.service

**2. Requires — жёсткая зависимость:**
```ini
[Unit]
Requires=postgresql.service
```
- Если PostgreSQL не запустится — наш сервис **не запустится**
- Если PostgreSQL упадёт — наш сервис **тоже упадёт**

**3. Wants — мягкая зависимость:**
```ini
[Unit]
Wants=redis.service
```
- Попытаться запустить Redis
- Если Redis не запустится — наш сервис **всё равно запустится**
- Если Redis упадёт — наш сервис **продолжит работать**

**4. BindsTo — очень жёсткая зависимость:**
```ini
[Unit]
BindsTo=postgresql.service
```
- Если PostgreSQL остановится — наш сервис **немедленно остановится**
- Ещё жёстче, чем Requires

**5. PartOf — часть другого сервиса:**
```ini
[Unit]
PartOf=docker.service
```
- Если Docker остановится — наш сервис остановится
- Но если наш сервис остановится — Docker продолжит работать
- Односторонняя зависимость

---

### Таблица зависимостей

| Директива | Что делает | Когда использовать |
|-----------|------------|-------------------|
| `After=` | Запустить после | Всегда (определяет порядок) |
| `Before=` | Запустить до | Редко (обратный порядок) |
| `Requires=` | Жёсткая зависимость | Критичные зависимости (БД) |
| `Wants=` | Мягкая зависимость | Некритичные зависимости (кеш) |
| `BindsTo=` | Очень жёсткая | Контейнеры, связанные процессы |
| `PartOf=` | Часть другого сервиса | Дочерние сервисы |

---

### Примеры зависимостей

**Приложение зависит от PostgreSQL (жёсткая):**
```ini
[Unit]
Description=My Application
After=network.target postgresql.service
Requires=postgresql.service

[Service]
Type=simple
ExecStart=/opt/myapp/app
Restart=always

[Install]
WantedBy=multi-user.target
```

**Приложение хочет Redis, но не критично (мягкая):**
```ini
[Unit]
Description=My Application
After=network.target redis.service
Wants=redis.service

[Service]
Type=simple
ExecStart=/opt/myapp/app
Restart=always

[Install]
WantedBy=multi-user.target
```

**Воркер зависит от PostgreSQL И Redis:**
```ini
[Unit]
Description=Celery Worker
After=network.target postgresql.service redis.service
Requires=postgresql.service redis.service

[Service]
Type=simple
ExecStart=/usr/bin/celery -A tasks worker
Restart=always

[Install]
WantedBy=multi-user.target
```

---

### Targets (группы сервисов)

**Основные targets:**
- `network.target` — сеть готова
- `multi-user.target` — многопользовательский режим (без GUI)
- `graphical.target` — графический режим
- `basic.target` — базовая система
- `sysinit.target` — инициализация системы

**Порядок запуска при загрузке:**
1. `sysinit.target` — инициализация системы
2. `basic.target` — базовые сервисы
3. `network.target` — сеть
4. `multi-user.target` — пользовательские сервисы
5. `graphical.target` — GUI

---

### Проверка зависимостей

```bash
# Показать зависимости сервиса
systemctl list-dependencies nginx

# Показать обратные зависимости (кто зависит от этого сервиса)
systemctl list-dependencies --reverse nginx

# Показать все зависимости (рекурсивно)
systemctl list-dependencies --all nginx

# Показать только неудачные сервисы
systemctl --failed
```

---

### ВАЖНО: Зависимости работают в обратную сторону!

**Неправильно:**
```bash
sudo systemctl start database
# Запустится только database, api и frontend НЕ запустятся
```

**Правильно:**
```bash
sudo systemctl start frontend
# Запустятся: frontend → api → database (вся цепочка)
```

**Правило:** Запускай **верхний уровень**, зависимости запустятся автоматически.

---

## Типичные ошибки (не повторяй!)

1. ❌ `type` вместо `Type` (чувствительность к регистру)
2. ❌ `Type=oneshot` с `Restart=always` (несовместимо!)
3. ❌ `multy-user.target` вместо `multi-user.target`
4. ❌ Забыть `sudo systemctl daemon-reload` после изменения файла
5. ❌ `OnUnitActiveSec` в service файле (это для timer!)
6. ❌ `[ "$NUM" > 0 ]` вместо `[ "$NUM" -gt 0 ]` (> это редирект!)
7. ❌ Несколько `OnCalendar` создают несколько расписаний
8. ❌ Опечатки: `Requiers` вместо `Requires`
9. ❌ Запускать database вместо frontend (зависимости в обратную сторону)
10. ❌ Забыть `curl -L` для редиректов

---

## Полезные команды для экзамена

```bash
# Создать/изменить сервис
sudo nano /etc/systemd/system/myapp.service

# Перечитать конфигурацию
sudo systemctl daemon-reload

# Запустить/остановить/перезапустить
sudo systemctl start myapp
sudo systemctl stop myapp
sudo systemctl restart myapp

# Включить/отключить автозапуск
sudo systemctl enable myapp
sudo systemctl disable myapp

# Статус сервиса
systemctl status myapp

# Логи сервиса
journalctl -u myapp -f
journalctl -u myapp -n 50
journalctl -u myapp --since "1 hour ago"
journalctl -u myapp -p err

# Список timers
systemctl list-timers

# Зависимости
systemctl list-dependencies myapp

# Использование диска логами
journalctl --disk-usage

# Очистка логов
sudo journalctl --vacuum-time=7d
```

---

**Повтори эту шпаргалку перед экзаменом!**
