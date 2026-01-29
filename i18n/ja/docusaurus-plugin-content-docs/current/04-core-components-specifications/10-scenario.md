---
sidebar_position: 10
title: "需要生成"
---

この章では、MaaS Blender の `scenario` 配下で利用できる 3 つの需要生成コンポーネントを説明します。
各コンポーネントはシミュレーション内のユーザーが行動を開始する `DEMAND` イベントを発行します。
それぞれ、需要をいつ・どのように生成するか異なります。

- 確率的ウィンドウ生成: 統計的・確率的に需要を生成したい場合に使用します。
- 履歴リプレイ: 需要のタイミングと内容を厳密に固定したい（再現やベンチマーク用途で）場合に使用します。
- 通勤パターン: 指定時刻に繰り返される日次の通勤行動をモデル化したい場合に使用します。

## 確率的ウィンドウ生成（`DemandGenerator`）

- ファイル: `maasblender/src/scenario/generator/generator.py`
- 主要クラス: `DemandGenerator`

### 動作
- 1 つ以上のウィンドウ `[begin, end)` と、そのウィンドウ内で期待される需要数を定義します。
- ジェネレーターはウィンドウを 1 分刻みのスロットに離散化し、各スロットについて次の確率でベルヌーイ試行を行います。
  `p = expected_demands / number_of_slots`。
- 成功したスロットの時刻に需要が生成されます。
- `resv`（予約発行時刻）が与えられている場合、`DEMAND` イベントは `resv` の時点で発行され、
  `dept` には想定出発時刻（`[begin, end)` 内の 1 分スロット）が設定されます。

### 設定

```json
{
  "seed": 123,                               // 再現性のための乱数シード
  "userIDFormat": "U%03d",                 // 例: U001, U002, ...
  "demandIDFormat": "D_%d",                // 例: D_1, D_2, ... 
  "demands": [
    // 複数の重なり合うウィンドウを定義可能
    {
      "begin": 10.0,                        // ウィンドウ開始（分）
      "end": 200.0,                         // ウィンドウ終了（分）
      "expected_demands": 2.0,              // ウィンドウ内の期待需要数
      "resv": 5.0,                          // 予約発行時刻（分）— 任意
      "org": {"locationId": "Org", "lat": 35.0, "lng": 139.0},
      "dst": {"locationId": "Dst", "lat": 35.7, "lng": 139.7},
      "user_type": "commuter",             // 任意
      "service": "my-mobility-service"     // 任意
    }
  ]
}
```

::::warning
1 分あたりの `p` が 0.1 を超える場合は、単位時間をより細かくしてポアソン過程の近似精度を高めてください。
::::

#### 例

```json
{
  "seed": 128,
  "userIDFormat": "U%03d",
  "demands": [
    {
      "begin": 10.0,
      "end": 200.0,
      "expected_demands": 2.0,
      "resv": 7.0,
      "org": {"locationId": "Org", "lat": 35.0, "lng": 139.0},
      "dst": {"locationId": "Dst", "lat": 35.7, "lng": 139.7},
      "service": "mobility-service-for-test"
    }
  ]
}
```

この設定ではシミュレーション時刻 7 分に、およそ 2 件の予約需要が発生します。それぞれの `dept` は `[10, 200)` ウィンドウ内の異なる時刻になります。

### 出力
- `users()` は生成済みユーザーの一覧を返します。
- `DEMAND` イベントには `userId`、自動採番された `demandId`、`org`、`dst`、任意の `service` に加え、以下が含まれます。
  - 事前予約の場合: `dept`（想定出発時刻）
  - 即時出発の場合: `dept` フィールドが存在しない（または `null`）場合「今すぐ出発」を意味します。

## 履歴リプレイ（`HistoricalScenario`）

- ファイル: `maasblender/src/scenario/historical/historical.py`
- 主要クラス: `HistoricalScenario`

### 動作
- 各入力レコードは、`DEMAND` を発行する正確な時刻と、その完全なペイロードを指定します。
- レコードに `user_id` または `demand_id` が欠けている場合、指定されたフォーマット（例: `U%03d`、`D_%d`）で自動的に補完します。

### 設定

```json
{
  "user_id_format": "U%03d",          // レコードに user_id がない場合に使用
  "demand_id_format": "D_%d",         // レコードに demand_id がない場合に使用
  "settings": [
    {
      "time": 480.0,                   // 正確な発行時刻（開始からの分）
      "org": {"locationId": "A", "lat": 35.1, "lng": 139.1},
      "dst": {"locationId": "B", "lat": 35.7, "lng": 139.7},
      "dept": 495.0,                   // 想定出発時刻
      "user_id": "U123",              // 任意；欠けていれば自動採番
      "demand_id": "D_901",           // 任意；欠けていれば自動採番
      "user_type": "visitor",         // 任意；user_id に紐づく属性
      "service": "bus-line-12",       // 任意
      "actual_duration": 23.5          // 任意；分析用メタデータ
    }
  ]
}
```

### 出力
- `users()` は既知のすべてのユーザーを返します。
- 指定された `time` ちょうどに、与えられたペイロードで `DEMAND` が発行されます。

## 通勤パターン（`CommuterScenario`）

- ファイル: `maasblender/src/scenario/commuter/commuter.py`
- 主要クラス: `CommuterScenario`

#### 動作
- 各通勤者について、毎日 2 つの `DEMAND` が生成されます。
  1) `deptOut` に往路（`org` → `dst`）
  2) `deptIn` に復路（元の方向を自動で反転）
- このパターンは 1440 分（1 日）ごとに繰り返されます。
- `demandId` は指定された `demand_id_format` から生成されます。

#### 設定

```json
{
  "demand_id_format": "D_%d",
  "commuters": {
    "U001": {
      "deptOut": 480.0,   // 08:00
      "deptIn": 1080.0,   // 18:00
      "org": {"locationId": "Home", "lat": 35.0, "lng": 139.0},
      "dst": {"locationId": "Work", "lat": 35.7, "lng": 139.7},
      "user_type": "office",
      "service": "rail"
    },
    "U002": {
      "deptOut": 510.0,   // 08:30
      "deptIn": 1110.0,   // 18:30
      "org": {"locationId": "Home2", "lat": 35.2, "lng": 139.2},
      "dst": {"locationId": "Work2", "lat": 35.6, "lng": 139.6}
    }
  }
}
```

#### 出力
- `users()` はすべての通勤者を返します。
- 毎日、`deptOut` と `deptIn` に `DEMAND` イベントが発行されます。復路イベントでは起点/目的地が自動で反転されます。

---

### 運用上の共通メモ

- いずれもイベントスケジューリングに `simpy` を使用して実装されています。
- すべての時刻はシミュレーション開始から経過した分で表します。
- `DemandGenerator` は乱数シードを用いて、確率的な結果の再現性を確保します。
- `HistoricalScenario` と `CommuterScenario` は、与えられた入力が同じであれば決定的に動作します。
