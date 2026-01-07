# Tools

Utility scripts for certificate generation and LDAP debugging. Located in `tools/` directory.

## Certificate Generator

Script: `tools/certs`

Generates self-signed SSL certificates for all services. Certificates are valid for 10 years (3650 days).

**Prerequisites**

OpenSSL must be installed:

```terminal
command -v openssl
```

**Commands**

Generate all certificates:

```terminal
tools/certs all
```

Generate CA only:

```terminal
tools/certs ca
```

Generate specific service certificates:

For Nginx:
```terminal
tools/certs nginx
```

For PostgreSQL:
```terminal
tools/certs postgres
```

For PgBouncer:
```terminal
tools/certs pgbouncer
```

For Backend:
```terminal
tools/certs backend
```

Show required environment variables:

```terminal
tools/certs env
```

Show configured domains:

```terminal
tools/certs domains
```

Verify client certificate:

```terminal
tools/certs verify
```

Clean all generated certificates:

```terminal
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

## LDAP Debugger

Script: `tools/ldap`

Tests LDAP/Active Directory connectivity and authentication. Useful for troubleshooting LDAP configuration issues.

**Prerequisites**

Required tools:

Check for ldapwhoami:
```terminal
ldapwhoami
```

Check for ldapsearch:
```terminal
ldapsearch
```

Check for openssl:
```terminal
openssl
```

Install on Ubuntu/Debian:

```terminal
apt-get install ldap-utils openssl
```

**Usage**

Test LDAP authentication:

```terminal
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

- `LDAP_HOST` - See [Configuration - LDAP](configuration.md#ldap)
- `LDAP_PORT` - See [Configuration - LDAP](configuration.md#ldap)
- `LDAP_USE_SSL` - See [Configuration - LDAP](configuration.md#ldap)
- `LDAP_CA_FILE` - See [Configuration - LDAP](configuration.md#ldap)
- `LDAP_USER` - See [Configuration - LDAP](configuration.md#ldap)
- `LDAP_PWD` - See [Configuration - LDAP](configuration.md#ldap)
- `LDAP_DOMAIN` - See [Configuration - LDAP](configuration.md#ldap)
- `LDAP_OU` - See [Configuration - LDAP](configuration.md#ldap)
- `LDAP_DCS` - See [Configuration - LDAP](configuration.md#ldap)

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

