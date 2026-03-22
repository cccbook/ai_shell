# 15. Запланированные задачи и мониторинг

---

## 15.1 Основы crontab

### Базовый формат

```
мин час день месяц день_недели  команда
┬    ┬    ┬    ┬       ┬         ┬
│    │    │    │       │         │
│    │    │    │       │         └─ команда
│    │    │    │       └─ день недели (0-7, 0 и 7 — воскресенье)
│    │    │    └─ месяц (1-12)
│    │    └─ день месяца (1-31)
│    └─ час (0-23)
└─ минута (0-59)
```

### Часто используемые примеры

```bash
# Каждую минуту
* * * * * /path/to/script.sh

# Каждый час в 30 минут
30 * * * * /path/to/script.sh

# Ежедневно в 3:00
0 3 * * * /path/to/script.sh

# Еженедельно в понедельник в 9:00
0 9 * * 1 /path/to/script.sh

# Каждые 5 минут
*/5 * * * * /path/to/script.sh
```

---

## 15.2 Управление crontab

```bash
# Показать текущий crontab
crontab -l

# Редактировать crontab
crontab -e

# Удалить crontab
crontab -r

# Создать файл crontab
cat > mycron << 'EOF'
# Ежедневное резервное копирование
0 3 * * * /scripts/backup.sh >> /var/log/backup.log 2>&1

# Еженедельная очистка
0 4 * * 0 /scripts/clean-logs.sh

# Проверка работоспособности каждые 5 минут
*/5 * * * * /scripts/health-check.sh
EOF

crontab mycron
crontab -l
```

---

## 15.3 Скрипт проверки работоспособности

```bash
cat > scripts/health-check.sh << 'EOF'
#!/bin/bash
set -euo pipefail

ALERT_EMAIL="admin@example.com"
WEB_URL="https://myapp.com/health"
LOG_FILE="/var/log/health-check.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $@" >> "$LOG_FILE"
}

check_web() {
    if curl -sf "$WEB_URL" &>/dev/null; then
        log "WEB: OK"
        return 0
    else
        log "WEB: FAILED"
        return 1
    fi
}

check_disk() {
    local usage=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
    if [[ $usage -gt 90 ]]; then
        log "DISK: WARNING (${usage}%)"
        return 1
    fi
    log "DISK: OK (${usage}%)"
    return 0
}

FAILED=0

check_web || ((FAILED++))
check_disk || ((FAILED++))

if [[ $FAILED -gt 0 ]]; then
    echo "Health check failed, $FAILED items abnormal" | mail -s "Alert" "$ALERT_EMAIL"
fi

exit $FAILED
EOF

chmod +x scripts/health-check.sh
```

---

## 15.4 Скрипт автоматической очистки

```bash
cat > scripts/clean-logs.sh << 'EOF'
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log"
MAX_AGE_DAYS=30

# Удалить старые логи
find "$LOG_DIR" -name "*.log" -mtime +$MAX_AGE_DAYS -delete

# Сжать старые логи
find "$LOG_DIR" -name "*.log" -mtime +7 -exec gzip {} \;

echo "Cleanup complete: $(date)"
EOF

chmod +x scripts/clean-logs.sh
```

---

## 15.5 Упражнения

1. Настройте crontab, который каждую минуту выводит "hello" в файл
2. Напишите скрипт мониторинга дискового пространства с оповещением при превышении 80%
3. Создайте скрипт автоматического резервного копирования базы данных MySQL
4. Используйте systemd timer вместо cron для запуска проверки работоспособности каждую минуту
