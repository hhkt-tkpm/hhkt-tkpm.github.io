---
sidebar_position: 2
---

# Backend Architecture Overview

## Introduction

This document provides a comprehensive overview of the backend architecture for the Student Management System built with Spring Boot. The system follows a layered architecture pattern with clear separation of concerns, ensuring maintainability, scalability, and testability.

## System Architecture

### Technology Stack

| Component         | Technology                               | Purpose                                    |
| ----------------- | ---------------------------------------- | ------------------------------------------ |
| **Framework**     | Spring Boot 3.x                          | Core application framework                 |
| **Build Tool**    | Maven                                    | Dependency management and build automation |
| **Database**      | PostgreSQL                               | Primary data storage                       |
| **ORM**           | Spring Data JPA/Hibernate                | Object-relational mapping                  |
| **Validation**    | Jakarta Validation (Bean Validation 3.0) | Data validation                            |
| **Documentation** | OpenAPI/Swagger                          | API documentation                          |
| **Testing**       | JUnit 5, Mockito                         | Unit testing                               |

### Architectural Layers

The application follows a **4-tier layered architecture**:

![Layered Architecture Diagram](https://i.ibb.co/prGXtgWB/4Layer.png)

#### 1. Presentation Layer (Controller)

The presentation layer handles HTTP requests and responses, providing RESTful API endpoints.

**Key Characteristics:**
- RESTful API design following HTTP standards
- Request/Response mapping using DTOs
- Input validation using Bean Validation
- Consistent error handling and response formatting
- Cross-cutting concerns handled by filters and interceptors

**Example Structure:**
```java
public class ProgramController {
    private final IProgramService programService;

    @GetMapping("")
    public APIResponse getAllPrograms() {
        log.info("Received request to get all programs");

        return APIResponse.builder()
                .status(HttpStatus.OK.value())
                .message("Success")
                .data(programService.getAllPrograms())
                .build();
    }
}
```

#### 2. Business Logic Layer (Service)

The service layer contains the core business logic and orchestrates operations between different components.

**Key Characteristics:**
- Business rule implementation
- Transaction management
- Cross-entity operations
- Data transformation between DTOs and entities
- Exception handling for business scenarios

**Example Structure:**
```java
public class ProgramServiceImpl implements IProgramService {
    private final IProgramRepository programRepository;

    @Override
    public List<ProgramResponse> getAllPrograms() {
        List<Program> programs = programRepository.findAll();

        log.info("Retrieved all programs from database");

        return programs.stream()
                .map(ProgramMapper::mapToResponse)
                .collect(Collectors.toList());
    }
}
```

#### 3. Data Access Layer (Repository)

The repository layer provides abstraction over data access operations using Spring Data JPA.

**Key Characteristics:**
- CRUD operations
- Custom query methods
- Database transaction management
- Entity relationship handling

**Example Structure:**
```java
@Repository
public interface IProgramRepository extends JpaRepository<Program, Integer> {
    Optional<Program> findByProgramName(String programName);
}
```

#### 4. Data Layer (Database)

PostgreSQL database for schema management.

**Database Design:**

![Database Schema Diagram](https://i.ibb.co/3mNKshNr/Database.png)

## Core Components

### Domain Entities

The system manages several core entities with well-defined relationships:

| Entity           | Purpose                  | Key Relationships              |
| ---------------- | ------------------------ | ------------------------------ |
| **Program**      | Academic programs/majors | One-to-Many with Students      |
| **Student**      | Student information      | Many-to-One with Program       |
| **Faculty**      | Teaching staff           | One-to-Many with Courses       |
| **Course**       | Academic courses         | Many-to-One with Faculty       |
| **Class**        | Course instances         | Many-to-One with Course        |
| **Registration** | Student enrollments      | Many-to-One with Student/Class |

### Data Transfer Objects (DTOs)

**Request DTOs:** Handle incoming data with validation annotations

**Response DTOs:** Structure outgoing data with necessary fields

### Mappers

Utility classes for converting between entities and DTOs, ensuring clean separation between layers.

```java
public class ProgramMapper {
    public static Program mapToDomain(ProgramRequest request) {
        return Program.builder()
                .programName(request.getProgramName())
                .build();
    }

    public static ProgramResponse mapToResponse(Program program) {
        return ProgramResponse.builder()
                .id(program.getId())
                .programName(program.getProgramName())
                .createdAt(program.getCreatedAt())
                .updatedAt(program.getUpdatedAt())
                .createdBy(program.getCreatedBy())
                .updatedBy(program.getUpdatedBy())
                .build();
    }
}
```

## Cross-Cutting Concerns

### Translation Filter

The system implements internationalization through a custom `TranslationFilter`:

**Features:**
- Automatic request/response translation
- Language detection from Accept-Language header
- JSON content translation
- Support for Vietnamese (vi) and English (en)

```java
public class TranslationFilter implements Filter {

    public static final ThreadLocal<String> CURRENT_LANGUAGE = new ThreadLocal<>();
    public static final String DEFAULT_LANGUAGE = "vi";

    @Autowired
    private TranslationService translationService;

    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        // And so on...
    }
}
```

### Auditing

All entities extend `Auditable` class providing:
- Creation timestamp and user
- Last modification timestamp and user
- Automatic audit field population

### Validation

**Built-in Validations:**
- Jakarta Bean Validation annotations
- Custom validators for business rules
- Email domain validation
- Phone number format validation

## API Design Principles

### RESTful Conventions

| HTTP Method | Purpose                  | Example Endpoint            |
| ----------- | ------------------------ | --------------------------- |
| GET         | Retrieve resources       | `GET /api/programs`         |
| POST        | Create new resource      | `POST /api/programs`        |
| PUT         | Update existing resource | `PUT /api/programs/{id}`    |
| DELETE      | Remove resource          | `DELETE /api/programs/{id}` |

### Response Format

All API responses follow a consistent structure:

```json
{
  "status": 200,
  "message": "Success",
  "data": { ... }
}
```

### Pagination

API endpoints that return collections support pagination:

**Request Parameters:**
- `page` - Zero-based page index (default: 0)
- `size` - Page size (default: 10)
- `sort` - Sorting criteria (format: `property,direction`, e.g., `name,asc`)

**Implementation:**
```java
public class PaginationInfo {
    private int currentPage;
    private int pageSize;
    private long totalItems;
    private int totalPages;

    public PaginationInfo(Page<?> page) {
        this.currentPage = page.getNumber();
        this.pageSize = page.getSize();
        this.totalItems = page.getTotalElements();
        this.totalPages = page.getTotalPages();
    }
}
```

### Error Handling
Standardized error responses with appropriate HTTP status codes:

| Status Code | Scenario               | Response Structure                         |
| ----------- | ---------------------- | ------------------------------------------ |
| 400         | Validation errors      | Error details with field-specific messages |
| 404         | Resource not found     | Error message indicating missing resource  |
| 500         | Internal server errors | Generic error message                      |

## Security Considerations

### Input Validation
- All user inputs validated at controller level
- Custom validators for business-specific rules
- SQL injection prevention through JPA/Hibernate

### Data Integrity
- Database constraints enforcement
- Transaction management for data consistency

## Testing Strategy

### Unit Testing
- Service layer testing with mocked dependencies
- Validator testing with isolated test cases

### Integration Testing
- Controller testing with @WebMvcTest
- End-to-end API testing

## Future Enhancements

### Scalability Considerations
- Microservice decomposition possibilities
- Horizontal scaling strategies
- Database sharding considerations

### Technology Upgrades
- Spring Boot version migration path
- Java version compatibility
- Database technology alternatives

---

*This document serves as a comprehensive guide to understanding the backend architecture. For specific implementation details, refer to the source code and accompanying technical documentation.*