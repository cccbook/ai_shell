# 11. Логирование и вывод

---

## 11.1 Почему логирование важно

Без логирования:
- Не знаем, где находится скрипт
- Не знаем, почему он завершился неудачно
- Не знаем, что он сделал при успехе

С логированием:
- Можем отслеживать прогресс
- Имеем достаточно информации для отладки неудач
- Можем проверять историю выполнения

---

## 11.2 Базовые уровни логирования

```bash
#!/bin/bash

DEBUG=0
INFO=1
WARN=2
ERROR=3

LOG_LEVEL=${LOG_LEVEL:-$INFO}

log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    if [[ $level -ge $LOG_LEVEL ]]; then
        echo "[$timestamp] $message"
    fi
}

log $DEBUG "Это отладка"
log $INFO "Это информация"
log $WARN "Это предупреждение"
log $ERROR "Это ошибка"
```

---

## 11.3 Цветной вывод

```bash
#!/bin/bash

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'  # Без цвета

log_info() { echo -e "${GREEN}[INFO]${NC} $@"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $@"; }
log_error() { echo -e "${RED}[ERROR]${NC} $@" >&2; }

log_info "Установка завершена"
log_warn "Используются значения по умолчанию"
log_error "Ошибка подключения"
```

---

## 11.4 Вывод в файл

```bash
#!/bin/bash

LOG_FILE="/var/log/myapp.log"

log() {
    local message="[$(date '+%Y-%m-%d %H:%M:%S')] $@"
    echo "$message"
    echo "$message" >> "$LOG_FILE"
}

log "Приложение запущено"
```

---

## 11.5 `tee`: Вывод на экран и в файл одновременно

```bash
# Показать и сохранить одновременно
echo "Привет" | tee output.txt

# Режим добавления
echo "Мир" | tee -a output.txt

# Перехватить stderr тоже
./script.sh 2>&1 | tee output.log
```

---

## 11.6 Индикаторы прогресса

### Простые точки

```bash
echo -n "Обработка"
for i in {1..10}; do
    echo -n "."
    sleep 0.2
done
echo " Готово"
```

### Прогресс-бар

```bash
draw_progress() {
    local current=$1
    local total=$2
    local width=40
    local percent=$((current * 100 / total))
    local chars=$((width * current / total))
    
    printf "\r[%s%s] %3d%%" \
        "$(printf '%*s' $chars | tr ' ' '=')" \
        "$(printf '%*s' $((width - chars)) | tr ' ' '-')" \
        "$percent"
    
    [[ $current -eq $total ]] && echo
}
```

---

## 11.7 Объяснение `2>&1`

```bash
# 1 = stdout, 2 = stderr

# Перенаправить stderr в stdout
command 2>&1

# Перенаправить stdout в файл, stderr на экран
command > output.txt

# Перенаправить оба в файл
command > output.txt 2>&1

# Перенаправить оба в /dev/null (скрыть)
command > /dev/null 2>&1
```

---

## 11.8 Краткий справочник

| Синтаксис | Описание |
|-----------|----------|
| `echo "текст"` | Базовый вывод |
| `echo -e "\033[31m"` | Цветной вывод |
| `2>&1` | Перенаправить stderr в stdout |
| `> file` | Перезаписать файл |
| `>> file` | Добавить в файл |
| `tee file` | Вывести и сохранить |
| `tee -a file` | Режим добавления |
| `/dev/null` | Отбросить вывод |

---

## 11.9 Упражнения

1. Напишите скрипт, отображающий сообщения INFO, WARN, ERROR разными цветами
2. Используйте `tee` для одновременного отображения и сохранения вывода в лог-файл
3. Создайте скрипт обработки файлов с прогресс-баром
4. Создайте библиотеку логирования с поддержкой вывода в файл и уровней логирования
