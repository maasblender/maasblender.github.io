---
sidebar_position: 30
title: "経路プランナー"
---

本章では、MaaS Blenderにおける経路プランナーコンポーネントを説明します。
経路プランナーは、ユーザーの移動意図（`org`、`dst`、`dept`）を1つ以上の経路候補に変換します。各経路は順序付けされた区間（`leg`）で構成されます。
各経路は、移動元から目的地への移動方法を表し、複数の交通手段が関わる場合もあります。

本プロジェクトは、`maasblender/src/planner` の下に2つのリファレンス実装を用意しています：
- シンプルプロセス内プランナー：ネットワーク上での決定的な経路探索を実行します。
- OpenTripPlannerを使用したサービス：GTFSとOpenStreetMapデータを使用したマルチモーダルな経路探索を実行します。

どちらの実装も同じインターフェースを提供します。移動意図が与えられると、`Route` オブジェクトのリストを返します。

## シンプルプロセス内プランナー

- ファイル：`maasblender/src/planner/simple/`
- プライマリクラス：`SimplePlanner`

このプランナーは、移動ネットワーク（通常はGTFSまたはGBFSファイルから構築）に対する軽量なルーティング機能を提供します。

### 動作

- プランナーは与えられたデータから `MobilityNetwork` を構築します
  - ノードは位置情報（停留所、駅、または目的地）を表します。
  - エッジは移動サービスを表し、関連する移動時間とスケジュールを持ちます。
- `(org, dst, dept)` が与えられると、プランナーはネットワークから可能なパスを探索します。
  - 経路は1つ以上の区間で構成され、各区間は特定のサービスを使用します。
  - 徒歩はフォールバックとして計算され、距離ベースの移動時間を持ちます。
- 同じ入力に対して一貫した結果を生成します（ランダム性なし）。
- **出力形式**：`Route` オブジェクトのリストを返し、各オブジェクトには以下が含まれます：
  - `dept`：全体の出発時刻
  - `arrv`：全体の到着時刻
  - `trips`：個別の区間のリスト。各区間には `org`、`dst`、`dept`、`arrv`、`service` が含まれます。

### 設定

シンプルプランナーは通常、GTFSまたはGBFSファイル入力を含むブローカーセットアップを通じて設定されます。

#### GTFS設定

```json
{
  "planner": {
    "type": "planner",
    "endpoint": "http://planner",
    "details": {
      "networks": {
        "gtfs": {
          "type": "gtfs",
          "input_files": [
            {
              "filename": "gtfs.zip"
            }
          ],
          "agency_id": "7230001002032"          // 任意：特定の事業者でフィルタリング
        }
      },
      "reference_time": "20251016",             // YYYYMMDD形式
      "walking_meters_per_minute": 50.0         // 徒歩速度
    }
  }
}
```

- `networks`：ルーティングに使用する1つ以上の移動ネットワークを定義します。
  - `type`：`"gtfs"` はGTFSベースのネットワーク構築を示します。
  - `input_files`：読み込むGTFS zip ファイルのリスト（API経由で個別にアップロード）。
  - `agency_id`：交通事業者
- `reference_time`：YYYYMMDD形式のシミュレーション参照日付。
- `walking_meters_per_minute`：歩行区間の推定歩行速度。

### 出力

- `Route` オブジェクトのリストを返します。各オブジェクトは実行可能な移動選択肢を表します。
- 各 `Route` は以下を含みます：
  - `dept`：全体の出発時刻（シミュレーション開始からの経過した分）。
  - `arrv`：全体の到着時刻（シミュレーション開始からの経過した分）。
  - `trips`：順序付けされた区間のリスト。各区間には以下が含まれます：
    - `org`：移動元位置情報（`locationId`、`lat`、`lng`）。
    - `dst`：目的地位置情報（`locationId`、`lat`、`lng`）。
    - `dept`：区間の出発時刻。
    - `arrv`：区間の到着時刻。
    - `service`：サービス識別子（例：`"walk"`、`"bus_line_1"`）。
- 経路は通常、到着時刻でソートされていますが、実装の詳細によってソート順が異なる場合があります。

#### GBFS設定

```json
{
  "planner": {
    "type": "planner",
    "endpoint": "http://planner",
    "details": {
      "networks": {
        "bike_share": {
          "type": "gbfs",
          "input_files": [
            {
              "filename": "gbfs_feed.json"
            }
          ]
        }
      },
      "reference_time": "20251016",
      "walking_meters_per_minute": 50.0
    }
  }
}
```

## OpenTripPlannerを使用したサービス

- ファイル：`maasblender/src/planner/opentripplanner/`
- プライマリクラス：`OTPPlanner`

このプランナーはOpenTripPlanner（OTP）と統合します。OTPはオープンソースのマルチモーダル経路計画エンジンです。
Maas Blenderでは、OTPは通常、ブローカーセットアップ経由でGTFSおよびGBFSファイルを使用して設定されます。

### 動作

- プランナーはREST APIを介してOTPサーバーと通信します。
- OTPはアップロードされたファイルからルーティンググラフを構築します：
  - GTFSは交通スケジュール、GBFSはバイクシェアステーション用。
  - OTP設定はグラフ構築設定とルーティング設定用。
  - OpenStreetMap（OSM）は徒歩、自転車、および道路ネットワーク用。
- 移動元、目的地、出発時刻を含む計画リクエストをOTPエンドポイントに送信します。
  - OTPは1つ以上の旅程（itinerary）を返し、各旅程は異なるモード（徒歩、バス、電車など）で構成された区間を含みます。
  - プランナーはOTPの応答をMaaS Blenderの `Route` 形式に変換します。
- 徒歩 → バス → 乗り継ぎ → 電車 → 徒歩のような複雑な組み合わせ (マルチモーダル) を処理します。

### 設定

OTPプランナーはブローカーセットアップを通じて、GTFS/GBFSファイル入力とOTP設定で設定されます：

:::warning
ブローカーセットアップを開始する前に、必要なすべてのファイル（`otp-config.zip`、`gtfs.zip`など）がアップロードされていることを確認してください。
OTPグラフ構築プロセスはこれらのファイルが必要であり、不足している場合は失敗します。
:::

#### 基本設定構造

```json
{
  "planner": {
    "type": "planner",
    "endpoint": "http://planner",
    "details": {
      "otp_config": {
        "input_files": [
          {
            "filename": "otp-config.zip"
          }
        ]
      },
      "networks": {
        "gtfs": {
          "type": "gtfs",
          "input_files": [
            {
              "filename": "gtfs.zip"
            }
          ],
          "agency_id": "7230001002032"          // 任意：特定の事業者でフィルタリング
        }
      },
      "reference_time": "20251016",             // YYYYMMDD形式（必須、8文字）
      "modes": ["WALK", "TRANSIT"],             // 任意：許可されている交通モード
      "walking_meters_per_minute": 50.0,        // 任意：Noneの場合、router_config.jsonから読み込み
      "timezone": 9                              // 任意：タイムゾーンオフセット（デフォルト：+9）
    }
  }
}
```

**必須パラメータ：**
- `otp_config`：OTP設定ファイル（OSMデータ、ビルド設定、ルーター設定を含む）。
  - `input_files`：アップロードする設定zipファイルのリスト。
- `networks`：ネットワーク設定（GTFS、GBFSなど）の辞書。
- `reference_time`：YYYYMMDD形式のシミュレーション参照日付（正確に8文字である必要があります）。

**オプションパラメータ：**
- `modes`：許可されている交通モード（例：`["WALK", "TRANSIT", "BICYCLE"]`）のリスト。指定されない場合、OTPは利用可能なすべてのモードを使用します。
- `walking_meters_per_minute`：歩行速度。`null` の場合、OTPの `router_config.json` から値が読み込まれます。
- `timezone`：タイムゾーンオフセット（時単位、デフォルト：`+9` JST用）。

#### OTP設定ファイル（otp-config.zip）

`otp-config.zip` には以下が含まれます：

1. オープンストリートマップデータ (例えば `map.osm.pbf`)
2. `build-config.json` (任意)：
   ```json
   {
     "areaVisibility": true,
     "platformEntriesLinking": true,
     "matchBusRoutesToStreets": true
   }
   ```
3. `router-config.json` (任意)：
   ```json
   {
     "routingDefaults": {
       "walkSpeed": 1.4,
       "bikeSpeed": 5.0,
       "carSpeed": 15.0
     }
   }
   ```
 
これらの設定ファイルは、OTPがルーティンググラフを構築する方法と、経路計算のデフォルトパラメータを制御します。

:::info
OTP設定オプションの詳細については、[OpenTripPlannerドキュメント](https://docs.opentripplanner.org/) を参照してください。
:::

## 共通操作上の注意事項

- 両プランナーは同じインターフェースを実装します：`plan(org, dst, dept)` → `List[Route]`。
- すべての位置情報はWGS84座標（緯度/経度）を使用します。

