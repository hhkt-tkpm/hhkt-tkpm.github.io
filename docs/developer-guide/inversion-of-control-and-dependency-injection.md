---
sidebar_position: 8
---


# Inversion of Control and Dependency Injection

## Introduction

Inversion of Control (IoC) and Dependency Injection (DI) are core principles of the Spring Framework that enable loose coupling and testable code architecture. This document explains how IoC and DI are implemented in our Student Management System, focusing on practical patterns and real implementations from the codebase.

## Inversion of Control (IoC) Overview

### What is IoC?

IoC is a design principle where the control of object creation and lifecycle management is transferred from the application code to an external container (Spring IoC Container).

**Traditional Approach (Without IoC):**
```java
public class ProgramController {
    private IProgramService programService;
    
    public ProgramController() {
        this.programService = new ProgramServiceImpl(); // Tight coupling
    }
}
```

**IoC Approach (Our Implementation):**
```java
@RestController
@RequiredArgsConstructor
public class ProgramController {
    private final IProgramService programService; // Container provides implementation
}
```

### Spring IoC Container in Our System

The Spring container manages all components in our Student Management System:

```
Spring IoC Container
├── Controllers (ProgramController, StudentController, etc.)
├── Services (ProgramServiceImpl, StudentServiceImpl, etc.)
├── Repositories (IProgramRepository, IStudentRepository, etc.)
├── Validators (EmailDomainValidator, PhoneNumberValidator, etc.)
└── Filters (TranslationFilter)
```

## Dependency Injection Patterns

### 1. Constructor Injection (Primary Pattern)

Our system exclusively uses constructor injection for all controllers and services, ensuring immutable dependencies and fail-fast behavior.

**Implementation with Lombok:**
```java
@RestController
@RequestMapping("/api/programs")
@RequiredArgsConstructor  // Lombok generates constructor
@Slf4j
public class ProgramController {
    private final IProgramService programService;
    
    @PostMapping("")
    public APIResponse addProgram(@RequestBody @Valid ProgramRequest request) {
        ProgramResponse program = programService.addProgram(request);
        return APIResponse.builder()
                .status(HttpStatus.CREATED.value())
                .message("Success")
                .data(program)
                .build();
    }
}
```

**Service Layer Implementation:**
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class ProgramServiceImpl implements IProgramService {
    private final IProgramRepository programRepository;
    
    @Override
    public ProgramResponse addProgram(ProgramRequest request) {
        if (programRepository.findByProgramName(request.getProgramName()).isPresent()) {
            throw new RuntimeException("Program already exists");
        }
        // Business logic implementation
    }
}
```

**Why Constructor Injection:**
- **Immutability:** Dependencies cannot be changed after object creation
- **Required Dependencies:** Ensures all dependencies are provided at creation time
- **Testability:** Easy to mock dependencies in unit tests
- **Thread Safety:** Immutable fields are inherently thread-safe

### 2. Field Injection (Framework Constraints Only)

Used only when framework constraints prevent constructor injection:

**TranslationFilter Example:**
```java
@Component
@Order(1)
@Slf4j
public class TranslationFilter implements Filter {
    
    @Autowired
    private TranslationService translationService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    // Servlet Filter requires no-arg constructor
    // Framework instantiates with default constructor
}
```

**EmailDomainValidator Example:**
```java
@Component
public class EmailDomainValidator implements ConstraintValidator<EmailDomain, String> {

    private final IEmailDomainRepository emailDomainRepository;

    @Autowired  // Required for Bean Validation framework
    public EmailDomainValidator(IEmailDomainRepository emailDomainRepository) {
        this.emailDomainRepository = emailDomainRepository;
    }
}
```

## Component Architecture and Annotations

### Spring Stereotype Annotations in Our System

| Component Type         | Annotation        | Purpose                      | Examples from Our Code                      |
| ---------------------- | ----------------- | ---------------------------- | ------------------------------------------- |
| **Controllers**        | `@RestController` | HTTP request handling        | `ProgramController`                         |
| **Business Logic**     | `@Service`        | Service layer implementation | `ProgramServiceImpl`                        |
| **Data Access**        | `@Repository`     | Data access abstraction      | `IProgramRepository`                        |
| **General Components** | `@Component`      | Generic Spring beans         | `TranslationFilter`, `EmailDomainValidator` |

### Real Component Dependencies

**ProgramController Dependency Chain:**
```
ProgramController
     ↓ (depends on)
IProgramService
     ↓ (implemented by)
ProgramServiceImpl
     ↓ (depends on)  
IProgramRepository
     ↓ (auto-implemented by Spring Data JPA)
ProgramRepositoryImpl (Generated)
```

### Interface-Based Design Pattern

Our system follows interface-based dependency injection for loose coupling:

**Service Interface:**
```java
public interface IProgramService {
    ProgramResponse addProgram(ProgramRequest request);
    List<ProgramResponse> getAllPrograms();
    ProgramResponse getProgramById(Integer id);
    ProgramResponse updateProgram(Integer id, ProgramRequest request);
    void deleteProgram(Integer id);
}
```

**Benefits:**
- **Abstraction:** Controllers depend on interfaces, not implementations
- **Testability:** Easy to create mock implementations
- **Flexibility:** Can swap implementations without changing controllers
- **Maintainability:** Clear contracts between layers

## Repository Auto-Implementation

### Spring Data JPA Magic

```java
@Repository
public interface IProgramRepository extends JpaRepository<Program, Integer> {
    Optional<Program> findByProgramName(String programName);
}
```

**What Spring Does Automatically:**
1. **Proxy Creation:** Generates implementation at runtime
2. **Method Implementation:** Creates SQL queries from method names
3. **Transaction Management:** Handles database transactions
4. **Exception Translation:** Converts database exceptions to Spring exceptions

**Custom Query Methods:**
- `findByProgramName` → `SELECT * FROM programs WHERE program_name = ?`
- Spring generates implementation based on method naming conventions

## Custom Component Injection

### Validator Components

Custom validators demonstrate DI in specialized components:

```java
@Component
public class EmailDomainValidator implements ConstraintValidator<EmailDomain, String> {

    private final IEmailDomainRepository emailDomainRepository;

    @Autowired
    public EmailDomainValidator(IEmailDomainRepository emailDomainRepository) {
        this.emailDomainRepository = emailDomainRepository;
    }
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        // Validation logic using injected repository
    }
}
```

### Filter Components

```java
@Component
@Order(1)
@Slf4j
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

        // And so on with filter logic...
    }
}
```

## Configuration and Bean Definition

### Application Configuration

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public FilterRegistrationBean<TranslationFilter> translationFilterRegistration(
            TranslationFilter translationFilter) {
        FilterRegistrationBean<TranslationFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(translationFilter);
        registration.addUrlPatterns("/*");
        registration.setOrder(1);
        return registration;
    }
}
```

### Bean Scopes

| Scope                  | Description                   | Usage in Our System           |
| ---------------------- | ----------------------------- | ----------------------------- |
| `@Singleton` (default) | One instance per container    | Services, Repositories        |
| `@Prototype`           | New instance per request      | Not commonly used             |
| `@Request`             | One instance per HTTP request | Not used in our stateless API |
| `@Session`             | One instance per HTTP session | Not applicable                |

## Testing with Dependency Injection

### Unit Testing with Mocks

```java
@ExtendWith(MockitoExtension.class)
public class ProgramServiceTest {
    @Mock
    private IProgramRepository programRepository;

    @InjectMocks
    private ProgramServiceImpl programService;

    @Test
    public void shouldGetProgramById() {
        Program program = new Program(1, "Computer Science");

        when(programRepository.findById(1)).thenReturn(Optional.of(program));

        ProgramResponse programResponse = programService.getProgramById(1);

        assertThat(programResponse.getId()).isEqualTo(1);
        assertThat(programResponse.getProgramName()).isEqualTo("Computer Science");
    }
}
```

### Integration Testing

```java
@WebMvcTest(ProgramController.class)
@MockBean(JpaMetamodelMappingContext.class)
@Import(TestConfig.class)
public class ProgramControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProgramServiceImpl programService;

    @Test
    public void shouldGetProgramById() throws Exception {
        ProgramResponse program = ProgramResponse.builder().id(1).programName("Computer Science").build();

        when(programService.getProgramById(1)).thenReturn(program);

        mockMvc.perform(get("/api/programs/1"))
                .andExpect(jsonPath("$.status").value(HttpStatus.OK.value()))
                .andExpect(jsonPath("$.data.id").value(1))
                .andExpect(jsonPath("$.data.programName").value("Computer Science"));
    }
}
```

---

*This guide provides comprehensive coverage of dependency injection patterns used in the Student Management System. For practical implementation examples, refer to the source code in the respective packages.*