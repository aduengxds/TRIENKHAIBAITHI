# ĐỒ ÁN MÔN HỌC: TRIỂN KHAI & QUẢN TRỊ HỆ THỐNG
## Đề 19: Website Tin tức / Cổng thông tin (News Portal / Content Management System)

---

## 1. Giới thiệu dự án
Hệ thống Cổng thông tin tin tức được thiết kế và triển khai theo mô hình **Production-Ready Enterprise**, tích hợp đầy đủ các tiêu chuẩn về vận hành, bảo mật (Hardening), giám sát (Prometheus + Grafana), phân tích log tập trung (Loki + LogQL) và quản lý cơ sở dữ liệu qua giao diện web trực quan.

### Tech Stack:
- **Web App / CMS:** WordPress (PHP 8.2-FPM)
- **Database:** MySQL 8.0 + phpMyAdmin
- **Reverse Proxy & Web Server:** Nginx (với các cấu hình Security Headers & Caching)
- **Containerization:** Docker & Docker Compose
- **Monitoring & Observability:** Prometheus, Grafana, Loki, Promtail

---

## 2. Cấu trúc thư mục dự án
```text
baithi/
├── docker/
│   ├── nginx/
│   │   └── conf.d/
│   │       └── default.conf
│   ├── prometheus/
│   │   └── prometheus.yml
│   ├── loki/
│   │   └── loki-config.yaml
│   └── promtail/
│   │   └── promtail-config.yaml
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 3. Hướng dẫn Triển khai (Deployment Guide)

### Yêu cầu tiên quyết:
- Đã cài đặt Docker và Docker Compose trên máy chủ Ubuntu/Linux hoặc Windows (WSL2).

### Các bước thực hiện:
1. **Clone/Copy dự án** vào thư mục làm việc trên server.
2. **Tạo file môi trường `.env`** từ file mẫu:
   ```bash
   cp .env.example .env
   ```
   *(Có thể chỉnh sửa lại mật khẩu database nếu cần thiết trong file `.env`)*

3. **Khởi chạy hệ thống bằng Docker Compose:**
   ```bash
   docker compose up -d
   ```

4. **Kiểm tra trạng thái các container:**
   ```bash
   docker compose ps
   ```

---

## 4. Truy cập các dịch vụ hệ thống

| Dịch vụ | Đường dẫn truy cập / Port | Mô tả |
| :--- | :--- | :--- |
| **Website Tin tức** | `http://localhost/` | Giao diện chính cổng thông tin tin tức |
| **Trang quản trị (Admin)** | `http://localhost/wp-admin` | Đăng nhập quản trị viên đăng bài viết, danh mục |
| **phpMyAdmin** | `http://localhost:8081/` | Quản trị cơ sở dữ liệu MySQL trực quan |
| **Prometheus** | `http://localhost:9090/` | Giám sát metrics hệ thống & container |
| **Grafana** | `http://localhost:3000/` | Dashboard trực quan hóa & LogQL |
| **Loki** | `http://localhost:3100/` | Log Aggregation Engine |

---

## 5. Giám sát & Truy vấn Log (LogQL)

Trong **Grafana**, bạn có thể kết nối với **Loki** datasource và sử dụng các câu lệnh **LogQL** sau để phân tích log:
- **Lọc lỗi HTTP 5xx của Nginx:**
  ```logql
  {container_name="news_nginx"} | json | status >= 500
  ```
- **Theo dõi lỗi ứng dụng PHP/WordPress:**
  ```logql
  {container_name="news_app"} |~ "error|FATAL|Warning"
  ```
- **Kiểm tra truy cập trang đăng nhập Admin:**
  ```logql
  {container_name="news_nginx"} |~ "POST /wp-login.php"
  ```

---

## 6. Bảo mật & Hardening (Yêu cầu kỹ thuật)
- **Nginx Security Headers:** Đã cấu hình chống Clickjacking (`X-Frame-Options`), XSS Protection và ẩn thông tin phiên bản server (`server_tokens off`).
- **Network Isolation:** Database MySQL chỉ hoạt động trong mạng nội bộ Docker (`news_net`), không mở port trực tiếp ra ngoài internet.
- **Firewall & UFW:** Khuyến nghị kích hoạt UFW trên hệ thống thật chỉ mở các port cần thiết (`22`, `80`, `443`).
