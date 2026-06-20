---
sidebar_position: 20
title: "Scenario: Demand Generator"
---

This chapter specifies the reference implementations for demand-generation components in MaaS Blender.
Each component emits `DEMAND` events that kick off user activity in the simulation.
They differ by how and when those demands are created.

The project currently ships three reference implementations under `maasblender/src/scenario`:

- Stochastic Window Generator: to simulate aggregate, probabilistic flows over time windows.
- Historical Replay: when you need exact timing and content of demands (replay/benchmark).
- Commuter Pattern: to model recurring daily commuter behavior at specified times.

## Stochastic Window Generator (`DemandGenerator`)

- File: `maasblender/src/scenario/generator/generator.py`
- Primary class: `DemandGenerator`

### Behavior
- You define one or more windows `[begin, end)` with an expected number of demands in that window.
- The generator discretizes the window into 1-minute slots and, for each slot,
  draws a Bernoulli trial with probability:
  `p = expected_demands / number_of_slots`.
- For each success, a demand is created at that slot’s time.
- The meaning of the sampled slot depends on `arrive_by`:
  - `arrive_by: false` (default): the slot is treated as the desired departure time.
  - `arrive_by: true`: the slot is treated as the desired arrival time.
- `resv` (reservation emission time) controls when the `DEMAND` event is emitted.
  If `resv` is omitted, leave-at demands are emitted at the sampled slot time,
  while arrive-by demands are emitted immediately at simulation time `0.0`.

### Configuration

```json5
{
  "seed": 123,                               // RNG seed for reproducibility
  "userIDFormat": "U%03d",                 // e.g., U001, U002, ...
  "demandIDFormat": "D_%d",                // e.g., D_1, D_2, ...
  "demands": [
    // you can use multiple overlapping windows
    {
      "begin": 10.0,                        // window start (minutes)
      "end": 200.0,                         // window end (minutes)
      "expected_demands": 2.0,              // expected count within the window
      "resv": 5.0,                          // reservation time (minutes) — optional
      "arrive_by": false,                   // optional; default false
      "org": {"locationId": "Org", "lat": 35.0, "lng": 139.0},
      "dst": {"locationId": "Dst", "lat": 35.7, "lng": 139.7},
      "user_type": "commuter",             // optional
      "service": "my-mobility-service"     // optional
    }
  ]
}
```

:::warning
If `p > 0.1` per minute, use a smaller unit time to better approximate a Poisson process.
:::

#### Example

```json5
{
  "seed": 128,
  "userIDFormat": "U%03d",
  "demands": [
    {
      "begin": 10.0,
      "end": 200.0,
      "expected_demands": 2.0,
      "resv": 7.0,
      "org": {"locationId": "Org", "lat": 35.0, "lng": 139.0},
      "dst": {"locationId": "Dst", "lat": 35.7, "lng": 139.7},
      "service": "mobility-service-for-test"
    }
  ]
}
```
This configuration usually yields two reservation demands around minute 7, each with a different `dept` inside the `[10, 200)` window.

#### Arrive-by Example

```json5
{
  "seed": 129,
  "userIDFormat": "U%03d",
  "demands": [
    {
      "begin": 10.0,
      "end": 200.0,
      "expected_demands": 2.0,
      "arrive_by": true,
      "resv": 5.0,
      "org": {"locationId": "Org", "lat": 35.0, "lng": 139.0},
      "dst": {"locationId": "Dst", "lat": 35.7, "lng": 139.7},
      "user_type": "test-user"
    }
  ]
}
```

With this configuration, each `DEMAND` is emitted at minute `5.0`, and `arrv`
is set to a sampled arrival time inside `[10, 200)`. If `resv` is omitted,
the same arrive-by demand is still generated with the sampled `arrv`, but the
event itself is emitted at simulation time `0.0`.

### Output
- `users()` returns the list of created users.
- `DEMAND` events carry `userId`, auto-assigned `demandId`, `org`, `dst`, optional `service`, and exactly one time constraint.

| Mode | `resv` | Event emission time | Time constraint |
|------|--------|---------------------|-----------------|
| Leave-at | omitted | `sampled_slot` | `dept = null`, `arrv = null` |
| Leave-at | set | `resv` | `dept = sampled_slot`, `arrv = null` |
| Arrive-by | omitted | `0.0` | `dept = null`, `arrv = sampled_slot` |
| Arrive-by | set | `resv` | `dept = null`, `arrv = sampled_slot` |

## Historical Replay (`HistoricalScenario`)

- File: `maasblender/src/scenario/historical/historical.py`
- Primary class: `HistoricalScenario`

### Behavior
- Each input record specifies exactly when to emit a `DEMAND` and its full payload.
- If a record lacks `user_id` or `demand_id`, the scenario fills them using provided formats (e.g., `U%03d`, `D_%d`).
- Each record must specify exactly one of `dept` (leave-at) or `arrv` (arrive-by).
- If `time` is omitted, the implementation fills it automatically:
  - leave-at: `time = dept`
  - arrive-by: `time = 0.0`

### Configuration
The `/setup` payload accepts a collection of `HistoricalDemandSetting` items, plus two ID formats:

```json5
{
  "userIDFormat": "U%03d",            // used when a record has no user_id
  "demandIDFormat": "D_%d",           // used when a record has no demand_id
  "trips": [
    {
      "time": 480.0,                   // exact emission time (minutes since start)
      "org": {"locationId": "A", "lat": 35.1, "lng": 139.1},
      "dst": {"locationId": "B", "lat": 35.7, "lng": 139.7},
      "dept": 495.0,                   // intended departure time
      "user_id": "U123",              // optional; auto-assigned if missing
      "demand_id": "D_901",           // optional; auto-assigned if missing
      "user_type": "visitor",         // optional; associated to user_id
      "service": "bus-line-12",       // optional
      "actual_duration": 23.5          // optional metadata for analysis
    }
  ]
}
```

### Output
- `users()` returns all known users.
- A `DEMAND` is emitted exactly at `time` with the provided payload.
- For arrive-by demands, `arrv` is set and `dept = null`.

## Commuter Pattern (`CommuterScenario`)

- File: `maasblender/src/scenario/commuter/commuter.py`
- Primary class: `CommuterScenario`
- Purpose: Generate two daily demands per configured commuter: one outbound (home → work) and one inbound (work → home), repeating every day. Each leg can independently use leave-at or arrive-by mode.

#### Behavior
- For each commuter, two `DEMAND`s are created daily:
  1) outbound (`org` → `dst`)
  2) inbound (`dst` → `org`), automatically reversing the original direction.
- For each leg, you configure exactly one of:
  - leave-at: `deptOut` / `deptIn`
  - arrive-by: `arrvOut` / `arrvIn`
- In leave-at mode, the `DEMAND` is emitted at the configured departure time and carries `dept`.
- In arrive-by mode, the `DEMAND` is emitted `leadTime` minutes before the target arrival time and carries `arrv`.
- The pattern repeats every 1440 minutes (1 day).
- `demandId` values are generated from the provided `demandIDFormat`.
- The setup schema validates that:
  - `deptOut` and `arrvOut` cannot both be specified, and one of them is required.
  - `deptIn` and `arrvIn` cannot both be specified, and one of them is required.
  - The outbound demand emission time must be earlier than or equal to the inbound demand emission time.

#### Configuration

```json5
{
  "demandIDFormat": "D_%d",
  "commuters": {
    "U001": {
      "deptOut": 480.0,   // 08:00
      "deptIn": 1080.0,   // 18:00
      "org": {"locationId": "Home", "lat": 35.0, "lng": 139.0},
      "dst": {"locationId": "Work", "lat": 35.7, "lng": 139.7},
      "user_type": "office",
      "service": "rail"
    },
    "U002": {
      "arrvOut": 540.0,   // 09:00 target arrival
      "arrvIn": 1140.0,   // 19:00 target arrival back home
      "leadTime": 15.0,   // emit the arrive-by demand 15 minutes earlier
      "org": {"locationId": "Home2", "lat": 35.2, "lng": 139.2},
      "dst": {"locationId": "Work2", "lat": 35.6, "lng": 139.6}
    }
  }
}
```

#### Output
- `users()` returns all configured commuters.
- Every day, two `DEMAND` events are emitted per commuter.
- For inbound events, origin and destination are automatically reversed.
- Each event carries exactly one time constraint:
  - leave-at legs set `dept`
  - arrive-by legs set `arrv`

### Common Operational Notes

- All three components use `simpy` for event scheduling.
- All times are expressed in minutes from the simulation start.
- All three components produce `DEMAND` events carrying `DemandInfo`.
- Determinism and Seeding:
  - `DemandGenerator` uses a random seed to ensure reproducible stochastic outcomes.
  - `HistoricalScenario` and `CommuterScenario` are deterministic given their inputs.
