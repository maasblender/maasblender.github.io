---
sidebar_position: 11
title: ユニットイベントサイクル
---

## ユーザーのイベントサイクル

シミュレーションにおけるユーザーの行動は、需要生成から到着までの 6 段階のイベント列としてモデル化されます。

```
DEMAND → RESERVE → RESERVED → DEPART → DEPARTED → ARRIVED
```

複数のモビリティサービスを利用する場合、各サービスごとに `RESERVE` から `DEPARTED` までのイベントが繰り返されます。

```
DEMAND → RESERVE → RESERVED → DEPART → DEPARTED → ARRIVED
       → RESERVE → RESERVED → DEPART → DEPARTED → ARRIVED
       ... (モビリティサービスの数だけ繰り返す。)
```

## モビリティイベントサイクル

モビリティユニット（例：バス、タクシー、シェア車両）は、ユーザーの需要を満たすために地点間を繰り返し移動します。
モビリティユニット自身が需要を生成することはなく、モビリティユニットの行動は出発と到着のみで構成されます。

最初の `ARRIVED` は、シミュレーション開始時点での初期位置を表します。
各 `DEPARTED` → `ARRIVED` は、ルート上の 1つの移動区間に対応しています。

```
ARRIVED (初期配置)
  → DEPARTED → ARRIVED
  → DEPARTED → ARRIVED
  ... (繰り返す。)
```
