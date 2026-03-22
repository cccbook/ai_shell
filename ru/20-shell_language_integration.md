# 20. Связь Shell с другими языками

---

## 20.1 Какие задачи лучше решать в Shell

Задачи, где Shell превосходит:
- Операции с файлами
- Системное администрирование
- Автоматизация рабочих процессов
- Композиция пайплайнов
- Быстрое прототипирование

Задачи, лучше подходящие для других языков:
- Сложная обработка данных (Python, AWK)
- Веб-программирование (JavaScript, Go)
- Численные вычисления (Python, R, Julia)
- Системное программирование (C, Rust)

---

## 20.2 Shell вызывает Python

### Простой вызов

```bash
#!/bin/bash

# Вызвать Python-скрипт
python3 process_data.py input.csv

# Передать аргументы
python3 script.py arg1 arg2

# Получить вывод
result=$(python3 -c "print('hello from python')")
echo "$result"
```

### Встроенный Python в Shell

```bash
#!/bin/bash

python3 << 'PYEOF'
import csv

with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    total = sum(int(row['amount']) for row in reader)
    print(f"Total: {total}")
PYEOF
```

---

## 20.3 Python вызывает Shell

### Использование `subprocess`

```python
import subprocess

# Выполнить простую команду
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)

# Выполнить сложную команду
result = subprocess.run(
    'find . -name "*.py" | wc -l',
    shell=True,
    capture_output=True,
    text=True
)
```

---

## 20.4 Shell вызывает JavaScript/Node.js

```bash
#!/bin/bash

# Выполнить команду Node.js
node -e "console.log('hello from node')"

# Выполнить Node.js скрипт
node process-json.js data.json
```

---

## 20.5 Инструменты связывания

### `jq`: Обработка JSON

```bash
# Разобрать JSON
echo '{"name": "Alice", "age": 25}' | jq '.name'

# Читать из файла
jq '.users[] | select(.age > 18)' data.json
```

### `yq`: Обработка YAML

```bash
# Разобрать YAML
yq '.database.host' config.yaml

# Конвертировать YAML в JSON
yq -o=json config.yaml
```

---

## 20.6 Краткий справочник

| Связь | Синтаксис |
|-------|-----------|
| Shell→Python | `python3 script.py` или `python3 << PYEOF` |
| Python→Shell | `subprocess.run(['ls'])` |
| Shell→Node | `node script.js` или `node << JSEOF` |
| Разбор JSON | `jq '.key' file.json` |
| Разбор YAML | `yq '.key' file.yaml` |
| Оркестрация | `make` |

---

## 20.7 Упражнения

1. Используйте Shell для вызова Python для обработки CSV-файла
2. Используйте Python `subprocess` для выполнения Shell-команд
3. Используйте `jq` для разбора JSON-ответа API
4. Используйте Makefile для оркестрации задач Shell, Python и Node.js
