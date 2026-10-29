# CLAUDE.md — MediStock

MediStock is a hospital supply inventory REST API: products, batches, department
stock, stock movements, purchase orders and alerts, with FEFO picking and expiry
enforcement. It is a portfolio project built by Moritz together with Claude Code.

**The full specification is in [`docs/spec.md`](docs/spec.md). Read it before
starting any task.** Where this file and the spec disagree, this file wins.

## Overrides to the spec

- **Java 25** (not 21). JDK: Amazon Corretto 25.
- **Spring Boot:** the current stable version, as set in the parent POM (not 3.x).
- Base Docker image: `amazoncorretto:25` (or an equivalent Corretto 25 image).

## Project structure

Multi-module Maven project. Parent POM in the root (`packaging: pom`), parent of
the parent is `spring-boot-starter-parent`.

```
medistock/
├── medistock-domain/        # JPA entities, enums, value objects
├── medistock-repository/    # Spring Data JPA repositories
├── medistock-service/       # Business logic, validation, scheduled jobs
├── medistock-api/           # REST controllers, DTOs, exception handling, security
├── medistock-bootstrap/     # @SpringBootApplication, config, Flyway migrations, seed data
├── docs/
└── pom.xml
```

### Dependency rules (enforced, do not break)

```
bootstrap → api → service → repository → domain
```

- A module may only depend on modules to its right. Never the reverse.
- Controllers never use repositories directly. They call services.
- Entities never leave the service layer through the API. Controllers use DTOs.
- Put each dependency in the module that uses it. Shared versions go in the
  parent's `<dependencyManagement>` (Spring Boot's BOM covers most of them).
- `spring-boot-maven-plugin` goes **only** in `medistock-bootstrap`.

### Packages

- Root package: `io.github.moritzkohr.medistock`
- `@SpringBootApplication` lives in the root package (bootstrap module).
- Each module uses its own sub-package: `...medistock.domain`,
  `...medistock.repository`, `...medistock.service`, `...medistock.api`.
- Group ID: `io.github.moritzkohr`. Artifact IDs match the module folder names.

## Build and run

```bash
./mvnw clean verify                              # build + all tests (Docker must run)
docker compose up -d postgres                    # database only
./mvnw -pl medistock-bootstrap -am spring-boot:run
docker compose up --build                        # full stack
```

Swagger UI: `http://localhost:8080/swagger-ui.html`

## Conventions

- **Build tool:** Maven only. No Gradle files.
- **Injection:** constructor injection only. No field `@Autowired`.
- **DTOs:** Java records. Map between entities and DTOs explicitly in the API
  layer; ask before adding a mapping library.
- **Schema:** Flyway migrations in `medistock-bootstrap/src/main/resources/db/migration`,
  named `V<n>__<description>.sql`. Never edit a migration that has been committed;
  add a new one. `spring.jpa.hibernate.ddl-auto=validate`.
- **Transactions:** `@Transactional` on service methods, not on controllers or
  repositories. Stock transfers must be atomic.
- **Errors:** one global `@RestControllerAdvice` with the error format from the spec.
  Business rule violations throw specific exceptions mapped to HTTP status codes.
- **Time:** `Instant` for timestamps, `LocalDate` for expiry dates. Inject a `Clock`
  wherever logic depends on "today", so tests can control it.
- **Configuration:** Spring profiles `local`, `test`, `prod`. No secrets in the repo;
  use environment variables and `.env` (git-ignored, with `.env.example` committed).
- **Language:** code, comments, commit messages and docs in English.

## Testing

- Unit tests for every service method (JUnit 5 + Mockito).
- Integration tests against real PostgreSQL via Testcontainers. Do not use H2.
- Business rules (FEFO, expiry block, low-stock alerts, order transitions) each
  need tests for the normal case and the edge cases.
- A task is only done when `./mvnw clean verify` passes.

## How to work with me

- **Stay in the current stage.** The spec defines three stages. Don't build
  features from a later stage unless I ask.
- **Work in small steps.** One coherent change at a time (e.g. one entity with its
  migration and repository), then stop so I can review it.
- **Ask before you:** add a dependency, change the module structure, change a
  committed migration, or deviate from the spec.
- **Explain non-obvious decisions** briefly when you make them. I need to be able
  to explain every part of this project in an interview.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `test:`, `refactor:`,
  `docs:`, `build:`, `chore:`). Small and focused. Don't commit or push unless I ask.
- Don't change `README.md` or the roadmap checkboxes without asking.
