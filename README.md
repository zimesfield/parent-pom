# Zimesfield Platform

A enterprise-grade, multi-module Maven project providing a curated, production-ready platform for building microservices and reusable starter libraries. Designed with best practices for dependency management, build configuration, and Maven Central publication.

**Group ID:** `com.zimesfield`  
**Current Version:** `1.0.0`  
**License:** Apache License 2.0

---

## 📋 Project Overview

The Zimesfield Platform is built on a **separation of concerns** principle, where each POM file has a specific responsibility:

- **Version Management:** Single source of truth via `${revision}` property
- **Dependency Management:** Centralized BOM for consistent dependency versions
- **Plugin Management:** Shared plugin configuration for all modules
- **Release Automation:** Publication to Maven Central via Sonatype Central
- **Reusable Starters:** Pre-configured starter modules for microservices

---

## 🏗️ Project Structure
parent-pom/ 
├── pom.xml # Root aggregator (multi-module coordinator) 
├── platform-bom/ # Dependency Management BOM 
│ └── pom.xml 
├── platform-parent/ # Parent POM (plugin management & defaults) 
│ └── pom.xml 
├── platform-release/ # Release configuration aggregator 
│ └── pom.xml └── starters/ # Reusable starter modules 
├── service-starter/ # Spring Boot microservice starter 
├── modulith-starter/ # Spring Modulith modular starter 
├── openapi-starter/ # OpenAPI 3.0 + code generation starter 
└── library-starter/ # Generic reusable library starter
---

## 📦 Module Responsibilities

### 1. **Root POM** (`pom.xml`)
**Purpose:** Multi-module aggregator

- **GroupId:** `com.zimesfield`
- **ArtifactId:** `zimesfield-platform`
- **Packaging:** `pom`
- **Modules:** Lists all child modules for coordinated builds
- **Key Property:** `${revision}` = `1.0.0` (centralized versioning)

**Key Features:**
- Maven Wrapper included (`mvnw`, `mvnw.cmd`) for consistent Maven version
- SCM configuration pointing to GitHub
- Developer metadata (Cyprian Omenuko)
- Profile support for selective module publication

### 2. **Platform BOM** (`platform-bom/pom.xml`)
**Purpose:** Dependency version management (Bill of Materials)

- **ArtifactId:** `platform-bom`
- **Packaging:** `pom`
- **Scope:** Centralized version control

**Imported BOMs:**
- `org.springframework.boot:spring-boot-dependencies` (3.5.14)
- `org.springframework.cloud:spring-cloud-dependencies` (2025.1.1)
- `org.springframework.modulith:spring-modulith-bom` (1.4.11)
- `io.mongock:mongock-bom` (5.5.0)
- `com.playtika.reactivefeign:feign-reactor-bom` (4.2.1)

**Managed Dependencies Include:**
- **Spring:** Boot, Cloud, Modulith, OpenAPI (SpringDoc 2.8.17)
- **Database:** MongoDB (Mongock), Liquibase (4.33.0), H2, PostgreSQL
- **Code Generation:** MapStruct (1.6.3), Lombok (1.18.46)
- **Utilities:** ULID Creator (5.2.3), Problem Library (0.29.1)
- **Observability:** Micrometer (1.2.1)
- **OpenAPI:** Jackson DataBind Nullable (0.2.10)

**Build Plugins (PluginManagement):**
- Maven Compiler (3.15.0) - Java 21 compatible
- Maven Surefire/Failsafe (Testing)
- Spotless (2.46.1) - Code formatting
- JaCoCo (0.8.14) - Code coverage
- Maven Source/Javadoc (3.4.0, 3.12.0)
- Maven GPG (3.2.8) - Artifact signing
- Flatten Maven Plugin (1.6.0) - POM flattening for CI-friendly versions
- Sonatype Central Publishing (0.10.0) - Maven Central deployment

**Release Profile:**
Activates signing and source/javadoc generation when `mvn -Prelease install`

### 3. **Platform Parent** (`platform-parent/pom.xml`)
**Purpose:** Parent POM for all modules (plugin and build defaults)

- **ArtifactId:** `platform-parent`
- **Packaging:** `pom`
- **Java Version:** 21

**Key Properties:**
- `java.version` = 21
- `maven.compiler.release` = 21
- `project.build.sourceEncoding` = UTF-8
- Lombok version (1.18.38)
- MapStruct version (1.6.3)

**DependencyManagement:**
- Imports the BOM (`platform-bom`) via `<dependencyManagement><import/></dependencyManagement>`
- Ensures all child modules follow the same version constraints

**PluginManagement:**
- Compiler configuration with Lombok & MapStruct annotation processors
- Test plugins (Surefire, Failsafe)
- Code quality (Spotless, JaCoCo)
- Release tools (GPG, Source, Javadoc, Flatten, Sonatype Central)

---

### 4. **Platform Release** (`platform-release/pom.xml`)
**Purpose:** Release aggregator (coordinates publication to Maven Central)

- **ArtifactId:** `platform-release`
- **Packaging:** `pom`
- **Parent:** `platform-parent`

**Purpose:**
- Groups artifacts for release
- Activates signing and publication plugins via `mvn -Prelease deploy`
- Release profile enables GPG signing and Sonatype Central publishing

---

### 5. **Starters** (`starters/*/pom.xml`)

#### **Service Starter** - Microservice Starter
- **ArtifactId:** `service-starter`
- **Best for:** Building Spring Boot microservices with Zimesfield standards

**Profiles:**
- `dev` (default): H2 database, Spring DevTools, auto-reload
- `prod`: PostgreSQL configuration, production Liquibase settings
- `docker-compose`: Spring Boot Docker Compose support
- `no-liquibase`: Skip database migrations
- `api-docs`: Enable OpenAPI documentation

**Dependencies (Transitive via BOM):**
- Spring Boot Web/WebFlux
- Spring Cloud
- Liquibase (database versioning)
- Testing (TestContainers)
- Build info (git-commit-id-maven-plugin)

**Configuration:**
- Customizable properties: `start-class`, `application-name`, `db-url`, `db-username`, `db-password`
- Liquibase master file: `src/main/resources/config/liquibase/master.yaml`

---

#### **Modulith Starter** - Modular Architecture Starter
- **ArtifactId:** `modulith-starter`
- **Best for:** Building modular monoliths with Spring Modulith

**Direct Dependencies:**
- `org.springframework.modulith:spring-modulith-starter-core`

**Use Cases:**
- Domain-driven design with module boundaries
- Automatic API documentation between modules
- Event-driven module communication

---

#### **OpenAPI Starter** - API-First Development Starter
- **ArtifactId:** `openapi-starter`
- **Best for:** API-first development with automatic code generation

**Direct Dependencies:**
- Spring Boot WebFlux, AOP, AutoConfigure
- SpringDoc OpenAPI (WebFlux)
- Zalando Problem Library (structured error responses)

**Features:**
- OpenAPI 3.0 code generation (v7.22.0)
- Spring Controller generation
- Reactive (WebFlux) support
- Type mappings for Instant, BigDecimal, Problem types
- Customizable API package structure

**Configuration Properties:**
- `api-input-spec`: OpenAPI YAML/JSON file path
- `api-package`: Generated API package
- `api-model-package`: Generated models package

---

#### **Library Starter** - Reusable Library Starter
- **ArtifactId:** `library-starter`
- **Best for:** Building reusable libraries/utilities

**Minimal Dependencies:**
- Lombok only

**Plugins (PluginManagement):**
- Standard build plugins (Checkstyle, Compiler, Enforcer, etc.)
- Source/Javadoc generation
- Spring Boot Maven Plugin

---

## 🚀 Key Features & Best Practices

### ✅ Centralized Version Management
```xml
<!-- Single source of truth -->
<revision>1.0.0</revision>

<!-- All modules inherit: -->
<version>${revision}</version>

