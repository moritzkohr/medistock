# MVP Spec — MediStock: Hospital Supply Inventory Platform

## Project summary

A multi-module Spring Boot REST API for managing medical supplies across hospital
departments. Demonstrates enterprise Java architecture, relational data modelling,
Docker containerisation, and AWS deployment. Built with Claude Code.

**Target employers:** just.nerds, Brockhaus AG, x-root
**Repo name:** `medistock`
**Language/runtime:** Java 21, Spring Boot 3.x
**Build tool:** Maven (multi-module)

---

## Module structure

```
medistock/
├── medistock-domain/        # JPA entities, enums, shared constants
├── medistock-repository/    # Spring Data JPA repositories
├── medistock-service/       # Business logic, validation, scheduling
├── medistock-api/           # REST controllers, DTOs, exception handling
├── medistock-bootstrap/     # Main application entry point, seed data
└── docker-compose.yml
```

---

## Core entities

| Entity | Key fields |
|---|---|
| `Product` | id, name, category (enum), unit, minimumStockLevel, requiresColdStorage |
| `Batch` | id, product, batchNumber, expiryDate, quantity, receivedAt |
| `Department` | id, name, location, contactPerson, hasColdStorage |
| `StockLevel` | id, product, department, currentQuantity, lastUpdated |
| `StockMovement` | id, product, batch, fromDepartment, toDepartment, quantity, type (enum), movedAt, movedBy |
| `Supplier` | id, name, contactEmail, phone, leadTimeDays |
| `PurchaseOrder` | id, supplier, product, quantity, status (enum), orderedAt, expectedDelivery |
| `Alert` | id, type (enum), product, department, message, resolved, createdAt |

### Key enums

- `ProductCategory`: MEDICATION, SURGICAL, DIAGNOSTIC, CONSUMABLE, EQUIPMENT
- `MovementType`: INTAKE, TRANSFER, USAGE, DISPOSAL, RETURN
- `OrderStatus`: DRAFT, SUBMITTED, CONFIRMED, DELIVERED, CANCELLED
- `AlertType`: LOW_STOCK, EXPIRY_WARNING, EXPIRED, COLD_CHAIN_BREACH

---

## Business logic (service layer)

These are the non-trivial rules that make the domain interesting:

- **Expiry enforcement:** batches expiring within 30 days trigger an `EXPIRY_WARNING`
  alert automatically; expired batches cannot be used in stock movements
- **FEFO picking:** stock movements always consume the earliest-expiring batch first
  (First Expired, First Out)
- **Critical stock threshold:** when `currentQuantity` drops below
  `product.minimumStockLevel`, a `LOW_STOCK` alert is created and a draft
  `PurchaseOrder` is generated automatically
- **Cold storage flag:** products with `requiresColdStorage = true` are flagged
  in the UI and excluded from departments where `hasColdStorage = false`
- **Transfer validation:** a transfer movement must decrement the source
  department's `StockLevel` and increment the destination's atomically
  (handled in a `@Transactional` service method)
- **Scheduled jobs:** a daily `@Scheduled` task scans all batches for expiry
  warnings and generates alerts; a second job checks stock levels department-wide

---

## REST API endpoints

### Products
```
GET    /api/v1/products
POST   /api/v1/products
GET    /api/v1/products/{id}
PUT    /api/v1/products/{id}
DELETE /api/v1/products/{id}
GET    /api/v1/products/{id}/stock        # stock levels across all departments
GET    /api/v1/products/low-stock         # products below minimum threshold
```

### Batches
```
GET    /api/v1/batches
POST   /api/v1/batches                    # intake a new batch
GET    /api/v1/batches/{id}
GET    /api/v1/batches/expiring?days=30   # batches expiring within N days
```

### Departments
```
GET    /api/v1/departments
POST   /api/v1/departments
GET    /api/v1/departments/{id}
GET    /api/v1/departments/{id}/stock     # full stock list for a department
```

### Stock movements
```
GET    /api/v1/movements
POST   /api/v1/movements                  # create a transfer, usage, or disposal
GET    /api/v1/movements?productId=&departmentId=&from=&to=
```

### Purchase orders
```
GET    /api/v1/orders
POST   /api/v1/orders
GET    /api/v1/orders/{id}
PATCH  /api/v1/orders/{id}/status        # advance order through status lifecycle
```

### Alerts
```
GET    /api/v1/alerts
GET    /api/v1/alerts?resolved=false
PATCH  /api/v1/alerts/{id}/resolve
```

### Auth
```
POST   /api/v1/auth/login                # returns JWT
```

---

## Security

- Spring Security with JWT authentication
- Two roles: `ROLE_ADMIN` (full access), `ROLE_VIEWER` (read-only, for recruiters)
- Recruiter access: create a `VIEWER` account per recruiter, share credentials
  directly — no AWS-level access control needed

---

## Tech stack

| Layer | Technology |
|---|---|
| Language | Java 21 |
| Framework | Spring Boot 3.x |
| ORM | JPA / Hibernate |
| Database | PostgreSQL 16 (RDS on AWS) |
| Migration | Flyway |
| Auth | Spring Security + JWT (jjwt) |
| Docs | SpringDoc OpenAPI / Swagger UI |
| Testing | JUnit 5, Mockito, Testcontainers |
| Build | Maven (multi-module) |
| Container | Docker, Docker Compose |
| Cloud | AWS EC2 (t3.micro) + RDS (db.t3.micro) + ECR |
| CI | GitHub Actions (build + test on push) |

---

## Stages

### Stage 1 — Core API (submittable on its own)
- All entities, repositories, and service layer implemented
- Full CRUD endpoints for products, batches, departments, suppliers
- Stock movement endpoint with FEFO logic and transactional safety
- Flyway migrations for schema management
- Seed data on startup (realistic medical supply dataset)
- Swagger UI available at `/swagger-ui.html`
- Dockerfile for the Spring Boot app
- Runs locally via `docker compose up --build`
- Unit tests for all service methods (JUnit 5 + Mockito), written alongside the code
- Integration tests for all endpoints (Testcontainers + PostgreSQL), written alongside the code

### Stage 2 — Quality layer
- JWT auth with ADMIN and VIEWER roles
- Input validation with Bean Validation (`@Valid`) on all DTOs
- Global exception handler (`@ControllerAdvice`) with consistent error response format
- Scheduled expiry and low-stock alert jobs
- Auto-generated purchase orders on low stock
- GitHub Actions CI pipeline

### Stage 3 — AWS deployment
- Docker image pushed to AWS ECR via GitHub Actions
- App deployed to EC2 (t3.micro), PostgreSQL on RDS (db.t3.micro)
- Environment variables via AWS Parameter Store (no secrets in repo)
- HTTPS via a self-signed cert or Let's Encrypt (optional)
- README includes live URL, recruiter login credentials, and architecture diagram

---

## README structure

1. Project overview and motivation
2. Architecture diagram (modules + AWS)
3. How to run locally (`docker-compose up`)
4. API reference (link to Swagger UI)
5. Design decisions (FEFO, multi-module structure, Flyway over Liquibase)
6. AWS deployment notes
7. Recruiter access instructions

---

## What this demonstrates to employers

- Multi-module Maven project structure (enterprise-grade)
- Non-trivial JPA entity relationships and business logic
- Spring Security with JWT
- Scheduled background jobs
- Testcontainers integration tests
- Docker + AWS deployment
- Clean REST API design with OpenAPI documentation
- Built collaboratively with Claude Code (demonstrates AI tooling fluency)
