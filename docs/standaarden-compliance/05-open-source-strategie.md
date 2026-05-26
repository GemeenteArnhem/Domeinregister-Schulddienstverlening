# Open Source Strategie

Dit register is **open source** en volgt EUPL-1.2 licentie. Dit document beschrijft onze aanpak.

Referenties:
- https://opensource.org/
- https://www.eupl.eu/

---

## Waarom Open Source?

### Voor Publieke Sector

✅ **Transparantie**: Code is openbaar, geen black boxes
✅ **Geen vendor lock-in**: Andere gemeenten kunnen fork maken
✅ **Kostenbesparings**: Gedeelde tools, geen licentiekosten
✅ **Innovatie**: Community kan bijdragen en verbeteren

### Voor Users (Burgers, Gemeenten)

✅ **Controle**: Jouw gegevens worden niet op geheim software verwerkt
✅ **Security through openness**: Bugs worden snel ontdekt
✅ **Duurzaamheid**: Niet afhankelijk van bedrijf dat falliet gaat
✅ **Interoperabiliteit**: Standaarden in plaats van proprietaire formaten

---

## Licentie: EUPL-1.2

### Wat betekent EUPL-1.2?

✅ **Free to use**: Iedereen mag het gebruiken, wijzigen, distribueren
✅ **Copyleft**: Wijzigingen moeten ook open source zijn
✅ **Compatible**: Samenwerking met GPL, LGPL, AGPL
✅ **Multilingual**: Erkend in alle EU talen
✅ **Public sector friendly**: Aanbevolen voor overheid

### Jouw rechten

Je mag:
- 🔄 **Gebruiken** voor welk doel dan ook (commercieel/non-profit)
- ✏️ **Wijzigen** en verbeteren
- 📢 **Distribueren** aan anderen
- 📖 **Publiceren** je wijzigingen

Je moet:
- ©️ **Credit geven** aan originele auteur
- 📜 **Licentie behouden** (EUPL-1.2 voor wijzigingen)
- 📝 **Changes documenteren** (changelog)

Je mag NIET:
- ❌ Handelsmerk/logo gebruiken (VNG Realisatie = beschermd)
- ❌ Garanties geven (zij niet van schuld vrijgesteld, zie artikel 17)

---

## Contributor Guidelines

### Code of Conduct

We volgen de **Contributor Covenant v2.0**:

✅ **Respectful**: Iedereen is welkom, ongeacht achtergrond
✅ **Inclusive**: Geen discriminatie, harassing, toxiciteit
✅ **Safe**: Meld problemen confidentieel naar `conduct@...`

### Hoe Bijdragen?

1. **Fork** de repo
2. **Branch**: `feature/description`
3. **Commit**: "feat: small description"
4. **Push** naar fork
5. **Pull Request**: Beschrijf wat en waarom
6. **Review**: Minimaal 2 approvals
7. **Tests**: CI/CD moet slagen
8. **Merge**: Maintainer merged en released

### Contributor License Agreement (CLA)

Voor substantiële bijdragen (>50 lines):

Wij vragen je een CLA te ondertekenen. Dit zorgt ervoor dat:
- Jij het recht hebt je code te delen
- Jij geen patentenanspraken maakt
- Jij akkoord gaat met EUPL-1.2 licentie

CLA template: [CLA.md](./CLA.md) (in voorbereiding)

---

## Community & Support

### Governance

Besluitvorming gebeurt transparent:

✅ **Issues**: Iedereen kan suggesties doen
✅ **Discussions**: Q&A, RFC (Request for Comments)
✅ **Roadmap**: Publieke planning (GitHub Projects)
✅ **Decisions**: Documented in ADRs (Architecture Decision Records)

### Support Channels

| Channel | Doel | Response Time |
|---------|------|----------------|
| 🐛 Issues | Bugs, features | < 7 dagen |
| 💬 Discussions | Vragen, design | < 7 dagen |
| 🔒 Security | Vulnerabilities | SECURITY.md |
| 📧 Email | Formal inquiries | < 3 dagen |

### Maintainer Recognition

Contributors met 10+ merged PRs:

- 🏅 Named in README
- 📜 Certificate of contribution
- 🎁 Limited edition sticker (optional)

---

## Reuse & Derivative Works

### Jij mag fork maken als

✅ Je volgt EUPL-1.2 (open source)
✅ Je credit geeft aan origineel
✅ Je changes gepubliceerd maakt
✅ Je community participatie heeft

### Jij mag NIET (tegen EUPL)

❌ Closed source fork maakt
❌ Licentie weglaat of verandert
❌ Ons handelsmerk gebruik (anders dan credit)

### Upstreaming Changes

We verwelkomen upstream contributions:

```
Fork → Fix/Feature → PR → Review → Merge → Release
```

Grote features? Maak eerst een Issue/Discussion voor alignment.

---

## Dependency Management

### Policy

✅ **Prefer open source**: Geen proprietary libs (exceptions require approval)
✅ **License compatibility**: Enkel EUPL-compatible (FSF, OSI approved)
✅ **Security scanning**: npm audit, Safety, trivy
✅ **Regular updates**: Monthly dependency updates

### Forbidden Licenses

Wij gebruiken NIET:

❌ **Proprietary**: Microsoft Visual Studio, etc.
❌ **GPL-2 (strict)**: Incompatible met EUPL
❌ **Commons Clause**: Commerciale restricties
❌ **Unlicensed code**: Moet licentie hebben

### Approved Licenses

✅ MIT, Apache 2.0, BSD
✅ LGPL 2.1+, GPL 3.0+
✅ EUPL itself
✅ ISC, Unlicense

---

## Attribution & Credits

### In Code

```python
# Modified from schulddienstverlening-register
# https://github.com/gemeentearnhem/schulddienstverlening-register
# Licensed under EUPL-1.2
# Original author: Gemeente Arnhem <contact@...>
```

### In Documentation

```markdown
Based on [Schulddienstverlening Register](https://github.com/gemeentearnhem/schulddienstverlening-register)
(EUPL-1.2) © 2026 Gemeente Arnhem
```

###  In Release Notes

```
## 1.2.0 - 2026-05-26

### Contributors

- @johndoe: Added user authentication
- @janedoe: Fixed database performance
- VNG Community: Testing & feedback
```

---

## Sustainability & Long-term Support

### Versioning & Support

| Major | Status | Support Until |
|-------|--------|----------------|
| 0.x | Alpha | Feb 2026 |
| 1.x | Stable | Feb 2027 |
| 2.x | Legacy | Feb 2028 |

Minstens 2 major versions gleichzeitig supported.

### Archiving Plan

Mocht dit project ooit gestopt worden:

✅ Code blijft openbaar op GitHub
✅ Archive mode: No new features, security fixes enkel
✅ Transfer to community fork (optioneel)
✅ Migrations guide naar alternatief

---

## Compliance & Legal

### GDPR Data in Public Code

❌ **Nooit in git**: Privacy keys, API tokens, personal data
✅ **Obfuscated**: Example data met dummy BSNs
✅ **Redacted**: Migrations met generieke data
✅ **Randomized**: Test fixtures

### Patent & Trademark

**Patents**: Contributors grants perpetual license; geen submarine patents.

**Trademarks**: 
- "Schulddienstverlening Register" = VNG Realisatie
- Jij mag gebruiken ter identificatie ("fork van...")
- Enkel credit nodig, geen approval

### Dispute Resolution

Geschillen over open source:

1. Discussie in issue/PR
2. BDFL decision
3. Escalatie naar VNG governance council
4. Legal arbitration (ECLI, Netherlands)

---

## Getting Started as Contributor

```bash
# 1. Fork https://github.com/gemeentearnhem/schulddienstverlening-register

# 2. Clone your fork
git clone https://github.com/YOUR-USERNAME/schulddienstverlening-register.git
cd schulddienstverlening-register

# 3. Add upstream remote
git remote add upstream https://github.com/gemeentearnhem/schulddienstverlening-register.git

# 4. Create branch
git checkout -b feature/myfeature

# 5. Make changes
# ... (code, tests, docs)

# 6. Commit & push
git commit -m "feat: add new endpoint"
git push origin feature/myfeature

# 7. Open PR on GitHub
# https://github.com/gemeentearnhem/schulddienstverlening-register/compare
```

---

## Resources

- [EUPL Official](https://www.eupl.eu/)
- [Open Source Initiative](https://opensource.org/)
- [Free Software Foundation](https://www.fsf.org/)
- [GitHub Open Source Guide](https://opensource.guide/)
- [Contributor Covenant](https://www.contributor-covenant.org/)

---

**Status**: ✅ EUPL-1.2, actively maintained, community welcome!
