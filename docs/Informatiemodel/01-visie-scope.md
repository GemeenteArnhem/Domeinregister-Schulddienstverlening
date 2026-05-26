# Visie & Scope: Schulddienstverlening Register

## Visie

Een centraal, open, beveiligd informatieregister waar schuldhulpverleners en 
gemeenten schulddossiers van burgers kunnen beheren en delen, conform Nederlandse 
wet- en regelgeving (Faillissementswet, WBH, GDPR).

## Context: GGM-Schulddienstverlening

Dit register implementeert de **Schulddienstverlening-component** uit het 
Gemeentelijke Gegevensmodel (GGM).

**GGM-referentie**: 
- https://www.gemmaonline.nl/
- Thema: "Schulddienstverlening"
- Versie: GGM 2024

## Doelgroepen

### 1. Schuldhulpverleners
- Kunnen schulddossiers inzien en bijwerken
- Kunnen afspraken met burgers registreren
- Kunnen voortgang tracken

### 2. Gemeenten (beleidsmedewerkers)
- Overzicht schulddienstverleners
- Rapportage & statistieken
- Beleid bepalen

### 3. Burgers
- Inzicht in eigen schulddossier (read-only)
- Kunnen aanvragen voor schuldhulp indienen
- Kunnen afspraken inzien

## Scope: Wat wel

✅ **Schulddossiers** - kern: alles over schuldsituatie burger
✅ **Schuldhulpverleners** - registratie dienstverleners
✅ **Afspraken** - schuldhulptrajecten & maatregelen
✅ **Documenten** - bijlagen (contract, schuldoverzicht, etc.)
✅ **Audit trail** - wie deed wat en wanneer
✅ **Beveiliging** - autorisatie, versleuteling, compliance

## Scope: Wat niet (out of scope)

❌ **Financiële administratie** - niet dit register, maar boekhoudpakket
❌ **Incassovorderingen** - dat is KvK / incassobureaus
❌ **NEFT/schuldenaar details** - alleen kerngegevens
❌ **Procesvoering** - rechtsgang, rechtbank
❌ **Faillissement beheer** - dat is curatoren

## Gebruiksscenario's

### Scenario 1: Burger meldt zich aan
1. Burger vult intakeformulier in (via portal)
2. Schuldhulpverlener ziet aanmelding
3. Schuldhulpverlener maakt schulddossier aan
4. Burger ziet eigen dossier (read-only)

### Scenario 2: Schuldhulpverlener sluit overeenkomst
1. Schuldhulpverlener voegt schuldoverzicht toe (document)
2. Schuldhulpverlener definieert maatregelen (spaarplan, schuldbemiddeling)
3. Burger ontvangt afspraken via email
4. Burger kan inzien, niet wijzigen

### Scenario 3: Rapportage naar gemeente
1. Gemeente vraagt statistieken op
2. API levert: aantal dossiers, voortgang, soort schuld
3. Geen persoonlijke gegevens in rapportage (anoniem)

## Niet-functionele Eisen

### Performance
- Schulddossier ophalen: < 100ms (p95)
- Zoeken naar schulddossiers: < 500ms (10.000 dossiers)
- API throughput: 1.000 req/sec

### Beschikbaarheid
- Uptime: 99.5% (SLA)
- Graceful degradation (read-only mode) bij problemen

### Beveiliging
- HTTPS enkel (TLS 1.3+)
- OAuth2 + JWT authenticatie
- GDPR-compliant (versleuteling, data minimalisatie)
- BIO2-conforme logging

### Schaal
- 100.000+ actieve dossiers
- Multi-tenant (verschillende gemeenten)
- Partitionering naar gemeente

## Niet-functionele Risico's

**Risico**: Gevoelige schuldinformatie lekt
**Mitigatie**: Versleuteling, strenge autorisatie, audit logging, penetration testing

**Risico**: Systeem overbelast (veel gemeenten)
**Mitigatie**: Caching, read replicas, auto-scaling

**Risico**: Systeem overbelast (veel gemeenten)
**Mitigatie**: Caching, read replicas, auto-scaling

**Risico**: GDPR-schending (data langer dan nodig)
**Mitigatie**: Automated data deletion, data minimalisatie, Privacy Impact Assessment

---

## Use Case Diagram

```
┌─────────────────────────────────────────┐
│         Schulddienstverlening            │
│         Register System                  │
└─────────────────────────────────────────┘
          ▲                    ▲
         /                      \
        /                        \
   [Burger]  [Schuldhulpverle-ner]  [Gemeente]
   - Read own dossier    - Manage dossiers  - View stats
   - Submit request      - Track progress   - Compliance
   - Pay plan            - Document        - Reporting
```

## Technology Stack (indicatief)

### Backend
- **Runtime**: Python 3.11+ (FastAPI) of Node.js (Express)
- **Database**: PostgreSQL 15+ (relational)
- **Cache**: Redis (sessions, rate limit)
- **Search**: Elasticsearch (optional, for large queries)
- **Message queue**: RabbitMQ (async tasks, emails)

### Frontend
- **Web**: React/Vue (SPA)
- **Portal (Burger)**: Static HTML (accessibility first)
- **Admin**: Streamlit (quick dashboards)

### Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes (production)
- **Cloud**: Azure Government / Sovereign cloud
- **CI/CD**: GitHub Actions

### Security
- **Auth**: Keycloak (OAuth2/OIDC)
- **Secrets**: HashiCorp Vault
- **Monitoring**: ELK Stack
- **API Gateway**: Kong

---

## Success Criteria

### Adoption
- ✅ 10+ municipalities using by Dec 2026
- ✅ 50,000+ active dossiers by Q2 2027
- ✅ 99.5% API uptime

### Quality
- ✅ Zero CRITICAL security vulnerabilities
- ✅ <100ms API latency (p95)
- ✅ GDPR/BIO2 full compliance

### Community
- ✅ 50+ community contributions (code, docs)
- ✅ Active GitHub discussions
- ✅ Public roadmap with 6-month horizon

---

## Governance & Decision Making

- **Steering committee**: VNG Realisatie (quarterly)
- **Architecture review**: Architects (monthly)
- **Community feedback**: GitHub Discussions (ongoing)
- **Roadmap**: Public (https://github.com/...projects)

---

## Versioning & Release Plan

See [docs/00-governance.md](../00-governance.md) for full versioning strategy.

Quick timeline:
- **Q2 2026**: v0.1.0 (alpha)
- **Q3 2026**: v0.2.0 (beta)
- **Q4 2026**: v1.0.0 (stable)
- **2027+**: v1.x (maintenance), v2.0 (planning)

---

**Document version**: 1.2 | **Last update**: Mei 2026