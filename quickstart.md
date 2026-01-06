# Anonym Chat Setup

Docker Compose template for deploying a secure chat application with flexible configuration options.

## Quick Navigation

- [Overview](#overview) - Project description and features
- [Quick Start](#quick-start) - Installation and deployment
- [Architecture](#architecture) - System design and requirements
- [Configuration](#configuration) - Environment variables reference
- [Tools](#tools) - Certificate and LDAP utilities

## Overview

This project is a deployment setup template for Anonym Chat, not the application itself. It is meant to be modified and provides a ready-to-use Docker Compose configuration with sensible defaults for deploying the complete chat solution on-premise.
It can be deployed in multiple configurations. It supports both local containerized and external services, with comprehensive SSL/TLS support and multiple authentication modes.

### Key Features

- **Flexible Deployment**: Use local Docker services or connect to external infrastructure
- **Dual Authentication**: Database-based or LDAP/Active Directory authentication
- **SSL/TLS Support**: Full encryption for all services with self-signed or production certificates
- **Configuration-Driven**: All settings managed through `.env` file
- **Scalable Architecture**: Connection pooling with PgBouncer, configurable worker processes

### Deployment Modes

**Local Services**
- PostgreSQL database container
- MinIO S3-compatible storage
- All services within Docker network

**External Services**
- Connect to external PostgreSQL instance
- Use external S3-compatible storage
- Hybrid configurations supported

### Authentication Modes

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

## Quick Start

### Installation Requirements

See [Architecture - Requirements](#requirements) for detailed system requirements.

### Installation

**1. Add SSH key to GitHub**

This is a private repository. Ensure your SSH key is added to your GitHub account before cloning.

**2. Clone repository**

```bash
git clone git@github.com:Giorgi-Sekhniashvili/anonym-chat-setup.git
```

**3. Navigate to project**

```bash
cd anonym-chat-setup
```

**4. Create environment file**

```bash
cp .env.example .env
```

**5. Configure environment**

Edit `.env` and set required configurations. See [Configuration](#configuration) for detailed options.

**6. Configure SSL certificates (Optional)**

You can skip this step if you are not using ssl.

**For Nginx:**

- **Production certificates**: Place your certificates in `./certs/nginx/` named as `${APP_DOMAIN}.crt` and `${APP_DOMAIN}.key`
- **Let's Encrypt (free)**: Ensure no certificate files for your domain exist in `./certs/nginx/` - certificates will be generated automatically
- **Self-signed certificates**: Generate using `tools/certs` (see [Tools](#tools)) and set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env` (see [Configuration - Local/Unsafe](#localunsafe))

**For Database SSL:**

1. Place client certificates in `./certs/backend/`
2. Place server certificates in `./certs/pgbouncer/`
3. Place server certificates in `./certs/postgres/`

**For LDAP SSL:**

Place CA certificate in `./certs/backend/${LDAP_CA_FILE}` where `${LDAP_CA_FILE}` matches your `.env` configuration.

**Generate all self-signed certificates:**

```bash
tools/certs all
```

See [Tools - Certificate Generator](#certificate-generator) for more options.

**7. Login to Docker registry (optional)**

Only required if using pre-built images from private registry.

```bash
docker login registry.anonym.giorgis.xyz
```

Enter the username and password provided to you.

**8. Start services**

```bash
docker login registry.anonym.giorgis.xyz
```

Enter the username and password provided to you.

**8. Start services**

```bash
docker compose up -d
```

**9. Post-deployment**

For database authentication mode, change the superuser password after first login for security.

### Update or Reconfigure

**1. Pull latest changes**

```bash
git pull
```

Or apply your local changes if working on a custom branch.

**2. Review patch notes**

Check patch notes provided with updates for breaking changes or migration steps.

**3. Stop affected services**

Stop all services:

```bash
docker compose down
```

Stop specific services:

```bash
docker compose down backend frontend
```

**4. Apply environment changes**

Edit `.env` if configuration updates are required.

**5. Run migrations**

If database schema changes are included:

```bash
docker compose up -d --build migrate
```

**6. Restart services**

```bash
docker compose up -d
```

### Common Issues

**Cached volumes after configuration changes**

Docker volumes persist data between restarts. After configuration changes, you may need to remove volumes:

```bash
docker compose down -v <service_name>
```

**Warning**: Removing ```postgres``` service with volumes will result removing the `postgres-data` volume and will delete all database data.

**Self-signed Nginx certificates**

When using self-signed certificates for Nginx, set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env`. See [Configuration - Local/Unsafe](#localunsafe).

**Let's Encrypt not activating**

Ensure no certificate files for your domain exist in `./certs/nginx/`. The presence of `${APP_DOMAIN}.crt` or `${APP_DOMAIN}.key` will prevent the ACME companion from generating certificates.

**LDAP connection issues**

Verify LDAP SSL configuration:
- Ensure `LDAP_CA_FILE` is present in `./certs/backend/` when using SSL
- Use the LDAP debugging tool: `tools/ldap username password`

See [Tools - LDAP Debugger](#ldap-debugger) for detailed troubleshooting.

## Architecture

### Overview

The application uses a microservices architecture with separate frontend and backend services, connection pooling layer, and support for both local and external data services.

```
┌─────────────────────────────────────────────────────────┐
│                     nginx-proxy                         │
│              (SSL termination, routing)                 │
└────────────────────┬────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
    ┌────▼────┐            ┌─────▼──────┐
    │ Frontend│            │  Backend   │
    │ (Next.js│            │ (FastAPI)  │
    └─────────┘            └──────┬─────┘
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
              ┌─────▼────┐  ┌─────▼────┐  ┌────▼─────┐
              │ PgBouncer│  │  MinIO   │  │   LDAP   │
              │  (Pool)  │  │   (S3)   │  │(External)│
              └─────┬────┘  └──────────┘  └──────────┘
                    │
              ┌─────▼────┐
              │ PostgreSQL│
              └──────────┘
```

### Requirements

```
Hardware
├── CPU: 2+ cores
├── RAM: 4GB+
└── Disk: 100GB+ storage

Software
├── Docker Engine 20.10+
├── Docker Compose 2.0+
├── Ubuntu 22.04 (or compatible)
└── OpenSSL (for certificate generation)
```

### Services

```
nginx-proxy
├── Reverse proxy and SSL termination
├── Routes traffic to frontend
└── Manages Let's Encrypt certificates via acme-companion

acme-companion
├── Automatic SSL certificate generation and renewal
└── Only active when no domain certificate files exist in ./certs/nginx/

frontend
├── Next.js application serving user interface
├── Communicates with backend API
└── Port 3000 (internal)

backend
├── FastAPI application handling business logic
├── Manages authentication, chat, file uploads
└── Connects to database, S3, and optionally LDAP

pgbouncer
├── Connection pooling for PostgreSQL
├── Reduces database connection overhead
└── Supports SSL for client and server connections

postgres
├── PostgreSQL database
├── Stores application data
└── Optional SSL encryption

minio
├── S3-compatible object storage
├── Stores uploaded files and media
└── Can be replaced with external S3 service

migrate
├── Database migration service
├── Runs Alembic migrations on startup
└── Exits after completion
```

### Directory Structure

```
.
├── backend/              Backend application source
├── frontend/             Frontend application source
├── pgbouncer/            PgBouncer configuration and Dockerfile
├── postgres/             PostgreSQL configuration and Dockerfile
├── nginx/                Nginx virtual host configuration
├── certs/                SSL certificates directory
│   ├── backend/          Client certificates for backend
│   ├── pgbouncer/        Server certificates for PgBouncer
│   ├── postgres/         Server certificates for PostgreSQL
│   └── nginx/            Nginx SSL certificates
├── tools/                Utility scripts
│   ├── certs             Certificate generation tool
│   └── ldap              LDAP debugging tool
├── docker-compose.yml    Service orchestration
└── .env                  Configuration file
```

### Authentication Modes

**Database Mode** (`LDAP_USE_LDAP=False`)
- Users register with email and password
- User accounts stored in PostgreSQL
- Superuser created from `APP_SUPERUSER_EMAIL` and `APP_SUPERUSER_PASSWORD`
- Recommended: Change superuser password after first login
- Standard password reset and account management

**LDAP Mode** (`LDAP_USE_LDAP=True`)
- Registration disabled in frontend
- Recommended for Microsoft Active Directory
- Authentication flow:
  1. Service account (from `LDAP_USER` and `LDAP_PWD`) connects to LDAP server
  2. Service account queries directory for user data
  3. User credentials validated against directory
  4. User matched by `sAMAccountName` in Active Directory
- Superuser identified by matching `APP_SUPERUSER_EMAIL` to directory email
- `APP_SUPERUSER_PASSWORD` ignored
- Superuser can grant admin privileges to other users via `/admin` endpoint

## Configuration

Configuration is managed through the `.env` file. Copy `.env.example` to `.env` and adjust values for your deployment.

### General

**APP_ENV**
- Environment mode
- Values: `dev`, `prod`
- Default: `dev`

**APP_WORKERS**
- Number of backend worker processes
- Adjust based on CPU cores and load
- Recommended: 2-4 for small deployments, 4-8 for production
- Default: `3`

**APP_DOMAIN**
- Primary domain for the application
- Used for Nginx routing and Let's Encrypt
- Example: `chat.example.com`

**APP_CERTS_PATH**
- Base path for SSL certificates
- Relative to project root
- Default: `./certs`

**APP_LOG_LEVEL**
- Application logging verbosity
- Values: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`
- Default: `INFO`

**APP_SECRET_KEY**
- Secret key for session encryption and JWT signing
- Generate a strong random string
- Example: `openssl rand -hex 32`

**APP_ALLOWED_ORIGINS**
- CORS allowed origins as JSON array
- Include all domains that will access the API
- Example: `["https://chat.example.com","http://localhost:3000"]`

**APP_SUPERUSER_EMAIL**
- Email for initial superuser account
- In database mode: creates account with this email
- In LDAP mode: matches directory user by email to grant admin privileges
- Example: `admin@chat.example.com`

**APP_SUPERUSER_PASSWORD**
- Password for initial superuser account
- Only used in database mode
- Ignored in LDAP mode
- Example: `SecurePassword123`

### Local/Unsafe

**NODE_TLS_REJECT_UNAUTHORIZED**
- Disable TLS certificate validation in Node.js
- Required when using self-signed certificates for Nginx
- Values: `0` (disable), `1` (enable)
- Only use for testing/development
- Comment out or remove for production with valid certificates

### PgBouncer

Connection pooling configuration. Adjust based on expected concurrent users and database capacity.

**PGBOUNCER_LISTEN_PORT**
- Port PgBouncer listens on
- Default: `6432`

**PGBOUNCER_POOL_MODE**
- Connection pooling strategy
- Values: `session`, `transaction`, `statement`
- Recommended: `transaction` for most applications
- Default: `transaction`

**PGBOUNCER_MAX_CLIENT_CONN**
- Maximum client connections to PgBouncer
- Should be higher than expected concurrent users
- Default: `5000`

**PGBOUNCER_MIN_POOL_SIZE**
- Minimum database connections to maintain
- Default: `30`

**PGBOUNCER_DEFAULT_POOL_SIZE**
- Default database connections per user/database pair
- Default: `60`

**PGBOUNCER_RESERVE_POOL_SIZE**
- Additional connections for high load
- Default: `10`

**PGBOUNCER_RESERVE_POOL_TIMEOUT**
- Timeout for reserve pool connections (seconds)
- Default: `10`

**PGBOUNCER_CLIENT_TLS_SSLMODE**
- SSL mode for client connections to PgBouncer
- Values: `disable`, `allow`, `prefer`, `require`, `verify-ca`, `verify-full`
- Use `verify-full` with SSL certificates
- Default: `verify-full`

**PGBOUNCER_SERVER_TLS_SSLMODE**
- SSL mode for PgBouncer connections to PostgreSQL
- Values: same as `PGBOUNCER_CLIENT_TLS_SSLMODE`
- Default: `verify-full`

### Database

**DB_NAME**
- PostgreSQL database name
- Default: `anonym`

**DB_HOST**
- Database host
- For local: `pgbouncer` (uses connection pooling)
- For external: external hostname or IP
- Default: `pgbouncer`

**DB_USER**
- Database username
- Must match client certificate CN when using SSL
- Default: `postgres`

**DB_PASSWORD**
- Database password
- Default: `postgres`

**DB_PORT**
- Database port
- For PgBouncer: `6432`
- For direct PostgreSQL: `5432`
- Default: `6432`

**DB_MAX_CONNECTIONS**
- Maximum connections PostgreSQL will accept
- Must be higher than PgBouncer pool sizes
- Default: `120`

**DB_POOL_SIZE**
- SQLAlchemy connection pool size per backend worker
- Default: `15`

**DB_MAX_OVERFLOW**
- Additional connections beyond pool size
- Default: `5`

**DB_POOL_TIMEOUT**
- Connection acquisition timeout (seconds)
- Default: `35`

**DB_POOL_RECYCLE**
- Recycle connections after this many seconds
- Prevents stale connections
- Default: `1800` (30 minutes)

**DB_POOL_PRE_PING**
- Test connections before using
- Recommended: `True`
- Default: `True`

### Database Encryption

**DB_USE_SSL**
- Enable SSL for database connections
- Values: `True`, `False`
- Default: `True`

**DB_SSL_CA_FILE**
- CA certificate filename in `./certs/backend/`
- Default: `ca.pem`

**DB_SSL_CERT_FILE**
- Client certificate filename in `./certs/backend/`
- Default: `client-certificate.pem`

**DB_SSL_KEY_FILE**
- Client private key filename in `./certs/backend/`
- Default: `client-private-key.pem`

### LDAP

Configuration for LDAP/Active Directory authentication. Recommended for Microsoft Active Directory environments.

**LDAP_USE_LDAP**
- Enable LDAP authentication mode
- Values: `True`, `False`
- When `True`: disables registration, uses directory authentication
- Recommended for Microsoft Active Directory
- Default: `False`

**LDAP_HOST**
- LDAP server hostname or IP
- Example: `ad.example.com`

**LDAP_PORT**
- LDAP server port
- Standard LDAP: `389`
- LDAPS: `636`
- Default: `389`

**LDAP_USE_SSL**
- Enable SSL/TLS for LDAP
- Values: `True`, `False`, `starttls`, `ldaps`
- Recommended: `True` or `ldaps` for production
- Default: `False`

**LDAP_CA_FILE**
- CA certificate filename in `./certs/backend/`
- Required when `LDAP_USE_SSL=True`
- Example: `ldap_ca.pem`

**LDAP_USER**
- Service account username for LDAP queries
- Should have read access to user directory
- Example: `ldap_service_user`

**LDAP_PWD**
- Service account password
- Example: `ServicePassword123`

**LDAP_DOMAIN**
- Active Directory domain
- Example: `anonym.domain.com`

**LDAP_OU**
- Organizational Unit path for service account
- Example: `OU=SERVICE_USERS,OU=ACCOUNTS`

**LDAP_DCS**
- Domain Components for base DN
- Example: `DC=anonym,DC=domain,DC=com`

### MinIO

S3-compatible object storage configuration. Can be replaced with external S3 service (AWS, DigitalOcean Spaces, etc.).

**MINIO_ACCESS_KEY_ID**
- S3 access key
- For local MinIO: set any value
- For external S3: use provided access key
- Example: `s3_username`

**MINIO_SECRET_ACCESS_KEY**
- S3 secret key
- For local MinIO: set any value
- For external S3: use provided secret key
- Example: `s3_password`

**MINIO_BUCKET_NAME**
- S3 bucket name for file storage
- Bucket created automatically for local MinIO
- For external S3: create bucket manually
- Default: `chat-bucket`

**MINIO_ENDPOINT_URL**
- S3 endpoint URL
- For local MinIO: `http://minio:9000`
- For AWS S3: `https://s3.amazonaws.com`
- For other providers: use provider-specific endpoint

### OpenAI

**OPENAI_API_KEY**
- OpenAI API key for AI features
- Obtain from OpenAI platform
- Example: `sk-...`

**OPENAI_DEFAULT_MODEL_ID**
- Default OpenAI model to use
- Example: `gpt-4`, `gpt-3.5-turbo`
- Default: `gpt-4`

### Next.js

**NEXT_PUBLIC_API_URL**
- Backend API URL for frontend
- Should match `APP_DOMAIN`
- Example: `https://chat.example.com/api`

**NEXT_PUBLIC_USE_LDAP**
- Frontend LDAP mode flag
- Values: `true`, `false`
- Should match `LDAP_USE_LDAP`
- Controls registration UI visibility

## Tools

Utility scripts for certificate generation and LDAP debugging. Located in `tools/` directory.

### Certificate Generator

Script: `tools/certs`

Generates self-signed SSL certificates for all services. Certificates are valid for 10 years (3650 days).

**Prerequisites**

OpenSSL must be installed:

```bash
command -v openssl
```

**Commands**

Generate all certificates:

```bash
tools/certs all
```

Generate CA only:

```bash
tools/certs ca
```

Generate specific service certificates:

```bash
tools/certs nginx
```

```bash
tools/certs postgres
```

```bash
tools/certs pgbouncer
```

```bash
tools/certs backend
```

Show required environment variables:

```bash
tools/certs env
```

Show configured domains:

```bash
tools/certs domains
```

Verify client certificate:

```bash
tools/certs verify
```

Clean all generated certificates:

```bash
tools/certs clean
```

**Certificate Details**

- **CA Certificate**: Self-signed root CA for signing service certificates
- **Nginx Certificate**: Self-signed certificate for domain (not from CA)
- **PostgreSQL Certificate**: Server certificate signed by CA
- **PgBouncer Certificate**: Server certificate signed by CA
- **Backend Certificate**: Client certificate signed by CA, CN must match `DB_USER`

**After Generation**

1. Verify client certificate CN matches `DB_USER`
2. Restart services to load new certificates
3. Set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env` for self-signed Nginx certificates

**Certificate Locations**

- CA: `./certs/ca.pem`, `./certs/ca.key`
- Nginx: `./certs/nginx/${APP_DOMAIN}.crt`, `./certs/nginx/${APP_DOMAIN}.key`
- PostgreSQL: `./certs/postgres/server.crt`, `./certs/postgres/server.key`, `./certs/postgres/ca.pem`
- PgBouncer: `./certs/pgbouncer/server.crt`, `./certs/pgbouncer/server.key`, `./certs/pgbouncer/ca.pem`
- Backend: `./certs/backend/client-certificate.pem`, `./certs/backend/client-private-key.pem`, `./certs/backend/ca.pem`

### LDAP Debugger

Script: `tools/ldap`

Tests LDAP/Active Directory connectivity and authentication. Useful for troubleshooting LDAP configuration issues.

**Prerequisites**

Required tools:

```bash
ldapwhoami
```

```bash
ldapsearch
```

```bash
openssl
```

Install on Ubuntu/Debian:

```bash
apt-get install ldap-utils openssl
```

**Usage**

Test LDAP authentication:

```bash
tools/ldap username password
```

Replace `username` and `password` with actual directory credentials.

**Test Steps**

The tool performs the following checks:

1. Network reachability to LDAP server
2. SSL/TLS certificate validation (if enabled)
3. Service account bind test
4. User search by `sAMAccountName`
5. User authentication with discovered DN

**Configuration**

The tool reads configuration from `.env` file. Ensure the following variables are set:

- `LDAP_HOST` - See [Configuration - LDAP](#ldap)
- `LDAP_PORT` - See [Configuration - LDAP](#ldap)
- `LDAP_USE_SSL` - See [Configuration - LDAP](#ldap)
- `LDAP_CA_FILE` - See [Configuration - LDAP](#ldap)
- `LDAP_USER` - See [Configuration - LDAP](#ldap)
- `LDAP_PWD` - See [Configuration - LDAP](#ldap)
- `LDAP_DOMAIN` - See [Configuration - LDAP](#ldap)
- `LDAP_OU` - See [Configuration - LDAP](#ldap)
- `LDAP_DCS` - See [Configuration - LDAP](#ldap)

**Common Issues**

**Network unreachable**
- Verify `LDAP_HOST` and `LDAP_PORT`
- Check firewall rules
- Ensure DNS resolution works

**SSL handshake failed**
- Verify `LDAP_USE_SSL` matches server configuration
- Ensure `LDAP_CA_FILE` is present in `./certs/backend/`
- Check certificate validity

**Service bind failed**
- Verify `LDAP_USER` and `LDAP_PWD` credentials
- Check `LDAP_OU` and `LDAP_DCS` are correct
- Ensure service account has directory read permissions

**User not found**
- Verify user exists in directory
- Check `LDAP_DCS` base DN is correct
- Ensure user is in searchable OU

**User authentication failed**
- Verify user password is correct
- Check user account is not locked or disabled
- Ensure user has login permissions

---

## Best Practices

### Version Control

When customizing this template:

1. Create a private offline Git branch for your modifications
2. Keep the original branch as backup
3. Merge updates from upstream carefully
4. Document your changes in commit messages

### Security

- Use strong passwords for all services
- Enable SSL/TLS for production deployments
- Use production certificates from trusted CA (Let's Encrypt, commercial CA)
- Restrict `APP_ALLOWED_ORIGINS` to known domains
- Keep Docker images updated
- Regularly rotate `APP_SECRET_KEY` and service passwords

### Scaling

- Increase `APP_WORKERS` based on CPU cores
- Adjust PgBouncer pool sizes for concurrent users
- Monitor database connections and adjust `DB_MAX_CONNECTIONS`
- Consider external PostgreSQL for high-traffic deployments
- Use external S3 service for better file storage scalability

### Monitoring

- Check Docker logs: `docker compose logs -f [service]`
- Monitor resource usage: `docker stats`
- Review application logs in backend container
- Set up external monitoring for production

---

**Support**: For issues or questions, contact the development team.

**License**: Proprietary - All rights reserved.

