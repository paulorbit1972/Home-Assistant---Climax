# Climax — Software Architecture Document (SAD)

Status: **Proposed architecture, not yet implemented**. Version 0.1.

## Context and boundaries
Climax is a Home Assistant custom integration, not a replacement for existing Toon, Zigbee, weather, Hue or plug integrations. It reads existing HA entities and (in future) issues HA service calls to commissioned actuators. Existing `Toon.yaml` and `Humidity.yaml` remain unchanged and authoritative during M0–M2.

## Logical components
- **Config/Zone Registry**: stable zone UUID, editable display name, optional HA area, feature configuration, device lifecycle and role bindings.
- **Entity Binding Resolver**: validate referenced entity_id, domain, capabilities, availability, and type; do not create duplicates of source entities.
- **Climate Monitor**: snapshot room/outdoor T, RH, absolute humidity, dew point, air quality; preserve measurement provenance.
- **Thermal Analytics**: passively detect cooling windows after CV demand ends; wait 15 min for residual heat; exclude renewed heating, unknown heat sources, windows/ventilation disturbances where detectable; calculate cooling slope and outside-adjusted metrics with quality flags. Results are indicators, **not** measured U/Rc values.
- **Demand Engine**: pure functions for heating, cooling, ventilation and moisture demand; no direct side effects.
- **Resource Arbiter**: resolve shared boiler, floor loop, extraction fan and cross-zone conflicts.
- **Actuator Adapters**: capability-specific HA service interfaces (climate, switch, fan, cover/valve), disabled until commissioned.
- **Safety Supervisor**: minimum flow design review, stale-data lockout, fail-safe behavior, minimum on/off, command rate limits, override precedence.
- **Diagnostics**: explain decisions, input freshness, reason codes and unavailable features.

## State and data model
Zone: `zone_id` immutable; `name` editable; `area_m2` optional; `enabled`; `role_bindings` keyed by semantic role; `devices[]` with capabilities, lifecycle and commissioned flag. Shared devices reference building-level resource IDs. Multiple zones may reference a shared sensor, but control ownership must be singular.

HA config entry and options flow should own user-editable settings; versioned migrations required. Use stable unique IDs for new Climax-created entities; **do not reuse** Toon/Humidity unique IDs. HA registry identity is distinct from a user-editable entity_id. See HA developer entity registry guidance.

## Execution
M0: no integration runtime. M1–M2: read-only and diagnostic entities only. Future control mode is explicit opt-in, default off; per-actuator commissioning and rollback. Restart/offline policy documented per device, no unverified claim of guaranteed fail-safe for external hardware.

## Non-functional requirements
No blocking I/O in HA event loop; event-driven subscriptions, bounded updates; no polling duplicate sources without reason; support unavailable/unknown; avoid logging sensitive occupancy history; keep configuration backward compatible; tests with mocked HA states; no automatic merge.

## Open technical risks
1. Toon 2 command path for demand outside Living Room is not verified.
2. Hydraulic bypass/minimum flow and floor pump behavior need physical validation.
3. External heating devices may maintain their own schedules after HA failure.
4. Air-quality sensor model, true CO2 vs eCO2 and PM capability unverified.
5. Outdoor weather values are model-derived rather than on-site measurements.
6. Metering sources for gas/electricity and Hue-on-Toon UI are not confirmed.

## Acceptance criteria for M0
Architecture, zone catalogue, existing entity mapping, proposed Climax entities and risk log reviewed. No change to running HA, no new duplicate sensors, no control actions.
