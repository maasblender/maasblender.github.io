---
sidebar_position: 50
title: "Evaluation: Analysis Sidecar"
---

This chapter specifies the Evaluation component in MaaS Blender.
The Evaluation component is an auxiliary sidecar that runs alongside the simulation to observe and analyze system behavior in real time.
Unlike other components that drive the simulation forward, the evaluation component is strictly observational and does not emit events or influence simulation execution.

The project currently provides a **Simple Evaluator** implementation that analyzes route planning results for usability metrics. 
This evaluator evaluates which services are reservable for each DEMAND event, enabling analysis of service coverage and availability.

#### Behavior

- The evaluation component processes `DEMAND` events to analyze available routes and their reservability.
- For each demand event, the evaluator queries the route planner and checks which services are reservable.
- Generates evaluation results in real time during simulation, capturing route availability data.

#### Configuration

```json
{
  "planner": {
    "endpoint": "http://broker/plan"
  },
  "reservable": {
    "endpoint": "http://broker/reservable"
  },
  "evaluation_timing": "departure"
}
```

- `planner.endpoint`: The endpoint for route planning queries (typically the broker's `/plan` endpoint).
- `reservable.endpoint`: The endpoint for checking service reservability (typically the broker's `/reservable` endpoint).
- `evaluation_timing`: When to evaluate routes:
  - `"departure"` (default): Evaluate at the actual departure time specified in the demand event.
  - `"demand"`: Evaluate immediately when the demand event is created.

## Post-Simulation Analysis

### Evaluation Output Format

The Simple Evaluator generates an `evaluation.txt` file containing evaluation results in JSON Lines format (one JSON object per line).
Each line represents the evaluation result for a single `DEMAND` event.

```json
{
  "demand_id": "D_1",
  "time": 570.0,
  "event_time": 563.309169,
  "org": "1_01",
  "dst": "7_01",
  "actual_service": "gtfs",
  "plans": [
    [
      {
        "org": "1_01",
        "dst": "7_01",
        "dept": 570.0,
        "arrv": 578.0,
        "service": "gtfs",
        "reservable": true
      }
    ],
    [
      {
        "org": "1_01",
        "dst": "1_02",
        "dept": 570.0,
        "arrv": 572.0,
        "service": "walking",
        "reservable": true
      },
      {
        "org": "1_02",
        "dst": "7_01",
        "dept": 580.0,
        "arrv": 590.0,
        "service": "flex",
        "reservable": true
      }
    ]
  ]
}
```

- **`demand_id`** (string): Unique identifier of the demand event being evaluated
- **`time`** (number): Departure time in minutes from simulation start
- **`event_time`** (number): The time when the DEMAND event was created
- **`org`** (string): Origin location ID
- **`dst`** (string): Destination location ID
- **`actual_service`** (string | null): The service actually reserved by the user (if known at evaluation time)
- **`plans`** (array of arrays): All route alternatives returned by the planner
  - Each route alternative is an array of trip legs
  - Each trip leg contains:
    - **`org`** (string): Origin location ID of this leg
    - **`dst`** (string): Destination location ID of this leg
    - **`dept`** (number): Departure time of this leg (minutes from simulation start)
    - **`arrv`** (number): Arrival time of this leg (minutes from simulation start)
    - **`service`** (string): Service name used for this leg (e.g., "gtfs", "flex", "walking")
    - **`reservable`** (boolean): Whether this service leg can be reserved

