# Architecture

MedicBD follows a hybrid Rails structure: it keeps a public web layer and a secure API layer while sharing the same domain model and database.

## High-level structure

```text
Rails app
├── Web layer
│   └── public pages and browsing experience
├── API v1 layer
│   ├── sessions
│   ├── doctors
│   ├── chambers
│   ├── districts
│   ├── specializations
│   └── doctor schedules
├── Domain models
│   ├── Doctor
│   ├── Chamber
│   ├── District
│   ├── Specialization
│   ├── DoctorSchedule
│   └── MedicUser
└── PostgreSQL database
```

## API and web separation

The application intentionally separates concerns:

- API controllers handle JSON responses and JWT authentication
- web controllers serve HTML pages and browser-based interactions
- shared models remain reusable across both layers

This keeps the backend flexible while allowing the web layer to evolve independently.

## Main responsibilities

### API layer

- user login and authentication
- doctor search and retrieval
- specialization lookup
- chamber and district data access
- schedule management for admin workflows

### Web layer

- public browsing pages
- list-based pages for specialties and related content
- server-rendered views using Rails conventions

### Data layer

- PostgreSQL stores structured healthcare and location data
- model associations define the relationships between doctors, chambers, disticts, and schedules

## Security boundaries

- JWT authentication is enforced for protected API routes
- web and API authorization concerns are intentionally separated
- secret material should stay in environment variables or encrypted credentials, not in public documentation

## Why this architecture works

It gives the project a safe migration path from backend-first delivery toward a richer full-stack experience without destabilizing the API contract.
