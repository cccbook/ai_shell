# 14. Скрипты деплоя

---

## 14.1 Ключевые элементы хорошего скрипта деплоя

Хороший скрипт деплоя должен:
1. Поддерживать режим предварительного просмотра (dry-run)
2. Иметь понятный вывод логов
3. Выполнять откат при неудаче
4. Обрабатывать прерывания
5. Отслеживать версии

---

## 14.2 Простой скрипт деплоя

```bash
cat > deploy.sh << 'EOF'
#!/bin/bash
set -euo pipefail

DEPLOY_HOST="server.example.com"
DEPLOY_PATH="/var/www/myapp"
DEPLOY_USER="deploy"

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log() { echo -e "${GREEN}[INFO]${NC} $@"; }
warn() { echo -e "${YELLOW}[WARN]${NC} $@"; }
error() { echo -e "${RED}[ERROR]${NC} $@" >&2; }

check() {
    command -v rsync &>/dev/null || { error "Требуется rsync"; exit 1; }
    if ! ssh -o ConnectTimeout=5 "$DEPLOY_USER@$DEPLOY_HOST" "exit 0" 2>/dev/null; then
        error "Не удаётся подключиться к $DEPLOY_HOST"
        exit 1
    fi
    log "Предварительные проверки пройдены"
}

backup() {
    log "Создание резервной копии..."
    TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    ssh "$DEPLOY_USER@$DEPLOY_HOST" "cp -r $DEPLOY_PATH $DEPLOY_PATH.backup.$TIMESTAMP"
    log "Резервная копия создана: $DEPLOY_PATH.backup.$TIMESTAMP"
}

deploy() {
    log "Начало деплоя..."
    rsync -avz --delete \
        --exclude='.git' \
        --exclude='node_modules' \
        --exclude='.env' \
        ./ "$DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH/"
    log "Файлы синхронизированы"
}

restart() {
    log "Перезапуск сервиса..."
    ssh "$DEPLOY_USER@$DEPLOY_HOST" "systemctl restart myapp"
    sleep 2
    
    if ssh "$DEPLOY_USER@$DEPLOY_HOST" "systemctl is-active myapp" | grep -q "active"; then
        log "Сервис успешно запущен"
    else
        error "Не удалось запустить сервис"
        exit 1
    fi
}

main() {
    log "🚀 Начало деплоя"
    log "Цель: $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH"
    
    check
    backup
    deploy
    restart
    
    log "✅ Деплой завершён!"
}

main
EOF

chmod +x deploy.sh
```

---

## 14.3 Поддержка режима предварительного просмотра (dry-run)

```bash
cat > deploy.sh << 'EOF'
#!/bin/bash
set -euo pipefail

DRY_RUN=false
DEPLOY_HOST="server.example.com"

while [[ $# -gt 0 ]]; do
    case "$1" in
        -n|--dry-run)
            DRY_RUN=true
            shift
            ;;
        *)
            shift
            ;;
    esac
done

log() { echo "[INFO] $@"; }

deploy() {
    if $DRY_RUN; then
        log "[DRY-RUN] Синхронизация файлов в $DEPLOY_HOST"
        rsync -avz --dry-run ./ "$DEPLOY_HOST:/tmp/test/"
    else
        log "Начало деплоя..."
        rsync -avz ./ "$DEPLOY_HOST:/var/www/app/"
    fi
}

deploy
EOF
```

---

## 14.4 Деплой в несколько окружений

```bash
cat > deploy.sh << 'EOF'
#!/bin/bash
set -euo pipefail

ENVIRONMENT="${1:-staging}"

case "$ENVIRONMENT" in
    staging)
        HOST="staging.example.com"
        PATH="/var/www/staging"
        ;;
    production)
        HOST="prod.example.com"
        PATH="/var/www/production"
        CONFIRM=true
        ;;
    *)
        echo "Неизвестное окружение: $ENVIRONMENT"
        exit 1
        ;;
esac

log() { echo "🚀 [$ENVIRONMENT] $@"; }

if [[ "$CONFIRM" == "true" ]]; then
    echo "Будет выполнен деплой в production: $HOST"
    read -p "Продолжить? (д/н) " -n 1 -r
    echo
    [[ $REPLY =~ ^[Дд]$ ]] || exit 0
fi

log "Начало деплоя в $HOST"

rsync -avz --delete \
    --exclude='.env' \
    ./ "$HOST:$PATH/"

ssh "$HOST" "cd $PATH && npm install --production"

log "Деплой завершён"
EOF
```

---

## 14.5 Упражнения

1. Напишите скрипт деплоя с поддержкой опции `--dry-run`
2. Добавьте функциональность резервного копирования в скрипт деплоя
3. Напишите скрипт деплоя для нескольких окружений (dev/staging/production)
4. Создайте скрипт отката, который показывает список версий и позволяет выбрать нужную для отката
