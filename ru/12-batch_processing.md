# 12. Пакетная обработка файлов

---

## 12.1 Пакетное переименование

### Простая замена расширения

```bash
# Изменить все .txt на .md
for f in *.txt; do
    mv "$f" "${f%.txt}.md"
done
```

### Добавить префикс

```bash
# Добавить префикс thumb_ ко всем изображениям
for f in *.jpg *.png *.gif; do
    [[ -f "$f" ]] || continue
    mv "$f" "thumb_$f"
done
```

### Удалить пробелы

```bash
# Заменить пробелы в именах файлов на подчёркивания
for f in *\ *; do
    [[ -f "$f" ]] || continue
    mv "$f" "${f// /_}"
done
```

### Добавить номера

```bash
# Пронумеровать файлы 001, 002, 003...
i=1
for f in *.jpg; do
    [[ -f "$f" ]] || continue
    mv "$f" "$(printf '%03d.jpg' $i)"
    ((i++))
done
```

---

## 12.2 Пакетная обработка изображений

```bash
#!/bin/bash
set -euo pipefail

SIZE=${1:-800}

for img in *.jpg *.png; do
    [[ -f "$img" ]] || continue
    
    echo "Обработка: $img"
    
    if command -v convert &>/dev/null; then
        # Создать миниатюру
        convert "$img" -resize "${SIZE}x${SIZE}>" "thumb_$img"
        
        # Создать крупную версию
        convert "$img" -resize 1920x1080 "large_$img"
        
        echo "✓ Готово: thumb_$img, large_$img"
    fi
done
```

---

## 12.3 Пакетный поиск и замена

```bash
#!/bin/bash

# Заменить "foo" на "bar" во всех .txt файлах
for f in *.txt; do
    [[ -f "$f" ]] || continue
    sed -i.bak 's/foo/bar/g' "$f"
done
```

---

## 12.4 Пакетное сжатие

```bash
#!/bin/bash

# Сжать каждый файл
for f in *.txt; do
    [[ -f "$f" ]] || continue
    gzip "$f"
done

# Распаковать все
gunzip *.gz
```

---

## 12.5 Пакетная загрузка

```bash
#!/bin/bash
set -euo pipefail

while read -r url; do
    filename=$(basename "$url")
    echo "Загрузка: $url"
    curl -L -o "$filename" "$url"
done < urls.txt
```

---

## 12.6 Краткий справочник

| Задача | Команда |
|--------|---------|
| Изменить расширение | `mv "$f" "${f%.old}.new"` |
| Добавить префикс | `mv "$f" "prefix_$f"` |
| Удалить пробелы | `mv "$f" "${f// /_}"` |
| Добавить номера | `mv "$f" "$(printf '%03d' $i)"` |
| Пакетная замена | `for f in *.txt; do sed -i 's/a/b/g' "$f"; done` |
| Пакетное сжатие | `for f in *.txt; do gzip "$f"; done` |
| Пакетная загрузка | `while read url; do curl -LO "$url"; done < urls.txt` |

---

## 12.7 Упражнения

1. Пакетно переименовать 10 файлов с `.txt` на `.md`
2. Добавить префикс `thumb_` ко всем изображениям и создать миниатюры
3. Заменить `old-site.com` на `new-site.com` во всех `.html` файлах
4. Создать скрипт очистки, удаляющий все файлы `__pycache__`, `.pyc` и `.log`
