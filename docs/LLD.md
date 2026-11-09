# Climax — Low Level Design / meetregister
Status: ontwerpnotities, geen kant-en-klare productie-YAML — 2026-11-07

## Vastgestelde entiteiten
- `climate.living_room_toon_thermostat` (Toon Climate 2.0.5).
- `sensor.cv_power` (CV-smartplug, W).
- `switch.pomp_new` (vloerverwarmingscirculatiepomp).
- Toekomstig: slaapkamer-vraagstatussen en TRV-entiteiten, **nog niet bekend**; géén entity-ID's verzinnen.

## Toon Climate: gedocumenteerd en geobserveerd
- `hvac_modes`: `heat`, `auto`.
- `preset_modes`: `none`, `away`, `home`, `comfort`, `sleep`, `eco`.
- `current_temperature` en `temperature`; veranderen van setpoint via HA getest.
- `hvac_action`: `heating` / `idle`; **geen pompstatus**.
- `burner_info`: 0 uit, 1 CV, 2 warmtapwater, 3 voorverwarmen (betekenis uit Toon Climate-documentatie).
- `modulation_level`: gerapporteerde modulatie, soms 0 terwijl `burner_info=1`.
- `current_internal_boiler_setpoint`: interne gerapporteerde keteltemperatuurinstelling; uitlezen ≠ direct instellen.
- `ot_comm_error`: bij metingen 0.
- `program_state`: 1 normaal schema, 2 tijdelijke override waargenomen.
- `next_setpoint`, `next_state`, `next_switch_time`.

## Vastgelegde praktijksamples (niet volledige gesynchroniseerde tijdreeks)
| Proef | Toon HVAC | burner_info | Modulatie | Vermogen | Opmerking |
|---|---|---:|---:|---:|---|
| CV-vraag | heating | 1 | 0% gemeld | eerder 60-80 W bereik | Werkelijke branderstatus met modulatie 0 niet gegarandeerd |
| Woonkamer tijdelijk idle | idle | 0 | 0% | **60 W gelijktijdig** | Keteldisplaycode **6** waargenomen; vermoedelijke pompnaloop |
| Later in rust | idle | 0 | 0% | **0 W** | Displaycode toen niet vastgelegd |
| Warm tapwater | idle | 2 | **69%** | geen strikt gelijktijdige waarde | CSV rond test had 90-95 W piek; tijdkoppeling niet bevestigd |
| Na tapwater | heating | 1 | 0% | niet gelijktijdig vastgelegd | CV-status hervat |

Verdere metingen: `current_internal_boiler_setpoint` ca. 29-34 °C bij idle, 50 °C na tapwater, 52 °C tijdens tapwater, 76 °C in een eerdere verwarmingsmomentopname. Niet afleiden dat dit gemeten aanvoertemperatuur is.
Smartplug-CSV vertoonde korte 0 W-registraties tussen hogere waarden; een enkele 0 W is niet genoeg om stilstand vast te stellen.
Check vóór harde interpretatie welke exacte Avanta-uitvoering en displaycode-definities gelden.

## Pompbeslistabel — concept, nog niet implementeren
| Slaapkamer-CV-vraag | Woonkamer-CV-vraag | Ketel/transport | Pomp |
|---|---|---|---|
| JA | ieder | ieder | UIT (hoogste prioriteit) |
| NEE | JA | CV-water beschikbaar | AAN |
| NEE | NEE | gevalideerde naloop van eerder woonkamerbedrijf | AAN zolang nuttig |
| NEE | NEE | uitsluitend tapwater / rust | UIT |
| onbekend | ieder | ieder | veilige fallback nog ontwerpen |

Belangrijk: woonkamerwarmtevraag op zich garandeert geen CV-circulatie; tapwaterprioriteit kan deze tijdelijk onderbreken. Bestaande pompautomatiseringen zijn vooralsnog leidend en worden niet vervangen door deze tabel.

## Nog te ontwerpen
- Bron voor betrouwbare booleaan `slaapkamer_cv_vraag`, aggregatie bij meerdere slaapkamers; hysterese tegen pendelen.
- Toestand/history van vorige actieve zone voor pompnaloop (zo minimaal mogelijk, zonder nieuwe datasets).
- Fail-safe bij `unknown` / `unavailable`; keuze vanuit veiligheid en comfort.
- Herstel van het Toon-programma na externe setpoint-override.
- Bij eigen CV-vraag vanuit slaapkamer: toets of Toon-setpointregeling voldoende werkt of directe ketel-API nodig is.
- Tijdsmetingen bij tapwater->CV-omschakeling, pompstop, OpenTherm-storingen.
