# README - Bài tập 04  
## Khai thác n8n để tự động đăng bài lên WordPress

**Sinh viên thực hiện:** Nguyễn Nguyệt Linh  
**Lớp:** 58KTPM  
**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421  

---

# 1. Mô tả hệ thống

Hệ thống được triển khai bằng Docker trên Ubuntu gồm 5 service:

- MariaDB: hệ quản trị cơ sở dữ liệu cho WordPress
- phpMyAdmin: quản lý database
- WordPress: website đăng bài
- n8n: workflow automation
- Cloudflared: public các service ra Internet thông qua Cloudflare Tunnel

Sau khi hoàn thành, hệ thống cho phép:

- Người dùng nhắn tin qua Telegram Bot
- Nội dung được gửi vào n8n
- Google Gemini AI sinh bài viết HTML + CSS
- n8n tự động đăng bài lên WordPress

---

# 2. Cấu trúc hệ thống

## Domain sử dụng

| Service | Domain |
|---|---|
| WordPress | https://blog.nguyetlinh.id.vn |
| phpMyAdmin | https://db.nguyetlinh.id.vn |
| n8n | https://n8n.nguyetlinh.id.vn |

---

# 3. Docker Compose

File: `docker-compose.yml`

```yaml
services:

  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      TZ: "Asia/Ho_Chi_Minh"
      MARIADB_ROOT_PASSWORD: root123
      MARIADB_DATABASE: wordpress_db
      MARIADB_USER: wp_user
      MARIADB_PASSWORD: wp_pass
    volumes:
      - mariadb_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mariadb
      PMA_ARBITRARY: 1

  wordpress:
    image: wordpress:latest
    container_name: wordpress_site
    restart: always
    ports:
      - "8000:80"
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_NAME: wordpress_db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_pass
    depends_on:
      - mariadb
    volumes:
      - wordpress_data:/var/www/html

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - TZ=Asia/Ho_Chi_Minh
      - WEBHOOK_URL=https://n8n.nguyetlinh.id.vn/
      - N8N_HOST=n8n.nguyetlinh.id.vn
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
    volumes:
      - n8n_data:/home/node/.n8n

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: always
    command: tunnel --no-autoupdate run
    environment:
      - TUNNEL_TOKEN=YOUR_TOKEN

volumes:
  mariadb_data:
  wordpress_data:
  n8n_data:
```

---

# 4. Khởi động hệ thống

## Pull image và chạy container

```bash
docker compose up -d
```

## Kiểm tra container

```bash
docker ps
```

Kết quả cần có:

- mariadb
- phpmyadmin
- wordpress_site
- n8n
- cloudflared

Trạng thái phải là:

```text
Up
```

---

# 5. Cấu hình Cloudflare Tunnel

Tạo Tunnel trên Cloudflare Zero Trust.

Cấu hình Public Hostname:

| Service | Hostname | URL |
|---|---|---|
| WordPress | blog.nguyetlinh.id.vn | wordpress:80 |
| phpMyAdmin | db.nguyetlinh.id.vn | phpmyadmin:80 |
| n8n | n8n.nguyetlinh.id.vn | n8n:5678 |

---

# 6. Cài đặt WordPress

Truy cập:

```text
https://blog.nguyetlinh.id.vn
```

Thực hiện:

- tạo tài khoản admin
- cấu hình website
- hoàn tất setup WordPress

Sau khi cài đặt, WordPress tự động tạo các bảng trong MariaDB.

---

# 7. Kiểm tra cơ sở dữ liệu

Truy cập:

```text
https://db.nguyetlinh.id.vn
```

Đăng nhập:

```text
user: root
password: root123
```

Quan sát:

- trước khi setup WordPress: database chưa có bảng
- sau khi setup: xuất hiện các bảng như:
  - wp_posts
  - wp_users
  - wp_options
  - wp_comments

---

# 8. Cấu hình Telegram Bot

Sử dụng Telegram và chat với:

```text
@BotFather
```

## Tạo bot

```text
/newbot
```

Sau khi tạo thành công sẽ nhận được:

- Bot Token

Bot được dùng để nhận nội dung người dùng gửi.

---

# 9. Cấu hình Google Gemini API

Truy cập:

```text
https://aistudio.google.com/api-keys
```

Tạo API Key để dùng cho node AI trong n8n.

---

# 10. Cấu hình n8n

Truy cập:

```text
https://n8n.nguyetlinh.id.vn
```

## Workflow gồm các node:

### 1. Telegram Trigger

- Nhận tin nhắn từ Telegram Bot

### 2. Google Gemini - Message a model

Prompt:

```text
{{ $json.message.text }}

Hãy viết bài blog chuyên nghiệp bằng HTML+CSS.
Trả về JSON theo format:

{
  "post_title": "...",
  "post_content": "..."
}
```

Bật:

```text
Output Content as JSON
```

### 3. Code JavaScript

```javascript
const rawText = $input.first().json.content.parts[0].text;

const cleanData = JSON.parse(rawText);

return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
```

### 4. WordPress - Create a Post

Cấu hình:

- WordPress URL
- Username
- Application Password

Mapping dữ liệu:

```text
Title -> {{ $json.title }}
Content -> {{ $json.content }}
Status -> Publish
```

---

# 11. Kết quả đạt được

Hệ thống hoạt động thành công:

1. Người dùng gửi tin nhắn qua Telegram
2. Telegram Trigger nhận nội dung
3. Google Gemini sinh bài viết HTML + CSS
4. JavaScript xử lý JSON
5. WordPress tự động đăng bài
6. Bài viết xuất hiện ngay trên website WordPress

---

# 12. Nhận xét

Qua bài tập này em đã hiểu:

- cách triển khai hệ thống bằng Docker Compose
- cách sử dụng Cloudflare Tunnel để public service
- cách cấu hình WordPress với MariaDB
- cách xây dựng workflow automation bằng n8n
- cách tích hợp Telegram Bot, Google Gemini AI và WordPress API

Hệ thống giúp tự động hóa quy trình tạo nội dung và đăng bài, giảm thao tác thủ công và tăng hiệu quả quản lý website.
# CÁCH THỨC HOẠT ĐỘNG
## Bước 1: Docker ps

<img width="1478" height="442" alt="image" src="https://github.com/user-attachments/assets/beb4efd4-88c6-4ceb-98a6-8a21b78c49b0" />

<img width="1902" height="892" alt="Screenshot 2026-05-23 131843" src="https://github.com/user-attachments/assets/18d7eb6c-3f12-4bd5-8d82-705ce8278a2c" />

## Bước 2: File docker-compose.yml

<img width="1459" height="1003" alt="image" src="https://github.com/user-attachments/assets/dca88c2e-82ed-4f98-90c3-054324fb7c24" />

## Bước 3: Giao diện WordPress

<img width="1902" height="960" alt="image" src="https://github.com/user-attachments/assets/82dad49a-ac4b-4a44-89c9-9c9d4473475f" />
## Bước 4: Giao diện PhpMyAdmin

<img width="1908" height="914" alt="image" src="https://github.com/user-attachments/assets/5fcf64c3-24b4-4fa3-bd18-d52b25718c19" />

## Bước 5: Workflow n8n

https://n8n.nguyetlinh.id.vn/workflow/CxDJvSFE9l7z8YmV

<img width="1889" height="970" alt="image" src="https://github.com/user-attachments/assets/61d1b5e9-e210-44fc-984a-5f5ee914e02d" />

## Bước 6: Telegram bot

<img width="1080" height="2316" alt="image" src="https://github.com/user-attachments/assets/cbcf5e79-0065-4f2a-ae85-d17bf8029e40" />

## Bước 7: Bài đăng tự động

<img width="1900" height="908" alt="Screenshot 2026-05-25 201241" src="https://github.com/user-attachments/assets/e22b9e9c-14f4-4763-8a27-a6157e2a57bd" /> 

## Bước 8: Log container hoạt động

<img width="1645" height="801" alt="image" src="https://github.com/user-attachments/assets/99de4ee1-e25e-4b8f-80a0-3b8ddbb6ac1a" />
