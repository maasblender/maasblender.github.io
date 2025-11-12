---
sidebar_position: 11
title: Unit Event Cycle
---

## User Event Cycle

A user's travel in the simulation is represented by a sequence of events (6 steps), from creating a demand to arriving at the destination.

```
DEMAND → RESERVE → RESERVED → DEPART → DEPARTED → ARRIVED
```

If the trip requires multiple services, steps 2–5 repeat for each segment.
Each segment corresponds to a leg of the route using a specific mobility service.

```
DEMAND → RESERVE → RESERVED → DEPART → DEPARTED → ARRIVED
       → RESERVE → RESERVED → DEPART → DEPARTED → ARRIVED
       ... (repeat for each segment)
```

## Mobility Event Cycle

A mobility unit (e.g., bus, taxi, or shared vehicle) repeatedly moves between locations to serve user demands.
Unlike a user, a mobility unit does not create demands; it simply departs from one location and arrives at another.

The life cycle is a continuous sequence of DEPARTED → ARRIVED events.

```
DEPARTED → ARRIVED → DEPARTED →  ... → ARRIVED
```

A mobility unit may serve multiple user demands along its route, but its events only track movement.
