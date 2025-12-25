---
sidebar_position: 10
title: Concept
---

MaaS Blender is designed as an **event-driven, discrete-time simulation platform** for evaluating Mobility as a Service (MaaS) systems. 

Rather than simulating continuous motion or enforcing a global time step, MaaS Blender models system dynamics as a sequence of **timestamped events**.
All changes in system state — such as demand creation, reservation decisions, departures, and arrivals — are expressed explicitly as events occurring at specific simulation times.

### Event-Centric Modeling
In MaaS Blender, events are the **primary mechanism for interaction** between components.

* Components typically interact by emitting and receiving events.
* Each event represents an observable change in the system state.

This approach encourages loose coupling between components.

### Separation of Concerns

The simulation architecture separates responsibilities across independent components, including:

* **The Broker**, which orchestrates execution and event propagation
* **User models**, which generate travel demands and react to service outcomes
* **Mobility service models**, which manage vehicles, capacity, and operations
* **Routing and planning components**, which propose feasible travel options

Each component is responsible only for its own internal logic and state, and communicates exclusively via events.

### Deterministic Execution

MaaS Blender enforces single-component execution at any point in simulation time.
Components are executed sequentially in a deterministic order determined by their next scheduled event time.

As a result, simulation behavior is deterministic given identical inputs and deterministic component implementations.
This determinism makes simulation runs reproducible and easier to debug, and allows evaluation results to be consistently interpreted.

### Design Goals

The core design goals of MaaS Blender are:

* **Modularity**: enable integration of diverse mobility services and models
* **Determinism**: ensure reproducible simulation results
* **Transparency**: make system behavior explainable through explicit event traces

These principles guide all architectural decisions described in the following sections.
