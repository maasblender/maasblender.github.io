---
sidebar_position: 10
title: Event Specification
---

The simulation represents state changes of both **users** and **mobility units** (such as vehicles) as events.  
Events are output in **JSON format** and are processed in chronological order.

## Common Event Structure

Every event has the following fields:

| Field       | Type           | Description                           |
|-------------|----------------|---------------------------------------|
| `eventType` | string         | The type of the event                 |
| `time`      | number         | Simulation timestamp of the event     |
| `source`    | string or null | Component that generated the event    |
| `details`   | object         | Additional event-specific information |

### Event Type

Each event includes an eventType field. The value must be one of the following six types:

| Type       | Description                                                                |
|------------|----------------------------------------------------------------------------|
| `DEMAND`   | Indicates that a travel demand has been created by a user.                 |
| `RESERVE`  | Indicates that a reservation request has been made for an existing demand. |
| `RESERVED` | Indicates whether a reservation request has been accepted or rejected.     |
| `DEPART`   | Indicates that a departure is scheduled or confirmed.                      |
| `DEPARTED` | Indicates that the user or mobility unit has left the origin location.     |
| `ARRIVED`  | Indicates that the user or mobility unit has reached the destination.      |

These types define the life cycle of both user travel and mobility unit movement.

### Location Object

Location is commonly used in events to describe a geographic point.

| Field        | Type   | Description         |
|--------------|--------|---------------------|
| `locationId` | string | Location identifier |
| `lat`        | number | Latitude            |
| `lng`        | number | Longitude           |

## Demand event

A `DEMAND` event represents that a user has created a request to travel from an origin to a destination.
Either `dept` or `arrv` may be null depending on whether the demand is based on:
- Leave-at time (departure fixed): `dept` is set, `arrv` is null
- Arrive-by time (arrival fixed): `arrv` is set, `dept` is null

| Field      | Type           | Required | Description                                |
|------------|----------------|----------|--------------------------------------------|
| `userId`   | string         | Yes      | Identifier of the user making the request. |
| `demandId` | string         | Yes      | Unique identifier for this travel demand.  |
| `org`      | `Location`     | Yes      | The origin location of the demand.         |
| `dst`      | `Location`     | Yes      | The destination location of the demand.    |
| `dept`     | number or null | Optional | Desired departure time.                    |
| `arrv`     | number or null | Optional | Desired arrival time.                      |

### Demand example

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

This event represents:
At simulation time `120.5`, a travel demand with ID `dmd-001` is created by user `user-001`.
The user wants to travel from location `A` to location `B` and aims to depart at time `130.0`.

## Reserve event

| Field / Subfield   | Type   | Required | Description                                               |
| ------------------ | ------ | -------- | --------------------------------------------------------- |
| `service`          | string | Required | ID of the mobility service requested for the reservation. |
| `details.userId`   | string | Required | ID of the user making the reservation request.            |
| `details.demandId` | string | Required | ID of the travel demand associated with this request.     |
| `details.org`      | object | Required | Origin location of the requested trip.                    |
| `details.dst`      | object | Required | Destination location of the requested trip.               |
| `details.dept`     | number | Required | Requested departure time.                                 |
| `details.arrv`     | number | Optional | Requested arrival time (for arrive-by demands).           |


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
    "dept": 130.0,
    "arrv": null
  }
}
```

This event represents:
At simulation time 121.0, user user-001 requests a reservation for travel demand dmd-001 on service service-001.
The trip is planned from location A to location B with an intended departure at time 130.0.

## Reserved event

The RESERVED event represents the result of a reservation request.
There are two variations:

- Reservation Success – The request is accepted and a route is assigned.
- Reservation Failure – The request is rejected and no route is assigned.

### Reserved event example in the success case

| Field / Subfield   | Type    | Required | Description                                                                                                       |
| ------------------ | ------- | -------- | ----------------------------------------------------------------------------------------------------------------- |
| `details.success`  | boolean | Required | Always `true`. Indicates the reservation was accepted.                                                            |
| `details.userId`   | string  | Required | ID of the user who made the reservation request.                                                                  |
| `details.demandId` | string  | Required | ID of the travel demand associated with this reservation.                                                         |
| `details.route`    | array   | Required | Assigned route. Each element represents a trip segment with origin, destination, departure, arrival, and service. |


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

This event represents:
At simulation time 121.5, the reservation request for travel demand dmd-001 by user user-001 is accepted.
The assigned route uses service service-001, departing from location A at 130.0 and arriving at location B at 150.0.

### Reserved event example in the failure case


| Field / Subfield   | Type    | Required | Description                                               |
| ------------------ | ------- | -------- | --------------------------------------------------------- |
| `details.success`  | boolean | Required | Always `false`. Indicates the reservation was rejected.   |
| `details.userId`   | string  | Required | ID of the user who made the reservation request.          |
| `details.demandId` | string  | Required | ID of the travel demand associated with this reservation. |
| `details.route`    | array   | Optional | Empty array. No route is assigned.                        |



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

This event represents:
At simulation time 121.5, the reservation request for travel demand dmd-002 by user user-001 was rejected.
No route could be assigned, so the route array is empty.

## Depart event

The `DEPART` event indicates that a user who has a confirmed reservation has arrived at the departure location and is ready to depart using the assigned mobility service.

- This event essentially notifies the mobility service that the user is present and the trip can start.
- If, for any reason, the user fails to reach the departure location, this event will not occur.

| Field / Subfield   | Type   | Required | Description                                             |
| ------------------ | ------ | -------- | ------------------------------------------------------- |
| `service`          | string | Required | ID of the mobility service that the user will use.      |
| `details.userId`   | string | Required | ID of the user who is ready to depart.                  |
| `details.demandId` | string | Required | ID of the travel demand associated with this departure. |


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

This event represents:
At simulation time 130.0, the departure of user user-001 for travel demand dmd-001 is scheduled on service service-001.

why?

## Departed event

The DEPARTED event represents the departure of either a user or a mobility unit.
There are two variations:
- User departure – indicates that a user has actually left the origin location.
- Mobility departure – indicates that a mobility unit has left a location to start a trip segment.

### User departure

| Field / Subfield | Type       | Required | Description                      |
| ---------------- | ---------- |----------| -------------------------------- |
| `userId`         | string     | Required | ID of the user.                  |
| `demandId`       | string     | Required | ID of the travel demand.         |
| `mobilityId`     | string     | Optional | ID of the mobility, if relevant. |
| `location`       | `Location` | Required | Departure location.              |

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

This event represents:
At simulation time 130.0, user user-001 for travel demand dmd-001 has actually departed from location A.

### Mobility departure

| Field / Subfield | Type       | Required | Description              |
| ---------------- | ---------- | -------- | ------------------------ |
| `mobilityId`     | string     | Required | ID of the mobility unit. |
| `location`       | `Location` | Required | Departure location.      |

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

This event represents:
At simulation time 128.0, mobility unit mob-001 has departed from location A.
This marks the start of a trip segment that may serve one or more travel demands.

## Arrived event

The `ARRIVED` event represents the arrival of either a user or a mobility unit.
There are two variations:
- User arrival – indicates that a user has reached the destination.
- Mobility arrival – indicates that a mobility unit (e.g., bus, taxi) has arrived at a location.

### User Arrival

| Field / Subfield | Type       | Required | Description              |
|------------------|------------|----------|--------------------------|
| `userId`         | string     | Required | ID of the user.          |
| `demandId`       | string     | Required | ID of the travel demand. |
| `moilityId`      | string     | Optional | ID of the mobility.      |
| `location`       | `Location` | Required | Arrival location         |


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

This event represents:
At simulation time `150.0`, user `user-001` for travel demand `dmd-001` has arrived at location `B`.

### Mobility Arrival

| Field / Subfield | Type       | Required | Description              |
| ---------------- | ---------- | -------- | ------------------------ |
| `mobilityId`     | string     | Required | ID of the mobility unit. |
| `location`       | `Location` | Required | Arrival location.        |

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

This event represents:
At simulation time 145.0, mobility unit mob-001 has arrived at location C.

## User Life Cycle

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

## Mobility Life Cycle

A mobility unit (e.g., bus, taxi, or shared vehicle) repeatedly moves between locations to serve user demands.
Unlike a user, a mobility unit does not create demands; it simply departs from one location and arrives at another.

The life cycle is a continuous sequence of DEPARTED → ARRIVED events.

```
DEPARTED → ARRIVED → DEPARTED →  ... → ARRIVED
```

A mobility unit may serve multiple user demands along its route, but its events only track movement.
