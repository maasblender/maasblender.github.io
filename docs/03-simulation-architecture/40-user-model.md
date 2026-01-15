---
sidebar_position: 40
title: User Model
---

The User Model is the component responsible for representing user behavior as an autonomous decision-making entity within MaaS Blender.

## Events

| Consumed Events | Emitted Events |
|-----------------|----------------|
| `DEMAND`        | `RESERVE`      |
| `RESERVED`      | `DEPART`       |
| `DEPARTED`      |                |
| `ARRIVED`       |                |


## Role and Responsibilities

The User Model governs how a user reacts to travel intentions, evaluates available options, 
and interacts with mobility services through events.

Its primary responsibilities are:

- Receiving travel intentions as `DEMAND` events
- Evaluating route candidates based on internal decision logic
- Selecting a route and reserving the required mobility services
- Managing the travel lifecycle by reacting to events emitted by mobility services  
  (For details of the travel lifecycle, refer to [User Event Cycle](../02-simulation-model/11-unit-life-cycle.md).)

## Demand Handling and Route Selection

The User Model begins its process upon receiving a `DEMAND` event, 
which represents the user’s intention to travel from an origin to a destination.

Based on the demand, the User Model evaluates multiple route candidates using its internal logic.
Route candidates are typically computed using a Route Planner and then scored/filtered by the User Model.
The evaluation strategy is pluggable and implementation-dependent. 
For example, the simplest logic may select the route with the shortest arrival time,
while more advanced models may consider cost, transfers, or user preferences.

Once a route candidate is selected, the User Model proceeds to reserve the mobility services required for that route.

## Reservation Flow

To reserve a mobility service, the User Model emits a `RESERVE` event.
This event is consumed by the corresponding mobility service.

Afterward, the User Model observes the `RESERVED` event emitted by the mobility service:

- If the reservation fails, the User Model selects an alternative route candidate and retries the reservation process.
- If the reservation succeeds and the user is able to start the trip, the User Model emits a `DEPART` event.

## Trip Execution and Event Monitoring

During the trip, the User Model may need to observe additional events emitted by mobility services, such as:

- `DEPARTED` events, indicating that the mobility service has actually started
- `ARRIVED` events, indicating that the user has reached a stop or destination

This monitoring is especially important in the following cases:

- Transfers between mobility services
  When switching to another service, the User Model must issue a new `DEPART` event for the following leg of the journey.

- Mobility services that do not support _advance_ reservations
  In such cases, the User Model must wait until the user actually arrives at the service location before emitting a `RESERVE` event.

Through this event-driven interaction, the User Model continuously adapts the user’s behavior to the availability
and state of mobility services throughout the trip.
