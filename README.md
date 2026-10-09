# Bài 4: Cấu hình Reverse Proxy Nginx cho ứng dụng Spring Boot

## Mục tiêu
- Cấu hình máy chủ ảo Nginx Server Block làm Reverse Proxy định tuyến lưu lượng mạng.
- Sử dụng Path Matching để phân chia lưu lượng tĩnh phục vụ trực tiếp và lưu lượng động chuyển tiếp đến API backend Spring Boot.

## Cấu trúc thư mục hiện tại
- `index.html`: File HTML chứa thông tin cá nhân của học viên, sẽ được host dưới dạng nội dung tĩnh bằng Nginx.
- `spring-proxy.conf`: File cấu hình Server Block của Nginx. Dùng để tạo định tuyến (Routing).

## Yêu cầu Bối cảnh
- Phục vụ người dùng truy cập trang chủ tĩnh tại địa chỉ IP của máy chủ qua cổng `80` mặc định (đường dẫn `/`).
- Yêu cầu API bắt đầu bằng tiền tố `/api/` sẽ được chuyển tiếp (Reverse Proxy) đến ứng dụng Spring Boot đang chạy ở cổng `8082`.

---

## Các Bước Triển Khai Trên Máy Chủ Linux (Nginx)

### Bước 1: Khởi tạo trang tĩnh
Tạo cấu trúc thư mục chứa trang tĩnh và sao chép nội dung tệp HTML:
```bash
sudo mkdir -p /var/www/html/
sudo nano /var/www/html/index.html
# Dán nội dung của tệp index.html vào đây
```

### Bước 2: Tạo tệp cấu hình máy chủ ảo Nginx
Tạo tệp cấu hình mới:
```bash
sudo nano /etc/nginx/sites-available/spring-proxy.conf
```
Dán nội dung từ tệp `spring-proxy.conf` có trong dự án này vào.

### Bước 3: Kích hoạt cấu hình (Symlink)
Tạo liên kết mềm từ `sites-available` sang `sites-enabled`:
```bash
sudo ln -s /etc/nginx/sites-available/spring-proxy.conf /etc/nginx/sites-enabled/
```
*(Nếu cần, hãy xoá cấu hình default cũ bằng `sudo rm /etc/nginx/sites-enabled/default` để tránh xung đột cổng 80).*

### Bước 4: Kiểm tra và Reload Nginx
Trước khi tải lại dịch vụ, cần đảm bảo tệp cấu hình không có lỗi:
```bash
sudo nginx -t
```
Kết quả mong đợi:
```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Sau đó tải lại Nginx:
```bash
sudo systemctl reload nginx
```

---

## Kiểm Tra Kết Quả

**1. Kiểm tra trang web tĩnh**
```bash
curl -i http://localhost/
```
*Kết quả:* Trả về mã trạng thái `HTTP/1.1 200 OK` kèm theo nội dung trang HTML tĩnh.

**2. Kiểm tra Reverse Proxy đến ứng dụng Spring Boot**
*(Yêu cầu: Ứng dụng Spring Boot phải đang chạy tại cổng `8082`)*
```bash
curl -I http://localhost/api/health
```
*Kết quả:* Trả về mã trạng thái `200 OK` hoặc mã tương ứng từ Spring Boot backend, không bị lỗi `502 Bad Gateway`.
