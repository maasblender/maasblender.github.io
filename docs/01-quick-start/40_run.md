---
sidebar_position: 40
title: Run
---

## Running the Simulation

From `examples/quick-start`, use Docker Compose to start the required services.
The first build downloads container images and dependencies and may take several minutes.

```bash
docker compose up -d
docker compose ps -a
```

Confirm that all six services (`broker`, `walking`, `user_model`, `scenario_commuter`, `scheduled`, and `planner`) are running.

```bash
python execute_simulation.py
```

The script uploads the bundled files, configures the services, and runs the simulation. OpenTripPlanner initialization can take several minutes.

Example output (progress values vary):

```text
============================================================
GTFS Validity Check
============================================================
[✅ OK] reference_time=20251016 is within GTFS valid period (2025-10-16 - 2026-03-31) for service 'gtfs'.
{'message': 'successfully uploaded. otp-config.zip'}
{'message': 'successfully uploaded. gtfs.zip'}
{'message': 'successfully uploaded. gtfs.zip'}
{'message': 'successfully configured.'}
{'message': 'successfully started.'}
{'message': 'successfully run.'}
running: 575.0
...
All events recorded to events.txt
successfully finished.
```

## Simulation Output

When the script finishes, the results are saved to `events.txt` in the current directory.
The `events.txt` file contains a chronological log of all events processed by MaaS Blender during the simulation. 
By analyzing these events, you can calculate various performance and usage metrics of the simulated mobility services.

## Stop the Services

Remove the example containers and network:

```shell
docker compose down
```
