# Bedrijfsregels

Dit document beschrijft de business logic en validatie regels.

---

## Registratie & Intake

### BR-001: Verplichtheid Schuldhulp aanvraag
- **Omschrijving**: Burger mag alleen schuldhulp aanvragen als ze inwonerschap hebben in deelnemende gemeente
- **Implementatie**: Verificatie via gemeente_code in BRP koppeling
- **Foutboodschap**: "U woont niet in een gemeente met schuldhulpverlening"

### BR-002: Minimale leeftijd
- **Omschrijving**: Burger moet minimaal 16 jaar zijn om hulp aan te vragen
- **Berekening**: `TODAY - geboortedatum ≥ 16 jaren`
- **Foutboodschap**: "U bent nog te jong voor schuldhulpverlening"

### BR-003: Unieke Intake per 12 maanden
- **Omschrijving**: Burger mag max 1x per 12 maanden opnieuw intake doen
- **Logica**: Vorige AFGEROND of GESLOTEN dossier + 12m wachttijd
- **Exception**: Admin kan waiver geven (reden required)

---

## Schulddossier Lifecycle

### BR-010: Dossier Status Transitions
- **NIEUW** → **ACTIEF**: Bij eerste schuldhulpmaatregel start
- **ACTIEF** → **AFGEROND**: Alle schulden BETAALD of KWIJTGESCHOLDEN
- **ACTIEF** → **GESLOTEN**: Admin archvering, ≥ 6 maanden inactief
- **NIEUW** → **GESLOTEN**: Direct (intake afgewezen)

**Blocked transitions**: Geen backward moves (AFGEROND → ACTIEF verboden)

### BR-011: Mandatory Schulden
- **Omschrijving**: Schulddossier moet minstens 1 schuld hebben
- **Validatie**: `COUNT(schuld WHERE dossier_id = X) ≥ 1`
- **Moment**: Na intake acceptance (status ACTIEF)

### BR-012: Dossier Archivering
- **Omschrijving**: GESLOTEN dossiers kunnen gearchiveerd worden
- **Timing**: ≥ 6 maanden GESLOTEN
- **Archief**: Read-only met audit trail behavior

---

## Schulden Management

### BR-020: Schuld Bedrag Validatie
- **Omschrijving**: Schuldbedrag moet positief zijn
- **Regel**: `bedrag > 0`
- **Exception**: Schulden < EUR 50 worden manueel gereviewd (overhead vs. effect)

### BR-021: Schuld Status Lifecycle
```
OPEN ──→ BETAALD (afgelost)
│    ├─→ KWIJTGESCHOLDEN (vergeven)
│    ├─→ INGEVORDERD (schuldeiser escalteert)
│    └─→ BETWIST (burger bezwaar in)
│
BETWIST ──→ OPEN (schuldeiser aanvaard schuld)
│    └──→ BEËINDIGD (dispute resolution)

VOORBIJ ← (import historische schuld, static)
```

### BR-022: Totaal Schuldbedrag Berekening
- **Omschrijving**: Dossier totaal = SOM aller schoolden status OPEN
- **Berekening**: 
  ```sql
  SELECT SUM(bedrag) FROM schuld 
  WHERE dossier_id = X AND status IN ('OPEN', 'BETWIST')
  ```
- **Caching**: TTL 15 minuten (refresh on schuld change)

### BR-023: Schuld Afbetaling
- **Regel**: Once Schuld status BETAALD → immutable (no edit)
- **Audit**: Totale afbetaalde bedrag gelogd

---

## Schuldhulpmaatregelen

### BR-030: Maatregel Start Validatie
- **Omschrijving**: Maatregel mag start enkel als dossier ACTIEF
- **Regel**: `dossier.status = 'ACTIEF'` voorwaarde
- **Foutboodschap**: "Kan maatregel niet starten: dossier niet actief"

### BR-031: Parallelle Maatregelen
- **Omschrijving**: Schulddossier kan meerdere ACTIEVE maatregelen hebben
- **Voorbeeld**: 
  - Schuldbemiddeling (onderhandelen schuldeiser)
  - Budgetbeheer (structureren uitgaven)
  - Beiden tegelijk OK

### BR-032: Maatregel Duration Validatie
- **Omschrijving**: Einddatum moet ≥ startdatum zijn
- **Regel**: `einddatum_gepland ≥ start_datum`

### BR-033: Maatregel Beëindiging
- **Omschrijving**: Beëindiging vereist reden + datum
- **Status**: AFGEROND (succesvol) vs. BEËINDIGD (voortijdig)
- **Reden**: AFGEROND_SUCCESVOL / VOORTIJDIG_BURGER / VOORTIJDIG_HULPVERLENER / ANDERE

### BR-034: Succesvol Afgerond Criteria
- **Omschrijving**: Maatregel → AFGEROND als alle doelen bereikt
- **Definitie doelen**:
  - Schuldbemiddeling: Schuldreducering bereikt (bijv. 30%)
  - Budgetbeheer: Maandelijks sparen + schuldafbetaling
  - Insolventie: Wettelijke procedure voltooid
- **Approval**: Schuldhulpverlener confirms completion

---

## Autorisatie & Toegangscontrole

### BR-040: Burger Dossier Eigenaarschap
- **Omschrijving**: Burger ziet enkel EIGEN schulddossier (read-only)
- **Implementatie**: `dossier.burger_id = @current_user.bsn` filter
- **Edit**: Burger kan NIET wijzigen (schuldhulpverlener does)

### BR-041: Schuldhulpverlener Toegang
- **Omschrijving**: Hulpverlener ziet enkel dossiers waar zij aan werken
- **Tabel**: `dossier_hulpverlener` junction table
- **Rollen**:
  - VERANTWOORDELIJKE: Full edit
  - BETROKKEN: Limited edit (alleen maatregel)
  - OBSERVER: Read-only

### BR-042: Gemeente Statistieke Toegang
- **Omschrijving**: Gemeente beleidsmedewerker ziet ALLEEN geanonimiseerde statistieken
- **Data voornemen**: GEEN individuele de Burgernaam / BSN
- **Geaggregeerd**: Counts, percentages, trends per week/maand

### BR-043: Admin Override
- **Omschrijving**: Admin (zeer beperkt) kan alle dossiers aanpassen
- **Logging**: Alle admin acties gelog (required voor audit)
- **Approval**: 4-ogen principle voor sensitive changes

---

## Data Privacy & GDPR

### BR-050: Data Minimalisatie
- **Omschrijving**: Enkel noodzakelijke persoonsgegevens verwerkt
- **Vereenvoudigd**: Burger via referentie (BSN) + BRP koppeling
- **Geen dubbel**: Email niet in dossier als BRP heeft

### BR-051: Right to Erasure
- **Omschrijving**: Burger kan verwijdering aanvragen (GDPR art. 17)
- **Implementatie**: Soft delete (deleted_at timestamp)
- **Audit trail**: Behouden (wettelijk vereist)
- **Data export**: Eerst exporteren (art. 20), dan wissen

### BR-052: Data Retention
- **Scholdd open**: 7 jaren (Faillissementswet requirement)
- **Gesloten dossier**: 5 jaren (nauwkeurigheid/relevantie)
- **Audit trail**: 7 jaren (compliance)
- **Automatisch**: Delete scripts jaarlijks gecron

### BR-053: Consent Tracking
- **Omschrijving**: Documenten consent voor data sharing
- **Vereist**: Burger ondertekening voor bepaald maatregelen (bijv. schuldeiser contacteren)
- **Opslag**: Toestemming entity met datum + houder

---

## Finantiele Validaties

### BR-060: Schuldbedrag Grenzen
- **Minimum**: EUR 1,00 (business decision)
- **Maximum**: EUR 999.999.999,99 (practical limit)
- **Gelykstelling**: Geen valuta conversie (EUR only)

### BR-061: Afbetaling Tracking
- **Omschrijving**: Afbetaalde bedrag ≤ origineel bedrag
- **Regel**: `bedrag_afgelost ≤ bedrag`
- **Procentage**: `afgelost% = (bedrag_afgelost / bedrag) * 100`

---

## Operationeel & Integriteit

### BR-070: Audit Trail Immutability
- **Omschrijving**: Audit records kunnen NOOIT gewijzigd/verwijderd
- **Implementatie**: DB trigger: BEFORE DELETE/UPDATE → ERROR
- **Beperking**: Creatie + read only

### BR-071: Encryption at Rest
- **Omschrijving**: Gevoelige velden versleuteld in database
- **Velden**: BSN, email, telefoonnummer (customer request)
- **Cipher**: AES-256, keys in HSM

### BR-072: Timestamp Auditability
- **Omschrijving**: Alle entiteiten hebben created_at + updated_at
- **Timezone**: UTC (niet lokaal)
- **Precisie**: Secunde granulariteit
- **Immutable**: created_at never wijzigt

---

## Alerting & Monitoring

### BR-080: Anomaly Detection
- **Omschrijving**: System detecteert ongewone patronen
- **Triggers**:
  - >10 dossier aanmakingen per uur (possibly abuse)
  - Schuldbedrag >EUR 1M (data quality check)
  - Admin login outside EU tiez (security alert)

### BR-081: SLA Alerting
- **Omschrijving**: Alert as SLA breached
- **SLA**: 4 uur response op hulpvraag
- **Action**: Auto-escalatie naar supervisor

---

## Testing & Validation

### BR-090: Input Sanitization
- **Omschrijving**: Alles input validated + sanitized
- **Regex**: Streng beperkt (whitelist beter dan blacklist)
- **Injection prevention**: Parameterized queries (GEEN string concat)

### BR-091: State Machine Validation
- **Omschrijving**: Transitions enkel via allowed paths
- **Enforcement**: App-level state machine
- **Example**:
  ```python
  allowed_transitions = {
    'NIEUW': ['ACTIEF', 'GESLOTEN'],
    'ACTIEF': ['AFGEROND', 'GESLOTEN'],
    'AFGEROND': ['GESLOTEN'],
    'GESLOTEN': []  # Terminal state
  }
  ```

---

**Document version**: 1.1 | **Last update**: Mei 2026
