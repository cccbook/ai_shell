# 21. Создайте свою ИИ Shell инструментарий

---

## 21.1 Зачем вам нужен инструментарий

Когда вы обнаруживаете, что повторно делаете одно и то же, пора автоматизировать и собрать это в инструментарий.

ИИ особенно хорош в:
- Быстром создании инструментов
- Обёртывании сложных рабочих процессов в простые команды
- Постоянном улучшении часто используемых скриптов

---

## 21.2 Структура каталогов инструментария

```
~/bin/
├── lib/              # Общие библиотеки
│   ├── logging.sh
│   ├── utils.sh
│   └── colors.sh
├── project/          # Шаблоны проектов
│   ├── python/
│   ├── nodejs/
│   └── static-site/
├── scripts/          # Инструментальные скрипты
│   ├── git-clean
│   ├── docker-clean
│   ├── backup
│   └── log-parse
└── bin/
    ├── greet         # Исполняемый инструмент
    └── monitor       # Исполняемый инструмент
```

---

## 21.3 Создайте свою библиотеку

### logging.sh

```bash
cat > ~/bin/lib/logging.sh << 'EOF'
#!/bin/bash

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log_info() { echo -e "${GREEN}[INFO]${NC} $@"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $@"; }
log_error() { echo -e "${RED}[ERROR]${NC} $@" >&2; }
EOF

source ~/bin/lib/logging.sh
```

### utils.sh

```bash
cat > ~/bin/lib/utils.sh << 'EOF'
#!/bin/bash

need_command() {
    command -v "$1" &>/dev/null || {
        echo "Required command: $1"
        exit 1
    }
}

need_file() {
    [[ -f "$1" ]] || {
        echo "Required file: $1"
        exit 1
    }
}

need_dir() {
    [[ -d "$1" ]] || {
        echo "Required directory: $1"
        exit 1
    }
}
EOF
```

---

## 21.4 Практический инструмент: git-clean

```bash
cat > ~/bin/git-clean << 'EOF'
#!/bin/bash
set -euo pipefail

DRY_RUN=false
while [[ $# -gt 0 ]]; do
    case "$1" in
        -n|--dry-run) DRY_RUN=true; shift ;;
        *) shift ;;
    esac
done

if $DRY_RUN; then
    echo "[DRY-RUN] Would delete:"
fi

git branch --merged main | grep -v "main\|master\|develop" | while read branch; do
    if $DRY_RUN; then
        echo "  Delete branch: $branch"
    else
        git branch -d "$branch"
        echo "Deleted: $branch"
    fi
done

git clean -n -d
EOF

chmod +x ~/bin/git-clean
```

---

## 21.5 Практический инструмент: docker-clean

```bash
cat > ~/bin/docker-clean << 'EOF'
#!/bin/bash
set -euo pipefail

echo "Stopping all containers..."
docker stop $(docker ps -aq) 2>/dev/null || true

echo "Removing stopped containers..."
docker container prune -f

echo "Removing unused images..."
docker image prune -af

echo "Removing unused networks..."
docker network prune -f

echo "Removing build cache..."
docker builder prune -af

echo "✅ Docker cleanup complete"
docker system df
EOF

chmod +x ~/bin/docker-clean
```

---

## 21.6 Сделайте инструменты доступными в PATH

```bash
# Проверить, есть ли ~/bin в PATH
echo $PATH | grep -q "$HOME/bin" && echo "Set" || echo "Not set"

# Добавить в PATH (добавить в ~/.bashrc или ~/.zshrc)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 21.7 Позвольте ИИ помочь расширить инструментарий

```bash
# Человек: помоги создать инструмент для анализа логов доступа Nginx

# ИИ:
cat > ~/bin/nginx-analyze << 'EOF'
#!/bin/bash

if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <access_log_file>"
    exit 1
fi

FILE=$1

echo "=== Nginx Analysis: $FILE ==="
echo ""

echo "File size: $(du -h "$FILE" | cut -f1)"
echo "Total lines: $(wc -l < "$FILE")"
echo ""

echo "=== Top 10 IPs ==="
awk '{print $1}' "$FILE" | sort | uniq -c | sort -rn | head -10
echo ""

echo "=== Top 10 URLs ==="
awk '{print $7}' "$FILE" | sort | uniq -c | sort -rn | head -10
echo ""

echo "=== Status Code Distribution ==="
awk '{print $9}' "$FILE" | sort | uniq -c | sort -rn
EOF

chmod +x ~/bin/nginx-analyze
```

---

## 21.8 Постоянное улучшение

```bash
# Просматривайте инструментарий ежегодно
# - Какие инструменты редко используются? Удалите
# - Какие инструменты можно улучшить?
# - Какие повторяющиеся задачи можно автоматизировать?

# Версионируйте ваш инструментарий
cd ~/bin
git init
git add .
git commit -m "Initial version"
```

---

## 21.9 Упражнения

1. Создайте структуру каталогов ~/bin
2. Запишите часто используемые функции в переиспользуемые библиотеки
3. Напишите инструмент для задач, которые вы повторяете ежедневно
4. Попросите ИИ помочь создать инструмент анализа логов Nginx
5. Поместите ваш инструментарий под контроль версий Git
