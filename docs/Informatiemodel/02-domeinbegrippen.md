# Domeinbegrippen (Semantiek)

Alignment met GGM Schulddienstverlening.

## Schulddossier

**Definitie (GGM)**: 
Geheel van gegevens die betrekking hebben op de schuldsituatie en schuldhulpverlening 
van een natuurlijke persoon.

**In onze context**:
- Centraal object: alles draait hieromheen
- Hoort bij precies 1 burger (burgergegevens)
- Kan meerdere schuldposities hebben
- Kan meerdere maatregelen hebben
- Immutable kerngegevens (burger, intakedatum)

**Lifecycle**:
- NIEUW → ACTIEF → AFGEROND → GESLOTEN

---

## Schuldhulpverlener

**Definitie (GGM)**:
Organisatie of persoon die schuldhulp verleent.

**Voorbeelden**:
- Stadsbrede schuldhulporganisatie
- Sociaal raadsliedenbureau
- Gemeente (eigen afdeling)
- Commerciële schuldbemiddelaar

**Attributen**:
- Naam organisatie
- BSN/KvK (identificatie)
- Contactgegevens
- Specialisatie (schuldbemiddeling, budgetbeheer, insolventietraject)
- Autorisatie (welke soort schuldhulp mag zij geven?)

---

## Schuld / Schuldpositie

**Definitie (GGM)**:
Een vordering, meestal geldelijk, van een schuldeiser op de schuldenaar.

**Erbij horen**:
- **Schuldeiser**: Wie heeft geld vorderingen? (bank, overheid, bedrijf)
- **Schuldbedrag**: Hoeveel?
- **Soort schuld**: Hypotheek, consumentenkrediet, belasting, huur-achterstand, etc.
- **Status**: Actiève schuld, beëindigd, opeisbaar, etc.

---

## Schuldhulpmaatregel / Afspraak

**Definitie (GGM)**:
Actie die schuldhulpverlener en burger gezamenlijk ondernemen om schuldsituatie 
te verbeteren.

**Voorbeelden**:
- Schuldbemiddeling (onderhandelen met schuldeisers)
- Budgetbeheer (burger geeft geld beheer uit handen)
- Spaarplan (gestructureerd sparen voor schuldsaldo)
- Insolventietraject (faillissement/schuldsaneringsregeling)

**Karakteristieken**:
- Heeft startdatum en (meestal) einddatum
- Kan milestones hebben (mijlpalen)
- Moet nagekomen worden door burger (verplichtingen)

---

## Autorisatie & Verantwoordingsperspectief

**Burger-perspectief**:
- Eigenaar van eigen schulddossier
- Ziet alles (read-only in zelfserviceportaal)
- Moet toestemming geven voor bepaalde acties

**Schuldhulpverlener-perspectief**:
- Krijgt alleen toegang tot dossiers waar zij aan werken (op-dracht basis)
- Mag lezen, schrijven, updaten
- Kan documenten toevoegen

**Gemeente-perspectief**:
- Statistieken-/rapportagerechten
- Geen individuele burgergegevens (anoniem)
- Kan bepaalde schuldhulpverleners accrediteren

**Admin-perspectief**:
- Volledige beheer (zeer beperkt aantal personen)

---

## Kernrelaties (semantisch)

```
┌──────────────────────────────────┐
│    Burger (persoon)              │
│  (BSN, naam, adres, telefoonnr)  │
└────────────┬─────────────────────┘
             │ (eigenaar van)
             ▼
┌──────────────────────────────────┐
│    Schulddossier                 │
│  (status, startdatum)            │
└────────────┬──────────┬──────────┘
             │          │
        (bevat)    (gelinkt met)
             │          │
    ┌────────▼──┐   ┌───▼─────────────────┐
    │ Schuld    │   │ Schuldhulpmaatregel │
    │ (bedrag)  │   │ (type, doelstelling)│
    └───────────┘   └────────┬────────────┘
                             │ (gaat met)
                             ▼
                      ┌──────────────────┐
                      │Schuldhulpverlener│
                      │(organisatie)     │
                      └──────────────────┘
```

---

## Additionele Begrippen

### Document/Bijlage

**Definitie**: Ondersteunend bewijsmateriaal van schuldsituatie en afspraken.

**Soorten**:
- Schuldoverzicht (schuldeisers, bedragen)
- Inkomstenverklaring (burger financiën)
- Overeenkomst schuldhulp
- Betalingsplan / spaarschema

### Status Enumeraties

**Dossier Lifecycle**:
- NIEUW → ACTIEF → AFGEROND → GESLOTEN

**Schuldhulpmaatregel Types**:
- SCHULDBEMIDDELING
- BUDGETBEHEER
- SPAARPLAN
- INSOLVENTIETRAJECT
- JURIDISCHE_BEGELEIDING
- MAATSCHAPPELIJK_WERK

---

**Document version**: 1.3 | **Last update**: Mei 2026