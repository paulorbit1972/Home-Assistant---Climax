# Climax — System Architecture Document (SAD)
Status: onderzoeksbaseline / versie 0.1 — 2026-11-07

## Doel en systeemgrens
Climax beoogt zonegestuurde klimaatregeling met Home Assistant als coördinator en Toon 2 als bestaande thermostaat en OpenTherm-weg naar de CV-ketel. De huidige warmteregeling blijft behouden totdat elke uitbreiding afzonderlijk is geaccepteerd.

## Feitelijke installatie (geverifieerd door gebruiker en live-metingen)
- HA Core 2026.11.0 als VM op UGREEN DXP2800.
- ZHA en Sonoff Zigbee Dongle Plus MG24, geen operationele Zigbee2MQTT.
- Rooted Toon 2, Toon Climate 2.0.5, entity `climate.living_room_toon_thermostat`.
- Remeha Avanta, communiceert via OpenTherm en bestaande Toon-ketelmodule. Exact model/typeplaat nog niet vastgelegd.
- Toon Climate setpoint lezen en wijzigen vanuit HA succesvol getest.
- HA-ketelvermogen: `sensor.cv_power` via smartplug.
- HA-vloerverwarmingspomp: `switch.pomp_new`, bestaande periodieke AAN/UIT-automatiseringen.
- Andere vertrekken hebben radiatoren; Zigbee-TRV's zijn nog niet gekozen/geïnstalleerd.
- IR-panelen in slaapkamers bestaan, worden vervangen; airco's woonkamer/loft gepland.

## Bestaande verbindingen
HA -> LAN / Toon Climate -> gerootte Toon 2 -> Toon-ketelmodule -> OpenTherm -> Remeha Avanta.
HA -> Zigbee of schakelverbinding -> `switch.pomp_new` -> vloerverwarmingscirculatiepomp.
HA <- smartplug -> `sensor.cv_power`.

## Niet-doelen voor eerste fase
- Geen vervanging van de Toon-ketelmodule / geen nieuwe OpenTherm Gateway.
- Geen centrale regelkleppen, hydraulische herbouw of ongeteste CV-aansturing.
- Geen aanpassingen aan de stabiele Jack-S repository of runtime.
- Geen dashboard- of energietariefoptimalisatie voordat betrouwbare basiszones bestaan.

## Risico's en beperkingen
- Uitschakelen van alleen de vloerverwarmingspomp is **geen** hydraulische afsluiting van de woonkamer.
- Slimme TRV's mogen niet de keteldoorstroming volledig blokkeren; hydraulische voorziening/bypass en ketelminimumflow moeten eerst worden beoordeeld.
- Toon Climate rapporteert `hvac_action` en `burner_info`, maar geen bewezen rechtstreekse CV-pompstatus.
- 0 W uit de smartplug trad ook kortstondig op tussen hogere metingen: niet als enkelvoudig uitschakelbewijs gebruiken.
- Als HA, LAN of Zigbee niet beschikbaar is moet de basisverwarming veilig functioneren; fail-safe nog te ontwerpen.
- Warmwatervraag krijgt ketelprioriteit; CV-regeling mag tapwaterbedrijf niet verstoren.

## Provenance / bewijsniveau
**Bevestigd:** lokale API en HA-besturing; Toon-telemetrie bij CV en tapwater; 60 W bij `idle`/`burner_info=0` met keteldisplaycode 6; later 0 W bij zelfde Toon-melding.
**Waarschijnlijk maar niet hard bewezen:** die 60 W is specifiek ketelcirculatie/pompnaloop. Displaycode 6 werd gezien; betekenis moet nog tegen exacte type/servicehandleiding worden gecontroleerd.
**Open:** root-API voor directe onafhankelijke warmtevraag; compatibiliteit met Versatile Thermostat; gedrag bij uitval, pompnaloop en gelijktijdige zonevragen.

## Referenties
- Toon Climate: https://github.com/cyberjunky/home-assistant-toon_climate
- Toon BoilerStatus: https://github.com/cyberjunky/home-assistant-toon_boilerstatus
- Versatile Thermostat: https://github.com/jmcollin78/versatile_thermostat
- Zoned Heating (referentie, onderhoudsstatus controleren): https://github.com/nielsfaber/zoned-heating
