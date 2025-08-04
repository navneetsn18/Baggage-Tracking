# Deployment Plan: Realtime Baggage Tracking System (Hackathon Edition)

**Version**: 1.0 (Hackathon)
**Date**: 2025-08-04

## 1. Overview

This document outlines a simplified strategy for containerizing and deploying the Realtime Baggage Tracking System for a 2-day hackathon. The focus is on getting a functional MVP up and running quickly, leveraging free-tier compatible AWS services and local development tools.

## 2. Dockerization Strategy

Every microservice and the frontend application will be containerized using Docker. This ensures a consistent and reproducible environment for local development and simplified deployment.

### 2.1. Microservice Dockerfile Template (Spring Boot)

Each Java-based microservice will have a `Dockerfile` at its root.

```dockerfile
# Stage 1: Build the application
FROM openjdk:17-jdk-slim as build
WORKDIR /app
COPY . .
RUN ./mvnw clean package -DskipTests

# Stage 2: Create the final, lightweight image
FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 2.2. Frontend Dockerfile Template (Next.js)

The Next.js frontend will also have a `Dockerfile`.

```dockerfile
# Stage 1: Build the application
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Create the final production image
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/public ./public
COPY --from=build /app/.next ./.next
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/package.json ./package.json
EXPOSE 3000
CMD ["npm", "start"]
```

### 2.3. Docker Compose for Local Development

A `docker-compose.yml` file will be created at the project root to orchestrate the entire system locally. This will define all services, including databases (PostgreSQL) and messaging (Kafka), allowing a developer to bring up the full stack with a single command: `docker-compose up`.

```yaml
version: '3.8'
services:
  # Example for one microservice
  auth-service:
    build: ./auth-service # Assuming auth-service is in a subfolder
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/authdb
      SPRING_DATASOURCE_USERNAME: user
      SPRING_DATASOURCE_PASSWORD: password
      KAFKA_BOOTSTRAP_SERVERS: kafka:9092
    depends_on:
      - postgres
      - kafka

  # Frontend
  frontend:
    build: ./frontend # Assuming frontend is in a subfolder
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_BASE_URL: http://localhost:8080 # Adjust as needed for API Gateway or direct service access

  # Database
  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: authdb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"

  # Kafka (simplified for local)
  kafka:
    image: confluentinc/cp-kafka:7.0.1
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    depends_on:
      - zookeeper

  zookeeper:
    image: confluentinc/cp-zookeeper:7.0.1
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
```

## 3. AWS Deployment Strategy (Hackathon MVP)

For a 2-day hackathon, we will deploy to a single, free-tier eligible AWS EC2 instance.

### 3.1. EC2 Setup

1.  **Launch EC2 Instance**: Launch a `t2.micro` (or `t3.micro`) instance in a public subnet.
2.  **Security Group**: Configure a security group to allow inbound traffic on ports 22 (SSH), 80 (HTTP), and 443 (HTTPS, if using Nginx/certbot). Also, allow ports for direct service access if not using Nginx (e.g., 3000 for frontend, 8080 for backend).
3.  **Install Docker**: SSH into the EC2 instance and install Docker and Docker Compose.

### 3.2. Deployment Steps

1.  **Build Docker Images Locally**: Build all microservice and frontend Docker images on your local machine.
2.  **Push to Docker Hub (or build on EC2)**:
    *   **Option A (Recommended for speed)**: Push your Docker images to a public Docker Hub repository.
    *   **Option B (Alternative)**: Copy your project source code to the EC2 instance and build the Docker images directly on the EC2 instance.
3.  **Transfer `docker-compose.yml`**: Copy the `docker-compose.yml` file to the EC2 instance.
4.  **Run Services**: On the EC2 instance, navigate to the directory containing `docker-compose.yml` and run `docker-compose up -d`. This will pull the images (if using Docker Hub) or build them (if using local source) and start all services.

### 3.3. CORS Resolution

The frontend and backend will run on different ports (or potentially different subdomains if using Nginx). CORS will be handled by configuring the Spring Boot microservices to allow requests from the frontend's origin.

Example (Spring Boot `WebConfig.java`):

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:3000", "http://YOUR_EC2_PUBLIC_IP:3000") // Adjust frontend URL
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true);
    }
}
```

## 4. Simplified Monitoring & Logging

-   **Logs**: All service logs will be accessible via `docker-compose logs -f`. For basic monitoring, `docker stats` can be used.
-   **Health Checks**: Basic HTTP health check endpoints will be implemented (e.g., `/health`) for manual verification.

## 5. Testing Strategy (Hackathon MVP)

-   **Unit Tests**: Developers are encouraged to write basic unit tests for critical logic.
-   **Manual Testing**: Primary testing will involve manual verification of features through the frontend.

## 6. Development Environment

-   Developers will use their preferred IDEs.
-   All services can be run locally using `docker-compose up` for a full development environment.