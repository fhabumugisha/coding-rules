---
description: Active on Java 21+, Spring Boot 4 and Spring Framework 7 — backend rules and best practices
---

# AI Rules for Spring Boot 4 / Spring Framework 7 Projects

## Scope
Rules target backend services implemented with Java (recommended baseline: Java 21+), Spring Boot 4 and Spring Framework 7. Verify the Spring Boot 4 minimum Java version in official release notes before finalizing toolchains.

## General Guidelines
- Act as a **Senior Java Developer** and apply SOLID, DRY, KISS, YAGNI.
- Target **Java 21+** and **Spring Boot 4** conventions.
- Follow **OWASP** security best practices (input validation, secure defaults, CSRF protection where applicable).
- Produce clean, production-ready code with clear comments and Javadoc for public APIs.

## Project Structure & Packaging
- Build tool: **Maven** (keep pom.xml tidy; use dependencyManagement).
- Organize code by feature (feature-driven packages) into controller, service, repository, model, dto, config, and mapping packages.
- Use package naming that expresses features, e.g., `com.example.orders.controller`, `com.example.orders.service`.
- Prefer immutable DTOs (Java records or final classes) for API boundaries.
- Add package declaration and all imports in code examples.

## Language & Libraries
- Use `jakarta.*` APIs (e.g., `jakarta.persistence`, `jakarta.validation`) — no `javax.*`.
- Use Lombok for boilerplate reduction (e.g., `@RequiredArgsConstructor`, `@Getter`) but:
  - Avoid `@Data` on JPA entities; prefer explicit implementations or `@Getter`/`@Setter`.
  - Prefer records for DTOs where immutability is desired.
- Use MapStruct (or small hand-written mappers) to convert between entities and DTOs. MapStruct works well with records.

## Configuration
- Use `application.yaml` for environment-specific configuration.
- Prefer `@ConfigurationProperties` with constructor binding (records or immutable classes supported).
- Use `MessageSource` for i18n:
  - Provide `messages.properties`, `messages_en.properties`, `messages_fr.properties`, etc.
  - Validation messages must reference message keys.

## Entities & Repositories
- Entities in `com.example.model`:
  - Use `@Entity` and `@Table(name = "table_name").`
  - Use `@Id @GeneratedValue(strategy = GenerationType.IDENTITY)` (verify DB strategy).
  - Use `jakarta.persistence` annotations.
  - Use `@JsonIgnore` or DTO mapping to prevent infinite recursion; prefer DTOs over exposing entities.
  - Use classes for JPA entities (do NOT use records as managed entities).
  - Guard equals/hashCode implementations; avoid using primary key values if entities are not yet persisted.
  - Consider soft-delete fields (e.g., `deleted_at`), and implement queries that exclude soft-deleted rows (use filters or explicit where clauses).
  - Use auditing fields (createdBy, createdDate, lastModifiedBy, lastModifiedDate) and Spring Data auditing (`org.springframework.data.annotation.*`).

- Repositories in `com.example.repository`:
  - Extend `JpaRepository<Entity, Long>` or appropriate ID type.
  - Use `@Query` for custom queries and `@EntityGraph` for fetch optimization.
  - Use projections or DTO-based queries where needed for performance.

## Service Layer
- Define service interfaces and `@Service` implementations.
- Use constructor-based dependency injection (Lombok's `@RequiredArgsConstructor` is acceptable).
- Annotate transactional boundaries with `@Transactional` (from `org.springframework.transaction.annotation`).
- Add Javadoc for all service interface methods.

## Controller Guidelines
- Place controllers in `com.example.controller`.
- Use `@RestController` and `@RequestMapping("/api/resource").`
- Use `@GetMapping`, `@PostMapping`, etc., and explicit request/response DTOs.
- Use `jakarta.validation.Valid` and constraint annotations from `jakarta.validation.constraints`.
- Return `ResponseEntity<ApiResponse<T>>` or a standardized problem details format (RFC 7807) for errors.
- Document APIs with OpenAPI (springdoc or compatible library) and keep docs up-to-date.

## Exception Handling
- Implement a global exception handler (`@ControllerAdvice`) that:
  - Converts exceptions into consistent API error responses (consider RFC 7807).
  - Handles `ResourceNotFoundException` (404), `ValidationException` (400), `AccessDeniedException` (403).
  - Logs exceptions appropriately and avoids leaking sensitive information.

## Security Best Practices
- Use Spring Security with JWT-based authentication or OAuth2 resource server as appropriate.
- Configure security using bean-based approach with `SecurityFilterChain` (do not use deprecated WebSecurityConfigurerAdapter).
- Protect endpoints with method-level security (`@PreAuthorize`) where needed.
- Validate and whitelist CORS origins, use CSRF protection for browser-based endpoints, and store secrets in vaults/secure stores.

## Spring AI 2 Best Practices
- When integrating Spring AI 2, centralize LLM configuration and follow AIOps best practices:
  - Use a multiLL (multi-LLM / multi-provider) configuration strategy so the application can route requests to a primary model and failover or use specialized models for different tasks. Expose provider/model configuration via `@ConfigurationProperties` and support runtime overrides.
  - Externalize prompts and prompt templates into a versioned `/prompts` folder (e.g., `src/main/resources/prompts/`) and reference them by key. Keep prompts as text files, support parameter placeholders, and include tests for prompt validation and expected outputs. Version prompts and keep them out of code to enable safe editing and review.
  - Trace and attribute every AI call and its cost per user: capture model used, tokens/payload size, response size, estimated price per-call (based on provider pricing), and associate costs with a user or tenant id. Persist call metadata to a billing/telemetry store for auditing and quota enforcement. Anonymize or redact PII before storage and implement caching and rate-limiting to reduce cost and latency.
  - Additional safeguards: validate inputs to prompts, limit token sizes, use deterministic settings where appropriate, implement retries with exponential backoff, and secure API keys via secrets management. Emit structured logs and metrics (model latency, token usage, cost) to your observability stack.

## Testing
- Use JUnit 5.
- Unit tests: Mockito (MockitoExtension) for service layer tests.
- Web layer tests: MockMvc for servlet apps; WebTestClient for reactive apps.
- Repository tests: `@DataJpaTest`.
- Integration tests: `@SpringBootTest` with Testcontainers for real DBs; prefer Testcontainers over embedded DBs for parity.
- Follow Given-When-Then structure.
- Use Maven surefire and failsafe plugins; ensure CI runs tests and integration tests.
- Prioritize meaningful tests over raw coverage numbers.

## Build, CI & Deployment
- Continue with Maven dependency management. Keep plugins compatible with Java 21 (maven-compiler-plugin, surefire, failsafe).
- Provide separate profiles for dev, test, prod.
- Containerization: use Docker and a JRE/JDK 21 base image. Consider JIB for registry publishing.
- If targeting native/AOT images, follow Spring AOT guidance and test native images in CI.

## Observability & Documentation
- Expose actuator endpoints (metrics, health) and secure them.
- Use structured logging (JSON) and centralized log collection.
- Provide OpenAPI docs and a brief README for each module.

## Example snippets (use jakarta.* imports) 

### Sample Entity (jakarta.persistence)
```java
package com.example.users.model;

import jakarta.persistence.*;
import com.fasterxml.jackson.annotation.JsonIgnore;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(nullable = false)
    private String password;

    // Avoid exposing sensitive fields in DTOs; do not return entity directly from controllers
}
```

### Sample Repository
```java
package com.example.users.repository;

import com.example.users.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

### Sample DTO (record) and Mapper (MapStruct)
```java
package com.example.users.dto;

public record UserDto(Long id, String username) {}
```

```java
// MapStruct interface example
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDto toDto(User user);
    User toEntity(UserDto dto);
}
```

### Sample Controller
```java
package com.example.users.controller;

import com.example.users.dto.UserDto;
import com.example.users.service.UserService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getUserById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }
}
```

## Notes and caveats
- Verify Spring Boot 4 release notes for any removed or changed APIs that affect your project.
- Test all third-party libraries for Jakarta / Java 21 compatibility.
- Avoid premature optimization; apply AOT changes only if you plan native/AOT builds.

## Migration checklist (short)
- javax.* → jakarta.*
- Upgrade Java and CI images to Java 21
- Upgrade dependencies to compatible versions
- Convert security config to SecurityFilterChain style
- Run tests, fix JPA/serialization/validation issues
- Audit and update third-party libs
