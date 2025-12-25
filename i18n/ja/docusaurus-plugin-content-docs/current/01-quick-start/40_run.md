---
sidebar_position: 40
title: シミュレーション
---

## シミュレーションの実行

まず、Docker Compose を使用して必要なサービスを起動します。

```bash
docker compose up -d
```

すべてのサービスが起動したら、シミュレーションスクリプトを実行します。

```bash
$ python execute_simulation.py
{'message': 'successfully uploaded. otp-config.zip'}
{'message': 'successfully uploaded. gtfs.zip'}
{'message': 'successfully uploaded. gtfs.zip'}
{'message': 'successfully configured.'}
{'message': 'successfully started.'}
{'message': 'successfully run.'}
running: 602.0
running: 673.0
running: 747.0
running: 818.0
running: 891.0
running: 964.0
running: 1035.0
running: 1110.0
running: 1980.0
successfully finished.
All events recorded to events.txt
```

## シミュレーション結果

シミュレーションが完了すると、`events.txt` ファイルが生成されます。
`events.txt` には、シミュレーション中に MaaS Blender によって処理されたすべてのイベントが時系列で記録されています。
これらのイベントから、シミュレーションされたモビリティサービスの利用状況などを分析することができます。