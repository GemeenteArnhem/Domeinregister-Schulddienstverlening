# API Endpoint Definities

Alle endpoints van het Schulddienstverlening Register API.

Base URL: `https://api.schulddienstverlening.nl/v1`

---

## Resource: Schulddossiers

### GET /schulddossiers
Lijst met schulddossiers (per huidigegebeurder of gemeente)

**Autorisatie**:
- Burger: Eigen schulddossiers only
- Hulpverlener: Assigned dossiers
- Gemeente: None (use /stats)

**Query Parameters**:
- `status`: NIEUW | ACTIEF | AFGEROND | GESLOTEN
- `skip`: Offset (default: 0)
- `limit`: Items per page (default: 20, max: 100)
- `sort`: created_at, -modified_at (ascending/descending)

**Response** (200 OK):
```json
{
  "data": [
    {
      "id": "uuid",
      "burger_id": "123456789",
      "status": "ACTIEF",
      "intake_datum": "2026-05-01",
      "totaal_schuld_bedrag": 15000.00,
      "created_at": "2026-05-26T10:00:00Z"
    }
  ],
  "meta": {
    "total": 42,
    "skip": 0,
    "limit": 20
  }
}
```

---

### POST /schulddossiers
Maak nieuw schulddossier aan (intake)

**Autorisatie**: Hulpverlener (met accreditatie)

**Body**:
```json
{
  "burger_id": "123456789",
  "intake_datum": "2026-05-26",
  "dossier_verantwoordelijke_id": "uuid",
  "beschrijving": "Aanmelding via gemeente"
}
```

**Response** (201 Created):
```json
{
  "id": "uuid",
  "burger_id": "123456789",
  "status": "NIEUW",
  "intake_datum": "2026-05-26",
  "created_at": "2026-05-26T13:20:14Z"
}
```

**Errors**:
- `400 Bad Request`: Missing required fields
- `409 Conflict`: Duplicate intake within 12 months
- `422 Unprocessable Entity`: Burger age < 16

---

### GET /schulddossiers/{dossier_id}
Detailsbladpagina dossier

**Response** (200 OK):
```json
{
  "id": "uuid",
  "burger_id": "123456789",
  "status": "ACTIEF",
  "intake_datum": "2026-05-01",
  "dossier_verantwoordelijke_id": "uuid-hulpverlener",
  "totaal_schuld_bedrag": 15000.00,
  "schulden": [
    {
      "id": "uuid",
      "schuldeiser_naam": "ING Bank",
      "bedrag": 10000.00,
      "status": "OPEN"
    }
  ],
  "maatregelen": [
    {
      "id": "uuid",
      "type": "SCHULDBEMIDDELING",
      "status": "ACTIEF",
      "start_datum": "2026-05-15"
    }
  ]
}
```

---

### PUT /schulddossiers/{dossier_id}
Update dossier status of beschrijving

**Authorization**: Verantwoordelijke hulpverlener only

**Body**:
```json
{
  "status": "AFGEROND",
  "beschrijving": "Alle schulden afgelost!"
}
```

**Response** (200 OK): Updated dossier

**Errors**:
- `403 Forbidden`: Geen authorization
- `409 Conflict`: Invalid state transition

---

## Resource: Schulden

### POST /schulddossiers/{dossier_id}/schulden
Voeg schuld toe aan dossier

**Body**:
```json
{
  "schuldeiser_naam": "ABN AMRO",
  "soort_schuld": "CONSUMENTENKREDIET",
  "bedrag": 8500.00,
  "status": "OPEN"
}
```

**Response** (201 Created): New schuld object

---

### PUT /schulddossiers/{dossier_id}/schulden/{schuld_id}
Update schuld status of bedrag

**Body**:
```json
{
  "status": "BETAALD",
  "bedrag_afgelost": 8500.00
}
```

**Immutability**: BETAALD schulden kunnen niet meer wijzigen

---

### DELETE /schulddossiers/{dossier_id}/schulden/{schuld_id}
Verwijder schuld (soft delete)

**Response** (204 No Content)

---

## Resource: Schuldhulpmaatregelen

### POST /schulddossiers/{dossier_id}/maatregelen
Voeg maatregel toe

**Body**:
```json
{
  "type": "SCHULDBEMIDDELING",
  "schuldhulpverlener_id": "uuid",
  "start_datum": "2026-05-15",
  "doelstelling": "Schulden reduceren tot EUR 5000"
}
```

**Response** (201 Created)

---

### PUT /schulddossiers/{dossier_id}/maatregelen/{maatregel_id}
Update maatregel status

**Body**:
```json
{
  "status": "AFGEROND",
  "einddatum_werkelijk": "2026-05-26"
}
```

---

## Resource: Documenten

### POST /schulddossiers/{dossier_id}/documenten
Upload bijlage

**Multipart Form Data**:
- `file`: Binary file (max 50MB)
- `soort_document`: SCHULDOVERZICHT | OVEREENKOMST | etc.
- `zichtbaar_voor_burger`: true | false

**Response** (201 Created):
```json
{
  "id": "uuid",
  "bestandsnaam": "schuldoverzicht.pdf",
  "upload_datum": "2026-05-26T13:20:14Z"
}
```

---

### GET /schulddossiers/{dossier_id}/documenten
Lijst documentenvan dossier

**Query**: `?zichtbaar_voor_burger=true` (filter)

---

### DELETE /schulddossiers/{dossier_id}/documenten/{document_id}
Verwijder document

**Response** (204 No Content)

---

## Resource: Schuldhulpverleners

### GET /schuldhulpverleners
Lijst hulpverleners (publiek gefiltreed)

**Response**:
```json
{
  "data": [
    {
      "id": "uuid",
      "organisatie_naam": "Schuldhulp Amsterdam",
      "specialisaties": ["SCHULDBEMIDDELING"],
      "accreditatie_gemeenten": ["0363"],
      "bereik": "GEMEENTE"
    }
  ]
}
```

---

### GET /schuldhulpverleners/{hulpverlener_id}
Details hulpverlener

---

## Statistics/Reporting

### GET /stats/dossiers
Geanonimiseerde statistieken (enkel gemeente)

**Query**:
- `gemeente_code`: Gemeente filter (required voor niet-admin)
- `period`: week | month | quarter

**Response** (200 OK):
```json
{
  "totaal_dossiers": 154,
  "percentage_actief": 62.3,
  "percentage_afgerond": 28.5,
  "gemiddelde_schuldbedrag": 12450.00,
  "schulddossiers_per_soort": {
    "SCHULDBEMIDDELING": 95,
    "BUDGETBEHEER": 45,
    "SPAARPLAN": 14
  }
}
```

---

## Error Responses

### 400 Bad Request
```json
{
  "error": "BAD_REQUEST",
  "message": "Missing required field: doelstelling"
}
```

### 401 Unauthorized
```json
{
  "error": "UNAUTHORIZED",
  "message": "Invalid or expired JWT token"
}
```

### 403 Forbidden
```json
{
  "error": "FORBIDDEN",
  "message": "Insufficient permissions to access dossier"
}
```

### 404 Not Found
```json
{
  "error": "NOT_FOUND",
  "message": "Schulddossier with id {id} not found"
}
```

### 409 Conflict
```json
{
  "error": "CONFLICT",
  "message": "Cannot transition from GESLOTEN to ACTIEF"
}
```

### 422 Unprocessable Entity
```json
{
  "error": "VALIDATION_ERROR",
  "details": [
    {
      "field": "bedrag",
      "message": "Must be > 0"
    }
  ]
}
```

### 429 Too Many Requests
```json
{
  "error": "RATE_LIMIT_EXCEEDED",
  "retry_after": 60
}
```

### 500 Internal Server Error
```json
{
  "error": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

---

## Response Headers

Alle responses incluyen:
- `X-Request-ID`: Unique request identifier (for logging)
- `X-RateLimit-Limit`: Rate limit (e.g., 1000)
- `X-RateLimit-Remaining`: Remaining requests
- `X-RateLimit-Reset`: Reset time (Unix timestamp)
- `X-API-Version`: Current API version (v1)

---

## Pagination

Default: Offset/limit paginatie

```
GET /schulddossiers?skip=20&limit=20
```

Response meta:
```json
{
  "meta": {
    "total": 154,
    "skip": 20,
    "limit": 20,
    "has_next": true,
    "has_prev": true
  }
}
```

---

## Sorting

`sort` query parameter:

- `sort=created_at` (ascending)
- `sort=-modified_at` (descending, dash prefix)
- `sort=totaal_schuld_bedrag,-status`

---

**Document version**: 1.0 | **Last update**: Mei 2026
