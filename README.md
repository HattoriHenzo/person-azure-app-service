# Person Manager (application-spring)

Simple Spring Boot REST API for managing `Person` records (CRUD), built as a demo/learning project.

## Overview

The application exposes a REST API to create, read, update, and delete persons, backed by an in-memory H2 database accessed through Spring Data JPA. Validation and business rules (e.g. required fields, birth date not in the future) are enforced in the service layer, with dedicated exception types mapped to proper HTTP status codes.

## Tech Stack

- Java 21
- Spring Boot 4.1.2 (Spring Web, Spring Data JPA)
- H2 in-memory database
- Lombok
- Maven (with Maven Wrapper)
- JUnit 5, Mockito, AssertJ, Spring Boot Test (`@WebMvcTest`, `@DataJpaTest`, `@SpringBootTest` + `WebTestClient`)

## API Endpoints

| Method | Path            | Description               |
| ------ | --------------- | -------------------------- |
| GET    | `/persons`       | List all persons           |
| GET    | `/persons/{id}`  | Get a person by id         |
| POST   | `/persons`       | Create a new person        |
| PUT    | `/persons`       | Update an existing person  |
| DELETE | `/persons/{id}`  | Delete a person by id      |

A Postman collection is available in the [`postman`](postman) folder for manually exercising these endpoints.

## Getting Started

Build and run the application using the Maven Wrapper:

```bash
./mvnw clean install
./mvnw spring-boot:run
```

The H2 console is available at `/h2-console` once the application is running.

## Running Tests

```bash
./mvnw clean test
```
