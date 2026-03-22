# 7. Điều Khiển Luồng và Vòng Lặp Điều Kiện

---

## 7.1 Kết Hợp Các Lệnh: Bản Chất Của Shell

Các lệnh đơn lẻ có năng lực hạn chế. Chỉ bằng cách **kết hợp** chúng, bạn mới có thể hoàn thành các công việc phức tạp.

AI mạnh mẽ phần lớn vì nó thành thạo các kết hợp này:

```bash
cat access.log | grep "ERROR" | sort | uniq -c | sort -rn | head -10
```

Điều này có nghĩa: "Từ access.log, tìm lỗi, đếm số lần xuất hiện, hiển thị top 10"

---

## 7.2 `|` (Pipe): Nghệ Thuật Luồng Dữ Liệu

Pipe biến **output** của lệnh trước thành **input** của lệnh tiếp theo.

```bash
# Sắp xếp nội dung file
cat unsorted.txt | sort

# Tìm các lệnh phổ biến nhất
history | awk '{print $2}' | sort | uniq -c | sort -rn | head -10

# Trích xuất IP từ log và đếm
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -5
```

### Pipe stderr

```bash
# Gửi stderr đến pipe
command1 2>&1 | command2

# Hoặc Bash 4+
command1 |& command2
```

---

## 7.3 `&&`: Thực Thi Tiếp Chỉ Khi Thành Công

**Chỉ khi `command1` thành công (exit code = 0) thì `command2` mới được thực thi.**

```bash
# Tạo thư mục rồi cd vào đó
mkdir -p project && cd project

# Biên dịch rồi chạy
gcc -o program source.c && ./program

# Download rồi giải nén
curl -L -o archive.tar.gz http://example.com/file && tar -xzf archive.tar.gz
```

---

## 7.4 `||`: Thực Thi Tiếp Chỉ Khi Thất Bại

**Chỉ khi `command1` thất bại (exit code ≠ 0) thì `command2` mới được thực thi.**

```bash
# Tạo file nếu không tồn tại
[ -f config.txt ] || echo "Config thiếu" > config.txt

# Thử một cách, fallback sang cách khác
cd /opt/project || cd /home/user/project

# Đảm bảo thành công ngay cả khi thất bại (phổ biến trong makefile)
cp file.txt file.txt.bak || true
```

### Kết Hợp `&&` và `||`

```bash
# Biểu thức điều kiện
[ -f config ] && echo "Tìm thấy" || echo "Không tìm thấy"

# Tương đương với:
if [ -f config ]; then
    echo "Tìm thấy"
else
    echo "Không tìm thấy"
fi
```

---

## 7.5 `;`: Thực Thi Bất Kể

```bash
# Cả ba đều được thực thi
mkdir /tmp/test ; cd /tmp/test ; pwd
```

---

## 7.6 `$()`: Thay Thế Lệnh

**Thực thi lệnh, thay `$()` bằng output của nó.**

```bash
# Cách dùng cơ bản
echo "Hôm nay là $(date +%Y-%m-%d)"
# Output: Hôm nay là 2026-03-22

# Trong biến
FILES=$(ls *.txt)

# Lấy tên thư mục
DIR=$(dirname /path/to/file.txt)
BASE=$(basename /path/to/file.txt)

# Tính toán
echo "Kết quả là $((10 + 5))"
# Output: Kết quả là 15
```

### vs Dấu Ngược

```bash
# Cả hai tương đương
echo "Hôm nay là $(date +%Y)"
echo "Hôm nay là `date +%Y`"

# Nhưng $() tốt hơn vì có thể lồng nhau
echo $(echo $(echo nested))
```

---

## 7.7 `[[ ]]` và `[ ]`: Kiểm Tra Điều Kiện

### Kiểm Tra File

```bash
[[ -f file.txt ]]      # File thường tồn tại
[[ -d directory ]]     # Thư mục tồn tại
[[ -e path ]]           # Bất kỳ loại nào tồn tại
[[ -L link ]]           # Liên kết tượng trưng tồn tại
[[ -r file ]]           # Có thể đọc
[[ -w file ]]           # Có thể ghi
[[ -x file ]]           # Có thể thực thi
[[ file1 -nt file2 ]]  # file1 mới hơn file2
```

### Kiểm Tra Chuỗi

```bash
[[ -z "$str" ]]        # Chuỗi rỗng
[[ -n "$str" ]]        # Chuỗi không rỗng
[[ "$str" == "value" ]] # Bằng nhau
[[ "$str" =~ pattern ]]  # Khớp regex
```

### Kiểm Tra Số

```bash
[[ $num -eq 10 ]]      # Bằng
[[ $num -ne 10 ]]      # Không bằng
[[ $num -gt 10 ]]      # Lớn hơn
[[ $num -lt 10 ]]      # Nhỏ hơn
```

---

## 7.8 `if`: Câu Lệnh Điều Kiện

```bash
if [[ condition ]]; then
    # làm gì đó
elif [[ condition2 ]]; then
    # làm gì khác
else
    # dự phòng
fi
```

### Ví Dụ Hoàn Chỉnh

```bash
#!/bin/bash

FILE="config.yaml"

if [[ ! -f "$FILE" ]]; then
    echo "Lỗi: $FILE không tồn tại"
    exit 1
fi

if [[ -r "$FILE" ]]; then
    echo "File có thể đọc"
else
    echo "File không thể đọc"
fi
```

---

## 7.9 `for`: Vòng Lặp

### Cú Pháp Cơ Bản

```bash
for variable in list; do
    # sử dụng $variable
done
```

### Các Pattern Phổ Biến Của AI

```bash
# Xử lý tất cả file .txt
for file in *.txt; do
    echo "Đang xử lý $file"
done

# Phạm vi số
for i in {1..10}; do
    echo "Lần lặp $i"
done

# Mảng
for color in red green blue; do
    echo $color
done

# Vòng lặp kiểu C (Bash 3+)
for ((i=0; i<10; i++)); do
    echo $i
done
```

---

## 7.10 `while`: Vòng Lặp Có Điều Kiện

```bash
# Đọc từng dòng
while IFS= read -r line; do
    echo "Đọc: $line"
done < file.txt

# Vòng lặp đếm
count=0
while [[ $count -lt 10 ]]; do
    echo $count
    ((count++))
done
```

---

## 7.11 `case`: Khớp Pattern

```bash
case $ACTION in
    start)
        echo "Đang khởi động service..."
        ;;
    stop)
        echo "Đang dừng service..."
        ;;
    restart)
        $0 stop
        $0 start
        ;;
    *)
        echo "Cách dùng: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

### Pattern Wildcard

```bash
case "$filename" in
    *.txt)
        echo "File văn bản"
        ;;
    *.jpg|*.png|*.gif)
        echo "File ảnh"
        ;;
    *)
        echo "Loại không xác định"
        ;;
esac
```

---

## 7.12 Tham Khảo Nhanh

| Ký hiệu | Tên | Mô tả |
|---------|-----|-------|
| `\|` | Pipe | Chuyển output sang input tiếp theo |
| `&&` | AND | Thực thi tiếp chỉ khi lệnh trước thành công |
| `\|\|` | OR | Thực thi tiếp chỉ khi lệnh trước thất bại |
| `;` | Dấu chấm phẩy | Thực thi bất kể |
| `$()` | Thay thế lệnh | Thực thi, thay bằng output |
| `[[ ]]` | Kiểm tra điều kiện | cú pháp kiểm tra được khuyến nghị |
| `if` | Điều kiện | rẽ nhánh dựa trên điều kiện |
| `for` | Vòng lặp đếm | lặp qua danh sách |
| `while` | Vòng lặp có điều kiện | lặp khi điều kiện đúng |
| `case` | Khớp pattern | rẽ nhánh đa hướng |

---

## 7.13 Bài Tập

1. Dùng `|` để kết hợp `ls`, `grep`, `wc` đếm các file `.log`
2. Dùng `&&` để đảm bảo `cd` thành công trước khi tiếp tục
3. Dùng vòng lặp `for` để tạo 10 thư mục (dir1 đến dir10)
4. Dùng `while read` để đọc và hiển thị /etc/hosts
5. Viết máy tính đơn giản với `case` (cộng, trừ, nhân, chia)
