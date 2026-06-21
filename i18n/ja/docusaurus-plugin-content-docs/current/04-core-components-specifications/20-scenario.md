---
sidebar_position: 20
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
- サンプリングされたスロットの意味は `arrive_by` によって変わります。
  - `arrive_by: false`（デフォルト）: スロットは希望出発時刻として扱われます。
  - `arrive_by: true`: スロットは希望到着時刻として扱われます。
- `resv`（予約発行時刻）は、`DEMAND` イベントをいつ発行するかを決めます。`resv` が指定されない場合、Leave-at 需要はサンプリングされたスロットの時刻に、Arrive-by 需要はシミュレーション開始直後の `0.0` に発行されます。

### 設定

```json5
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

```json5
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

#### Arrive-by の例

```json5
{
  "seed": 129,
  "userIDFormat": "U%03d",
  "demands": [
    {
      "begin": 10.0,
      "end": 200.0,
      "expected_demands": 2.0,
      "arrive_by": true,
      "resv": 5.0,
      "org": {"locationId": "Org", "lat": 35.0, "lng": 139.0},
      "dst": {"locationId": "Dst", "lat": 35.7, "lng": 139.7},
      "user_type": "test-user"
    }
  ]
}
```

この設定では、各 `DEMAND` は 5.0 分に発行され、`arrv` には `[10, 200)` の中からサンプリングされた到着時刻が入ります。同じ設定で `resv` を省略すると、到着時刻制約はそのままで、`DEMAND` 自体はシミュレーション時刻 `0.0` に発行されます。

### 出力
- `users()` は生成済みユーザーの一覧を返します。
- `DEMAND` イベントには `userId`、自動採番された `demandId`、`org`、`dst`、任意の `service` に加え、ちょうど 1 つの時間制約が入ります。

| モード | `resv` | イベント発行時刻       | 時間制約                                 |
|------|--------|----------------|--------------------------------------|
| Leave-at | なし | `sampled_slot` | `dept = null`, `arrv = null`         |
| Leave-at | あり | `resv`         | `dept = sampled_slot`, `arrv = null` |
| Arrive-by | なし | `0.0`          | `dept = null`, `arrv = sampled_slot` |
| Arrive-by | あり | `resv`         | `dept = null`, `arrv = sampled_slot` |

## 履歴リプレイ（`HistoricalScenario`）

- ファイル: `maasblender/src/scenario/historical/historical.py`
- 主要クラス: `HistoricalScenario`

### 動作
- 各入力レコードは、`DEMAND` を発行する正確な時刻と、その完全なペイロードを指定します。
- レコードに `user_id` または `demand_id` が欠けている場合、指定されたフォーマット（例: `U%03d`、`D_%d`）で自動的に補完します。
- 各レコードでは、`dept`（leave-at）または `arrv`（arrive-by）のどちらか一方だけを指定します。
- `time` を省略した場合、実装は次のように補完します。
  - leave-at: `time = dept`
  - arrive-by: `time = 0.0`

### 設定

```json5
{
  "userIDFormat": "U%03d",            // レコードに user_id がない場合に使用
  "demandIDFormat": "D_%d",           // レコードに demand_id がない場合に使用
  "trips": [
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
- arrive-by 需要には `arrv` が設定され、`dept = null` になります。

## 通勤パターン（`CommuterScenario`）

- ファイル: `maasblender/src/scenario/commuter/commuter.py`
- 主要クラス: `CommuterScenario`
- 用途: 各通勤者について、往路（自宅 → 職場）と復路（職場 → 自宅）の 2 つの需要を毎日繰り返し生成します。各区間は leave-at / arrive-by を独立に選べます。

#### 動作
- 各通勤者について、毎日 2 つの `DEMAND` が生成されます。
  1) 往路（`org` → `dst`）
  2) 復路（`dst` → `org`、起点と終点は自動で反転）
- 各区間では、次のどちらか一方だけを選びます。
  - leave-at: `deptOut` / `deptIn`
  - arrive-by: `arrvOut` / `arrvIn`
- leave-at の区間では、設定された出発時刻に `DEMAND` を発行し、`dept` を設定します。
- arrive-by の区間では、目標到着時刻の `leadTime` 分前に `DEMAND` を発行し、`arrv` を設定します。
- このパターンは 1440 分（1 日）ごとに繰り返されます。
- `demandId` は指定された `demandIDFormat` から生成されます。
- スキーマでは次の検証が行われます。
  - `deptOut` と `arrvOut` の両方指定は不可、かつどちらか一方は必須
  - `deptIn` と `arrvIn` の両方指定は不可、かつどちらか一方は必須
  - 往路の `DEMAND` 発行時刻は、復路の発行時刻以下でなければならない

#### 設定

```json5
{
  "demandIDFormat": "D_%d",
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
      "arrvOut": 540.0,   // 09:00 目標到着時刻
      "arrvIn": 1140.0,   // 19:00 帰宅側の目標到着時刻
      "leadTime": 15.0,   // arrive-by 需要を 15 分前に発行
      "org": {"locationId": "Home2", "lat": 35.2, "lng": 139.2},
      "dst": {"locationId": "Work2", "lat": 35.6, "lng": 139.6}
    }
  }
}
```

#### 出力
- `users()` はすべての通勤者を返します。
- 毎日、通勤者ごとに 2 件の `DEMAND` イベントが発行されます。
- 復路イベントでは起点/目的地が自動で反転されます。
- 各イベントは時間制約をちょうど 1 つだけ持ちます（leave-at は `dept`、arrive-by は `arrv`）。

---

### 運用上の共通メモ

- いずれもイベントスケジューリングに `simpy` を使用して実装されています。
- すべての時刻はシミュレーション開始から経過した分で表します。
- `DemandGenerator` は乱数シードを用いて、確率的な結果の再現性を確保します。
- `HistoricalScenario` と `CommuterScenario` は、与えられた入力が同じであれば決定的に動作します。
