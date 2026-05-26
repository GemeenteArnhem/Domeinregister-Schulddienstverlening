# Bijdragen aan Schulddienstverlening Register

We waarderen bijdragen van iedereen! Dit document beschrijft hoe je kunt bijdragen.

## Voordat je begint

- Lees onze [CODE OF CONDUCT](CODE_OF_CONDUCT.md)
- Bekijk bestaande [issues](../../issues) en [pull requests](../../pulls)
- Controleer [SECURITY.md](./SECURITY.md) voor veiligheidsrapportage

## Bijdrageproces

### 1. Setup loctale omgeving

```bash
# Fork & clone
git clone https://github.com/YOUR-USERNAME/schulddienstverlening-register.git
cd schulddienstverlening-register

# Maak branch aan
git checkout -b feature/your-descriptive-name
# of voor bugfixes:
git checkout -b bugfix/issue-description
```

### 2. Maak je wijzigingen

**Voor documentatie**:
- Update markdown bestanden in `docs/`
- Zorg voor duidelijke referenties naar GGM objecttypen
- Test markdown syntax lokaal

**Voor code/OpenAPI**:
- Voeg tests toe
- Update OpenAPI spec
- Zorg voor backward compatibility

### 3. Commit & Push

```bash
# Commit met duidelijk bericht
git commit -m "feat: add schuldhulpmaatregel endpoint"
# of
git commit -m "fix: correct schema validation"

# Push naar je fork
git push origin feature/your-descriptive-name
```

**Commit message format**:
- `feat:` Nieuwe feature
- `fix:` Bugfix
- `docs:` Documentatie wijziging
- `refactor:` Code restructuring
- `perf:` Performance improvement
- `test:` Tests toevoegen/verbeteren

### 4. Pull Request

1. Open PR tegen `main` branch
2. Beschrijf wat je hebt veranderd en waarom
3. Link gerelateerde issues (betreffen #123)
4. Wacht op review van maintainers

### 5. Code Review

- Minimaal 2 approvals van maintainers nodig
- CI/CD tests moeten slagen
- Resolveconflicten met main branch

### 6. Merge

Nadat review compleet is, zal maintainer PR mergen.

## Design Principles

Zorg dat je bijdrage volgt aan:

✅ **Nederlandse API Design Rules v3.0**
- RESTful endpoints
- Correct HTTP status codes
- Versioning strategie

✅ **Common Ground Principes**
- API-first design
- Gestandaardiseerde velden (GGM)
- Geen vendor lock-in

✅ **NERD-leidraad**
- Versleuteling in transit (HTTPS)
- Logging van gevoelige operaties

✅ **Open Source Best Practices**
- Duidelijke documentatie
- Backward compatibility waar mogelijk
- Tests voor nieuwe functionaliteit

## Coding Standards

### OpenAPI spec
- Valide OpenAPI 3.1 YAML
- Alle endpoints gedocumenteerd met examples
- Security schemes correct geconfigureerd

### Documentatie
- Markdown syntax
- GGM referenties waar passend
- Code examples waar nodig

## Vragen?

- **Issues**: Gebruik GitHub Issues voor bugs/features
- **Discussions**: Voor vragen of design discussions
- **Security**: Zie [SECURITY.md](./SECURITY.md)

## Pull Request Template

```markdown
## Beschrijving
Kort op wat je hebt veranderd.

## Type wijziging
- [ ] Bug fix
- [ ] Nieuwe feature
- [ ] Breaking change
- [ ] Documentatie update

## Gerelateerde issues
Betreffen #(issue nummer)

## Testing
Hoe heb je dit getest?

## Checklist
- [ ] Mijn code volgt de stijlrichtlijnen
- [ ] Ik heb tests geschreven/geupdate
- [ ] Ik heb documentatie geupdate
- [ ] Ik heb geen nieuwe warnings geïntroduceerd
- [ ] Mijn wijzigingen hebben geen breaking impact (tenzij MAJOR)
```

## Licentie

Door bij te dragen, ga je akkoord dat je bijdrages onder EUPL-1.2 licentie worden gepubliceerd.

Dank voor je bijdrage! 🙏
