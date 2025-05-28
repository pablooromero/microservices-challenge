# Microservices Challenge

## 1. Introduction

This project is a microservices-based application developed with Spring Boot and Java 21. The system manages users, sales points, and accreditations, with email notifications and quality analysis. It is designed to be modular, scalable, and maintainable, utilizing modern tools and development best practices.

## 2. General Architecture

The system comprises the following main microservices:

* **eureka-service**: Service discovery server using Netflix Eureka. It allows services to register and find each other dynamically.
* **api-gateway**: Single entry point for all client requests. It handles routing, JWT token validation, and can apply cross-cutting concerns filters. Uses Spring Cloud Gateway.
* **user-service**: Manages all user-related logic, including registration, authentication (direct login and OAuth2 with Google), JWT generation, and user administration operations.
* **sales-point-service**: Manages sales points and the travel costs between them, including logic to calculate the shortest path (Dijkstra's algorithm).
* **accreditations-service**: Handles the creation and querying of accreditations, interacting with user-service and sales-point-service for additional information.
* **email-service**: Responsible for asynchronously sending emails (e.g., registration confirmation, accreditation PDFs) via RabbitMQ.

### Communication Flow (Simplified):

* Clients interact with the api-gateway.
* The api-gateway routes requests to the appropriate microservices.
* Services communicate via REST or asynchronously via RabbitMQ.
* All services (except Eureka) register with eureka-service.
* Distributed traces are sent to Zipkin.

## 3. Main Features

* User registration and authentication (credentials and OAuth2 with Google)
* JWT token generation and validation for API security
* Role-based authorization (e.g., admin-only endpoints)
* CRUD management for Sales Points and Costs between them
* Shortest path calculation between sales points
* CRUD management for Accreditations
* PDF generation for accreditation receipts
* Asynchronous email sending for notifications (registration, accreditations)
* Service discovery with Eureka
* Centralized routing and security at the API Gateway
* Resilience patterns with Resilience4j (Circuit Breaker, Retry, Rate Limiter)
* Distributed tracing with Micrometer and Zipkin
* Global and structured exception handling
* API documentation with OpenAPI (Swagger UI)
* Code quality analysis with SonarQube and coverage with JaCoCo
* Containerization with Docker and orchestration with Docker Compose

## 4. Technology Stack

* **Language**: Java 21
* **Main Framework**: Spring Boot
* **Microservices**: Spring Cloud
* **Service Discovery**: Spring Cloud Netflix Eureka
* **Gateway**: Spring Cloud Gateway
* **Security**: Spring Security (OAuth2, JWT)
* **Database**: PostgreSQL (per service), H2 (tests)
* **ORM**: Spring Data JPA (Hibernate)
* **Messaging**: RabbitMQ (Spring AMQP)
* **PDF**: Apache PDFBox
* **Email**: Spring Boot Starter Mail (JavaMailSender)
* **Resilience**: Resilience4j
* **Observability**: Micrometer Tracing, Zipkin, Spring Boot Actuator
* **API Docs**: Springdoc OpenAPI
* **Code Quality**: SonarQube, JaCoCo
* **Build Tool**: Apache Maven
* **Containerization**: Docker, Docker Compose
* **Utilities**: Lombok, JJWT

## 5. Prerequisites

* JDK 21
* Apache Maven 3.6.3+
* Git
* Docker & Docker Compose
* IDE (IntelliJ IDEA, Eclipse, VSCode)

**Optional (local runs):**

* SonarQube
* RabbitMQ
* Zipkin
* PostgreSQL

## 6. Getting Started

```bash
git clone https://github.com/pablooromero/microservices-challenge
cd microservices-challenge

# Initialize Git submodules
git submodule update --init --recursive

# Update to latest submodules
git submodule update --remote --merge
```

## 7. Building the Project

To build a specific service (e.g., user-service):

```bash
cd user-service
mvn clean package
cd ..
```

To build Docker images:

```bash
docker-compose build
```

## 8. Running the Application

### Setup .env File

Create a `.env` in the project root (same level as `docker-compose.yml`):

```dotenv
JWT_SECRET="your_long_and_secure_base64_jwt_secret_here"
JWT_EXPIRATION=3600000

DB_USER=postgres
DB_PASSWORD=your_secure_db_password

RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest

SPRING_MAIL_USERNAME="your_gmail_username@gmail.com"
SPRING_MAIL_PASSWORD="your_gmail_app_password"

CLIENT_ID="your_google_client_id"
CLIENT_SECRET="your_google_client_secret"

SONAR_TOKEN="your_sonarqube_token"
```

### Start Services

```bash
docker-compose up --build -d
```

### Stop Services

```bash
docker-compose down
```

### Stop and Remove Volumes

```bash
docker-compose down -v
```

### Key URLs

* API Gateway: [http://localhost:8080](http://localhost:8080)
* Eureka Server: [http://localhost:8761](http://localhost:8761)
* RabbitMQ Management: [http://localhost:15672](http://localhost:15672) (user: guest, pass: guest)
* Zipkin: [http://localhost:9411](http://localhost:9411)
* SonarQube: [http://localhost:9000](http://localhost:9000)

## 9. Application Testing

### Run Tests

```bash
mvn test
# or
mvn clean verify
```

### Manual E2E Testing

Use Postman or curl:

* **Register**: `POST /api/auth/register`
* **Login**: `POST /api/auth/login`
* **Create Accreditation**: `POST /api/accreditations`

Check:

* DB updates
* Email received
* Zipkin traces

### Security Tests

* 403: Access admin endpoint without token
* 401: Access any secured endpoint without token

## 10. Code Quality and Coverage

### JaCoCo

* Run tests: `mvn clean test`
* Coverage report: `target/site/jacoco/index.html`

### SonarQube

```bash
mvn clean verify sonar:sonar
```

Properties configured in `pom.xml`.

## 11. API Documentation

Accessible via Swagger UI through the API Gateway:

* User Service: [http://localhost:8080/user-service/swagger-ui.html](http://localhost:8080/user-service/swagger-ui.html)
* Accreditations Service: [http://localhost:8080/accreditations-service/swagger-ui.html](http://localhost:8080/accreditations-service/swagger-ui.html)
* Sales Point Service: [http://localhost:8080/sales-point-service/swagger-ui.html](http://localhost:8080/sales-point-service/swagger-ui.html)

## 12. Security

* Spring Security with JWT
* OAuth2 Resource Server (all services)
* JWT issuer: `user-service`
* Role-based access control (@PreAuthorize)
* API Gateway validates JWTs
* JWT propagation in REST communication

## 13. Assumptions Made

* Basic familiarity with Java, Spring Boot, Maven, Git, Docker
* Services run on localhost with default ports
* Gmail app password used if 2FA enabled
* Docker containers can communicate with each other
* `.env` is used for secret management

## 14. Default Admin User

Email: test@example.com

Password: Test12345

Use these credentials to log in and obtain an admin JWT token for testing protected endpoints.
