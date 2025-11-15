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

The life cycle of a mobility unit is a continuous sequence of `DEPARTED → ARRIVED` events.
The first `ARRIVED` event indicates that the mobility unit is present at its initial location at the start of the simulation.
Each `DEPARTED → ARRIVED` pair represents a single trip segment along its route.

```
ARRIVED → DEPARTED → ARRIVED
        → DEPARTED → ARRIVED
        ... (repeat for each segment)
```

A mobility unit may serve multiple users along its route, but the events only track its movement between locations, not the individual user actions.





