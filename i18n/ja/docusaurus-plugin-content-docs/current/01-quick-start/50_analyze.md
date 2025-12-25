---
sidebar_position: 50
title: モビリティサービスの分析
---

シミュレーションを実行すると、`events.txt` が生成されます。
このファイルには、各イベントが 1 行 1 JSON（JSON Lines 形式）として時系列に記録されています。

各イベントの構造は、[イベント仕様](../02-simulation-model/10-event.md) に定義されています。

以下では、`events.txt` に記録されたイベントを対象として、
`Python` と `pandas` を用い、ユーザー視点およびモビリティユニット視点の簡単な分析例を示します。

## ユーザー視点の分析

以下のコードでは、`DataFrame` から徒歩以外の `RESERVED` イベントのみを抽出し、分析に利用します。

```python
import pandas as pd

# 行区切り JSON の読み込み
df = pd.read_json("events.txt", lines=True)

# RESERVED イベントのみを抽出
df = df[df["eventType"] == "RESERVED"]

# 徒歩による予約を除外
df = df[df["source"] != "walking"]

# details から必要なフィールドを展開
df["userId"] = df["details"].apply(lambda d: d.get("userId"))
df["demandId"] = df["details"].apply(lambda d: d.get("demandId"))
df["success"] = df["details"].apply(lambda d: d.get("success"))
df["org"] = df["details"].apply(lambda d: d["route"][0]["org"]["locationId"] if d.get("route") else None)
df["dst"] = df["details"].apply(lambda d: d["route"][0]["dst"]["locationId"] if d.get("route") else None)

# 分析しやすいように列を整理・並び替え
df["service"] = df["source"]
df = df[["time", "userId", "demandId", "service", "org", "dst", "success"]]
df = df.sort_values(["userId", "time"])
```

### 出力例

```yaml
             time  userId demandId service   org   dst  success
69     563.309169  CU_001      D_1    gtfs  1_01  7_01     True
2759  1120.356102  CU_001      D_2    gtfs  7_02  2_02     True
```

この表から、以下の点を確認できます。

- ユーザー (`userId`) がどのモビリティサービス (`service`) を予約し、その予約が成功したかどうか (`success`)
- 各移動の出発地 (`org`) と到着地 (`dst`)
- 予約のタイミング (`time`) や頻度と、ユーザー需要 (`demandId`) との関係

## モビリティ視点の分析

以下のコードでは、`DataFrame` からユーザーがモビリティユニットに乗車した区間のイベントのみを抽出し、分析に利用します。

```python
import pandas as pd

# 行区切り JSON の読み込み
df = pd.read_json("events.txt", lines=True)

# DEPARTED と ARRIVED イベントのみを抽出
df = df[df["eventType"].isin(["DEPARTED", "ARRIVED"])]

# details から mobilityId と userId を抽出
df["mobilityId"] = df["details"].apply(lambda d: d.get("mobilityId"))
df["userId"] = df["details"].apply(lambda d: d.get("userId"))
df["demandId"] = df["details"].apply(lambda d: d.get("demandId"))
df["locationId"] = df["details"].apply(lambda d: d["location"].get("locationId") if d.get("location") else None)

# mobilityId と userId の両方が存在する行のみを残す
df = df[df["mobilityId"].notna() & df["userId"].notna()]

# 必要な列を選択
df = df[["time", "eventType", "userId", "mobilityId", "demandId", "locationId"]]

# userId・mobilityId・time で並び替え
df = df.sort_values(["userId", "mobilityId", "time"]).reset_index(drop=True)
```

### 出力例

```yaml
     time eventType  userId               mobilityId demandId locationId
0   570.0  DEPARTED  CU_001  1平日・土日祝_09時30分_系統101001      D_1       1_01
1   578.0   ARRIVED  CU_001  1平日・土日祝_09時30分_系統101001      D_1       7_01
2  1121.0  DEPARTED  CU_001  1平日・土日祝_18時00分_系統101002      D_2       7_02
3  1131.0   ARRIVED  CU_001  1平日・土日祝_18時00分_系統101002      D_2       2_02
```

この表から、以下の点を把握できます。
- どのユーザー (`userId`) が、どのモビリティユニット (`mobilityId`) に乗車したか
- 各乗車区間における出発時刻 (`time`) および到着時刻 (`time`)