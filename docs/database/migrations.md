# Database Migrations & Versioning

Veilige database schema evolution met backwards compatibility.

---

## Strategy: Semantic Versioning for Schemas

Migrations tagged per API version:

- **Major** (1.0.0 → 2.0.0): Breaking schema changes (dropped columns/tables)
- **Minor** (1.0.0 → 1.1.0): Backward-compatible additions (new columns, tables)
- **Patch** (1.0.0 → 1.0.1): Data corrections (constraint fixes, indexes)

---

## Migration Tools

**Tool**: Alembic (Python) or Flyway (Java)

```bash
# Initialize migration environment
alembic init migrations

# Create migration
alembic revision --autogenerate -m "Add schuldhulpmaatregel table"

# Apply migration
alembic upgrade head

# Rollback
alembic downgrade -1
```

---

## Versioning Table

```sql
CREATE TABLE schema_version (
  version INT PRIMARY KEY,
  description VARCHAR(500),
  type VARCHAR(20),  -- DDL|DML|REPAIR
  applied_at TIMESTAMP DEFAULT NOW(),
  applied_by VARCHAR(255),
  execution_time_ms INT,
  success BOOLEAN,
  error_message TEXT
);

INSERT INTO schema_version VALUES (
  1, 'Initialize burger, schulddossier, schuld tables', 'DDL',
  NOW(), 'migration-script', 245, true, NULL
);
```

---

## Migration Timeline

### v0.1.0 (Alpha - May 2026)

```sql
-- M001_initial_schema.sql
CREATE TABLE burger (...);
CREATE TABLE schulddossier (...);
CREATE TABLE schuld (...);
CREATE TABLE schuldhulpverlener (...);
CREATE TABLE schuldhulpmaatregel (...);
CREATE TABLE document (...);
CREATE TABLE audit_log (...);

-- Update schema_version
INSERT INTO schema_version (version, description, type, applied_by)
VALUES (1, 'Initialize core tables', 'DDL', 'alembic');
```

### v0.2.0 (Beta - July 2026)

```sql
-- M002_add_dossier_hulpverlener_junction.sql
CREATE TABLE dossier_hulpverlener (...);

-- M003_add_encryption.sql
ALTER TABLE burger ALTER email TYPE BYTEA;

-- M004_add_stats_view.sql
CREATE MATERIALIZED VIEW v_dossier_stats AS ...
CREATE INDEX idx_stats_gemeente ON v_dossier_stats(gemeente_code);

-- M005_add_audit_trail_indexes.sql
CREATE INDEX idx_audit_entity ON audit_log(...);
```

### v1.0.0 (Stable - October 2026)

```sql
-- M006_deprecate_old_columns.sql
-- (No actual removal, just deprecated flag for docs)

-- M007_add_performance_indexes.sql
-- Additional indexes based on production query patterns
```

---

## Migration Patterns

### Add Column (Non-Breaking)

```sql
-- Forward
ALTER TABLE schulddossier ADD COLUMN gemeente_code VARCHAR(4);
ALTER TABLE schulddossier ADD CONSTRAINT check_gemeente CHECK (gemeente_code ~ '^\d{4}$');

-- Backward
ALTER TABLE schulddossier DROP COLUMN gemeente_code;
```

### Add Table (Non-Breaking)

```sql
-- Forward
CREATE TABLE new_feature (...);
CREATE INDEX idx_new_feature ON new_feature(...);

-- Backward
DROP TABLE new_feature;
```

### Rename Column (Breaking - Requires careful handling)

```sql
-- Forward (with backwards compat window)
ALTER TABLE schulddossier RENAME COLUMN old_name TO new_name;
-- Keep view for compatibility:
CREATE VIEW schulddossier_v1 AS 
  SELECT *, new_name AS old_name FROM schulddossier;

-- Backward
-- Not possible without manual intervention (wait until v2.0)
```

### Drop Column (Breaking - Major version)

```sql
-- Forward (v2.0)
ALTER TABLE schulddossier DROP COLUMN deprecated_column;

-- Procedure:
-- 1. v1.5: Mark column as deprecated in docs
-- 2. v1.8: Add warning in API when column accessed
-- 3. v2.0: BREAKING - Remove column
```

---

## Deployment Safety

### Dark Launch Windows

1. **Deploy** new code + migration
2. **Monitor** for issues (5-10 minutes in prod)
3. **Validate** data integrity
4. **Activate** feature flag (if needed)
5. **Cleanup** old code path (v+2)

### Rollback Strategy

Each migration has Down script:

```python
# Using Alembic
def upgrade():
    op.create_table('new_table', ...)

def downgrade():
    op.drop_table('new_table')
```

**Safety**:
- Automatic backup before major migrations
- Test rollback in staging first
- Keep downtime < 5 seconds (online DDL)

---

## Data Migration Examples

### M008: Populate gemeente_code from adres_id (v0.3.0)

```python
# Backward-compatible migration
def upgrade():
    # Add column first
    op.add_column('schulddossier', 
                   sa.Column('gemeente_code', sa.String(4), nullable=True))
    
    # Populate from adres lookup
    connection = op.get_bind()
    connection.execute("""
    UPDATE schulddossier s
    SET gemeente_code = (
        SELECT gemeente_code FROM basisregistratie_adres 
        WHERE id = (SELECT adres_id FROM burger WHERE bsn = s.burger_id)
    )
    """)
    
    # Add constraint after population
    op.alter_column('schulddossier', 'gemeente_code',
        nullable=False, existing_type=sa.String(4))

def downgrade():
    op.drop_column('schulddossier', 'gemeente_code')
```

### M009: Denormalize totaal_schuld_bedrag (v1.0.0)

```python
def upgrade():
    # Add computed column
    op.execute("""
    ALTER TABLE schulddossier
    ADD COLUMN totaal_schuld_bedrag DECIMAL(12,2)
    GENERATED ALWAYS AS (
        SELECT COALESCE(SUM(bedrag), 0)::DECIMAL(12,2)
        FROM schuld WHERE dossier_id = schulddossier.id AND status IN ('OPEN', 'BETWIST')
    ) STORED;
    """)
    
    # Create index for query optimization
    op.create_index(
        'idx_dossier_schuld_sum',
        'schulddossier',
        ['totaal_schuld_bedrag'],
        unique=False
    )

def downgrade():
    op.drop_column('schulddossier', 'totaal_schuld_bedrag')
```

---

## Testing Migrations

### Unit Test

```python
# test_migrations.py
def test_upgrade_m001():
    # Setup
    downgrade_version(0)
    
    # Execute migration
    upgrade_version(1)
    
    # Verify
    inspector = inspect(engine)
    tables = inspector.get_table_names()
    assert 'burger' in tables
    assert 'schulddossier' in tables
    
    # Verify constraints
    constraints = inspector.get_pk_constraint('burger')
    assert constraints['constrained_columns'] == ['bsn']

def test_downgrade_m001():
    upgrade_version(1)
    downgrade_version(0)
    
    inspector = inspect(engine)
    tables = inspector.get_table_names()
    assert 'burger' not in tables
```

### Data Integrity Test

```python
def test_data_consistency_after_m008():
    """Verify gemeente_code populated correctly"""
    upgrade_version(8)
    
    # Check no NULLs
    result = session.query(Schulddossier).filter_by(gemeente_code=None).count()
    assert result == 0
    
    # Spot check
    dossier = session.query(Schulddossier).first()
    burger = session.query(Burger).filter_by(bsn=dossier.burger_id).first()
    adres = session.query(Adres).filter_by(id=burger.adres_id).first()
    assert dossier.gemeente_code == adres.gemeente_code
```

---

## Deployment Checklist

- [ ] Migration tested in dev
- [ ] Data integrity verified
- [ ] Rollback script tested
- [ ] Code deployed (with feature flag if needed)
- [ ] Migration applied
- [ ] Monitoring alerts checked
- [ ] Data validation queries run
- [ ] Team notified on #deployments Slack

---

## Retention & Cleanup

### Active Period

- **Dossier ACTIEF**: No cleanup
- **Dossier GESLOTEN**: 5-7 years (legal requirement)
- **Audit logs**: 7 years (mandatory)
- **Soft-deleted records**: 90 days (GDPR right to erasure)

### Cleanup Jobs

```sql
-- M010: Add cleanup job (v1.2.0)

-- Soft-delete cleanup (after 90 days)
DELETE FROM burger WHERE deleted_at < NOW() - INTERVAL '90 days';
DELETE FROM schulddossier WHERE deleted_at < NOW() - INTERVAL '90 days';

-- Audit log archival (older than 7 years)
INSERT INTO audit_log_archive SELECT * FROM audit_log 
  WHERE timestamp < NOW() - INTERVAL '7 years';
DELETE FROM audit_log WHERE timestamp < NOW() - INTERVAL '7 years';

-- Index maintenance
REINDEX TABLE schulddossier;
VACUUM ANALYZE;
```

---

## Monitoring

### Migration Metrics

```sql
SELECT 
    version,
    description,
    execution_time_ms,
    applied_at
FROM schema_version
ORDER BY version DESC
LIMIT 10;
```

### Performance Impact Post-Migration

```sql
-- Check slow queries after migration
SELECT query, calls, mean_time FROM pg_stat_statements
WHERE query LIKE '%schulddossier%'
ORDER BY mean_time DESC LIMIT 10;
```

---

**Document version**: 1.0 | **Last update**: Mei 2026
