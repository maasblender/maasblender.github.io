---
sidebar_position: 10
title: Event Specification
---

State changes of **user** units (e.g., passengers) and **mobility** units (e.g., buses) are recorded as **events**. 
All **events** are emitted as JSON and processed chronologically.

## Common Event Structure

Every event has the following fields:

| Field       | Type           | Description                           |
|-------------|----------------|---------------------------------------|
| `eventType` | string         | [The type of the event](.#event-type) |
| `time`      | number         | Simulation timestamp of the event     |
| `source`    | string or null | Component that generated the event    |
| `details`   | object         | Additional event-specific information |

### Event type

Each event includes an `eventType` field. The value must be one of the following six types:

| Type       | Description                                                                |
|------------|----------------------------------------------------------------------------|
| `DEMAND`   | Indicates that a travel demand has been created by a user.                 |
| `RESERVE`  | Indicates that a reservation request has been made for an existing demand. |
| `RESERVED` | Indicates whether a reservation request has been accepted or rejected.     |
| `DEPART`   | Indicates that a departure is scheduled or confirmed.                      |
| `DEPARTED` | Indicates that the user or mobility unit has left the origin location.     |
| `ARRIVED`  | Indicates that the user or mobility unit has reached the destination.      |

These types of events represents the life cycle of both user travel and mobility movement.

### Location object

Location is commonly used in events to describe a geographic point.

| Field        | Type   | Description         |
|--------------|--------|---------------------|
| `locationId` | string | Location identifier |
| `lat`        | number | Latitude            |
| `lng`        | number | Longitude           |

### Trip object

A Trip represents a single travel segment assigned to the user as part of a reserved route.
A route is composed of one or more Trip objects, each indicating a movement from an origin to a destination using a specific mobility service.

| Field     | Type     | Required | Description                                |
| --------- | -------- | -------- | ------------------------------------------ |
| `org`     | Location | Required | Origin location of the segment.            |
| `dst`     | Location | Required | Destination location of the segment.       |
| `dept`    | number   | Required | Scheduled departure time.                  |
| `arrv`    | number   | Required | Scheduled arrival time.                    |
| `service` | string   | Required | Mobility service assigned to this segment. |

## Demand Event

A `DEMAND` event indicates that a user has created a travel request from an origin to a destination.
Exactly one of `dept` or `arrv` must be null, depending on the type of requested schedule:
- Leave-at time (departure fixed): `dept` is set, `arrv` is null
- Arrive-by time (arrival fixed): `arrv` is set, `dept` is null

| Field      | Type           | Required | Description                                                                                |
|------------|----------------|----------|--------------------------------------------------------------------------------------------|
| `userId`   | string         | Required | ID of the user making the travel demand.                                                   |
| `demandId` | string         | Required | ID for this travel demand.                                                                 |
| `org`      | `Location`     | Required | The origin location of the demand.                                                         |
| `dst`      | `Location`     | Required | The destination location of the demand.                                                    |
| `service`  | string         | Optional | A value used by the user-model component, e.g., to bind the request to a specific service. |
| `dept`     | number or null | Optional | Desired departure time.                                                                    |
| `arrv`     | number or null | Optional | Desired arrival time.                                                                      |

### Demand event example

```json
{
  "eventType": "DEMAND",
  "time": 120.5,
  "details": {
    "userId": "user-001",
    "demandId": "dmd-001",
    "org": { "locationId": "A", "lat": 35.0, "lng": 135.0 },
    "dst": { "locationId": "B", "lat": 35.1, "lng": 135.1 },
    "dept": 130.0,
    "arrv": null
  }
}
```

This means:
> At simulation time `120.5`, a travel demand with ID `dmd-001` is created by user `user-001`.
The user wants to travel from location `A` to location `B` and aims to depart at time `130.0`.

## Reserve Event

A `RESERVE` event indicates that a user has issued a reservation request.
After receiving a `DEMAND` event, the user-model component evaluates candidate trip options and, 
when it selects one, it emits a `RESERVE` event to express the user’s intention to reserve that option.

A `RESERVE` event does not guarantee that the reservation is accepted by a mobility service.
It simply represents a user-initiated request, and the mobility service may later confirm or reject it depending on service availability.

| Field / Subfield   | Type   | Required | Description                                                       |
|--------------------|--------|----------|-------------------------------------------------------------------|
| `service`          | string | Required | Name of the mobility service the user is attempting to reserve.   |
| `details.userId`   | string | Required | ID of the user making the reservation request.                    |
| `details.demandId` | string | Required | ID of the travel demand associated with this reservation request. |
| `details.org`      | object | Required | The origin location of the requested trip.                        |
| `details.dst`      | object | Required | The destination location of the requested trip.                   |
| `details.dept`     | number | Required | Requested departure time.                                         |
| `details.arrv`     | number | Optional | Requested arrival time (for arrive-by demands).                   |

### Reserve event example
```json
{
  "eventType": "RESERVE",
  "time": 121.0,
  "service": "service-001",
  "details": {
    "userId": "user-001",
    "demandId": "dmd-001",
    "org": { "locationId": "A", "lat": 35.0, "lng": 135.0 },
    "dst": { "locationId": "B", "lat": 35.1, "lng": 135.1 },
    "dept": 130.0
  }
}
```

This means: 
> At simulation time `121.0`, user `user-001` requests a reservation for travel demand `dmd-001` on service `service-001`.
The trip is planned from location `A` to location `B` with an intended departure at time `130.0`.

## Reserved Event

A `RESERVED` event represents the outcome of a reservation request.
There are two possible results:

- Reservation Success – The request is accepted and a route is assigned.
- Reservation Failure – The request is rejected and no route is assigned.

| Field      | Type            | Required | Description                                                                                                       |
|------------|-----------------|----------|-------------------------------------------------------------------------------------------------------------------|
| `success`  | boolean         | Required | Indicates the reservation was accepted.                                                                           |
| `userId`   | string          | Required | ID of the user who made the reservation request.                                                                  |
| `demandId` | string          | Required | ID of the travel demand associated with this reservation.                                                         |
| `route`    | array of `Trip` | Required | Assigned route. Each element represents a trip segment with origin, destination, departure, arrival, and service. |

### Reserved event example in the success case

```json
{
  "eventType": "RESERVED",
  "time": 121.5,
  "details": {
    "success": true,
    "userId": "user-001",
    "demandId": "dmd-001",
    "route": [
      {
        "org": { "locationId": "A", "lat": 35.0, "lng": 135.0 },
        "dst": { "locationId": "B", "lat": 35.1, "lng": 135.1 },
        "dept": 130.0,
        "arrv": 150.0,
        "service": "service-001"
      }
    ]
  }
}
```

This means:

> At simulation time `121.5`, the reservation request for travel demand `dmd-001` by user `user-001` is accepted.
The assigned route uses service `service-001`, departing from location `A` at `130.0` and arriving at location `B` at `150.0`.

### Reserved event example in the failure case

```json
{
  "eventType": "RESERVED",
  "time": 121.5,
  "details": {
    "success": false,
    "userId": "user-001",
    "demandId": "dmd-002",
    "route": []
  }
}
```

This means:
> At simulation time `121.5`, the reservation request for travel demand `dmd-002` by user `user-001` was rejected.
No route could be assigned, so the `route` array is empty.

## Depart event

The `DEPART` event indicates that a user — who already has a confirmed reservation — 
has actually arrived at the departure location and is ready to begin the trip using the assigned mobility service.

A reservation does not guarantee that the user will show up.
A user may fail to reach the pickup stop due to delays, cancellations, or external circumstances.
Therefore, the `DEPART` event acts as an explicit acknowledgment from the user-model component 
that the user has reached the departure point and the mobility service can safely proceed.

If the user never arrives, no `DEPART` event is generated.
In that case, the mobility service may handle it internally as a no-show, depending on the simulator specification.

| Field / Subfield   | Type   | Required | Description                                             |
|--------------------|--------|----------|---------------------------------------------------------|
| `service`          | string | Required | ID of the mobility service that the user will use.      |
| `details.userId`   | string | Required | ID of the user who is ready to depart.                  |
| `details.demandId` | string | Required | ID of the travel demand associated with this departure. |

### Depart event example

```json
{
  "eventType": "DEPART",
  "time": 130.0,
  "service": "service-001",
  "details": {
    "userId": "user-001",
    "demandId": "dmd-001"
  }
}
```

This means:

> At simulation time `130.0`, the departure of user `user-001` for travel demand `dmd-001` is scheduled on service `service-001`.

## Departed event

The `DEPARTED` event indicates that a user or mobility unit has departed from a location.

There are two variations:
- User departure: Indicates that a user unit has left the origin location.
- Mobility departure: Indicates that a mobility unit has left a location.

| Field        | Type       | Required | Description              |
|--------------|------------|----------|--------------------------|
| `location`   | `Location` | Required | Departure location.      |
| `userId`     | string     | Optional | ID of the user.          |
| `demandId`   | string     | Optional | ID of the travel demand. |
| `mobilityId` | string     | Optional | ID of the mobility.      |

### User departure event example

```json
{
  "eventType": "DEPARTED",
  "time": 130.0,
  "details": {
    "userId": "user-001",
    "demandId": "dmd-001",
    "location": { "locationId": "A", "lat": 35.0, "lng": 135.0 }
  }
}
```

This means:
> At simulation time `130.0`, user `user-001` for travel demand `dmd-001` has actually departed from location `A`.

### Mobility departure event example

```json
{
  "eventType": "DEPARTED",
  "time": 128.0,
  "details": {
    "mobilityId": "mob-001",
    "location": { "locationId": "A", "lat": 35.0, "lng": 135.0 }
  }
}
```

This means
> At simulation time `128.0`, mobility unit `mob-001` has departed from location `A`.

## Arrived event

The `ARRIVED` event indicates that a user or mobility unit has arrived at a location

There are two variations:
- User arrival: Indicates that a user has reached the destination.
- Mobility arrival: Indicates that a mobility unit has arrived at a location.

| Field       | Type       | Required | Description              |
|-------------|------------|----------|--------------------------|
| `location`  | `Location` | Required | Arrival location         |
| `userId`    | string     | Optional | ID of the user.          |
| `demandId`  | string     | Optional | ID of the travel demand. |
| `moilityId` | string     | Optional | ID of the mobility.      |

### User arrival event example

```json
{
  "eventType": "ARRIVED",
  "time": 150.0,
  "details": {
    "userId": "user-001",
    "demandId": "dmd-001",
    "location": { "locationId": "B", "lat": 35.1, "lng": 135.1 }
  }
}
```

This means:
> At simulation time `150.0`, user `user-001` for travel demand `dmd-001` has arrived at location `B`.

### Mobility arrival event example

```json
{
  "eventType": "ARRIVED",
  "time": 145.0,
  "details": {
    "mobilityId": "mob-001",
    "location": { "locationId": "C", "lat": 35.2, "lng": 135.2 }
  }
}
```

This means:
> At simulation time `145.0`, mobility unit `mob-001` has arrived at location `C`.
