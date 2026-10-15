# Hotel Management System --- Microservices Architecture

A backend-focused Hotel Management System built with **Java, Spring
Boot, and Spring Cloud**. The project demonstrates how user, hotel, and
rating responsibilities can be separated into independent microservices,
with service discovery, inter-service communication, gateway routing,
centralized configuration, fault tolerance, and token-based security.

> **Note:** This README describes the intended learning-project
> architecture. Verify every feature, endpoint, port, and dependency
> against your repository before claiming it is implemented. Replace the
> repository URL placeholder below with your actual GitHub URL.

## Table of Contents

-   [Overview](#overview)
-   [Architecture](#architecture)
-   [Technology Stack](#technology-stack)
-   [Services and Components](#services-and-components)
-   [Request Flow](#request-flow)
-   [Prerequisites](#prerequisites)
-   [Getting Started](#getting-started)
-   [Configuration](#configuration)
-   [Example API Endpoints](#example-api-endpoints)
-   [Testing](#testing)
-   [Implementation Checklist](#implementation-checklist)
-   [Troubleshooting](#troubleshooting)
-   [Future Improvements](#future-improvements)
-   [Learning Outcomes](#learning-outcomes)

## Overview

The system is organized around three core business services:

-   **User Service** --- manages user information.
-   **Hotel Service** --- manages hotel information.
-   **Rating Service** --- manages ratings and feedback associated with
    users and hotels.

Supporting Spring Cloud components can provide service discovery,
gateway routing, centralized configuration, and fault-tolerance
patterns. The tutorial scope also covers Spring Security and Okta for
token-based authentication and authorization.

Each business service exposes REST APIs. Services should access their
own data through their own persistence layer and communicate with other
services through APIs rather than directly querying each other's
databases.

## Architecture

``` text
                       Client / Postman
                              |
                              v
                     Spring Cloud Gateway
                       /      |      \
                      v       v       v
                User Service Hotel Service Rating Service
                      \       |       /
                       \-- HTTP calls --/
                              |
                      Eureka Registry

      Config Server  -> External configuration
      MySQL           -> Business data persistence
      Okta + Security -> Token validation / authorization
      Resilience4j    -> Fault-tolerance patterns
```

This is a conceptual architecture. The exact request flow depends on the
endpoints implemented in the source code. For example, a combined
response containing user, hotel, and rating data must be explicitly
implemented; Eureka and the Gateway do not create that aggregation
automatically.

## Technology Stack

  Technology                    Purpose
  ----------------------------- ----------------------------------------------
  Java                          Primary programming language
  Spring Boot                   Independent REST services
  Spring Web                    REST controllers and HTTP APIs
  Spring Data JPA / Hibernate   Persistence layer
  MySQL                         Relational data storage
  Spring Cloud Netflix Eureka   Service registration and discovery
  Spring Cloud OpenFeign        Declarative service-to-service HTTP clients
  Spring Cloud Gateway          Request routing
  Spring Cloud Config           Centralized external configuration
  Resilience4j                  Circuit breaker, retry, and rate limiting
  Spring Security               Authentication and authorization integration
  Okta                          Identity provider in the tutorial scope
  Maven                         Build and dependency management
  Postman                       API testing

Use compatible Spring Boot and Spring Cloud versions. Check the actual
`pom.xml` files for the versions and dependencies used by your
implementation.

## Services and Components

### 1. User Service

Responsible for user data and user-related APIs, such as creating a user
and retrieving a user by ID.

Example fields may include `id`, `name`, `email`, and `about`.

### 2. Hotel Service

Responsible for hotel information, such as hotel name, location, and
description.

### 3. Rating Service

Responsible for ratings and feedback associated with users and hotels.
Example fields may include `id`, `userId`, `hotelId`, `rating`, and
`feedback`.

### 4. Eureka Service Registry

Services register their instance information with Eureka. Clients using
discovery can resolve a logical service name to an available instance.
Eureka is a registry; it does not store business records.

### 5. API Gateway

Provides a common entry point for client requests and routes matching
paths to the appropriate service. Typical route patterns include
`/users/**`, `/hotels/**`, and `/ratings/**`, depending on the actual
controller mappings.

### 6. Config Server

Serves externalized configuration from a configured backend, commonly a
Git repository. Client import settings and refresh behaviour depend on
the Spring Boot / Spring Cloud versions and setup.

### 7. Resilience4j

Can be configured for: - **Circuit Breaker** --- temporarily
short-circuits calls when configured failure conditions are met. -
**Retry** --- retries selected failures within configured limits. -
**Rate Limiter** --- restricts the permitted rate of calls.

These features must be configured and tested; adding an annotation alone
does not confirm that the behaviour works.

### 8. Security with Okta

The tutorial scope covers Spring Security and Okta for token-based
authentication and authorization. OAuth 2.0 is an authorization
framework, OIDC adds an identity layer, and JWT is a token format. The
exact flow depends on the issuer, client, scopes, and resource-server
configuration.

## Request Flow

A typical gateway-routed request can follow this sequence:

1.  A client sends a request to the API Gateway.
2.  The Gateway matches a route and resolves the target service,
    potentially using Eureka and load balancing.
3.  The request reaches the target service's controller.
4.  The service layer performs business logic and calls its repository
    if database access is needed.
5.  If another service's data is required, an HTTP client such as
    OpenFeign can call that service.
6.  The service builds its response and returns it through the Gateway.

A combined response that includes data from multiple services must be
implemented explicitly in the application code.

## Prerequisites

-   JDK compatible with the project's Spring Boot version
-   Maven (or Maven Wrapper, if included)
-   MySQL Server
-   IntelliJ IDEA, Eclipse, or another Java IDE
-   Postman or another HTTP client
-   Git

If enabled, you may also need a Git repository for Spring Cloud Config
and an Okta issuer/application configuration.

## Getting Started

> **Important:** Commands, database names, and ports below are examples.
> Check the actual service folders, `pom.xml`, `application.properties`
> / `application.yml`, and Config Server repository before running the
> project.

### 1. Clone the repository

``` bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

Replace the placeholders with your real repository URL and folder name.

### 2. Create the databases

If each service uses a separate database, create the database
names configured in your project. Example:


Use the names from your actual configuration if they differ.

### 3. Configure environment-specific values

Set the MySQL credentials, JDBC URLs, Eureka URL, Config Server URL, and
Okta settings where applicable.

**Never commit real passwords, client secrets, access tokens, or private
credentials to GitHub.** Use environment variables or an appropriate
secret-management solution.

### 4. Build each service

From each service directory, run:

``` bash
mvn clean package
```

If the repository includes Maven Wrapper, use `./mvnw clean package` on
Linux/macOS or `mvnw.cmd clean package` on Windows.

### 5. Start the components

A typical local startup order is:

1.  MySQL
2.  Config Server, if clients require it during startup
3.  Eureka Server
4.  User Service
5.  Hotel Service
6.  Rating Service
7.  API Gateway

The exact order depends on startup dependencies. From each service
directory, you can usually run:

``` bash
mvn spring-boot:run
```

Run the command separately for each application unless your repository
is explicitly configured as a multi-module Maven build.

### 6. Verify service registration

Open the Eureka dashboard using the host and port configured in your
project. A common local default is `http://localhost:8761`, but verify
your actual settings. Confirm that the expected services are registered.

### 7. Test the APIs

Test each service directly with Postman first. Then test the
corresponding Gateway routes. Confirm the real controller mappings and
route configuration before using the examples below.

## Configuration

Check these settings in the relevant configuration files:

-   `spring.application.name`
-   `server.port`
-   `spring.datasource.url`
-   `spring.datasource.username`
-   `spring.datasource.password`
-   `eureka.client.service-url.defaultZone`
-   Config Server import / bootstrap settings
-   Gateway routes
-   Okta issuer, scopes, and security rules
-   Resilience4j timeouts and thresholds

Example ports and service names in tutorials may differ from your
repository.

## Example API Endpoints

The paths below are **illustrative**. Confirm the actual controller
mappings before testing.

  -----------------------------------------------------------------------------------
  Service           Method            Example path                  Purpose
  ----------------- ----------------- ----------------------------- -----------------
  User Service      `POST`            `/users`                      Create a user

  User Service      `GET`             `/users/{id}`                 Retrieve a user

  Hotel Service     `POST`            `/hotels`                     Create a hotel

  Hotel Service     `GET`             `/hotels`                     List hotels

  Hotel Service     `GET`             `/hotels/{id}`                Retrieve a hotel

  Rating Service    `POST`            `/ratings`                    Create a rating

  Rating Service    `GET`             `/ratings/users/{userId}`     Retrieve a user's
                                                                    ratings

  Rating Service    `GET`             `/ratings/hotels/{hotelId}`   Retrieve a
                                                                    hotel's ratings
  -----------------------------------------------------------------------------------

### Example: Create a user

``` http
POST /users
Content-Type: application/json
```

``` json
{
  "name": "Example User",
  "email": "user@example.com",
  "about": "Hotel management system user"
}
```

The actual fields and response format depend on your entity, DTOs,
validation, and controller implementation.

## Testing

Recommended verification sequence:

1.  Confirm each service starts independently.
2.  Test each service's API directly with Postman.
3.  Confirm that services register with Eureka.
4.  Test OpenFeign calls between services.
5.  Test Gateway routes.
6.  Verify Config Server configuration loading.
7.  Stop a downstream service and observe timeout / Circuit Breaker
    behaviour if configured.
8.  Test retry and rate limiting using safe requests.
9.  Test protected endpoints with and without a valid token if security
    is enabled.

Mark a feature complete only after it has been tested in the running
application.

## Implementation Checklist

Update this checklist to match the actual state of your repository.

-   [ ] User Service APIs and persistence
-   [ ] Hotel Service APIs and persistence
-   [ ] Rating Service APIs and persistence
-   [ ] Eureka Server and service registration
-   [ ] OpenFeign inter-service communication
-   [ ] Spring Cloud Gateway routes
-   [ ] Spring Cloud Config Server and client configuration
-   [ ] Resilience4j Circuit Breaker
-   [ ] Resilience4j Retry
-   [ ] Resilience4j Rate Limiter
-   [ ] Spring Security / Okta token validation
-   [ ] End-to-end API tests
-   [ ] README setup instructions verified against the repository

## Troubleshooting

  -----------------------------------------------------------------------
  Issue                               Things to check
  ----------------------------------- -----------------------------------
  Maven dependency resolution fails   Spring Boot / Spring Cloud
                                      compatibility, dependency
                                      management, Maven connectivity

  MySQL connection fails              MySQL status, database name, port,
                                      username/password, JDBC URL

  Service does not appear in Eureka   Eureka URL, client dependency,
                                      application name, startup logs

  Feign returns 404                   Service ID, controller path, HTTP
                                      method, path variables

  Feign connection refused            Target service status, registered
                                      instance, host/port

  Gateway returns 404                 Route ID, path predicate, target
                                      service ID, route configuration

  Config Server cannot load config    Git URL, file names, profile,
                                      client import settings, credentials

  Circuit Breaker does not activate   Dependency/version, annotation,
                                      instance configuration, thresholds

  API returns 401                     Token, issuer, expiry, signature,
                                      audience

  API returns 403                     Required permission or scope may be
                                      missing

  Feign request is unauthorized       Token propagation and downstream
                                      security rules
  -----------------------------------------------------------------------

## Future Improvements

If not already implemented, possible enhancements include:

-   Dockerfiles and Docker Compose for local orchestration
-   Centralized exception handling and request validation
-   DTOs instead of exposing persistence entities directly
-   OpenAPI / Swagger documentation
-   Unit and integration tests with JUnit and Mockito
-   Structured logging, correlation IDs, and distributed tracing
-   Health checks and metrics
-   CI pipeline for automated builds and tests
-   Secure secret management and environment-specific configuration

## Learning Outcomes

This project provides practice with:

-   Designing service boundaries in a Spring Boot microservices
    application
-   Building REST APIs and database-backed service layers
-   Discovering services with Eureka
-   Making service-to-service calls with OpenFeign
-   Routing requests with Spring Cloud Gateway
-   Externalizing configuration with Spring Cloud Config
-   Understanding fault-tolerance patterns with Resilience4j
-   Integrating Spring Security with an identity provider
-   Testing service interactions and failure scenarios

## License and Attribution 
Distributed under the MIT License. Developed by Abhishek Jamkar.

Distributed under the MIT License. Developed by **Abhishek Jamkar**.
Distributed under the MIT License. Developed by **Abhishek Jamkar**.where appropriate and follow its licensing terms.
