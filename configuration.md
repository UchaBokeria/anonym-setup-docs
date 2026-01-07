# Configuration

Configuration is managed through the `.env` file. Copy `.env.example` to `.env` and adjust values for your deployment.

## General

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

## Local/Unsafe

**NODE_TLS_REJECT_UNAUTHORIZED**
- Disable TLS certificate validation in Node.js
- Required when using self-signed certificates for Nginx
- Values: `0` (disable), `1` (enable)
- Only use for testing/development
- Comment out or remove for production with valid certificates

## PgBouncer

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

## Database

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

## Database Encryption

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

## LDAP

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

## MinIO

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

## OpenAI

**OPENAI_API_KEY**
- OpenAI API key for AI features
- Obtain from OpenAI platform
- Example: `sk-...`

**OPENAI_DEFAULT_MODEL_ID**
- Default OpenAI model to use
- Example: `gpt-4`, `gpt-3.5-turbo`
- Default: `gpt-4`

## Next.js

**NEXT_PUBLIC_API_URL**
- Backend API URL used by frontend (browser) through Nginx
- Should match `APP_DOMAIN`
- Example: `https://chat.example.com/api`
- Recommended: Leave as default

**NEXT_PRIVATE_API_URL**
- Backend API URL used by Next.js server-side
- Internal Docker network communication
- Example: `http://backend:8000/api`
- Recommended: Leave as default

**NEXT_PUBLIC_USE_LDAP**
- Frontend LDAP mode flag
- Values: `true`, `false`
- Should match `LDAP_USE_LDAP`
- Controls registration UI visibility

