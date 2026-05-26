# Objecttypen

Dit document beschrijft alle kernobjecttypen in het Schulddienstverlening Register.

Referentie: GGM Schulddienstverlening v2024

---

## Burger

**Omschrijving**: Natuurlijke persoon die schuldhulp nodig heeft.

**Herkomst**: BRP (Basisregistratie Personen) - via koppeling

**Attributen**:

| Attribuut | Type | Lengte | Verplicht | Opmerking |
|-----------|------|--------|-----------|-----------|
| bsn | String | 9 | Ja | Unieke identifier, BSN |
| voornamen | String | 200 | Ja | Officiële voornamen |
| achternaam | String | 200 | Ja | Officiële familienaam |
| geboortedatum | Date | - | Ja | ISO 8601 format |
| geslacht | Enum (M/V/X) | - | Nee | Geslacht |
| adres_id | UUID | - | Ja | FK naar BAG adres |
| email | String | 255 | Nee | Primair contact |
| telefoonnummer | String | 20 | Nee | Mobiel/vast |
| preferred_contact | Enum | - | Nee | EMAIL / TELEFOON / POST |
| opmerking_contactgegevens | String | 500 | Nee | Bijzonderheden contact |
| created_at | DateTime | - | Ja | Systeem gegenereerd |
| updated_at | DateTime | - | Ja | Systeem gegenereerd |
| deleted_at | DateTime | - | Nee | Soft delete |

**Constraints**:
- BSN moet valide zijn (11-proef)
- Leeftijd ≥ 16 jaar (schuldhulpverlening)
- Email xor telefoonnummer moet ingevuld zijn

---

## Schulddossier

**Omschrijving**: Centraal registratieobject; de collectie van alle schuld- en hulpverleningsgegevens voor 1 burger.

**Lifecycle**: NIEUW → ACTIEF → AFGEROND → GESLOTEN

**Attributen**:

| Attribuut | Type | Verplicht | Opmerking |
|-----------|------|-----------|-----------|
| id | UUID | Ja | PK, gegenereerd |
| burger_id | String(9) | Ja | FK naar Burger (BSN) |
| status | Enum | Ja | NIEUW / ACTIEF / AFGEROND / GESLOTEN |
| intake_datum | Date | Ja | Wanneer intake plaatsgevonden |
| intake_hulpverlener_id | UUID | Ja | FK naar Schuldhulpverlener (initiator) |
| dossier_verantwoordelijke_id | UUID | Ja | FK naar Schuldhulpverlener (leading) |
| totaal_schuld_bedrag | Decimal(12,2) | Ja | Calculated: SUM(schuld.bedrag) |
| principale_schuldeiser | String | Nee | Grootste vordering |
| beschrijving | Text | Nee | Intake notes |
| created_at | DateTime | Ja |  |
| updated_at | DateTime | Ja |  |
| archived_at | DateTime | Nee | Archief datum (enkel GESLOTEN)  |

---

## Schuldhulpverlener

**Omschrijving**: Organisatie of afdeling die schuldhulp verleent.

**Attributen**:

| Attribuut | Type | Verplicht | Opmerking |
|-----------|------|-----------|-----------|
| id | UUID | Ja | PK |
| organisatie_naam | String | Ja | Officiële naam |
| kvk_nummer | String(8) | Ja | Kamer van Koophandel nummer |
| website | String | Nee | https://... |
| email | String | Ja | Organisatie email |
| telefoonnummer | String | Ja | Hoofdnummer |
| adres_id | UUID | Ja | FK naar BAG |
| specialisaties | JSON(array) | Ja | SCHULDBEMIDDELING, BUDGETBEHEER, etc. |
| accreditatie_datum | Date | Ja | Wanneer erkend door gemeente |
| accreditatie_vervaldatum | Date | Nee | Wanneer herkeuring nodig |
| accreditatie_gemeenten | Array | Ja | Gemeenten waar erkend (multi-tenant) |
| bereik | String | Nee | Landelijk / Regio / Gemeente |
| status | Enum | Ja | ACTIEF / PAUZE / INACTIEF |
| created_at | DateTime | Ja |  |
| updated_at | DateTime | Ja |  |

---

## Schuld

**Omschrijving**: Individuele vordering (schuld) die onderdeel is van een schulddossier.

**Attributen**:

| Attribuut | Type | Verplicht | Opmerking |
|-----------|------|-----------|-----------|
| id | UUID | Ja | PK |
| dossier_id | UUID | Ja | FK naar Schulddossier |
| schuldeiser_naam | String | Ja | Wie heeft schuld vorderingen? |
| schuldeiser_id | UUID | Nee | FK naar Schuldeiser (master data) |
| bedrag | Decimal(12,2) | Ja | Origineel schuldbedrag |
| bedrag_afgelost | Decimal(12,2) | Nee | Reeds afgelost bedrag |
| soort_schuld | Enum | Ja | HYPOTHEEK / CONSUMENTENKREDIET / BELASTING / HUUR / ZIEKENHUIS / OVERIGE |
| status | Enum | Ja | OPEN / BETWIST / VOORBIJ / INGEVORDERD / KWIJTGESCHOLDEN / BETAALD |
| aanschrijfdatum | Date | Nee | Waarop schuldeiser aanschrijving gedaan |
| vervaldatum_origineel | Date | Nee | Originele vervaldatum schuld |
| oorzaak_ontstaan | String | Nee | Hoe schuld ontstaan (werkloosheid, ziekte) |
| created_at | DateTime | Ja |  |
| updated_at | DateTime | Ja |  |

---

## Schuldhulpmaatregel (Afspraak)

**Omschrijving**: Interventie/maatregel tussen schuldhulpverlener en burger.

**Attributen**:

| Attribuut | Type | Verplicht | Opmerking |
|-----------|------|-----------|-----------|
| id | UUID | Ja | PK |
| dossier_id | UUID | Ja | FK naar Schulddossier |
| schuldhulpverlener_id | UUID | Ja | FK naar Schuldhulpverlener |
| type | Enum | Ja | Zie 02-domeinbegrippen |
| status | Enum | Ja | GESLOTEN / ACTIEF / OPGESCHORT / AFGEROND |
| start_datum | Date | Ja | Wanneer gestart |
| einddatum_gepland | Date | Nee | Wanneer geëindigd gepland |
| einddatum_werkelijk | Date | Nee | Wanneer werkelijk eindigd |
| beëindiging_reden | Enum | Nee | AFGEROND_SUCCESVOL / VOORTIJDIG_BURGER / VOORTIJDIG_HULPVERLENER / ANDERE |
| doelstelling | Text | Ja | Wat willen we bereiken |
| beschrijving | Text | Nee | Details van maatregel |
| created_at | DateTime | Ja |  |
| updated_at | DateTime | Ja |  |

---

## Document (Bijlage)

**Omschrijving**: Ondersteunend bewijsmateriaal (PDF, scan, etc.)

**Attributen**:

| Attribuut | Type | Verplicht | Opmerking |
|-----------|------|-----------|-----------|
| id | UUID | Ja | PK |
| dossier_id | UUID | Ja | FK naar Schulddossier |
| maatregel_id | UUID | Nee | FK naar Schuldhulpmaatregel (optional) |
| soort_document | Enum | Ja | SCHULDOVERZICHT / OVEREENKOMST / INKOMSTEN / BETALINGSPLAN / JURIDISCH / OVERIGE |
| bestandsnaam | String | Ja | Originele filename (GDPR) |
| file_path | String | Ja | S3 bucket path (geëncrypteerd) |
| file_size | Integer | Ja | Bytes |
| mime_type | String | Ja | application/pdf, image/jpeg, etc. |
| upload_datum | DateTime | Ja | Wanneer geupload |
| uploader_id | UUID | Ja | FK naar Schuldhulpverlener (wie uploaded) |
| zichtbaar_voor_burger | Boolean | Ja | Mag burger dit zien? |
| created_at | DateTime | Ja |  |

---

## Audit Trail

**Omschrijving**: Immutable log van allee wijzigingen.

**Attributen**:

| Attribuut | Type |
|-----------|------|
| id | UUID |
| timestamp | DateTime |
| actor_type | Enum (USER / SYSTEM) |
| actor_id | UUID |
| entity_type | Enum (DOSSIER / SCHULD / MAATREGEL) |
| entity_id | UUID |
| action | Enum (CREATE / UPDATE / DELETE / VIEW / EXPORT) |
| old_value | JSON (nullable) |
| new_value | JSON (nullable) |
| ip_address | INET |
| user_agent | String |

---

## Statistiek/Rapportage

**Omschrijving**: Geaggregeerde view voor beleidsmakers (anoniem!).

**Berekinde velden** (met caching):

- Totaal dossiers (per gemeente, status, leeftijdscategorie)
- Gemiddelde schuldbedrag
- Succes rate per maatregel type
- Gemiddelde duratie hulpverlening
- Schuldeiser categorieën (% per soort)

---

## Relaties schema

```
Burger ──1──[maakt]──N─→ Schulddossier
                             │
                       ├──1──[bevat]──N─→ Schuld
                       ├──1──[krijgt]──N─→ Schuldhulpmaatregel ──M──[gecoördineerd door]──M─→ Schuldhulpverlener
                       └──1──[verzamelt]──N─→ Document
```

---

**Document version**: 1.0 | **Last update**: Mei 2026
