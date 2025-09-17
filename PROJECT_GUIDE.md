# Smart Banking & Digital Wallet System - Complete Project Guide

## 🎯 Project Overview

### Goal
Build a comprehensive bank-like digital wallet system that demonstrates enterprise-level microservices architecture, security, and scalability principles used in real FinTech platforms like Paytm, PhonePe, and GPay.

### Key Features
- User account management with KYC
- Multi-account banking system
- Secure fund transfers (UPI/IMPS/NEFT simulation)
- Digital wallet functionality
- Card management system
- Real-time notifications
- Payment gateway integration
- Enterprise security with JWT

## 🏗️ System Architecture

### Core Microservices

#### 1. User/KYC Service
**Responsibility:** Customer management and authentication
- User registration (username, email, phone)
- KYC document management (PAN, Aadhaar - mock format)
- Authentication and authorization
- Role-based access control (User, Admin)

**Technologies:** Spring Boot, Spring Security, JWT, MySQL
**Key APIs:**
- `POST /users/register`
- `POST /users/login`
- `POST /users/kyc`
- `GET /users/profile`

#### 2. Account Service
**Responsibility:** Banking account operations
- Account creation and management
- Balance tracking and updates
- Account types (Savings, Current)
- Credit/Debit operations

**Technologies:** Spring Boot, JPA/Hibernate, MySQL
**Key APIs:**
- `POST /accounts/create`
- `GET /accounts/{userId}`
- `GET /accounts/{accountId}/balance`
- `POST /accounts/{accountId}/credit`
- `POST /accounts/{accountId}/debit`

#### 3. Transaction Service
**Responsibility:** Money transfer and transaction management
- Fund transfers between accounts
- Transaction history and statements
- ACID transaction handling
- Saga pattern implementation

**Technologies:** Spring Boot, JPA, MySQL, Distributed Transaction Management
**Key APIs:**
- `POST /transactions/transfer`
- `GET /transactions/{userId}/history`
- `GET /transactions/{transactionId}`
- `GET /transactions/{accountId}/statement`

#### 4. Card Service
**Responsibility:** Debit/Credit card management
- Virtual card creation
- Card linking to accounts
- Masked card number storage
- Card transaction simulation

**Technologies:** Spring Boot, JPA, MySQL
**Key APIs:**
- `POST /cards/create`
- `GET /cards/{userId}`
- `POST /cards/{cardId}/link-account`
- `POST /cards/{cardId}/transaction`

#### 5. Payment Gateway Service
**Responsibility:** External payment processing
- UPI/Wallet payment simulation
- Merchant payment processing
- Integration with external APIs (Razorpay Sandbox)
- Payment status tracking

**Technologies:** Spring Boot, External API Integration, Async Processing
**Key APIs:**
- `POST /payments/initiate`
- `POST /payments/callback`
- `GET /payments/{paymentId}/status`
- `POST /payments/merchant-pay`

#### 6. Notification Service
**Responsibility:** Transaction alerts and notifications
- Email/SMS notifications (simulated)
- Real-time transaction alerts
- Asynchronous message processing

**Technologies:** Spring Boot, Kafka/RabbitMQ, Email Integration
**Key APIs:**
- `POST /notifications/send`
- `GET /notifications/{userId}`
- `POST /notifications/transaction-alert`

### Supporting Components

#### API Gateway
- **Purpose:** Single entry point for all client requests
- **Technology:** Spring Cloud Gateway / Zuul
- **Features:** 
  - Request routing
  - Load balancing
  - Rate limiting
  - Authentication validation

#### Service Registry
- **Purpose:** Service discovery and registration
- **Technology:** Eureka Server
- **Features:**
  - Dynamic service registration
  - Health checks
  - Load balancing support

#### Configuration Server
- **Purpose:** Centralized configuration management
- **Technology:** Spring Cloud Config
- **Features:**
  - Environment-specific configurations
  - Dynamic configuration updates
  - Configuration versioning

## 🗄️ Database Design

### User Service Database
```sql
-- Users table
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone_number VARCHAR(15) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role ENUM('USER', 'ADMIN') DEFAULT 'USER',
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- KYC table
CREATE TABLE kyc_details (
    kyc_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    pan_number VARCHAR(10),
    aadhaar_number VARCHAR(12),
    verification_status ENUM('PENDING', 'VERIFIED', 'REJECTED') DEFAULT 'PENDING',
    document_url VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### Account Service Database
```sql
-- Accounts table
CREATE TABLE accounts (
    account_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    account_number VARCHAR(20) UNIQUE NOT NULL,
    user_id BIGINT NOT NULL,
    account_type ENUM('SAVINGS', 'CURRENT') DEFAULT 'SAVINGS',
    balance DECIMAL(15,2) DEFAULT 0.00,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_account_number (account_number)
);
```

### Transaction Service Database
```sql
-- Transactions table
CREATE TABLE transactions (
    transaction_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    transaction_reference VARCHAR(50) UNIQUE NOT NULL,
    from_account_id BIGINT,
    to_account_id BIGINT,
    amount DECIMAL(15,2) NOT NULL,
    transaction_type ENUM('CREDIT', 'DEBIT', 'TRANSFER') NOT NULL,
    status ENUM('PENDING', 'SUCCESS', 'FAILED', 'CANCELLED') DEFAULT 'PENDING',
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_from_account (from_account_id),
    INDEX idx_to_account (to_account_id),
    INDEX idx_transaction_ref (transaction_reference)
);
```

### Card Service Database
```sql
-- Cards table
CREATE TABLE cards (
    card_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    card_number VARCHAR(20) NOT NULL, -- Encrypted/Hashed
    masked_number VARCHAR(20) NOT NULL, -- **** **** **** 1234
    user_id BIGINT NOT NULL,
    linked_account_id BIGINT,
    card_type ENUM('DEBIT', 'CREDIT') DEFAULT 'DEBIT',
    expiry_date DATE NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_linked_account (linked_account_id)
);
```

### Payment Service Database
```sql
-- Payments table
CREATE TABLE payments (
    payment_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    payment_reference VARCHAR(50) UNIQUE NOT NULL,
    user_id BIGINT NOT NULL,
    merchant_name VARCHAR(100),
    amount DECIMAL(15,2) NOT NULL,
    payment_method ENUM('UPI', 'CARD', 'WALLET') NOT NULL,
    status ENUM('INITIATED', 'SUCCESS', 'FAILED', 'CANCELLED') DEFAULT 'INITIATED',
    gateway_response TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_payment_ref (payment_reference)
);
```

### Notification Service Database
```sql
-- Notifications table
CREATE TABLE notifications (
    notification_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    notification_type ENUM('EMAIL', 'SMS', 'PUSH') NOT NULL,
    title VARCHAR(200) NOT NULL,
    message TEXT NOT NULL,
    status ENUM('PENDING', 'SENT', 'FAILED') DEFAULT 'PENDING',
    sent_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id)
);
```

## 🚀 Implementation Roadmap

### Phase 1: Foundation (Week 1-2)
**Objectives:** Set up project structure and core infrastructure
- [ ] Create GitHub repository with proper structure
- [ ] Set up parent POM with dependency management
- [ ] Configure Service Registry (Eureka Server)
- [ ] Set up Configuration Server
- [ ] Create API Gateway
- [ ] Set up MySQL databases for each service
- [ ] Implement User Service with basic CRUD operations
- [ ] Add Spring Security with JWT authentication

**Deliverables:**
- Working authentication system
- Service discovery setup
- Database schemas created
- Basic API Gateway routing

### Phase 2: Core Banking Services (Week 3-4)
**Objectives:** Implement core banking functionality
- [ ] Complete Account Service implementation
- [ ] Add account creation and balance management
- [ ] Implement Transaction Service
- [ ] Add basic fund transfer functionality
- [ ] Ensure ACID properties for transactions
- [ ] Add proper error handling and validation
- [ ] Write unit tests for critical functions

**Deliverables:**
- Working account management
- Basic fund transfer capability
- Transaction history and statements
- Proper error handling

### Phase 3: Advanced Features (Week 5-6)
**Objectives:** Add advanced banking features
- [ ] Implement Card Service
- [ ] Add virtual card creation and management
- [ ] Integrate Payment Gateway Service
- [ ] Connect with Razorpay Sandbox API
- [ ] Implement Notification Service
- [ ] Set up Kafka for async messaging
- [ ] Add transaction notifications

**Deliverables:**
- Card management system
- Payment gateway integration
- Real-time notifications
- Async message processing

### Phase 4: Production Readiness (Week 7-8)
**Objectives:** Polish and prepare for production
- [ ] Add comprehensive input validation
- [ ] Implement proper exception handling
- [ ] Add logging and monitoring
- [ ] Write comprehensive tests (Unit + Integration)
- [ ] Create API documentation (Swagger)
- [ ] Set up Docker containers
- [ ] Add security enhancements
- [ ] Performance optimization

**Deliverables:**
- Production-ready code
- Comprehensive documentation
- Docker deployment
- Complete test coverage

## 🔧 Technology Stack

### Backend Technologies
- **Framework:** Spring Boot 3.x
- **Security:** Spring Security + JWT
- **Database:** MySQL 8.0
- **ORM:** JPA/Hibernate
- **API Gateway:** Spring Cloud Gateway
- **Service Discovery:** Eureka
- **Messaging:** Apache Kafka
- **Documentation:** Swagger/OpenAPI 3
- **Testing:** JUnit 5, Mockito, TestContainers
- **Build Tool:** Maven
- **Containerization:** Docker

### External Integrations
- **Payment Gateway:** Razorpay Sandbox API
- **Email Service:** JavaMail API (Gmail SMTP)
- **SMS Service:** Mock implementation

## 📁 Project Structure

```
smart-banking-system/
├── README.md
├── pom.xml (Parent POM)
├── docker-compose.yml
├── .gitignore
├── docs/
│   ├── API_Documentation.md
│   ├── Database_Schema.md
│   └── Deployment_Guide.md
├── config-server/
│   ├── pom.xml
│   └── src/main/java/...
├── eureka-server/
│   ├── pom.xml
│   └── src/main/java/...
├── api-gateway/
│   ├── pom.xml
│   └── src/main/java/...
├── user-service/
│   ├── pom.xml
│   ├── src/main/java/...
│   └── src/main/resources/
├── account-service/
│   ├── pom.xml
│   ├── src/main/java/...
│   └── src/main/resources/
├── transaction-service/
│   ├── pom.xml
│   ├── src/main/java/...
│   └── src/main/resources/
├── card-service/
│   ├── pom.xml
│   ├── src/main/java/...
│   └── src/main/resources/
├── payment-service/
│   ├── pom.xml
│   ├── src/main/java/...
│   └── src/main/resources/
└── notification-service/
    ├── pom.xml
    ├── src/main/java/...
    └── src/main/resources/
```

## 🔐 Security Implementation

### Authentication & Authorization
- **JWT Tokens:** Stateless authentication
- **Role-based Access:** USER vs ADMIN roles
- **API Security:** Secure endpoints with proper authorities
- **Password Encryption:** BCrypt for password hashing

### Data Security
- **Sensitive Data Encryption:** Card numbers, PAN details
- **SQL Injection Prevention:** Parameterized queries
- **Input Validation:** Comprehensive validation for all inputs
- **CORS Configuration:** Proper CORS setup for API access

### Transaction Security
- **Idempotency:** Prevent duplicate transactions
- **Rate Limiting:** API rate limiting to prevent abuse
- **Audit Logging:** Transaction audit trails
- **Balance Verification:** Proper balance checks before debits

## 🧪 Testing Strategy

### Unit Testing
- **Service Layer:** Business logic testing
- **Repository Layer:** Data access testing
- **Controller Layer:** API endpoint testing
- **Security:** Authentication and authorization testing

### Integration Testing
- **Database Integration:** Test with real database
- **Service Integration:** Inter-service communication
- **External API Integration:** Mock external services
- **End-to-End Scenarios:** Complete user workflows

### Testing Tools
- **JUnit 5:** Unit testing framework
- **Mockito:** Mocking framework
- **TestContainers:** Integration testing with real databases
- **WireMock:** External API mocking

## 📊 Monitoring & Observability

### Logging
- **Structured Logging:** JSON formatted logs
- **Log Levels:** Appropriate log levels (INFO, WARN, ERROR)
- **Correlation IDs:** Track requests across services
- **Sensitive Data:** Mask sensitive information in logs

### Health Checks
- **Actuator Endpoints:** Service health monitoring
- **Database Health:** Database connectivity checks
- **External Service Health:** Payment gateway health
- **Custom Health Indicators:** Business-specific health checks

## 🚀 Deployment Strategy

### Local Development
```bash
# Start infrastructure services
docker-compose up -d mysql kafka zookeeper

# Start Spring Boot services
./mvnw spring-boot:run -pl eureka-server
./mvnw spring-boot:run -pl config-server
./mvnw spring-boot:run -pl api-gateway
./mvnw spring-boot:run -pl user-service
./mvnw spring-boot:run -pl account-service
# ... other services
```

### Docker Deployment
```bash
# Build all services
./mvnw clean package

# Build Docker images
docker-compose build

# Start complete system
docker-compose up -d
```

## 📝 API Documentation Examples

### User Service APIs
```http
POST /api/users/register
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "phoneNumber": "+919876543210",
  "password": "securePassword123"
}
```

### Fund Transfer API
```http
POST /api/transactions/transfer
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "fromAccountId": 12345,
  "toAccountId": 67890,
  "amount": 500.00,
  "description": "Payment for dinner",
  "transactionType": "TRANSFER"
}
```

## 🎯 Key Learning Outcomes

By completing this project, you will master:
- **Microservices Architecture:** Design and implementation
- **Spring Boot Ecosystem:** Security, Data, Cloud
- **Database Design:** Normalized schema design
- **Distributed Systems:** Inter-service communication
- **Security:** Authentication, authorization, data protection
- **Messaging:** Asynchronous communication patterns
- **Testing:** Comprehensive testing strategies
- **DevOps:** Containerization and deployment

## 🏆 Interview Talking Points

### System Design Questions
- "How would you design a payment system?"
- "Explain microservices vs monolith trade-offs"
- "How do you ensure data consistency across services?"
- "How would you handle high transaction volume?"

### Technical Deep-Dive
- "Explain your authentication strategy"
- "How do you prevent double-spending in transfers?"
- "Describe your error handling approach"
- "How do you ensure transaction atomicity?"

### Scalability & Performance
- "How would you scale this system to handle millions of users?"
- "What caching strategies would you implement?"
- "How do you monitor system performance?"
- "Explain your database optimization techniques"

## 🚀 Future Enhancements

### Advanced Features
- **Fraud Detection:** ML-based transaction monitoring
- **Blockchain Integration:** Cryptocurrency support
- **Analytics Dashboard:** Transaction analytics
- **Mobile SDK:** Mobile payment integration
- **International Transfers:** Multi-currency support

### Technical Improvements
- **CQRS Pattern:** Command Query Responsibility Segregation
- **Event Sourcing:** Event-based architecture
- **GraphQL:** Alternative API layer
- **Kubernetes:** Container orchestration
- **Service Mesh:** Istio for service communication

## 📚 Learning Resources

### Documentation
- [Spring Boot Official Documentation](https://spring.io/projects/spring-boot)
- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Microservices.io](https://microservices.io/)
- [Razorpay API Documentation](https://razorpay.com/docs/api/)

### Books
- "Microservices Patterns" by Chris Richardson
- "Spring Boot in Action" by Craig Walls
- "Building Microservices" by Sam Newman
- "Java: The Complete Reference" by Herbert Schildt

### Online Courses
- Spring Boot & Microservices on Udemy
- Java Brains YouTube Channel
- Baeldung Spring Tutorials
- Official Spring Boot Guides

---

## 🎯 Success Metrics

### Code Quality
- [ ] 90%+ test coverage
- [ ] Zero critical security vulnerabilities
- [ ] Proper error handling for all edge cases
- [ ] Clean, documented code following SOLID principles

### Functionality
- [ ] All user stories implemented
- [ ] End-to-end transaction flows working
- [ ] Real-time notifications functioning
- [ ] Payment gateway integration successful

### Documentation
- [ ] Complete API documentation
- [ ] Architecture diagrams
- [ ] Database schema documentation
- [ ] Deployment instructions

### Interview Readiness
- [ ] Can explain every architectural decision
- [ ] Comfortable discussing scalability challenges
- [ ] Ready to demonstrate live system
- [ ] Prepared for technical deep-dive questions

---

**Remember:** This is not just a project; it's your gateway to senior developer roles in FinTech. Take time to understand each concept deeply, document your learnings, and prepare compelling stories about the challenges you solved.

**Good luck building your career-defining project! 🚀**
