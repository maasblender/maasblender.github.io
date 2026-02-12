---
sidebar_position: 70
title: Evaluation Sidecar 
---

A evaluation component is an auxiliary component that runs alongside the simulation to observe 
and evaluate system behavior in real time.

While MaaS Blender is fundamentally event-driven, certain aspects of system behavior cannot be fully reconstructed
from the event log alone. The Evaluation Sidecar exists to capture such information at the moment it occurs,
without affecting the simulation flow.

## Events

A evaluation component consumes all events in the system but does not emit events of its own.
It is strictly observational and must not influence simulation behavior.

| Consumed Events | Emitted Events |
|-----------------|----------------|
| `DEMAND`        | _(none)_       |
| `RESERVE`       |                |
| `RESERVED`      |                |
| `DEPART`        |                |
| `DEPARTED`      |                |
| `ARRIVED`       |                |

## Purpose and Motivation

The primary purpose of the evaluation component is to enable richer analysis than post-hoc event inspection alone can provide.

For example, when a user successfully reserves a mobility service,
it may be important to know whether alternative route candidates were also reservable at that exact time.
Because this information is only available during simulation execution, it must be observed in real time.

## Design Rationale

By externalizing evaluation logic into a sidecar component:

- other simulation components remain simple and focused,
- analytical concerns do not leak into execution logic,
- and multiple evaluation strategies can be developed independently.
