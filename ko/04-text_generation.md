# 4. 텍스트 생성 및 쓰기

---

## 4.1 왜 AI는 편집기가 필요 없는가

대부분의 인간 엔지니어의 코드 작성 워크플로우:
1. 편집기 열기 (VS Code, Vim, Emacs...)
2. 코드 입력
3. 파일 저장
4. 편집기 닫기

AI에게:
```
"Python 프로그램을 작성해줘" = 일부 텍스트 생성
"이 프로그램을 파일에 저장해줘" = 텍스트를 디스크에 쓰기
```

AI의 코드 생성 프로세스는 **텍스트 생성 프로세스**입니다. 그래서 AI는 Shell의 텍스트 도구를 사용합니다:

- `echo`: 한 줄의 텍스트 출력
- `printf`: 형식화된 출력
- `heredoc`: 여러 줄의 텍스트 출력 (가장 중요!)

---

## 4.2 `echo`: 가장 간단한 출력

### 기본 사용법

```bash
# 문자열 출력
echo "Hello, World!"

# 변수 출력
name="Alice"
echo "Hello, $name!"

# 여러 값 출력
echo "Today is $(date +%Y-%m-%d)"
```

### `echo` 함정

```bash
# echo는 기본적으로 줄바꿈 추가
echo -n "Loading: "  # 줄바꿈 없음
```

### `echo`로 파일 쓰기

```bash
# 덮어쓰기
echo "Hello, World!" > file.txt

# 추가
echo "Second line" >> file.txt
```

**참고**: 여러 줄 파일에 `echo`를 사용하는 것은 고통스럽습니다, 그래서 AI는 코드에 거의 사용하지 않습니다. `heredoc`이 스타입니다.

---

## 4.3 `printf`: 더 강력한 형식화된 출력

### `echo`와의 비교

```bash
# printf는 C 스타일 형식 지원
printf "Value: %.2f\n" 3.14159
# 출력: Value: 3.14

printf "%s\t%s\n" "Name" "Age"
```

### 테이블 생성

```bash
printf "%-15s %10s\n" "Name" "Price"
printf "%-15s %10.2f\n" "iPhone" 999.99
printf "%-15s %10.2f\n" "MacBook" 1999.99
```

---

## 4.4 heredoc: AI의 핵심 코드 작성 무기

### heredoc이란?

heredoc은 **여러 줄 텍스트를 그대로 출력**하기 위한 특수 Shell 문법입니다.

```bash
cat << 'EOF'
All this content
will be output verbatim
including newlines, spaces
EOF
```

### 파일 쓰기 (AI의 가장 일반적인 사용)

```bash
cat > program.py << 'EOF'
#!/usr/bin/env python3

def hello(name):
    print(f"Hello, {name}!")

if __name__ == "__main__":
    hello("World")
EOF
```

### 왜 `'EOF'` (작은따옴표)를 사용하는가?

```bash
# 작은따옴표 EOF: 아무것도展開 안 함
cat << 'EOF'
HOME is: $HOME
Today is: $(date)
EOF
# 출력: HOME is: $HOME (展開 안 됨)

# 큰따옴표 EOF 또는 따옴표 없음:展開 됨
cat << EOF
HOME is: $HOME
EOF
# 출력: HOME is: /home/ai (展開됨)
```

**AI의 선택**: 거의 항상 `'EOF'` (작은따옴표)를 사용합니다. 코드는 일반적으로 Shell 변수展開이 필요하지 않기 때문입니다.

---

## 4.5 AI가 heredoc으로 다양한 파일 작성

### Python 프로그램 작성

```bash
cat > src/main.py << 'EOF'
#!/usr/bin/env python3
"""Main entry point"""

import sys
import os

def main():
    print("Python + C Hybrid Project")
    print(f"Working directory: {os.getcwd()}")

if __name__ == "__main__":
    main()
EOF
```

### Shell 스크립트 작성

```bash
cat > scripts/deploy.sh << 'EOF'
#!/bin/bash
set -e

DEPLOY_HOST="server.example.com"

echo "🚀 Deploying to $DEPLOY_HOST"
rsync -avz --delete ./dist/ "$DEPLOY_HOST:/var/www/app/"
ssh "$DEPLOY_HOST" "systemctl restart myapp"
echo "✅ Deploy complete!"
EOF

chmod +x scripts/deploy.sh
```

### 설정 파일 작성

```bash
cat > config.json << 'EOF'
{
    "site_name": "My Blog",
    "author": "Anonymous",
    "theme": "minimal"
}
EOF
```

### Dockerfile 작성

```bash
cat > Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "-m", "http.server", "8000"]
EOF
```

---

## 4.6 heredoc 함정과 해결책

### 함정 1: 작은따옴표 포함

```bash
# 문제: 작은따옴표 EOF는 작은따옴표 허용 안 함
cat << 'EOF'
He's going.
EOF
# 출력: syntax error

# 해결책: 큰따옴표 EOF 사용
cat << "EOF"
He's going.
EOF
```

### 함정 2: `$` 포함 (하지만 전개不希望)

```bash
# 문제: 큰따옴표 EOF는 $를 전개함
cat << "EOF"
Price is $100
EOF
# 출력: Price is (비어있음)

# 해결책: 개별적으로 이스케이프
cat << "EOF"
Price is $$100
EOF
# 출력: Price is $100
```

---

## 4.7 실습: 처음부터 완전한 프로젝트 구축

```bash
# 1. 디렉토리 구조 생성
mkdir -p myblog/{src,themes,content}

# 2. 설정 파일 생성
cat > myblog/config.json << 'EOF'
{
    "site_name": "My Blog",
    "author": "Anonymous",
    "theme": "minimal"
}
EOF

# 3. Python 메인 프로그램 생성
cat > myblog/src/blog.py << 'EOF'
#!/usr/bin/env python3
"""Simple blog generator"""
import json
from pathlib import Path

def load_config():
    config_path = Path(__file__).parent.parent / "config.json"
    with open(config_path) as f:
        return json.load(f)

if __name__ == "__main__":
    config = load_config()
    print(f"Generating: {config['site_name']}")
EOF

# 4. 빌드 스크립트 생성
cat > myblog/build.sh << 'EOF'
#!/bin/bash
set -e
echo "🔨 Building blog..."
cd "$(dirname "$0")"
python3 src/blog.py
echo "✅ Build complete!"
EOF

chmod +x myblog/build.sh

# 5. 구조 확인
find myblog -type f | sort
```

---

## 4.8 빠른 참조

| 도구 | 용도 | 예시 |
|------|------|------|
| `echo` | 한 줄 출력 | `echo "Hello"` |
| `echo -n` | 줄바꿈 없음 | `echo -n "Loading..."` |
| `printf` | 형식화된 출력 | `printf "%s: %d\n" "age" 25` |
| `>` | 파일 덮어쓰기 | `echo "hi" > file.txt` |
| `>>` | 파일에 추가 | `echo "hi" >> file.txt` |
| `<< 'EOF'` | heredoc (전개 안 함) | 코드에 선호됨 |
| `<< "EOF"` | heredoc (전개함) | 거의 사용 안 함 |

---

## 4.9 연습 문제

1. `echo -n`과 `for` 루프로 Loading 애니메이션 생성
2. `printf`로 형식화된 테이블 (이름, 나이, 직업) 생성
3. heredoc으로 20줄 Python 프로그램 작성
4. heredoc으로 docker-compose.yml 작성
