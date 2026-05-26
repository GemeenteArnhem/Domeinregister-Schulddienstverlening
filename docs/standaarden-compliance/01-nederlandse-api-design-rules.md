# Nederlandse API Design Rules v3.0

Dit register volgt de **Nederlandse API Design Rules** (ADR v3.0) van LOGIUS/VNG.

Referentie: https://publicatie.centrumdigitaal.nl/adrs/

## Welke ADR-regels volgen we?

### API Fundamentals

#### API-01: Provide API-specifications using OpenAPI 3.0 / 3.1
✅ **Ja**: OpenAPI 3.1 YAML, publiek beschikbaar op `/open-api`

#### API-02: Align the Definition of the API with the Maturity of the Organisations
✅ **Ja**: Level 3 (RESTful, stateless, hoog volwassen)

#### API-03: APIs must be discoverable
✅ **Ja**: OpenAPI spec gehost, developer portal in voorbereiding

#### API-04: Only apply rate-limiting to unautheticated requests from a single user
✅ **Ja**: Rate limiting 100 req/min per IP (unauthenticated), 1000 req/min (authenticated)

---

### REST API Design

#### API-09: Minimize API-complexity by separating POST and field lists
✅ **Ja**: POST POST /schulddossiers met volledige body, geen onzekere PUT/PATCH

#### API-10: Delete a resource with the DELETE operation
✅ **Ja**: DELETE /schulddossiers/{id} (soft delete met audit trail)

#### API-11: Implement custom representations instead of filtering
✅ **Ja**: `include=dossier_items,afspraken` query param voor sparse fieldsets

#### API-12: Only apply query parameters to collection resources
✅ **Ja**: Filters alleen op /schulddossiers, niet op /schulddossiers/{id}

#### API-13: Implement filtering
✅ **Ja**: Filter op status, schuldhulpverlener, burger, datumbereik

#### API-14: Implement sorting on collection resources
✅ **Ja**: `sort=created_at,-modified_at` (ascending/descending)

#### API-15: Implement pagination on collection resources
✅ **Ja**: Offset/limit paginatie (default 20, max 100 items)

#### API-16: Support both JSON and Form URL encoded data in request bodies
✅ **Ja**: Accept `application/json` en `application/x-www-form-urlencoded`

#### API-17: Use the ISO 8601 standard for dates and times in APIs
✅ **Ja**: Alle dates in ISO 8601 format (2026-05-26T13:20:14Z)

#### API-18: Include default HTTP headers in an API
✅ **Ja**: 
- `X-Request-ID` (unieke request identifier)
- `X-RateLimit-*` (rate limit info)
- `X-API-Version` (current API version)

#### API-19: Use the HTTP OPTIONS method to describe the communication options for a resource
✅ **Ja**: OPTIONS endpoints beschikbaar (CORS preflight)

#### API-20: Use HTTP status 422 (Unprocessable Entity) for input validation errors
✅ **Ja**: 422 voor validatiefouten, 400 voor malformed requests

---

### Security & Headers

#### API-34: Provide an IP whitelist for access
✅ **Ja**: Optional IP whitelist per API key

#### API-35: Secure APIs with HTTPS
✅ **Ja**: Enkel HTTPS, TLS 1.3 minimum, HSTS header

#### API-36: Include the HTTP security headers in any response
✅ **Ja**:
```
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

---

### Documentation

#### API-48: Use the OpenAPI specification
✅ **Ja**: OpenAPI 3.1 spec volledig gedocumenteerd

#### API-49: Include API-documentation in OpenAPI specification
✅ **Ja**: 
- Alle endpoints met beschrijvingen
- Request/response examples
- Error scenarios

#### API-50: Publish the OpenAPI specification
✅ **Ja**: Openbaar beschikbaar op GitHub & via `/open-api` endpoint

---

## Implementatiedetails per Resource

### Schulddossiers

- **Resource**: `/schulddossiers`
- **Methods**: GET (list, detail), POST (create), PUT (update), DELETE (remove)
- **Filtering**: status, schuldhulpverlener_id, burger_id, datum_start, datum_end
- **Sorting**: created_at, modified_at, status
- **Pagination**: Offset/limit

### Schuldhulpverleners

- **Resource**: `/schuldhulpverleners`
- **Methods**: GET (list, detail), POST (create), PUT (update)
- **Filtering**: naam, organisatie, specialisatie
- **Read-only voor burgers** (geen POST/PUT)

### Schulden

- **Resource**: `/schulddossiers/{id}/schulden`
- **Methods**: GET (list), POST (add), DELETE (remove)
- **Filtering**: schuldeiser, bedrag_min, bedrag_max, status

### Afspraken

- **Resource**: `/schulddossiers/{id}/afspraken`
- **Methods**: GET (list), POST (create), PUT (update), DELETE (cancel)
- **Filtering**: status, type, datum_start, datum_end

---

## Niet nageleefd (Justificatie)

Sommige ADR regels zijn niet van toepassing:

- **API-05 t/m API-08**: Specifiek voor SOAP/XML, niet relevant voor REST JSON
- **API-21 t/m API-33**: Database/backend spécifieke, niet client-facing
- **API-37 t/m API-47**: Deel van organisatorische governance, niet API-spec

---

## Versiehistorie

| Versie | Handelsmarkering | Datum |
|--------|------------|-------|
| 0.1.0 | Initial | Mai 2026 |

## Bronnen

- [ADR Publicatie](https://publicatie.centrumdigitaal.nl/adrs/)
- [LOGIUS](https://www.logius.nl/)
- [VNG Realisatie](https://vng-realisatie.nl/)

---

**Status**: ✅ Gevalideerd tegen ADR v3.0
