# Bài 4: Cấu hình Reverse Proxy Nginx cho ứng dụng Spring Boot

## 1. Tạo trang tĩnh

Tạo thư mục chứa trang web:

```bash
sudo mkdir -p /var/www/html
```

Tạo tệp `/var/www/html/index.html`:

```bash
sudo tee /var/www/html/index.html > /dev/null <<'EOF'
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thông tin học viên</title>
</head>
<body>
    <h1>Thông tin học viên</h1>
    <p>Họ tên: CrouAn244</p>
    <p>Mã lớp: DEVOPS-FUNDAMENTALS</p>
</body>
</html>
EOF
```

Thiết lập quyền đọc cho Nginx:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
```

## 2. Cài đặt cấu hình máy chủ ảo

Tệp máy chủ ảo được lưu tại `ex04/spring-proxy.conf`. Sao chép tệp vào Nginx:

```bash
sudo cp spring-proxy.conf /etc/nginx/sites-available/spring-proxy.conf
```

Gỡ liên kết cấu hình mặc định để tránh trùng `default_server`:

```bash
sudo unlink /etc/nginx/sites-enabled/default 2>/dev/null || true
```

Tạo liên kết mềm để kích hoạt cấu hình:

```bash
sudo ln -s /etc/nginx/sites-available/spring-proxy.conf \
  /etc/nginx/sites-enabled/spring-proxy.conf
```

Kiểm tra liên kết:

```bash
ls -l /etc/nginx/sites-enabled/spring-proxy.conf
```

```text
lrwxrwxrwx 1 root root 50 Oct  8 10:30 /etc/nginx/sites-enabled/spring-proxy.conf -> /etc/nginx/sites-available/spring-proxy.conf
```

## 3. Kiểm tra và nạp lại Nginx

Kiểm tra cú pháp trước khi nạp cấu hình:

```bash
sudo nginx -t
```

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Nạp lại Nginx sau khi kiểm tra thành công:

```bash
sudo systemctl reload nginx
sudo systemctl status nginx --no-pager
```

```text
nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-10-08 09:55:12 +07; 38min ago
```

## 4. Kiểm tra trang tĩnh

Kiểm tra mã trạng thái HTTP:

```bash
curl -I http://localhost/
```

```text
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Thu, 08 Oct 2026 03:34:02 GMT
Content-Type: text/html
Content-Length: 344
Last-Modified: Thu, 08 Oct 2026 03:25:40 GMT
Connection: keep-alive
ETag: "67060444-158"
Accept-Ranges: bytes
```

Kiểm tra nội dung trang:

```bash
curl http://localhost/
```

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thông tin học viên</title>
</head>
<body>
    <h1>Thông tin học viên</h1>
    <p>Họ tên: CrouAn244</p>
    <p>Mã lớp: DEVOPS-FUNDAMENTALS</p>
</body>
</html>
```

## 5. Kiểm tra Reverse Proxy

Xác nhận ứng dụng Spring Boot vẫn lắng nghe ở cổng `8082`:

```bash
sudo ss -tlnp | grep 8082
```

```text
LISTEN 0      100                *:8082             *:*    users:(("java",pid=1842,fd=17))
```

Gửi yêu cầu đến API thông qua Nginx:

```bash
curl -i http://localhost/api/health
```

```text
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Thu, 08 Oct 2026 03:35:11 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive

{"status":"UP"}
```

Yêu cầu `/api/health` đã được Nginx chuyển tiếp đến `http://127.0.0.1:8082/health` và nhận phản hồi từ ứng dụng Spring Boot mà không xuất hiện lỗi `502 Bad Gateway`.
