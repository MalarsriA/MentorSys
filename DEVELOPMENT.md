# Development Guide - MentorSys

A comprehensive step-by-step guide for developing MentorSys using TDD approach with Maven.

## Table of Contents

1. [Environment Setup](#environment-setup)
2. [Project Structure](#project-structure)
3. [TDD Workflow](#tdd-workflow)
4. [Phase 1: Foundation (Week 1-2)](#phase-1-foundation)
5. [Phase 2: Core Features (Week 3-4)](#phase-2-core-features)
6. [Phase 3: Real-time Features (Week 5-6)](#phase-3-real-time-features)
7. [Phase 4: Advanced Features (Week 7-8)](#phase-4-advanced-features)
8. [Best Practices](#best-practices)

---

## Environment Setup

### 1. Install Prerequisites

```bash
# Check Java version (17+)
java -version

# Check Maven version (3.8+)
mvn -version

# For PostgreSQL (Development can use H2)
# Install from: https://www.postgresql.org/download/
```

### 2. Configure IDE

**VS Code:**
```bash
# Install Extensions
# - Extension Pack for Java
# - Spring Boot Extension Pack
# - REST Client
```

**IntelliJ IDEA:**
```
Settings → Plugins → Install:
  - Spring Boot
  - Database Tools and SQL
  - Maven Helper
```

### 3. Clone and Setup

```bash
# Clone repository
git clone https://github.com/MalarsriA/MentorSys.git
cd MentorSys

# Install dependencies
mvn clean install

# Verify setup
mvn test
```

### 4. Application Properties

Create `src/main/resources/application.yml`:

```yaml
spring:
  application:
    name: mentorsys
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
  h2:
    console:
      enabled: true
  datasource:
    url: jdbc:h2:mem:mentorsysdb
    driverClassName: org.h2.Driver
    username: sa
    password:

server:
  port: 8080
  servlet:
    context-path: /api

logging:
  level:
    root: INFO
    com.mentorsys: DEBUG
```

---

## Project Structure

```
src/main/java/com/mentorsys/
├── MentorSysApplication.java         # Main application class
├── config/
│   ├── SecurityConfig.java           # Spring Security configuration
│   ├── WebSocketConfig.java          # WebSocket configuration
│   └── JwtConfig.java                # JWT configuration
├── controller/
│   ├── UserController.java
│   ├── MentorController.java
│   ├── SessionController.java
│   └── NotificationController.java
├── service/
│   ├── UserService.java
│   ├── UserServiceImpl.java
│   ├── MentorService.java
│   ├── MentorServiceImpl.java
│   ├── SessionService.java
│   ├── SessionServiceImpl.java
│   ├── NotificationService.java
│   └── NotificationServiceImpl.java
├── repository/
│   ├── UserRepository.java
│   ├── MentorRepository.java
│   ├── SessionRepository.java
│   └── NotificationRepository.java
├── entity/
│   ├── User.java
│   ├── Mentor.java
│   ├── Session.java
│   ├── Notification.java
│   └── BaseEntity.java
├── dto/
│   ├── UserDTO.java
│   ├── UserRegistrationDTO.java
│   ├── MentorDTO.java
│   ├── SessionDTO.java
│   └── NotificationDTO.java
├── exception/
│   ├── ResourceNotFoundException.java
│   ├── DuplicateResourceException.java
│   ├── UnauthorizedException.java
│   └── GlobalExceptionHandler.java
├── security/
│   ├── JwtTokenProvider.java
│   ├── JwtAuthenticationFilter.java
│   ├── CustomUserDetailsService.java
│   └── SecurityUtils.java
└── websocket/
    ├── WebSocketEventListener.java
    ├── NotificationHandler.java
    └── MessageController.java

src/test/java/com/mentorsys/
├── controller/
│   └── UserControllerTest.java
├── service/
│   └── UserServiceTest.java
└── repository/
    └── UserRepositoryTest.java
```

---

## TDD Workflow

### The Red-Green-Refactor Cycle

```
1. RED: Write a failing test
   ├─ Define expected behavior
   ├─ Test should fail (feature doesn't exist)
   └─ Commit test: git commit -m "test: red test for feature X"

2. GREEN: Write minimal code to pass test
   ├─ Implement only what's needed
   ├─ All tests pass
   └─ Commit code: git commit -m "feat: implement feature X"

3. REFACTOR: Improve code quality
   ├─ Refactor for readability
   ├─ Refactor for performance
   ├─ All tests still pass
   └─ Commit refactor: git commit -m "refactor: improve feature X"
```

### Test Naming Convention

```
ClassName + Test.java

Examples:
- UserServiceTest.java
- UserControllerTest.java
- UserRepositoryTest.java
```

### Test Method Naming

```
test + (Given/When/Then) + Scenario

Examples:
- testCreateUserWithValidData()
- testCreateUserWithDuplicateEmail()
- testUpdateUserWithInvalidId()
- testDeleteUserWhenUserExists()
- testGetUserByEmailWhenEmailNotFound()
```

### Test Structure (AAA Pattern)

```java
@Test
void testScenarioDescription() {
    // Arrange: Set up test data
    User user = new User("john@example.com", "John", "password123");
    
    // Act: Execute the method under test
    User savedUser = userService.createUser(user);
    
    // Assert: Verify the result
    assertNotNull(savedUser.getId());
    assertEquals("john@example.com", savedUser.getEmail());
}
```

---

## Phase 1: Foundation (Week 1-2)

### Objectives
- ✅ Set up project structure
- ✅ Create base entities
- ✅ Implement basic CRUD operations
- ✅ Learn TDD approach

### Step 1.1: Create Base Entity

**File:** `src/main/java/com/mentorsys/entity/BaseEntity.java`

```java
@MappedSuperclass
@Data
public abstract class BaseEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @CreationTimestamp
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(nullable = false)
    private LocalDateTime updatedAt;
}
```

### Step 1.2: Create User Entity

**File:** `src/main/java/com/mentorsys/entity/User.java`

```java
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User extends BaseEntity {
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String firstName;
    
    @Column(nullable = false)
    private String lastName;
    
    @Column(nullable = false)
    private String password;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserRole role;
    
    @Column(nullable = false)
    private Boolean active = true;
}
```

### Step 1.3: Write First Test (RED)

**File:** `src/test/java/com/mentorsys/service/UserServiceTest.java`

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserServiceImpl userService;
    
    @Test
    void testCreateUserWithValidData() {
        // Arrange
        UserDTO userDTO = new UserDTO(
            "john@example.com",
            "John",
            "Doe",
            "password123"
        );
        
        User expectedUser = new User(
            "john@example.com",
            "John",
            "Doe",
            "encodedPassword",
            UserRole.MENTEE,
            true
        );
        
        when(userRepository.save(any(User.class))).thenReturn(expectedUser);
        
        // Act
        UserDTO result = userService.createUser(userDTO);
        
        // Assert
        assertNotNull(result);
        assertEquals("john@example.com", result.getEmail());
        verify(userRepository, times(1)).save(any(User.class));
    }
}
```

Run test (will FAIL):
```bash
mvn test -Dtest=UserServiceTest#testCreateUserWithValidData
```

### Step 1.4: Implement Service (GREEN)

**File:** `src/main/java/com/mentorsys/service/UserServiceImpl.java`

```java
@Service
@Slf4j
@Transactional
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    public UserDTO createUser(UserDTO userDTO) {
        
        // Validate email not already exists
        if (userRepository.existsByEmail(userDTO.getEmail())) {
            throw new DuplicateResourceException("Email already exists");
        }
        
        // Create new user
        User user = new User();
        user.setEmail(userDTO.getEmail());
        user.setFirstName(userDTO.getFirstName());
        user.setLastName(userDTO.getLastName());
        user.setPassword(passwordEncoder.encode(userDTO.getPassword()));
        user.setRole(UserRole.MENTEE);
        user.setActive(true);
        
        User savedUser = userRepository.save(user);
        
        return mapToDTO(savedUser);
    }
    
    private UserDTO mapToDTO(User user) {
        return new UserDTO(
            user.getEmail(),
            user.getFirstName(),
            user.getLastName(),
            null // Don't expose password
        );
    }
}
```

Run test (should PASS):
```bash
mvn test -Dtest=UserServiceTest#testCreateUserWithValidData
```

### Step 1.5: Refactor (REFACTOR)

Extract mapper to separate class:

**File:** `src/main/java/com/mentorsys/dto/mapper/UserMapper.java`

```java
@Component
public class UserMapper {
    
    public UserDTO toDTO(User user) {
        return new UserDTO(
            user.getEmail(),
            user.getFirstName(),
            user.getLastName(),
            null
        );
    }
    
    public User toEntity(UserDTO userDTO) {
        User user = new User();
        user.setEmail(userDTO.getEmail());
        user.setFirstName(userDTO.getFirstName());
        user.setLastName(userDTO.getLastName());
        return user;
    }
}
```

Update service to use mapper:

```java
@Autowired
private UserMapper userMapper;

@Override
public UserDTO createUser(UserDTO userDTO) {
    // ... validation code ...
    
    User savedUser = userRepository.save(user);
    return userMapper.toDTO(savedUser);
}
```

Commit:
```bash
git add .
git commit -m "feat: implement user creation service with mapper"
```

### Step 1.6: Additional Tests

Write more tests following same pattern:

```java
@Test
void testCreateUserWithDuplicateEmail() {
    // Arrange - user already exists
    when(userRepository.existsByEmail("john@example.com"))
        .thenReturn(true);
    
    UserDTO userDTO = new UserDTO(...);
    
    // Act & Assert
    assertThrows(DuplicateResourceException.class,
        () -> userService.createUser(userDTO));
}

@Test
void testCreateUserWithInvalidEmail() {
    // Arrange
    UserDTO userDTO = new UserDTO("invalid-email", ...);
    
    // Act & Assert
    assertThrows(InvalidDataException.class,
        () -> userService.createUser(userDTO));
}
```

---

## Phase 2: Core Features (Week 3-4)

### Objectives
- ✅ Implement REST controllers
- ✅ Add authentication (JWT)
- ✅ Create service layer for all entities
- ✅ Add exception handling

### Key Services to Implement (TDD)

1. **AuthService** - Login, Register, Token generation
2. **MentorService** - Create, Update, Get mentor profile
3. **SessionService** - Schedule, Update, Cancel sessions
4. **NotificationService** - Send, Retrieve notifications

Each should follow same TDD pattern from Phase 1.

---

## Phase 3: Real-time Features (Week 5-6)

### WebSocket Implementation

**File:** `src/main/java/com/mentorsys/config/WebSocketConfig.java`

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/user", "/topic");
        config.setApplicationDestinationPrefixes("/app");
        config.setUserDestinationPrefix("/user");
    }
    
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOrigins("*")
                .withSockJS();
    }
}
```

---

## Best Practices

### 1. Code Organization
```
✅ One class per file
✅ Clear naming conventions
✅ Single Responsibility Principle
✅ Dependency Injection
```

### 2. Testing Guidelines
```
✅ Test one thing per test method
✅ Use descriptive test names
✅ Follow AAA pattern
✅ Mock external dependencies
✅ Keep tests independent
✅ Aim for 80%+ coverage
```

### 3. Commit Practices
```bash
# Feature commit
git commit -m "feat: add user registration endpoint"

# Test commit (RED)
git commit -m "test: add test for user registration"

# Refactor commit
git commit -m "refactor: improve service layer performance"

# Bug fix
git commit -m "fix: handle null pointer exception"

# Documentation
git commit -m "docs: update API documentation"
```

### 4. Pull Request Workflow
```bash
# Create feature branch
git checkout -b feature/user-authentication

# Make changes, test locally
mvn clean test

# Push and create PR
git push origin feature/user-authentication
```

---

## Running Tests

```bash
# Run all tests
mvn test

# Run with output
mvn test -X

# Run specific class
mvn test -Dtest=UserServiceTest

# Run specific method
mvn test -Dtest=UserServiceTest#testCreateUserWithValidData

# Run with coverage
mvn clean test jacoco:report

# View coverage report
open target/site/jacoco/index.html
```

---

## Useful Resources

- [Spring Boot Testing](https://spring.io/guides/gs/testing-web/)
- [JUnit 5 User Guide](https://junit.org/junit5/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [Spring Security Guide](https://spring.io/guides/topicals/spring-security-architecture/)

---

**Next Step:** Review [TESTING_GUIDE.md](./TESTING_GUIDE.md) for detailed testing strategies.
