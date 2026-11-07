# Onderzoek 01 — Werkt de Remeha Avanta modulerend?
Status: technisch onderzoek + open live-test, 2026-11-07. **Geen aanpassing aan HA of ketel.**

## Centrale conclusie
OpenTherm **relative modulation level** (ID 17) is relatief binnen het bereik van minimum- en maximummodulatie. **0% kan betekenen dat een brandende ketel op minimumlast werkt**, niet per definitie dat de brander uit is. Combineer met daadwerkelijke brander/vlamstatus. Dit nuanceert eerdere interpretaties van `modulation_level: 0` in Toon Climate.

## Bestaande metingen
- Toon Climate 2.0.5: `climate.living_room_toon_thermostat`.
- CV: `hvac_action: heating`, `burner_info: 1`, `modulation_level: 0` meerdere keren; `current_internal_boiler_setpoint` wisselt o.a. 31/50/76°C.
- Warm tapwater: `hvac_action: idle`, `burner_info: 2`, `modulation_level: 69`, setpoint 52°C.
- Tijdens Toon idle/branderinfo 0: smartplug 60 W, keteldisplay code 6; later 0 W. Oude Avanta-handleiding vermeldt code 6 voor CV-pompnaloop.
- Dit toont werkende Toon-telemetrie, **maar bewijst nog niet** hoe het brandervermogen tijdens CV-vraag verandert.

## Verificatiemethoden op volgorde
1. **Voorkeur: uitlezen op het Avanta-display, zonder parameters te wijzigen.** Bij de oorspronkelijke Avanta is gebruikersuitlezing: `t1` actuele aanvoer-°C, `t2` retour-°C, `FL` ionisatiestroom in µA en `nF` ventilatortoerental in honderden rpm. Tijdens een langere, natuurlijke CV-verwarmingsperiode `nF` op twee momenten vergelijken; gelijktijdig Toon `burner_info`, `modulation_level` en setpoint registreren. Als ventilatortoerental tijdens daadwerkelijk branden wijzigt, is dat sterk bewijs voor fysiek veranderend brandervermogen. Niet de ketel openmaken, geen serviceparameters of geforceerde vollast gebruiken.
2. **Aanvullend:** rooted Toon BoilerStatus-app en Toon BoilerStatus-integratie kunnen actuele aanvoer/retour, modulatie en setpoint beschikbaar stellen; nog niet geïnstalleerd/bewezen op deze Toon. Documenteerde lokale test-URL `http://TOON_IP/boilerstatus/boilervalues.txt` vereist eerst BoilerStatus-app op Toon. Metingen uit dezelfde OpenTherm-bron zijn geen volledig onafhankelijke bevestiging.
3. **Indien nodig:** echte gasverbruiksmeting over tijd en temperatuurtrend correleren; momentaan elektrisch ketelvermogen uit smartplug is geen maat voor gasbrandervermogen.
4. Bij twijfel over branderwerking door deskundige installateur laten controleren; niet zelf gasblok/CO₂/servicemenu aanpassen.

## Eerste uit te voeren controle
Wanneer er **natuurlijke CV-warmtevraag** is (niet tapwater), kijk of het Avanta-bedieningspaneel overeenkomt met de oude handleiding (vier knoppen: R, -, +, enter). Lees tijdens branderbedrijf `nF` af en herhaal na enkele minuten; noteer beide waarden + Toon `burner_info`/`modulation_level`. Als paneel anders is, eerst exacte typevariant/typeplaat vastleggen. Geen wijzigingen.

## Betrouwbaarheid
**Documentatie bevestigd:** OpenTherm ID17 relatieve modulatie; Avanta display nF, FL, t1, t2; ketel ondersteunt modulerende OpenTherm-regeling.
**Waarschijnlijk:** 0% bij CV was minimumlast; kan ook andere status/meet-/implementatiefactor zijn.
**Open:** werkelijke CV-toerentalverandering; exact type Avanta; Toon BoilerStatus-gegevens; eventuele grenswaarden of storingen.

## Bronnen
- Remeha Avanta Installatie- en Servicehandleiding, §2.8.2 en §2.7.2: https://remehatools.be/wp-content/uploads/documents/archive/REMEHA/Gas/Avanta%20%28Plus%29/060701%20IM%20SM%20Avanta%20NL.pdf
- OpenTherm Gateway community FAQ (0% = minimum bij brandende ketel): https://github-wiki-see.page/m/rvdbreemen/OTGW-firmware/wiki/Frequently-Asked-Questions
- Toon Climate: https://github.com/cyberjunky/home-assistant-toon_climate
- Toon BoilerStatus: https://github.com/cyberjunky/home-assistant-toon_boilerstatus
- Communitygesprek: https://www.reddit.com/r/Klussers/comments/1q9vm8e/nacht_modulatie_remeha/
