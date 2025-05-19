# 🚀 Triển khai n8n bằng Docker tại Local

Đây là hướng dẫn triển khai **n8n** – nền tảng workflow tự động hóa mã nguồn mở – sử dụng **Docker Compose** tại máy local, bao gồm cấu hình PostgreSQL, `.env`, và volume để lưu dữ liệu an toàn.

---

## 🧱 Yêu cầu hệ thống

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- Git (nếu dùng repo)

---

## 📁 Cấu trúc thư mục

```bash
n8n-local/
├── docker-compose.yml
├── .env
└── README.md
```

---

## ⚙️ 1. Tạo file `.env`

```env
# PostgreSQL
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8npass
POSTGRES_DB=n8n

# n8n Auth
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=admin123

# n8n Settings
N8N_HOST=localhost
N8N_PORT=5678
WEBHOOK_URL=http://localhost:5678/
NODE_ENV=development
```

---

## 🐳 2. File `docker-compose.yml`

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    ports:
      - "${N8N_PORT}:${N8N_PORT}"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=${N8N_PORT}
      - WEBHOOK_URL=${WEBHOOK_URL}
      - NODE_ENV=${NODE_ENV}
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - postgres
    restart: always

  postgres:
    image: postgres
    container_name: n8n_postgres
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

volumes:
  n8n_data:
  postgres_data:
```

---

## ▶️ 3. Khởi chạy dịch vụ

```bash
docker-compose up -d
```

Sau đó truy cập: [http://localhost:5678](http://localhost:5678)

**Tài khoản đăng nhập:**
- Username: `admin`
- Password: `admin123`

---

## 📦 Backup & Restore

### Backup volume dữ liệu n8n:
```bash
docker run --rm --volumes-from n8n -v $(pwd):/backup busybox tar cvf /backup/n8n_data.tar /home/node/.n8n
```

---

## 💡 Ghi chú bảo mật

- Đổi `N8N_BASIC_AUTH_PASSWORD` trong file `.env` để bảo vệ hệ thống.
- Đừng commit file `.env` lên Git — thêm vào `.gitignore`.

---

## 📬 Thông tin liên hệ

Nếu bạn cần hỗ trợ hoặc nâng cấp triển khai cho production, hãy liên hệ dev của hệ thống.
