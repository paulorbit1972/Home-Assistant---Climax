# Climax — High Level Design (HLD)
Status: voorstel / niet geïmplementeerd — 2026-11-07

## Ontwerpprincipes
1. Gebruik HA als centrale zonecoördinator; behoud Toon als fysieke woonkamerthermostaat en bestaande OpenTherm-verbinding.
2. Maak logisch onderscheid tussen **CV-warmteproductie** en **vloerverwarmingspomp**.
3. Een actieve **slaapkamer-CV-warmtevraag blokkeert de vloerverwarmingspomp**; dit is de enige voorlopig afgesproken afgifteregeling. Geen extra motorische kleppen.
4. Pompnaloop na woonkamerverwarming mag waar zinvol blijven werken; pompnaloop na slaapkamerverwarming activeert woonkamerpomp niet.
5. Warmtapwaterbedrijf zonder CV-circulatie mag de vloerverwarmingspomp niet inschakelen.
6. Ontwikkel minimale wijzigingen met expliciete tests, versiebeheer en rollback.

## Logische functies
- **Zonevraag**: per ruimte doeltemperatuur, gemeten temperatuur, beschikbaarheid en gekozen verwarmingsbron.
- **Bronkeuze (later)**: CV versus IR-panelen of airco, met comfort als eerste eis; eventueel energieprijzen.
- **CV-vraag**: verzamelt warmtevraag voor radiatoren/vloerverwarming. De ketel-aansturing via Toon-setpoint werkt, maar onafhankelijke ketelvraag is nog te onderzoeken.
- **Pompbeslissing**: scheidt woonkamerafgifte van overige CV-zones; slaapkamerblokker gaat vóór woonkamer-aanvraag en vóór verbruikssignalen.
- **Bewaking**: Toon-state, smartplugvermogen, communicatiefouten, zonestatussen, uitvalscenario's.

## Concept van exclusieve zoneprioriteit
Als een of meer slaapkamers CV-warmte vragen, woonkamerpomp UIT. Als geen slaapkamer CV-warmte vraagt, dan alleen woonkamerpomp AAN als er daadwerkelijk woonkamer-CV-vraag of aantoonbaar nuttige naloop vanuit woonkamerbedrijf is én CV-circulatie mogelijk is.
Hiermee voorkomen we het **actief** voeden van de vloer via de pomp; passieve doorstroming is niet uitgesloten.

## Afhankelijkheid: onafhankelijke CV-vraag
Te onderzoeken of rooted Toon 2, naast wijzigen van het kamer-setpoint, rechtstreeks ketelvraag of OpenTherm-setpoint kan geven. Geen bewezen write-API hiervoor. Alternatieven: tijdige setpoint-override met herstel / bestaande zoneregelintegratie / eventuele OpenTherm Gateway. Deze keuze is **niet genomen**.

## Fasering
0. Documentatiebaseline en risico-inventarisatie (deze PR).
1. Onderzoek root-API/OpenTherm, actuele bestaande integraties en keteltype; geen wijzigingen.
2. Eén slaapkamer als proefzone (ZHA TRV + geschikte kamertemperatuurmeting); alleen monitoren.
3. Controleren van gezamenlijke CV-warmtevraag en veilig herstel van Toon-programma.
4. Toevoegen slaapkamerprioriteit aan bestaande pompautomatiseringen; behoud 10/5-min-cadans tenzij tests andere noodzaak aantonen.
5. Extra slaapkamers en robuustheidstests.
6. IR-panelen; later airco's en bronkeuze.
7. Dashboards en eventuele koppeling met Jack-S alleen via bestaande HA-entiteiten.

## Acceptatiecriteria per fase
- Feitelijke sensor/actuatorwaarden en tijden reproduceerbaar.
- Geen tapwateronderbreking, ongewenste ketelactivatie of hydraulische blokkering.
- Handmatige Toon-bediening en ketelprogramma blijven bruikbaar.
- Veilig gedrag bij `unavailable`, HA-herstart, Zigbee-storing en wegvallende Toon-verbinding.
- Geen regressie van bestaande pompautomaten en geen wijzigingen aan Jack-S.
