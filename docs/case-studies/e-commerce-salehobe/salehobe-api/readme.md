# Salehobe-api
<a name="readme-top"></a>

<div>

# 📗 Table of Contents

- [📖 About the Project](#about-project)
  - [🛠 Built With](#built-with)
    - [Key Features](#key-features)
    - [Project Evaluation](#project-evaluation)
    - [Architecture Overview](#architecture-overview)
- [💻 Getting Started](#getting-started)
  - [Setup](#setup)
  - [Branching-Strategy](#branching-strategy)
- [🔌 API-endpoints](#api-endpoints)
- .[🧭 Controller Structure](#controller-structure)
- .[🔐 Authentication & Authorization](#authentication-authorization)
- .[🧪 Testing Strategy](#testing-strategy)
- [🧱 Pitfalls:Solved](#pitfalls)
- [🔭 Future Features](#future-features)
- [👥 Authors](#authors)
- [📝 License](#license)
- [🙏 Acknowledgements](#acknowledgements)

## 📖 About the Project <a name="about-project"></a>

SALEHOBE-API is a modular, versioned Rails 8 backend built with PostgreSQL. It follows Technical Agile and Evolutionary Architecture principles, emphasizing planned iteration, clean controller inheritance, and schema-first database design.

## 🛠 Built With <a name="built-with"></a>

-  Ruby on Rails 8.1 — API-only mode

-  PostgreSQL — custom schema: salehobe

-  RSpec — request-spec-first testing strategy

-  WT-ready architecture — future-proofed for multi-client auth

## Key Features <a name="key-features"></a>

-  Modular controller inheritance: Application → Api → Version → Resource

-  Custom schema-first DB setup: salehobe.*

-  Auth v1: session-based with has_secure_password

-  Auth v2 (planned): JWT with scopes and expiration

-  Lightweight documentation: one page per frozen decision

## Project Evaluation <a name="project-evaluation"></a>

-  Principle, Practice,Branching
-  One branch = one architectural responsibility
-  Documentation
-  Written only when decisions are frozen (e.g., auth, DB v1, API contract)
-  Schema Setup
-  Schema created before tables; verified via \dt salehobe.*
-  Controller Structure
    Three-layer API inheritance; no premature abstraction

-  Testing
    Request specs first; failing specs allowed in feature branches

## Architecture Overview <a name="architecture-overview"></a>

- Controllers: Three-layer API structure with clean inheritance

- Schemas: Custom schema creation order enforced to avoid Rails pitfalls

- Auth: Roles treated as contracts, not labels

## 💻 Getting Started <a name="getting-started"></a>

### Setup <a name="setup"></a>

-  Ensure PostgreSQL is running and supports pg_catalog.plpgsql.

-  Create the schema before running any table migrations:

-  create_schema "salehobe"

-  Set schema_search_path in database.yml:

-  schema_search_path: salehobe,public

-  Run migrations and verify with:
```
\dt salehobe.*
```

## Branching Strategy <a name="branching-strategy"></a>

-  auth-v1: session-based login

-  auth-rspec: request specs for login/token flow

-  schema-init: migration to create salehobe schema

-  products-api: product and variant endpoints

-  Documentation Guideline <a name="documentation-guideline"></a>

-  Write docs only when decisions are frozen

-  One page per topic

-  Update docs in the same PR as the change

## 🔌 API Endpoints (v1)<a name="api-endpoints"></a>

{Base URL}/api/v1

Examples:
```
Endpoint	                      Method

/admins/login                   POST
/products	                      GET/POST
/products/filter_by_category	  GET
/orders	                        GET/POST
/orders/:id	                    GET
```

<!-- 👉 Full payload examples are documented in docs/api.md -->

## 🧱 Pitfalls Solved <a name="pitfalls"></a>

Rails + PostgreSQL Schema Creation Order

Tables default to public if schema is created late

Fix:
- raname to keep schema migration first in the sequence

- Verify with \dt salehobe.*

- schema_search_path affects new tables only

## 🧭 Controller Structure <a name="controller-structure"></a>

app/controllers/
├── application_controller.rb
├── api/
│   ├── base_controller.rb
│   └── v1/
│       ├── base_controller.rb
│       ├── admins_controller.rb
│       ├── products_controller.rb

ApplicationController: Rails plumbing only

Api::BaseController: API-wide behavior

Api::V1::BaseController: version contract (auth, pagination, etc.)

Resource controllers inherit cleanly

## 🔐 Authentication & Authorization <a name="authentication-authorization"></a>

### v1 (current)

- has_secure_password

- Session or signed cookie

- Single trusted client (admin panel)

- Minimal attack surface

### v2 (planned)

- JWT with scopes and expiration

- Multi-client support (web, mobile, 3rd-party)

- No schema changes required

 ***See docs/auth for authentication details.***

## 🧪 Testing Strategy <a name="testing-strategy"></a>

- RSpec in development and test for generator support

- Request spec first for client-visible behavior

- Failing specs allowed in feature branches

## Commit strategy for testing:
  ```
| Spec state  | Commit allowed? | Branch                  |
| ----------- | --------------- | ----------------------- |
| **Pending** | ❌ No           | Any                     |
| **Failing** | ✅ *Yes*        | **Feature branch only** |
| **Passing** | ✅              | `main` / `develop`      |
```

## 🔭 Future Features <a name="future-features"></a>

- Stripe and  mobile banking (Bkash, Nagad) payment

- JWT-based multi-client authentication

- Role-based authorization contracts

- Rate limiting and feature flags in Api::V1::BaseController

- HTML controllers under web/ namespace


## 👥 Authors:<a name="authors"></a>

**Md Arafat Hossain**

- GitHub: <a href="https://github.com/HossainAraf">HossainAraf </a>

- LinkedIn: <a href="https://www.linkedin.com/in/hossain-arafat-engineer"/> Md. Arafat Hossain </a>

## 📄 License<a name="license"></a>
MIT License — /LICENSE.md

## 🙏 Acknowledgments<a name="acknowledgements"></a>

- Family support
- Microverse
    — structure, standards, and discipline
</div>
