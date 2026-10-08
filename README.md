# Bài 3: Thiết lập Cơ sở dữ liệu và tự cấu hình dịch vụ Systemd cho Spring Boot

## 1. Mục tiêu

- Thiết lập cơ sở dữ liệu MySQL cho ứng dụng Spring Boot.
- Tạo user Linux riêng để chạy ứng dụng.
- Tự viết file Systemd `spring-app.service`.
- Cấu hình ứng dụng chạy bằng user `spring-runner`.
- Cấu hình tự động restart khi ứng dụng gặp lỗi.
- Kiểm tra trạng thái service và port 8082.

---

## 2. Thiết lập cơ sở dữ liệu MySQL

### Tạo database

```sql
CREATE DATABASE springboot_db;
```

### Tạo tài khoản MySQL

```sql
CREATE USER 'spring-admin'@'localhost' IDENTIFIED BY 'SpringSecure@123';
```

### Cấp quyền

```sql
GRANT ALL PRIVILEGES ON springboot_db.* TO 'spring-admin'@'localhost';
FLUSH PRIVILEGES;
```

### Kiểm tra quyền

```sql
SHOW GRANTS FOR 'spring-admin'@'localhost';
```

---

## 3. Tạo Linux user

Tạo user hệ thống `spring-runner`:

```bash
useradd --system --no-create-home --shell /usr/sbin/nologin spring-runner
```

Kiểm tra:

```bash
id spring-runner
```

User này được sử dụng để chạy ứng dụng Spring Boot và không có shell đăng nhập thông thường.

---

## 4. Tạo thư mục ứng dụng

```bash
mkdir -p /opt/spring-app
chown spring-runner:spring-runner /opt/spring-app
```

Kiểm tra:

```bash
ls -ld /opt/spring-app
```

---

## 5. Cấu hình Systemd

Tạo file:

```bash
nano /etc/systemd/system/spring-app.service
```

Nội dung file:

```ini
[Unit]
Description=Spring Boot Application
After=network.target mysql.service
Requires=mysql.service

[Service]
User=spring-runner
Group=spring-runner
WorkingDirectory=/opt/spring-app
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Giải thích

| Cấu hình | Ý nghĩa |
|---|---|
| `User=spring-runner` | Chạy ứng dụng bằng user `spring-runner` |
| `Group=spring-runner` | Sử dụng group `spring-runner` |
| `WorkingDirectory=/opt/spring-app` | Thư mục làm việc |
| `ExecStart=...` | Lệnh chạy file Spring Boot |
| `Restart=on-failure` | Tự restart khi ứng dụng lỗi |
| `RestartSec=10` | Chờ 10 giây trước khi restart |
| `After=network.target mysql.service` | Chạy sau network và MySQL |
| `Requires=mysql.service` | Phụ thuộc vào MySQL |

---

## 6. Nạp và kích hoạt service

Sau khi tạo file:

```bash
systemctl daemon-reload
```

Kích hoạt service khi hệ thống khởi động:

```bash
systemctl enable spring-app.service
```

---

# 7. Kiểm tra Systemd service

Lệnh kiểm tra:

```bash
systemctl status spring-app.service --no-pager
```

## Kết quả tham khảo

```text
● spring-app.service - Spring Boot Application
     Loaded: loaded (/etc/systemd/system/spring-app.service; enabled)
     Active: active (running)
   Main PID: 12345 (java)
      Tasks: 20
     Memory: 150.0M
        CPU: 2.100s
     CGroup: /system.slice/spring-app.service
             └─12345 /usr/bin/java -jar /opt/spring-app/app.jar
```

### Ý nghĩa kết quả mong đợi

- `Loaded: loaded`: Systemd đã nhận file service.
- `enabled`: service được thiết lập tự khởi động cùng hệ thống.
- `Active: active (running)`: service đang hoạt động.
- Process Java chạy bằng file `/opt/spring-app/app.jar`.

---

# 8. Kiểm tra port 8082

Lệnh:

```bash
ss -tlnp | grep 8082
```

```text
LISTEN 0      128        0.0.0.0:8082       0.0.0.0:*    users:(("java",pid=12345,fd=123))
```

### Ý nghĩa

- `LISTEN`: đang có chương trình lắng nghe kết nối.
- `0.0.0.0:8082`: ứng dụng đang lắng nghe trên port 8082.
- `java`: tiến trình Java của ứng dụng Spring Boot.

---

# 9. Các lệnh kiểm tra đã sử dụng

### Kiểm tra MySQL

```bash
systemctl status mysql --no-pager
```

Kết quả thực tế đã xác nhận MySQL:

```text
Active: active (running)
Status: "Server is operational"
```

### Kiểm tra Linux user

```bash
id spring-runner
```

### Kiểm tra thư mục ứng dụng

```bash
ls -ld /opt/spring-app
```

### Kiểm tra service

```bash
systemctl status spring-app.service --no-pager
```

### Kiểm tra port

```bash
ss -tlnp | grep 8082
```

---

# 10. Kết luận

Đã thực hiện các nội dung cấu hình:

- [x] Cài đặt MySQL.
- [x] Tạo database `springboot_db`.
- [x] Tạo user MySQL `spring-admin`.
- [x] Cấp quyền cho `spring-admin`.
- [x] Tạo Linux user `spring-runner`.
- [x] Tạo thư mục `/opt/spring-app`.
- [x] Tạo file `/etc/systemd/system/spring-app.service`.
- [x] Cấu hình chạy ứng dụng bằng `spring-runner`.
- [x] Cấu hình `Restart=on-failure`.
- [x] Cấu hình `RestartSec=10`.
- [x] Nạp cấu hình bằng `systemctl daemon-reload`.
- [x] Enable service bằng `systemctl enable`.
- [ ] Bổ sung kết quả thực tế của `systemctl status spring-app.service`.
- [ ] Bổ sung kết quả thực tế của `ss -tlnp | grep 8082`.

---

## 11. Tệp nộp

```text
spring-app.service
README.md
```

File Systemd:

```text
/etc/systemd/system/spring-app.service
```

