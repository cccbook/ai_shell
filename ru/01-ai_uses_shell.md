# 1. Как ИИ использует Shell

## Реальный проект с нуля

---

### 1.1 Фон проекта: Гибридный проект Python + C

Представьте, что вы — ИИ, которому поручено написать высокопроизводительный HTTP-клиент.

Ваша стратегия ясна: использовать **Python** для верхнего уровня логики (поддерживаемость, расширяемость) и **C** для критически важного сетевого уровня (быстрота, низкое потребление памяти). Затем Python вызывает C через `ctypes`.

Этот проект требует:
- Файл на C (`curl.c`): реализует высокопроизводительный HTTP GET
- Файл на Python (`curl.py`): оборачивает в удобный API
- Скрипт сборки: автоматизирует процесс компиляции
- Тестовые файлы: проверяют функциональность

В мире ИИ всё начинается с одной команды `mkdir`.

---

### 1.2 Использование Shell для создания структуры каталогов

Когда ИИ начинает новый проект, первое — не написание кода, а создание **чистой структуры каталогов**.

```bash
mkdir -p curl-project/src
mkdir -p curl-project/include
mkdir -p curl-project/tests
mkdir -p curl-project/scripts
```

Почему ИИ использует `mkdir -p`? Потому что `-p` безопасен:
- Если каталог уже существует, ошибки не будет
- Можно создать вложенные каталоги за один раз

Люди часто ошибаются, выполняя `mkdir project` и `mkdir project/src` отдельно, а потом удивляются, почему `src` не существует. Привычка ИИ — **сделать правильно с первого раза**.

Теперь ИИ записывает эти команды в скрипт `setup.sh` для будущего воспроизведения:

```bash
cat > setup.sh << 'EOF'
#!/bin/bash
set -e

PROJECT_NAME="curl-project"

mkdir -p "$PROJECT_NAME/src"
mkdir -p "$PROJECT_NAME/include"
mkdir -p "$PROJECT_NAME/tests"
mkdir -p "$PROJECT_NAME/scripts"

echo "✓ Структура проекта создана: $PROJECT_NAME/"
ls -la "$PROJECT_NAME"
EOF

chmod +x setup.sh
./setup.sh
```

Вы заметили? ИИ использовал синтаксис `cat > file << 'EOF'` — это **heredoc**, основная техника, которую ИИ использует для записи кода без редактора. Мы подробно рассмотрим это в Главе 4.

Вывод:

```
✓ Структура проекта создана: curl-project/
total 8
drwxr-xr-x  3 ai  staff   96 Mar 22 10:30 curl-project/
drwxr-xr-x  2 ai  staff   64 Mar 22 10:30 curl-project/include/
drwxr-xr-x  2 ai  staff   64 Mar 22 10:30 curl-project/scripts/
drwxr-xr-x  2 ai  staff   64 Mar 22 10:22 curl-project/src/
drwxr-xr-x  2 ai  staff   64 Mar 22 10:30 curl-project/tests/
```

---

### 1.3 Написание кода на C с помощью heredoc

Теперь ИИ пишет основной код на C. Для людей открыть Vim или VS Code естественно. Но подход ИИ таков:

```bash
cat > src/curl.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>

struct MemoryBuffer {
    char *data;
    size_t size;
};

static size_t write_callback(void *contents, size_t size, 
                             size_t nmemb, void *userp) {
    size_t realsize = size * nmemb;
    struct MemoryBuffer *mem = (struct MemoryBuffer *)userp;
    
    char *ptr = realloc(mem->data, mem->size + realsize + 1);
    if (!ptr) {
        fprintf(stderr, "Ошибка выделения памяти\n");
        return 0;
    }
    
    mem->data = ptr;
    memcpy(&(mem->data[mem->size]), contents, realsize);
    mem->size += realsize;
    mem->data[mem->size] = 0;
    
    return realsize;
}

int fetch_url(const char *url, char **response, size_t *len) {
    CURL *curl;
    CURLcode res;
    struct MemoryBuffer chunk;
    
    chunk.data = malloc(1);
    chunk.size = 0;
    
    curl = curl_easy_init();
    if (!curl) {
        fprintf(stderr, "Ошибка инициализации CURL\n");
        free(chunk.data);
        return 1;
    }
    
    curl_easy_setopt(curl, CURLOPT_URL, url);
    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_callback);
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, (void *)&chunk);
    curl_easy_setopt(curl, CURLOPT_FOLLOWLOCATION, 1L);
    curl_easy_setopt(curl, CURLOPT_TIMEOUT, 30L);
    
    res = curl_easy_perform(curl);
    
    if (res != CURLE_OK) {
        fprintf(stderr, "Запрос не удался: %s\n", curl_easy_strerror(res));
        curl_easy_cleanup(curl);
        free(chunk.data);
        return 1;
    }
    
    long response_code;
    curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &response_code);
    printf("HTTP код ответа: %ld\n", response_code);
    
    *response = chunk.data;
    *len = chunk.size;
    
    curl_easy_cleanup(curl);
    return 0;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Использование: %s <URL>\n", argv[0]);
        return 1;
    }
    
    char *result = NULL;
    size_t len = 0;
    
    int status = fetch_url(argv[1], &result, &len);
    
    if (status == 0 && result) {
        printf("Содержимое ответа (%zu байт):\n%s\n", len, result);
        free(result);
    }
    
    return status;
}
EOF
```

**Что делает этот код?**

1. Использует libcurl для реализации HTTP GET запросов
2. Динамически выделяет память для хранения содержимого ответа
3. Реализует callback-функцию для обработки потоков данных
4. Предоставляет функцию `fetch_url()` для внешних вызовов
5. Функция `main()` служит инструментом командной строки

---

### 1.4 Написание Python-обёртки с помощью heredoc

Теперь ИИ пишет файл на Python для вызова C:

```bash
cat > src/__init__.py << 'EOF'
"""
curl-py: Гибридный HTTP-клиент Python + C
"""

import ctypes
import os
from pathlib import Path

class CurlError(Exception):
    pass

class CurlFetcher:
    def __init__(self, lib_path=None):
        if lib_path is None:
            base_dir = Path(__file__).parent
            lib_path = base_dir / "libcurl_ext.so"
        
        if not os.path.exists(lib_path):
            raise CurlError(f"Библиотека не найдена: {lib_path}")
        
        self.lib = ctypes.CDLL(str(lib_path))
        self.lib.fetch_url.argtypes = [ctypes.c_char_p, 
                                       ctypes.POINTER(ctypes.c_char_p),
                                       ctypes.POINTER(ctypes.c_size_t)]
        self.lib.fetch_url.restype = ctypes.c_int
    
    def get(self, url):
        response = ctypes.c_char_p()
        length = ctypes.c_size_t()
        
        if isinstance(url, str):
            url = url.encode('utf-8')
        
        result = self.lib.fetch_url(url, ctypes.byref(response), 
                                    ctypes.byref(length))
        
        if result != 0:
            raise CurlError(f"Запрос не удался, код ошибки: {result}")
        
        return response.value.decode('utf-8')
EOF
```

Затем пишет удобный интерфейс:

```bash
cat > curl.py << 'EOF'
#!/usr/bin/env python3
"""
curl.py - HTTP-клиент командной строки
Использование: python curl.py <URL>
"""

from src import CurlFetcher, CurlError
import sys

def main():
    if len(sys.argv) != 2:
        print(f"Использование: {sys.argv[0]} <URL>", file=sys.stderr)
        sys.exit(1)
    
    url = sys.argv[1]
    
    try:
        fetcher = CurlFetcher()
        print(f"Запрос: {url}")
        content = fetcher.get(url)
        print(content)
    except CurlError as e:
        print(f"Ошибка: {e}", file=sys.stderr)
        sys.exit(1)
    except KeyboardInterrupt:
        print("\nОтменено", file=sys.stderr)
        sys.exit(130)

if __name__ == "__main__":
    main()
EOF
```

---

### 1.5 Компиляция кода на C с помощью gcc

Компиляция — один из самых «видимых» этапов в работе ИИ. ИИ делает это так:

```bash
gcc -shared -fPIC -o libcurl_ext.so src/curl.c -lcurl -Wall -Wextra
```

**Почему ИИ пишет именно так?**

- `-shared -fPIC`: создаёт позиционно-независимый код, генерирует разделяемую библиотеку
- `-O2`: оптимизация уровня 2, улучшает эффективность выполнения
- `-lcurl`: компонует библиотеку libcurl
- `-Wall -Wextra`: включает все предупреждения, избегает скрытых проблем

ИИ оборачивает это в скрипт сборки:

```bash
cat > scripts/build.sh << 'EOF'
#!/bin/bash
set -e

LIB_NAME="libcurl_ext.so"
SRC_FILE="src/curl.c"

echo "🔨 Компиляция $SRC_FILE ..."

if ! command -v gcc &> /dev/null; then
    echo "Ошибка: gcc не найден, установите Xcode Command Line Tools"
    echo "Выполните: xcode-select --install"
    exit 1
fi

gcc -shared -fPIC -O2 \
    -o "$LIB_NAME" \
    "$SRC_FILE" \
    -lcurl \
    -Wall \
    -Wextra

if [ $? -eq 0 ]; then
    echo "✓ Сборка успешна: $LIB_NAME"
    ls -lh "$LIB_NAME"
else
    echo "✗ Сборка не удалась"
    exit 1
fi
EOF

chmod +x scripts/build.sh
./scripts/build.sh
```

---

### 1.6 Автоматизация всего процесса с помощью Shell-скрипта

В этом суть ИИ: **автоматизация повторяющейся работы**.

```bash
cat > scripts/dev.sh << 'EOF'
#!/bin/bash
set -e

PROJECT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
cd "$PROJECT_DIR"

echo "============================================"
echo "  Среда разработки curl-project"
echo "============================================"

echo ""
echo "🧹 Очистка старых файлов..."
rm -f libcurl_ext.so
rm -rf __pycache__ src/__pycache__

echo ""
echo "🔨 Компиляция кода на C..."
./scripts/build.sh

if [ -f "libcurl_ext.so" ]; then
    echo ""
    echo "🧪 Тестирование C-программы..."
    gcc -o curl_test src/curl.c -lcurl 2>/dev/null && {
        ./curl_test https://httpbin.org/get || true
        rm -f curl_test
    } || echo "(Пропуск теста C, отсутствует libcurl-dev)"
fi

echo ""
echo "🐍 Тестирование Python-обёртки..."
if [ -f "src/__init__.py" ]; then
    python3 -c "from src import CurlFetcher; print('✓ Python-модуль загружен успешно')" 2>/dev/null || {
        echo "⚠ Тест Python-модуля требует предварительной компиляции C"
    }
fi

echo ""
echo "============================================"
echo "  Готово!"
echo "  Выполните 'python3 curl.py <URL>' для запуска"
echo "============================================"
EOF

chmod +x scripts/dev.sh
```

Запуск:

```bash
./scripts/dev.sh
```

```
============================================
  Среда разработки curl-project
============================================

🧹 Очистка старых файлов...

🔨 Компиляция кода на C...
✓ Сборка успешна: libcurl_ext.so
-rwxr-xr-x  1 ai  staff  45032 Mar 22 10:35 libcurl_ext.so

🐍 Тестирование Python-обёртки...
✓ Python-модуль загружен успешно

============================================
  Готово!
  Выполните 'python3 curl.py <URL>' для запуска
============================================
```

---

### 1.7 Рабочий процесс отладки и исправления ИИ

#### Уровень 1: Ошибки компиляции

Предположим, первый код на C ИИ содержит синтаксическую ошибку:

```bash
gcc -shared -fPIC -o libcurl_ext.so src/curl.c -lcurl
```

Вывод:

```
src/curl.c:15:10: error: unknown type name 'size_t'
```

Ответ ИИ: **Исправить немедленно**. Частые проблемы:
- Забыт `#include <stdlib.h>` (`size_t`, `malloc`, `free`)
- Забыт `#include <string.h>` (`memcpy`)
- Проблемы с путём к заголовочному файлу libcurl

Исправление простое:

```bash
cat > src/curl.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>
// ... полное содержимое ...
EOF
```

#### Уровень 2: Ошибки выполнения

Если программа компилируется, но не запускается:

```bash
./curl_test https://example.com
# Вывод: Запрос не удался: Couldn't resolve host name
```

ИИ проверяет:
1. Работает ли сеть: `ping -c 1 8.8.8.8`
2. Доступен ли DNS: `nslookup example.com`
3. Блокирует ли файрвол

#### Уровень 3: Логические ошибки

Логические ошибки сложнее всего отлаживать. Метод ИИ — **пошаговая проверка**:

```bash
bash -x ./scripts/build.sh
```

Выводит каждую выполненную команду и значения переменных

#### Уровень 4: Проблемы с памятью

Если в программе на C есть утечки памяти, ИИ предлагает использовать `valgrind`:

```bash
valgrind --leak-check=full ./curl_test https://example.com
```

---

### 1.8 Полная структура проекта

```
curl-project/
├── src/
│   ├── curl.c           # Реализация на C
│   └── __init__.py     # Python-обёртка
├── include/            # Заголовочные файлы C (резерв)
├── tests/              # Тестовые файлы (резерв)
├── scripts/
│   ├── setup.sh        # Инициализация каталогов
│   ├── build.sh        # Скрипт сборки
│   └── dev.sh          # Среда разработки
├── libcurl_ext.so      # Результат сборки
└── curl.py             # Инструмент командной строки
```

---

### 1.9 Ключевые выводы по Shell из этого проекта

1. **`mkdir -p`**: безопасно создаёт каталоги, поддерживает вложенную структуру
2. **`cat > file << 'EOF'`**: записывает файлы без редактора
3. **`chmod +x`**: даёт разрешение на выполнение
4. **`set -e`**: останавливает скрипт при любой ошибке команды
5. **`&&` и `||`**: объединяет условные команды
6. **`gcc ...`**: стандартный рабочий процесс компиляции C
7. **`bash -x`**: отлаживает скрипт пошагово

---

### 1.10 Итоги главы

Когда ИИ пишет код, он **никогда не покидает Терминал**.

От создания структуры папок, написания кода, компиляции, запуска до отладки — всё делается на чёрном экране с белым текстом. Никаких подсказок VS Code, никаких кликов мышью, никакого WYSIWYG-редактора.

Это не ограничение, это **эффективность**.

После прочтения этой главы вы должны понимать:
- Как ИИ использует отдельные команды для выполнения задач
- Как heredoc делает Shell «генератором текста»
- Почему рабочий процесс ИИ такой быстрый и воспроизводимый

В следующих главах мы подробно рассмотрим каждый аспект: от команд к скриптам, от отладки к оптимизации.
