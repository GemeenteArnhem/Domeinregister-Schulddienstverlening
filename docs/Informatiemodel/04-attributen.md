# Attributen per Objecttype

Gedetailleerde beschrijving van alle attributen met types, validaties, en voorbeelden.

---

## Burger Attributen

| Attribuut | Type | Min | Max | Verplicht | Pattern | Voorbeeld |
|-----------|------|-----|-----|-----------|---------|-----------|
| bsn | String | 9 | 9 | Ja | \d{9} | "123456789" |
| voornamen | String | 1 | 200 | Ja | [a-zA-Z\s\-'] | "Jan Willem" |
| achternaam | String | 1 | 200 | Ja | [a-zA-Z\s\-'\.]  | "de Vries" |
| geboortedatum | Date | - | - | Ja | YYYY-MM-DD | "1985-03-15" |
| geslacht | Enum | - | - | Nee | M/V/X | "M" |
| email | Email | 5 | 255 | Nee | RFC 5322 | "jan@example.nl" |
| telefoonnummer | String | 10 | 20 | Nee | ^\+31\|0[0-9]{1,3}[0-9]{6,8}$ | "+31612345678" |

**Validaties**:
- BSN: 11-proef (checksum algoritme)
- Email: DNS lookup (domain Must exist)
- Leeftijd: ≥ 16 jaar
- Telefoonnummer: NL/BE/DE prefix allowed

---

## Schulddossier Attributen

| Attribuut | Type | Verplicht | Opmerking |
|-----------|------|-----------|-----------|
| id | UUID v4 | Ja | Auto-generated |
| burger_id | String(9) | Ja | FK naar Burger.bsn |
| status | Enum | Ja | [NIEUW, ACTIEF, AFGEROND, GESLOTEN] |
| intake_datum | Date | Ja | ISO 8601 |
| dossier_verantwoordelijke_id | UUID | Ja | FK naar Schuldhulpverlener |
| totaal_schuld_bedrag | Decimal(12,2) | Ja | Euro, > 0 |
| beschrijving | Text | Nee | Max 5000 chars |

**Validaties**:
- Status transition rules (BR-010)
- Burger moet ACTIEF zijn (geen deleted_at)

---

## Schuld Attributen

| Attribuut | Type | Min | Max | Verplicht | Default | Opmerking |
|-----------|------|-----|-----|-----------|---------|-----------|
| id | UUID | - | - | Ja | UUID v4 | Auto-generated |
| dossier_id | UUID | - | - | Ja | - | FK naar Schulddossier |
| schuldeiser_naam | String | 2 | 255 | Ja | - | Naam schuldeiser org |
| bedrag | Decimal(12,2) | 1.00 | 999999999.99 | Ja | - | Euro, 2 decimals |
| bedrag_afgelost | Decimal(12,2) | 0 | bedrag | Nee | 0.00 | Afbetaalde bedrag |
| soort_schuld | Enum | - | - | Ja | - | Zie classificatie |
| status | Enum | - | - | Ja | OPEN | Schuld lifecycle |
| vervaldatum_origineel | Date | - | - | Nee | - | ISO 8601 |

**Enums Soort Schuld**:
```
HYPOTHEEK
CONSUMENTENKREDIET
BELASTING
HUUR_ACHTERSTAND
ZIEKENHUIS_MEDISCH
ADVOCAAT_JURIDISCH
INCASSO
VOORMALIGE_WERKGEVER
ENERGIE_WATER
OVERIGE
```

---

## Schuldhulpverlener Attributen

| Attribuut | Type | Verplicht | Validatie | Voorbeeld |
|-----------|------|-----------|-----------|-----------|
| id | UUID | Ja | Auto | UUID |
| organisatie_naam | String | Ja | Max 255 | "Schuldhulp Amsterdam" |
| kvk_nummer | String(8) | Ja | NL KvK format | "34123456" |
| email | Email | Ja | RFC 5322 | "info@schuldhulp.nl" |
| telefoonnummer | String | Ja | NL format | "+31612345678" |
| specialisaties | JSON Array | Ja | Min 1 item | ["SCHULDBEMIDDELING", "BUDGETBEHEER"] |
| bereik | Enum | Ja | LANDELIJK / REGIO / GEMEENTE | "GEMEENTE" |

**Specialisaties Enum**:
- SCHULDBEMIDDELING (negotiatie met schuldeisers)
- BUDGETBEHEER (beheer burger gefinancierde)
- SPAARPLAN (structured saving)
- INSOLVENTIETRAJECT (bankruptcy / SSR)
- JURIDISCHE_BEGELEIDING (legal support)
- MAATSCHAPPELIJK_WERK (social work)
- KREDIETREHABILATIE (credit restoration)

---

## Schuldhulpmaatregel Attributen

| Attribuut | Type | Verplicht | Opmerking |
|-----------|------|-----------|-----------|
| id | UUID | Ja | PK |
| dossier_id | UUID | Ja | FK |
| schuldhulpverlener_id | UUID | Ja | FK |
| type | Enum | Ja | [SCHULDBEMIDDELING, BUDGETBEHEER, SPAARPLAN, etc.] |
| status | Enum | Ja | [GESLOTEN, ACTIEF, OPGESCHORT, AFGEROND] |
| start_datum | Date | Ja | ISO 8601, ≤ TODAY |
| einddatum_gepland | Date | Nee | ≥ start_datum (if set) |
| einddatum_werkelijk | Date | Nee | Observed end (nullable until actual) |
| doelstelling | Text | Ja | Max 1000 chars, not null |
| beschrijving | Text | Nee | Max 5000 chars |

**Type Enum**:
- SCHULDBEMIDDELING
- BUDGETBEHEER
- SPAARPLAN
- INSOLVENTIETRAJECT
- JURIDISCHE_BEGELEIDING
- MAATSCHAPPELIJK_WERK

---

## Document Attributen

| Attribuut | Type | Verplicht | Opmerking |
|-----------|------|-----------|-----------|
| id | UUID | Ja | PK |
| dossier_id | UUID | Ja | FK |
| soort_document | Enum | Ja | [SCHULDOVERZICHT, OVEREENKOMST, INKOMSTEN, etc.] |
| bestandsnaam | String | Ja | Original filename (GDPR) |
| file_path | String | Ja | S3 path (encrypted) |
| file_size | Integer | Ja | Bytes (max 50MB) |
| mime_type | String | Ja | application/pdf, image/jpeg, etc. |
| upload_datum | DateTime | Ja | UTC timezone |
| zichtbaar_voor_burger | Boolean | Ja | Default false (for sensitive docs) |

**Soort Document Enum**:
- SCHULDOVERZICHT
- OVEREENKOMST
- INKOMSTEN_VERKLARING
- BETALINGSPLAN
- RECHTSDOCUMENT
- MEDISCHE_VERKLARING
- WERKGEVERVERKLARING
- OVERIGE

---

## Audit Trail Attributen

| Attribuut | Type | Opmerking |
|-----------|------|-----------|
| id | UUID | PK |
| timestamp | DateTime | UTC |
| actor_type | Enum | USER / SYSTEM |
| actor_id | UUID | User or System process |
| entity_type | Enum | DOSSIER / SCHULD / MAATREGEL / DOCUMENT |
| entity_id | UUID | Resource being acted on |
| action | Enum | CREATE / UPDATE / DELETE / VIEW / EXPORT |
| old_value | JSON | Previous state (nullable) |
| new_value | JSON | New state (nullable) |
| ip_address | INET | Client IP (for security) |
| user_agent | String | Browser/client info |

---

## Validation Rules Summary

### Numeric
- Bedragen: Positive, max 999999999.99 EUR
- BSN: Valid 11-digit checksum
- KvK: 8 digits, Dutch chamber of commerce

### String
- Email: RFC 5322 compliant, DNS verified
- URLs: https:// only, max 2000 chars
- Telefoonnummer: E.164 format (+31612345678)

### Date/Time
- Format: ISO 8601 (YYYY-MM-DD)
- Timezone: Always UTC
- No future dates (booking/audit)

### Enum
- Case-sensitive (ACTIEF, not actief)
- Whitelist validation (no arbitrary values)

---

## Field Encryption (at rest)

**Encrypted fields**:
- bsn (Burger)
- email (Burger, Schuldhulpverlener)
- telefoonnummer (all)
- iban (future payments)

**Method**: AES-256, keys in HSM

---

**Document version**: 1.0 | **Last update**: Mei 2026
