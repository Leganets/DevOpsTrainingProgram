# Шпаргалка: Неделя 4 — Bash scripting углублённо

## Урок 13: Функции в Bash

### Синтаксис функции

**Вариант 1 (классический):**
```bash
function my_function {
    echo "Hello"
}
```

**Вариант 2 (современный, рекомендуется):**
```bash
my_function() {
    echo "Hello"
}
```

**Вызов функции:**
```bash
my_function    # Просто имя, без ()
```

---

### Функции с аргументами

```bash
greet() {
    local name=$1    # Первый аргумент
    echo "Hello, $name!"
}

greet "Alice"        # Вывод: Hello, Alice!
```

**Важно:** `$1` внутри функции — это аргумент **функции**, а не скрипта!

---

### Локальные переменные (local)

```bash
my_function() {
    local var="inside"    # Локальная переменная
    echo "$var"
}

var="outside"
my_function              # Вывод: inside
echo "$var"              # Вывод: outside
```

**Правило:** Всегда используй `local` для переменных внутри функций.

---

### Возврат значений

**Вариант 1: через echo (для данных)**
```bash
get_cpu_usage() {
    local cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
    echo "$cpu"    # Вернуть значение
}

CPU=$(get_cpu_usage)
echo "CPU: $CPU%"
```

**Вариант 2: через return (только exit code 0-255)**
```bash
check_port() {
    local port=$1
    if ss -tuln | grep -q ":$port "; then
        return 0    # Успех
    else
        return 1    # Ошибка
    fi
}

if check_port 80; then
    echo "Port 80 is open"
fi
```

**Важно:** `return` возвращает **exit code**, а не данные! Для данных используй `echo`.

---

### Проверка количества аргументов

```bash
check_service() {
    if [ $# -eq 0 ]; then
        echo "Error: No service name provided"
        return 1
    fi
    
    local service=$1
    systemctl is-active "$service"
}
```

`$#` — количество аргументов функции (или скрипта).

---

### Порядок определения функций

**Важно:** Функция должна быть **определена ДО вызова**!

**❌ Неправильно:**
```bash
my_function    # Ошибка: function not found

my_function() {
    echo "Hello"
}
```

**✅ Правильно:**
```bash
my_function() {
    echo "Hello"
}

my_function    # Работает
```

---

## Урок 14: Обработка ошибок в Bash

### set -e (exit on error)

```bash
#!/bin/bash
set -e    # Остановить скрипт при любой ошибке

cd /nonexistent/directory    # Ошибка!
rm -rf *                     # НЕ выполнится
```

**Что делает:** Останавливает скрипт при **любой ошибке** (exit code != 0).

---

### set -u (exit on undefined variable)

```bash
#!/bin/bash
set -u    # Остановить при использовании неопределённой переменной

echo "Hello, $USRE"    # Ошибка: USRE: unbound variable
```

---

### set -o pipefail (ошибки в pipe)

```bash
#!/bin/bash
set -e
set -o pipefail    # Проверять exit code ВСЕХ команд в pipe

cat /nonexistent/file | grep "error"    # Скрипт остановится
```

**Без pipefail:** bash проверяет exit code **последней команды** в pipe.  
**С pipefail:** bash проверяет exit code **всех команд** в pipe.

---

### Комбинация: set -euo pipefail

**Рекомендуемый шаблон для всех скриптов:**
```bash
#!/bin/bash
set -euo pipefail    # Строгий режим
```

**Что это даёт:**
- `set -e` — остановка при ошибке
- `set -u` — остановка при неопределённой переменной
- `set -o pipefail` — остановка при ошибке в pipe

---

### Exit codes (коды возврата)

```bash
systemctl is-active nginx
echo $?    # Выведет exit code предыдущей команды

# 0 — успех
# 1-255 — ошибка
```

**Использование в условиях:**
```bash
if systemctl is-active nginx > /dev/null 2>&1; then
    echo "Nginx is running"
else
    echo "Nginx is NOT running"
fi
```

---

### Ручная обработка ошибок

**Вариант 1: через `||` (OR)**
```bash
cp /source/file /destination/ || {
    echo "Error: Failed to copy file"
    exit 1
}
```

**Вариант 2: через `if !`**
```bash
if ! cp /source/file /destination/; then
    echo "Error: Failed to copy file"
    exit 1
fi
```

---

### trap — обработка сигналов и выхода

**Пример 1: cleanup при выходе**
```bash
#!/bin/bash
set -e

cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/myapp.lock
}

trap cleanup EXIT    # Выполнить cleanup при выходе

touch /tmp/myapp.lock
echo "Working..."
sleep 5

# cleanup() выполнится автоматически
```

**Пример 2: обработка Ctrl+C**
```bash
#!/bin/bash

handle_interrupt() {
    echo ""
    echo "Script interrupted! Cleaning up..."
    exit 1
}

trap handle_interrupt INT    # Обработать Ctrl+C

while true; do
    echo "Working... (Press Ctrl+C to stop)"
    sleep 2
done
```

---

### /dev/null и перенаправление потоков

**Потоки:**
- **stdin (0)** — стандартный ввод
- **stdout (1)** — стандартный вывод
- **stderr (2)** — стандартный вывод ошибок

**`/dev/null`** — "чёрная дыра", всё что туда отправляется — исчезает.

---

### Перенаправление потоков

**1. `> /dev/null` — скрыть только stdout**
```bash
ls /existing_dir > /dev/null    # Скрыть список файлов
ls /nonexistent > /dev/null     # Ошибка ВСЁ РАВНО выведется!
```

**2. `2> /dev/null` — скрыть только stderr**
```bash
ls /nonexistent 2> /dev/null    # Ошибка скрыта
```

**3. `> /dev/null 2>&1` — скрыть ВСЁ (stdout + stderr)**
```bash
ls /nonexistent > /dev/null 2>&1    # Ничего не выведется
```

**Что происходит:**
1. `> /dev/null` — перенаправить **stdout (1)** в `/dev/null`
2. `2>&1` — перенаправить **stderr (2)** туда же, куда идёт **stdout (1)**

**Порядок важен!**
```bash
# ✅ ПРАВИЛЬНО
command > /dev/null 2>&1    # Сначала stdout, потом stderr

# ❌ НЕПРАВИЛЬНО
command 2>&1 > /dev/null    # stderr пойдёт на экран
```

**4. `&> /dev/null` — скрыть ВСЁ (короткая запись)**
```bash
ls /nonexistent &> /dev/null    # Ничего не выведется
```

**Это то же самое, что `> /dev/null 2>&1`, но короче!**

---

### Сравнение перенаправлений

| Команда | Что скрывает |
|---------|-------------|
| `> /dev/null` | Только stdout (1) |
| `2> /dev/null` | Только stderr (2) |
| `> /dev/null 2>&1` | Всё (stdout + stderr) |
| `&> /dev/null` | Всё (stdout + stderr) |

---

### Когда использовать что?

**Скрыть только ошибки:**
```bash
grep "pattern" file.txt 2> /dev/null
```

**Скрыть только вывод:**
```bash
ping -c 1 google.com > /dev/null
```

**Скрыть ВСЁ:**
```bash
systemctl is-active nginx &> /dev/null
# или
systemctl is-active nginx > /dev/null 2>&1
```

---

## Урок 15: Аргументы и опции (getopts)

### Позиционные аргументы

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
```

---

### Специальные переменные

| Переменная | Описание |
|-----------|----------|
| `$0` | Имя скрипта |
| `$1, $2, $3...` | Позиционные аргументы |
| `$@` | Все аргументы (как массив) |
| `$*` | Все аргументы (как одна строка) |
| `$#` | Количество аргументов |
| `$?` | Exit code последней команды |
| `$$` | PID текущего процесса |

---

### getopts — парсинг опций

**Синтаксис:**
```bash
while getopts "опции" переменная; do
    case $переменная in
        опция1) действие ;;
        опция2) действие ;;
    esac
done
```

---

### Пример 1: Простые флаги (без значений)

```bash
#!/bin/bash

verbose=false
debug=false

while getopts "vd" opt; do
    case $opt in
        v) verbose=true ;;
        d) debug=true ;;
        \?) echo "Invalid option: -$OPTARG" >&2; exit 1 ;;
    esac
done

echo "Verbose: $verbose"
echo "Debug: $debug"
```

**Запуск:**
```bash
./script.sh -v        # Verbose: true
./script.sh -vd       # Verbose: true, Debug: true
```

---

### Пример 2: Опции со значениями

**Двоеточие после буквы** означает, что опция **требует значение**:

```bash
#!/bin/bash

file=""
count=10

while getopts "f:c:" opt; do
    case $opt in
        f) file=$OPTARG ;;      # -f требует значение
        c) count=$OPTARG ;;     # -c требует значение
        \?) echo "Invalid option: -$OPTARG" >&2; exit 1 ;;
        :) echo "Option -$OPTARG requires an argument" >&2; exit 1 ;;
    esac
done

echo "File: $file"
echo "Count: $count"
```

**Запуск:**
```bash
./script.sh -f data.txt -c 5
# File: data.txt
# Count: 5
```

---

### Синтаксис getopts

```bash
while getopts "abc:d:e" opt; do
```

**Расшифровка:**
- `a` — флаг без значения (`-a`)
- `b` — флаг без значения (`-b`)
- `c:` — опция со значением (`-c value`)
- `d:` — опция со значением (`-d value`)
- `e` — флаг без значения (`-e`)

---

### Обработка ошибок в getopts

```bash
while getopts "f:c:" opt; do
    case $opt in
        f) file=$OPTARG ;;
        c) count=$OPTARG ;;
        \?) 
            echo "Invalid option: -$OPTARG" >&2
            exit 1
            ;;
        :) 
            echo "Option -$OPTARG requires an argument" >&2
            exit 1
            ;;
    esac
done
```

**Специальные case:**
- `\?)` — неизвестная опция
- `:)` — опция без обязательного значения

---

### Получение оставшихся аргументов

```bash
#!/bin/bash

verbose=false

while getopts "v" opt; do
    case $opt in
        v) verbose=true ;;
    esac
done

shift $((OPTIND - 1))    # Убрать обработанные опции

echo "Verbose: $verbose"
echo "Remaining arguments: $@"
```

**Запуск:**
```bash
./script.sh -v file1.txt file2.txt
# Verbose: true
# Remaining arguments: file1.txt file2.txt
```

---

### Функция usage (help)

```bash
usage() {
    echo "Usage: $0 [OPTIONS]"
    echo ""
    echo "Options:"
    echo "  -f FILE     Input file (required)"
    echo "  -o FILE     Output file (optional)"
    echo "  -v          Verbose mode"
    echo "  -h          Show this help"
    echo ""
    echo "Examples:"
    echo "  $0 -f input.txt"
    echo "  $0 -f input.txt -o result.txt -v"
    exit 1
}
```

---

### Проверка обязательных параметров

```bash
if [ -z "$service" ] || [ -z "$action" ]; then
    echo "Error: -s and -a are required"
    usage
fi
```

---

### Валидация значений

```bash
case $action in
    start|stop|restart|status)
        # Валидный action
        ;;
    *)
        echo "Error: Invalid action '$action'"
        exit 1
        ;;
esac
```

---

## Урок 16: Массивы и ассоциативные массивы

### Создание массивов

**Способ 1: через скобки**
```bash
services=("nginx" "mysql" "redis")
```

**Способ 2: по элементам**
```bash
services[0]="nginx"
services[1]="mysql"
services[2]="redis"
```

**Способ 3: пустой массив**
```bash
services=()
```

---

### Доступ к элементам

```bash
services=("nginx" "mysql" "redis")

echo "${services[0]}"    # nginx (первый элемент)
echo "${services[1]}"    # mysql (второй элемент)
echo "${services[@]}"    # nginx mysql redis (все элементы)
echo "${#services[@]}"   # 3 (количество элементов)
```

**Важно:** Используй `${services[@]}`, а не `$services`.

---

### Добавление элементов

```bash
services=("nginx" "mysql")

# Добавить в конец
services+=("redis")

echo "${services[@]}"    # nginx mysql redis
```

---

### Цикл по массиву

```bash
services=("nginx" "mysql" "redis")

for service in "${services[@]}"; do
    echo "Checking $service..."
    systemctl status "$service"
done
```

---

### Индексы массива

```bash
services=("nginx" "mysql" "redis")

# Получить все индексы
echo "${!services[@]}"    # 0 1 2

# Цикл с индексами
for i in "${!services[@]}"; do
    echo "Index $i: ${services[$i]}"
done
```

---

### Удаление элементов

```bash
services=("nginx" "mysql" "redis")

# Удалить элемент по индексу
unset services[1]

echo "${services[@]}"    # nginx redis
```

**Важно:** После `unset` индексы НЕ пересчитываются!

---

### Срезы массива

```bash
services=("nginx" "mysql" "redis" "postgresql" "mongodb")

# Получить элементы с индекса 1, количество 3
echo "${services[@]:1:3}"    # mysql redis postgresql

# Получить все элементы начиная с индекса 2
echo "${services[@]:2}"      # redis postgresql mongodb
```

---

## Ассоциативные массивы (hash maps)

**Ассоциативный массив** — массив с **текстовыми ключами**.

---

### Создание ассоциативного массива

**Важно:** Нужно объявить через `declare -A`!

```bash
declare -A ports

ports[nginx]=80
ports[mysql]=3306
ports[redis]=6379
```

**Или сразу при создании:**
```bash
declare -A ports=(
    [nginx]=80
    [mysql]=3306
    [redis]=6379
)
```

---

### Доступ к элементам

```bash
declare -A ports=(
    [nginx]=80
    [mysql]=3306
    [redis]=6379
)

echo "${ports[nginx]}"     # 80
echo "${ports[@]}"         # 80 3306 6379 (все значения)
echo "${!ports[@]}"        # nginx mysql redis (все ключи)
echo "${#ports[@]}"        # 3 (количество элементов)
```

---

### Цикл по ассоциативному массиву

```bash
declare -A ports=(
    [nginx]=80
    [mysql]=3306
    [redis]=6379
)

# Цикл по ключам
for service in "${!ports[@]}"; do
    echo "$service runs on port ${ports[$service]}"
done
```

---

### Проверка существования ключа

```bash
declare -A ports=(
    [nginx]=80
    [mysql]=3306
)

# Проверить, существует ли ключ
if [[ -v ports[nginx] ]]; then
    echo "nginx port: ${ports[nginx]}"
fi
```

---

### Добавление и удаление элементов

```bash
declare -A ports=(
    [nginx]=80
    [mysql]=3306
)

# Добавить элемент
ports[redis]=6379

# Удалить элемент
unset ports[mysql]

echo "${!ports[@]}"    # nginx redis
```

---

### Разница между обычными и ассоциативными массивами

| Обычный массив | Ассоциативный массив |
|---------------|---------------------|
| `array=(val1 val2)` | `declare -A array=([key1]=val1)` |
| Индексы: числа (0, 1, 2...) | Ключи: строки (любые) |
| `${array[0]}` | `${array[key]}` |
| Не нужен `declare` | **Обязательно** `declare -A` |

---

### Передача массивов в функции

**Решение 1: передать элементы**
```bash
check_services() {
    local services=("$@")    # Получить все аргументы как массив
    
    for service in "${services[@]}"; do
        echo "Checking $service..."
    done
}

my_services=("nginx" "mysql" "redis")
check_services "${my_services[@]}"    # Передать элементы
```

**Решение 2: использовать глобальные массивы**
```bash
working=()
failed=()

generate_report() {
    # Функция использует глобальные массивы working и failed
    for service in "${working[@]}"; do
        echo "✓ $service"
    done
}
```

---

## Типичные ошибки (не повторяй!)

1. ❌ Забыть `local` для переменных в функциях
2. ❌ Использовать `return` для возврата данных (используй `echo`)
3. ❌ Забыть `set -euo pipefail` в начале скрипта
4. ❌ Использовать `2>&1 > /dev/null` вместо `> /dev/null 2>&1` (порядок важен!)
5. ❌ Использовать `$OPTARG` без двоеточия в getopts (например, `"f"` вместо `"f:"`)
6. ❌ Забыть проверку обязательных параметров после getopts
7. ❌ Использовать `$services` вместо `${services[@]}` для массивов
8. ❌ Забыть `declare -A` для ассоциативных массивов
9. ❌ Использовать `${service_ports[@]}` вместо `${service_ports[$service]}`
10. ❌ Забыть кавычки в `"${array[@]}"`

---

## Полезные команды для экзамена

```bash
# Функции
my_function() {
    local var=$1
    echo "$var"
}
result=$(my_function "value")

# Обработка ошибок
set -euo pipefail
trap cleanup EXIT

# Перенаправление
command &> /dev/null              # Скрыть всё
command > /dev/null 2>&1          # Скрыть всё (длинная запись)
command 2> /dev/null              # Скрыть только ошибки

# getopts
while getopts "f:v" opt; do
    case $opt in
        f) file=$OPTARG ;;
        v) verbose=true ;;
    esac
done

# Массивы
array=("val1" "val2")
array+=("val3")
for item in "${array[@]}"; do
    echo "$item"
done

# Ассоциативные массивы
declare -A ports=([nginx]=80 [mysql]=3306)
for service in "${!ports[@]}"; do
    echo "$service: ${ports[$service]}"
done
```

---

## Созданные скрипты

1. **system_monitor.sh** — мониторинг системы с функциями (CPU, Memory, Disk)
2. **check_services_safe.sh** — проверка сервисов с обработкой ошибок и trap
3. **service_manager.sh** — управление сервисами через getopts (-s, -a, -v)
4. **multi_service_monitor.sh** — мониторинг нескольких сервисов с ассоциативными массивами

---

**Повтори эту шпаргалку перед экзаменом!**
