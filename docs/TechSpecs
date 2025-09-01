# Technical Specification Document
## Epic: Core Framework & Auth

---

### 1. Architecture Overview

- **Backend:** Java / Spring Boot microservices
- **Frontend:** React with TypeScript
- **Database:** PostgreSQL  
- **Authentication:** JWT token-based authentication and session management
- **Infrastructure:** Docker & Docker Compose for orchestration
- **CI/CD:** GitHub Actions for builds, deployment, and security enforcement

---

### 2. Infrastructure & Repository

- **Repository Structure:**
    - `/backend`: Spring Boot services, setup for modular architecture.
    - `/frontend`: React codebase with hooks and state management using Redux or Context API.
    - `/database`: SQL migration scripts handled by Flyway or Liquibase.
    - `/docker`: Service-specific Dockerfiles, one Docker Compose root.
- **Deployment:**  
    - All core services communicated via Docker Compose networks.
    - Standard GitHub Actions used for building, linting, and deploying to development/staging.
- **Environment Variables:**
    - Secure by using `.env` (for local) and CI/CD secret store (for deployment).
- **Build Artifacts Validation:**
    - Each build generates reports for code coverage, test results, and security scan logs.

---

### 3. Authentication System

- **User Registration:**
    - Endpoint for signup; requires unique username/email and password. Triggers email validation workflow (SMTP integration).
- **Login & Logout:**
    - Login endpoint issues secure JWT access & refresh tokens.
    - Logout endpoint invalidates session and rotates tokens.
- **Session Management:**
    - JWTs have expiry (configurable, default 15 minutes for access, 7 days for refresh).
    - Server validates tokens on every secured request; issues new access token on refresh.
- **Password Handling:**
    - Passwords hashed with bcrypt.
    - Password reset via secure one-time token sent to validated email.
- **Account Lockout:**
    - Configurable maximum failed attempts (default: 5), with exponential back-off lockout policy.
- **Security Protections:**
    - CSRF not required for APIs, but CORS configuration enforced.
    - Security headers (CSP, X-Frame-Options, HSTS, etc.) set at load balancer/REST layer.
    - Rate limiting with Redis (or similar) for sensitive endpoints.

---

### 4. Authorization & RBAC

- **Roles & Permissions:**
    - User roles predefined (e.g., `admin`, `user`, `ops`), with expandable role sets.
    - Permissions stored in DB and served as JWT claims.
    - Role assignments are mutable by authorized admins in real-time; changes take effect immediately.
- **API and UI Controls:**
    - Backend checks required roles/permissions for every endpoint using middleware/interceptor or Spring Security annotations.
    - Frontend does permission-based rendering by inspecting JWT claims.
- **Role Hierarchy:**
    - Support for parent/child role inheritance; granular, can stack permissions using config.

---

### 5. Security & Testing

- **Input Validation:**
    - All user input sanitized using server-side middleware and frontend form validation.
- **Rate Limiting & Abuse Protection:**
    - Generic rate limiter for all endpoints, with stricter thresholds on auth-sensitive routes.
- **Error Handling:**
    - All error messages log detailed traces server-side but always return generic, non-sensitive info to users.
- **Automated Tests:**
    - **Unit:** JUnit, Jest (backend, frontend)
    - **Integration:** RESTful API flows, DB migrations
    - **Security:** Penetration tests on all endpoints, automated by tools such as OWASP ZAP or Snyk
    - **Code Quality:** Pre-merge linting, code style checks, minimum 80% code coverage enforced via CI

---

### 6. Environments & Deployment

- **Development:**  
    - Local/dev setup with sample secrets, hot-reload, frequent deploys.
- **Staging:**  
    - Mimics production, using managed secrets, full external email, prod-like DB and access rules.
- **Database Migration:**  
    - Handled with consistent versioning scripts and CI-integrated checks to prevent drift.

---

### 7. Acceptance Criteria Mapping

| Business Requirement             | Technical Implementation                    |
|----------------------------------|---------------------------------------------|
| Repo deploys via Docker Compose  | Multi-container Docker Compose file         |
| CI/CD auto-runs tests            | GitHub Actions with integrated checks       |
| Secure Auth w/ JWT/Email/Login   | Spring Security config, JWT tokens setup    |
| Password reset, lockout          | Secure endpoints, expiring tokens           |
| RBAC enforced on all APIs/UIs    | Spring annotations, React guards            |
| Security headers, rate limits    | Global filters, NGINX/LB config             |
| All environments validated       | CI/CD deploys with test verification        |
| Test coverage >=80%              | Test suites + pre-merge gates               |
| No sensitive error leakage       | Exception mapping & logging discipline      |

---

### 8. Risks & Mitigations

- **JWT Blacklist Unavailability:**  
  Solution: Use short-lived access tokens; rotate refresh tokens and DB blacklist for critical operations.
- **Role Drift:**  
  Solution: All changes to roles/permissions are tracked and auditable.
- **Misconfigurations in Environment Variables:**  
  Solution: Template sample `.env`, lock down overrides, validate on CI deploys.
- **Security Regression:**  
  Solution: Run automated OWASP/Snyk scans on every CI/CD pipeline.

---

### 9. Documentation

- Auto-generate API documentation via Swagger/OpenAPI.
- README files for all setup (local, dev, staging).
- Architecture & permission matrix docs stored in `/docs`.

---
