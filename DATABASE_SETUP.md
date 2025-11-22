# PostgreSQL Database Setup with Traefik SSL

This guide explains how to set up and connect to your PostgreSQL database secured with Traefik's Let's Encrypt SSL certificates.

## Overview

Your setup includes:
- **PostgreSQL 16**: Secure database server
- **pgAdmin**: Web-based database management interface with SSL
- **Traefik**: Automatic SSL/TLS certificate management via Let's Encrypt

## Configuration

### 1. Update Environment Variables

Copy `.env.example` to `.env` and update the values:

```bash
cp .env.example .env
nano .env
```

Update these database-specific variables:
- `DB_USER`: Database username (default: dbadmin)
- `DB_PASSWORD`: Strong password for database access
- `DB_NAME`: Database name (default: myappdb)
- `PGADMIN_DOMAIN`: Domain for pgAdmin web interface (e.g., pgadmin.yourdomain.com)
- `PGADMIN_EMAIL`: Email for pgAdmin login
- `PGADMIN_PASSWORD`: Password for pgAdmin login

### 2. DNS Configuration

Point your pgAdmin domain to your server's IP address:
```
A Record: pgadmin.yourdomain.com → YOUR_SERVER_IP
```

### 3. Start the Services

```bash
docker-compose up -d
```

This will:
- Start PostgreSQL database
- Start pgAdmin web interface
- Automatically obtain SSL certificates from Let's Encrypt

## Connecting to the Database

### Option 1: Via pgAdmin Web Interface (Recommended for Management)

1. Navigate to `https://pgadmin.yourdomain.com`
2. Login with your `PGADMIN_EMAIL` and `PGADMIN_PASSWORD`
3. Add a new server:
   - **General Tab**:
     - Name: My PostgreSQL Server
   - **Connection Tab**:
     - Host: `postgres` (container name)
     - Port: `5432`
     - Maintenance database: Value of `DB_NAME`
     - Username: Value of `DB_USER`
     - Password: Value of `DB_PASSWORD`
     - Save password: ✓

### Option 2: Direct Database Connection from Internet

To connect directly to PostgreSQL from external applications, you need to expose the database port through Traefik.

#### Add TCP Router to docker-compose.yml

Add this to the Traefik command section:

```yaml
# In traefik service command section, add:
- --entrypoints.postgres.address=:5432
```

Add this to the Traefik ports section:

```yaml
ports:
  - 80:80
  - 443:443
  - 5432:5432  # PostgreSQL port
```

Add labels to the postgres service:

```yaml
# In postgres service, add labels:
labels:
  traefik.enable: true
  traefik.tcp.routers.postgres.rule: HostSNI(`*`)
  traefik.tcp.routers.postgres.entrypoints: postgres
  traefik.tcp.routers.postgres.service: postgres
  traefik.tcp.services.postgres.loadbalancer.server.port: 5432
```

Then restart:
```bash
docker-compose down
docker-compose up -d
```

#### Connection String Examples

**PostgreSQL Connection String:**
```
postgresql://DB_USER:DB_PASSWORD@your-server-ip:5432/DB_NAME
```

**Python (psycopg2):**
```python
import psycopg2

conn = psycopg2.connect(
    host="your-server-ip",
    port=5432,
    database="DB_NAME",
    user="DB_USER",
    password="DB_PASSWORD",
    sslmode="prefer"
)
```

**Node.js (pg):**
```javascript
const { Client } = require('pg');

const client = new Client({
  host: 'your-server-ip',
  port: 5432,
  database: 'DB_NAME',
  user: 'DB_USER',
  password: 'DB_PASSWORD',
  ssl: false  // Set to true if you configure PostgreSQL SSL
});

await client.connect();
```

**Command Line (psql):**
```bash
psql -h your-server-ip -p 5432 -U DB_USER -d DB_NAME
```

### Option 3: Enable PostgreSQL Native SSL (Advanced)

For end-to-end encryption, you can configure PostgreSQL to use SSL certificates:

1. Generate self-signed certificates or use Traefik's certificates
2. Mount certificates to PostgreSQL container
3. Configure PostgreSQL to require SSL connections

This requires additional configuration in `postgresql.conf` and certificate management.

## Security Best Practices

1. **Strong Passwords**: Use complex passwords for `DB_PASSWORD` and `PGADMIN_PASSWORD`
2. **Firewall Rules**: Restrict port 5432 access to trusted IP addresses only
3. **Regular Backups**: Set up automated database backups
4. **Update Regularly**: Keep PostgreSQL and pgAdmin images updated
5. **Network Isolation**: Consider using Docker networks to isolate database access
6. **SSL Connections**: Enable PostgreSQL SSL for production environments

## Database Backup

### Manual Backup
```bash
docker exec postgres pg_dump -U DB_USER DB_NAME > backup_$(date +%Y%m%d).sql
```

### Restore from Backup
```bash
docker exec -i postgres psql -U DB_USER DB_NAME < backup_20240101.sql
```

## Troubleshooting

### Check Container Logs
```bash
docker logs postgres
docker logs pgadmin
docker logs traefik
```

### Verify Database is Running
```bash
docker exec postgres pg_isready -U DB_USER
```

### Test Connection from Container
```bash
docker exec -it postgres psql -U DB_USER -d DB_NAME
```

### SSL Certificate Issues
- Ensure DNS is properly configured
- Check Traefik logs: `docker logs traefik`
- Verify Let's Encrypt rate limits haven't been exceeded
- Ensure ports 80 and 443 are accessible from the internet

## Customization

### Modify Initial Database Schema

Edit `init-db.sql` to customize the initial database structure. This script runs only on first container creation.

### Change PostgreSQL Configuration

Create a custom `postgresql.conf` and mount it:
```yaml
volumes:
  - ./postgresql.conf:/etc/postgresql/postgresql.conf
```

## Monitoring

Monitor database performance through pgAdmin or add monitoring tools like:
- Prometheus + Grafana
- pgBadger for log analysis
- pg_stat_statements extension

## Support

For issues:
- PostgreSQL: https://www.postgresql.org/docs/
- pgAdmin: https://www.pgadmin.org/docs/
- Traefik: https://doc.traefik.io/traefik/
