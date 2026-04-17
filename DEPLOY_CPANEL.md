# 🚀 Hướng Dẫn Deploy THD EDU QUIZ Lên cPanel

---

## 📋 Tổng Quan Kiến Trúc

```
cPanel Server
├── public_html/              ← Frontend React (đã build)
│   ├── index.html
│   ├── static/
│   └── .htaccess             ← Cần tạo thủ công
│
└── quiz-backend/             ← Backend Node.js
    ├── index.js
    ├── .env                  ← Cần tạo thủ công trên server
    └── node_modules/
```

**Flow hoạt động:**
```
Người dùng → gojoforums.site         → public_html (React)
Người dùng → gojoforums.site/api     → Node.js App (Backend)
```

---

## 🛠 BƯỚC 1: Chuẩn Bị Trước Khi Upload

### 1.1 Build Frontend (trên máy local)

```bash
# Tại thư mục gốc của dự án
npm run build
```

Sau khi chạy xong, thư mục `build/` sẽ được tạo ra. Đây là thứ bạn cần upload.

### 1.2 Chuẩn Bị Backend

Không cần build backend. Chỉ cần upload toàn bộ thư mục `quiz-backend/` **NGOẠI TRỪ** thư mục `node_modules/` (vì nó rất nặng, sẽ cài lại trên server).

---

## 📁 BƯỚC 2: Upload Files Lên cPanel

### 2.1 Upload Frontend

1. Đăng nhập vào **cPanel** → **File Manager**
2. Vào thư mục `public_html`
3. **Xóa** tất cả file cũ trong `public_html` (nếu có)
4. Upload **toàn bộ nội dung** trong thư mục `build/` (không upload cả folder `build`, mà upload các file bên trong)
5. Sau khi upload, `public_html` sẽ trông như:
   ```
   public_html/
   ├── index.html
   ├── static/
   ├── Trollface.png
   └── ...
   ```

### 2.2 Upload Backend

1. Vào thư mục **home** (một cấp trên `public_html`)
2. Tạo thư mục mới tên `quiz-backend`
3. Upload **toàn bộ nội dung** thư mục `quiz-backend/` **NGOẠI TRỪ** `node_modules/`

> ⚠️ **Quan trọng:** Không để backend trong `public_html` vì sẽ bị public ra ngoài.

---

## 🗄 BƯỚC 3: Tạo Database MySQL Trên cPanel

1. **cPanel** → **MySQL Databases**
2. **Tạo Database mới:** Đặt tên `quiz_website` (tên đầy đủ sẽ là `username_quiz_website`)
3. **Tạo User MySQL mới:** Đặt tên và mật khẩu mạnh
4. **Gán User vào Database:** Chọn **All Privileges**
5. Ghi nhớ các thông tin:
   - **DB Host:** `localhost`
   - **DB Name:** `username_quiz_website`cefiazen_quiz_website
   - **DB User:** `username_dbuser`cefiazen_Hoangthanhlich
   - **DB Password:** `mật_khẩu_bạn_đặt`

---

## ⚙️ BƯỚC 4: Cấu Hình Node.js App Trên cPanel

1. **cPanel** → **Setup Node.js App**
2. Nhấn **"Create Application"**
3. Điền các thông số:

| Trường | Giá trị |
|--------|---------|
| **Node.js version** | 18.x hoặc 20.x |
| **Application mode** | Production |
| **Application root** | `quiz-backend` |
| **Application URL** | `yourdomain.com/api` |
| **Application startup file** | `index.js` |

4. Nhấn **Create** → cPanel sẽ cấp cho bạn thông tin về đường dẫn.

### 4.1 Cài đặt Dependencies

Sau khi tạo App, trong trang quản lý Node.js App, nhấn nút:
- **"Run NPM Install"** → Đợi cài xong (có thể mất 2-5 phút)

Hoặc dùng **Terminal** trong cPanel:
```bash
cd ~/quiz-backend
npm install --production
```

---

## 🔐 BƯỚC 5: Tạo File `.env` Trên Server

Trong **File Manager**, vào thư mục `quiz-backend/`, tạo file mới tên `.env` với nội dung:

```env
# ============================================
#  THD EDU QUIZ - PRODUCTION CONFIG
# ============================================

# ===== DATABASE =====
DB_HOST=localhost
DB_PORT=3306
DB_USER=username_dbuser
DB_PASSWORD=mật_khẩu_của_bạn
DB_NAME=username_quiz_website

# ===== SERVER =====
JWT_SECRET=một_chuỗi_bí_mật_dài_và_ngẫu_nhiên_ở_đây
NODE_ENV=production
CORS_ORIGIN=https://yourdomain.com
BACKEND_PORT=3001
BASE_PATH=/api

# ===== GEMINI AI =====
GEMINI_API_KEY=AIza...key_của_bạn...
GEMINI_MODEL=gemini-2.5-flash

# ===== SMTP =====
SMTP_HOST=smtp.gmail.com
SMTP_PORT=465
SMTP_SECURE=true
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_FROM="THD EDU QUIZ <your-email@gmail.com>"

# ===== OTP =====
OTP_TTL_SECONDS=600
OTP_THROTTLE_SECONDS=60
OTP_MAX_ATTEMPTS=5
```

> 🔑 **Lưu ý:** Thay `username_` bằng username cPanel thực tế của bạn.

---

## 🌐 BƯỚC 6: Cấu Hình `.htaccess` Trong `public_html`

Tạo file `.htaccess` trong thư mục `public_html/` với nội dung:

```htaccess
Options -MultiViews
RewriteEngine On

# Chuyển tiếp /api/* sang Node.js backend
RewriteCond %{REQUEST_URI} ^/api [NC]
RewriteRule ^(.*)$ http://localhost:3001/$1 [P,L]

# React Router - Redirect tất cả route về index.html
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_URI} !^/api
RewriteRule ^ index.html [L]
```

> ⚠️ **Lưu ý:** Port `3001` phải khớp với `BACKEND_PORT` trong file `.env` trên server.

---

## ▶️ BƯỚC 7: Khởi Động Backend

Trong **cPanel → Setup Node.js App**, tìm ứng dụng của bạn và nhấn nút **Start** (hoặc **Restart** nếu đang chạy).

Để kiểm tra backend đã chạy chưa, truy cập:
```
https://yourdomain.com/api/health
```
Nếu trả về `{"status":"ok"}` → **Thành công!**

---

## ✅ BƯỚC 8: Kiểm Tra Sau Deploy

| Kiểm tra | URL | Kết quả mong đợi |
|----------|-----|-----------------|
| Frontend load | `https://yourdomain.com` | Trang Welcome hiện ra |
| Backend health | `https://yourdomain.com/api/health` | `{"status":"ok"}` |
| Đăng nhập | Thử đăng nhập trên web | Cookie được set, không lỗi 401 |
| Tạo Quiz AI | Thử tính năng AI | Gemini trả về câu hỏi |

---

## 🔧 Xử Lý Lỗi Thường Gặp

### ❌ Lỗi 502 Bad Gateway khi vào `/api`
- **Nguyên nhân:** Backend chưa khởi động hoặc sai port.
- **Cách sửa:** Vào cPanel → Setup Node.js App → Restart. Kiểm tra `BACKEND_PORT` trong `.env` khớp với port trong `.htaccess`.

### ❌ Lỗi 404 khi refresh trang
- **Nguyên nhân:** Thiếu hoặc sai file `.htaccess`.
- **Cách sửa:** Kiểm tra lại nội dung file `.htaccess` trong `public_html`.

### ❌ Lỗi 401 khi đăng nhập
- **Nguyên nhân:** `CORS_ORIGIN` không khớp với domain thực tế.
- **Cách sửa:** Đảm bảo `CORS_ORIGIN=https://yourdomain.com` (đúng giao thức `https://`, không có dấu `/` cuối).

### ❌ Database connection failed
- **Nguyên nhân:** Thông tin DB sai.
- **Cách sửa:** Kiểm tra lại `DB_USER`, `DB_PASSWORD`, `DB_NAME` trong `.env` trên server. Đảm bảo đã thêm `username_` prefix.

---

## 🔄 Cập Nhật Code Sau Deploy

Khi có thay đổi code:

```bash
# 1. Build lại frontend
npm run build

# 2. Upload lại thư mục build/ lên public_html/

# 3. Nếu sửa backend: Upload lại các file .js tương ứng

# 4. Restart Node.js App trong cPanel
```

---

*Tài liệu này áp dụng cho **THD EDU QUIZ** - hệ thống quản lý bài kiểm tra trực tuyến.*
