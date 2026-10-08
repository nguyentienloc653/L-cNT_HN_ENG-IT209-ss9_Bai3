# BÀI TẬP 3: VIẾT DOCKERFILE 3 DÒNG ĐÓNG GÓI TRANG WEB HTML TĨNH

- **Môn học / Chuyên đề**: DevOps & Java Fundamental
- **Session**: 09 - Docker Cơ Bản & Containerization
- **Bài tập**: Bài 3 - Viết Dockerfile đóng gói trang Web HTML tĩnh
- **Đường dẫn nộp bài**: `homework/session_09/ex3/`
- **Các tệp nộp bài**: `index.html`, `Dockerfile`, `README.md`

---

## 1. Mục tiêu & Bối cảnh
- Tự viết tệp `Dockerfile` ngắn gọn để đóng gói một trang HTML tĩnh thành một Docker Image hoàn chỉnh.
- Sử dụng base image `nginx:alpine` siêu nhẹ nhằm tối ưu kích thước image và tài nguyên hệ thống.
- Chuyển tiếp cổng (Port Forwarding) từ máy chủ lưu trữ (host port `8081`) tới cổng mặc định của Nginx (`80`).
- Kiểm tra tính đúng đắn của ứng dụng thông qua lệnh `curl`.

---

## 2. Cấu trúc thư mục & Mã nguồn

### 2.1. Cấu trúc thư mục
```text
homework/session_09/ex3/
├── Dockerfile
├── index.html
└── README.md
```

### 2.2. Nội dung tệp `index.html`
```html
<h1>Hello Docker Session 09!</h1>
```

### 2.3. Nội dung tệp `Dockerfile`
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

---

## 3. Các bước thực thi & Kết quả Terminal

### Bước 1: Build Docker Image
Chạy lệnh build image từ `Dockerfile` tại thư mục hiện hành và đặt tag là `my-html-app:v1`:
```bash
docker build -t my-html-app:v1 .
```

**Output Terminal:**
```text
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM nginx:alpine
alpine: Pulling from library/nginx
e72112c14215: Pull complete
e76228b47809: Pull complete
64c8194480fe: Pull complete
6c53d0b2a666: Pull complete
e2de96513ba9: Pull complete
d9aae54b5831: Pull complete
9a9a644fdd6a: Pull complete
745dfb2690dd: Pull complete
Digest: sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2
Status: Downloaded newer image for nginx:alpine
 ---> df221db836e1
Step 2/2 : COPY index.html /usr/share/nginx/html/index.html
 ---> d52bfae56254
Successfully built d52bfae56254
Successfully tagged my-html-app:v1
```

---

### Bước 2: Khởi chạy Container từ Image
Khởi chạy container ở chế độ chạy nền (`-d`), ánh xạ cổng `8081` ở host vào cổng `80` của container (`-p 8081:80`), đặt tên container là `html-app`:
```bash
docker run -d -p 8081:80 --name html-app my-html-app:v1
```

**Output Terminal:**
```text
8206ee17cbe1cafefca27b05e661c4f5a43aacbb11e23f4bbcdbbdf0f911fc58
```

Kiểm tra trạng thái container đang hoạt động:
```bash
docker ps
```

**Output Terminal:**
```text
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                                     NAMES
8206ee17cbe1   my-html-app:v1   "/docker-entrypoint.…"   12 seconds ago   Up 11 seconds   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   html-app
```

---

### Bước 3: Kiểm tra phản hồi ứng dụng (Verification)
Gửi yêu cầu HTTP GET đến cổng `8081`:
```bash
curl http://localhost:8081
```

**Kết quả nhận được:**
```html
<h1>Hello Docker Session 09!</h1>
```

Kiểm tra chi tiết HTTP Header:
```bash
curl -I http://localhost:8081
```

**Output Terminal:**
```text
HTTP/1.1 200 OK
Server: nginx/1.31.6
Date: Thu, 08 Oct 2026 15:56:44 GMT
Content-Type: text/html
Content-Length: 34
Last-Modified: Thu, 08 Oct 2026 15:55:57 GMT
Connection: keep-alive
ETag: "6ac7bd0d-22"
Accept-Ranges: bytes
```

---

## 4. Giải thích kỹ thuật

1. **`FROM nginx:alpine`**:
   - Sử dụng base image Nginx chạy trên nền tảng **Alpine Linux**.
   - Alpine là bản phân phối Linux siêu nhỏ gọn (chỉ vài megabyte), giúp image `my-html-app:v1` có kích thước cực nhẹ (~40MB), tải và khởi động tức thì, đồng thời giảm thiểu bề mặt tấn công bảo mật (attack surface).
2. **`COPY index.html /usr/share/nginx/html/index.html`**:
   - Sao chép tệp `index.html` từ thư mục làm việc cục bộ của máy host vào đường dẫn root web server mặc định của Nginx Alpine (`/usr/share/nginx/html/index.html`).
   - Tệp mới này sẽ ghi đè lên trang chào mừng mặc định của Nginx.
3. **Các tham số lệnh `docker run`**:
   - `-d` (*detached mode*): Chạy container ngầm dưới nền, giải phóng Terminal.
   - `-p 8081:80` (*port mapping*): Chuyển tiếp cổng `8081` trên máy chủ host vào cổng `80` (cổng HTTP mặc định của Nginx) bên trong container.
   - `--name html-app`: Đặt tên dễ nhớ cho container để thuận tiện quản trị (`docker stop html-app`, `docker rm html-app`).

---

## 5. Hướng dẫn dọn dẹp môi trường (Tùy chọn)
Khi muốn dừng và xóa container hoặc image sau khi kiểm tra xong:
```bash
# Dừng container
docker stop html-app

# Xóa container
docker rm html-app

# Xóa image (nếu không cần dùng lại)
docker rmi my-html-app:v1
```
