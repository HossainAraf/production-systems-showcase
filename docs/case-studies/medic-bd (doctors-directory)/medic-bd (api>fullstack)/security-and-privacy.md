# Security and privacy

This project is designed with basic production-aware security practices and a public-safe documentation standard.

## Security practices in the app

- API routes enforce authentication for protected actions
- JWT-based authorization is used for request validation
- web and API concerns remain separated to reduce accidental auth misuse
- request data is expected to be validated before persistence
- environment-sensitive values should not be checked into public repositories

## Public documentation rules

The public-facing docs intentionally avoid:

- personal names or contact details
- deployment URLs or production hostnames
- credential values, tokens, or secrets
- internal email addresses or account identifiers
- environment-specific configuration values that should stay private

## Recommended handling for production

- store secrets in environment variables or encrypted credentials
- keep production DB credentials out of version control
- restrict CORS policies to trusted client origins
- rotate JWT secrets and database passwords regularly
- review application logs to ensure no sensitive data is leaked

## Privacy principle

The project documentation is written for showcase and portfolio use. It communicates technical value without exposing private operational data.
