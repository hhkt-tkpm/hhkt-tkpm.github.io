# Data Validation

## Introduction

Data validation is a critical aspect of the Student Management System that ensures data integrity, security, and business rule compliance. This document covers the comprehensive validation strategy implemented using Jakarta Bean Validation (formerly JSR-303) and custom validation logic.

## Validation Architecture

### Multi-Layer Validation Strategy

```
┌─────────────────────────────────────┐
│        Client-Side Validation       │
│         (Frontend - Optional)       │
├─────────────────────────────────────┤
│       Controller Validation         │
│      (Request DTO Validation)       │
├─────────────────────────────────────┤
│        Service Validation           │
│     (Business Logic Validation)     │
├─────────────────────────────────────┤
│       Database Validation           │
│      (Constraints & Triggers)       │
└─────────────────────────────────────┘
```

### Validation Flow Diagram

```
[DIAGRAM_PLACEHOLDER_VALIDATION_FLOW]
```

## Jakarta Bean Validation

### Core Validation Annotations

| Annotation  | Purpose                       | Usage                |
| ----------- | ----------------------------- | -------------------- |
| `@NotNull`  | Null check                    | Required fields      |
| `@NotBlank` | Null, empty, whitespace check | String fields        |
| `@NotEmpty` | Null and empty check          | Collections, arrays  |
| `@Size`     | Length constraints            | Strings, collections |
| `@Min/@Max` | Numeric range validation      | Numbers              |
| `@Email`    | Email format validation       | Email fields         |
| `@Pattern`  | Regex pattern matching        | Custom formats       |
| `@Valid`    | Nested object validation      | Complex objects      |

### Request DTO Validation

**Example from ProgramRequest:**
```java
@Getter
@Setter
@Builder
public class ProgramRequest {
    @NotBlank(message = "Program name is required")
    private String programName;
}
```

**Extended Example for Student Registration:**
```java
public class StudentRequest {
    @NotNull(message = "Student id is required")
    private String studentId;

    @NotBlank(message = "Full name is required")
    private String fullName;

    @NotNull(message = "Date of birth is required")
    private LocalDate dob;

    @NotBlank(message = "Gender is required")
    private String gender;

    @NotBlank(message = "Intake is required")
    private String intake;

    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    @EmailDomain
    private String email;

    @NotBlank(message = "Phone country is required")
    private String phoneCountry;

    @NotBlank(message = "Phone number is required")
    private String phone;

    @NotBlank(message = "Nationality is required")
    private String nationality;

    @NotNull(message = "Faculty id is required")
    private Integer facultyId;

    @NotNull(message = "Program id is required")
    private Integer programId;

    @NotNull(message = "Student status id is required")
    private Integer studentStatusId;

    private List<AddressRequest> addresses;

    private List<DocumentRequest> documents;
}
```

### Controller-Level Validation

**Implementation in ProgramController:**
```java
@PostMapping("")
public APIResponse addProgram(@RequestBody @Valid ProgramRequest request) {
    log.info("Received request to add program: {}", request.getProgramName());
    
    ProgramResponse program = programService.addProgram(request);
    
    return APIResponse.builder()
            .status(HttpStatus.CREATED.value())
            .message("Success")
            .data(program)
            .build();
}
```

**Key Points:**
- `@Valid` annotation triggers validation
- Validation occurs before method execution
- Automatic error response generation for validation failures

## Custom Validation

### Custom Annotation Creation

**EmailDomain Annotation:**
```java
@Constraint(validatedBy = EmailDomainValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface EmailDomain {
    String message() default "Invalid email domain";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

### Custom Validator Implementation

**EmailDomainValidator:**
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
        if (email == null) {
            return true; // Let @NotNull handle null validation
        }

        String domain = email.substring(email.indexOf('@') + 1);
        
        List<org.example.backend.domain.EmailDomain> allowedDomains = 
            emailDomainRepository.findAll();

        return allowedDomains.stream()
                .anyMatch(allowedDomain -> domain.equals(allowedDomain.getDomain()));
    }
}
```

### Usage in Request DTOs

```java
[CODE_PLACEHOLDER_CUSTOM_VALIDATION_USAGE]
```

## Service Layer Validation

### Business Logic Validation

**Example from ProgramServiceImpl:**
```java
@Override
public ProgramResponse addProgram(ProgramRequest request) {
    // Business rule validation
    if (programRepository.findByProgramName(request.getProgramName()).isPresent()) {
        log.error("Program already exists");
        throw new RuntimeException("Program already exists");
    }

    Program program = ProgramMapper.mapToDomain(request);
    program = programRepository.save(program);

    log.info("Program saved to database successfully");
    return ProgramMapper.mapToResponse(program);
}
```

### Complex Business Validation

```java
@Override
@Transactional
public ClassRegistrationResponse addClassRegistration(ClassRegistrationRequest classRegistrationRequest) {

    log.info("add class registration with request: {}", classRegistrationRequest);

    ClassRegistration classRegistration = ClassRegistrationMapper.mapFromClassRegistrationRequestToDomain(classRegistrationRequest);

    log.info("check if class exists");
    Class aClass = classRepository.findById(classRegistrationRequest.getClassId())
            .orElseThrow(() -> {
                log.error("Class not found");
                return new RuntimeException("Class not found");
            });

    log.info("check if the course of the class is active");
    if (!aClass.getCourse().getIsActive()) {
        log.error("Course of the class is not active");
        throw new RuntimeException("Course of the class is not active");
    }

    log.info("check if max students reached");
    if (aClass.getClassRegistrations().size() >= aClass.getMaxStudents()) {
        log.error("Max students reached");
        throw new RuntimeException("Max students reached");
    }

    log.info("check if student exists");
    Student student = studentRepository.findById(classRegistrationRequest.getStudentId())
            .orElseThrow(() -> {
                log.error("Student not found");
                return new RuntimeException("Student not found");
            });

    log.info("set class and student");
    classRegistration.setAClass(aClass);
    classRegistration.setStudent(student);
    classRegistration = classRegistrationRepository.save(classRegistration);

    log.info("class registration created successfully");

    // Add the registration history
    log.info("Add class registration history");
    ClassRegistrationHistoryRequest classRegistrationHistoryRequest = ClassRegistrationHistoryMapper.mapFromClassRegistrationDomainToClassRegistrationHistoryRequest(classRegistration);
    classRegistrationHistoryRequest.setReason("Class registration created");

    classRegistrationHistoryService.addClassRegistrationHistory(classRegistrationHistoryRequest);
    log.info("class registration history added successfully");

    return ClassRegistrationMapper.mapFromDomainToClassRegistrationResponse(classRegistration);
}
```

## Error Handling and Response

### Validation Error Structure

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "programName",
      "message": "Program name is required",
      "rejectedValue": null
    }
  ]
}
```

## Database-Level Validation

### Entity Constraints

**Program Entity with Constraints:**
```java
@Entity
@Table(name = "programs")
public class Program extends Auditable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "program_name", nullable = false, unique = true, length = 100)
    private String programName;
    
    // Additional constraints
}
```

### Database Migration Constraints

```sql
ALTER TABLE addresses
    ADD CONSTRAINT addresses_address_type_check
    CHECK (address_type IN ('Thường Trú', 'Tạm Trú', 'Nhận Thư'));
```

## Validation Patterns by Entity

### Student Validation

```java
public class Student extends Auditable {
    @Id
    @Column(name = "student_id", length = 10)
    private String studentId;

    @Column(name = "full_name", nullable = false)
    private String fullName;

    @Column(name = "dob", nullable = false)
    private LocalDate dob;

    @Column(name = "gender", nullable = false)
    private String gender;

    // Additional fields with validation
}
```

### Registration Validation

```java
public class ClassRegistration extends Auditable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "status", nullable = false)
    @Enumerated(EnumType.STRING)
    private RegistrationStatus status;
}
```

## Testing Validation Logic

### Unit Testing Validators

```java
@ExtendWith(MockitoExtension.class)
public class EmailDomainValidatorTest {

    @Test
    public void shouldReturnFalseWhenDomainIsNotAllowed() {
        List<EmailDomain> allowedDomains = Arrays.asList(
                createEmailDomain("gmail.com"),
                createEmailDomain("yahoo.com"),
                createEmailDomain("hotmail.com")
        );

        when(emailDomainRepository.findAll()).thenReturn(allowedDomains);

        assertFalse(validator.isValid("test@outlook.com", context));
        assertFalse(validator.isValid("user@example.com", context));
    }
}
```

### Controller Testing with Invalid Data

```java
@Test
public void givenProgramIdNotFound_whenGetProgramById_shouldThrowException() {
    when(programRepository.findById(1)).thenReturn(Optional.empty());

    RuntimeException exception = assertThrows(RuntimeException.class, () -> {
        programService.getProgramById(1);
    });

    assertThat(exception.getMessage()).isEqualTo("Program not found");
}
```

## Migration and Upgrade Path

### Jakarta Migration (JSR-303 to Jakarta)

```java
// Old (JSR-303)
import javax.validation.constraints.NotBlank;

// New (Jakarta)
import jakarta.validation.constraints.NotBlank;
```

### Validation Framework Updates

- Bean Validation specification changes
- Custom validator compatibility
- Message interpolation updates
- Performance improvements

---

*This comprehensive guide covers all aspects of data validation in the Student Management System. For implementation details and examples, refer to the source code in the respective packages and the accompanying test cases.*