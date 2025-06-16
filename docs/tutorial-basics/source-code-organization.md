---
sidebar_position: 2
---

# Source code organization

Tài liệu này là hướng dẫn cho các nhà phát triển về cấu trúc của dự án Quản lý sinh viên. Nó được thiết kế để giúp các nhà phát triển mới tìm hiểu về mã nguồn. Dự án được tổ chức thành các thư mục chính sau:

## 📁 Client (Frontend)

### 📂 src/

- **app/**

  - class-management/ - Quản lý lớp học
  - course-management/ - Quản lý khóa học
  - enroll-class/ - Đăng ký lớp học
  - reference-management/ - Quản lý tham chiếu
  - status-rules-configuration/ - Cấu hình quy tắc trạng thái
  - student-management/ - Quản lý sinh viên
  - transcript/ - Bảng điểm
  - layout.tsx - Layout chính của ứng dụng
  - page.tsx - Trang chủ

- **components/** - Chứa các component UI có thể tái sử dụng
- **interfaces/** - Định nghĩa các interface TypeScript
- **libs/**
  - api/ - Các hàm gọi API
  - hooks/ - Custom React hooks
  - services/ - Các service xử lý logic
  - stores/ - Quản lý state với Zustand
  - utils/ - Các hàm tiện ích
  - validators/ - Validate form với Zod/Yup

### 📄 Các file cấu hình

- package.json - Quản lý dependencies
- tsconfig.json - Cấu hình TypeScript
- next.config.ts - Cấu hình Next.js
- eslint.config.mjs - Cấu hình ESLint

## 📁 Server (Backend)

### 📂 src/main/java/org/example/backend/

- **common/** - Các class và utility dùng chung
- **config/** - Cấu hình ứng dụng
- **controller/** - Xử lý các request HTTP
- **domain/** - Các entity chính
- **dto/**
  - data/ - DTO cho dữ liệu
  - request/ - DTO cho request
  - response/ - DTO cho response
- **mapper/** - Chuyển đổi giữa entity và DTO
- **repository/** - Tương tác với database
- **service/**
  - Import/ - Xử lý import dữ liệu
  - export/ - Xử lý export dữ liệu
  - impl/ - Implement các service interface
- **validator/** - Validate dữ liệu

### 📂 src/main/resources/

- config/ - File cấu hình
- db/ - Script database
- font/ - Font chữ
- application.yml - Cấu hình Spring Boot

### 📂 src/test/

- controller/ - Test các controller
- service/ - Test các service
- validator/ - Test các validator

### 📄 Các file cấu hình

- pom.xml - Quản lý dependencies Maven
- docker-compose.yml - Cấu hình Docker
- application.yml - Cấu hình ứng dụng

## 📁 Docs

Chứa các tài liệu hướng dẫn:

- The Broken Window Theory & The Boy Scout Rule in Software Development.pdf
- Unit Test Coverage và Best Practices.pdf

## Giải thích chi tiết

### Client (Frontend)

Frontend được xây dựng bằng Next.js (React + TypeScript), sử dụng kiến trúc module hóa với các thư mục riêng biệt cho từng chức năng. Mỗi module trong thư mục `app` đại diện cho một chức năng chính của hệ thống.

### Server (Backend)

Backend được xây dựng bằng Spring Boot, tuân theo kiến trúc phân lớp rõ ràng:

- Controller: Xử lý request/response
- Service: Chứa business logic
- Repository: Tương tác với database
- DTO: Định nghĩa cấu trúc dữ liệu truyền nhận
- Domain: Định nghĩa các entity

### Testing

Mỗi layer đều có test riêng để đảm bảo chất lượng code:

- Unit test cho service và validator
- Integration test cho controller
- E2E test cho các luồng chính
