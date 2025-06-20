---
sidebar_position: 1
---

# Coding Standards

## General Principles

### Code Quality Principles
- **Broken Window Theory**: Fix small issues immediately to prevent larger problems
- **Boy Scout Rule**: Always leave the code cleaner than you found it
- **DRY (Don't Repeat Yourself)**: Avoid code duplication
- **SOLID Principles**: Follow Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion
- **Clean Code**: Write self-documenting, readable code

### Project Structure
- Maintain clear separation between client and server codebases
- Follow established architectural patterns (MVC for backend, component-based for frontend)
- Use consistent folder structure across similar modules

## Java Backend Standards

### Package Structure
````java
org.example.backend/
├── controller/          # REST controllers
├── service/            # Business logic
│   ├── impl/          # Service implementations
│   ├── export/        # Export services (PDF, Excel)
│   └── Import/        # Import services
├── repository/         # Data access layer
├── domain/            # Entity classes
├── dto/               # Data Transfer Objects
│   ├── request/       # Request DTOs
│   ├── response/      # Response DTOs
│   └── data/          # Data DTOs
├── mapper/            # Object mappers
├── config/            # Configuration classes
├── common/            # Common utilities
└── validator/         # Custom validators
````

### Naming Conventions
- **Classes**: PascalCase (e.g., `StudentController`, `TranslationService`)
- **Methods**: camelCase (e.g., `findStudentById`, `validateRequest`)
- **Variables**: camelCase (e.g., `studentId`, `courseCode`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_STUDENTS`, `DEFAULT_PAGE_SIZE`)
- **Packages**: lowercase (e.g., `controller`, `service.impl`)

### Service Layer Standards
````java
@Service
@Slf4j
public class StudentServiceImpl implements StudentService {
    
    // Use constructor injection
    private final StudentRepository studentRepository;
    private final ValidationService validationService;
    
    public StudentServiceImpl(StudentRepository studentRepository, 
                             ValidationService validationService) {
        this.studentRepository = studentRepository;
        this.validationService = validationService;
    }
    
    // Log important operations
    @Override
    public StudentResponse createStudent(CreateStudentRequest request) {
        log.info("Creating student with email: {}", request.getEmail());
        // Implementation
    }
}
````

### Controller Standards
````java
@RestController
@RequestMapping("/api/students")
@Validated
public class StudentController {
    
    // Use standard HTTP status codes
    @PostMapping
    public ResponseEntity<ApiResponse<StudentResponse>> createStudent(
            @Valid @RequestBody CreateStudentRequest request) {
        
        StudentResponse student = studentService.createStudent(request);
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(ApiResponse.success(student));
    }
    
    // Include pagination for list endpoints
    @GetMapping
    public ResponseEntity<ApiResponse<PagedResponse<StudentResponse>>> getAllStudents(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        // Implementation
    }
}
````

### Exception Handling
````java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ApiResponse<Object>> handleValidation(ValidationException ex) {
        return ResponseEntity.badRequest()
            .body(ApiResponse.error(ex.getMessage()));
    }
}
````

### Database Entity Standards
````java
@Entity
@Table(name = "students")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long studentId;
    
    @Column(name = "student_code", unique = true, nullable = false)
    private String studentCode;
    
    // Use proper JPA relationships
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "faculty_id")
    private Faculty faculty;
    
    // Include audit fields
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
````

## TypeScript Frontend Standards

### Project Structure
````
src/
├── app/                    # Next.js app directory
│   ├── [feature]/         # Feature-based modules
│   │   ├── components/    # Feature components
│   │   └── page.tsx      # Page component
├── components/            # Shared components
│   └── ui/               # UI components
├── interfaces/           # TypeScript interfaces
├── libs/                # Utilities and services
│   ├── api/             # API clients
│   ├── hooks/           # Custom hooks
│   ├── services/        # Business services
│   ├── stores/          # State management
│   ├── utils/           # Utility functions
│   └── validators/      # Form validators
````

### Naming Conventions
- **Components**: PascalCase (e.g., `StudentModal`, `CourseTable`)
- **Files**: kebab-case for utilities, PascalCase for components
- **Variables/Functions**: camelCase (e.g., `studentData`, `handleSubmit`)
- **Interfaces**: PascalCase with descriptive names (e.g., `StudentResponse`, `CreateCourseRequest`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`, `DEFAULT_PAGE_SIZE`)

### Component Standards
````tsx
// Use functional components with TypeScript
interface StudentModalProps {
    student?: Student;
    visible: boolean;
    onClose: () => void;
    onSubmit: (student: CreateStudentRequest) => void;
}

export const StudentModal: React.FC<StudentModalProps> = ({
    student,
    visible,
    onClose,
    onSubmit
}) => {
    const t = useTranslations('student-management');
    const [form] = Form.useForm();
    
    // Use proper effect dependencies
    useEffect(() => {
        if (student) {
            form.setFieldsValue(student);
        } else {
            form.resetFields();
        }
    }, [student, form]);
    
    const handleSubmit = async () => {
        try {
            const values = await form.validateFields();
            onSubmit(values);
        } catch (error) {
            console.error('Validation failed:', error);
        }
    };
    
    return (
        <Modal
            title={t(student ? 'edit-student' : 'add-student')}
            open={visible}
            onCancel={onClose}
            onOk={handleSubmit}
        >
            {/* Form implementation */}
        </Modal>
    );
};
````

### Custom Hooks Standards
````tsx
// Custom hooks should start with 'use'
export const useStudents = (params: StudentQueryParams) => {
    return useQuery({
        queryKey: ['students', params],
        queryFn: () => studentApi.getStudents(params),
        staleTime: 5 * 60 * 1000, // 5 minutes
    });
};

export const useCreateStudent = () => {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: studentApi.createStudent,
        onSuccess: () => {
            queryClient.invalidateQueries({ queryKey: ['students'] });
            message.success('Student created successfully');
        },
        onError: (error) => {
            message.error(`Failed to create student: ${error.message}`);
        },
    });
};
````

### Interface Standards
````tsx
// Use descriptive interface names
export interface Student {
    studentId: string;
    fullName: string;
    email: string;
    faculty: string;
    program: string;
    studentStatus: string;
    createdAt: string;
    updatedAt: string;
}

export interface CreateStudentRequest {
    fullName: string;
    email: string;
    facultyId: number;
    programId: number;
    // Only include required fields for creation
}

export interface StudentResponse extends Student {
    // Additional fields in response
    addresses: Address[];
    documents: Document[];
}
````

## Database Standards

### Naming Conventions
- **Tables**: plural, snake_case (e.g., `students`, `class_registrations`)
- **Columns**: snake_case (e.g., `student_id`, `created_at`)
- **Foreign Keys**: `[referenced_table]_id` (e.g., `faculty_id`, `course_id`)
- **Indexes**: `idx_[table]_[column(s)]` (e.g., `idx_students_email`)

### Migration Standards
````sql
-- V1_20250319081659_create_table.sql
-- Use descriptive migration names with timestamps

CREATE TABLE students (
    student_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    student_code VARCHAR(20) NOT NULL UNIQUE,
    full_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    faculty_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (faculty_id) REFERENCES faculties(faculty_id),
    INDEX idx_students_email (email),
    INDEX idx_students_faculty (faculty_id)
);
````

## API Standards

### Endpoint Naming
- Use RESTful conventions
- Use plural nouns for resources
- Use kebab-case for multi-word resources

````
GET    /api/students              # Get all students
POST   /api/students              # Create student
GET    /api/students/{id}         # Get student by ID
PUT    /api/students/{id}         # Update student
DELETE /api/students/{id}         # Delete student
GET    /api/students/search       # Search students
GET    /api/class-registrations   # Multi-word resource
````

### Response Format
````json
{
    "status": 200,
    "message": "Success",
    "data": {
        // Response data
    },
    "paginationInfo": {
        "page": 0,
        "size": 10,
        "totalElements": 100,
        "totalPages": 10,
        "hasNext": true,
        "hasPrevious": false
    }
}
````

### Error Response Format
````json
{
    "status": 400,
    "message": "Validation failed",
    "errors": [
        {
            "field": "email",
            "message": "Email domain is not allowed"
        }
    ]
}
````

## Testing Standards

### Unit Test Standards
````java
@ExtendWith(MockitoExtension.class)
class TranslationServiceTest {
    
    @Mock
    private ExternalTranslationApi translationApi;
    
    private TranslationService translationService;
    
    @BeforeEach
    void setUp() {
        translationService = new TranslationServiceImpl(translationApi);
    }
    
    @Test
    void translateJsonNode_WithObjectNode_ShouldTranslateAllTextFields() {
        // Given
        ObjectNode originalNode = objectMapper.createObjectNode();
        originalNode.put("name", "John Doe");
        originalNode.put("description", "Software Engineer");
        
        // When
        JsonNode result = translationService.translateJsonNode(originalNode, "en", "vi");
        
        // Then
        assertThat(result.get("description").asText()).isEqualTo("Kỹ sư phần mềm");
    }
}
````

### Test Naming
- Use descriptive test names: `methodName_WithCondition_ShouldExpectedBehavior`
- Group related tests in nested classes
- Use meaningful test data

## Documentation Standards

### Code Comments
````java
/**
 * Creates a new student in the system with full validation.
 * 
 * @param request Student creation request containing all required fields
 * @return Created student with generated ID and timestamps
 * @throws ValidationException if request data is invalid
 * @throws BusinessException if business rules are violated
 */
public StudentResponse createStudent(CreateStudentRequest request) {
    // Implementation
}
````

### API Documentation
- Use comprehensive Swagger/OpenAPI documentation
- Include request/response examples
- Document all validation rules
- Provide usage examples

### README Standards
- Include setup instructions
- Document system requirements
- Provide usage examples
- Include project structure overview

## Translation and Internationalization

### Translation Helper Standards
````typescript
// Use type-safe translation helpers
export async function translateResponse<T extends object>(
    data: T,
    typeName: string
): Promise<T> {
    const locale = Cookies.get('NEXT_LOCALE') || 'vi';
    if (locale !== 'en') return data;
    
    const fieldsToTranslate = getTranslatableFields(typeName);
    const result = { ...data };
    
    for (const key of fieldsToTranslate) {
        const value = (data as any)[key];
        if (typeof value === 'string') {
            const translated = await translateText(value, 'vi', 'en');
            (result as any)[key] = translated;
        }
    }
    
    return result;
}
````

### Translation Service Standards
- Cache translations to avoid repeated API calls
- Handle translation failures gracefully
- Support bidirectional translation (vi ↔ en)
- Use consistent field mapping for different entity types

### Best Practices Summary

1. **Consistency**: Follow established patterns across the codebase
2. **Type Safety**: Use TypeScript interfaces and Java generics
3. **Error Handling**: Implement comprehensive error handling
4. **Performance**: Consider caching, pagination, and lazy loading
5. **Security**: Validate all inputs and use proper authentication
6. **Maintainability**: Write clean, well-documented code
7. **Testing**: Maintain good test coverage
8. **Internationalization**: Support multiple languages properly

These coding standards should be followed by all team members to ensure consistency, maintainability, and quality across the Student Management System codebase.