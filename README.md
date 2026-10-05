# Bài tập 3: Cài đặt Nginx và Cấu hình Website Tĩnh cơ bản (Basic Nginx Web Server)

## Thông tin sinh viên
- **Họ và tên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 02 - Bài tập 3

---

## Tệp tin đính kèm
- [ptit-web.conf](./ptit-web.conf): File cấu hình Server Block Nginx lắng nghe Port 80.
- [index.html](./index.html): Trang HTML tĩnh hiển thị thông điệp chào mừng PTIT.
- [IT209_Session02_BT3_PhamThanhPhong_N24DTCN120.md](./IT209_Session02_BT3_PhamThanhPhong_N24DTCN120.md): Báo cáo chi tiết từng bước thực hiện và kiểm tra.

---

## Hướng dẫn triển khai nhanh trên Ubuntu Server
```bash
# 1. Cài đặt Nginx
sudo apt update && sudo apt install -y nginx

# 2. Tạo web root và copy file index.html
sudo mkdir -p /var/www/ptit-web/html
sudo cp index.html /var/www/ptit-web/html/

# 3. Copy file cấu hình Nginx
sudo cp ptit-web.conf /etc/nginx/sites-available/
sudo ln -s /etc/nginx/sites-available/ptit-web.conf /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default

# 4. Kiểm tra cấu hình và reload dịch vụ
sudo nginx -t
sudo systemctl reload nginx
```
Truy cập: `http://<IP_ADDRESS_DROPLET>`
