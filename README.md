# Multi-Tenant SaaS Point of Sale (POS) System

A full-stack retail management and point-of-sale application built with **Spring Boot, Java, React, and MySQL**. The project is organized around store and branch operations, with role-oriented interfaces for platform administrators, store teams, branch teams, and cashiers.



## Contents

- [Overview](#overview)
- [Features represented in the codebase](#features-represented-in-the-codebase)
- [Technology stack](#technology-stack)
- [Architecture](#architecture)
- [User roles](#user-roles)
- [Backend API overview](#backend-api-overview)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [Configuration](#configuration)
- [Running the application](#running-the-application)
- [Testing](#testing)
- [Security and deployment notes](#security-and-deployment-notes)
- [Potential future improvements](#potential-future-improvements)

## Overview

Retail businesses often need a connected way to manage products, inventory, sales, employees, branches, customers, and cashier shifts. This project brings those workflows into a single POS platform.

The backend exposes REST APIs for the business domains, while the React frontend presents different application areas based on the signed-in user's role. Store and branch relationships are represented in the JPA domain model, and the application includes subscription and payment-related modules.

## Features represented in the codebase

The following capabilities are represented by backend controllers/services, domain entities, or frontend routes in the supplied source:

- **Authentication and account flows:** signup, login, forgot-password and reset-password endpoints; JWT-related security components.
- **Store and branch administration:** store records, branch management, employee management, and store moderation endpoints.
- **Product catalogue:** product and category management, store product listing, and product search.
- **Inventory:** branch inventory records and stock update/retrieval endpoints.
- **Point-of-sale orders:** order creation, order retrieval, branch/cashier/customer order views, and recent/today order endpoints.
- **Customer management:** customer create, update, retrieve, list, and delete endpoints.
- **Cashier operations:** order creation, order history, returns, customer lookup, and shift summary frontend routes.
- **Refunds and shifts:** refund endpoints and shift start/end, current shift, and shift report endpoints.
- **Analytics:** branch and store analytics endpoints for sales, products, cashiers, payment breakdowns, and alerts.
- **Subscriptions and payments:** subscription plans, subscription lifecycle endpoints, and Razorpay/Stripe dependencies.
- **Role-oriented frontend areas:** super-admin, store, branch-manager, cashier, and authentication routes.

Availability and completeness of individual workflows should be confirmed by running and testing the application. A route or dependency alone does not prove that a complete external integration is configured.

## Technology stack

| Area | Technologies found in the source |
|---|---|
| Backend | Java 17, Spring Boot 3.5.3, Spring Web |
| Persistence | Spring Data JPA, Hibernate, MySQL Connector/J |
| Security | Spring Security, JWT (JJWT 0.12.6), BCrypt password encoder usage in security configuration |
| Validation | Spring Boot Validation |
| Email | Spring Boot Starter Mail |
| Payment libraries | Razorpay Java SDK 1.4.8, Stripe Java SDK 28.3.1 |
| Frontend | React 19, Vite 7, JavaScript (ES modules) |
| Client state and HTTP | Redux Toolkit, React Redux, Axios |
| Routing | React Router 7 |
| UI and styling | Tailwind CSS 4, Radix UI components, Lucide React |
| Forms and validation | React Hook Form, Formik, Yup, Zod, `@hookform/resolvers` |
| Charts | Recharts |
| Build tools | Maven, Vite, npm-compatible scripts |

## Architecture

The backend follows a layered organization:

```text
React frontend
     |
     | HTTP / REST
     v
Spring Boot controllers
     |
     v
Service layer (business operations)
     |
     v
Spring Data JPA repositories
     |
     v
MySQL database
```

Cross-cutting backend packages include configuration/security, domain enums, DTO payloads, entity-to-DTO mappers, utilities, and exception handling.

### Main domain areas

The JPA model includes entities for:

- **Organization and access:** `Store`, `Branch`, `User`, `PasswordResetToken`
- **Catalogue and stock:** `Product`, `Category`, `Inventory`
- **Sales and customers:** `Order`, `OrderItem`, `Customer`
- **Cashier operations:** `ShiftReport`, `Refund`, `PaymentSummary`
- **Subscriptions and payments:** `SubscriptionPlan`, `Subscription`, `PaymentOrder`

The model uses store/branch relationships to represent organizational ownership. Verify authorization and tenant isolation for every data-access path before deploying with multiple independent businesses.

## User roles

The `UserRole` enum defines these roles:

| Role | Intended application area |
|---|---|
| `ROLE_ADMIN` | Platform / super-admin operations |
| `ROLE_STORE_ADMIN` | Store administration |
| `ROLE_STORE_MANAGER` | Store management |
| `ROLE_BRANCH_MANAGER` | Branch management |
| `ROLE_BRANCH_ADMIN` | Branch administration |
| `ROLE_BRANCH_CASHIER` | Cashier / POS operations |
| `ROLE_CUSTOMER` | Customer role represented in the domain model |

The frontend routes users to role-specific application areas based on the role returned in the user profile. The exact permissions for each API must be verified in the backend security and service code.

## Backend API overview

The following table summarizes controller base paths and representative operations found in the backend source. Most paths below are relative to the listed base path.

| Area | Base path | Operations represented in the controller |
|---|---|---|
| Authentication | `/auth` | `POST /signup`, `/login`, `/forgot-password`, `/reset-password` |
| Users | `/api/users` and `/users` | Profile and role-specific user lookups; user listing and updates |
| Stores | `/api/stores` | Create, retrieve, update, delete, employee operations, moderation |
| Branches | `/api/branches` | Create, retrieve, list by store, update, delete |
| Employees | `/api/employees` | Add by store/branch, update, delete, retrieve, list |
| Products | `/api/products` | Create, retrieve, update, delete, list/search by store |
| Categories | `/api/categories` | Create, list by store, update, delete |
| Inventory | `/api/inventories` | Create, update, delete, retrieve by ID/product/branch |
| Orders | `/api/orders` | Create, retrieve, list by branch/cashier/customer, today/recent orders, delete |
| Customers | `/api/customers` | Create, update, delete, retrieve, list |
| Refunds | `/api/refunds` | Create, list/filter, retrieve, delete |
| Shift reports | `/api/shift-reports` | Start/end shift, current shift, retrieve/list, delete |
| Branch analytics | `/api/branch-analytics` | Daily sales, top products/cashiers, category sales, overview, payment breakdown |
| Store analytics | `/api/store/analytics` | Overview, sales trends, monthly/daily/category/payment/branch reports, alerts |
| Admin dashboard | `/api/super-admin` | Dashboard summary, store registrations, store status distribution |
| Subscriptions | `/api/subscriptions` | Subscribe, upgrade, activate, cancel, payment status, admin/store queries |
| Subscription plans | `/api/super-admin/subscription-plans` | Create, update, list, retrieve, delete plans |
| Payments | `/api/payments` | Create payment order/link and proceed with payment |

For request and response schemas, inspect the corresponding classes under `pos-backend/src/main/java/com/zosh/payload/` and the controller methods. No generated OpenAPI/Swagger specification was found in the supplied project files.

## Project structure

```text
.
├── pos-backend/
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/zosh/
│       │   │   ├── configrations/  # Application, security and JWT configuration
│       │   │   ├── controller/    # REST controllers
│       │   │   ├── domain/        # Enums and domain constants
│       │   │   ├── exception/     # Custom exceptions and global handling
│       │   │   ├── mapper/        # Entity / DTO mapping
│       │   │   ├── modal/         # JPA entities (package name in source)
│       │   │   ├── payload/       # Request and response DTOs
│       │   │   ├── repository/    # Spring Data repositories
│       │   │   ├── service/       # Business services
│       │   │   └── util/          # Utility classes
│       │   └── resources/
│       │       └── application.yml
│       └── test/
└── pos-frontend-vite/
    ├── package.json
    ├── vite.config.js
    ├── public/
    └── src/
        ├── components/
        ├── context/
        ├── hooks/
        ├── pages/
        ├── Redux Toolkit/
        ├── routes/
        └── utils/
```

## Getting started

### Prerequisites

Install the following before starting:

- Java 17
- Maven 3.6.3+ (or use the Maven wrapper included in `pos-backend`)
- Node.js and npm (use a current Node.js release compatible with Vite 7)
- MySQL Server
- Git

### 1. Clone the repository

```bash
git clone https://github.com/vinnymishra/Multi-Tenant-SaaS-Application.git
cd Multi-Tenant-SaaS-Application
```

The commands assume the repository root contains the `pos-backend` and `pos-frontend-vite` directories.

### 2. Create the MySQL database

Create a database for the application. The supplied configuration uses `pos` as the default database name:

```sql
CREATE DATABASE pos;
```

Use a dedicated database user for local development rather than a privileged production account.

### 3. Configure the backend

Open `pos-backend/src/main/resources/application.yml` and configure the database, mail, payment gateway, and frontend reset URL settings for your environment. Do not commit credentials or API secrets.

The datasource currently reads these environment variables (with defaults in the YAML):

- `DB_HOST`
- `DB_PORT`
- `DB_NAME`
- `DB_USERNAME`
- `DB_PASSWORD`

The supplied YAML also contains mail and payment configuration placeholders/values that must be replaced with valid local configuration if those features are to be exercised. The current Razorpay, Stripe, and mail settings are not fully environment-variable-driven in the supplied file; externalize them before sharing or deploying the project.

### 4. Start the backend

From the repository root:

```bash
cd pos-backend
```

On Windows:

```powershell
.\mvnw.cmd spring-boot:run
```

On macOS/Linux:

```bash
./mvnw spring-boot:run
```

Alternatively, if Maven is installed:

```bash
mvn spring-boot:run
```

The supplied configuration sets the backend port to **5000**.

### 5. Install and start the frontend

Open another terminal from the repository root:

```bash
cd pos-frontend-vite
npm install
npm run dev
```

Vite prints the local development URL in the terminal (commonly `http://localhost:5173`). Use the URL printed by your local Vite process.

### 6. Build the frontend

```bash
cd pos-frontend-vite
npm run build
```

To run the configured lint command:

```bash
npm run lint
```

## Configuration

Before running the application, review `pos-backend/src/main/resources/application.yml` and set environment-specific values for:

| Setting | Purpose |
|---|---|
| `DB_HOST`, `DB_PORT` | MySQL host and port |
| `DB_NAME` | Database/schema name |
| `DB_USERNAME`, `DB_PASSWORD` | Database credentials |
| Mail SMTP username and app password | Password-reset email delivery |
| Razorpay API key and secret | Razorpay payment operations |
| Stripe API key | Stripe payment operations |
| Frontend reset URL | Link used in password-reset flow |

Do not paste real credentials into README files, screenshots, issue reports, or commits. Prefer environment variables or a secrets manager, and commit only a sanitized example configuration.

## Running the application

Start MySQL first, then the Spring Boot backend, and finally the Vite frontend. The frontend uses the backend REST API; confirm that the API base URL and CORS settings match your local setup before testing flows.

The frontend route groups in the source include:

- `/auth/*` — login, onboarding, password reset, and a placeholder forgot-password page
- `/super-admin/*` — platform dashboard, stores, requests, subscription plans, settings
- `/store/*` — store dashboard, branches, categories, employees, products, sales, reports, upgrades, settings, alerts
- `/branch/*` — branch dashboard, orders, refunds, transactions, inventory, employees, customers, reports, settings
- `/cashier/*` — order creation/history, returns, customers, shift summary

Some frontend routes are role-gated by the application routing logic. A route being present does not by itself establish that all related backend authorization checks are complete.

## Testing

The supplied backend archive contains a Spring Boot test source file (`PosSystemApplicationTests.java`). The source archive does not include evidence of a comprehensive unit, integration, end-to-end, or security test suite, and no test execution results are provided here.

Run the backend tests with:

```bash
cd pos-backend
./mvnw test
```

On Windows, use `mvnw.cmd test`. Run frontend lint/build checks with `npm run lint` and `npm run build` from `pos-frontend-vite`.

## Security and deployment notes

This project includes authentication and payment-related code, but the supplied source should be treated as a development codebase until it has been reviewed and hardened.

Before publishing or deploying:

1. **Rotate any credential or signing secret that has ever been committed.** Removing a value from the latest file does not remove it from Git history.
2. Remove hardcoded credentials and secrets from YAML, Java classes, frontend files, and Git history. Use environment variables or a secrets manager.
3. Review JWT key strength, token expiry, validation, revocation strategy, and secure storage on the client.
4. Verify tenant ownership and role authorization on every read, write, update, delete, analytics, and payment-related endpoint. Never trust a `storeId`, `branchId`, or user ID supplied by the client without checking access on the server.
5. Review payment gateway signature verification, webhook handling, idempotency, and payment state transitions before accepting real payments.
6. Use HTTPS, restrictive CORS rules, secure cookie/storage practices, and production-safe error/logging settings.
7. Reconsider `spring.jpa.hibernate.ddl-auto: update` for production; use reviewed database migrations.
8. Disable verbose SQL logging and development tools in production.
9. Add automated tests for cross-tenant access, role boundaries, order/stock consistency, refunds, and payment flows.

## Potential future improvements

These are recommendations, not claims about current implementation:

- Add OpenAPI/Swagger documentation with example requests and responses.
- Add integration tests for tenant isolation and role-based access control.
- Add database migrations with Flyway or Liquibase.
- Add Docker Compose for reproducible local startup of MySQL and the application services.
- Add CI checks for backend tests, frontend linting, and production builds.
- Add deployment instructions and environment-specific configuration examples.
- Add sanitized screenshots or a short demo GIF after capturing the running application.
- Add a documented API error format and pagination/filtering conventions.

## Contributing

For changes, create a focused branch, describe the problem being solved, and include relevant tests or manual verification steps in the pull request. Do not commit secrets, local environment files, build outputs, or dependency directories.

## License

No license file or explicit open-source license was identified in the supplied archive. Add a `LICENSE` file if you intend to grant others permission to use, modify, and distribute this project.
