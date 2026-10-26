---
sidebar_position: 10
title: Prerequisites
---

Before you begin, make sure you have the following installed on your machine:

- **Git** — used to clone the repository
- **Docker** and **Docker Compose** — used to run the simulation environment in containers
- **Python 3.9+** — used to run execution scripts that interact with Maas Blender services to run simulations

The example publishes ports **3000**, **3001**, and **3010**; these ports must be available.

## Install Python Dependencies

The execution script used in this guide depends on the `httpx` library for making HTTP requests.

Please install it before running the simulation:

```shell
python -m pip install httpx
```
