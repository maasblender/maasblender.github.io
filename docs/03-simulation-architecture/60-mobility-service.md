---
sidebar_position: 60
title: Mobility Service
---

A mobility service represents a transport service that can be integrated into MaaS Blender.
Examples include public transit, ride-hailing, bike sharing, car sharing, and on-demand services.

The core value of MaaS Blender lies in its ability to integrate multiple heterogeneous mobility services.
To enable this, MaaS Blender defines a minimal, event-based integration contract.
Any service that complies with this contract can participate in the simulation, 
regardless of its internal implementation or data source.

## Events

| Consumed Events | Emitted Events |
|-----------------|----------------|
| `RESERVE`       | `RESERVED`     |
| `DEPART`        | `DEPARTED`     |
|                 | `ARRIVED`      |

## Responsibilities of a Mobility Service

A Mobility Service is responsible for managing its own operational resources, such as
vehicles, drivers, passenger capacity, and any service-specific constraints.

:::info
MaaS Blender does not impose internal modeling requirements.
Each service is free to implement its own logic, as long as it respects the event semantics.
:::

## Reservation Flow (`RESERVE` → `RESERVED`)

Upon receiving a `RESERVE` event, the mobility service must determine whether it can serve the requested passenger.
This decision may consider factors such as 
vehicle availability, driver availability, maximum passenger capacity, service-specific constraints.

The service must then emit a `RESERVED` event indicating whether the reservation succeeded or failed.

:::warning
Even for services that do not require reservations in reality, the `RESERVE` / `RESERVED` interaction is mandatory.
This requirement exists to unify the interaction model across all mobility services and simplify orchestration at the User Model level.
:::

## Trip Execution Flow (DEPART → DEPARTED / ARRIVED)

Upon receiving a `DEPART` event, the mobility service initiates the actual movement of the passenger.
The service determines the actual travel duration according to its internal logic.

When the trip starts, the service emits a `DEPARTED` event.

When the passenger reaches the destination, the service emits an `ARRIVED` event.

:::warning
if the `DEPART` event is emitted before the agreed time,
the mobility service must transport the passenger.

if the `DEPART` event is NOT emitted by the agreed time,
the mobility service MUST NOT transport the passenger.
:::

:::info
If the simulation requires explicit modeling of vehicle or driver movement,
the mobility service may also emit `DEPARTED` and `ARRIVED` events for those units.
:::

## Design Rationale

By standardizing integration at the event level:

- MaaS Blender enables seamless integration of diverse mobility services.
- Mobility service developers can participate without deep knowledge of internal frameworks or data formats.
