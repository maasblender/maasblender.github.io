---
sidebar_position: 30
title: 実行例
---

## 必要なリポジトリのクローン

まず、Github上のリポジトリからコードをCloneします。

```
git clone https://github.com/maasblender/maasblender.git
```

今回の例で使用するのは以下のディレクトリに存在するコード・データが対象です。

```
examples/get-started/
├── compose.yml             # 必要なコンテナおよびサービスを定義する Docker Compose ファイル
└── execution.py            # シミュレータへのファイル登録、連携、実行管理を行う Python スクリプト

src/
├── base_simulators/        # スケジュール型・オンデマンド型のベースシミュレータ
├── evaluation/             # 評価処理（シミュレーション結果の分析など）コンポーネント
├── planner/                # 経路検索コンポーネント（例：OpenTripPlanner）
├── scenario/               # シナリオ生成・管理
├── simulation_broker/      # 各サービスを統括するBrokerコンポーネント
└── user_model/             # ユーザーの行動モデル（例：経路選択など）
```

## 必要なファイルの作成

これらのファイルは利用する地域やサービス、設定に応じて各自で準備してください。  
準備したファイルは`examples/get-started/` 配下に配置してください。
ここでは経路探索エンジンとして[OpenTripPlanner](https://www.opentripplanner.org/)の利用を想定します。

```
examples/get-started/
├── otp-config.zip           # OpenTripPlanner の設定ファイル
├── gtfs.zip                 # 固定ルート公共交通の GTFS データ（例：路線バス）
├── gtfs_flex.zip            # オンデマンド交通用の GTFS-Flex データ（例：デマンドバス）
└── broker_setup.json        # ブローカー構成ファイル
```

それぞれの入手または作成する方法は、追記予定です。

## シミュレーションの実行

以下のコマンドで、必要なサービスを起動し、シミュレーションを実行します、

```sh
cd examples/get-started
docker compose up -d
python execution.py
```

実行完了後、以下のファイルが生成されています。
```
output/
└── events.txt
```

events.txtはMaaS Blenderが扱う全てのイベント情報を記録したログです。
このログを分析することで対象とした交通網における利用率や利便性が算出可能です。
