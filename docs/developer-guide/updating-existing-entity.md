---
sidebar_position: 6
---

# Updating Existing Entities - Adding New Properties

This guide explains how to add new properties to existing entities in the HHKT-Ex-TKPM Spring Boot application. We'll use the `Student` entity as an example, but the same principles apply to other entities like `Course`, `Class`, `Program`, etc.

## Overview

The HHKT-Ex-TKPM system follows a layered architecture with:

- **Domain Layer**: JPA entities with database mappings
- **DTO Layer**: Data Transfer Objects for API communication
- **Service Layer**: Business logic implementation
- **Controller Layer**: REST API endpoints
- **Repository Layer**: Data access layer

## Step-by-Step Guide

### 1. Update the Domain Entity

Navigate to `server/src/main/java/org/example/backend/domain/` and update your entity class.

**Example: Adding `phoneNumber` property to Student entity**

```java
@Entity
@Table(name = "students")
public class Student {
    // ... existing properties

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "student_code")
    private String studentCode;

    @Column(name = "full_name")
    private String fullName;

    // NEW PROPERTY - Add this
    @Column(name = "phone_number", length = 15)
    private String phoneNumber;

    // ... existing properties

    // Add getter and setter for new property
    public String getPhoneNumber() {
        return phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }

    // ... existing getters and setters
}
```

**Important Considerations:**

- Use appropriate JPA annotations (`@Column`, `@NotNull`, etc.)
- Choose proper data types and constraints
- Follow naming conventions (camelCase for Java, snake_case for database)

### 2. Update Database Schema

Create or update migration files in `server/src/main/resources/db/`.

**Create a new migration file (e.g., `V2__add_phone_number_to_students.sql`):**

```sql
-- Add new column to existing table
ALTER TABLE students
ADD COLUMN phone_number VARCHAR(15);

-- Add index if needed for performance
CREATE INDEX idx_students_phone_number ON students(phone_number);

-- Add constraints if required
-- ALTER TABLE students ADD CONSTRAINT chk_phone_number_format
-- CHECK (phone_number REGEXP '^[0-9+\-\s()]+$');
```

**Alternative: Update existing schema file if in development:**

```sql
-- In your main schema file
CREATE TABLE students (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    student_code VARCHAR(20) NOT NULL UNIQUE,
    full_name VARCHAR(100) NOT NULL,
    phone_number VARCHAR(15), -- Add this line
    -- ... other columns
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 3. Update Data Transfer Objects (DTOs)

Update the relevant DTO classes in `server/src/main/java/org/example/backend/dto/`.

**Update Request DTO (`dto/request/StudentUpdateRequest.java`):**

```java
public class StudentUpdateRequest {
    // ... existing fields

    private String fullName;
    private String email;

    // NEW FIELD - Add this
    @Size(max = 15, message = "Phone number must not exceed 15 characters")
    @Pattern(regexp = "^[0-9+\\-\\s()]*$", message = "Invalid phone number format")
    private String phoneNumber;

    // ... existing fields

    // Add getter and setter
    public String getPhoneNumber() {
        return phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }

    // ... existing getters and setters
}
```

**Update Response DTO (`dto/response/StudentResponse.java`):**

```java
public class StudentResponse {
    // ... existing fields

    private Long id;
    private String studentCode;
    private String fullName;

    // NEW FIELD - Add this
    private String phoneNumber;

    // ... existing fields

    // Add getter and setter
    public String getPhoneNumber() {
        return phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }

    // ... existing getters and setters
}
```

### 4. Update Mapper Classes

Update the mapper in `server/src/main/java/org/example/backend/mapper/`.

**Update StudentMapper:**

```java
@Component
public class StudentMapper {

    public StudentResponse toResponse(Student student) {
        StudentResponse response = new StudentResponse();
        response.setId(student.getId());
        response.setStudentCode(student.getStudentCode());
        response.setFullName(student.getFullName());
        // NEW MAPPING - Add this
        response.setPhoneNumber(student.getPhoneNumber());
        // ... other mappings
        return response;
    }

    public Student toEntity(StudentCreateRequest request) {
        Student student = new Student();
        student.setStudentCode(request.getStudentCode());
        student.setFullName(request.getFullName());
        // NEW MAPPING - Add this
        student.setPhoneNumber(request.getPhoneNumber());
        // ... other mappings
        return student;
    }

    // ... other mapping methods
}
```

### 5. Update Service Layer

Update the service implementation in `server/src/main/java/org/example/backend/service/impl/`.

**Update StudentServiceImpl:**

```java
@Service
@Transactional
public class StudentServiceImpl implements StudentService {

    // ... existing methods

    @Override
    public StudentResponse updateStudent(Long id, StudentUpdateRequest request) {
        Student student = studentRepository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("Student not found with id: " + id));

        // Update existing properties
        Optional.ofNullable(request.getFullName()).ifPresent(student::setFullName);
        Optional.ofNullable(request.getEmail()).ifPresent(student::setEmail);

        // NEW PROPERTY UPDATE - Add this
        Optional.ofNullable(request.getPhoneNumber()).ifPresent(student::setPhoneNumber);

        // ... update other properties

        Student savedStudent = studentRepository.save(student);
        return studentMapper.toResponse(savedStudent);
    }

    // ... other methods
}
```

### 6. Update Validation (Optional)

Add custom validation if needed in `server/src/main/java/org/example/backend/validator/`.

**Create PhoneNumberValidator (if complex validation needed):**

```java
@Component
public class PhoneNumberValidator {

    public boolean isValid(String phoneNumber) {
        if (phoneNumber == null || phoneNumber.trim().isEmpty()) {
            return true; // Allow null/empty if not required
        }

        // Custom validation logic
        return phoneNumber.matches("^[0-9+\\-\\s()]{7,15}$");
    }

    public String sanitize(String phoneNumber) {
        return phoneNumber != null ? phoneNumber.trim() : null;
    }
}
```

### 7. Update Frontend (Client-side)

Update the React/NextJS components in `client/src/`.

**Update Student Interface (`interfaces/Student.ts`):**

```typescript
export interface Student {
  id: number;
  studentCode: string;
  fullName: string;
  phoneNumber?: string; // Add this field
  // ... other properties
}

export interface StudentUpdateRequest {
  fullName?: string;
  email?: string;
  phoneNumber?: string; // Add this field
  // ... other properties
}
```

**Update Student Form Component:**

```tsx
// In your student form component
<div className="form-group">
  <label htmlFor="phoneNumber">Phone Number</label>
  <input
    type="tel"
    id="phoneNumber"
    name="phoneNumber"
    value={formData.phoneNumber || ""}
    onChange={handleInputChange}
    placeholder="Enter phone number"
    maxLength={15}
  />
</div>
```

## Testing Your Changes

### 1. Unit Tests

Create/update unit tests in `server/src/test/java/org/example/backend/`:

```java
@ExtendWith(MockitoExtension.class)
class StudentServiceImplTest {

    @Test
    void shouldUpdateStudentPhoneNumber() {
        // Given
        Student existingStudent = new Student();
        existingStudent.setId(1L);
        existingStudent.setPhoneNumber("123456789");

        StudentUpdateRequest request = new StudentUpdateRequest();
        request.setPhoneNumber("987654321");

        when(studentRepository.findById(1L)).thenReturn(Optional.of(existingStudent));
        when(studentRepository.save(any(Student.class))).thenReturn(existingStudent);

        // When
        StudentResponse response = studentService.updateStudent(1L, request);

        // Then
        assertEquals("987654321", response.getPhoneNumber());
    }
}
```

### 2. Integration Tests

Test your API endpoints:

```java
@SpringBootTest
@AutoConfigureTestDatabase
class StudentControllerIntegrationTest {

    @Test
    void shouldUpdateStudentWithPhoneNumber() {
        // Create test data and verify API response includes new field
    }
}
```

### 3. Manual Testing

1. **Run the application:**

   ```bash
   cd server
   docker-compose up -d
   ./mvnw spring-boot:run
   ```

2. **Test via Swagger UI:**

   - Navigate to `http://localhost:9000/swagger-ui.html`
   - Test the update endpoint with the new field

3. **Test via Postman:**
   ```json
   PUT /api/students/1
   {
     "fullName": "Updated Name",
     "phoneNumber": "123-456-7890"
   }
   ```

## Best Practices

### 1. Database Considerations

- **Always use migrations** for schema changes in production
- **Add indexes** for frequently queried fields
- **Consider nullable vs non-null** constraints carefully
- **Use appropriate data types** and lengths

### 2. Validation

- **Add proper validation** at DTO level
- **Sanitize input data** before saving
- **Provide meaningful error messages**

### 3. Documentation

- **Update API documentation** (Swagger annotations)
- **Document business rules** for new fields
- **Update user guides** if applicable

### 4. Backward Compatibility

- **Make new fields optional** to avoid breaking existing API consumers
- **Provide default values** where appropriate
- **Version your APIs** if making breaking changes

## Common Issues and Troubleshooting

### Database Migration Issues

```bash
# If migration fails, check:
1. Database connection
2. Migration file syntax
3. Existing data conflicts
4. Permission issues
```

### Validation Errors

```java
// Common validation annotations
@NotNull(message = "Field is required")
@Size(min = 1, max = 50, message = "Length must be between 1 and 50")
@Pattern(regexp = "pattern", message = "Invalid format")
@Email(message = "Invalid email format")
```

### Mapping Issues

```java
// Ensure all fields are mapped in both directions
// Entity -> DTO
// DTO -> Entity
// Check for null handling
```

## Conclusion

Following this guide ensures that your new properties are properly integrated across all layers of the HHKT-Ex-TKPM application. Remember to:

1. ✅ Update the domain entity
2. ✅ Create database migration
3. ✅ Update DTOs and validation
4. ✅ Update service layer logic
5. ✅ Update mappers
6. ✅ Update frontend interfaces
7. ✅ Write tests
8. ✅ Test thoroughly

This systematic approach maintains code quality and ensures your new features work correctly across the entire application stack.
