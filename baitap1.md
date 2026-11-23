# -Môn học : Phát triển ứng dụng trên nền wed

# -Họ và tên: Ngô Thị Hiền 

# -Mã sinh viên : K225480106014

# Bài tập 1:
1. giả lập linux os: hyperV, virtualBox, vmware, wsl
2. cài đặt docker compose trên os đó
3. cài trên docker compose : các dịch vụ: nginx, nodered, mariadb, phpmyadmin, cloudflared (cần domain xịn)
4. cấu hình nginx có thể chạy 2 website  với 2 domain khác nhau
   # Bài Làm
# 1. Giả lập Linux OS: Hyper-V, VirtualBox, VMware, WSL

-Mục đích: Tạo môi trường Linux (thường là Ubuntu) ngay trên máy Windows/macOS để chạy Docker và triển khai hệ thống web.

-WSL (đặc biệt WSL2): Chạy nhân Linux thật ngay trên Windows với tốc độ nhanh, nhẹ, không cần cấp phát một máy ảo nặng nề.

-VirtualBox / VMware / Hyper-V: Tạo máy ảo độc lập có phần cứng ảo riêng (CPU, RAM, mạng), cài hệ điều hành Ubuntu đầy đủ.
  
   Code / Lệnh thực hiện
   
   ```bash
wsl --install -d Ubuntu
```
Sau khi khởi động lại máy và vào terminal Ubuntu, kiểm tra hệ điều hành:

```bash
uname -a
```
<img width="581" height="332" alt="image" src="https://github.com/user-attachments/assets/dd669ec4-8a5a-4636-a257-a370f58d56f2" />

# 2. Cài đặt Docker Compose trên OS Linux

-Docker Engine: Chạy các container cô lập, chia sẻ kernel với host Linux nên nhẹ và khởi động nhanh.

-Docker Compose: Cho phép khai báo nhiều container, volume lưu dữ liệu và mạng nội bộ trong một file duy nhất (docker-compose.yml), quản lý bằng lệnh docker compose.

Code / Lệnh thực hiện
Chạy trên terminal Ubuntu/Linux:

```bash
sudo apt update
sudo apt install -y curl ca-certificates gnupg
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
```
Kiểm tra cài đặt:

```bash
docker --version
docker compose version
```
<img width="574" height="338" alt="image" src="https://github.com/user-attachments/assets/f46deaad-b0a5-4908-a8d3-f2ad986f4548" />

# 3. Cài các dịch vụ trên Docker Compose (nginx, nodered, mariadb, phpmyadmin, cloudflared)

-nginx: Web server & Reverse Proxy, nhận traffic cổng 80 (hoặc 443) và điều hướng tới đúng website/dịch vụ.

-nodered: Công cụ lập trình luồng (flow) trên nền Node.js, chạy cổng nội bộ 1880.

-mariadb: Hệ quản trị cơ sở dữ liệu quan hệ, dùng volume để giữ dữ liệu khi restart/xóa container.

-phpmyadmin: Giao diện web quản lý cơ sở dữ liệu MariaDB, kết nối tới host mariadb trong cùng Docker network.

-cloudflared: Chạy Cloudflare Tunnel giúp đưa dịch vụ lên domain thật qua mạng Cloudflare mà không cần mở port modem.

Code / File cấu hình
Tạo thư mục dự án:

```bash
mkdir -p bai1/nginx/conf.d bai1/site1 bai1/site2
cd bai1
```
Tạo file docker-compose.yml (thay YOUR_CLOUDFLARE_TUNNEL_TOKEN bằng token của bạn trên Cloudflare Zero Trust):

```bash
YAML
services:
  nginx:
    image: nginx:alpine
    container_name: bai1_nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./site1:/usr/share/nginx/html/site1:ro
      - ./site2:/usr/share/nginx/html/site2:ro
    networks:
      - bai1_net

  nodered:
    image: nodered/node-red:latest
    container_name: bai1_nodered
    restart: unless-stopped
    ports:
      - "1880:1880"
    volumes:
      - nodered_data:/data
    networks:
      - bai1_net

  mariadb:
    image: mariadb:11
    container_name: bai1_mariadb
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: rootpass123
      MARIADB_DATABASE: bai1_db
      MARIADB_USER: bai1_user
      MARIADB_PASSWORD: userpass123
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - bai1_net

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: bai1_phpmyadmin
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mariadb
      PMA_PORT: 3306
    depends_on:
      - mariadb
    networks:
      - bai1_net

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: bai1_cloudflared
    restart: unless-stopped
    command: tunnel --no-autoupdate run --token ${CLOUDFLARE_TUNNEL_TOKEN:-YOUR_CLOUDFLARE_TUNNEL_TOKEN}
    networks:
      - bai1_net

volumes:
  nodered_data:
  mariadb_data:

networks:
  bai1_net:
    driver: bridge
```
<img width="569" height="340" alt="image" src="https://github.com/user-attachments/assets/82e6f34a-5d06-4147-bf08-cba8f6e911cd" />

# 4. Cấu hình Nginx chạy 2 website với 2 domain khác nhau

-Khi 2 domain cùng trỏ về máy chủ Nginx, Nginx dựa vào HTTP header Host và khai báo server_name trong từng khối server để phân biệt.

-Mỗi khối server có thể trỏ tới thư mục HTML (root) hoặc dịch vụ riêng biệt.

# Code / File cấu hình
# Tạo nội dung cho site1/index.html:

```bash
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <title>Domain 1</title>
</head>
<body>
  <h1>Xin chào từ Website 1 (Domain 1)</h1>
</body>
</html>
```
<img width="578" height="283" alt="image" src="https://github.com/user-attachments/assets/75cf24bb-89a5-4510-a5d2-c9dc6855b5e3" />

Tạo nội dung cho site2/index.html:

```bash
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <title>Domain 2</title>
</head>
<body>
  <h1>Xin chào từ Website 2 (Domain 2)</h1>
</body>
</html>
```
<img width="577" height="298" alt="image" src="https://github.com/user-attachments/assets/124873e3-fc3b-41c2-8201-a45f6a833eef" />

Tạo file nginx/conf.d/default.conf (thay domain1.example.com và domain2.example.com bằng 2 domain của bạn):

```bash
Nginx
server {
    listen 80;
    server_name domain1.example.com;

    location / {
        root /usr/share/nginx/html/site1;
        index index.html;
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80;
    server_name domain2.example.com;

    location / {
        root /usr/share/nginx/html/site2;
        index index.html;
        try_files $uri $uri/ =404;
    }
}
```
Khởi chạy và kiểm tra các container:

```bash
docker compose up -d
docker compose ps
```
Khi test local trước khi có DNS thật, thêm vào file hosts (C:\Windows\System32\drivers\etc\hosts trên Windows hoặc /etc/hosts trên Linux):

```bash
Plaintext
127.0.0.1 domain1.example.com domain2.example.com
```
<img width="578" height="317" alt="image" src="https://github.com/user-attachments/assets/64ae5f4a-0a7f-46fa-b587-ee8eabc3b79c" />
<img width="572" height="304" alt="image" src="https://github.com/user-attachments/assets/f50fa7f0-8e92-4cef-98ce-f688dd5afe01" />
