---
sidebar_position: 40
title: Run
---

## Running the Simulation

First, start the required services using Docker Compose:

```bash
docker compose up -d 
````

Once all services are up and running, execute the simulation script:

```bash
$ python execute_simulation.py
{'message': 'successfully uploaded. otp-config.zip'}
{'message': 'successfully uploaded. gtfs.zip'}
{'message': 'successfully uploaded. gtfs.zip'}
{'message': 'successfully configured.'}
{'message': 'successfully started.'}
{'message': 'successfully run.'}
running: 602.0
running: 673.0
running: 747.0
running: 818.0
running: 891.0
running: 964.0
running: 1035.0
running: 1110.0
running: 1980.0
successfully finished.
All events recorded to events.txt
```

## Simulation Output

After the simulation finishes, the `events.txt` will be generated:
The `events.txt` file contains a chronological log of all events processed by MaaS Blender during the simulation. 
By analyzing this file, you can calculate various performance and usage metrics of the simulated mobility services.