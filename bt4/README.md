# Hướng Dẫn Bài 4: Tự Động Triển Khai Ứng Dụng Spring Boot (Container) Lên VPS Từ Xa Qua SSH

---

## 1. Mục tiêu bài tập
- Xây dựng quy trình tự động hóa CI/CD:
  1. **Job `build-image`**: Đóng gói ứng dụng thành Docker Image và đẩy lên **GitHub Container Registry (GHCR)**.
  2. **Job `deploy-vps`**: Kết nối từ xa tới máy chủ VPS qua **SSH Action** (`appleboy/ssh-action@v1.0.3`), kéo Docker Image mới về và khởi chạy lại Container.
- Đảm bảo ứng dụng chạy nền ổn định trên cổng `8080` và phản hồi endpoint kiểm tra trạng thái `/actuator/health` với mã **HTTP 200 OK**.

---

## 2. Cấu trúc thư mục

```text
session12/
├── .github/
│   └── workflows/
│       ├── cd-multi-env.yml
│       └── deploy-vps.yml         <-- Kịch bản CI/CD triển khai VPS tại root repo
├── Dockerfile                     <-- Dockerfile ứng dụng tại root để GitHub Actions build
├── bt3/
└── bt4/
    ├── .github/
    │   └── workflows/
    │       └── deploy-vps.yml     <-- Tệp kịch bản nộp bài trong thư mục bt4
    ├── Dockerfile                 <-- Dockerfile mô phỏng Spring Boot Actuator
    ├── screenshots/              <-- Thư mục chứa ảnh chụp màn hình nộp bài
    │   └── .gitkeep
    └── README.md                 <-- Hướng dẫn chi tiết thiết lập & kiểm thử
```

---

## 3. Chuẩn bị máy chủ VPS & Cấu hình GitHub Secrets

### Bước 3.1: Chuẩn bị trên máy chủ VPS
Đảm bảo VPS của bạn đã cài đặt Docker và user có quyền chạy lệnh docker:
```bash
# Cài đặt docker (nếu VPS chưa có)
sudo apt update
sudo apt install -y docker.io

# Cho phép user hiện tại chạy docker không cần sudo (ví dụ user: ubuntu)
sudo usermod -aG docker $USER
```

### Bước 3.2: Tạo cặp SSH Key (nếu chưa có)
Mở terminal trên máy tính của bạn (hoặc ngay trên VPS):
```bash
ssh-keygen -t rsa -b 4096 -C "github-actions-deploy"
```
- Public key (`id_rsa.pub`): Đưa nội dung này vào file `~/.ssh/authorized_keys` trên máy chủ VPS.
- Private key (`id_rsa`): Dùng để thêm vào GitHub Secrets.

### Bước 3.3: Khai báo Repository Secrets trên GitHub
1. Mở GitHub Repository của bạn trên trình duyệt.
2. Vào **Settings** ➔ ở menu bên trái chọn **Secrets and variables** ➔ chọn **Actions**.
3. Bấm **New repository secret** và thêm lần lượt các Secret sau:

| Tên Secret | Giá trị (Ví dụ) | Mô tả |
| :--- | :--- | :--- |
| `SSH_HOST` | `103.xxx.xxx.xxx` | Địa chỉ IP Public của máy chủ VPS |
| `SSH_USER` | `root` hoặc `ubuntu` | Tên đăng nhập SSH vào VPS |
| `SSH_KEY` | `-----BEGIN OPENSSH PRIVATE KEY----- ... -----END OPENSSH PRIVATE KEY-----` | Toàn bộ nội dung khóa riêng tư (Private Key) |
| `SSH_PORT` *(tùy chọn)* | `22` | Cổng SSH nếu VPS đổi cổng khác mặc định |

### Bước 3.4: Bật quyền ghi Packages cho GitHub Actions
1. Vào **Settings** ➔ **Actions** ➔ **General**.
2. Cuộn xuống phần **Workflow permissions**:
   - Chọn **Read and write permissions**.
   - Bấm **Save**.
*(Bước này cho phép `GITHUB_TOKEN` có quyền đẩy và tải image lên/xuống GitHub Container Registry)*.

---

## 4. Chi tiết kịch bản CI/CD: `.github/workflows/deploy-vps.yml`

Kịch bản gồm 2 Job chạy tuần tự:

```mermaid
graph TD
    A[Push code vào main] --> B[Job 1: build-image]
    B -->|Build & Push GHCR Thành công| C[Job 2: deploy-vps]
    C -->|SSH vào VPS| D[Đăng nhập GHCR trên VPS]
    D --> E[Kéo Image mới: docker pull ghcr.io/.../app:latest]
    E --> F[Khởi chạy lại: docker stop & rm & run -p 8080:8080]
    F --> G[Kiểm tra HTTP 200 OK tại /actuator/health]
```

### Điểm nổi bật trong kịch bản:
- **Tự động chuyển tên người dùng về chữ thường (`lowercase`)**: Docker không cho phép tên registry/image chứa chữ in hoa. Workflow đã tích hợp sẵn bước chuẩn hóa tự động để không bao giờ bị lỗi invalid reference format.
- **Sử dụng `appleboy/ssh-action@v1.0.3`**: Thực hiện chuẩn 3 bước như yêu cầu đề bài:
  1. `docker login ghcr.io ...`
  2. `docker pull ghcr.io/<username>/app:latest`
  3. `docker stop app || true && docker rm app || true && docker run -d --name app -p 8080:8080 ...`

---

## 5. Hướng dẫn kiểm thử & Chụp ảnh nộp bài

### Bước 5.1: Đẩy mã nguồn lên GitHub
Mở PowerShell tại thư mục `session12` và chạy:
```powershell
git add .
git commit -m "feat(bt4): add VPS deployment workflow and Dockerfile"
git push origin main
```

### Bước 5.2: Theo dõi log thực thi và Chụp ảnh SSH Deploy (Ảnh 1)
1. Truy cập tab **Actions** trên GitHub repository.
2. Bấm vào lượt chạy mới nhất của workflow **Build and Deploy Spring Boot to VPS**.
3. Chờ job `build-image` hoàn tất, sau đó bấm vào job `deploy-vps`.
4. Mở rộng bước **Thực thi câu lệnh triển khai trên VPS qua SSH**.
5. 📸 **CHỤP ẢNH MÀN HÌNH (Minh chứng 1)**:
   - Chụp lại màn hình log chứa các dòng:
     - `Kéo Docker image mới nhất...`
     - `Dừng và xóa container cũ...`
     - Bảng `docker ps` hiển thị container `app` đang ở trạng thái `Up ... 0.0.0.0:8080->8080/tcp`.
   - Lưu ảnh vào: `bt4/screenshots/01_ssh_deploy_log.png`.

### Bước 5.3: Kiểm tra ứng dụng trên trình duyệt (Ảnh 2)
1. Mở trình duyệt web hoặc dùng curl:
   ```text
   http://<ĐỊA_CHỈ_IP_VPS>:8080/actuator/health
   ```
2. Trình duyệt hiển thị dữ liệu JSON phản hồi mã **HTTP 200 OK**:
   ```json
   {
     "status": "UP",
     "components": {
       "diskSpace": { "status": "UP" },
       "ping": { "status": "UP" }
     },
     "service": "Spring Boot Demo Service"
   }
   ```
3. 📸 **CHỤP ẢNH MÀN HÌNH (Minh chứng 2)**:
   - Chụp toàn bộ cửa sổ trình duyệt hiển thị thanh địa chỉ IP và nội dung JSON `{"status": "UP"}`.
   - Lưu ảnh vào: `bt4/screenshots/02_actuator_health.png`.

---

## 6. Sản phẩm nộp bài
1. Tệp kịch bản: `bt4/.github/workflows/deploy-vps.yml`
2. Ảnh chụp màn hình:
   - `bt4/screenshots/01_ssh_deploy_log.png`: Log SSH Deploy thành công trên GitHub Actions.
   - `bt4/screenshots/02_actuator_health.png`: Trình duyệt truy cập endpoint actuator health trả về 200 OK.
