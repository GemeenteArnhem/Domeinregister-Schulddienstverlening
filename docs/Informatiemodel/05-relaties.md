# Relaties tussen Objecttypen

Dit document beschrijft kardinaliteiten, foreign keys, en join patterns.

---

## 1-to-N Relaties

### Burger ──1─ OWNS ─N─→ Schulddossier

**Kardinaliteit**: 1:N (één burger → veel dossiers, maar historisch!)

**Foreign Key**:
```sql
ALTER TABLE schulddossier 
ADD CONSTRAINT fk_burger_dossier 
  FOREIGN KEY (burger_id) REFERENCES burger(bsn) 
  ON DELETE RESTRICT  -- Burger kan niet verwijderd met aktieve dossiers
  ON UPDATE CASCADE;
```

**Multipliciteit**:
- 0..* (Burger kan 0 dossiers hebben, of 1+)
- Praktijk: Burger krijgt 1 aktief dossier, rest gesloten (archief)

**Query Examples**:
```sql
-- Alle dossiers voor burger
SELECT * FROM schulddossier WHERE burger_id = '123456789';

-- Dossiers gegroepeerd op status
SELECT status, COUNT(*) FROM schulddossier 
WHERE burger_id = '123456789' GROUP BY status;
```

---

### Schulddossier ──1─ CONTAINS ─N─→ Schuld

**Kardinaliteit**: 1:N (één dossier → 1+ schulden)

**Constraint**: Dossier MOET ≥1 schuld hebben (BR-011)

```sql
ALTER TABLE schuld 
ADD CONSTRAINT fk_dossier_schuld 
  FOREIGN KEY (dossier_id) REFERENCES schulddossier(id) 
  ON DELETE CASCADE;  -- Schulden verwijderd met dossier
```

**Aggregatie**:
```sql
-- Totaal schuldbedrag per dossier
SELECT dossier_id, SUM(bedrag) as totaal 
FROM schuld 
WHERE status = 'OPEN' 
GROUP BY dossier_id;
```

---

### Schulddossier ──1─ HAS ─N─→ Schuldhulpmaatregel

**Kardinaliteit**: 1:N (één dossier → 0..* maatregelen, parallelle OK)

```sql
ALTER TABLE schuldhulpmaatregel 
ADD CONSTRAINT fk_dossier_maatregel 
  FOREIGN KEY (dossier_id) REFERENCES schulddossier(id) 
  ON DELETE CASCADE;
```

**Parallelle Maatregelen**:
```
Dossier [ACTIEF]
├── Maatregel 1: SCHULDBEMIDDELING [ACTIEF]
├── Maatregel 2: BUDGETBEHEER [ACTIEF]
└── Maatregel 3: SPAARPLAN [OPGESCHORT]
```

---

### Schuldhulpmaatregel ──1─ STORES ─N─→ Document

**Kardinaliteit**: 1:N (maatregel → ondersteunende docs)

```sql
ALTER TABLE document 
ADD CONSTRAINT fk_maatregel_document 
  FOREIGN KEY (maatregel_id) REFERENCES schuldhulpmaatregel(id) 
  ON DELETE SET NULL;  -- Doc blijft, referentie verwijderd
```

**Voorbeeld**:
- Schuldbemiddeling → Overeenkomst PDF
- Schuldbemiddeling → Schuldoverzicht gecompileerd
- Budgetbeheer → Maandelijks spaarplan

---

## M-to-N Relaties

### Schuldhulpverlener ──M─ WORKS_ON ─N─← Schulddossier

**Kardinaliteit**: M:N (veel hulpverleners worked veel dossiers)

**Junction Table**:
```sql
CREATE TABLE dossier_hulpverlener (
  dossier_id UUID REFERENCES schulddossier(id) ON DELETE CASCADE,
  hulpverlener_id UUID REFERENCES schuldhulpverlener(id) ON DELETE CASCADE,
  rol ENUM ('VERANTWOORDELIJKE', 'BETROKKEN', 'OBSERVER'),
  start_datum DATE,
  eind_datum DATE NULL,
  PRIMARY KEY (dossier_id, hulpverlener_id)
);
```

**Roles**:
- VERANTWOORDELIJKE: Primary contact, full edit
- BETROKKEN: Secondary, limited edit
- OBSERVER: Read-only (audit/compliance)

**Query Examples**:
```sql
-- Alle hulpverleners voor dossier
SELECT h.* FROM schuldhulpverlener h
JOIN dossier_hulpverlener dh ON h.id = dh.hulpverlener_id
WHERE dh.dossier_id = ?;

-- Alle dossiers voor hulpverlener
SELECT d.* FROM schulddossier d
JOIN dossier_hulpverlener dh ON d.id = dh.dossier_id
WHERE dh.hulpverlener_id = ? AND dh.rol IN ('VERANTWOORDELIJKE', 'BETROKKEN');
```

---

### Schuldeiser ──M─ HAS_VORDERINGEN ─N─← Schuld

**Kardinaliteit**: M:N (one schuldeiser → many schulden, one schuld → one schuldeiser voor v1)

**Note**: In v1 simplistic: One schuld = one schuldeiser.
Future: Allow joint debtors.

```sql
-- V1: Simplistic
VIEW schuldenaar_schuldbedrag AS
SELECT schuldeiser_naam, COUNT(*) as aantal, SUM(bedrag) as totaal
FROM schuld
WHERE status = 'OPEN'
GROUP BY schuldeiser_naam;
```

---

## Referential Integrity Patterns

### Cascade vs. Restrict

| Path | Delete Behavior | Reason |
|------|-----------------|--------|
| Burger DELETE | **RESTRICT** | Never delete burgers with active dossiers |
| Dossier DELETE | **RESTRICT** (unless soft) | Keep audit trail |
| Schuld DELETE | **CASCADE** | Schulden tied to dossier |
| Document DELETE | **CASCADE** | Docs tied to maatregel |
| Schuldhulpverlener DELETE | **RESTRICT** | Never delete, just mark inactive |

---

## Query Patterns

### List all schulden for dossier

```sql
SELECT s.* FROM schuld s
WHERE s.dossier_id = $1
ORDER BY s.bedrag DESC;
```

### Totaal schuld per gemeente

```sql
SELECT DISTINCT(h.accreditatie_gemeenten),
       COUNT(DISTINCT d.id) as dossiers,
       SUM(s.bedrag) as totaal_schuld
FROM schulddossier d
JOIN schuld s ON d.id = s.dossier_id
JOIN schuldhulpverlener h ON d.dossier_verantwoordelijke_id = h.id
WHERE d.status = 'ACTIEF'
  AND s.status IN ('OPEN', 'BETWIST')
GROUP BY h.accreditatie_gemeenten;
```

### Hulpverleners per dossier met rollen

```sql
SELECT h.organisatie_naam, dh.rol, dh.start_datum
FROM dossier_hulpverlener dh
JOIN schuldhulpverlener h ON dh.hulpverlener_id = h.id
WHERE dh.dossier_id = $1
  AND (dh.eind_datum IS NULL OR dh.eind_datum > NOW());
```

---

## ER Diagram

```
┌─────────────┐
│    Burger   │
│ PK: bsn     │
└──────┬──────┘
       │ 1:N
       │
       ▼
┌──────────────────────┐
│ Schulddossier        │
│ PK: id               │
│ FK: burger_id        │◄─ 1
└──────┬───────┬───────┘
       │       │
    1:N│       │1:N
       │       │
       ▼       ▼
┌──────────┐  ┌────────────────────┐
│  Schuld  │  │Schuldhulpmaatregel │
│ FK: dos..│  │ PK: id             │
└──────────┘  │ FK: dos, hulpverl..│
              └─────────┬──────────┘
                        │
                     1:N│
                        ▼
              ┌──────────────────┐
              │   Document       │
              │ PK: id           │
              │ FK: maatregel_id │
              └──────────────────┘

(M:N omitted for clarity: dossier_hulpverlener junction table)
```

---

## Performance Considerations

### Indexes

```sql
-- Speed up burger dossier list
CREATE INDEX idx_dossier_burger ON schulddossier(burger_id, status);

-- Speed up schuld totals
CREATE INDEX idx_schuld_dossier_status ON schuld(dossier_id, status);

-- Speed up hulpverlener dossier queries
CREATE INDEX idx_dh_hulpverl_rol ON dossier_hulpverlener(hulpverlener_id, rol);

-- Audit trail queries
CREATE INDEX idx_audit_entity ON audit_log(entity_type, entity_id, timestamp DESC);
```

### Denormalization (Optional)

Cache `schulddossier.totaal_schuld_bedrag`:
- Update trigger on schuld INSERT/UPDATE/DELETE
- TTL 15 minutes if no trigger
- Verify daily consistency check

---

**Document version**: 1.0 | **Last update**: Mei 2026
