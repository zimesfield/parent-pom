# parent-pom

This repository is a parent multi-module Maven project that provides a curated platform for building microservice starters and release artifacts that are safe to publish to Maven Central.

It contains:
- `parent-pom` (root aggregator / base pom)
- `platform-bom/` (dependencyManagement BOM)
- `platform-parent/` (parent POM for modules: plugin configuration, plugin versions, build defaults)
- `platform-release/` (release artifacts aggregator, coordinate packaging for publication)
- `starters/` (reusable starter modules)
    - `service-starter/` (opinionated starter for microservices)
    - `modulith-starter/`
    - `openapi-starter/`
    - `library-starter/`

Goals:
- Provide a single source of versions via a BOM to avoid dependency conflicts.
- Provide a consistent parent POM with best-practice plugin and build configuration.
- Produce artifacts that can be released to Maven Central (OSSRH).
- Provide reusable starters for microservice projects.

## Recommended POM responsibilities

We recommend splitting responsibilities across POMs:

1. Root `pom.xml` (aggregator)
    - Packaging: `pom`
    - Modules: list `platform-bom`, `platform-parent`, `platform-release`, `starters/*`
    - Minimal plugin configuration. Aggregates modules so `mvn -pl ...` works.

2. `platform-bom/pom.xml`
    - Packaging: `pom`
    - Contains `<dependencyManagement>` with exact versions for all shared dependencies (Spring/Guava/Jackson etc.)
    - Exportable as BOM to let consumers import versions via `<dependencyManagement><import scope="import"/></dependencyManagement>`

3. `platform-parent/pom.xml`
    - Packaging: `pom`
    - Inherits the BOM (via `<dependencyManagement>`) or references it in `<dependencyManagement>` to ensure child modules follow versions.
    - Centralizes plugin management and build defaults (maven-compiler-plugin, maven-surefire-plugin, enforcer plugin, encoding).
    - Declares properties for Java version, project.version, groupId, plugin versions.

4. `platform-release/pom.xml`
    - Packaging: `pom`
    - This module lists the artifacts intended for release to Maven Central (starter JARs, BOM, other libraries).
    - Configures distributionManagement (or inherits from parent) for release automation (note: credentials are provided at release-time via CI)

5. `starters/service-starter/pom.xml`
    - Depends on the parent and imports the BOM to inherit versions.
    - Minimal dependencies and configuration for a microservice starter (spring-boot-starter, actuator, logging, tracing if desired)
    - Should be publishable as a library / starter with proper metadata (name, description, license, scm, developers).

## Example POM snippets

Note: `com.example` and `1.0.0-SNAPSHOT` with your groupId/version.

Root aggregator (`pom.xml`):
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.zimesfield</groupId>
  <artifactId>parent-pom</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>

  <modules>
    <module>platform-bom</module>
    <module>platform-parent</module>
    <module>platform-release</module>
    <module>starters/library-starter</module>
    <module>starters/service-starter</module>
    <!-- etc -->
  </modules>
</project>
