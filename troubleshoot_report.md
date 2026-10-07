# EX5 - Troubleshoot Port 8082

## 1. Mục tiêu

- Chẩn đoán tiến trình đang chiếm dụng cổng 8082.
- Xác định PID của tiến trình.
- Sử dụng lệnh `kill` để giải phóng cổng.
- Khởi động lại dịch vụ `spring-app.service`.
- Xác nhận cổng 8082 được chuyển sang cho ứng dụng Spring Boot quản lý.

---

## 2. Giả lập tiến trình chiếm cổng 8082

Chạy Python HTTP Server dưới nền:

```bash
python3 -m http.server 8082 > /dev/null 2>&1 &
```

Kiểm tra tiến trình:

```bash
sudo ss -tlnp | grep 8082
```

Kết quả:

```text
LISTEN 0 5 0.0.0.0:8082 0.0.0.0:* users:(("python3",pid=135099,fd=3))
```

Thông tin tiến trình:

- Port: `8082`
- Process: `python3`
- PID: 135099

---

## 3. Kiểm tra thông tin tiến trình

Sử dụng `ps` để kiểm tra chi tiết:

```bash
ps -fp 135099
```

Kết quả:

```text
UID          PID    PPID  C STIME TTY          TIME CMD
root       135099       1  0 22:00 ?        00:00:00 python3 -m http.server 8082
```

Qua kết quả trên có thể xác định tiến trình `python3` đang chạy HTTP Server và chiếm cổng `8082`.

---

## 4. Kết thúc tiến trình đang chiếm cổng

Sử dụng lệnh `kill` để dừng tiến trình:

```bash
kill 135099
```

Kiểm tra lại cổng:

```bash
sudo ss -tlnp | grep 8082
```

Nếu không có kết quả trả về, cổng `8082` đã được giải phóng.

Trong trường hợp tiến trình không dừng bằng `kill`, có thể sử dụng:

```bash
kill -9 12345
```

`kill -9` chỉ được sử dụng khi tiến trình không phản hồi với lệnh `kill` thông thường.

---

## 5. Khởi động lại Spring Boot

Sau khi giải phóng cổng 8082, khởi động lại dịch vụ:

```bash
sudo systemctl restart spring-app.service
```

Kiểm tra trạng thái:

```bash
sudo systemctl status spring-app.service
```

Kết quả mong đợi:

```text
Active: active (running)
```

---

## 6. Kiểm tra cổng 8082 sau khi khởi động Spring Boot

Sử dụng:

```bash
sudo ss -tlnp | grep 8082
```

Kết quả mong đợi:

```text
LISTEN 0 100 0.0.0.0:8082 0.0.0.0:* users:(("java",pid=15678,fd=123))
```

Có thể kiểm tra thêm bằng:

```bash
sudo lsof -i :8082
```

Kết quả cho thấy tiến trình `java` đang lắng nghe trên cổng `8082`.

---

## 7. Kết luận

Ban đầu, cổng `8082` bị tiến trình `python3` chiếm dụng nên khi Spring Boot\
khởi động có thể xảy ra lỗi:

```text
Address already in use
```

Sau khi sử dụng `ss` để xác định tiến trình và PID, tiến trình `python3`\
được kết thúc bằng lệnh:

```bash
kill 12345
```

Cổng `8082` được giải phóng thành công. Sau đó dịch vụ:

```bash
sudo systemctl restart spring-app.service
```

được khởi động lại và tiến trình `java` đã tiếp quản cổng `8082`.

Qua bài thực hành, đã thực hiện được quy trình chẩn đoán và xử lý lỗi\
xung đột cổng mạng trên Linux.
