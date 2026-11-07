# Climax — Climate Management & Zone Control

**Status:** onderzoeksbaseline, geen productiecode. **Baseline:** 7 november 2026.

Climax is een zelfstandig Home Assistant-project voor zoneregeling en het stapsgewijs combineren van CV, vloerverwarming, radiatorzones, IR-panelen en toekomstige airco's. Het project is gescheiden van [Jack-S Smart Energy V4](https://github.com/paulorbit1972/Home-Assistant---Smart-Power-plan): Jack-S blijft ongewijzigd.

## Uitgangspunten
- Klein, robuust en verifieerbaar; eerst bestaande Home Assistant-integraties onderzoeken.
- Bestaande ketelregeling via gerootte Toon 2 en Toon-ketelmodule behouden.
- Eén wijziging / test tegelijk; user voert HA-configuratie, SSH en herstarts zelf uit.
- Voorlopig **geen** extra gateway, zonekleppen of eigen custom component.
- Geen hydraulische wijzigingen zonder toetsing van minimale keteldoorstroming.
- Maak onderscheid tussen feiten, hypotheses en nog te testen gedrag.

## Bevestigde werkende basis
- Home Assistant Core 2026.11.0 in VM op UGREEN DXP2800; Zigbee via ZHA en Sonoff MG24.
- Gerootte Toon 2, verbonden met Remeha Avanta via bestaande Toon-ketelmodule en OpenTherm.
- Toon Climate versie **2.0.5**: `climate.living_room_toon_thermostat`.
- API `/happ_thermstat?action=getThermostatInfo` reageert lokaal; temperatuur uitlezen én vanuit HA aanpassen live getest.
- Toon 2 omgevings-API `/tsc/sensors` werkt (nog niet als aparte sensoren ingericht).
- Vloerverwarmingspomp via `switch.pomp_new`, ketelvermogen via `sensor.cv_power`.
- Bestaande HA-automatiseringen controleren inschakelen elke 10 minuten, uitschakelen elke 5 minuten. **Niet wijzigen voordat zone-ontwerp en acceptatietests rond zijn.**

## Beoogd systeem
- Woonkamer: Toon 2, CV/vloerverwarming met bestaande schakelbare pomp; later airco.
- Slaapkamers: CV-radiatoren met toekomstige Zigbee-thermostaatkranen; later vervangende IR-panelen.
- Loft: toekomstige airco.
- **Regelwens:** als slaapkamer(s) CV-warmte vragen, wordt de vloerverwarmingspomp geblokkeerd, zelfs als de woonkamer ook warmte vraagt. Geen poging tot volledige hydraulische afsluiting.
- De CV-ketel moet op termijn op gezamenlijke zonevraag kunnen reageren; onafhankelijke ketelwarmtevraag via de root-API is **nog niet aangetoond**.

## Documenten
- [WERKINSTRUCTIE](docs/WERKINSTRUCTIE.md) — proces, werkwijze en acceptatie
- [SAD](docs/SAD.md) — feitelijke installatie, grenzen en risico's
- [HLD](docs/HLD.md) — doelarchitectuur, fasering, keuzes
- [LLD](docs/LLD.md) — entiteiten, meetresultaten, conceptbeslislogica

> Let op: de hier beschreven logica is een **ontwerp**, geen uitgerolde functionaliteit.
