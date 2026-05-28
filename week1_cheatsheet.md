# Шпаргалка: Неделя 1 — Linux Basics

## Урок 1: Мониторинг процессов

### Команды для мониторинга
```bash
# Интерактивный мониторинг процессов
top                    # Базовый мониторинг
htop                   # Улучшенная версия (если установлен)

# Список всех процессов
ps aux                 # Все процессы с деталями

# Сортировка процессов
ps aux --sort=-%cpu    # По CPU (descending)
ps aux --sort=-%mem    # По памяти (descending)

# Топ-N процессов
ps aux --sort=-%cpu | head -10    # Топ-10 по CPU
ps aux --sort=-%mem | head -5     # Топ-5 по памяти
```

### Флаги ps aux
- `a` — показать процессы всех пользователей
- `u` — показать владельца процесса
- `x` — показать процессы без терминала

### Колонки ps aux
- `$1` — USER (пользователь)
- `$2` — PID (process ID)
- `$3` — %CPU (использование CPU)
- `$4` — %MEM (использование памяти)
- `$11` — COMMAND (имя процесса)

### Права на выполнение
```bash
chmod +x script.sh     # Добавить право на выполнение
chmod -x script.sh     # Убрать право на выполнение
./script.sh            # Запустить скрипт
```

---

## Урок 2: Networking и порты

### Проверка портов
```bash
# Показать все слушающие порты
ss -tuln               # Без процессов
ss -tulnp              # С процессами (нужен root)

# Проверить конкретный порт
ss -tulnp | grep :8080

# Проверить TCP соединение
nc -zv localhost 8080  # -z (scan), -v (verbose)
```

### Флаги ss
- `-t` — TCP
- `-u` — UDP
- `-l` — listening (слушающие)
- `-n` — numeric (не резолвить имена)
- `-p` — process (показать процесс)

### Извлечение данных
```bash
# Последняя колонка (процесс)
awk '{print $NF}'

# Извлечь PID из строки users:(("nginx",pid=1234,fd=6))
grep -oP 'pid=\K[0-9]+'

# Извлечь имя процесса
grep -oP '"\K[^"]+' | head -1

# Взять только первую строку
head -1
```

---

## Урок 3: systemd и сервисы

### Управление сервисами
```bash
# Статус сервиса
systemctl status nginx
systemctl is-active nginx    # Только статус (active/inactive)

# Запуск/остановка
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx       # Перечитать конфиг без перезапуска

# Автозапуск
systemctl enable nginx       # Включить автозапуск
systemctl disable nginx      # Отключить автозапуск

# Информация о сервисе
systemctl show nginx -p ActiveEnterTimestamp    # Время запуска
```

### Логи через journalctl
```bash
# Логи сервиса
journalctl -u nginx

# Последние N строк
journalctl -u nginx -n 20

# В реальном времени (как tail -f)
journalctl -u nginx -f

# Без пейджера (для скриптов)
journalctl -u nginx --no-pager
```

### Состояния сервиса
- `active (running)` — работает
- `inactive (dead)` — остановлен
- `failed` — упал с ошибкой
- `activating` — запускается

---

## Урок 4: Bash scripting

### Переменные
```bash
# Объявление
VAR="value"
PORT=$1                # Первый аргумент скрипта

# Использование
echo "$VAR"
echo "Port: $PORT"

# Результат команды в переменную
RESULT=$(ls -la)
STATUS=$(systemctl is-active nginx)
```

### Условия
```bash
# Проверка пустой переменной
if [ -z "$VAR" ]; then
    echo "Empty"
fi

# Сравнение строк
if [ "$STATUS" = "active" ]; then
    echo "Running"
fi

if [ "$STATUS" != "active" ]; then
    echo "Not running"
fi

# Сравнение чисел
if [ "$NUM" -gt 80 ]; then    # greater than
    echo "More than 80"
fi

if [ "$NUM" -lt 50 ]; then    # less than
    echo "Less than 50"
fi

if [ "$NUM" -eq 100 ]; then   # equal
    echo "Exactly 100"
fi
```

### Массивы
```bash
# Объявление
services=("nginx" "mysql" "redis")

# Цикл по массиву
for svc in "${services[@]}"; do
    echo "Checking $svc"
done
```

### Циклы
```bash
# For с диапазоном
for i in {1..5}; do
    echo "Iteration $i"
done

# For по файлам
for file in /var/log/*.log; do
    echo "Processing $file"
done

# While
while true; do
    echo "Running..."
    sleep 5
done
```

### Проверка диска и памяти
```bash
# Процент использования диска (корневой раздел)
df -h / | awk 'NR==2 {print $5}' | sed 's/%//'

# Процент использования памяти
free | awk 'NR==2 {printf "%.0f", $3/$2*100}'
```

### Логирование
```bash
# Вывод в консоль И в файл
echo "Message" | tee -a /var/log/app.log

# Только в файл (append)
echo "Message" >> /var/log/app.log

# Timestamp
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
echo "[$TIMESTAMP] Message" >> /var/log/app.log
```

### Перенаправление вывода
```bash
command 2>/dev/null        # Скрыть stderr (ошибки)
command > /dev/null        # Скрыть stdout
command &> /dev/null       # Скрыть всё
```

---

## Важные концепции

### Pipe (|)
Передаёт вывод одной команды на вход другой:
```bash
ps aux | grep nginx | awk '{print $2}'
```

### Подстановка команд $()
Выполняет команду и подставляет результат:
```bash
STATUS=$(systemctl is-active nginx)
echo "Status: $STATUS"
```

### Пробелы в условиях
```bash
# ПРАВИЛЬНО
if [ -z "$VAR" ]; then

# НЕПРАВИЛЬНО
if [-z "$VAR"]; then      # Нет пробелов
if [ -z "$VAR"]; then     # Нет пробела перед ]
```

### Exit codes
```bash
exit 0    # Успех
exit 1    # Ошибка

# Проверка exit code предыдущей команды
if [ $? -eq 0 ]; then
    echo "Success"
fi
```

---

## Полезные команды для экзамена

```bash
# Найти процесс по имени
ps aux | grep nginx

# Убить процесс по PID
kill -9 1234

# Проверить, слушается ли порт
ss -tulnp | grep :8080

# Проверить статус сервиса
systemctl is-active nginx

# Последние логи сервиса
journalctl -u nginx -n 10

# Использование диска
df -h

# Использование памяти
free -h

# Текущая дата
date '+%Y-%m-%d %H:%M:%S'
```

---

## Типичные ошибки (не повторяй!)

1. ❌ `chmod -x` вместо `chmod +x`
2. ❌ `echo "systemctl..."` вместо `$(systemctl...)`
3. ❌ `if [$VAR]` вместо `if [ "$VAR" ]` (нет пробелов)
4. ❌ Забыть `head -1` при множественных результатах
5. ❌ Опечатки в именах переменных ($PROCESS_ONFO вместо $PROCESS_INFO)
6. ❌ Захардкодить значение вместо использования переменной
7. ❌ Лишний пробел в regex: `'"\K [^"]+`' вместо `'"\K[^"]+`'

---

## Созданные скрипты

1. **monitor.sh** — топ процессов по CPU/Memory
2. **check_ports.sh** — проверка портов и процессов
3. **check_service.sh** — статус сервиса, uptime, логи
4. **health_check.sh** — комплексная проверка (сервисы, диск, память, логирование)

---

**Повтори эту шпаргалку перед экзаменом!**
