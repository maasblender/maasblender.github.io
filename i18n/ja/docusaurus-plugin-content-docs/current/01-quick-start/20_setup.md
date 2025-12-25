---
sidebar_position: 20
title: セットアップ
---

## リポジトリのクローン

まず、Maas Blender のリポジトリをクローンし、サンプルプロジェクトのディレクトリに移動します。

```shell
git clone https://github.com/maasblender/maasblender.git
cd maasblender
git checkout tags/v0.8.0
cd maasblender/examples/01-quick-start
```

このディレクトリには、Docker Compose を使用して Maas Blender の動作を試すための最小構成が含まれます。

```
/examples/quick-start/
├── broker_setup.json     # ブローカーの設定ファイル
├── compose.yml           # 必要なコンテナおよびサービスを定義した Docker Compose ファイル
├── execute_simulation.py # シミュレータとのファイル登録・連携・実行管理を行う Python スクリプト
├── gtfs.zip              # サンプルの GTFS ファイル
└── otp-config.zip        # OpenTripPlanner 用の設定ファイル
```

## シミュレーションデータの準備

Maas Blender は、[GTFS](https://gtfs.org/), [GTFS-Flex](https://gtfs.org/extensions/flex/), [GBFS](https://gbfs.org/)
などのオープンなモビリティデータ標準をサポートしています。

このクイックスタートでは、富山市の「まいどはや」バスの [GTFS データ](https://opdt.city.toyama.lg.jp/dataset/toyamacity-bus-gtfs-jp/resource/43903d9f-1d9c-42f0-bd01-8a5fc1cb828c) を使用します。

:::info
実際のシミュレーションでは、任意の GTFS データセットを使用することができます。
:::
