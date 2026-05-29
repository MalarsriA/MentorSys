# MentorSys - Real-time Mentorship Platform

A real-time mentorship platform built with Java Spring Boot backend and modern frontend technologies. This project demonstrates TDD (Test-Driven Development) practices with comprehensive documentation.

## 📋 Project Overview

**MentorSys** is designed to connect mentors and mentees in real-time with features like:
- Real-time messaging and notifications
- Session scheduling and management
- Profile management
- Progress tracking
- Real-time notifications via WebSocket

## 🏗️ Architecture

### Technology Stack

**Backend:**
- Java 17
- Spring Boot 3.2
- Spring Security (JWT Authentication)
- Spring WebSocket (Real-time communication)
- Spring Data JPA
- PostgreSQL / H2 (Development)
- Maven 3.8+

**Frontend (Parallel Learning):**
- HTML5 / CSS3 / JavaScript (Foundation)
- React.js (Modern framework)
- WebSocket Client
- Bootstrap / Tailwind CSS (Styling)

## 🚀 Quick Start

### Prerequisites
- JDK 17+
- Maven 3.8+
- Git
- PostgreSQL 12+ (Production) or H2 (Development)

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/MalarsriA/MentorSys.git
cd MentorSys
```

2. **Build the project**
```bash
mvn clean install
```

3. **Run tests**
```bash
mvn test
```

4. **Run the application**
```bash
mvn spring-boot:run
```

The application will start on `http://localhost:8080`

## 📁 Project Structure

```
MentorSys/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/mentorsys/
│   │   │       ├── MentorSysApplication.java
│   │   │       ├── config/                 # Configuration classes
│   │   │       ├── controller/             # REST API Controllers
│   │   │       ├── service/                # Business Logic
│   │   │       ├── repository/             # Data Access Layer
│   │   │       ├── entity/                 # JPA Entities
│   │   │       ├── dto/                    # Data Transfer Objects
│   │   │       ├── exception/              # Custom Exceptions
│   │   │       ├── security/               # Security Configuration
│   │   │       └── websocket/              # WebSocket handlers
│   │   └── resources/
│   │       ├── application.yml             # Application properties
│   │       └── application-test.yml        # Test properties
│   └── test/
│       └── java/
│           └── com/mentorsys/
│               ├── controller/             # Controller Tests
│               ├── service/                # Service Tests
│               └── repository/             # Repository Tests
├── pom.xml                                 # Maven configuration
├── README.md                               # This file
├── DEVELOPMENT.md                          # Development guide
├── API_DOCUMENTATION.md                    # API docs
└── TESTING_GUIDE.md                        # Testing guide
```

## 📚 Documentation Guide

This project includes comprehensive documentation:

1. **[DEVELOPMENT.md](./DEVELOPMENT.md)** - Step-by-step development guide
2. **[API_DOCUMENTATION.md](./API_DOCUMENTATION.md)** - REST API endpoints
3. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - TDD approach and testing strategies
4. **[DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)** - Database design
5. **[FRONTEND_GUIDE.md](./FRONTEND_GUIDE.md)** - Frontend development (Parallel learning)

## 🎯 Development Approach: TDD

We follow **Test-Driven Development (TDD)** with the Red-Green-Refactor cycle:

1. **Red** - Write a failing test
2. **Green** - Write minimal code to pass the test
3. **Refactor** - Improve code while tests pass

### TDD Workflow

```
1. Create Test Class (suffix: Test.java)
2. Write test for desired feature (FAILING)
3. Implement minimal code (PASSING)
4. Refactor for clarity and performance
5. Commit to repository
```

## 🧪 Testing Strategy

```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=UserServiceTest

# Run with code coverage
mvn clean test jacoco:report

# Run integration tests
mvn verify
```

**Test Coverage Target:** 80%+

## 🔐 Security

- JWT Token-based Authentication
- Role-based Access Control (RBAC)
- CORS Configuration
- Input Validation
- SQL Injection Prevention via Parameterized Queries

## 📡 Real-time Features

### WebSocket Implementation

- **Endpoint:** `/ws` (STOMP over WebSocket)
- **Topics:**
  - `/user/queue/notifications` - Personal notifications
  - `/topic/messages/{sessionId}` - Session messages
  - `/topic/presence` - User presence updates

## 🤝 Contributing

1. Create a feature branch: `git checkout -b feature/feature-name`
2. Write tests first (TDD approach)
3. Implement feature
4. Ensure all tests pass: `mvn test`
5. Create Pull Request

## 📖 Learning Path

### Week 1-2: Foundation
- [ ] Set up development environment
- [ ] Understand project structure
- [ ] Write first unit tests
- [ ] Learn Spring Boot basics

### Week 3-4: Core Features
- [ ] Implement User entity and service (TDD)
- [ ] Create REST controllers
- [ ] Build authentication service
- [ ] Write integration tests

### Week 5-6: Real-time Features
- [ ] Implement WebSocket configuration
- [ ] Create messaging service
- [ ] Add real-time notifications
- [ ] Build frontend consumer (parallel)

### Week 7-8: Advanced Features
- [ ] Session scheduling
- [ ] Progress tracking
- [ ] Search and filtering
- [ ] Performance optimization

## 🛠️ Useful Maven Commands

```bash
# Clean build
mvn clean install

# Skip tests during build
mvn clean install -DskipTests

# Run specific profile
mvn clean install -Pdev

# Check dependencies
mvn dependency:tree

# Format code
mvn spotless:apply

# Generate reports
mvn site
```

## 📊 Code Quality Tools

- **JaCoCo** - Code coverage
- **Checkstyle** - Code style
- **SureFire** - Test runner

Reports available in: `target/site/`

## 🐛 Troubleshooting

### Common Issues

1. **Port already in use**
```bash
lsof -i :8080  # Find process
kill -9 <PID>  # Kill process
```

2. **Database connection issues**
- Check `application.yml` configuration
- Ensure PostgreSQL is running (if using)
- Check database credentials

3. **Test failures**
- Clear Maven cache: `mvn clean`
- Rebuild: `mvn install`
- Check logs in `target/`

## 📞 Support & Questions

For questions or issues:
1. Check existing [GitHub Issues](https://github.com/MalarsriA/MentorSys/issues)
2. Create detailed bug report
3. Join discussions in repository

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

## 🎓 Learning Resources

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Spring WebSocket Guide](https://spring.io/guides/gs/messaging-stomp-websocket/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

**Last Updated:** 2026-05-29  
**Version:** 1.0.0-SNAPSHOT  
**Maintainer:** MalarsriA
