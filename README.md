# THD EDU QUIZ - Hệ Thống Quản Lý Bài Kiểm Tra Trực Tuyến

Chào mừng bạn đến với **THD EDU QUIZ**, một nền tảng quản lý và thực hiện bài kiểm tra trực tuyến mạnh mẽ, giao diện hiện đại và chuyên nghiệp.
# Dự án gốc 
[https://github.com/HoanBuCon/Quiz-Website/tree/master](https://github.com/HoanBuCon/Quiz-Website/tree/master)
---

##  Tính Năng Chính
- **Quản lý lớp học:** Tạo và quản lý các lớp học công khai hoặc riêng tư.
- **Tạo bài kiểm tra:** Hỗ trợ tạo đề thi từ file Word (định dạng chuẩn).
- **Thống kê & Kết quả:** Theo dõi hoạt động làm bài thông qua biểu đồ Contribution Graph (như GitHub).
- **Bảo mật:** Xác thực qua Cookie HttpOnly, hỗ trợ chế độ bảo trì toàn trang.

---

##  Yêu Cầu Hệ Thống
- **Node.js**: Phiên bản 18.x trở lên.
- **MySQL**: 5.7 hoặc 8.0.
- **Trình duyệt**: Chrome, Edge, Firefox bản mới nhất.

---

##  1. Hướng Dẫn Cài Đặt Local (XAMPP / MySQL)

### Bước 1: Chuẩn bị Cơ sở dữ liệu
1. Mở **XAMPP Control Panel** và Start **Apache** & **MySQL**.
2. Truy cập `http://localhost/phpmyadmin`.
3. Tạo một database mới tên là `quiz_web`.
4. Import file SQL (nếu có) hoặc để hệ thống tự tạo bảng khi chạy backend lần đầu.

### Bước 2: Cấu hình Môi trường
Tạo file `.env` tại thư mục gốc của dự án với nội dung:
```env
# Database
DB_HOST=localhost
DB_USER=root
DB_PASS=
DB_NAME=quiz_web

# Auth
JWT_SECRET=your_secret_key_2024
BACKEND_PORT=4000

# Frontend
PORT=3000
CORS_ORIGIN=http://localhost:3000
```

### Bước 3: Cài đặt và Chạy
1. **Cài đặt thư viện:**
   ```bash
   npm install
   cd quiz-backend
   npm install
   cd ..
   ```
2. **Chạy hệ thống (Mở 2 terminal song song):**
   - Terminal 1 (Backend): `cd quiz-backend && npm run dev`
   - Terminal 2 (Frontend): `npm start`

---

##  2. Host qua Cloudflare Tunnel (Cho phép truy cập từ xa)

Nếu bạn muốn chạy server tại nhà nhưng dùng tên miền (domain) để truy cập:

1. **Cài đặt cloudflared:** Tải và cài đặt trên máy tính của bạn.
2. **Login và Tạo Tunnel:**
   ```bash
   cloudflared tunnel login
   cloudflared tunnel create <tên_tunnel>
   ```
3. **Cấu hình file config.yml:**
   ```yaml
   tunnel: <ID_TUNNEL>
   credentials-file: C:\Users\User\.cloudflared\<ID_TUNNEL>.json
   ingress:
     - hostname: yourdomain.xyz
       service: http://localhost:3000
     - service: http_status:404
   ```
4. **Cập nhật CORS:** Trong file `.env`, đổi `CORS_ORIGIN` thành `https://yourdomain.xyz`.
5. **Chạy Tunnel:** `cloudflared tunnel run <tên_tunnel>`

---

## ☁️ 3. Triển Khai Trên cPanel (Production)

### Bước 1: Build Frontend
1. Chạy lệnh: `npm run build`.
2. Toàn bộ code trong thư mục `/build` sẽ được tải lên `public_html` của cPanel.

### Bước 2: Setup Node.js App cho Backend
1. Trên cPanel, tìm mục **Setup Node.js App**.
2. Tạo app mới:
   - **Application root:** `/quiz-backend`
   - **Application URL:** `domain.com/api` (hoặc tùy chọn)
   - **Startup file:** `index.js`
3. Upload toàn bộ nội dung trong thư mục `quiz-backend` lên server.
4. Chỉnh sửa file `.env` trên server để trỏ đúng thông tin Database của cPanel.

### Bước 3: Cấu hình .htaccess (Quan trọng)
Trong thư mục `public_html`, hãy đảm bảo có file `.htaccess` để điều hướng React Router:
```htaccess
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

---

## ⚠️ Lưu Ý Quan Trọng
- **CORS:** Luôn đảm bảo `CORS_ORIGIN` trong file `.env` khớp với Domain/IP mà bạn đang sử dụng để truy cập web.
- **Port:** Nếu chạy Local, Backend dùng port `4000`, Frontend dùng port `3000`.
- **Maintenance:** Để tắt/bật chế độ bảo trì, hãy chỉnh sửa `IS_MAINTENANCE_MODE` trong `src/utils/maintenanceConfig.ts`.

---

##  Hỗ Trợ
Nếu gặp lỗi trong quá trình setup, vui lòng kiểm tra console của trình duyệt hoặc log của Node.js để biết thêm chi tiết.

**Brand:** THD EDU QUIZ
**Author:** Hoang
