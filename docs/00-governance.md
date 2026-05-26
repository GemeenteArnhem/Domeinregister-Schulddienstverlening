
# Governance & Open Source Strategy

## Eigenaarschap & Beheer

**Eigenaar**: Gemeente Arnhem
**Repository**: github.com/gemeentearnhem/schulddienstverlening-register
**Maintainers**: Paul Reinold, Ludo Klein Holte

## Licentie

**EUPL-1.2** (Europese Publieke Licentie)

## Versiebeleid

**Semantic Versioning**: MAJOR.MINOR.PATCH

- **MAJOR**: Breaking changes (nieuwe objecttypen, verwijderde velden)
- **MINOR**: Backwards-compatible features (nieuw endpoint, optioneel veld)
- **PATCH**: Bug fixes

Bijvoorbeeld: 1.2.3 = Major 1, Minor 2, Patch 3

## Release Cyclus

- **Patch releases**: Elke 2 weken (bugfixes)
- **Minor releases**: Elke 2 maanden (features)
- **Major releases**: 1x per jaar (planning)

## Backward Compatibility Gaurantie

- API ondersteunt minstens 2 major versions terug
- Database migrations zijn verplicht (geen data loss)
- Deprecated velden worden 2 releases gewaarschuwd

## Code Review & Contribution Process

1. Fork repo
2. Branch: `feature/description` of `bugfix/description`
3. Commit met duidelijke messages
4. Create Pull Request
5. Code review (minimaal 2 approvals van maintainers)
6. CI/CD tests moeten slagen
7. Merge naar main

## Maintainer Verantwoordelijkheden

- Response time PR: < 7 dagen
- Security issues: < 3 dagen
- Release management
- Documentatie en changelog updates
- Security advisories uitleveren waar nodig

## Governance Model

Dit project volgt het **Benevolent Dictator For Life (BDFL)** model met maintainers:

- **Lead maintainer**: Gemeente Arnhem (Paul Reinold)
- **Core maintainers**: Minimaal 2 andere personen met merge-rights
- **Contributors**: Iedereen kan PR indienen

Besluiten over major changes (architectural decisions, breaking changes) worden genomen door core maintainers in consensus.

## Releaseplan 2026

| Versie | Datum | Plan |
|--------|-------|------|
| 0.1.0 | Q2 2026 | Alpha: kern informatiemodel + basis API |
| 0.2.0 | Q3 2026 | Beta: volledige API, start UI |
| 1.0.0 | Q4 2026 | Stable release, production-ready |

## Dependencies & Support

- **Node.js**: LTS versies (22.x+)
- **Python**: 3.11+
- **PostgreSQL**: 15+
- **Support**: Issues via GitHub, Security via SECURITY.md

## Audit & Compliance Verifikatie

- **Jaarlijkse security audit**: Externe partij
- **GDPR compliance check**: Jaarlijks
- **BIO2 assessment**: Jaarlijks
- **Code quality**: SonarQube CI/CD checks

## Communicatie

- **Releases**: via GitHub Releases
- **Breaking changes**: Mailinglijst VNG Fieldlab
- **Discussions**: GitHub Discussions
- **Chat**: [Slack channel - optioneel]

## Conflict Resolution

In geval van meningsverschillen over design/features:

1. Discussie in GitHub Issue/Discussion
2. Vote onder core maintainers (majority wins)
3. BDFL final decision indien nodig

## Deprecated Features

Features die niet meer ondersteuning krijgen:

- Marked as deprecated in docs
- Waarschuwing in API responses
- Minstens 2 releases supported
- Removed in volgende major version

---

**Laatste update**: Mei 2026
- Documentatie bijwerken