---
sidebar_position: 20
title: Setup
---

## Clone the Repository

First, clone the Maas Blender repository and move into the example project directory:

```shell
git clone https://github.com/maasblender/maasblender.git
cd maasblender
git checkout tags/v0.8.0
cd maasblender/examples/quick-start
```

This is a minimal example environment to quickly launch and experiment with MaasBlender using Docker Compose.  

```
/examples/get-started/
├── broker_setup.json     # Broker configuration file
├── compose.yml           # Docker Compose file defining required containers and services
├── execute_simulation.py # Python script for file registration, integration, and execution management with the simulator
├── gtfs.zip              # Sample GTFS file 
└── otp-config.zip        # Configuration file for OpenTripPlanner
```

## Prepare Simulation Data

Maas Blender supports multiple **open mobility data standards**, 
including [GTFS](https://gtfs.org/), [GTFS-Flex](https://gtfs.org/extensions/flex/) and [GBFS](https://gbfs.org/).  

For this Quick Start, we will use the GTFS data of the Maidohaya bus service in Toyama City, Japan,
available [here](https://opdt.city.toyama.lg.jp/dataset/toyamacity-bus-gtfs-jp/resource/43903d9f-1d9c-42f0-bd01-8a5fc1cb828c)

In actual simulations, you can use any GTFS dataset of your choice.  
Here, we use this one as a sample dataset, which has already been placed in the environment.
