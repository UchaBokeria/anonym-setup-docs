## Quick Start

### Installation Requirements

See [Architecture - Requirements](docs/architecture.md#requirements) for detailed system requirements.

### Installation

**1. Add SSH key to GitHub**

This is a private repository. Ensure your SSH key is added to your GitHub account before cloning.

**2. Clone repository**

```terminal
git clone git@github.com:Giorgi-Sekhniashvili/anonym-chat-setup.git
```

**3. Navigate to project**

```terminal
cd anonym-chat-setup
```

**4. Create environment file**

```terminal
cp .env.example .env
```

This creates your configuration file with default values. You can now configure the application using either method:

**Option A: Web UI Configuration (Recommended)**

The WUI will read the `.env` file and populate all fields with current values. See [Web UI Configuration](docs/web-ui-configuration.md) for detailed instructions.

**Option B: Manual Configuration**

Edit `.env` directly and set required configurations. See [Configuration](docs/configuration.md) for detailed options.

**5. Configure SSL certificates (Optional)**

You can skip this step if you are not using ssl.

**For Nginx:**

- **Production certificates**: Place your certificates in `./certs/nginx/` named as `${APP_DOMAIN}.crt` and `${APP_DOMAIN}.key`
- **Let's Encrypt (free)**: Ensure no certificate files for your domain exist in `./certs/nginx/` - certificates will be generated automatically
- **Self-signed certificates**: Generate using `tools/certs` (see [Tools](docs/tools.md)) and set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env` (see [Configuration - Local/Unsafe](docs/configuration.md#localunsafe))

**For Database SSL:**

1. Place client certificates in `./certs/backend/`
2. Place server certificates in `./certs/pgbouncer/`
3. Place server certificates in `./certs/postgres/`

**For LDAP SSL:**

Place CA certificate in `./certs/backend/${LDAP_CA_FILE}` where `${LDAP_CA_FILE}` matches your `.env` configuration.

**Generate all self-signed certificates:**

```terminal
tools/certs all
```

See [Tools - Certificate Generator](docs/tools.md#certificate-generator) for more options.

**Note**: Alternatively, see [Web UI Configuration - Certificates](docs/web-ui-configuration.md#certificates-tab) for web-based certificate management.

**6. Login to Docker registry (required)**

```terminal
docker login registry.anonym.giorgis.ink
```

Enter the username and password provided to you. Registry access credentials are provided before deployment.

**7. Start services**

```terminal
docker compose up -d
```

**8. Post-deployment**

For database authentication mode, change the superuser password after first login for security.

### Update or Reconfigure

**1. Pull latest changes**

```terminal
git pull
```

Or apply your local changes if working on a custom branch.

**2. Review patch notes**

Check patch notes provided with updates for breaking changes or migration steps.

**3. Stop affected services**

Stop all services:

```terminal
docker compose down
```

Stop specific services:

```terminal
docker compose down backend frontend
```

**4. Apply environment changes**

Edit `.env` if configuration updates are required.

**5. Run migrations**

If database schema changes are included:

```terminal
docker compose up -d --build migrate
```

**6. Restart services**

```terminal
docker compose up -d
```

### Common Issues

**Cached volumes after configuration changes**

Docker volumes persist data between restarts. After configuration changes, you may need to remove volumes:

```terminal
docker compose down -v <service_name>
```

**Warning**: Removing ```postgres``` service with volumes will result removing the `postgres-data` volume and will delete all database data.

**Self-signed Nginx certificates**

When using self-signed certificates for Nginx, set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env`. See [Configuration - Local/Unsafe](docs/configuration.md#localunsafe).

**Let's Encrypt not activating**

Ensure no certificate files for your domain exist in `./certs/nginx/`. The presence of `${APP_DOMAIN}.crt` or `${APP_DOMAIN}.key` will prevent the ACME companion from generating certificates.

**LDAP connection issues**

Verify LDAP SSL configuration:
- Ensure `LDAP_CA_FILE` is present in `./certs/backend/` when using SSL
- Use the LDAP debugging tool: `tools/ldap username password`

See [Tools - LDAP Debugger](docs/tools.md#ldap-debugger) for detailed troubleshooting.

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

**All rights reserved.**
