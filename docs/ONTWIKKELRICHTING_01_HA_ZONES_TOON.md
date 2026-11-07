# Ontwikkelrichting 01 — HA-zones met Toon 2 als autonome CV-thermostaat

**Status:** gekozen als eerste concrete ontwikkelrichting, **nog niet geïmplementeerd**.  
**Besluit:** 8 november 2026. **Doel:** een minimaal werkend, toetsbaar prototype zonder hardwarewijzigingen.

## Kernbesluit

- **Home Assistant (Climax) beheert de zones:** per vertrek actuele temperatuur, doeltemperatuur, TRV-/beschikbaarheidsstatus, hysterese, gezamenlijke CV-warmtevraag en zoneprioriteiten.
- **Toon 2 behoudt de bestaande modulerende OpenTherm-verbinding** en blijft zelfstandig de woonkamer en CV-ketel regelen.
- De gezamenlijke **externe** CV-warmtevraag uit HA wordt via een nog te selecteren Toon-interface aangeboden. Eerst de aantoonbaar werkende Toon Climate / `happ_thermstat` setpoint-API als prototype onderzoeken; directe onafhankelijke OpenTherm-warmtevraag is **niet bewezen**.
- Zones kunnen later op Toon worden weergegeven en bediend, maar een eigen Toon-UI is **geen voorwaarde** voor het eerste prototype.
- **Slaapkamer-CV-warmtevraag blokkeert de woonkamer-vloerverwarmingspomp** (`switch.pomp_new`), ook als woonkamer tegelijk warmtevraag heeft, zoals eerder afgesproken. Pomp uit betekent niet dat de woonkamer hydraulisch volledig afgesloten is.

## Logische verdeling

```text
Zigbee TRV's / optionele kamersensoren / woonkamer (Toon)
                     |
                     v
          Home Assistant / Climax
        - zone-status en warmtevraag
        - hysterese / minimum cyclustijd
        - slaapkamerprioriteit
        - geldigheid / heartbeat
                     |
                     v
           Toon 2 (bestaande API)
        - tijdelijke externe CV-vraag
        - lokale watchdog en herstel
        - autonoom woonkamerprogramma
                     |
                     v
      bestaande Toon-ketelmodule / OpenTherm
                     |
                     v
                Remeha Avanta
```

## Cruciaal failsafe-ontwerp: uitval van HA mag geen blijvende vraag achterlaten

**Een HA-uitval annuleert de eerder ingestelde Toon-warmtevraag niet vanzelf.** Daarom hoort bij de koppeling een onafhankelijke **watchdog aan de Toon-kant** of een aantoonbaar gelijkwaardig mechanisme met een automatische vervaltijd.

Voorgestelde *prototypeparameters*, nog te testen: HA vernieuwt elke **30 seconden** een geldigheidssignaal; bij **90 seconden** zonder geldige update vervalt uitsluitend de externe zonevraag. Geen definitieve timing voordat netwerk- en rebootgedrag zijn getest.

- **HA/LAN uit:** Toon blijft de lokale woonkamer bedienen en mag blijven verwarmen volgens zijn oorspronkelijke programma; externe slaapkamerwarmtevraag vervalt.
- **Bij tijdelijke setpoint-override:** de oorspronkelijke Toon-modus, preset, programma en relevante setpointstatus moeten kunnen worden hersteld. Expiratie moet ook werken wanneer HA weg is, en na Toon-herstart geen langdurige externe warmtevraag achterlaten.
- **Toon onbereikbaar vanuit HA:** geen herhaalde onbeperkte setpointverhogingen. HA meldt de storing en stopt met ongecontroleerde sturing.
- **Zigbee-TRV of zonesensor `unknown`/`unavailable`:** geen nieuwe warmteaanvraag op basis van oude/stale data; passende veilige status/logging.
- **Herstel na HA-herstart:** actuele zonegegevens opnieuw inlezen; niet blind een verouderde warmtevraag hervatten.
- **Tapwaterprioriteit en ketelbeveiligingen** blijven altijd leidend.
- **Hydrauliek:** vóór actieve TRV-regeling minimale CV-doorstroming/bypass en consequenties van gesloten kranen controleren. Failsafe mag geen gevaarlijke gesloten-waterweg creëren.
- De bestaande pompautomatiseringen (10 minuten AAN-check, 5 minuten UIT-check) blijven **ongewijzigd** tot een afzonderlijk ontwerp en tests.

## Bestaande open-source referenties

- [lewjuh/zonal-heating](https://github.com/lewjuh/zonal-heating): virtuele kamerclimates, hysterese, anti-pendelen, diagnostiek; de huidige integratie stuurt de zonehoofdthermostaat met een **setpointverhoging**. Goed voorbeeld voor zonestatussen, maar niet zonder adapter/acceptatietests inzetten.
- [angelinid/MultiTRVHeating](https://github.com/angelinid/MultiTRVHeating): aggregatie van TRV-openingen, warmtebehoefte en prioriteit. Verwacht een rechtstreeks instelbaar OpenTherm-aanvoersetpoint via ESPHome, dat onze Toon Climate-integratie **niet** aanbiedt. Gebruik als algoritmische referentie, niet als drop-in oplossing.
- [Ierlandfan/freetoon-lvgl](https://github.com/Ierlandfan/freetoon-lvgl): onderzocht BoxTalk, Quby en Toon HTTP-API. Onderzoek naar bestaande interfaces; **niet** de alternatieve firmware/bridge op de productieve Toon installeren.

## Eerste implementatietraject (apart te autoriseren)

1. **Alleen observeren:** zonevraag uit één toekomstige slaapkamerradiator-TRV en eventueel externe kamersensor berekenen. Bestaande ketel-/pompaansturing blijft leidend.
2. **Adapter ontwerpen:** bepalen hoe gezamenlijke externe CV-vraag Toon kan bereiken. Start met bewezen kamer-setpoint-API; werk correct herstel van Toon-programma uit. De gewenste directe onafhankelijke CV-vraag blijft parallel een onderzoekspunt.
3. **Watchdog vóór aansturing:** expiratie bij verbindingsverlies onafhankelijk van HA laten werken; gecontroleerd toetsen zonder actieve ketelsturing.
4. **Beperkte proef:** één slaapkamer, gecontroleerde tijdvensters; meten van `burner_info`, `hvac_action`, temperatuur, TRV, ketelrespons en pomptoestand.
5. **Veiligheid/acceptatie:** woonkamer alleen; slaapkamer alleen; beide; geen vraag; tapwater; gesloten TRV; HA-stop; netwerkstoring; Toon-herstart; HA-herstart en herstel.
6. **Daarna pas:** slaapkamerprioriteit voor de vloerverwarmingspomp en uitbreiding naar meerdere zones.

## Beslis-/acceptatiecriteria

- Woonkamer blijft zonder HA zelfstandig normaal functioneren.
- Externe warmtevraag heeft een **harde, lokaal afdwingbare timeout** en kan Toon niet langdurig in kunstmatige verwarmingsvraag achterlaten.
- Een actief slaapkamerverzoek kan ketelwarmte verkrijgen, zonder ongewenste langdurige woonkamerverwarming; daadwerkelijk gedrag verifiëren.
- De originele Toon-gebruikersbediening/programma's blijven bruikbaar; na uitval/stop wordt oorspronkelijke modus betrouwbaar hersteld.
- Geen onveilige hydraulische toestand en geen verlies van warmtapwaterfunctionaliteit.
- Geen regressie in bestaande pompautomatiseringen en **geen wijzigingen aan Jack-S**.

**Statusconclusie:** dit is de **eerste gekozen ontwikkelrichting**, geen claim dat de Toon-koppeling of de watchdog al beschikbaar zijn. Implementatie alleen via afzonderlijke featurebranch/PR en live acceptatie.
