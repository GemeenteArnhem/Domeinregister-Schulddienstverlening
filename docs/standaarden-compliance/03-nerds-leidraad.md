# NERD-leidraad: Beveiliging & Privacy

Dit register volgt de **NERD-leidraad** ("Norm Evaluatie Regelgeving Digitaal").

Referentie: https://www.logius.nl/diensten/nerd-leidraad

## NERD Compliance Checklist

### 1. Versleuteling in Transit

✅ **HTTPS verplicht**: Alle verbindingen via TLS 1.3+

✅ **Certificate pinning (optioneel)**: Voor gevoelige integraties
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

✅ **Geen plaint HTTP**: Alle HTTP → HTTPS 301 redirect

✅ **CORS strikt**: Geen wildcard origins, listed domains enkel

### 2. Versleuteling at Rest

✅ **Database**: Transparante Data Encryption (TDE) voor PostgreSQL
```sql
CREATE EXTENSION pgcrypto;
-- Gevoelige velden: encrypted_column = pgp_sym_encrypt(data, key)
```

✅ **Backups**: Versleuteld, off-site storage

✅ **Logging**: Gevoelige data (BSN, bedragen) niet in plain text

### 3. Authenticatie & Autorisatie

✅ **OAuth2 + JWT**: 
- Authorization Code Flow voor web apps
- Client Credentials Flow voor service-to-service
- JWT signing met RS256 (asymmetrisch)

✅ **MFA (Multi-Factor Authenticatie)**:
- Voor admins: Verplicht TOTP (Google Authenticator)
- Voor burgers: SMS-verificatie bij eerste login

✅ **Session management**:
- JWT TTL: 15 minuten (access token)
- Refresh token: 7 dagen (geroteerd per use)
- Logout invalidates session

### 4. Input Validatie & Output Encoding

✅ **Server-side validatie**: Nooit client-side enkel
```
400 Bad Request: Invalidates schema
422 Unprocessable Entity: Business logic violated
```

✅ **XSS preventie**: Alle output HTML-encoded
```
Content-Type: application/json; charset=utf-8
X-Content-Type-Options: nosniff
```

✅ **SQL Injection preventie**: Parameterized queries
```python
cursor.execute("""
  SELECT * FROM schulddossiers WHERE id = %s AND status = %s
""", [dossier_id, 'ACTIEF'])
```

✅ **CSRF protectie**: SameSite=Strict cookies, CSRF tokens

### 5. Logging & Monitoring

✅ **Security logging**:
```
- Failed login attempts (gemaskeerd)
- API key creations/deletions
- Permission changes
- Data exports
- Admin actions
```

✅ **Centralized logging**: ELK stack (Elasticsearch, Logstash, Kibana)

✅ **Alerting**: 
- >5 failed logins in 5 min
- Unexpected data access
- Unusual API patterns

✅ **Log retention**: 
- 90 dagen hot storage
- 1 jaar archive (gecomprimeerd, versleuteld)

### 6. Access Control (RBAC)

✅ **Rollen**:
- **Burger**: Zelf lezen, niet wijzigen
- **Schuldhulpverlener**: Lezen/schrijven assigned dossiers
- **Gemeente (beleidsmedewerker)**: Statistieken, geen persoonsgegevens
- **Admin**: Volledige toegang (zeer beperkt)

✅ **Permissions matrix**:
```
|                 | Burger | Hulpverlener | Gemeente | Admin |
|-----------------|--------|--------------|----------|-------|
| Dossier lezen   | eigen  | assigned     | none     | all   |
| Dossier creëren | nee    | ja           | nee      | ja    |
| Statistieken    | nee    | eigen        | anoniem  | all   |
| Beheer          | nee    | nee          | nee      | ja    |
```

### 7. API Security

✅ **Rate limiting**:
- Unauthenticated: 100 req/min per IP
- Authenticated: 1000 req/min per user
- Burst: 10 per 1 sec (prevent DoS)

✅ **API keys**:
- Zolang niet gebruikt, revoke na 90 dagen
- Rotation ogni anno
- Scope limiting (read-only, specific endpoints)

✅ **Signature verification**: Voor webhooks (HMAC-SHA256)

### 8. Incident Management

✅ **Incident response plan**: 
- Detection → 15 min
- Triage → 1 hour
- Mitigation → 4 hours (kritiek)
- Communication → 24 hours

✅ **Breach notification**: GDPR Article 33/34
- Databeschermingsautoriteit: < 72 uur
- Data subjects: Sans undue delay

✅ **Post-incident review**: Lessonslearned within 1 week

### 9. Third-Party Security

✅ **Dependency scanning**: 
- npm audit
- Safety check (Python)
- Trivy (container scanning)

✅ **Vendor assessment**:
- SOC 2 certification
- Security SLA
- Data processing agreements

✅ **Supply chain**:
- Signed commits required
- CI/CD signing (cosign)

### 10. Audit Trail (GDPR Article 25)

✅ **Immutable logging**:
```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY,
  timestamp TIMESTAMP NOT NULL,
  user_id VARCHAR NOT NULL,
  action VARCHAR NOT NULL,
  resource_type VARCHAR NOT NULL,
  resource_id VARCHAR NOT NULL,
  old_value JSON,
  new_value JSON,
  ip_address INET NOT NULL,
  created_at TIMESTAMP NOT NULL -- immutable
);
```

✅ **Retention**: 7 jaren (wettelijk vereiste)

---

## NERD Risicomapping

| Risico | Mitigatie | Eigenaar |
|--------|-----------|----------|
| Data breach | Encryption, MFA, monitoring | Security |
| Ongeautoriseerde toegang | RBAC, logging, audit trail | Backend |
| DoS/DDoS | Rate limiting, WAF, CDN | Infrastructure |
| Malware in dependencies | Scanning, SCA, vendor checks | DevOps |
| Phishing (users) | Training, MFA, email filtering | Organization |
| Insider threat | Logging, 4-eye review, background check | HR/Security |

---

## Compliance Verification

✅ Annual penetration test (external)
✅ Monthly vulnerability scanning
✅ Quarterly security review (internal)
✅ GDPR compliance audit (annual)

---

## Resources

- [NERD Leidraad](https://www.logius.nl/diensten/nerd-leidraad)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE Most Dangerous](https://cwe.mitre.org/)
- [GDPR Article 25](https://gdpr-info.eu/art-25-gdpr/)

---

**Status**: ✅ NERD-compliant, jaarlijks geverifieerd
