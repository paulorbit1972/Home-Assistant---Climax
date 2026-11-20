# Climax — Entity & device register (M0)

Status: **Design inventory from supplied Toon.yaml, Humidity.yaml and user-described setup.** Entity IDs are configured/expected; not all have been runtime-verified. Do not treat this file as a live HA registry export.

## Existing Toon source entities
| Role | Entity ID | Origin | Notes |
|---|---|---|---|
| Central thermostat | `climate.living_room_toon_thermostat` | Toon integration | Observed `hvac_action=heating`, `burner_info=1` |
| Living room temperature | `sensor.toon_temperature` | Toon.yaml template | Existing unique_id `toon_temperature` |
| Toon target | `sensor.toon_target_temperature` | Toon.yaml template | `toon_target_temperature` |
| Burner status | `sensor.toon_burner_status` | Toon.yaml template | idle/heating/hot_water/unknown |
| Modulation | `sensor.toon_modulation` | Toon.yaml template | percentage |
| Boiler setpoint | `sensor.toon_boiler_setpoint` | Toon.yaml template | °C |
| OpenTherm error | `sensor.toon_opentherm_error` | Toon.yaml template | diagnostic |
| Program state | `sensor.toon_program_state` | Toon.yaml template | diagnostic |
| Next setpoint | `sensor.toon_next_setpoint` | Toon.yaml template | °C |
| Next state | `sensor.toon_next_state` | Toon.yaml template | diagnostic |
| Next switch | `sensor.toon_next_switch` | Toon.yaml template | timestamp compatibility to validate |
| HVAC action | `sensor.toon_hvac_action` | Toon.yaml template | heat/idle |
| Preset | `sensor.toon_preset` | Toon.yaml template | program |
| Living room humidity | `sensor.toon_humidity` | Toon.yaml REST | Toon local /tsc/sensors |
| TVOC | `sensor.toon_tvoc` | Toon.yaml REST | not particulate measurement |
| eCO2 | `sensor.toon_eco2` | Toon.yaml REST | estimated CO2, not true CO2 |
| Light intensity | `sensor.toon_light_intensity` | Toon.yaml REST | illuminance/intensity |

## Existing Humidity source entities
| Zone/role | Entity ID |
|---|---|
| Outdoor RH | `sensor.outdoor_humidity` |
| Outdoor temperature | `sensor.outdoor_temperature` |
| Outdoor weather | `weather.front_garden_zandvoort_netherlands` |
| Z1 AH | `sensor.absolute_humidity_living_room` |
| Z2 AH | `sensor.absolute_humidity_bedroom` |
| Z3 AH | `sensor.absolute_humidity_bedroom_2` |
| Z6 AH | `sensor.absolute_humidity_loft` |
| Outdoor AH | `sensor.absolute_humidity_outdoor` |
| Indoor average AH | `sensor.absolute_humidity_indoor_average` |
| Ventilation advice | `binary_sensor.ventilation_beneficial` |

Existing ventilation advice is on when average indoor AH exceeds outdoor AH by >= 1.0 g/m³; this is a drying recommendation, not an unconditional ventilation prohibition.

## Room source mapping
| Zone | Temperature | RH |
|---|---|---|
| Z1 Living Room | `sensor.toon_temperature` | `sensor.toon_humidity` |
| Z2 Bedroom 1 | `sensor.t_h_bedroom_temperature` | `sensor.t_h_bedroom_humidity` |
| Z3 Bedroom 2 | `sensor.t_h_bedroom_2_temperature` | `sensor.t_h_bedroom_2_humidity` |
| Z4 Bathroom 1 | TBD | TBD |
| Z5 Toilet 1 | TBD | TBD |
| Z6 Loft | `sensor.loft_t_h_loft_temperature` | `sensor.loft_t_h_loft_humidity` |
| Z7 Bathroom 2 | TBD | TBD |

## Known actuator references
| Device | Entity ID | State |
|---|---|---|
| Underfloor circulation pump | `switch.pomp_new` | Reported existing; verify current ID |
| Boiler electrical power | `sensor.cv_power` | Reported existing; verify current ID |
| Loft dehumidifier | `switch.loft_dehumidifier_plug_dehumidifier_switch` | Reported existing; verify |
| Front radiator Ezeefox fans | TBD smartplug | Fans ordered, plug available |
| Bedroom 2 TRV | TBD | Planned |
| Living Room AC | TBD | Planned |
| Loft AC | TBD | Planned |
| IR panels | TBD per device | Existing; do not infer control entity |
| Central ventilation/dampers | TBD | Planned |

## Proposed *new* Climax diagnostic entities (not created yet)
Naming examples, final IDs may be allocated by HA registry:
- `sensor.climax_z1_cooling_rate` ... `sensor.climax_z7_cooling_rate` where measurable (°C/h)
- `sensor.climax_z1_temperature_delta_outdoor` etc. (°C)
- `binary_sensor.climax_heating_demand` (aggregate calculated demand only)
- `sensor.climax_zone_status` (diagnostic)
- `sensor.climax_thermal_data_quality` (diagnostic)

Stable unique_id pattern: `<immutable-zone-uuid>_<metric-key>` for zone diagnostics; building metrics use immutable building ID. Never reuse existing template/REST unique IDs or rely on editable zone names for identity.

## Verification checklist
Confirm actual entity IDs in HA Developer Tools; check current availability, physical device mapping, sensor capabilities and unique IDs; confirm smartplug ownership and manual overrides. Source entities remain owned by their original integrations.
