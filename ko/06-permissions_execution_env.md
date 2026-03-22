# 6. 권한, 실행, 환경 및 설정

---

## 6.1 `chmod`: 권한의 기술

### Linux/Unix 권한 기본

```
drwxr-xr-x  3 ai  staff   96 Mar 22 10:30 .
-rw-r--r--  1 ai  staff  4096 Mar 22 10:30 README.md
-rwxr-xr-x  1 ai  staff   128 Mar 22 10:30 script.sh
```

9개의 문자는 세 그룹으로 나뉩니다:
- `rwx` (소유자): 읽기, 쓰기, 실행
- `r-x` (그룹): 읽기, 실행
- `r--` (기타): 읽기 전용

### chmod 두 가지 표현 방식

**숫자 (8진수)**:

```
r = 4, w = 2, x = 1

rwx = 7, r-x = 5, r-- = 4
```

일반적인 조합:
- `777` = rwxrwxrwx (위험!)
- `755` = rwxr-xr-x
- `644` = rw-r--r--
- `700` = rwx------
- `600` = rw-------

**기호**:

```bash
chmod u+x script.sh    # 소유자가 실행 추가
chmod g-w file.txt    # 그룹이 쓰기 제거
chmod +x script.sh    # 모두 실행 추가
```

### AI의 일반적인 chmod 사용

```bash
# 스크립트 실행 가능하게 만들기 (거의 모든 스크립트)
chmod +x script.sh

# 디렉토리 통과 가능하게 만들기
chmod +x ~/projects

# 그룹 쓰기 가능 디렉토리
chmod -R g+w project/
```

---

## 6.2 Shell 스크립트 실행

### 실행 방법

```bash
# 방법 1: 경로로 실행 (실행 권한 필요)
./script.sh

# 방법 2: bash 사용 (실행 권한 불필요)
bash script.sh

# 방법 3: source 사용 (현재 shell에서 실행)
source script.sh
```

### 언제 무엇을 사용하는가?

| 방법 | 언제 사용 | 특징 |
|------|----------|------|
| `./script.sh` | 표준 실행 | `chmod +x` 필요, 서브셸 |
| `bash script.sh` | 셸 지정 | 실행 권한 불필요 |
| `source script.sh` | 환경 설정 | 현재 셸에서 실행 |

### `source` vs `./script` 핵심 차이

```bash
# script.sh 내용: export MY_VAR="hello"

# ./로 실행
./script.sh
echo $MY_VAR  # 출력: (비어있음) ← 서브셸에서 변수 사라짐

# source로 실행
source script.sh
echo $MY_VAR  # 출력: hello ← 현재 셸에서 변수 유지
```

---

## 6.3 `export`: 환경 변수

```bash
# 환경 변수 설정
export NAME="Alice"
export PATH="$PATH:/new/directory"

# 모든 환경 변수 표시
export

# 일반적인 변수들
echo $HOME      # 홈 디렉토리
echo $USER      # 사용자 이름
echo $PATH      # 검색 경로
echo $PWD       # 현재 디렉토리
```

### 환경 변수 지속시키기

```bash
# ~/.bashrc에 추가
echo 'export EDITOR=vim' >> ~/.bashrc

# 변경 사항 적용
source ~/.bashrc
```

---

## 6.4 `source`: 파일 로드

동일한 의미: 파일의 내용을 현재 위치에 **붙여넣고** 실행.

### 일반적인 사용

```bash
# 가상 환경 로드
source venv/bin/activate

# .env 파일 로드
source .env

# 라이브러리 로드
source ~/scripts/common.sh
```

### 실용적 예: 모듈식 설정 파일

```bash
# config.sh
export DB_HOST="localhost"
export DB_PORT="5432"

# 다른 스크립트에서 사용
source config.sh
psql -h $DB_HOST -p $DB_PORT
```

---

## 6.5 `env`: 환경 관리

```bash
# 모든 환경 변수 표시
env

# 깨끗한 환경으로 실행
env -i HOME=/tmp PATH=/bin sh

# 변수 설정 후 실행
env VAR1=value1 VAR2=value2 ./my_program
```

### 명령 찾기

```bash
which python      # 명령 위치 찾기
type cd           # 셸 내장 명령 찾기
whereis gcc       # 모든 관련 파일 찾기
```

---

## 6.6 `sudo`: 권한 상승

```bash
# root로 실행
sudo rm /var/log/old.log

# 특정 사용자로 실행
sudo -u postgres psql

# 할 수 있는 것 표시
sudo -l
```

### 위험 경고

```bash
# 절대 이것을 실행하지 마세요!
sudo rm -rf /

# 이것도 절대 하지 마세요!
sudo curl http://unknown-site.com | sh
```

---

## 6.7 빠른 참조

| 명령 | 용도 | 일반적인 플래그 |
|------|------|----------------|
| `chmod` | 파일 권한 변경 | `+x` 실행 추가, `755` 8진수, `-R` 재귀 |
| `chown` | 소유자 변경 | `user:group`, `-R` 재귀 |
| `./script` | 스크립트 실행 (x 필요) | - |
| `bash script` | 스크립트 실행 (x 불필요) | - |
| `source` | 현재 셸에서 실행 | - |
| `export` | 환경 변수 설정 | `-n` 제거 |
| `env` | 환경 표시/관리 | `-i` 초기화 |
| `sudo` | root로 실행 | `-u user` 사용자 지정 |

---

## 6.8 연습 문제

1. 스크립트를 생성하고 `chmod 755`로 권한을 설정한 다음 실행하세요
2. 동일한 환경 변수 스크립트를 `source`와 `./`로 실행하여 차이를 관찰하세요
3. `env -i`로 깨끗한 환경을 생성하고 `python --version`을 실행하세요
4. `.env` 파일을 생성하고 `source`로 변수를 로드하세요
