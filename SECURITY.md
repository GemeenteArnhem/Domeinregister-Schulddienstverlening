# Beveiligingsrichtlijnen

Dit document beschrijft hoe we beveiligingsproblemen herkennen en aanpakken.

## Meldingsprocedure voor beveiligingskwetsbaarheden

**Meld NOOIT openbaar via GitHub Issues!**

Als je een beveiligingskwetsbaarheid ontdekt:

### 1. Email sturen naar

```
security@schulddienstverlening.register
```

**Onderwerp**: `[SECURITY] Vulnerability - <short description>`

### 2. Informatie die wij nodig hebben

- Beschrijving van de kwetsbaarheid
- Stappen om het probleem te reproduceren
- Potentiële impact
- Eventuele proof-of-concept (PLC) code (optioneel, maar nuttig)
- Je contactgegevens (email/naam)

### 3. Wat kunt u verwachten

- **Initiële bevestiging**: Binnen 3 werkdagen
- **Updates**: Minimaal om de week
- **Patch release**: Op basis van ernst (zie hieronder)
- **Credits**: Je naam in release notes (tenzij je anoniem wilt blijven)

## Ernstclassificatie (CVSS)

| CVSS | Ernst | Patch Timeline |
|------|-------|-----------------|
| 9.0-10.0 | Kritiek | ASAP (< 48u) |
| 7.0-8.9 | Hoog | < 1 week |
| 4.0-6.9 | Gemiddeld | < 2 weken |
| 0.1-3.9 | Laag | Volgende release |

## Beveiligingspraktijken

### During Development

- ✅ **Geheimen**: Nooit geharde geheimen in code. Gebruik environment variables.
- ✅ **Dependencies**: Controleer regelmatig op vulnerabilities (`npm audit`, `safety check`)
- ✅ **Logging**: Nooit persoonlijke informatie (BSN, creditcardnummers) loggen
- ✅ **Validatie**: Input validatie op alles wat binnenkomt
- ✅ **HTTPS**: Altijd TLS 1.3 of hoger

### API Security

- ✅ **OAuth2 + JWT**: Zie [docs/api-spec/security-oauth2-jwt.md](./docs/api-spec/security-oauth2-jwt.md)
- ✅ **CORS**: Strikt CORS-beleid, geen `*` in production
- ✅ **Rate limiting**: Voorkomen brute force
- ✅ **API versioning**: Support oude versies minstens 2 major releases

### Database Security

- ✅ **Versleuteling**: At rest (TDE) en in transit (TLS)
- ✅ **Access control**: Minimale privileges per user
- ✅ **Backups**: Versleuteld, off-site
- ✅ **Sanitizing**: Parameterized queries (geen SQL injection)

### Data Privacy (GDPR)

- ✅ **Data minimalisatie**: Verzamel alleen wat nodig is
- ✅ **Encryption**: Gevoelige data versleuteld
- ✅ **Right to delete**: Implementeer deleteeren van burgerdata
- ✅ **Data retention**: Automatische deletion na ingestelde periode
- ✅ **Audit trail**: Logging van wie wat las/wijzigde

### BIO2/ISO 27001 Compliance

- ✅ **Incident management**: Documenteer beveiligingsincidenten
- ✅ **Access logging**: Alle toegang wordt gelogd
- ✅ **Penetration testing**: Jaarlijks
- ✅ **Security training**: Voor maintainers/contributors

## Disclosure Timeline

Wij volgen de **Verantwoorde Openbaarmaking** (Responsible Disclosure):

1. **Day 0**: Kwetsbaarheid gerapporteerd aan `security@...`
2. **Day 1-3**: Wij bevestigen ontvangst
3. **Day 4-30**: Wij diagnosticeren en patch ontwikkelen
4. **Day 30**: Wij stellen embargo voor openbare release in (samen met reporter)
5. **Day 30+**: Patch released; kwetsbaarheid mag openbaar gemaakt worden

**Embargo verlengingen**: In uitzondering gevallen (complexe fix) kunnen we embargo verlengen. Dit bespreken we samen met je.

## Gerapporteerde Vulnerabilities

### Huidige status
- Geen actieve security issues
- Laatse security audit: [datum]

### Opgelost
(Leeg nu, zal worden bijgewerkt na eerste 3rd party audit)

## Security Headers

Production API bevat deze security headers:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

## CI/CD Security

- ✅ Branch protection: main branch vereist code review
- ✅ Dependency scanning: Automatisch per commit
- ✅ SAST: Static Application Security Testing
- ✅ Container scanning: Docker images gescand op vulnerabilities

## Secrets Management

- Nooit secrets in git (use `.gitignore`)
- Gebruik GitHub Secrets voor CI/CD
- Rotate credentials regelmatig
- Separate keys per environment (dev/staging/prod)

## Contact

- **Beveiligingsvragen**: security@schulddienstverlening.register
- **Bug reports**: GitHub Issues (niet voor vulns!)
- **Responsible Disclosure**: Zie bovenstaande procedure

---

**Dank dat je het veiliger stelt voor iedereen!** 🔒
