# Supabase Postgres External Setup Guide

This document provides detailed instructions on how to take a Postgres container setup for Supabase and deploy it on an external Postgres instance. It covers special configurations and extensions needed for Postgres to function properly with Supabase.

## Prerequisites

Before you begin, ensure you have the following:
- An external PostgreSQL server (version 12 or higher recommended)
- Access to the PostgreSQL server with superuser privileges
- Basic knowledge of PostgreSQL administration
- Docker and Docker Compose installed (for reference purposes)

## Step 1: Set Up Your External PostgreSQL Server

### Installation

If you don't already have PostgreSQL installed on your external server, you can install it using your preferred package manager:

```bash
# For Ubuntu/Debian
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib

# For CentOS/RHEL
sudo yum install epel-release
sudo yum install postgresql-server postgresql-contrib
sudo postgresql-setup initdb
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### Basic Configuration

1. Create a dedicated database for your Supabase project:
   ```bash
   psql -U postgres -c "CREATE DATABASE my_supabase_db;"
   ```

2. Create a user with appropriate privileges:
   ```bash
   psql -U postgres -c "CREATE USER my_supabase_user WITH PASSWORD 'your_secure_password';"
   psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE my_supabase_db TO my_supabase_user;"
   ```

3. Update `pg_hba.conf` to allow connections from your application servers:
   ```bash
   # Add these lines to your pg_hba.conf file
   host    all             all             0.0.0.0/0               md5
   host    all             all             ::/0                    md5
   ```

4. Update `postgresql.conf` with appropriate settings:
   ```bash
   # Add or modify these settings in your postgresql.conf file
   listen_addresses = '*'
   max_connections = 100
   shared_buffers = 256MB
   effective_cache_size = 768MB
   maintenance_work_mem = 64MB
   wal_level = replica
   ```

## Step 2: Install Required PostgreSQL Extensions

Supabase relies on several PostgreSQL extensions. You'll need to install these on your external PostgreSQL server:

```bash
# Connect to your PostgreSQL server
psql -U my_supabase_user -d my_supabase_db

# Install required extensions
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE EXTENSION IF NOT EXISTS pg_repack;
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pg_cron;
CREATE EXTENSION IF NOT EXISTS pgmq;
CREATE EXTENSION IF NOT EXISTS pgsodium;
CREATE EXTENSION IF NOT EXISTS supabase_vault;
CREATE EXTENSION IF NOT EXISTS pg_tle;
CREATE EXTENSION IF NOT EXISTS pg_repack;
```

## Step 3: Configure Supabase for External PostgreSQL

### Environment Variables

Create a `.env` file in your Supabase project directory with the connection details for your external PostgreSQL server:

```env
# Database connection
DB_HOST=your_external_postgres_host
DB_PORT=5432
DB_USER=my_supabase_user
DB_PASSWORD=your_secure_password
DB_NAME=my_supabase_db

# Supabase specific
SUPABASE_URL=https://your-supabase-project.supabase.co
SUPABASE_ANON_KEY=your_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
```

### Docker Compose Configuration

If you're using Docker Compose, update your `docker-compose.yml` to use your external PostgreSQL instance:

```yaml
version: '3'

services:
  # Other services...
  
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - ./data/postgresql:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    command: >
     postgres -c 'config_file=/etc/postgresql/postgresql.conf'
    # Override the default postgresql.conf with your external server's configuration
    # This is just for local development, your external server will use its own config
    volumes:
      - ./config/postgresql.conf:/etc/postgresql/postgresql.conf
```

## Step 4: Special Configurations for External PostgreSQL

### Time Zone Configuration

Ensure your external PostgreSQL server is using the correct time zone:

```bash
# For PostgreSQL 12+
ALTER SYSTEM SET timezone TO 'UTC';
SELECT pg_reload_conf();
```

### Connection Pooling

For better performance, consider setting up a connection pool:

```bash
# Create a pgBouncer user
psql -U postgres -c "CREATE USER pg_bouncer WITH PASSWORD 'your_pg_bouncer_password';"

# Configure pgBouncer
cat > /etc/pgbouncer/pgbouncer.ini << EOF
[databases]
* = host=your_external_postgres_host port=5432 user=my_supabase_user password=your_secure_password database=my_supabase_db

[pgbouncer]
listen_address = *
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/users.txt
admin_users = my_supabase_user,pg_bouncer
pool_mode = transaction
max_client_conn = 100
default_pool_size = 20
reserve_pool_size = 5
EOF

# Create users.txt
cat > /etc/pgbouncer/users.txt << EOF
my_supabase_user:your_secure_password
pg_bouncer:your_pg_bouncer_password
EOF

# Start pgBouncer
pg_ctl start
```

### Backup and Recovery

Set up regular backups for your external PostgreSQL server:

```bash
# Create a backup script
cat > /usr/local/bin/backup_postgres.sh << 'EOF'
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/var/backups/postgres"
mkdir -p $BACKUP_DIR
pg_dump -U my_supabase_user -d my_supabase_db -Fc -f $BACKUP_DIR/backup_$TIMESTAMP.dump
# Keep only the last 7 backups
ls -t $BACKUP_DIR/backup_*.dump | tail -n +8 | xargs rm -f
EOF

# Make it executable
chmod +x /usr/local/bin/backup_postgres.sh

# Set up a cron job for daily backups
crontab -l | { cat; echo "0 3 * * * /usr/local/bin/backup_postgres.sh"; } | crontab -
```

## Step 5: Migrating Data from Supabase to External PostgreSQL

If you need to migrate existing data from a Supabase PostgreSQL container to your external PostgreSQL server:

1. Dump the data from your Supabase PostgreSQL container:
   ```bash
   docker exec your_supabase_postgres_container pg_dump -U postgres -d supabase_db -Fc -f /tmp/supabase_backup.dump
   docker cp your_supabase_postgres_container:/tmp/supabase_backup.dump ./supabase_backup.dump
   ```

2. Restore the data to your external PostgreSQL server:
   ```bash
   pg_restore -U my_supabase_user -d my_supabase_db -Fc ./supabase_backup.dump
   ```

## Step 6: Testing the Connection

Test the connection from your Supabase application to your external PostgreSQL server:

```bash
psql "host=your_external_postgres_host port=5432 user=my_supabase_user password=your_secure_password dbname=my_supabase_db"
```

## Step 7: Monitoring and Maintenance

Set up monitoring for your external PostgreSQL server:

1. Install and configure pg_stat_statements:
   ```bash
   psql -U my_supabase_user -d my_supabase_db -c "CREATE EXTENSION IF NOT EXISTS pg_stat_statements;"
   ```

2. Install and configure pgBadger for log analysis:
   ```bash
   # Install pgbadger
   sudo apt-get install pgbadger
   
   # Generate a report
   pgbadger -f stderr -d /var/log/postgresql/postgresql.log -o /var/www/pgbadger/report.html
   ```

3. Set up alerts for critical events:
   ```bash
   # Create a function to send alerts
   psql -U my_supabase_user -d my_supabase_db <<EOF
   CREATE OR REPLACE FUNCTION alert_on_error()
   RETURNS pg_catalog.trigger AS \$$
   DECLARE
     error_text text;
   BEGIN
     error_text := TG_ARGV[0];
     -- Replace with your alerting system
     PERFORM pg_notify('alerts', error_text);
     RETURN NULL;
   END;
   $$ LANGUAGE plpgsql;
   
   -- Create a trigger for errors
   CREATE TRIGGER alert_trigger
   AFTER EXCEPTIONS ON DATABASE
   EXECUTE FUNCTION alert_on_error();
   EOF
   ```

## Troubleshooting

### Common Issues

1. **Connection Errors**:
   - Ensure your PostgreSQL server is listening on the correct port
   - Check firewall settings to allow connections
   - Verify user credentials and permissions

2. **Extension Missing Errors**:
   - Make sure all required extensions are installed
   - Check for typos in extension names

3. **Performance Issues**:
   - Review PostgreSQL configuration parameters
   - Check for slow queries using `pg_stat_statements`
   - Consider adding more memory or CPU resources

4. **Data Migration Issues**:
   - Ensure data types are compatible between Supabase and your external PostgreSQL
   - Check for any schema differences that might cause issues

## Conclusion

By following these steps, you should be able to successfully deploy a Supabase PostgreSQL setup on an external PostgreSQL server. Remember to regularly monitor your database, perform backups, and update your configuration as needed to ensure optimal performance and reliability.

For more information, refer to the official PostgreSQL documentation and the Supabase documentation.