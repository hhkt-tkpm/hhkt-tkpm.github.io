---
sidebar_position: 1
---

# Getting Started with your app development

## 1. Công cụ cần thiết cho phát triển

### Backend (Spring Boot)

- [Docker](https://www.docker.com/) - Container platform
- [JDK 17+](https://adoptium.net/) - Java Development Kit
- [Maven](https://maven.apache.org/) - Build tool
- [IntelliJ IDEA](https://www.jetbrains.com/idea/) - IDE

### Frontend (Next.js)

- [Node.js](https://nodejs.org/) (v18+) - JavaScript runtime
- [Visual Studio Code](https://code.visualstudio.com/) - Code editor
- npm hoặc yarn - Package manager

## 2. Cài đặt và chạy dự án

### Bước 1: Clone source code

```bash
git clone https://github.com/nguyen-anh-hao/HHKT-Ex-TKPM.git
cd HHKT-Ex-TKPM
```

### Bước 2: Cài đặt và chạy Backend

1. **Khởi động Docker & database**:

```bash
cd server
docker-compose up -d
```

💡 Sau khi container chạy thành công, seed database bằng các file .sql trong `src/main/resources/db`.

2. **Chạy ứng dụng Spring Boot**:

- Mở project trong IntelliJ IDEA
- Tìm và mở file `BackendApplication.java`
- Bấm nút Run hoặc sử dụng terminal:

```bash
mvn spring-boot:run
```

### Bước 3: Cài đặt và chạy Frontend

1. **Cài đặt dependencies**:

```bash
cd client
npm install
```

2. **Chạy ứng dụng Next.js**:

```bash
npm run dev
```

### Bước 4: Truy cập ứng dụng

- Frontend: http://localhost:3000
- Backend API: http://localhost:9000
- Swagger UI: http://localhost:9000/swagger-ui.html

## 3. Cấu trúc Source Code

### Frontend (client/)

```
client/
├── src/
│   ├── app/                    # Các chức năng chính
│   │   ├── class-management/   # Quản lý lớp học
│   │   ├── course-management/  # Quản lý khóa học
│   │   ├── enroll-class/       # Đăng ký lớp học
│   │   ├── reference-management/# Quản lý tham chiếu
│   │   ├── status-rules-configuration/ # Cấu hình quy tắc
│   │   ├── student-management/ # Quản lý sinh viên
│   │   └── transcript/         # Bảng điểm
│   ├── components/            # UI components
│   ├── interfaces/            # TypeScript interfaces
│   └── libs/                  # Utilities
│       ├── api/              # API calls
│       ├── hooks/            # Custom hooks
│       ├── services/         # Business logic
│       ├── stores/           # State management
│       ├── utils/            # Helper functions
│       └── validators/       # Form validation
```

### Backend (server/)

```
server/
├── src/
│   ├── main/
│   │   ├── java/org/example/backend/
│   │   │   ├── common/        # Shared utilities
│   │   │   ├── config/        # Configuration
│   │   │   ├── controller/    # API endpoints
│   │   │   ├── domain/        # Entities
│   │   │   ├── dto/          # Data transfer objects
│   │   │   │   ├── data/     # Data DTOs
│   │   │   │   ├── request/  # Request DTOs
│   │   │   │   └── response/ # Response DTOs
│   │   │   ├── mapper/       # Object mapping
│   │   │   ├── repository/   # Data access
│   │   │   ├── service/      # Business logic
│   │   │   │   ├── Import/   # Import services
│   │   │   │   ├── export/   # Export services
│   │   │   │   └── impl/     # Service implementations
│   │   │   └── validator/    # Data validation
│   │   └── resources/
│   │       ├── config/       # Configuration files
│   │       ├── db/          # Database scripts
│   │       └── application.yml # Main config
│   └── test/                # Unit tests
```

## 4. Các chức năng chính

### Quản lý sinh viên

- Xem danh sách sinh viên
- Thêm, sửa, xóa sinh viên
- Quản lý thông tin chi tiết

### Quản lý danh mục

- Khoa
- Chương trình
- Trạng thái sinh viên
- Domain email

### Quản lý khóa học

- Thêm, sửa, xóa môn học
- Quản lý thông tin chi tiết

### Quản lý lớp học

- Tạo và quản lý lớp học
- Phân công giảng viên
- Quản lý lịch học

### Đăng ký lớp học

- Đăng ký/hủy đăng ký
- Xem lịch học
- Quản lý điểm số

### Bảng điểm

- Xem bảng điểm
- Nhập điểm
- Tính điểm trung bình

### Cấu hình quy tắc

- Quy tắc chuyển trạng thái
- Quy tắc tính điểm
- Quy tắc đăng ký

### Đa ngôn ngữ (v6.0)

- Hỗ trợ nhiều ngôn ngữ
- Chuyển đổi ngôn ngữ dễ dàng
