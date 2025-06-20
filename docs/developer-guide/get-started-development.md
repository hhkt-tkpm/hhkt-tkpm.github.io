---
sidebar_position: 4
---

# Getting Started with Your App Development

## Required Tools for Development

### Backend (Spring Boot)

- [Docker](https://www.docker.com/) - Container platform
- [JDK 17+](https://adoptium.net/) - Java Development Kit
- [Maven](https://maven.apache.org/) - Build tool
- [IntelliJ IDEA](https://www.jetbrains.com/idea/) - IDE

### Frontend (Next.js)

- [Node.js](https://nodejs.org/) (v18+) - JavaScript runtime
- [Visual Studio Code](https://code.visualstudio.com/) - Code editor
- npm or yarn - Package manager

## Install and Run the Project

### Step 1: Clone the Source Code

```bash
git clone https://github.com/nguyen-anh-hao/HHKT-Ex-TKPM.git
cd HHKT-Ex-TKPM
```

### Step 2: Install and Run Backend

1. **Start Docker & database**:

```bash
cd server
docker-compose up -d
```

💡 After the container runs successfully, seed the database using the .sql files in `src/main/resources/db`.

2. **Run the Spring Boot application**:

- Open the project in IntelliJ IDEA
- Find and open the `BackendApplication.java` file
- Click the Run button or use the terminal:

```bash
mvn spring-boot:run
```

### Step 3: Install and Run Frontend

1. **Install dependencies**:

```bash
cd client
npm install
```

2. **Run the Next.js application**:

```bash
npm run dev
```

### Step 4: Access the Application

- Frontend: http://localhost:3000
- Backend API: http://localhost:9000
- Swagger UI: http://localhost:9000/swagger-ui.html

## Source Code Structure

### Frontend (client/)

```
client/
├── src/
│   ├── app/                    # Main features
│   │   ├── class-management/   # Class management
│   │   ├── course-management/  # Course management
│   │   ├── enroll-class/       # Class enrollment
│   │   ├── reference-management/# Reference management
│   │   ├── status-rules-configuration/ # Rules configuration
│   │   ├── student-management/ # Student management
│   │   └── transcript/         # Transcript
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

## Main Features

### Student Management

- View student list
- Add, edit, delete students
- Manage detailed information

### Category Management

- Faculty
- Program
- Student status
- Email domain

### Course Management

- Add, edit, delete courses
- Manage detailed information

### Class Management

- Create and manage classes
- Assign lecturers
- Manage class schedules

### Class Enrollment

- Register/unregister for classes
- View class schedules
- Manage grades

### Transcript

- View transcript
- Enter grades
- Calculate GPA

### Rules Configuration

- Status transition rules
- Grading rules
- Enrollment rules

### Multi-language Support (v6.0)

- Support for multiple languages
- Easy language switching

