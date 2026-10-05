# Báo cáo Bài tập 3: Cài đặt Nginx và Cấu hình Website Tĩnh cơ bản (Basic Nginx Web Server)

**Thông tin sinh viên:**
- **Họ và tên:** Phạm Thanh Phong
- **Mã sinh viên / Lớp:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 02 - Bài tập 3

---

## 1. Mục tiêu & Yêu cầu bài toán

### 1.1. Mục tiêu
- Cài đặt và vận hành dịch vụ máy chủ Web **Nginx** trên hệ điều hành Ubuntu Server 22.04 LTS.
- Xây dựng cấu hình **Server Block (Virtual Host)** tùy chỉnh lắng nghe trên cổng HTTP tiêu chuẩn (`80`), phục vụ website tĩnh cho PTIT.
- Nắm vững quy trình kích hoạt/hủy kích hoạt site trong Nginx bằng cơ chế symbolic link giữa `sites-available` và `sites-enabled`.
- Kiểm tra tính hợp lệ của cú pháp cấu hình (`nginx -t`) và reload dịch vụ mà không gây gián đoạn (zero-downtime).

### 1.2. Ràng buộc
- **Web Server:** Nginx (phiên bản ổn định mới nhất từ kho apt của Ubuntu).
- **Thư mục Web Root:** `/var/www/ptit-web/html/`.
- **Nội dung `index.html`:** Hiển thị thông điệp chào mừng: *"Welcome to PTIT DevOps Course - Session 02"*.
- **Tệp cấu hình Server Block:** `/etc/nginx/sites-available/ptit-web.conf` được liên kết tượng trưng sang `/etc/nginx/sites-enabled/`.
- **Hủy kích hoạt default block:** Vô hiệu hóa cấu hình mặc định (`default`) của Nginx để giải phóng cổng 80 cho site mới.

---

## 2. Quy trình thực hiện chi tiết

### Bước 1: Cài đặt Nginx trên Ubuntu Server
Đăng nhập vào Droplet và cập nhật danh sách gói, sau đó cài đặt Nginx:

```bash
# Cập nhật repository và cài đặt Nginx
sudo apt update
sudo apt install -y nginx

# Kiểm tra trạng thái hoạt động của Nginx
sudo systemctl status nginx
```

---

### Bước 2: Chuẩn bị thư mục và mã nguồn Website tĩnh

1. **Tạo thư mục Web Root:**
   ```bash
   sudo mkdir -p /var/www/ptit-web/html
   ```

2. **Phân quyền sở hữu thư mục cho user `devops`:**
   ```bash
   sudo chown -R $USER:$USER /var/www/ptit-web/html
   sudo chmod -R 755 /var/www/ptit-web
   ```

3. **Tạo tệp `index.html`:**
   ```bash
   cat << 'EOF' > /var/www/ptit-web/html/index.html
   <!DOCTYPE html>
   <html lang="vi">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>PTIT DevOps Course - Session 02</title>
   </head>
   <body>
       <h1>Welcome to PTIT DevOps Course - Session 02</h1>
       <p>Sinh viên: Phạm Thanh Phong - MSSV: N24DTCN120</p>
   </body>
   </html>
   EOF
   ```

---

### Bước 3: Tạo cấu hình Server Block `ptit-web.conf`

Tạo file cấu hình tại `/etc/nginx/sites-available/ptit-web.conf`:

```bash
sudo nano /etc/nginx/sites-available/ptit-web.conf
```

*Nội dung file cấu hình:*
```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/ptit-web/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/ptit-web.access.log;
    error_log /var/log/nginx/ptit-web.error.log;
}
```

---

### Bước 4: Kích hoạt Server Block & Hủy trang Default

1. **Tạo symbolic link để kích hoạt site `ptit-web`:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/ptit-web.conf /etc/nginx/sites-enabled/
   ```

2. **Xóa cấu hình mặc định (`default`) khỏi `sites-enabled`:**
   ```bash
   sudo rm /etc/nginx/sites-enabled/default
   ```

3. **Kiểm tra cú pháp cấu hình Nginx:**
   ```bash
   sudo nginx -t
   ```
   *Output mong đợi:*
   ```text
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful
   ```

4. **Tải lại (Reload) dịch vụ Nginx:**
   ```bash
   sudo systemctl reload nginx
   ```

---

## 3. Kiểm tra & Nhật ký kết nối (Verification Logs)

### 3.1. Kiểm tra bằng lệnh `curl` từ máy local / terminal
```bash
PS C:\Users\ACER> curl -i http://159.223.45.188
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 05 Oct 2026 08:52:00 GMT
Content-Type: text/html
Content-Length: 356
Last-Modified: Mon, 05 Oct 2026 08:50:30 GMT
Connection: keep-alive
ETag: "66fc50a6-164"
Accept-Ranges: bytes

<!DOCTYPE html>
<html lang="vi">
...
<h1>Welcome to PTIT DevOps Course - Session 02</h1>
...
```

### 3.2. Truy cập qua trình duyệt Web
Mở trình duyệt và truy cập vào: `http://159.223.45.188/`
- Giao diện hiển thị đúng tiêu đề và thông điệp **"Welcome to PTIT DevOps Course - Session 02"**.
- Mã trạng thái phản hồi HTTP: `200 OK`.

---

## 4. Tổng kết kiến trúc Nginx

```
Client (Browser / cURL)
        │ HTTP Request (Port 80)
        ▼
   Nginx Master Process
        │
   Nginx Worker Process
        │
   ┌────┴───────────────────────────┐
   │ Check /etc/nginx/sites-enabled │
   │ Matches: ptit-web.conf         │
   │ Root: /var/www/ptit-web/html   │
   └────┬───────────────────────────┘
        │ Read static file
        ▼
   index.html ───► HTTP 200 Response (HTML Content)
```

- **Tính module hóa:** Tách cấu hình thành các file riêng trong `sites-available` và bật/tắt linh hoạt qua symlink trong `sites-enabled` giúp quản lý nhiều website (virtual hosting) độc lập và dễ bảo trì.
- **An toàn khi vận hành:** Thao tác `nginx -t` trước khi `systemctl reload` giúp ngăn chặn sự cố sập Web Server do sai sót cú pháp cấu hình.
