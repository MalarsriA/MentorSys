# Testing Guide - MentorSys TDD Approach

Complete guide for implementing Test-Driven Development in MentorSys project.

## Table of Contents

1. [Testing Fundamentals](#testing-fundamentals)
2. [Unit Testing](#unit-testing)
3. [Integration Testing](#integration-testing)
4. [Testing Best Practices](#testing-best-practices)
5. [Coverage and Reports](#coverage-and-reports)
6. [Common Testing Scenarios](#common-testing-scenarios)

---

## Testing Fundamentals

### Test Types Pyramid

```
          /\
         /  \        E2E Tests
        /----\       (UI/Integration)
       /      \
      /--------\     Integration Tests
     /          \    (Service + DB)
    /            \
   /--============\  Unit Tests
  /                \ (Service/Controller)
```

### Test Scope in MentorSys

| Type | Scope | Coverage Target | Framework |
|------|-------|-----------------|-----------|
| **Unit** | Single method/class | 80%+ | JUnit 5, Mockito |
| **Integration** | Service layer + DB | 60%+ | Spring Test, TestContainers |
| **E2E** | Full request cycle | 40%+ | REST Assured, Selenium (Frontend) |

---

## Unit Testing

### 1. Service Layer Testing

#### Pattern: Test Service Methods

**File:** `src/test/java/com/mentorsys/service/UserServiceTest.java`

```java
@ExtendWith(MockitoExtension.class)
@DisplayName("UserService Unit Tests")
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;
    
    @InjectMocks
    private UserServiceImpl userService;
    
    private UserDTO testUserDTO;
    private User testUser;
    
    @BeforeEach
    void setUp() {
        // Initialize test data
        testUserDTO = new UserDTO(
            "john@example.com",
            "John",
            "Doe",
            "password123"
        );
        
        testUser = new User(
            1L,
            "john@example.com",
            "John",
            "Doe",
            "encodedPassword",
            UserRole.MENTEE,
            true,
            LocalDateTime.now(),
            LocalDateTime.now()
        );
    }
    
    // ==================== CREATE TESTS ====================
    
    @Test
    @DisplayName("Should create user with valid data")
    void testCreateUserWithValidData() {
        // Arrange
        when(userRepository.existsByEmail(testUserDTO.getEmail()))
            .thenReturn(false);
        when(passwordEncoder.encode(testUserDTO.getPassword()))
            .thenReturn("encodedPassword");
        when(userRepository.save(any(User.class)))
            .thenReturn(testUser);
        
        // Act
        UserDTO result = userService.createUser(testUserDTO);
        
        // Assert
        assertNotNull(result);
        assertEquals(testUserDTO.getEmail(), result.getEmail());
        assertEquals(testUserDTO.getFirstName(), result.getFirstName());
        verify(userRepository, times(1)).save(any(User.class));
    }
    
    @Test
    @DisplayName("Should throw exception when email already exists")
    void testCreateUserWithDuplicateEmail() {
        // Arrange
        when(userRepository.existsByEmail(testUserDTO.getEmail()))
            .thenReturn(true);
        
        // Act & Assert
        DuplicateResourceException exception = assertThrows(
            DuplicateResourceException.class,
            () -> userService.createUser(testUserDTO)
        );
        
        assertEquals("Email already exists", exception.getMessage());
        verify(userRepository, never()).save(any(User.class));
    }
    
    @Test
    @DisplayName("Should throw exception when email is invalid")
    void testCreateUserWithInvalidEmail() {
        // Arrange
        UserDTO invalidDTO = new UserDTO(
            "invalid-email",
            "John",
            "Doe",
            "password123"
        );
        
        // Act & Assert
        assertThrows(
            InvalidDataException.class,
            () -> userService.createUser(invalidDTO)
        );
    }
    
    @Test
    @DisplayName("Should throw exception when password is null")
    void testCreateUserWithNullPassword() {
        // Arrange
        UserDTO invalidDTO = new UserDTO(
            "john@example.com",
            "John",
            "Doe",
            null
        );
        
        // Act & Assert
        assertThrows(
            InvalidDataException.class,
            () -> userService.createUser(invalidDTO)
        );
    }
    
    // ==================== READ TESTS ====================
    
    @Test
    @DisplayName("Should get user by id")
    void testGetUserById() {
        // Arrange
        when(userRepository.findById(1L))
            .thenReturn(Optional.of(testUser));
        
        // Act
        UserDTO result = userService.getUserById(1L);
        
        // Assert
        assertNotNull(result);
        assertEquals(testUser.getEmail(), result.getEmail());
        verify(userRepository, times(1)).findById(1L);
    }
    
    @Test
    @DisplayName("Should throw exception when user not found")
    void testGetUserByIdWhenNotFound() {
        // Arrange
        when(userRepository.findById(999L))
            .thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(
            ResourceNotFoundException.class,
            () -> userService.getUserById(999L)
        );
    }
    
    @Test
    @DisplayName("Should get user by email")
    void testGetUserByEmail() {
        // Arrange
        when(userRepository.findByEmail("john@example.com"))
            .thenReturn(Optional.of(testUser));
        
        // Act
        UserDTO result = userService.getUserByEmail("john@example.com");
        
        // Assert
        assertNotNull(result);
        assertEquals("john@example.com", result.getEmail());
    }
    
    // ==================== UPDATE TESTS ====================
    
    @Test
    @DisplayName("Should update user with valid data")
    void testUpdateUserWithValidData() {
        // Arrange
        UserDTO updateDTO = new UserDTO(
            "john@example.com",
            "Johnny",
            "Smith",
            null
        );
        
        User updatedUser = new User(
            1L,
            "john@example.com",
            "Johnny",
            "Smith",
            "encodedPassword",
            UserRole.MENTEE,
            true,
            LocalDateTime.now(),
            LocalDateTime.now()
        );
        
        when(userRepository.findById(1L))
            .thenReturn(Optional.of(testUser));
        when(userRepository.save(any(User.class)))
            .thenReturn(updatedUser);
        
        // Act
        UserDTO result = userService.updateUser(1L, updateDTO);
        
        // Assert
        assertNotNull(result);
        assertEquals("Johnny", result.getFirstName());
        assertEquals("Smith", result.getLastName());
        verify(userRepository, times(1)).save(any(User.class));
    }
    
    @Test
    @DisplayName("Should throw exception when updating non-existent user")
    void testUpdateNonExistentUser() {
        // Arrange
        when(userRepository.findById(999L))
            .thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(
            ResourceNotFoundException.class,
            () -> userService.updateUser(999L, testUserDTO)
        );
    }
    
    // ==================== DELETE TESTS ====================
    
    @Test
    @DisplayName("Should delete user when exists")
    void testDeleteUserWhenExists() {
        // Arrange
        when(userRepository.existsById(1L))
            .thenReturn(true);
        
        // Act
        userService.deleteUser(1L);
        
        // Assert
        verify(userRepository, times(1)).deleteById(1L);
    }
    
    @Test
    @DisplayName("Should throw exception when deleting non-existent user")
    void testDeleteNonExistentUser() {
        // Arrange
        when(userRepository.existsById(999L))
            .thenReturn(false);
        
        // Act & Assert
        assertThrows(
            ResourceNotFoundException.class,
            () -> userService.deleteUser(999L)
        );
    }
}
```

### 2. Controller Layer Testing

**File:** `src/test/java/com/mentorsys/controller/UserControllerTest.java`

```java
@ExtendWith(MockitoExtension.class)
@WebMvcTest(UserController.class)
@DisplayName("UserController Unit Tests")
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @MockBean
    private JwtTokenProvider tokenProvider;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    private UserDTO testUserDTO;
    
    @BeforeEach
    void setUp() {
        testUserDTO = new UserDTO(
            "john@example.com",
            "John",
            "Doe",
            "password123"
        );
    }
    
    @Test
    @DisplayName("Should create user with POST request")
    void testCreateUser() throws Exception {
        // Arrange
        when(userService.createUser(any(UserDTO.class)))
            .thenReturn(testUserDTO);
        
        // Act & Assert
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(testUserDTO)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
    
    @Test
    @DisplayName("Should get user by id")
    void testGetUserById() throws Exception {
        // Arrange
        when(userService.getUserById(1L))
            .thenReturn(testUserDTO);
        
        // Act & Assert
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
    
    @Test
    @DisplayName("Should return 404 when user not found")
    void testGetUserNotFound() throws Exception {
        // Arrange
        when(userService.getUserById(999L))
            .thenThrow(new ResourceNotFoundException("User not found"));
        
        // Act & Assert
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound());
    }
}
```

---

## Integration Testing

### 1. Service + Repository Integration

**File:** `src/test/java/com/mentorsys/service/UserServiceIntegrationTest.java`

```java
@SpringBootTest
@Transactional
@DisplayName("UserService Integration Tests")
class UserServiceIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }
    
    @Test
    @DisplayName("Should persist user to database")
    void testCreateUserAndPersist() {
        // Arrange
        UserDTO userDTO = new UserDTO(
            "john@example.com",
            "John",
            "Doe",
            "password123"
        );
        
        // Act
        UserDTO savedUser = userService.createUser(userDTO);
        
        // Assert
        assertNotNull(savedUser.getId());
        assertTrue(userRepository.existsByEmail("john@example.com"));
        
        User dbUser = userRepository.findByEmail("john@example.com").orElse(null);
        assertNotNull(dbUser);
        assertTrue(passwordEncoder.matches("password123", dbUser.getPassword()));
    }
    
    @Test
    @DisplayName("Should retrieve user from database")
    void testGetUserFromDatabase() {
        // Arrange - Create user in database
        User user = new User();
        user.setEmail("jane@example.com");
        user.setFirstName("Jane");
        user.setLastName("Smith");
        user.setPassword(passwordEncoder.encode("password123"));
        user.setRole(UserRole.MENTOR);
        user.setActive(true);
        
        User savedUser = userRepository.save(user);
        
        // Act
        UserDTO result = userService.getUserById(savedUser.getId());
        
        // Assert
        assertNotNull(result);
        assertEquals("jane@example.com", result.getEmail());
    }
}
```

### 2. API Integration Tests

**File:** `src/test/java/com/mentorsys/integration/UserControllerIntegrationTest.java`

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.ANY)
@DisplayName("UserController Integration Tests")
class UserControllerIntegrationTest {
    
    @LocalServerPort
    private int port;
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    private String baseUrl;
    
    @BeforeEach
    void setUp() {
        baseUrl = "http://localhost:" + port + "/api";
        userRepository.deleteAll();
    }
    
    @Test
    @DisplayName("Should create user via REST API")
    void testCreateUserViaAPI() {
        // Arrange
        UserDTO userDTO = new UserDTO(
            "john@example.com",
            "John",
            "Doe",
            "password123"
        );
        
        // Act
        ResponseEntity<UserDTO> response = restTemplate.postForEntity(
            baseUrl + "/users",
            userDTO,
            UserDTO.class
        );
        
        // Assert
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertEquals("john@example.com", response.getBody().getEmail());
    }
}
```

---

## Testing Best Practices

### 1. Naming Conventions

```java
// ❌ Bad
void test1() { }
void testUser() { }

// ✅ Good
void testCreateUserWithValidData() { }
void testGetUserByIdWhenUserNotFound() { }
void testUpdateUserChangeEmailAndPhone() { }
```

### 2. AAA Pattern (Arrange-Act-Assert)

```java
@Test
void testScenario() {
    // ARRANGE: Set up test data
    UserDTO userDTO = createTestUserDTO();
    when(userRepository.save(any())).thenReturn(testUser);
    
    // ACT: Execute the method
    UserDTO result = userService.createUser(userDTO);
    
    // ASSERT: Verify results
    assertNotNull(result);
    assertEquals(userDTO.getEmail(), result.getEmail());
}
```

### 3. Mock vs Spy

```java
// Mock: Complete replacement
@Mock
private UserRepository userRepository; // All methods return defaults

// Spy: Real object with selective overrides
@Spy
private UserService userService; // Real implementation, can override methods
```

### 4. Assertions Library

```java
import static org.junit.jupiter.api.Assertions.*;

// Basic assertions
assertEquals(expected, actual);
assertNotNull(object);
assertTrue(condition);
assertFalse(condition);

// Collection assertions
assertIterableEquals(expected, actual);
assertAll("group name",
    () -> assertEquals(...),
    () -> assertTrue(...)
);

// Exception assertions
assertThrows(Exception.class, () -> method());

// Custom messages
assertEquals(expected, actual, "Custom error message");
```

### 5. Test Organization

```java
@ExtendWith(MockitoExtension.class)
@DisplayName("UserService Tests")
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserServiceImpl userService;
    
    // ==================== TEST DATA ====================
    
    @BeforeEach
    void setUp() {
        // Initialize test data
    }
    
    // ==================== CREATE OPERATIONS ====================
    
    @Test
    @DisplayName("Should create...")
    void testCreate() { }
    
    // ==================== READ OPERATIONS ====================
    
    @Test
    @DisplayName("Should retrieve...")
    void testRead() { }
    
    // ==================== UPDATE OPERATIONS ====================
    
    @Test
    @DisplayName("Should update...")
    void testUpdate() { }
    
    // ==================== DELETE OPERATIONS ====================
    
    @Test
    @DisplayName("Should delete...")
    void testDelete() { }
}
```

---

## Coverage and Reports

### Generate Code Coverage Report

```bash
# Generate JaCoCo report
mvn clean test jacoco:report

# View report
open target/site/jacoco/index.html  # Mac
start target/site/jacoco/index.html # Windows
xdg-open target/site/jacoco/index.html # Linux
```

### Coverage Configuration

Add to `pom.xml`:

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>PACKAGE</element>
                        <excludes>
                            <exclude>*Test</exclude>
                        </excludes>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

---

## Common Testing Scenarios

### Scenario 1: Testing Exception Handling

```java
@Test
@DisplayName("Should handle null input gracefully")
void testNullInputHandling() {
    // Arrange & Act & Assert
    assertThrows(
        InvalidDataException.class,
        () -> userService.createUser(null),
        "Should throw InvalidDataException for null input"
    );
}
```

### Scenario 2: Testing State Changes

```java
@Test
@DisplayName("Should deactivate user")
void testDeactivateUser() {
    // Arrange
    User user = testUser;
    user.setActive(true);
    
    when(userRepository.findById(1L)).thenReturn(Optional.of(user));
    when(userRepository.save(any())).thenReturn(user);
    
    // Act
    userService.deactivateUser(1L);
    
    // Assert - Verify state changed
    ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
    verify(userRepository).save(captor.capture());
    assertFalse(captor.getValue().getActive());
}
```

### Scenario 3: Testing List Operations

```java
@Test
@DisplayName("Should retrieve all mentors")
void testGetAllMentors() {
    // Arrange
    List<User> mentors = Arrays.asList(
        createMentor(1L, "mentor1@example.com"),
        createMentor(2L, "mentor2@example.com")
    );
    
    when(userRepository.findByRole(UserRole.MENTOR))
        .thenReturn(mentors);
    
    // Act
    List<UserDTO> result = userService.getAllMentors();
    
    // Assert
    assertEquals(2, result.size());
    assertTrue(result.stream()
        .allMatch(dto -> dto.getRole() == UserRole.MENTOR));
}
```

---

## Running Tests in CI/CD

### GitHub Actions Example

Create `.github/workflows/tests.yml`:

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK 17
      uses: actions/setup-java@v2
      with:
        java-version: '17'
        distribution: 'adopt'
    
    - name: Run tests
      run: mvn clean test
    
    - name: Generate coverage report
      run: mvn jacoco:report
    
    - name: Upload coverage
      uses: codecov/codecov-action@v2
      with:
        files: ./target/site/jacoco/jacoco.xml
```

---

**Next Step:** Implement first service with TDD following [DEVELOPMENT.md](./DEVELOPMENT.md) Phase 1.
