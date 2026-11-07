# Climax — Werkinstructie
Versie 0.1 / 2026-11-07 / uitsluitend onderzoeks- en ontwerpbasis

## Rollen
- Gebruiker: beslist over wijzigingen; voert zelf HA-configuratie, SSH, synchronisaties, configcheck, herstarts en live-acceptatie uit.
- Assistent: technische sparringpartner, systeemarchitect, reviewer; maakt onderscheid tussen feit, hypothese en te testen.
- Codex: implementeert toekomstige softwarewijzigingen via GitHub PR's na afgesproken scope.

## Afspraken
1. Eén concrete wijziging of controle tegelijk; kort en direct uitvoerbaar.
2. Eerst bestaande HA/Toon/ZHA-integraties onderzoeken; geen custom component zonder noodzaak.
3. Voor YAML-wijzigingen de **volledige** relevante YAML aanbieden en impact toelichten.
4. Elke nieuwe wijziging in aparte feature branch met PR, testplan, acceptatie en terugvalplan.
5. Geen willekeurige PR-HEAD naar HA synchroniseren; alleen expliciet beoordeelde/gepinde versie.
6. De huidige pompautomatiseringen (AAN elke 10 min, UIT elke 5 min) zijn werkend en worden niet zonder acceptatie aangepast.
7. Jack-S Smart Energy V4 1.0.8 (andere repo en HA-runtime) blijft ongemoeid.
8. Niet sturen op vermoedens uit afzonderlijke sensormomentopnames; samenhang en temporele logging vastleggen.

## Huidige toestand
- Toon Climate 2.0.5 actief; `climate.living_room_toon_thermostat` kan lokaal gelezen en via HA ingesteld worden.
- CV-smartplug `sensor.cv_power`; vloerverwarmingspomp `switch.pomp_new`.
- Rooted Toon API `/happ_thermstat?action=getThermostatInfo` bevestigd.
- Rooted Toon2 API `/tsc/sensors` bevestigd, waarden temperatuur, luchtvochtigheid, TVOC, eCO2 (geen directe CO2-meting), intensiteit.
- Bedieningsmogelijkheden voor **directe** ketelwarmtevraag via een aparte OpenTherm-write-API niet bevestigd.
- Voor actuele open punten zie SAD, HLD en LLD.

## Acceptatieritme
1. Scope + risico + exact doel vastleggen.
2. Bestaande entiteiten, configuratie, bronnen en versies inspecteren.
3. Minimale oplossing op papier, met stop-/fallbackcondities.
4. Codex PR / review op testbaarheid en regressies.
5. Door gebruiker gecontroleerde HA-sync, Configuration Check, herstart indien vereist.
6. Live-tests: woonkamer vraag, slaapkamer vraag, beide, geen vraag, tapwater, pompnaloop, HA/Zigbee/Toon offline.
7. Alleen na aantoonbare acceptatie merge/release; rollbackprocedure bewaren.

## Eerste onderzoeksopdracht
- Exacte Remeha Avanta-variant en relevante ketelcodes controleren.
- In rooted Toon documentatie/communities zoeken naar **onafhankelijke** write-aansturing via OpenTherm, niet alleen kamer-setpoint.
- Versatile Thermostat en andere bestaande oplossingen voor verzamelde warmtevraag evalueren.
- Fysieke CV-doorstroming/bypass en gevolgen van gesloten TRV's controleren.
- Eén ZHA-slaapkamerzone pas inrichten na keuze van geschikte kraan en meetstrategie.

Geen van bovenstaande werkzaamheden is op dit moment een opdracht om de werkende HA-installatie aan te passen.
