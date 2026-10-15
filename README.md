# BÀI 3: TÁCH BIỆT LUỒNG CI/CD ĐA MÔI TRƯỜNG (STAGING & PRODUCTION) VỚI GITHUB ENVIRONMENTS

- **Môn học:** DevOps / CI-CD
- **Bài thực hành:** Bài 3 - Session 12
- **Mục tiêu:**
  - Quản lý hạ tầng triển khai đa môi trường (**Staging** vs **Production**) thông qua tính năng **GitHub Environments**.
  - Cấu hình **Environment Protection Rules** yêu cầu phê duyệt thủ công (**Required reviewers**) trước khi deploy lên Production.
  - Phân tách luồng tự động: đẩy lên `develop` triển khai tự động lên `staging`, đẩy lên `main` triển khai lên `production` với bước phê duyệt kiểm soát an toàn.
- **Đường dẫn gợi ý:** `homework/session_12/ex3/`

---

## 1. Cấu trúc thư mục dự án

```text
PTIT_CNTT1_IT209_Session12_Bai03/
├── .github/
│   └── workflows/
│       └── cd-multi-env.yml       # Kịch bản CD đa môi trường Staging & Production
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── src/
│   ├── main/java/com/example/demo/DemoApplication.java
│   └── test/java/com/example/demo/DemoApplicationTests.java
├── image/                         # Thư mục lưu ảnh minh chứng phê duyệt Production
├── .gitignore
├── build.gradle
├── gradlew
├── gradlew.bat
├── settings.gradle
└── README.md                      # Báo cáo chi tiết và hướng dẫn cấu hình
```

---

## 2. Chi tiết kịch bản GitHub Actions Workflow (`.github/workflows/cd-multi-env.yml`)

```yaml
name: Multi-Environment CD (Staging & Production)

on:
  push:
    branches:
      - main
      - develop

jobs:
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://api.staging.example.com

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Set up JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build application
        run: ./gradlew build -x test

      - name: Deploy to Staging
        run: |
          echo "Deploying application to Staging environment: https://api.staging.example.com"
          echo "Commit SHA: ${{ github.sha }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Deployment to Staging completed successfully!"

  deploy-prod:
    name: Deploy to Production
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://api.prod.example.com

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Set up JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build application
        run: ./gradlew build -x test

      - name: Deploy to Production
        run: |
          echo "Deploying application to Production environment: https://api.prod.example.com"
          echo "Commit SHA: ${{ github.sha }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Manual review approved. Deployment to Production completed successfully!"
```

### Phân tích và giải thích các thành phần:

1. **Bộ lọc sự kiện (`on: push: branches: [ main, develop ]`)**:
   - Lắng nghe sự kiện commit được đẩy vào 2 nhánh chính đại diện cho 2 môi trường:
     - Nhánh `develop`: Môi trường kiểm thử tích hợp (Staging).
     - Nhánh `main`: Môi trường sản phẩm thực tế (Production).

2. **Job `deploy-staging`**:
   - Điều kiện thực thi: `if: github.ref == 'refs/heads/develop'`.
   - Gắn với môi trường: `environment: name: staging`.
   - Cơ chế: Tự động chạy và triển khai ngay lập tức lên Staging mà không yêu cầu phê duyệt thủ công.

3. **Job `deploy-prod`**:
   - Điều kiện thực thi: `if: github.ref == 'refs/heads/main'`.
   - Gắn với môi trường: `environment: name: production`.
   - Khi chạy đến Job này, GitHub Actions nhận diện môi trường `production` có cấu hình **Protection Rules** (Required Reviewers).
   - Tiến trình tự động tạm dừng lại ở trạng thái **Waiting for review** và gửi thông báo cho người duyệt.
   - Chỉ khi người có thẩm quyền phê duyệt (**Approve and deploy**), các bước biên dịch và triển khai thực sự mới được phép chạy.

---

## 3. Hướng dẫn thiết lập GitHub Environments (Bạn thực hiện trên GitHub)

### Bước 1: Tạo môi trường `staging`
1. Mở GitHub Repository của bạn.
2. Vào **Settings** -> Mục bên trái chọn **Environments**.
3. Nhấp vào nút **New environment**.
4. Nhập tên: `staging` -> Nhấp **Configure environment** -> Nhấp **Save**.

### Bước 2: Tạo môi trường `production` với Protection Rules
1. Nhấp lại vào **New environment**.
2. Nhập tên: `production` -> Nhấp **Configure environment**.
3. Tại mục **Environment protection rules**:
   - Tích chọn ô: **Required reviewers**.
   - Trong ô tìm kiếm, gõ và chọn tài khoản GitHub cá nhân của bạn (hoặc nhóm phụ trách).
   - Nhấp nút **Save protection rules**.

---

## 4. Hướng dẫn kiểm tra và nộp bài (Bạn tự thực hiện)

### Bước 1: Khởi tạo và đẩy code lên nhánh `main`
```bash
cd d:\Documents\DEVOPS\Session12\PTIT_CNTT1_IT209_Session12_Bai03
git init
git add .
git commit -m "feat: setup multi-environment CD workflow with staging and production protection"
git branch -M main
git remote add origin <URL_REPOSITORY_CỦA_BẠN>
git push -u origin main
```

*(Tùy chọn: Bạn có thể tạo nhánh `develop` và push thử: `git checkout -b develop && git push origin develop` để kiểm tra luồng staging tự động chạy không cần duyệt).*

### Bước 2: Kiểm tra phê duyệt trên GitHub Actions
1. Vào tab **Actions** trên GitHub repository.
2. Chọn lần chạy workflow mới nhất trên nhánh `main`.
3. Quan sát giao diện:
   - Job `deploy-staging` bị bỏ qua (Skipped) do điều kiện branch.
   - Job `deploy-prod` chuyển sang trạng thái màu vàng: **Waiting for review**.
   - Xuất hiện thông báo: *"deploy-prod needs approval from designated reviewers before continuing"*.
4. **Chụp ảnh màn hình cửa sổ phê duyệt (Review deployments)** hiển thị hộp thoại chờ duyệt và nút **Approve and deploy**.
5. Nhấp nút **Review deployments** -> Tích chọn `production` -> Nhấp **Approve and deploy**.
6. Quan sát Job `deploy-prod` tiếp tục chạy và hoàn tất với tích xanh ✅.
7. Lưu ảnh chụp màn hình vào thư mục `image/` để nộp bài.
