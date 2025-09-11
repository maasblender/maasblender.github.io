---
sidebar_position: 30
title: Quick Execution
---

## Cloning the Repository
First, clone the code from the GitHub repository:

```
git clone https://github.com/maasblender/maasblender.git
```

For this example, we will use the code and data located in the following directories:

```
examples/get-started/
├── compose.yml             # Docker Compose file defining required containers and services
└── execution.py            # Python script for file registration, integration, and execution management with the simulator

src/
├── base_simulators/        # Base simulators for scheduled and on-demand types
├── evaluation/             # Components for evaluation processing (e.g., analysis of simulation results)
├── planner/                # Route search component (e.g., OpenTripPlanner)
├── scenario/               # Scenario generation and management
├── simulation_broker/      # Broker component that orchestrates all services
└── user_model/             # User behavior model (e.g., route selection)
```

## Creating the Required Files
These files should be prepared individually according to target area, services, and settings being used.
Place the prepared files under the `examples/get-started/` directory.
This example assumes the use of [OpenTripPlanner](https://www.opentripplanner.org/) as the route search engine.

```
examples/get-started/
├── otp-config.zip           # Configuration file for OpenTripPlanner
├── gtfs.zip                 # GTFS data for fixed-route public transport (e.g., route buses)
├── gtfs_flex.zip            # GTFS-Flex data for on-demand transport (e.g., demand-responsive buses)
└── broker_setup.json        # Broker configuration file
```

Instructions on how to obtain or create each file will be added later.

## Running the Simulation
Use the following commands to start the required services and run the simulation:

```sh
cd examples/get-started
docker compose up -d
python execution.py
```

After execution, the following file will be generated:

```
output/
└── events.txt
```

events.txt is a log that records all event information handled by MaaS Blender.
By analyzing this log, you can calculate usage rates and convenience levels within the targeted transportation network.

