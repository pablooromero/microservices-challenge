Microservices Challenge
1. Introduction
This project is a microservices-based application developed with Spring Boot and Java 21. The system manages users, sales points, and accreditations, with email notifications and quality analysis. It is designed to be modular, scalable, and maintainable, utilizing modern tools and development best practices.

2. General Architecture
The system comprises the following main microservices:

eureka-service: Service discovery server using Netflix Eureka. It allows services to register and find each other dynamically.

api-gateway: Single entry point for all client requests. It handles routing, JWT token validation, and can apply cross-cutting concerns filters. Uses Spring Cloud Gateway.

user-service: Manages all user-related logic, including registration, authentication (direct login and OAuth2 with Google), JWT generation, and user administration operations.

sales-point-service: Manages sales points and the travel costs between them, including logic to calculate the shortest path (Dijkstra's algorithm).

accreditations-service: Handles the creation and querying of accreditations, interacting with user-service and sales-point-service for additional information.

email-service: Responsible for asynchronously sending emails (e.g., registration confirmation, accreditation PDFs) via RabbitMQ.

Communication Flow (Simplified):

Clients interact with the api-gateway.

The api-gateway routes requests to the appropriate microservices.

Services can communicate with each other via REST (e.g., accreditation-service calls user-service and sales-point-service) or asynchronously via RabbitMQ (e.g., user-service and accreditation-service publish events consumed by email-service).

All services (except Eureka) register with eureka-service.

Distributed traces are sent to Zipkin.


3. Main Features
User registration and authentication (credentials and OAuth2 with Google).

JWT token generation and validation for API security.

Role-based authorization (e.g., admin-only endpoints).

CRUD management for Sales Points and Costs between them.

Shortest path calculation between sales points.

CRUD management for Accreditations.

PDF generation for accreditation receipts.

Asynchronous email sending for notifications (registration, accreditations).

Service discovery with Eureka.

Centralized routing and security at the API Gateway.

Resilience patterns in inter-service communication (Circuit Breaker, Retry, Rate Limiter with Resilience4j).

Distributed tracing with Micrometer and Zipkin.

Global and structured exception handling.

API documentation with OpenAPI (Swagger UI).

Code quality analysis with SonarQube and coverage with JaCoCo.

Containerization with Docker and orchestration with Docker Compose.

4. Technology Stack
Language: Java 21

Main Framework: Spring Boot

Microservices: Spring Cloud

Spring Cloud Gateway

Spring Cloud Netflix Eureka Server & Client

Security: Spring Security (OAuth2 Resource Server, OAuth2 Client, JWT)

Database: PostgreSQL (per service), H2 (for tests)

ORM: Spring Data JPA (with Hibernate)

Messaging: RabbitMQ (with Spring AMQP)

PDF Generation: Apache PDFBox

Email Sending: Spring Boot Starter Mail (JavaMailSender)

Resilience: Resilience4j

Observability:

Micrometer Tracing

Zipkin

Spring Boot Actuator

API Docs: Springdoc OpenAPI

Code Quality:

SonarQube

JaCoCo

Build Tool: Apache Maven

Containerization: Docker, Docker Compose (or Podman)

Utility Libraries: Lombok, JJWT (for JWTs in user-service)

5. Prerequisites for Development Environment
JDK 21 (Oracle JDK, OpenJDK, Eclipse Temurin, etc.). Ensure JAVA_HOME is configured.

Apache Maven 3.6.3+ (Ensure mvn is in your system PATH).

Git.

Docker and Docker Compose.

IDE of your choice (e.g., IntelliJ IDEA, Eclipse, VSCode).

(Optional, if running locally outside Docker) A running SonarQube Server instance.

(Optional, if running locally outside Docker) A running RabbitMQ instance.

(Optional, if running locally outside Docker) A running Zipkin instance.

(Optional, if running locally outside Docker) Running and configured PostgreSQL instances.

6. Getting Started
Clone the Parent Repository:

git clone [https://github.com/pablooromero/microservices-challenge](https://github.com/pablooromero/microservices-challenge)
cd microservices-challenge

Initialize and Update Git Submodules:

git submodule update --init --recursive
# To get the latest versions of the submodules:
git submodule update --remote --merge

7. Building the Project
To build a specific microservice (e.g., user-service):

cd user-service
mvn clean package
cd ..

Repeat for other services. The -DskipTests option can be used to speed up packaging if tests have already been validated.
Executable JAR files will be located in the target/ folder of each microservice (e.g., user-service/target/user-service-0.0.1-SNAPSHOT.jar).

To build Docker images (after compiling the JARs):
From the root of the parent repository (where your docker-compose.yml is located):

docker-compose build

8. Running the Application
The recommended way to run the entire environment is using Docker Compose.

Prerequisites for Docker Compose:

Ensure Docker and Docker Compose are running.

Create a .env file in the root of your parent repository (same level as docker-compose.yml).

# Copy .env.example to .env and fill in the values

# General Secrets
JWT_SECRET="your_long_and_secure_base64_jwt_secret_here"
JWT_EXPIRATION=3600000

# Database Credentials
DB_USER=postgres
DB_PASSWORD=your_secure_db_password

# RabbitMQ Credentials
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest

# SMTP Credentials (for email-service, e.g., Gmail)
SPRING_MAIL_USERNAME="your_gmail_username@gmail.com"
SPRING_MAIL_PASSWORD="your_gmail_app_password"

# Google OAuth2 Client Credentials (for user-service)
CLIENT_ID="your_google_client_id"
CLIENT_SECRET="your_google_client_secret"

# SonarQube Token (for running local analysis if not in settings.xml or passed via -D)
SONAR_TOKEN="your_sonarqube_token"

To start all services with Docker Compose:
From the root of the parent repository:

docker-compose up --build -d

--build: Rebuilds images if Dockerfiles or source code have changed.

-d: Detached mode (runs in the background).

To stop the services:

docker-compose down

To stop and remove volumes (deletes DB data!):

docker-compose down -v

Key URLs (with default docker-compose.yml settings):

API Gateway: http://localhost:8080

Eureka Server: http://localhost:8761

RabbitMQ Management: http://localhost:15672 (user: guest, pass: guest)

Zipkin: http://localhost:9411

SonarQube (if included in Docker Compose): http://localhost:9000

9. Application Testing
Unit and Integration Tests:

Navigate to each microservice's directory.

Run tests with Maven:

mvn test

Or as part of the verify lifecycle:

mvn clean verify

End-to-End Testing (Manual):

Use a tool like Postman or cURL to send requests to the API Gateway (http://localhost:8080).

Registration and Login Flow:

POST /api/auth/register to create a user. Check the response and confirmation email.

POST /api/auth/login to get a JWT token.

Accreditation Creation Flow:

Use the obtained JWT.

Ensure SalePoints exist.

POST /api/accreditations with valid data.

Verify the response, record creation in accreditations_db, and the email with the PDF.

Check traces in Zipkin.

Test Protected Endpoints: Attempt to access admin endpoints without an admin token (expect 403), and user endpoints without any token (expect 401).

10. Code Quality and Coverage
JaCoCo (Coverage):

After running mvn clean test or mvn clean verify in a microservice, the HTML coverage report is in target/site/jacoco/index.html.

Class exclusions (DTOs, Configs, etc.) are configured in the jacoco-maven-plugin in each pom.xml.

SonarQube (Static Analysis & Quality):

Ensure your SonarQube server is running (http://localhost:9000).

Define the SONAR_TOKEN environment variable with your SonarQube token in the environment where Maven runs (or pass with -Dsonar.token).

From the parent repository's root directory (or an individual microservice's), run:

mvn clean verify sonar:sonar

Review results on the SonarQube dashboard. Properties like sonar.projectKey, sonar.projectName, sonar.host.url, and sonar.coverage.exclusions are defined in the pom.xml files.

11. API Documentation
Each microservice exposing a REST API (e.g., user-service, accreditations-service, sales-point-service) uses Springdoc OpenAPI to generate Swagger documentation.

Once services are running, you can access the Swagger UI for each:

Via API Gateway:

User Service: http://localhost:8080/user-service/swagger-ui.html

Accreditations Service: http://localhost:8080/accreditations-service/swagger-ui.html

Sales Point Service: http://localhost:8080/sales-point-service/swagger-ui.html


12. Security
Endpoint security is managed with Spring Security.

Services are configured as OAuth2 Resource Servers, validating JWTs.

user-service acts as the issuer of these JWTs.

Role-based authorization (@PreAuthorize) is used to restrict access.

The API Gateway also performs an initial layer of JWT validation.

Inter-service communication (using RestTemplate) propagates the original JWT using a ClientHttpRequestInterceptor.

13. Assumptions Made
Basic familiarity with Java, Spring Boot, Maven, Git, and Docker is assumed.

External service URLs (like PostgreSQL, RabbitMQ, Eureka, Zipkin when run locally outside Docker Compose) are defaulted to localhost with standard ports.

For Gmail email sending, an "App Password" is assumed to be generated and configured if the Gmail account has 2-factor authentication enabled.

The local network configuration allows communication between services and Docker instances.

An .env file will be created in the parent project root with necessary secrets and configurations.
