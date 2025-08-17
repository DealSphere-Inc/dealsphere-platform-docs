# DealSphere Phase 1 – Technical Landscape Document

## 1. Introduction

This document defines the technology landscape for DealSphere Phase 1 MVP, mapping core product requirements to recommended technologies for each platform layer. Updates reflect the team’s decision to use **HTTPS with Protocol Buffers (protobuf)** for service-to-service communication, not gRPC.

---

## 2. Technical Landscape Overview

| Domain/Layer                | Primary Technology Choices                                                         | Alternatives/Notes                      |
|-----------------------------|------------------------------------------------------------------------------------|-----------------------------------------|
| **Distributed Ledger (DLT)**| R3 Corda, Kotlin (contract logic)                                                  | Hyperledger Fabric, Quorum              |
| **Backend / API Layer**     | Spring Boot (Java/Kotlin), Corda SDK, REST, **HTTPS + Protobuf (service-to-service)** | Micronaut, Node.js (non-ledger APIs)    |
| **Frontend / Client Layer** | ReactJS, Redux, Material-UI                                                        | Angular, Ant Design                     |
| **Security & Identity**     | AES-256/TLS, OAuth2.0, OpenID Connect, AWS KMS/Azure Key Vault, Corda ACL          | Okta, Auth0, HashiCorp Vault            |
| **Document Management**     | Corda ledger (hash/meta), Encrypted AWS S3, Apache Tika/ElasticSearch              | Google Cloud/Azure Blob, MinIO          |
| **Payments Integration**    | Corda smart contracts (logic), Stripe/Plaid APIs (reminders/fiat movement)         | Direct bank APIs, PayPal                |
| **Analytics & Reporting**   | Apache Kafka, PostgreSQL/Redshift/BigQuery, Apache Superset, Metabase              | Snowflake, Tableau                      |
| **AI/ML & OCR**             | OpenAI API, Tesseract, scikit-learn, ElasticSearch ML plugins                      | AWS Textract, Google Vision             |
| **Fund Accounting**         | Custom Java/Kotlin logic, stored and verified via Corda                            | Python/NumPy for prototyping            |
| **Infra & DevOps**          | Docker, Kubernetes, AWS/Azure/GCP, GitHub Actions, Prometheus/Grafana, ELK Stack  | Jenkins, GitLab CI, DataDog             |
| **Testing & QA**            | JUnit/TestNG, Selenium/Cypress, Pact, Burp Suite                                   | Playwright, Karate                      |
| **Integration**             | REST APIs, **HTTPS + Protobuf (internal svc comm)**, Webhooks, WebSockets          | GraphQL for special cases               |

---

## 3. Component Technology Breakdown

### 3.1 Distributed Ledger (Blockchain)
- **R3 Corda** for on-ledger business logic, contract validation, audit, and participant privacy.
- Smart contracts: **Kotlin**.

### 3.2 Platform Security & Access Control
- **AES-256** at rest, **TLS 1.2+** in transit.
- **Key Management:** AWS KMS, Azure Key Vault.
- **Identity:** OAuth2.0, OpenID Connect, SSO options; Corda ACL for role-based access.

### 3.3 Backend/API Layer
- **Spring Boot** (Java/Kotlin) for business logic, REST API, Corda integration.
- **Service-to-Service Communication:**  
  - **HTTPS protocol** utilizing **Protocol Buffers (protobuf)** for efficient, contract-first serialization/deserialization between services.  
  - *No gRPC; protobuf messages over raw HTTPS.*
- **API Gateway:** Kong, AWS API Gateway (as needed).
- **Workflow Automation:** Camunda/Temporal.

### 3.4 Frontend/Client Layer
- **ReactJS** + **Redux** for modular UI; **Material-UI** for visuals and theming.
- Secure authentication with OAuth2/OpenID.

### 3.5 Document Management & Compliance
- **Off-chain Storage:** Encrypted S3/Blob, metadata/hashes in Corda.
- **Processing:** Apache Tika, Tesseract OCR, ElasticSearch.
- **Compliance:** Ledger-based audit trails.

### 3.6 Payments & Automated Capital Calls
- **Smart contracts** in Corda.
- **External payment APIs:** Stripe/Plaid.

### 3.7 Analytics & Reporting
- **Data streaming:** Apache Kafka (if needed).
- **Analytics DB:** PostgreSQL, Redshift, or BigQuery.
- **Dashboards:** Superset, Metabase, or custom.
- **Exports:** jsPDF, server PDF/Excel generator.

### 3.8 AI/ML Integration
- **OpenAI API** for language tasks.
- **OCR:** Tesseract, GCP Vision, or AWS Textract.
- **Search/Classification:** Elastic ML, custom models.

### 3.9 Fund Accounting
- **Custom JVM services** for ledger operations and compliance.
- **Prototyping:** Python/NumPy.

### 3.10 Infrastructure & DevOps
- **Cloud:** AWS, Azure, or GCP.
- **Containers:** Docker, Kubernetes.
- **CI/CD:** GitHub Actions.
- **Monitoring:** Prometheus, Grafana, ELK.

### 3.11 Testing & QA
- **Selenium**, **Cypress**, **JUnit/TestNG**, **Pact**, **Burp Suite**.

### 3.12 Integration
- **REST APIs** and **Webhooks** for third-party comms.
- **HTTPS + protobuf** for **internal microservice communication** (replace gRPC).
- **WebSockets** for real-time events.

---

## 4. Security, Compliance, and Governance Highlights

- Strong transport security and encryption.
- Immutable on-ledger audit trails.
- Role-based access enforced at multiple layers.
- Automated testing and vulnerability scanning.
- GDPR-ready and enterprise compliance support.

---

## 5. Extensibility & Future Readiness

- Pluggable AI/ML, multi-cloud deploy-ready.
- Versioned APIs and protobuf contracts.
- Incremental scaling via microservices and Kubernetes.

---

## 6. High-Level Architecture Diagram (Text)
```
+-----------------------------+         +---------------------------+
| Frontend/UI (React, Redux)  | <-----> | Backend API (Spring Boot) |
+-----------------------------+         +---------------------------+
                 |                                  |
                 |                                  |
                 v                                  v
+-----------------------------+         +---------------------------+
|   Blockchain (Corda Nodes)  |         |    Data/AI/Analytics      |
+-----------------------------+         +---------------------------+
                 |                                  |
                 +---------------+------------------+
                                 |
                                 v
    +-------------------+   +---------------------+   +---------------------------+
    | Encrypted Storage |   | Integrations        |   | Cloud Infra (Docker, K8s, |
    |       (S3)        |   | (REST, HTTPS+ PB)   |   | AWS/Azure/GCP, CI/CD, Sec)|
    +-------------------+   +---------------------+   +---------------------------+
```

*Note: All internal microservice communication uses HTTPS + protobuf for efficiency and future contract versioning.*

---

## 7. Conclusion

This landscape ensures the platform aligns with latest product and compliance requirements, including the switch to HTTPS + Protocol Buffers for service-to-service calls. It’s secure, adaptable, and ready for ongoing evolution in line with business goals.
