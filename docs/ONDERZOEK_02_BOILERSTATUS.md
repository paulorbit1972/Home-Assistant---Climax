# Onderzoek 02 — Toon BoilerStatus, API en modulatiemetingen

**Status:** live-waarnemingen vastgelegd; vervolgonderzoek nodig  
**Datum:** 7 november 2026  
**Impact:** uitsluitend documentatie; geen Home Assistant-, Toon-, ketel- of Jack-S-configuratie gewijzigd.

## Doel

Onderzoeken welke aanvullende ketelgegevens de gerootte Toon 2 via **BoilerStatus** beschikbaar stelt en of die helpen vaststellen of de Remeha Avanta daadwerkelijk moduleert.

## Toegang en huidige status

- In eerste instantie gaf `http://TOON_IP/boilerstatus/boilervalues.txt` **HTTP 404**.
- Vervolgens leverde dezelfde BoilerStatus-API valide JSON-data. De exacte tussenliggende installatie- of herstelstap is niet in dit onderzoek bevestigd; niet aannemen dat HA BoilerStatus al is geïnstalleerd.
- De JSON bevat onder meer `sampleTime`, `boilerSetpoint`, `roomTempSetpoint`, `roomTemp`, `boilerPressure`, `boilerOutTemp`, `boilerInTemp` en `boilerModulationLevel`.
- De bestaande **Toon Climate 2.0.5** via `climate.living_room_toon_thermostat` werkte reeds. Dit is een andere gegevensbron/app dan de aanvullende HA BoilerStatus-integratie.

## Onbewerkte momentopnamen

### 23:22

```json
{"sampleTime":"7-11-2026 23:22","boilerSetpoint":6,"roomTempSetpoint":16,"boilerPressure":0,"roomTemp":19.57,"boilerOutTemp":null,"boilerInTemp":null,"boilerModulationLevel":0}
```

### 23:24

```json
{"sampleTime":"7-11-2026 23:24","boilerSetpoint":64.9,"roomTempSetpoint":20.5,"boilerPressure":0,"roomTemp":19.87,"boilerOutTemp":null,"boilerInTemp":null,"boilerModulationLevel":64}
```

### 23:25

```json
{"sampleTime":"7-11-2026 23:25","boilerSetpoint":6,"roomTempSetpoint":16,"boilerPressure":0,"roomTemp":19.89,"boilerOutTemp":null,"boilerInTemp":null,"boilerModulationLevel":0}
```

**Context gebruiker:** thermostaat kortstondig handmatig op **21 °C** gezet. De API gaf tijdens de hoge-warmtevraagmeting `roomTempSetpoint: 20.5` terug. Dit verschil is niet verder onderzocht; de gerapporteerde JSON-waarde en de door gebruiker ingestelde waarde niet verwarren.

## Vergelijking

| Tijd | Kamertemperatuur | Gerapporteerd kamer-setpoint | Ketelsetpoint | Modulatie | Aanvoer/retour | Druk |
|---|---:|---:|---:|---:|---|---:|
| 23:22 | 19,57 °C | 16 °C | 6 °C | 0% | null / null | 0 |
| 23:24 | 19,87 °C | 20,5 °C | 64,9 °C | 64% | null / null | 0 |
| 23:25 | 19,89 °C | 16 °C | 6 °C | 0% | null / null | 0 |

## Bevestigde conclusies

1. De **lokale BoilerStatus-API is bereikbaar** en retourneert gegevens die veranderen na wijziging van de thermostaatvraag.
2. Tijdens de korte temperatuurverhoging veranderde `boilerSetpoint` van 6 naar **64,9** en `boilerModulationLevel` van 0 naar **64%**, waarna beide weer terugvielen.
3. De drie metingen tonen **variabele gerapporteerde OpenTherm-modulatie** in reactie op de veranderende vraag.
4. `boilerOutTemp` en `boilerInTemp` waren tijdens **alle drie** metingen `null`; `boilerPressure` bleef **0**. Deze waarden zijn daarom **niet betrouwbaar bruikbaar** voor actuele aanvoer/retour of keteldruk. Een gemelde 0 is geen bewijs van 0 bar.

## Wat niet bewezen is

- De 64% is een **gerapporteerde** modulatie. Er is geen gelijktijdig onafhankelijk gemeten ventilatortoerental of gasdebiet.
- De sprong 64% -> 0% viel samen met setpoint 20,5 -> 16 °C. Dat is **geen bewijs van terugmoduleren bij constante CV-warmtevraag**; mogelijk stopte de vraag.
- Niet met gelijktijdige `burner_info` of vlamstatus bewezen dat de 23:24-meting uitsluitend CV-branderbedrijf betrof.
- Niet vastgesteld waarom retour/aanvoer `null` zijn en keteldruk 0 blijft (ketelondersteuning, Toon-app, OpenTherm-gegevensbeschikbaarheid of parsing).
- Niet vastgesteld of Toon BoilerStatus al als afzonderlijke Home Assistant-integratie is toegevoegd. API-bereikbaarheid betekent dit niet automatisch.

## Volgende veilige test

1. Laat een **natuurlijke, langere CV-warmtevraag** plaatsvinden, met een zo constant mogelijk Toon-kamer-setpoint; vermijd tapwatergebruik gedurende de observatie.
2. Leg meerdere tijdstippen vast: `sampleTime`, `roomTempSetpoint`, `roomTemp`, `boilerSetpoint`, `boilerModulationLevel`, en gelijktijdig uit Toon Climate `hvac_action` en `burner_info`.
3. Kijk of `boilerModulationLevel` verandert **terwijl CV-warmtevraag voortduurt**.
4. Voor onafhankelijke bevestiging: zo mogelijk het **ventilatortoerental (nF)** rechtstreeks op het juiste Remeha Avanta-bedieningspaneel aflezen; eerst exact ketelmodel/paneel bevestigen. Geen ketelparameters of servicemenu-instellingen wijzigen.

## Relevante documentatie

- [Onderzoek 01 — Modulatie](ONDERZOEK_01_MODULATIE.md)
- [Toon BoilerStatus integratie](https://github.com/cyberjunky/home-assistant-toon_boilerstatus)
- [Toon Climate integratie](https://github.com/cyberjunky/home-assistant-toon_climate)

**Veiligheidsgrens:** Climax registreert deze waarden voorlopig uitsluitend voor onderzoek. Geen automatische ketelregeling op basis van ongevalideerde druk-, temperatuur- of modulatiemetingen.
