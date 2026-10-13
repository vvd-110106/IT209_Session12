# Hướng Dẫn Bài 3: Tách Biệt Luồng CI/CD Đa Môi Trường (Staging & Production) với GitHub Environments

---

## 1. Mục tiêu bài tập
- Hiểu và ứng dụng tính năng **GitHub Environments** trong GitHub Actions.
- Thiết lập quy tắc bảo vệ môi trường (**Environment Protection Rules**) với cơ chế **Manual Approval (Required Reviewers)** trước khi deploy lên Production.
- Tách biệt luồng deploy:
  - Push vào nhánh `develop` ➔ Tự động triển khai lên môi trường **Staging**.
  - Push vào nhánh `main` ➔ Tạm dừng chờ duyệt thủ công, sau khi bấm **Approve and deploy** mới triển khai lên môi trường **Production**.

---

## 2. Cấu trúc thư mục

```text
session12/
├── .github/
│   └── workflows/
│       └── cd-multi-env.yml       <-- GitHub Actions nhận diện file này ở root của repo
├── bt3/
│   ├── .github/
│   │   └── workflows/
│   │       └── cd-multi-env.yml   <-- Bản sao lưu trữ trong thư mục bài tập bt3
│   ├── screenshots/              <-- Thư mục chứa ảnh chụp màn hình nộp bài
│   │   └── .gitkeep
│   └── README.md                 <-- Hướng dẫn chi tiết cấu hình và kiểm thử
```

> **Lưu ý quan trọng về GitHub Actions**: 
> GitHub Actions chỉ tự động đọc và thực thi các file workflow nằm tại đường dẫn `.github/workflows/` ở **thư mục gốc (root)** của repository. Do đó file workflow đã được đặt ở cả thư mục `bt3/` (theo đúng yêu cầu nộp bài) và thư mục gốc của project để bạn test thực tế mượt mà nhất.

---

## 3. Các bước cấu hình trên GitHub Repository

### Bước 3.1: Tạo Môi trường `staging`
1. Mở Repository của bạn trên GitHub bằng trình duyệt.
2. Điều hướng tới: **Settings** ➔ ở thanh menu bên trái, tìm mục **Environments** (nằm dưới mục *Code and automation* hoặc *Secrets and variables*).
3. Bấm nút **New environment**.
4. Nhập tên: `staging` ➔ bấm **Configure environment**.
5. Môi trường `staging` này không cần cấu hình người duyệt (để tự động chạy khi push).

### Bước 3.2: Tạo Môi trường `production` & Bật Required Reviewers
1. Quay lại trang **Environments**, tiếp tục bấm **New environment**.
2. Nhập tên: `production` ➔ bấm **Configure environment**.
3. Tại phần **Environment protection rules**:
   - Tích chọn ô **Required reviewers**.
   - Trong ô tìm kiếm, gõ và chọn tài khoản GitHub của bạn (hoặc người đánh giá).
   - *(Tùy chọn)* Có thể tích chọn **Prevent self-review** nếu làm việc nhóm, nhưng với bài thực hành cá nhân thì **BỎ TÍCH** ô này để chính bạn có thể tự Approve.
4. Bấm **Save protection rules**.

---

## 4. Chi tiết file Workflow `.github/workflows/cd-multi-env.yml`

File kịch bản gồm 2 Job:
- **`deploy-staging`**:
  - Điều kiện: `if: github.ref_name == 'develop'`
  - Thuộc tính môi trường:
    ```yaml
    environment:
      name: staging
      url: https://staging.example.com
    ```
- **`deploy-prod`**:
  - Điều kiện: `if: github.ref_name == 'main'`
  - Thuộc tính môi trường:
    ```yaml
    environment:
      name: production
      url: https://api.prod.example.com
    ```
  - Khi Job này khởi chạy, GitHub kiểm tra thấy environment `production` có rule `Required reviewers`, GitHub Actions sẽ dừng lại ở trạng thái **Waiting for review**.

---

## 5. Các bước kiểm thử (Testing & Chụp ảnh nộp bài)

### Bước 5.1: Đẩy mã nguồn lên GitHub
Mở terminal tại thư mục dự án và chạy các lệnh sau:

```bash
# 1. Khởi tạo git (nếu chưa khởi tạo)
git init

# 2. Thêm remote tới GitHub repo của bạn
git remote add origin https://github.com/<tai-khoan-cua-ban>/<ten-repo>.git

# 3. Commit và push nhánh main
git add .
git commit -m "feat: config multi-env CI/CD pipeline"
git branch -M main
git push -u origin main
```

### Bước 5.2: Kiểm tra nhánh `main` và Chụp ảnh cửa sổ phê duyệt
1. Sau khi push lên nhánh `main`, vào tab **Actions** trên GitHub repository.
2. Bấm vào workflow đang chạy: bạn sẽ thấy job `deploy-staging` bị bỏ qua (skipped) do không phải nhánh `develop`.
3. Job `deploy-prod` sẽ hiển thị thông báo:
   > 🟡 **Waiting for review** - `production` needs approval.
4. 📸 **CHỤP ẢNH MÀN HÌNH TẠI ĐÂY (Ảnh nộp bài)**:
   - Bấm vào nút **Review deployments**.
   - Cửa sổ popup hiện ra hiển thị môi trường `production` và nút **Approve and deploy**.
   - Chụp lại toàn bộ màn hình này (lưu vào `bt3/screenshots/01_waiting_for_approval.png`).
5. Tích chọn ô `production` và bấm **Approve and deploy**.
6. Quan sát job tiếp tục thực thi và chuyển sang trạng thái ✅ **Passed (Tích xanh)**.
7. Chụp thêm ảnh màn hình hoàn tất thành công (lưu vào `bt3/screenshots/02_deploy_success.png`).

### Bước 5.3: Kiểm tra nhánh `develop`
Để kiểm tra luồng staging tự động chạy mà không cần duyệt:

```bash
# Tạo nhánh develop và push lên
git checkout -b develop
git push -u origin develop
```

- Vào tab **Actions**: Bạn sẽ thấy job `deploy-staging` tự động chạy thẳng và chuyển thành ✅ tích xanh mà không cần bất kỳ bước phê duyệt nào.

---

## 6. Sản phẩm nộp bài
1. Tệp kịch bản: `bt3/.github/workflows/cd-multi-env.yml`
2. Ảnh chụp màn hình cửa sổ phê duyệt: `bt3/screenshots/01_waiting_for_approval.png`
