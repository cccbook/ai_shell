# 13. Автоматизированное тестирование

---

## 13.1 Зачем использовать Shell для тестирования

- Быстрое выполнение простых проверок
- Тестирование системных команд и скриптов
- Интеграционное тестирование в CI/CD
- Регрессионное тестирование существующих скриптов

---

## 13.2 Базовый тестовый фреймворк

```bash
#!/bin/bash
set -euo pipefail

TESTS_RUN=0
TESTS_PASSED=0
TESTS_FAILED=0

test() {
    local name=$1
    local command=$2
    local expected=$3
    
    ((TESTS_RUN++))
    
    local actual
    actual=$(eval "$command" 2>/dev/null)
    
    if [[ "$actual" == "$expected" ]]; then
        echo "✓ $name"
        ((TESTS_PASSED++))
    else
        echo "✗ $name"
        echo "  Ожидалось: $expected"
        echo "  Получено: $actual"
        ((TESTS_FAILED++))
    fi
}

test "Сложение" "echo \$((1 + 1))" "2"
test "Строки равны" "echo привет" "привет"

echo ""
echo "Результаты: $TESTS_PASSED/$TESTS_RUN прошло"
[[ $TESTS_FAILED -eq 0 ]] || exit 1
```

---

## 13.3 Тестирование файлов и директорий

```bash
#!/bin/bash

test_file_exists() {
    local file=$1
    [[ -f "$file" ]] || { echo "✗ Файл не найден: $file"; return 1; }
}

test_dir_exists() {
    local dir=$1
    [[ -d "$dir" ]] || { echo "✗ Директория не найдена: $dir"; return 1; }
}

test_file_contains() {
    local file=$1
    local pattern=$2
    grep -q "$pattern" "$file" || { echo "✗ В файле отсутствует: $pattern"; return 1; }
}
```

---

## 13.4 Тестирование вывода команд

```bash
#!/bin/bash

test_success() {
    local name=$1
    shift
    if "$@"; then
        echo "✓ $name"
    else
        echo "✗ $name (код завершения: $?)"
        return 1
    fi
}

test_failure() {
    local name=$1
    shift
    if ! "$@" 2>/dev/null; then
        echo "✓ $name (корректно завершилось неудачей)"
    else
        echo "✗ $name (должно было завершиться неудачей, но успешно)"
        return 1
    fi
}

test_output() {
    local name=$1
    local expected=$2
    shift 2
    
    local output=$("$@" 2>/dev/null)
    if [[ "$output" == *"$expected"* ]]; then
        echo "✓ $name"
    else
        echo "✗ $name"
        return 1
    fi
}
```

---

## 13.5 Тестовый скрипт для CI

```bash
cat > scripts/test.sh << 'EOF'
#!/bin/bash
set -euo pipefail

RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

FAILED=0

run_test() {
    local name=$1
    local cmd=$2
    
    echo -n "Тест: $name ... "
    if eval "$cmd" &>/dev/null; then
        echo -e "${GREEN}✓${NC}"
    else
        echo -e "${RED}✗${NC}"
        ((FAILED++))
    fi
}

run_test "Скрипт инициализации существует" "[[ -x scripts/init-project.sh ]]"
run_test "Скрипт сборки существует" "[[ -f scripts/build.sh ]]"
run_test "Скрипт деплоя существует" "[[ -f scripts/deploy.sh ]]"

echo ""
if [[ $FAILED -eq 0 ]]; then
    echo -e "${GREEN}✅ Все тесты прошли${NC}"
    exit 0
else
    echo -e "${RED}❌ $FAILED тестов не прошло${NC}"
    exit 1
fi
EOF

chmod +x scripts/test.sh
```

---

## 13.6 Упражнения

1. Напишите тестовый фреймворк с функциями `test_eq`, `test_contains`, `test_success`
2. Напишите 5 тестов для одного из ваших скриптов
3. Создайте тестовый скрипт для CI с выводом в формате TAP
4. Протестируйте скрипт пакетной обработки, чтобы убедиться в корректной обработке всех файлов
