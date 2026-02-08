---
sidebar_position: 20
title: "ユーザーモデル"
---

本章では、MaaS Blenderにおけるユーザーモデルコンポーネントのリファレンス実装について説明します。
ユーザーモデルは、ユーザーの移動全体を通じた意思決定ロジックを表現します。
プランナーとモビリティサービスを、イベント駆動型の相互作用を通じて連携させることで、ユーザーの移動計画、経路選択、予約、移動の実行を管理します。

本プロジェクトには、`maasblender/src/user_model` に2つのリファレンス実装が用意されています:
- シンプルユーザーモデル：単純な経路選択を行う。
- お気に入りベースユーザーモデル：ユーザーごとのお気に入いサービスや並び替え基準など、より高度に設定できる。

どちらの実装も同じイベントの仕組みとライフサイクルを共有していますが、経路選択ポリシーと設定オプションが異なります。

### 共通ライフサイクル

すべてのユーザーモデル実装は以下の一般的なフローに従います：

1. `DEMAND` をユーザーモデルに `UserManager.demand(...)` で転送します。
2. `UserManager` は経路プランナー（`Planner`）を `plan(org, dst, dept)` で呼び出します。
3. ユーザーモデルは、その選択ポリシー（実装ごとに異なる）に応じて経路候補をランク付け/フィルタリングします。
4. 選択された経路は、タスク（`Reserve`、`Trip`、`Wait`）のシーケンスに変換されます。
5. `User` エンティティはタスクを順に実行し、適切なタイミングで `RESERVE` や `DEPART` を発行します。
6. `RESERVED` 失敗時には、ユーザーモデルは代替プランへのフォールバックを実施します。
7. ユーザーモデルは `DEPARTED` と `ARRIVED` を監視して、乗り継ぎを調整します。

## シンプルユーザーモデル

- ファイル：`user_model/simple/`
- プライマリクラス：`UserManager`

このモデルは、小規模な設定セットを備えたユーザー決定ポリシーを提供します。

### 動作

- `Planner.plan(org, dst, dept)` を呼び出して経路候補を取得します。
- `DEMAND` が `service` を提供する場合、設定モードを適用します：
  - `PreferenceMode.fixed`：指定されたサービスを含む計画のみを保持します。
  - その他：そのサービスを含む計画が最初に来るようにソートします。
- 一つの計画のみが返された場合、その計画に対してタスクを構築します。予約失敗時のフォールバックに徒歩を設定します。
- 二つ以上の計画が返された場合、一つ目の計画に対してタスクを構築します。予約失敗時のフォールバックに二つ目の計画をタスクに設定します。
- 経路に予約が必要なサービスが含まれる場合、`UserManager` はその区間に対して `Reserve` タスクを構築します。それ以外の場合、区間は直接 `Trip` タスク（例：徒歩またはサービス予約なし）として構築されます。
- `dept > now` で最初のタスクが `Trip` の場合、最初の区間の前に `Wait(dept)` タスクが挿入されます。

### 設定

```json
{
  "preference_mode": "fixed",                   // または他の値（設定モード用）
  "confirmed_services": ["taxi", "bus"]         // 予約が必要なサービス
}
```

- `preference_mode` （`jschema.query.PreferenceMode` から）：
  - `"fixed"`：需要ごとに指定されたサービスを含む計画のみにフィルタリングします。
  - その他：指定されたサービスを含む計画を優先していますが、他の計画も許可します。
- `confirmed_services`：予約が必要なサービスのリスト。

## お気に入りベースユーザーモデル

- ファイル：`user_model/favorite/`
- プライマリクラス：`UserManager`

このモデルは、お気に入りのサービス、徒歩時間制限、候補ソートなど、ユーザーごとの好みに基づく経路設定を提供します。

### 動作

- 各ユーザーが `RouteFilter` インスタンスを取得します。`UserType` が提供されている場合、フィルターは `FavoriteSortedRouteFilter` になり、以下が行われます：
  - `SortType` で計画をソート（例：最も早い到着、乗り継ぎ最小、最低コスト）。
  - お気に入いサービスの存在確認を行い、該当する場合は徒歩時間上限を適用します。
- `DEMAND` に `fixed_service` が指定されている場合、それが優先されます：計画がフィルタリングされ、指定されたサービスが含まれるようにします。
- 待機と失敗処理はシンプルモデルと同様です。

### 設定

```json
{
  "user_params": {
    "U001": {
      "favorite_service": "rail",
      "walking_time_limit_min": 15.0,
      "sort_type": "earliest_arrival"
    },
    "U002": {
      "favorite_service": "bus",
      "walking_time_limit_min": 10.0,
      "sort_type": "least_transfers"
    },
    "U003": null
  },
  "confirmed_services": ["taxi", "shuttle"]
}
```

- `user_params`：ユーザーIDをその設定（`UserType` または `null`）にマッピングする辞書：
  - `favorite_service`：優先されるモビリティサービス（例：`"rail"`、`"bus"`）。
  - `walking_time_limit_min`：許容最大徒歩時間（分）。
  - `sort_type`：`jschema.query.SortType` からのソート基準（例：`"earliest_arrival"`、`"least_transfers"`、`"lowest_cost"`）。
- `confirmed_services`：予約が必要なサービスのリスト（シンプルモデルと同じ）。

