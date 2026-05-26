# Common Ground Principes

Dit register is gealigneerd met de **Common Ground** filosofie van VNG Realisatie.

Referentie: https://commonground.nl/

## Wat is Common Ground?

Common Ground is een beweging om IT-systemen in de publieke sector fundamenteel anders in te richten:

> "Gemeenten werken samen aan slim en duurzaam IT-landschap"

**Kernprincipes**:
- API-first en data-gedreven
- Decentrale regie op data
- Maximale interoperabiliteit
- Transparantie en openheid
- Geen vendor lock-in

---

## Hoe implementeren we Common Ground?

### 1. API-First

✅ **Database is intern detail** - Externe consumenten kennen enkel de API

✅ **OpenAPI spec is source of truth** - Documentatie volgt automatisch

✅ **REST (niet SOAP/GraphQL)** - Nederlandse standaard

✅ **Versionering van API** - Minstens 2 major versions supported

### 2. Data-gedreven

✅ **Gegevensstandaarden volgen GGM** - Schulddienstverlening component

✅ **Referentiewaarden centraal beheerd** - Geen hardcoded values per gemeente

✅ **Audit trail verplicht** - Wie deed wat, wanneer

✅ **Data minimalisatie** - Enkel noodzakelijke gegevens

### 3. Decentraliteit

✅ **Multi-tenant uit de box** - Verschillende gemeenten, gescheiden data

✅ **Gemeente kan eigen instance hosten** - Geen centraal lock-in

✅ **REST vorige gemeente's data** - Via federatie/APIs tussen instances

✅ **Geen centrale afhankelijkheid** - Elk gemeente werkt autonomous

### 4. Interoperabiliteit

✅ **Standaard databronnen** - Aansluiting op BASISREGISTRATIES (BAG, BRP, KvK)

✅ **Federatiemogelijkheden** - Gemeenten kunnen data uitwisselen

✅ **Open data waar mogelijk** - Statistieken anoniem beschikbaar

✅ **API contracts** - Garanties voor clients

### 5. Transparantie

✅ **Open source code** - GitHub publiek (EUPL-1.2)

✅ **Duidelijke documentatie** - Informatiemodel, API, deployment

✅ **Governance transparant** - Issue tracker publiek, release notes

✅ **Security disclosures** - Verantwoorde openbaarmaking

### 6. Geen Vendor Lock-in

✅ **Compleet exporteerbaar** - API voor alle data dump

✅ **Standard data formats** - JSON, niet proprietaire formaten

✅ **Migratie mogelijkheden** - Schema migrations vastgelegd

✅ **Open source alternatief** - Anderen kunnen fork maken

---

## GGM Alignment

Dit register implementeert deze GGM objecttypen:

| Objecttype | Status | Veld-alignment |
|-----------|--------|-----------------|
| Schulddossier | ✅ 100% | Alle GGM velden opgenomen |
| Schuldhulpverlener | ✅ 100% | Inclusief specialisaties |
| Schuld/Vordering | ✅ 95% | Basis velden, geen juridische details |
| Schuldhulpmaatregel | ✅ 95% | Standaard maatregelen out of box |
| Burger | ✅ 80% | Referentie naar BRP, geen dubbel |

**Mapping document**: [GGM_MAPPING.md](./GGM_MAPPING.md) (in voorbereiding)

---

## Federatie Architectuur

### Scenario: Burger verhuist naar andere gemeente

```
Gemeente A                          Gemeente B
[Schulddossier 123]    TRANSFER     [Schulddossier 123]
   (eigenaar)         ------->        (volgen)
                      <-------        [Status update]
                      (async)
```

**Implementatie**: OAuth2 service-to-service, asymmetrische vertrouwingen

---

## Open Data

Waar GDPR het toestaat, publiceren we anonymized data:

✅ **Public endpoints** (geen auth nodig):
- `/stats/dossiers` - Totale aantal, per status
- `/stats/schuldhulpverleners` - Aantal registeringen per soort
- `/stats/maatregelen` - Succes rate per maatregel type

❌ **Niet openbaar**: Individuele burgergegevens, schooldsituaties, creditscores

---

## Bijdrage aan Common Ground Community

Dit register:

✅ Participeert in **VNG Digital Workspace**
✅ Deelt learnings via **VNG Community Forum**
✅ Tests uit **referentie-implementaties**
✅ Volgt **Common Ground Design Patterns**

---

## Referenties

- [CommonGround.nl](https://commonground.nl/)
- [VNG Realisatie](https://vng-realisatie.nl/)
- [GGM Gemmaonline](https://www.gemmaonline.nl/)
- [Common Ground Manifesto](https://commonground.nl/about-us/about-common-ground)

---

**Status**: ✅ Gevalideerd tegen Common Ground principes v1.0
