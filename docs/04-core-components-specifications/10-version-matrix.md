---
sidebar_position: 10
title: Version Compatibility
---

This page summarizes the version compatibility between MaaS Blender components and the external dependencies they rely on, including data format specifications.
Refer to this page when preparing input data or upgrading individual components.

## GTFS Flex Compatibility

[GTFS Flex](https://gtfs.org/extensions/flex/) has undergone a significant structural change between its earlier draft specification and the version officially incorporated into the GTFS standard.
The two variants differ in how location group membership and stop times are represented.

### Specification Differences

| Field / File                       | GTFS Flex Spec Draft                          | GTFS Flex Spec Standard                  |
|------------------------------------|-----------------------------------------------|------------------------------------------|
| Location group membership          | `location_id` column in `location_groups.txt` | Separate file `location_group_stops.txt` |
| Stop reference in `stop_times.txt` | `stop_id` column                              | `location_group_id` column               |

### Component Support Matrix

Legend: `⚪ Legacy` = old/maintenance-only, `🟢 Current` = stable/released, `🛠 Planned` = upcoming/in progress.

| Component                 | Lifecycle  | Old GTFS Flex Spec | New GTFS Flex Spec (v2) |
|---------------------------|:----------:|:------------------:|:-----------------------:|
| OTP Planner (OTP v2.4.0)  |  ⚪ Legacy  |         ✅          |            ❌            |
| OTP Planner (OTP v2.6.0)  | 🟢 Current |         ❌          |            ✅            |
| OTP Planner (OTP v2.8.1)  | 🛠 Planned |         ❌          |            ✅            |
| Route Deviation Simulator | 🟢 Current |         ❌          |            ✅            |
| On-Demand Simulator       | 🟢 Current |         ✅          |            ❌            |
| On-Demand Simulator       | 🛠 Planned |         ❌          |            ✅            |
| Simple Planner            | 🟢 Current |         ✅          |            ❌            |
