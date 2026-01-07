# Overview

This project is a deployment setup template for Anonym Chat, not the application itself. It is meant to be modified and provides a ready-to-use Docker Compose configuration with sensible defaults for deploying the complete chat solution on-premise.
It can be deployed in multiple configurations. It supports both local containerized and external services, with comprehensive SSL/TLS support and multiple authentication modes.

## Key Features

- **Flexible Deployment**: Use local Docker services or connect to external infrastructure
- **Dual Authentication**: Database-based or LDAP/Active Directory authentication
- **SSL/TLS Support**: Full encryption for all services with self-signed or production certificates
- **Configuration-Driven**: All settings managed through `.env` file
- **Scalable Architecture**: Connection pooling with PgBouncer, configurable worker processes

## Deployment Modes

**Local Services**
- PostgreSQL database container
- MinIO S3-compatible storage
- All services within Docker network

**External Services**
- Connect to external PostgreSQL instance
- Use external S3-compatible storage
- Hybrid configurations supported

## Authentication Modes

**Database Authentication**
- Traditional email/password registration
- User management through application
- Superuser account created on first deployment
- Recommended: Change superuser password after first login

**LDAP/Active Directory**
- Enterprise SSO integration
- Recommended for Microsoft Active Directory
- Disables registration, uses directory credentials
- Service account connects to LDAP server to query user data
- User authentication validated against directory
- Superuser matched by email in directory
- Superuser can grant admin privileges to other users via `/admin` endpoint

