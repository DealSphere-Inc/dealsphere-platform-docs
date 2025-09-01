# Technical Tasks for Core Framework & Auth Epic

## 1. Infrastructure & Repository Setup
- Create modular repository with `/backend`, `/frontend`, `/database`, `/docker` directories.
- Write Dockerfiles for backend, frontend, and database services.
- Implement a root Docker Compose file to orchestrate all services.
- Set up environment configuration management using `.env` files and CI/CD secret stores.
- Create initial README and repository documentation.

## 2. CI/CD Pipeline
- Configure GitHub Actions workflow for:
    - Linting and static code checks (backend and frontend)
    - Automated test execution (unit/integration/security)
    - Code coverage reporting
    - Build artifact publication and environment deployment (development & staging)
    - Security scans and linting as pre-merge gates

## 3. Backend Core (Spring Boot Services)
- Scaffold base Spring Boot microservice architecture.
- Integrate PostgreSQL and set up Flyway or Liquibase DB migration scripts.
- Implement core exception handling and logging framework.
- Create base entities and repositories (User, Role, Permission).
- Integrate environment variable/configuration management for secrets.

## 4. Authentication System
- Implement API endpoints for user registration, email verification, and login/logout.
- Integrate JWT-based access and refresh token generation/validation.
- Implement password hashing using bcrypt and password reset mechanism (email/token-based).
- Session management: enforce token expiry, implement secure refresh endpoint.
- Implement account lockout policy based on failed login attempts.
- Configure CORS and security headers at service entrypoint.

## 5. Authorization & RBAC
- Design and create database schema for roles, permissions, and assignments.
- Implement role management API (add, modify, delete roles; assign permissions).
- Enforce RBAC via middleware/interceptors & Spring Security annotations on endpoints.
- Ensure JWT claims correctly encode all roles/permissions of the user.
- Integrate role/permission refresh in real time for affected users.
- Develop initial role and permission matrix (admin, user, ops, etc.).

## 6. Frontend Foundation
- Scaffold React (TypeScript) app structure with routing and state management.
- Implement registration, login, logout, and password reset UI flows.
- Enable JWT storage and session management in the frontend (in-memory/token rotation).
- Implement RBAC-driven component rendering (hide/show UI elements by permission).
- Integrate form validation and error handling UX patterns.

## 7. Security Hardening
- Integrate security headers (CSP/Frame Options/HSTS, etc.) at NGINX or API gateway.
- Implement rate limiting logic for backend authentication endpoints (using Redis or similar).
- Validate and sanitize all input on backend APIs.
- Ensure generic error messages get returned for all API failures.

## 8. Automated Testing
- Write backend unit tests for all core modules (≥80% coverage).
- Implement integration tests for end-to-end registration/authentication flows.
- Write frontend unit and integration tests for core UI/auth flows.
- Automate security scanning (OWASP ZAP, Snyk) in CI/CD pipeline.
- Penetration and abuse test coverage for all sensitive flows and endpoints.

## 9. Deployment & Environment Management
- Set up and validate local/dev environments for the team.
- Automate deployment and environment validation for staging.
- Create and manage secrets for dev/staging environments.
- Automate DB migrations in deploy pipeline.

## 10. Documentation
- Auto-generate API documentation (Swagger/OpenAPI) and publish to `/docs`.
- Document architecture, setup steps, and environment configurations.
- Maintain permission/role matrix in project documentation.
