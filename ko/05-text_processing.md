# 5. 텍스트 처리

---

## 5.1 AI의 텍스트 처리 철학

AI의 세계에서 **모든 것은 텍스트**입니다.

- 코드는 텍스트
- 설정 파일은 텍스트
- 로그 파일은 텍스트
- JSON, HTML, Markdown은 모두 텍스트

그래서 텍스트 처리 명령어는 AI 도구 키트의 핵심입니다.

인간 엔지니어가 문제를 만나면: "이걸 처리할 도구가 필요해..."
AI가 문제를 만나면: "이건 `grep | sed | awk`로 한 줄로 해결 가능."

---

## 5.2 `cat`: 파일 읽기의 기술

### 기본 사용법

```bash
# 파일 내용 표시
cat file.txt

# 파일 결합
cat part1.txt part2.txt > whole.txt

# 줄 번호 표시
cat -n script.sh
```

### 실제 목적: 결합과 생성

```bash
cat << 'EOF' > newfile.txt
File content
Can write many lines
EOF
```

---

## 5.3 `head`와 `tail`: 필요한 것만 보기

### `head`: 처음 보기

```bash
# 첫 10줄 (기본값)
head file.txt

# 첫 5줄
head -n 5 file.txt

# 첫 100바이트
head -c 100 file.txt
```

### `tail`: 끝 보기

```bash
# 마지막 10줄 (기본값)
tail file.txt

# 마지막 5줄
tail -n 5 file.txt

# 실시간으로 파일 따르기 (가장 일반적!)
tail -f /var/log/syslog

# 따르며 grep
tail -f app.log | grep --line-buffered ERROR
```

### 특정 줄 범위 보기

```bash
# 100-150행 보기
tail -n +100 file.txt | head -n 50
```

---

## 5.4 `wc`: 카운팅 도구

```bash
# 줄 수 세기
wc -l file.txt

# 여러 파일의 줄 수 세기
wc -l *.py

# 디렉토리의 파일 수 세기
ls | wc -l
```

---

## 5.5 `grep`: 텍스트 검색의 왕

### 기본 사용법

```bash
# "error" 행 검색
grep "error" log.txt

# 대소문자 무시
grep -i "error" log.txt

# 줄 번호 표시
grep -n "error" log.txt

# 파일 이름만 표시
grep -l "TODO" *.md

# 반전 (일치하지 않는 행)
grep -v "debug" log.txt

# 전체 단어 일치
grep -w "error" log.txt
```

### 정규 표현식

```bash
# 시작 일치
grep "^Error" log.txt

# 끝 일치
grep "done.$" log.txt

#任意の 문자
grep "e.or" log.txt

# 범위
grep -E "[0-9]{3}-" log.txt
```

### 고급 기술

```bash
# 재귀적 검색
grep -r "TODO" src/

# 특정 확장자만
grep -r "TODO" --include="*.py" src/

# 주변 줄 표시
grep -B 2 -A 2 "ERROR" log.txt

# 여러 조건 (OR)
grep -E "error|warning|fatal" log.txt
```

---

## 5.6 `sed`: 텍스트 대체 도구

### 기본 대체

```bash
# 첫 번째 일치만 대체
sed 's/old/new/' file.txt

# 모든 일치 대체
sed 's/old/new/g' file.txt

# 파일 내 대체
sed -i 's/old/new/g' file.txt

# 백업 후 대체
sed -i.bak 's/old/new/g' file.txt
```

### 행 삭제

```bash
# 빈 행 삭제
sed '/^$/d' file.txt

# 주석 행 삭제
sed '/^#/d' file.txt

# 끝 공백 삭제
sed 's/[[:space:]]*$//' file.txt
```

### 실용적인 예시

```bash
# 일괄 확장자 변경 (.txt → .md)
for f in *.txt; do
    mv "$f" "$(sed 's/.txt$/.md/' <<< "$f")"
done

# Windows 줄바꿈 제거
sed -i 's/\r$//' file.txt
```

---

## 5.7 `awk`: 텍스트 처리의 만능 칼

### 기본 개념

`awk`는 텍스트를 행별로 처리하고, 자동으로 필드로 분리합니다 ($1, $2, $3...), 각 행에 대해 지정된 작업을 실행합니다.

### 기본 사용법

```bash
# 기본 공백 분리
awk '{print $1}' file.txt

# 구분자 지정
awk -F: '{print $1}' /etc/passwd

# 여러 필드 출력
awk -F: '{print $1, $3, $7}' /etc/passwd
```

### 조건 처리

```bash
# 일치하는 행만 처리
awk -F: '$3 > 1000 {print $1}' /etc/passwd

# BEGIN과 END
awk 'BEGIN {print "Start"} {print} END {print "Done"}' file.txt
```

### 실용적인 예시

```bash
# CSV 열 합계
awk -F, '{sum += $3} END {print sum}' data.csv

# 최대값 찾기
awk 'NR==1 {max=$3} $3>max {max=$3} END {print max}' data.csv

# 형식화된 출력
awk '{printf "%-20s %10.2f\n", $1, $2}' data.txt
```

---

## 5.8 실습: 모든 도구 결합

### 시나리오: 서버 로그 분석

```bash
# 1. 오류 메시지 찾기
grep -i "error" access.log

# 2. 오류 수 세기
grep -ci "error" access.log

# 3. 가장 흔한 오류 찾기
grep "error" access.log | awk '{print $NF}' | sort | uniq -c | sort -rn | head

# 4. 시간당 요청 수
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c
```

### 시나리오: 코드 일괄 수정

```bash
# 모든 .py 파일에서 "print"를 "logger.info"로 변경
find . -name "*.py" -exec sed -i 's/print(/logger.info(/g' {} +

# var를 const로 변경
find . -name "*.js" -exec sed -i 's/\bvar\b/const/g' {} +
```

---

## 5.9 빠른 참조

| 명령 | 용도 | 일반적인 플래그 |
|------|------|----------------|
| `cat` | 파일 표시/결합 | `-n` 줄 번호 |
| `head` | 파일 처음 보기 | `-n` 줄, `-c` 바이트 |
| `tail` | 파일 끝 보기 | `-n` 줄, `-f` 따르기 |
| `wc` | 카운팅 | `-l` 줄, `-w` 단어, `-c` 바이트 |
| `grep` | 텍스트 검색 | `-i` 무시, `-n` 줄 번호, `-r` 재귀, `-c` 카운트 |
| `sed` | 텍스트 대체 | `s/old/new/g`, `-i` 파일 내 |
| `awk` | 필드 처리 | `-F` 구분자, `{print}` 작업 |

---

## 5.10 연습 문제

1. `head`와 `tail`을 사용하여 100-120행 보기
2. `grep`을 사용하여 /etc/passwd에서 `/bin/bash`가 있는 모든 사용자 찾기
3. `sed`를 사용하여 모든 `\r\n`을 `\n`으로 대체
4. `awk`를 사용하여 숫자 파일의 최대, 최소, 평균 계산
