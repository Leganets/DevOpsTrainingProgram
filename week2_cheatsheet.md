# Шпаргалка: Неделя 2 — Networking

## Урок 5: HTTP запросы (curl, wget)

### curl — основные команды

```bash
# Простой GET запрос
curl https://api.github.com

# Показать только HTTP headers
curl -I https://google.com

# Следовать редиректам (301, 302)
curl -L https://google.com

# Сохранить в файл
curl -o output.html https://example.com
curl -O https://example.com/file.zip    # Имя из URL

# POST запрос с JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","age":30}'

# Показать детали запроса (debugging)
curl -v https://api.github.com

# Только HTTP код ответа
curl -o /dev/null -s -w "%{http_code}\n" https://google.com

# HTTP код И время ответа (один запрос)
curl -o /dev/null -s -w "%{http_code}|%{time_total}" https://google.com

# Таймаут
curl --max-time 5 https://slow-site.com
```

### Флаги curl (важные)

- `-I` — только headers (HEAD запрос)
- `-L` — следовать редиректам
- `-o <file>` — сохранить в файл (указать имя)
- `-O` — сохранить с именем из URL
- `-X <method>` — HTTP метод (GET, POST, PUT, DELETE)
- `-H <header>` — добавить header
- `-d <data>` — данные для POST/PUT
- `-v` — verbose (показать детали запроса/ответа)
- `-s` — silent (без прогресс-бара)
- `-w <format>` — формат вывода
- `--max-time <sec>` — таймаут

### Форматы вывода curl (-w)

- `%{http_code}` — HTTP код ответа
- `%{time_total}` — общее время запроса
- `%{time_connect}` — время соединения
- `%{size_download}` — размер загрузки

### wget — основные команды

```bash
# Скачать файл
wget https://example.com/file.zip

# Скачать с другим именем
wget -O myfile.zip https://example.com/file.zip

# Продолжить прерванную загрузку
wget -c https://example.com/bigfile.iso

# Тихий режим
wget -q https://example.com/file.zip
```

---

## Урок 6: DNS (dig, nslookup)

### dig — основные команды

```bash
# Простой DNS lookup
dig google.com

# Только IP адрес (короткий вывод)
dig +short google.com

# Конкретный тип записи
dig google.com A        # IPv4 адрес
dig google.com AAAA     # IPv6 адрес
dig google.com MX       # Mail servers
dig google.com NS       # Name servers
dig google.com TXT      # Text records

# Использовать конкретный DNS сервер
dig @8.8.8.8 google.com         # Google DNS
dig @1.1.1.1 google.com         # Cloudflare DNS

# Reverse DNS (IP → домен)
dig -x 8.8.8.8

# Показать путь резолва
dig +trace google.com
```

### Типы DNS записей

- **A** — IPv4 адрес (google.com → 142.250.185.46)
- **AAAA** — IPv6 адрес
- **CNAME** — Алиас (www.example.com → example.com)
- **MX** — Mail сервер
- **NS** — Name server (какой DNS отвечает за домен)
- **TXT** — Текстовые записи (SPF, DKIM, верификация)
- **PTR** — Reverse DNS (IP → домен)

### Извлечение данных из dig

```bash
# Только IP
dig +short google.com

# Query time
dig google.com | grep "Query time"

# Query time (только значение)
dig google.com | grep "Query time" | awk '{print $4, $5}'

# Первый IP (если несколько)
dig +short google.com | head -1
```

### nslookup — базовые команды

```bash
# Простой lookup
nslookup google.com

# Через конкретный DNS
nslookup google.com 8.8.8.8
```

---

## Урок 7: Firewall (ufw)

### Основные команды ufw

```bash
# Проверить статус
ufw status
ufw status verbose
ufw status numbered

# Включить/выключить firewall
ufw enable
ufw disable

# Разрешить порт
ufw allow 22          # SSH
ufw allow 80          # HTTP
ufw allow 443         # HTTPS

# Разрешить конкретный протокол
ufw allow 22/tcp
ufw allow 53/udp

# Разрешить диапазон портов
ufw allow 6000:6007/tcp

# Заблокировать порт
ufw deny 3306         # MySQL

# Разрешить с конкретного IP
ufw allow from 192.168.1.100

# Разрешить с подсети
ufw allow from 192.168.1.0/24

# Разрешить IP к конкретному порту
ufw allow from 192.168.1.100 to any port 22

# Удалить правило
ufw delete allow 80
ufw delete 3          # По номеру

# Сбросить все правила
ufw reset

# Логирование
ufw logging on
ufw logging off
```

### Политика по умолчанию

```bash
# Запретить входящие, разрешить исходящие
ufw default deny incoming
ufw default allow outgoing
```

### Базовая настройка web сервера

```bash
ufw allow 22/tcp      # SSH (ВАЖНО! Сначала разрешить SSH)
ufw allow 80/tcp      # HTTP
ufw allow 443/tcp     # HTTPS
ufw default deny incoming
ufw default allow outgoing
ufw enable
```

### Проверка статуса в скрипте

```bash
# Получить статус (active/inactive)
STATUS=$(ufw status | grep "Status:" | awk '{print $2}')

# Проверить активен ли
if [ "$STATUS" = "active" ]; then
    echo "Firewall is active"
fi
```

---

## Урок 8: Траблшутинг сети (ping, traceroute, mtr)

### ping — проверка доступности

```bash
# Простой ping
ping google.com

# Ограничить количество пакетов
ping -c 4 google.com

# Установить timeout (секунды)
ping -W 2 google.com

# Установить интервал
ping -i 0.5 google.com      # Каждые 0.5 сек

# Тихий режим (только статистика)
ping -c 10 -q google.com

# Ping конкретного IP
ping 8.8.8.8
```

### Что показывает ping

- `time=23.4 ms` — задержка (latency)
- `ttl=117` — Time To Live
- `0% packet loss` — процент потерянных пакетов
- `rtt min/avg/max/mdev` — минимум/среднее/максимум/отклонение

### traceroute — путь до хоста

```bash
# Показать путь до хоста
traceroute google.com

# Использовать ICMP вместо UDP
traceroute -I google.com

# Ограничить количество хопов
traceroute -m 15 google.com

# Не резолвить имена (быстрее)
traceroute -n google.com
```

### mtr — интерактивный traceroute

```bash
# Запустить mtr (интерактивный)
mtr google.com

# Режим отчёта (10 пакетов)
mtr -r -c 10 google.com

# Без резолва имён
mtr -n google.com
```

### Извлечение данных из ping

```bash
# Проверить успешность (есть ли ответ)
if echo "$PING_OUTPUT" | grep -q "bytes from"; then
    echo "Reachable"
fi

# Извлечь среднюю задержку
AVG=$(echo "$PING_OUTPUT" | grep "avg" | awk -F'/' '{print $5}')

# Извлечь packet loss
PACKET_LOSS=$(echo "$PING_OUTPUT" | grep "packet loss" | awk '{print $6}')
```

### Проверка доступности в скрипте

```bash
# Вариант 1: через grep
PING_OUTPUT=$(ping -c 5 -W 2 $HOST 2>&1)
if echo "$PING_OUTPUT" | grep -q "bytes from"; then
    echo "Reachable"
else
    echo "Unreachable"
fi

# Вариант 2: через exit code
if ping -c 5 -W 2 $HOST > /dev/null 2>&1; then
    echo "Reachable"
else
    echo "Unreachable"
fi
```

---

## Важные концепции

### HTTP коды ответа

- **200-299** — Success (OK)
- **300-399** — Redirect
- **400-499** — Client Error (404 Not Found, 403 Forbidden)
- **500-599** — Server Error (500 Internal Server Error, 502 Bad Gateway)

### DNS propagation

Когда меняешь DNS записи, изменения распространяются не мгновенно:
- Разные DNS серверы могут возвращать разные результаты
- TTL (Time To Live) определяет, как долго запись кешируется
- Проверяй через несколько DNS серверов (8.8.8.8, 1.1.1.1)

### GeoDNS / Anycast

Крупные сервисы (Google, Cloudflare) возвращают разные IP в зависимости от:
- Географии запроса
- Нагрузки на датацентры
- DNS сервера, который спрашиваешь

Это нормально и не является ошибкой.

### Typosquatting

Регистрация доменов с опечатками популярных сайтов:
- `cloudeflare.com` (с `e`) vs `cloudflare.com` (без `e`)
- Используется для фишинга или рекламы
- В production проверяй домены внимательно

---

## Полезные команды для экзамена

```bash
# Проверить HTTP код
curl -o /dev/null -s -w "%{http_code}" https://google.com

# Проверить IP домена
dig +short google.com

# Проверить через конкретный DNS
dig @8.8.8.8 google.com

# Проверить доступность
ping -c 5 google.com

# Диагностика маршрута
traceroute -n google.com

# Статус firewall
ufw status

# Разрешить порт
ufw allow 22/tcp
```

---

## Типичные ошибки (не повторяй!)

1. ❌ Два curl запроса вместо одного (используй `%{http_code}|%{time_total}`)
2. ❌ Забыть pipe `|` перед `head -1`
3. ❌ Проверять "0% packet loss" вместо "bytes from" для доступности
4. ❌ Использовать `grep -q` в pipe (не выводит ничего)
5. ❌ Забыть кавычки в условиях: `[ -z $VAR ]` вместо `[ -z "$VAR" ]`
6. ❌ Включить firewall без разрешения SSH (потеряешь доступ!)
7. ❌ Опечатки в именах переменных (`$PIGN_OUTPUT` вместо `$PING_OUTPUT`)

---

## Созданные скрипты

1. **check_endpoint.sh** — проверка HTTP эндпоинтов (код, время ответа)
2. **check_dns.sh** — проверка DNS через Google и Cloudflare, сравнение
3. **setup_firewall.sh** — рекомендации по настройке firewall
4. **network_diag.sh** — диагностика сети (ping + traceroute)

---

**Повтори эту шпаргалку перед экзаменом!**
