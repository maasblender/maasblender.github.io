---
sidebar_position: 40
title: シミュレーション
---

## シミュレーションの実行

`examples/quick-start` ディレクトリで、Compose を使用して必要なサービスを起動します。
初回のビルドではコンテナイメージや依存パッケージをダウンロードするため、数分かかることがあります。

```bash
docker compose up -d
docker compose ps -a
```

6サービス（`broker`、`walking`、`user_model`、`scenario_commuter`、`scheduled`、`planner`）がすべて稼働していることを確認してください。

```bash
python execute_simulation.py
```

スクリプトは同梱ファイルのアップロード、各サービスの設定、シミュレーションの実行を行います。OpenTripPlanner の初期化には数分かかることがあります。

出力例（進捗の数値は実行環境によって変わります）：

```text
============================================================
GTFS Validity Check
============================================================
[✅ OK] reference_time=20251016 is within GTFS valid period (2025-10-16 - 2026-03-31) for service 'gtfs'.
{'message': 'successfully uploaded. otp-config.zip'}
{'message': 'successfully uploaded. gtfs.zip'}
{'message': 'successfully uploaded. gtfs.zip'}
{'message': 'successfully configured.'}
{'message': 'successfully started.'}
{'message': 'successfully run.'}
running: 575.0
...
All events recorded to events.txt
successfully finished.
```

## シミュレーション結果

スクリプトが終了すると、カレントディレクトリの `events.txt` に結果が保存されます。
`events.txt` には、シミュレーション中に MaaS Blender によって処理されたすべてのイベントが時系列で記録されています。
これらのイベントから、シミュレーションされたモビリティサービスの利用状況などを分析することができます。

## サービスの停止

サンプルのコンテナとネットワークを削除します。

```shell
docker compose down
```
