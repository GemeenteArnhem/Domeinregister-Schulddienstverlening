# Database Schema

PostgreSQL 15+ design met performance, security, compliance in mind.

---

## Connection & Configuration

```sql
-- Connection string (with encryption)
postgresql://user:password@db.schulddienstverlening.nl:5432/schulddienstverlening
  ?sslmode=require
  &application_name=schulddienstverlening-app
  &connect_timeout=5
```

**Parameters**:
- `sslmode=require`: Enforce TLS 1.3
- `statement_timeout=300000`: 5 min query limit
- `idle_in_transaction_session_timeout=60000`: 1 min idle timeout

---

## Tables

### burger

**Purpose**: Central person registry (immutable)

```sql
CREATE TABLE burger (
  bsn VARCHAR(9) PRIMARY KEY,  -- Unique identifier
  voornamen VARCHAR(200) NOT NULL,
  achternaam VARCHAR(200) NOT NULL,
  geboortedatum DATE NOT NULL,
  geslacht CHAR(1),  -- M/V/X
  adres_id UUID NOT NULL REFERENCES basisregistratie_adres(id),
  email VARCHAR(255),
  telefoonnummer VARCHAR(20),
  preferred_contact VARCHAR(20),  -- EMAIL|TELEFOON|POST
  opmerking_contactgegevens TEXT,
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP,  -- Soft delete
  
  CHECK (LENGTH(bsn) = 9),
  CHECK (geboortedatum <= CURRENT_DATE),
  CHECK (preferred_contact IN ('EMAIL', 'TELEFOON', 'POST') OR preferred_contact IS NULL)
);

-- Indexes
CREATE UNIQUE INDEX idx_burger_bsn ON burger(bsn);
CREATE INDEX idx_burger_deleted_at ON burger(deleted_at);
```

---

### schulddossier

**Purpose**: Main aggregate root

```sql
CREATE TABLE schulddossier (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  burger_id VARCHAR(9) NOT NULL REFERENCES burger(bsn),
  status VARCHAR(20) NOT NULL DEFAULT 'NIEUW',
  intake_datum DATE NOT NULL,
  intake_hulpverlener_id UUID NOT NULL REFERENCES schuldhulpverlener(id),
  dossier_verantwoordelijke_id UUID NOT NULL REFERENCES schuldhulpverlener(id),
  
  -- Denormalized totals (cached)
  totaal_schuld_bedrag DECIMAL(12, 2) GENERATED ALWAYS AS (
    SELECT COALESCE(SUM(bedrag), 0)::DECIMAL(12, 2)
    FROM schuld WHERE dossier_id = schulddossier.id AND status IN ('OPEN', 'BETWIST')
  ) STORED,
  
  beschrijving TEXT,
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  archived_at TIMESTAMP,
  
  CHECK (status IN ('NIEUW', 'ACTIEF', 'AFGEROND', 'GESLOTEN')),
  CONSTRAINT unique_active_burger_intake 
    UNIQUE (burger_id, intake_datum)  -- Prevents duplicate intakes
);

-- Indexes
CREATE INDEX idx_dossier_burger ON schulddossier(burger_id, status);
CREATE INDEX idx_dossier_status ON schulddossier(status);
CREATE INDEX idx_dossier_created ON schulddossier(created_at DESC);
```

---

### schuld

**Purpose**: Individual debts within dossier

```sql
CREATE TABLE schuld (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dossier_id UUID NOT NULL REFERENCES schulddossier(id),
  
  schuldeiser_naam VARCHAR(255) NOT NULL,
  schuldeiser_id UUID REFERENCES schuldeiser(id),  -- External registry
  
  bedrag DECIMAL(12, 2) NOT NULL CHECK (bedrag > 0),
  bedrag_afgelost DECIMAL(12, 2) DEFAULT 0,
  soort_schuld VARCHAR(50) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'OPEN',
  
  aanschrijfdatum DATE,
  vervaldatum_origineel DATE,
  oorzaak_ontstaan TEXT,
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP,
  
  CHECK (status IN ('OPEN', 'BETWIST', 'VOORBIJ', 'INGEVORDERD', 'KWIJTGESCHOLDEN', 'BETAALD')),
  CHECK (bedrag_afgelost <= bedrag)
);

-- Indexes
CREATE INDEX idx_schuld_dossier_status ON schuld(dossier_id, status);
CREATE INDEX idx_schuld_status ON schuld(status);
```

---

### schuldhulpverlener

**Purpose**: Service provider registry

```sql
CREATE TABLE schuldhulpverlener (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organisatie_naam VARCHAR(255) NOT NULL,
  kvk_nummer VARCHAR(8) NOT NULL UNIQUE,
  website VARCHAR(500),
  email VARCHAR(255) NOT NULL,
  telefoonnummer VARCHAR(20) NOT NULL,
  adres_id UUID NOT NULL REFERENCES basisregistratie_adres(id),
  
  specialisaties VARCHAR[] NOT NULL,  -- PostgreSQL array
  bereik VARCHAR(50),  -- LANDELIJK|REGIO|GEMEENTE
  status VARCHAR(20) DEFAULT 'ACTIEF',  -- ACTIEF|PAUZE|INACTIEF
  
  accreditatie_datum DATE,
  accreditatie_vervaldatum DATE,
  accreditatie_gemeenten VARCHAR[] NOT NULL,  -- Gemeente codes
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP,
  
  CHECK (status IN ('ACTIEF', 'PAUZE', 'INACTIEF'))
);

-- Indexes
CREATE INDEX idx_hulpverlener_status ON schuldhulpverlener(status);
CREATE INDEX idx_hulpverlener_specialisaties ON schuldhulpverlener USING GIN(specialisaties);
```

---

### schuldhulpmaatregel

**Purpose**: Interventions/measures

```sql
CREATE TABLE schuldhulpmaatregel (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dossier_id UUID NOT NULL REFERENCES schulddossier(id),
  schuldhulpverlener_id UUID NOT NULL REFERENCES schuldhulpverlener(id),
  
  type VARCHAR(50) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'GESLOTEN',
  start_datum DATE NOT NULL,
  einddatum_gepland DATE,
  einddatum_werkelijk DATE,
  
  beëindiging_reden VARCHAR(50),  -- AFGEROND_SUCCESVOL|VOORTIJDIG_BURGER|...
  
  doelstelling TEXT NOT NULL,
  beschrijving TEXT,
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  
  CHECK (status IN ('GESLOTEN', 'ACTIEF', 'OPGESCHORT', 'AFGEROND')),
  CHECK (einddatum_gepland IS NULL OR einddatum_gepland >= start_datum),
  CHECK (einddatum_werkelijk IS NULL OR einddatum_werkelijk >= start_datum)
);

-- Indexes
CREATE INDEX idx_maatregel_dossier_status ON schuldhulpmaatregel(dossier_id, status);
CREATE INDEX idx_maatregel_hulpverlener ON schuldhulpmaatregel(schuldhulpverlener_id);
```

---

### dossier_hulpverlener (Junction Table)

**Purpose**: M:N relationship with roles

```sql
CREATE TABLE dossier_hulpverlener (
  dossier_id UUID NOT NULL REFERENCES schulddossier(id) ON DELETE CASCADE,
  hulpverlener_id UUID NOT NULL REFERENCES schuldhulpverlener(id) ON DELETE CASCADE,
  rol VARCHAR(50) NOT NULL,  -- VERANTWOORDELIJKE|BETROKKEN|OBSERVER
  
  start_datum DATE DEFAULT CURRENT_DATE,
  eind_datum DATE,
  
  PRIMARY KEY (dossier_id, hulpverlener_id),
  CHECK (rol IN ('VERANTWOORDELIJKE', 'BETROKKEN', 'OBSERVER')),
  CHECK (eind_datum IS NULL OR eind_datum >= start_datum)
);

CREATE INDEX idx_dh_hulpverlener_rol ON dossier_hulpverlener(hulpverlener_id, rol);
```

---

### document

**Purpose**: Attached files (S3 + metadata)

```sql
CREATE TABLE document (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dossier_id UUID NOT NULL REFERENCES schulddossier(id),
  maatregel_id UUID REFERENCES schuldhulpmaatregel(id),  -- Optional
  
  soort_document VARCHAR(50) NOT NULL,
  bestandsnaam VARCHAR(255) NOT NULL,  -- Original filename
  file_path VARCHAR(500) NOT NULL,  -- S3://bucket/key (encrypted)
  file_size INTEGER NOT NULL,  -- Bytes
  mime_type VARCHAR(100) NOT NULL,
  
  upload_datum TIMESTAMP DEFAULT NOW(),
  uploader_id UUID NOT NULL REFERENCES schuldhulpverlener(id),
  
  zichtbaar_voor_burger BOOLEAN DEFAULT FALSE,
  
  created_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP,
  
  CHECK (file_size > 0),
  CHECK (soort_document IN ('SCHULDOVERZICHT', 'OVEREENKOMST', 'INKOMSTEN', 'BETALINGSPLAN', 'JURIDISCH', 'OVERIGE'))
);

-- Indexes
CREATE INDEX idx_doc_dossier ON document(dossier_id);
CREATE INDEX idx_doc_maatregel ON document(maatregel_id);
```

---

### audit_log (Immutable)

**Purpose**: Immutable audit trail for GDPR, BIO2

```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp TIMESTAMP DEFAULT NOW(),  -- UTC, immutable
  
  actor_type VARCHAR(50),  -- USER|SYSTEM|ADMIN
  actor_id UUID,  -- User ID
  
  entity_type VARCHAR(50) NOT NULL,  -- DOSSIER|SCHULD|MAATREGEL|DOCUMENT
  entity_id UUID NOT NULL,
  resource_type VARCHAR(50),  -- For future use
  
  action VARCHAR(50) NOT NULL,  -- CREATE|UPDATE|DELETE|VIEW|EXPORT
  old_value JSONB,  -- Before state (for sensitive fields)
  new_value JSONB,  -- After state (masked)
  
  ip_address INET,  -- Client IP
  user_agent TEXT,  -- Browser/app info
  
  request_id UUID,  -- Correlation ID
  
  CHECK (action IN ('CREATE', 'UPDATE', 'DELETE', 'VIEW', 'EXPORT', 'LOGIN', 'LOGOUT'))
);

-- Indexes
CREATE INDEX idx_audit_entity ON audit_log(entity_type, entity_id, timestamp DESC);
CREATE INDEX idx_audit_actor ON audit_log(actor_id, timestamp DESC);
CREATE INDEX idx_audit_action ON audit_log(action, timestamp DESC);
CREATE INDEX idx_audit_timestamp_retention ON audit_log(timestamp) WHERE timestamp > NOW() - INTERVAL '7 years';
```

---

## Performance Optimizations

### Query Optimization

1. **Denormalized `totaal_schuld_bedrag`**: GENERATED ALWAYS STORED
   - Avoids repeated SUM aggregations
   - Updated automatically on schuld changes
   - Trade-off: Extra storage, consistency guaranteed

2. **Partial Indexes**: Only active dossiers

   ```sql
   CREATE INDEX idx_dossier_actief ON schulddossier(burger_id) 
   WHERE status != 'GESLOTEN';
   ```

3. **Covering Indexes**: Include columns for index-only scans

   ```sql
   CREATE INDEX idx_schuld_list ON schuld(dossier_id, status)
   INCLUDE (bedrag, schuldeiser_naam);  -- PG v14+
   ```

### Scaling Strategies

1. **Partitioning** (future): By `gemeente_code` or `creation_date`
2. **Read Replicas**: Statistics queries → replica
3. **Caching**: Redis for `totaal_schuld_bedrag`, top schuldeisers
4. **Materialized Views**: Daily stats snapshots

---

## Security

### Encryption

```sql
-- Columns encrypted via pgcrypto (application-level or TDE)
ALTER TABLE burger ALTER email TYPE BYTEA USING pgp_sym_encrypt(email, 'key');
```

### Row-Level Security (RLS)

```sql
-- Burger sees only own dossier
CREATE POLICY burger_sees_own ON schulddossier
  USING (burger_id = current_setting('jwt.claims.sub'));

-- Hulpverlener sees assigned dossiers
CREATE POLICY hulpverlener_sees_assigned ON schulddossier
  USING (EXISTS (
    SELECT 1 FROM dossier_hulpverlener 
    WHERE dossier_id = schulddossier.id 
    AND hulpverlener_id = current_setting('jwt.claims.hulpverlener_id')
  ));
```

---

## Versioning & Migration

See [migrations.md](./migrations.md) for versioning strategy and migration scripts.

---

**Document version**: 1.1 | **Last update**: Mei 2026
