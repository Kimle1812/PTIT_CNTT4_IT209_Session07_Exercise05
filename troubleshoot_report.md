# Bài 5: Chẩn đoán và xử lý xung đột cổng mạng

## 1. Mục tiêu

Xử lý lỗi `Address already in use` xảy ra khi cổng 8082 đang bị một tiến trình khác chiếm dụng.

## 2. Giả lập tiến trình chiếm cổng

Chạy lệnh:

```bash
python3 -m http.server 8082 > /dev/null 2>&1 &
```

Kiểm tra tiến trình:

```bash
ps aux | grep '[p]ython3'
```

Kết quả xác định được tiến trình Python đang chạy HTTP Server trên port 8082.

## 3. Chẩn đoán cổng 8082

Sử dụng:

```bash
sudo ss -tlnp | grep 8082
```

Kết quả:

```text
LISTEN ... 0.0.0.0:8082 ... users:(("python3",pid=XXXXX,fd=3))
```

PID của tiến trình chiếm cổng là:

```text
XXXXX
```

Có thể kiểm tra thêm bằng:

```bash
sudo lsof -i :8082
```

## 4. Kiểm tra thông tin tiến trình

```bash
ps -p XXXXX -f
```

Kết quả cho thấy tiến trình `python3 -m http.server 8082` đang chiếm cổng 8082.

## 5. Giải phóng cổng

Thực hiện:

```bash
kill XXXXX
```

Kiểm tra lại:

```bash
sudo ss -tlnp | grep 8082
```

Nếu tiến trình chưa kết thúc thì sử dụng:

```bash
kill -9 XXXXX
```

Sau đó kiểm tra lại cổng:

```bash
sudo ss -tlnp | grep 8082
```

Cổng 8082 đã được giải phóng.

## 6. Khởi động lại Spring Boot

Khởi động lại service:

```bash
sudo systemctl restart spring-app.service
```

Kiểm tra:

```bash
sudo systemctl status spring-app.service
```

Kết quả:

```text
Active: active (running)
```

## 7. Kiểm tra port sau khi xử lý

```bash
sudo ss -tlnp | grep 8082
```

Kết quả cho thấy:

```text
LISTEN ... 0.0.0.0:8082 ... users:(("java",pid=XXXXX,fd=...))
```

Cổng 8082 hiện do tiến trình Java của Spring Boot quản lý.

Kiểm tra tiến trình:

```bash
ps aux | grep '[j]ava'
```

Kết quả cho thấy ứng dụng chạy bằng:

```text
spring-runner
```

và thực thi:

```text
/usr/bin/java -jar /opt/spring-app/app.jar
```

## 8. Kết luận

Nguyên nhân lỗi `Address already in use` là do tiến trình `python3` chiếm cổng 8082.

Sau khi xác định PID bằng `ss`/`lsof`, tiến trình được kết thúc bằng `kill`. Cổng 8082 được giải phóng và service `spring-app.service` được khởi động lại thành công.

Cuối cùng, cổng 8082 được tiến trình Java của Spring Boot lắng nghe dưới quyền user `spring-runner`.
