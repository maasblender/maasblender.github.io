---
sidebar_position: 50
title: Analyze Output Events
---

After running a simulation, events are output in JSON format as specified in the [Event Specification](../02-simulation-model/10-event.md).

Here we demonstrate simple analysis techniques focusing on users and mobility units,
using `Python` and `pandas` for quick exploration.

## User-focused analysis

The following code selects all non-walking `RESERVED` events in a `DataFrame` for further analysis.

```python
import pandas as pd

# Read line-delimited JSON
df = pd.read_json("events.txt", lines=True)

# Extract RESERVED events
df = df[df["eventType"] == "RESERVED"]

# Exclude walking-based reservations
df = df[df["source"] != "walking"]

# Expand relevant fields from 'details'
df["userId"] = df["details"].apply(lambda d: d.get("userId"))
df["demandId"] = df["details"].apply(lambda d: d.get("demandId"))
df["success"] = df["details"].apply(lambda d: d.get("success"))
df["org"] = df["details"].apply(lambda d: d["route"][0]["org"]["locationId"] if d.get("route") else None)
df["dst"] = df["details"].apply(lambda d: d["route"][0]["dst"]["locationId"] if d.get("route") else None)

# Select and sort columns for easier analysis
df["service"] = df["source"]
df = df[["time", "userId", "demandId", "service", "org", "dst", "success"]]
df = df.sort_values(["userId", "time"])
```

Output:
```
             time  userId demandId service   org   dst  success
69     563.309169  CU_001      D_1    gtfs  1_01  7_01     True
2759  1120.356102  CU_001      D_2    gtfs  7_02  2_02     True
```

This table allows you to see:
- Which services users reserved and whether the reservations succeeded
- The origin and destination of each trip
- How the timing and frequency of reservations relate to user demand

## Mobility-focused analysis

The following code selects only the segments where users actually rode a mobility unit in a `DataFrame` for further analysis.

```python
import pandas as pd

# Read line-delimited JSON
df = pd.read_json("events.txt", lines=True)

# Keep only DEPARTED and ARRIVED events
df = df[df["eventType"].isin(["DEPARTED", "ARRIVED"])]

# Extract mobilityId and userId from details
df["mobilityId"] = df["details"].apply(lambda d: d.get("mobilityId"))
df["userId"] = df["details"].apply(lambda d: d.get("userId"))
df["demandId"] = df["details"].apply(lambda d: d.get("demandId"))
df["locationId"] = df["details"].apply(lambda d: d["location"].get("locationId") if d.get("location") else None)
df["time"] = df["time"]

# Keep only rows where both mobilityId and userId are present
df = df[df["mobilityId"].notna() & df["userId"].notna()]

# Select relevant columns
df = df[["time", "eventType", "userId", "mobilityId", "demandId", "locationId"]]

# Sort by userId, mobilityId, and time
df = df.sort_values(["userId", "mobilityId", "time"]).reset_index(drop=True)
```

Output:
```
     time eventType  userId               mobilityId demandId locationId
0   570.0  DEPARTED  CU_001  1平日・土日祝_09時30分_系統101001      D_1       1_01
1   578.0   ARRIVED  CU_001  1平日・土日祝_09時30分_系統101001      D_1       7_01
2  1121.0  DEPARTED  CU_001  1平日・土日祝_18時00分_系統101002      D_2       7_02
3  1131.0   ARRIVED  CU_001  1平日・土日祝_18時00分_系統101002      D_2       2_02
```

This table allows you to see:
- Which users rode which mobility units and during which time intervals
- The departure and arrival times for each passenger trip segment
