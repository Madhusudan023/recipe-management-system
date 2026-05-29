# recipe-management-system
# Online Recipe Manager — Backend Microservices

A production-style backend built with **Spring Boot microservices**, featuring JWT authentication, external API integration with Spoonacular, and full service orchestration via Spring Cloud.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Microservices](#microservices)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [API Documentation](#api-documentation)
- [API Endpoints](#api-endpoints)
- [Running Tests](#running-tests)
- [Project Structure](#project-structure)
- [User Stories Covered](#user-stories-covered)

---

## Project Overview

The Online Recipe Manager exposes RESTful APIs for:

- User registration and login with JWT-secured access
- Browsing and searching recipes powered by the [Spoonacular Food API](https://spoonacular.com/food-api)
- Saving and managing favourite recipes per user
- Creating, viewing, and deleting personal meal plans

All APIs are testable via Swagger UI or Postman. There is no frontend — this is a backend-only project.

---

## Architecture

# Online Recipe Manager - Microservices Architecture

## System Architecture

```text
                                ┌─────────────────────────────────────┐
                                │ Client (Postman / Swagger UI)      │
                                └────────────────┬────────────────────┘
                                                 │
                                                 ▼
                         ┌───────────────────────────────────────────┐
                         │               API Gateway                 │
                         │                 Port: 8080                │
                         │       JWT Validation & Request Routing    │
                         └───────┬──────────┬──────────┬────────────┘
                                 │          │          │
          ┌──────────────────────┘          │          └──────────────────────┐
          ▼                                 ▼                                 ▼
┌────────────────────┐          ┌────────────────────┐          ┌────────────────────┐
│    User Service    │          │   Recipe Service   │          │ Favourite Service  │
│     Port: 8081     │          │     Port: 8082     │          │     Port: 8083     │
│ Authentication     │          │ Recipe Management  │          │ User Favourites    │
└─────────┬──────────┘          └─────────┬──────────┘          └─────────┬──────────┘
          │                               │                               │
          └───────────────┬───────────────┴───────────────┬───────────────┘
                          │                               │
                          ▼                               ▼
               ┌────────────────────┐         ┌────────────────────┐
               │  Meal Plan Service │         │  Eureka Discovery  │
               │     Port: 8084     │         │     Port: 8761     │
               │ Meal Scheduling    │         │  Service Registry  │
               └─────────┬──────────┘         └────────────────────┘
                         │
                         ▼
               ┌────────────────────┐
               │   Config Server    │
               │     Port: 8888     │
               │ Centralized Config │
               └─────────┬──────────┘
                         │
         ┌───────────────┴────────────────┐
         ▼                                ▼
┌────────────────────┐        ┌─────────────────────────┐
│   MySQL Databases  │        │   Spoonacular API       │
│                    │        │    External API         │
│ • user_db          │        │ Recipe Data Provider    │
│ • recipe_db        │        └─────────────────────────┘
│ • favourites_db    │
│ • mealplan_db      │
└────────────────────┘
```

---

## Components Overview

### API Gateway
- Entry point for all client requests
- Performs JWT authentication and authorization
- Routes requests to appropriate microservices

### User Service
- User registration and login
- JWT token generation
- User profile management

### Recipe Service
- Recipe CRUD operations
- Integration with Spoonacular API
- Recipe search and filtering

### Favourite Service
- Save and manage favourite recipes
- User-specific favourite lists

### Meal Plan Service
- Weekly/monthly meal planning
- Recipe scheduling

### Eureka Discovery Server
- Service registration and discovery
- Enables dynamic communication between services

### Config Server
- Centralized configuration management
- Externalized `.properties` configuration

### MySQL Databases
Separate database for each microservice:
- `user_db`
- `recipe_db`
- `favourites_db`
- `mealplan_db`

### Spoonacular API
- External recipe and nutrition data provider

---

## Technology Stack

- Java 21
- Spring Boot
- Spring Cloud Gateway
- Spring Security + JWT
- Eureka Discovery Server
- Spring Cloud Config Server
- OpenFeign
- MySQL
- Maven
- Swagger / OpenAPI

---

**Key patterns used:**

- **API Gateway** — single entry point, JWT pre-validation, load-balanced routing via `lb://`
- **Service Discovery** — Netflix Eureka; services register by name, no hardcoded URLs
- **Config Server** — Spring Cloud Config with `native` (local filesystem) profile; all secrets and datasource config centralised here
- **Feign Client** — declarative HTTP calls between services using Eureka-resolved names
- **Async Messaging** — RabbitMQ for event publishing (e.g. meal plan created event)

---

## Tech Stack

| Category | Technology |
|---|---|
| Language | Java 21 |
| Framework | Spring Boot 3.5.0 |
| Security | Spring Security + JWT (jjwt 0.12.5) |
| Service Discovery | Netflix Eureka |
| API Gateway | Spring Cloud Gateway |
| Config Management | Spring Cloud Config (native/local) |
| Service Communication | OpenFeign |
| Message Bus | RabbitMQ |
| Database | MySQL 8 |
| ORM | Spring Data JPA / Hibernate |
| API Docs | SpringDoc OpenAPI (Swagger UI) |
| Testing | JUnit 5, Mockito, Spring Boot Test |
| Code Quality | JaCoCo, SonarLint |
| Build Tool | Maven |
| SCM | GitHub / GitLab |

---

## Microservices

| Service | Port | Responsibility |
|---|---|---|
| `config-server` | 8888 | Serves centralised `.properties` config to all services |
| `discovery-server` | 8761 | Eureka registry — all services register here |
| `api-gateway` | 8080 | Routes requests, validates JWT, single entry point |
| `user-service` | 8081 | Registration, login, JWT issuance, user profile |
| `recipe-service` | 8082 | Spoonacular integration — search, popular, detail |
| `favourites-service` | 8083 | Add, list, remove favourite recipes per user |
| `meal-plan-service` | 8084 | Create, view, delete meal plans |

---

## Prerequisites

Make sure the following are installed before running the project:

- Java 21+
- Maven 3.9+
- MySQL 8.0+
- RabbitMQ 3.x (running on default port 5672)
- A valid [Spoonacular API key](https://spoonacular.com/food-api) — free tier is sufficient

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-username/online-recipe-manager.git
cd online-recipe-manager
```

### 2. Create MySQL databases

Log in to MySQL and run:

```sql
CREATE DATABASE user_db;
CREATE DATABASE recipe_db;
CREATE DATABASE favourites_db;
CREATE DATABASE mealplan_db;
```

### 3. Set your secrets in the Config Server

Open `config-server/src/main/resources/config-repo/` and edit each `.properties` file with your actual values:
config-server/src/main/resources/config-repo/
├── user-service.properties
├── recipe-service.properties
├── favourites-service.properties
├── meal-plan-service.properties
└── api-gateway.properties
See the [Configuration](#configuration) section for what each file should contain.

### 4. Start services in order

Each service must start in the sequence below. Open a separate terminal for each:

```bash
# Terminal 1 — Config Server (must be first)
cd config-server
mvn spring-boot:run

# Terminal 2 — Eureka Discovery
cd discovery-server
mvn spring-boot:run

# Terminal 3 — User Service
cd user-service
mvn spring-boot:run

# Terminal 4 — Recipe Service
cd recipe-service
mvn spring-boot:run

# Terminal 5 — Favourites Service
cd favourites-service
mvn spring-boot:run

# Terminal 6 — Meal Plan Service
cd meal-plan-service
mvn spring-boot:run

# Terminal 7 — API Gateway (last)
cd api-gateway
mvn spring-boot:run
```

### 5. Verify everything is running

| Check | URL |
|---|---|
| Eureka dashboard | http://localhost:8761 |
| Config serving user-service | http://localhost:8888/user-service/default |
| User Service Swagger | http://localhost:8081/swagger-ui/index.html |
| Recipe Service Swagger | http://localhost:8082/swagger-ui/index.html |
| Gateway health | http://localhost:8080/actuator/health |

All four services should appear as `UP` in the Eureka dashboard before testing via the gateway.

---

## Configuration

All service configuration is managed centrally in the Config Server. Secrets are **never committed to source code**.

### `user-service.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/user_db
spring.datasource.username=YOUR_DB_USERNAME
spring.datasource.password=YOUR_DB_PASSWORD
spring.jpa.hibernate.ddl-auto=update

jwt.secret=YOUR_BASE64_ENCODED_256BIT_SECRET
jwt.expiration-ms=86400000
```

### `recipe-service.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/recipe_db
spring.datasource.username=YOUR_DB_USERNAME
spring.datasource.password=YOUR_DB_PASSWORD
spring.jpa.hibernate.ddl-auto=update

spoonacular.api-key=YOUR_SPOONACULAR_API_KEY
spoonacular.base-url=https://api.spoonacular.com
```

### `favourites-service.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/favourites_db
spring.datasource.username=YOUR_DB_USERNAME
spring.datasource.password=YOUR_DB_PASSWORD
spring.jpa.hibernate.ddl-auto=update
```

### `meal-plan-service.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mealplan_db
spring.datasource.username=YOUR_DB_USERNAME
spring.datasource.password=YOUR_DB_PASSWORD
spring.jpa.hibernate.ddl-auto=update

spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

> **Never commit actual credentials.** Add `config-server/src/main/resources/config-repo/*.properties`
> to `.gitignore` and share credentials with your team via a secure channel.

---

## API Documentation

Each service exposes Swagger UI at `/swagger-ui/index.html`.

| Service | Swagger URL |
|---|---|
| User Service | http://localhost:8081/swagger-ui/index.html |
| Recipe Service | http://localhost:8082/swagger-ui/index.html |
| Favourites Service | http://localhost:8083/swagger-ui/index.html |
| Meal Plan Service | http://localhost:8084/swagger-ui/index.html |

---

## API Endpoints

All endpoints below are accessed through the **API Gateway on port 8080**.

### Auth — no JWT required

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/users/register` | Register a new user |
| `POST` | `/api/users/login` | Login and receive JWT token |

### Users — JWT required

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/users/me` | Get authenticated user's profile |

### Recipes — JWT required

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/recipes/popular` | List popular/trending recipes |
| `GET` | `/api/recipes/search` | Search by name, cuisine, diet, or ingredients |
| `GET` | `/api/recipes/{id}` | Get full recipe detail with ingredients and steps |

### Favourites — JWT required

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/favourites` | Add a recipe to favourites |
| `GET` | `/api/favourites` | List all favourites for the logged-in user |
| `DELETE` | `/api/favourites/{recipeId}` | Remove a recipe from favourites |

### Meal Plans — JWT required

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/meal-plans` | Create a new meal plan |
| `GET` | `/api/meal-plans` | View all meal plans for the logged-in user |
| `DELETE` | `/api/meal-plans/{id}` | Delete a meal plan |

### Example: Login and use the token

```bash
# Step 1 — Login
curl -X POST http://localhost:8080/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"username": "john", "password": "password123"}'

# Response
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "tokenType": "Bearer",
  "expiresIn": 86400,
  "userId": "uuid-here"
}

# Step 2 — Use the token in subsequent requests
curl http://localhost:8080/api/recipes/popular \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

## Running Tests

### Run all tests

```bash
mvn test
```

### Run tests for a specific service

```bash
cd user-service
mvn test
```

### Generate JaCoCo coverage report

```bash
mvn verify
```

The coverage report is generated at `target/site/jacoco/index.html` inside each service module.

---

## Project Structure
# Project Structure

```text
online-recipe-manager/
│
├── config-server/                          # Spring Cloud Config Server
│   └── src/main/resources/
│       └── config-repo/                    # Centralized .properties files
│
├── discovery-server/                       # Netflix Eureka Discovery Server
│
├── api-gateway/                            # API Gateway Service
│   └── src/main/java/
│       └── filter/
│           └── JwtGatewayFilter.java       # JWT validation filter
│
├── user-service/                           # User Authentication Service
│   └── src/main/java/
│       ├── controller/                     # REST Controllers
│       ├── service/                        # Business Logic
│       ├── entity/                         # JPA Entities
│       ├── dto/                            # Request/Response DTOs
│       ├── repository/                     # JPA Repositories
│       ├── jwt_filter/                     # JWT Request Filter
│       ├── util/                           # Utility Classes
│       ├── conf/                           # Security & App Configurations
│       └── exceptions/                     # Custom Exceptions
│
├── recipe-service/                         # Recipe Management Service
│   └── src/main/java/
│       ├── controller/                     # Recipe APIs
│       ├── service/                        # Business Logic
│       ├── client/                         # OpenFeign Spoonacular Client
│       └── dto/                            # DTO Classes
│
├── favourites-service/                     # Favourite Recipes Service
│   └── src/main/java/
│       ├── controller/                     # Favourite APIs
│       ├── service/                        # Business Logic
│       ├── entity/                         # Favourite Entities
│       └── repository/                     # Database Repositories
│
├── meal-plan-service/                      # Meal Planning Service
│   └── src/main/java/
│       ├── controller/                     # Meal Plan APIs
│       ├── service/                        # Business Logic
│       ├── entity/                         # Meal Plan Entities
│       ├── repository/                     # Database Repositories
│       └── messaging/                      # RabbitMQ Event Handling
│
└── README.md                               # Project Documentation
```

---

# Folder Explanation

## config-server
Centralized configuration server that stores all microservice configuration files.

## discovery-server
Eureka server used for service registration and discovery.

## api-gateway
Single entry point for all client requests.
Handles:
- JWT validation
- Request routing
- Authentication filtering

## user-service
Handles:
- User registration
- Login authentication
- JWT token generation
- User management

## recipe-service
Handles:
- Recipe operations
- Spoonacular API integration
- Recipe searching and filtering

## favourites-service
Handles:
- Saving favourite recipes
- Managing user favourites

## meal-plan-service
Handles:
- Meal scheduling
- Meal plan management
- RabbitMQ asynchronous messaging

---

# User Stories Covered

| ID   | User Story                                                   | Service              |
|------|--------------------------------------------------------------|----------------------|
| US1  | Register with the application                                | `user-service`       |
| US2  | Login and receive a JWT token                                | `user-service`       |
| US3  | Browse popular recipes                                       | `recipe-service`     |
| US4  | Search recipes by name, cuisine, diet, and ingredients       | `recipe-service`     |
| US5  | View detailed recipe instructions and ingredients             | `recipe-service`     |
| US6  | Add a recipe to favourites                                   | `favourites-service` |
| US7  | View all favourite recipes                                   | `favourites-service` |
| US8  | Remove a recipe from favourites                              | `favourites-service` |
| US9  | Create a personal meal plan                                  | `meal-plan-service`  |
| US10 | View and delete a meal plan                                  | `meal-plan-service`  |

---
