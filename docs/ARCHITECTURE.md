# Climax — Product & functional design (M0 draft)

Status: **Design only; no control enabled.** Version 0.1 — 2026-11-20.

## Goal
Home Assistant climate orchestration with editable named zones, indoor/outdoor climate, presence, heating, cooling, ventilation, air quality, humidity and energy. Living Room is Zone 1 and the central UI location; it must not be the sole source of heating demand.

## Zone catalogue
| ID | Zone | Area | Existing equipment | Planned equipment |
|---|---|---:|---|---|
| z1 | Living Room | 45.2 m² | Rooted Toon 2, front-window radiator without valve, kitchen underfloor heating with smartplug pump, air purifier and air-quality monitor | Heat/cool air conditioner; 2 Ezeefox radiator fans ordered, spare smartplug available |
| z2 | Bedroom 1 | 16 m² | Radiator without valve, 600 W IR panel | — |
| z3 | Bedroom 2 | 10 m² | Large radiator, 600 W IR panel | Smart TRV |
| z4 | Bathroom 1 | 6 m² | Hygro-controlled extraction | 400 W IR panel; temperature/humidity-based ventilation |
| z5 | Toilet 1 | TBD | Currently part of Bathroom 1 | Separate toilet and ventilation |
| z6 | Loft | 20 m² | Temperature/humidity sensors | Air conditioner for both heating and cooling |
| z7 | Bathroom 2 | TBD | Not built | IR heating and ventilation |

Existing, ordered, planned, not-built, offline and disabled are different lifecycle statuses. Planned devices must never receive commands.

## Capabilities and mapping
Zone inputs: temperature, RH, absolute humidity, dew point, CO2 (true measurement), eCO2 (estimated, separately labelled), TVOC, particulate PM1/PM2.5/PM10, occupancy and optional window contacts. Every input is optional, with per-feature prerequisites and availability handling.

Zone actuators: heat source (CV, IR, heat-pump AC), cool source, TRV, floor circulation pump, extraction damper, extraction fan, air purifier, radiator booster fan, smartplug. Multiple actuators per zone. Shared boiler and central extraction are building-level resources, not owned by one zone.

## Control priorities (future)
1. Safety, sensor validity, manual override and anti-short-cycling.
2. Presence/comfort targets, moisture/air-quality protections.
3. Shared resource arbitration and hydraulic constraints.
4. Heating/cooling/ventilation demand calculation.
5. Device commands only after separate commissioning and explicit enable.

IR is supplemental only during configurable cheap-price windows or cold nights, and only with genuine demand. AC is a full heating/cooling source once installed. Tariff and PV forecasts are optional inputs, never assumed reliable.

## Hydraulics
The front-window radiator (former convector-pit arrangement, now panel-concealed blade radiator) and Bedroom 1 radiator have no controllable valves. With floor pump off, heating may still reach the open radiators. A disabled floor circulation pump is **not** a guaranteed closed hydraulic valve. Closing Bedroom 2's future TRV redirects flow but does not ensure isolation or safe minimum boiler flow. Do not claim an open radiator is a verified bypass.

## Toon and UI
Existing Toon climate entity is the source of CV observations; Toon-to-HA currently works. HA-to-Toon zone control, OpenTherm override, consumption display and Hue UI require feasibility research and separate safety testing. Hue remains managed by HA; Toon is a possible additional interface, not a dependency.

## Delivery
M0 docs/registry; M1 passive zone configuration; M2 monitoring/thermal performance; M3 guarded heating; M4 presence; M5 humidity/ventilation; M6 AC cooling; M7 energy; M8 Toon/Hue UI. Each milestone: feature branch, tests, draft PR, no automatic merge.
