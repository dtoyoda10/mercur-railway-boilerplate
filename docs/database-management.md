# Database Management

Everything you need to know about managing your PostgreSQL database.

## Quick Reference

```bash
# Initialize database (creates tables)
pnpm run db:init

# Run migrations
pnpm run db:migrate

# Create admin user
pnpm run db:admin

# Access database CLI
docker exec -it mercur-postgres psql -U postgres -d medusa_db
```

## Database Migrations

### What Are Migrations?

Migrations are version control for your database schema. They let you:
- Track schema changes over time
- Apply changes consistently across environments
- Roll back changes if needed

### Running Migrations

**First time setup:**
```bash
pnpm run db:init
```

This runs all migrations and sets up the database.

**After pulling new code:**
```bash
pnpm run db:migrate
```

This applies any new migrations that were added.

### Creating Custom Migrations

If you add custom modules or modify the schema:

```bash
cd apps/backend

# Generate a migration file
npx medusa migrations create MyCustomMigration

# Edit the generated file in src/migrations/
# Add your schema changes

# Run the migration
pnpm run db:migrate
```

**Example migration:**
```typescript
// src/migrations/1234567890_AddCustomField.ts
import { MigrationInterface, QueryRunner } from "typeorm"

export class AddCustomField1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE product
      ADD COLUMN custom_field VARCHAR(255)
    `)
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE product
      DROP COLUMN custom_field
    `)
  }
}
```

### Rolling Back Migrations

If a migration breaks something:

```bash
cd apps/backend

# Revert last migration
npx medusa migrations revert

# Revert multiple migrations
npx medusa migrations revert
npx medusa migrations revert
```

**Warning:** Only do this in development. Rolling back in production can cause data loss.

## Database Backups

### Why Backup?

- Hardware failures
- Software bugs
- Accidental deletions
- Security breaches
- Data corruption

**Rule of thumb:** If you can't afford to lose it, back it up.

### Railway Automatic Backups

Railway PostgreSQL includes daily backups (on paid plans).

**To enable:**
1. Go to Railway Dashboard
2. Click on PostgreSQL service
3. Go to Settings → Backups
4. Enable automated backups

**Restoration:**
1. Railway Dashboard → PostgreSQL → Backups
2. Select backup
3. Click "Restore"

### Manual Backups

**Create a backup:**
```bash
# Local database
pg_dump -U postgres -d medusa_db > backup_$(date +%Y%m%d).sql

# Railway database (get URL from Railway dashboard)
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql
```

**Compress backup:**
```bash
pg_dump -U postgres -d medusa_db | gzip > backup_$(date +%Y%m%d).sql.gz
```

**Schedule automated backups with cron:**
```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * pg_dump $DATABASE_URL | gzip > /backups/medusa_$(date +\%Y\%m\%d).sql.gz
```

### Restore from Backup

**From SQL file:**
```bash
# Drop existing database (WARNING: deletes all data)
dropdb medusa_db

# Create new database
createdb medusa_db

# Restore backup
psql -U postgres -d medusa_db < backup_20240115.sql
```

**From compressed backup:**
```bash
gunzip < backup_20240115.sql.gz | psql -U postgres -d medusa_db
```

**Railway database:**
```bash
# Get connection string from Railway dashboard
psql $DATABASE_URL < backup_20240115.sql
```

## Database Maintenance

### Vacuum Database

PostgreSQL needs regular vacuuming to reclaim space and optimize performance.

```bash
# Connect to database
psql $DATABASE_URL

# Vacuum all tables
VACUUM VERBOSE;

# Full vacuum (slower but more thorough)
VACUUM FULL;

# Analyze statistics (improves query performance)
ANALYZE;
```

**Auto-vacuum** is enabled by default in PostgreSQL 16.

### Check Database Size

```sql
-- Total database size
SELECT pg_size_pretty(pg_database_size('medusa_db'));

-- Size per table
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;
```

### Optimize Queries

**Find slow queries:**
```sql
-- Enable query logging (do this once)
ALTER DATABASE medusa_db SET log_min_duration_statement = 1000;  -- Log queries > 1 second

-- View slow queries in logs
SELECT * FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

**Add indices for slow queries:**
```sql
-- Example: Speed up product searches
CREATE INDEX idx_product_title ON product USING gin(to_tsvector('english', title));

-- Index on foreign keys
CREATE INDEX idx_order_customer ON "order"(customer_id);
```

## Database Monitoring

### Connection Count

```sql
-- Current connections
SELECT count(*) FROM pg_stat_activity;

-- Connections per database
SELECT datname, count(*)
FROM pg_stat_activity
GROUP BY datname;
```

**Max connections:**
```sql
SHOW max_connections;
```

If you hit the limit, either:
- Increase `max_connections` (Railway plans have limits)
- Use connection pooling (PgBouncer)
- Fix connection leaks in your code

### Long-Running Queries

```sql
-- Queries running > 5 minutes
SELECT
  pid,
  now() - query_start AS duration,
  query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '5 minutes';
```

**Kill a stuck query:**
```sql
SELECT pg_terminate_backend(12345);  -- Replace 12345 with PID
```

### Database Health

```sql
-- Check for bloat
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
  pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) AS index_size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Seeding Data

### Run Seed Script

```bash
cd apps/backend

# Run the seed script
npm run seed
```

This creates:
- Default sales channel
- Default region (US)
- Sample product categories
- Test seller account
- Sample products
- Commission rules

### Custom Seed Data

Edit `apps/backend/src/scripts/seed.ts` to customize:

```typescript
// Add your own products
await createProduct(container, {
  title: "My Custom Product",
  description: "Product description",
  // ... other fields
})
```

### Seed Production Database

**Careful!** Seeding adds data, it doesn't clear existing data.

```bash
# For production, you probably want to add specific data, not seed everything
# Connect to production database
psql $PRODUCTION_DATABASE_URL

# Then run specific INSERT statements
```

## Database Access

### Local Database

Using Docker:
```bash
# Connect to PostgreSQL CLI
docker exec -it mercur-postgres psql -U postgres -d medusa_db

# Or use a GUI (TablePlus, pgAdmin, DBeaver)
# Connection details:
# Host: localhost
# Port: 5432
# User: postgres
# Password: postgres
# Database: medusa_db
```

### Production Database (Railway)

**Via Railway CLI:**
```bash
railway connect postgres
```

**Direct connection:**
```bash
# Get DATABASE_URL from Railway dashboard
psql $DATABASE_URL
```

**Warning:** Be extra careful when accessing production database. Always backup before making changes.

## Common Tasks

### Reset Database (Development Only)

**Warning:** This deletes ALL data.

```bash
# Stop backend
# Drop database
docker-compose down -v

# Recreate
docker-compose up -d
pnpm run db:init
pnpm run db:admin
```

### Move Database to Different Server

```bash
# Backup from old server
pg_dump $OLD_DATABASE_URL > migration.sql

# Restore to new server
psql $NEW_DATABASE_URL < migration.sql

# Update DATABASE_URL in your .env
# Restart backend
```

### Duplicate Production to Staging

```bash
# Backup production
pg_dump $PROD_DATABASE_URL > prod_backup.sql

# Restore to staging
psql $STAGING_DATABASE_URL < prod_backup.sql

# Anonymize sensitive data in staging
psql $STAGING_DATABASE_URL <<SQL
UPDATE customer SET email = concat('test+', id, '@example.com');
UPDATE "user" SET email = concat('admin+', id, '@example.com');
-- Clear payment methods, etc.
SQL
```

## Troubleshooting

### "too many connections"

**Fix:**
```bash
# Kill idle connections
psql $DATABASE_URL -c "
  SELECT pg_terminate_backend(pid)
  FROM pg_stat_activity
  WHERE state = 'idle'
  AND query_start < now() - interval '10 minutes'
"
```

### "disk full" error

**Check disk usage:**
```sql
SELECT pg_size_pretty(pg_database_size('medusa_db'));
```

**Cleanup:**
1. Vacuum database
2. Delete old backups
3. Clear logs
4. Upgrade Railway plan for more storage

### Migrations failing

**Error:** `relation "product" already exists`

**Fix:**
```bash
# Check what migrations ran
psql $DATABASE_URL -c "SELECT * FROM migrations"

# Manually mark migration as run (if you know it's already applied)
psql $DATABASE_URL -c "
  INSERT INTO migrations (timestamp, name)
  VALUES (1234567890, 'ProductTable1234567890')
"
```

## Best Practices

- ✅ Backup before deploying schema changes
- ✅ Test migrations on staging first
- ✅ Use transactions for multi-step operations
- ✅ Add indices for frequently queried columns
- ✅ Monitor query performance
- ✅ Keep backups off-site (not on same server)
- ✅ Test backup restoration regularly
- ❌ Never run `DROP TABLE` in production without backup
- ❌ Don't expose database publicly
- ❌ Don't share production database credentials

## Resources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MedusaJS Database Guide](https://docs.medusajs.com/development/backend/database)
- [Railway PostgreSQL Docs](https://docs.railway.app/databases/postgresql)
- [pgAdmin](https://www.pgadmin.org/) - Free PostgreSQL GUI
- [TablePlus](https://tableplus.com/) - Modern database GUI

---

**Need Help?** Check [troubleshooting guide](troubleshooting.md) or open an issue.
