---
title: "Raspberry Piで測定したCO2濃度をAWS IoT Core経由でTimestreamに保存しGrafana Cloudで可視化"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iot", "raspberrypi", "awsiotcore", "timestream", "grafana"]
published: false
---

[前回](https://zenn.dev/thorie/articles/548emb_raspberrypi_room_condition)、Raspberry Pi Zero WHにCO2センサとLCDディスプレイを繋いで、室内のCO2濃度、温度、湿度を測定して表示しました。

今回は測定データを
1. [AWS IoT Core](https://aws.amazon.com/jp/iot-core/)に送信し
2. AWS IoT Coreがデータ受信イベント発生時に[AWS Lambda](https://aws.amazon.com/jp/lambda/)を呼び出し
3. AWS Lambdaが[Amazon Timestream](https://aws.amazon.com/jp/timestream/)にデータを保存する

ようにします。そして、

1. [Grafana Cloud](https://grafana.com/products/cloud/)上で、Timestreamの時系列データを可視化する

ようにします。

では、やっていきましょう。

## AWS IoT Coreでモノを作成

「モノ」ってなんやねんw まあ単純に「Things」の訳なのでしょうが。「ブツ」って訳すとヤバげですしね。物理的なデバイスだけではなくて論理的なモノも含めるからモノって表現とするしかないのでしょう。ただ、AWSコンソール上ではモノとデバイスとの表現がごちゃまぜですが…

ここではRasbrerry Piの物理デバイスのデータの送信先としてモノを、以下の手順で作成します。

1. AWSコンソールの[AWS IoT Core](https://us-west-2.console.aws.amazon.com/iot/home)の画面を表示します。
2. 画面左のナビゲーション・ペインから`管理` - `すべてのデバイス` - `モノ`を選択します。  右側の`モノを作成`ボタンをクリックします。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/IoTCore_Things.png?raw=true)
3. `モノを作成`画面で`1 つのモノを作成`を選択し`次へ`をクリックします。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/IoTCore_ThingCreate.png?raw=true)
4. `モノのプロパティを指定`画面の`モノの名前`に`raspberrypi-room-condition`と入力し`次へ`をクリックします。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/IoTCore_ThingProperty.png?raw=true)
5. `デバイス証明書を設定`画面で`新しい証明書を自動生成`を選択し`次へ`をクリックします。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/IoTCore_ThingDeviceCert.png?raw=true)
6. `証明書にポリシーをアタッチ`画面で`ポリシーを作成`をクリックします。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/IoTCore_ThingPolicyAttach.png?raw=true)
7. 別タブで`ポリシーを作成`画面が表示されます。`ポリシー名`に``raspberrypi-room-condition`と入力します。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/IoTCore_PolicyCreate.png?raw=true)
8. `ポリシードキュメント`で`JSON`をクリックし、`ポリシードキュメント`に以下を入力`次へ`をクリックします。(XXXXXXXXXXXXはAWSアカウントIDです。リージョンは適宜変更します)  
   ```json
   {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "iot:Connect",
                "Resource": "arn:aws:iot:us-west-2:XXXXXXXXXXXX:client/raspberrypi-room-condition"
            },
            {
                "Effect": "Allow",
                "Action": "iot:Publish",
                "Resource": "arn:aws:iot:us-west-2:XXXXXXXXXXXX:topic/room_condition"
            }
        ]
    }
   ```
9. `証明書にポリシーをアタッチ`画面に戻り、`ポリシーを作成`ボタンの左の`更新ボタン`をクリックして作成したポリシーを選択し`モノを作成`ボタンをクリックします。
10. 画面左で`設定`を選択し`デバイスデータエンドポイント`の値(`XXXXXXXXXXXXXX-XXX.iot.us-west-2.amazonaws.com`)を控えておきます。

これでAWS IoT Core側の設定は完了です。

## Raspberry Piから測定データをAWS IoT Coreへ送信

受信側(AWS IoT Core)の準備が出来たので、送信側(Raspberry Pi)からデータを送信する部分を実装していきます。

### パッケージのインストール

Raspberry Piで以下のコマンドを実行して、AWS IoT SDK for Python v2 をインストールします。

```bash
sudo pip install awsiotsdk
```

### 証明書の格納

Raspberry Piで以下のコマンドを実行し、証明書等を格納するディレクトリを作成します。

```bash
mkdir certificates
```

証明書等をダンロードしたPCから、必要なファイルをRaspberry Piにコピーします。(`XXXXXXXXXXXXXXXXXXXX`の部分はダウンロードしたファイルに合わせます)

```bash
scp XXXXXXXXXXXXXXXXXXXX-certificate.pem.crt pi@raspberrypi.local:~/certificates
scp XXXXXXXXXXXXXXXXXXXX-private.pem.key  pi@raspberrypi.local:certificates
scp AmazonRootCA1.pem pi@raspberrypi.local:certificates
```

### データ送信を実装

前回作成した[ファイル](https://zenn.dev/thorie/articles/548emb_raspberrypi_room_condition#lcd%E3%83%87%E3%82%A3%E3%82%B9%E3%83%97%E3%83%AC%E3%82%A4%E3%81%AB%E6%B8%AC%E5%AE%9A%E5%80%A4%E3%82%92%E8%A1%A8%E7%A4%BA)から以下の部分を追加します。XXXXの部分は適宜修正します。(全体のコードは[こちら](https://github.com/horie-t/programming-study/blob/master/raspberrypi_room_condition/measure_room_condition.py))


```python:/home/pi/measure_room_condition.py
#!/usr/bin/env python3

import time
import json
import sys

import digitalio
import board

# (中略)

from scd30_i2c import SCD30

from awscrt import io, mqtt, auth, http
from awsiot import mqtt_connection_builder

# Define ENDPOINT, CLIENT_ID, PATH_TO_CERTIFICATE, PATH_TO_PRIVATE_KEY, PATH_TO_AMAZON_ROOT_CA_1, MESSAGE, TOPIC, and RANGE
ENDPOINT = "XXXXXXXXXXXXXX-XXX.iot.us-west-2.amazonaws.com"
CLIENT_ID = "raspberrypi-room-condition"
PATH_TO_CERTIFICATE = "certificates/XXXXXXXXXXXXXXXXXXXX-certificate.pem.crt"
PATH_TO_PRIVATE_KEY = "certificates/XXXXXXXXXXXXXXXXXXXX-private.pem.key"
PATH_TO_AMAZON_ROOT_CA_1 = "certificates/AmazonRootCA1.pem"
TOPIC = "room_condition"

def init_display():
    """Initialize display, and return display object"""
    # Configuration for CS and DC pins (these are PiTFT defaults):

# (中略)

text_color1 = "#D9E5FF"

# Spin up resources
event_loop_group = io.EventLoopGroup(1)
host_resolver = io.DefaultHostResolver(event_loop_group)
client_bootstrap = io.ClientBootstrap(event_loop_group, host_resolver)
mqtt_connection = mqtt_connection_builder.mtls_from_path(
            endpoint=ENDPOINT,
            cert_filepath=PATH_TO_CERTIFICATE,
            pri_key_filepath=PATH_TO_PRIVATE_KEY,
            client_bootstrap=client_bootstrap,
            ca_filepath=PATH_TO_AMAZON_ROOT_CA_1,
            client_id=CLIENT_ID,
            clean_session=False,
            keep_alive_secs=6
            )

print("Connecting to {} with client ID '{}'...".format(ENDPOINT, CLIENT_ID), file=sys.stderr)

# Make the connect() call
connect_future = mqtt_connection.connect()
# Future.result() waits until a result is available
connect_future.result()
print("Connected!", file=sys.stderr)

# Setup SCD30
scd30 = SCD30()

measurment_interval_sec = 10
# (中略)

    disp.image(image)

    message = {"temperature" : temp, "co2": co2, "humidity": humi}
    mqtt_connection.publish(topic=TOPIC, payload=json.dumps(message), qos=mqtt.QoS.AT_LEAST_ONCE)
    print("Published: '" + json.dumps(message) + "' to the topic: '" + TOPIC + "'", file=sys.stderr)

    time.sleep(measurment_interval_sec)

```

前回作成した自動起動スクリプトを以下のように書き換えます。

```bash:/etc/init.d/measure_room_condition
#! /bin/sh

### BEGIN INIT INFO
# Provides:             scd30d
# Required-Start:       $remote_fs $syslog $network $named
# Required-Stop:        $remote_fs $syslog
# Default-Start:        2 3 4 5
# Default-Stop:
# Short-Description:    Measure room condition
### END INIT INFO

(cd /home/pi && ./measure_room_condition.py)
```

AWS側で受信できているか、以下のように確認します。

1. AWSコンソール画面の左側で、`テスト` - `MQTT テストクライアント` を選択します。
2. `MQTT テストクライアント`画面の`トピックのフィルター`に`room_condition`を入力し`サブスクライブ`ボタンをクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/IoTCore_MQTTTestClient.png?raw=true)
3. 画面下部の`サブスクリプション`で、10秒おきに受信データが表示されます。

## IoT Coreでデータ受信時のAWS Lambda呼び出し

IoT Coreでデータ受信できたので、このデータを引数にAWS Lambdaを呼び出してみます。

ここでは、AWS Lambdaのコードは[Serverless Framework](https://www.serverless.com/)を使ってデプロイします。

### Serverless Frameworkのインストール

Serverless Frameworkのインストール方法はインターネット上に多数あるので、[こちら](https://serverless.co.jp/blog/25/)等を参考にインストールしてください。

### Serverless Frameworkのプロジェクト作成

以下のコマンドを実行して、プロジェクト用のディレクトリを作成します。

```bash
mkdir aws_lambda
cd aws_lambda/
```

以下のコマンドを使って、Serverless Frameworkのプロジェクトを作成します。

```bash
serverless create --template aws-python3
```

以下の2つのファイルが作成されます。

```bash
$ ls
handler.py  serverless.yml
```

### データ受信処理の実装

ファイルを編集して、AWS IoT Coreからのデータを受け取れるようにします。

```python:handler.py
import json

def hello(event, context):
    print(event)

    return {
        "message": "Go Serverless v1.0! Your function executed successfully!",
        "event": event
    }
```

```yaml:serverless.yml
service: aws-lambda

provider:
  name: aws
  runtime: python3.8

  region: us-west-2

functions:
  hello:
    handler: handler.hello
    events:
      - iot:
         sql: "SELECT * FROM 'room_condition'"
```

### デプロイ

以下のコマンドを実行してデプロイします。

```bash
$ serverless deploy

Deploying aws-lambda to stage dev (us-west-2)

✔ Service deployed to stack aws-lambda-dev (49s)

functions:
  hello: aws-lambda-dev-hello (270 B)
```

CloudWatchのログに、温度、CO2濃度、湿度が表示されます。

![](https://github.com/horie-t/tech-note/blob/master/images/IoTCore_LambdaRecieve.png?raw=true)


## Amazon Timestreamのデータベース、テーブルの作成

AWS Lambdaがデータを受け取れるようになったので、Lambdaのデータ送信先のTimestreamをセットアップします。

データベース、テーブルを以下の手順で作成します。

1. AWSコンソールの[Timestream](https://console.aws.amazon.com/timestream)の画面を開きます。
2. ナビゲーション・ペインで `Database` を選択します。
3. `Create database` をクリックします。
4. `create database` ページで、次のように操作します。
   1. `Choose a configuration` で、`Standard database` を選択します。
   2. `Name` でデータベース名を入力(例: RoomCondition)します。
   3. `Create database` をクリックします。
5. ナビゲーション・ペインで `Tables` を選択します。
6. `Create table` をクリックします。
7. `Create table` ページで、以下のように操作します。
   1. `Database name` で、作成したデータベースを選択します。
   2. `Table name` で、テーブル名を入力(例: conditions)します。
   3. `Data retention` で保存期間について、 `Memory store retention`、 `Magnetic store retention` を設定します。
   4. `create table` をクリックします。

## AWS LambdaからTimestreamに書き込み

Timestream側でテーブルの作成ができたので、Lambdaからデータを書き込みます。

### IAMロールの作成

Timestreamのデータベースに書き込むための権限が必要なので、IAMロールを作成します。

1. AWSコンソールの[IAM](https://us-east-1.console.aws.amazon.com/iamv2/home)画面を開きます。
2. 画面の左のペインで`ロール`を選択し、`ロール`画面で`ロールを作成`ボタンをクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/IAM_Role.png?raw=true)
3. `信頼されたエンティティを選択`画面で`信頼されたエンティティタイプ`は`AWS のサービス`を選択し、`ユースケース`で`Lambda`を選択し、`次へ`ボタンをクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/IAM_Entity.png?raw=true)
4. `許可を追加`画面の許可ポリシーの検索ボックスで`AmazonTimestreamFullAccess`を入力し検索結果のポリシーにチェックをし`次へ`ボタンをクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/IAM_AddPermission.png?raw=true)
5. `名前、確認、および作成`画面で`ロール名`に`lambda-timestream-role`を入力し`ロールを作成`ボタンをクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/IAM_RoleName.png?raw=true)

### コードの修正

Lambdaのファイルを修正して、Timestreamにデータを転送するようにします。

```python:handler.py
import boto3
import time

DatabaseName = 'RoomCondition'
TableName = 'conditions'

def current_milli_time():
    return round(time.time() * 1000)

client = boto3.client('timestream-write', region_name='us-west-2')

def write_record(event, context):
    print(event)

    temperature = event['temperature']
    co2 = event['co2']
    humidity = event['humidity']

    current_time = str(current_milli_time())

    dimensions = [
        {'Name': 'deviceId', 'Value': '1'},
    ]

    co2_record = {
        'Dimensions': dimensions,
        'MeasureName': 'co2',
        'MeasureValue': str(co2),
        'MeasureValueType': 'DOUBLE',
        'Time': current_time
    }

    temperature_record = {
        'Dimensions': dimensions,
        'MeasureName': 'temperature',
        'MeasureValue': str(temperature),
        'MeasureValueType': 'DOUBLE',
        'Time': current_time
    }

    humidity_record = {
        'Dimensions': dimensions,
        'MeasureName': 'humidity',
        'MeasureValue': str(humidity),
        'MeasureValueType': 'DOUBLE',
        'Time': current_time
    }

    records = [co2_record, temperature_record, humidity_record]

    result = client.write_records(DatabaseName=DatabaseName, TableName=TableName, Records=records, CommonAttributes={})

    return result
```

YAMLファイルも修正します。

```yaml:serverless.yml
service: aws-lambda
frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.8
  region: us-west-2
  iam:
    role: arn:aws:iam::XXXXXXXXXXXX:role/lambda-timestream-role

functions:
  room_condition:
    handler: handler.write_record
    events:
      - iot:
         sql: "SELECT * FROM 'room_condition'"

```

`XXXXXXXXXXXX`はAWSのアカウントIDを設定します。

### デプロイ

再度デプロイします。

```bash
serverless deploy
```

AWSコンソールのTimesteamの画面で、以下のようにテーブルにデータが書き込まれている事を確認します。

1. AWSコンソールの[Timestream](https://console.aws.amazon.com/timestream)の画面を開きます。
2. ナビゲーション・ペインで `管理ツール` - `クエリエディタ` を選択します。
3. `クエリエディタ`画面で`データベース`で`RoomCondition`を選択し、`Query 1`に以下のクエリを入力し、`実行`ボタンをクリックします。
   ```sql
   SELECT BIN(time, 1m) as binned_time,
       avg(measure_value::double) as avg_co2,
       max(measure_value::double) as max_co2,
       min(measure_value::double) as min_co2
   FROM "RoomCondition"."conditions"
   WHERE measure_name = 'co2'
   GROUP BY BIN(time, 1m)
   ORDER BY binned_time DESC
   ```
   ![](https://github.com/horie-t/tech-note/blob/master/images/Timestream_Query.png?raw=true)

## Grafana Cloudで可視化

Amazon Timestreamにデータを保存できたので、Grafana Cloudで測定データを可視化します。

### アカウントの作成

[Grafana](https://grafana.com/)のサイトで`free acount`でアカウントを作成します。

### Timestreamへの接続先の調査

以下のコマンドを実行して、Timestreamの接続エンドポイントを調べます。リージョンは、適宜変更します。

```bash
aws timestream-query describe-endpoints --region us-west-2 | jq -r '.Endpoints[].Address'
```

### Timestream用プラグインのインストール

Timestream用プラグインを、以下の手順にてインストールします。

1. ダッシュボード画面を表示します。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_Dashboard.png?raw=true)
2. 画面の左下の、`Configration` - `Plugins` を選択します。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_AddPlugin.png?raw=true)
3. `Configuration`画面の検索ボックスで`amazon timestream`を入力し、検索結果のAmazon Timestreamをクリックします。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_Plugin.png?raw=true)
4. `Install via grafana.com`をクリックします。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_TimestreamPlugin.png?raw=true)
5. 別タブで`Install plugin`をクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_InstallTimestreamPlugin.png?raw=true)

### Timestreamへの接続

Timestreamのプラグインが使えるようになったので、以下の手順にてTimestreamに接続する設定をします。

1. 元の画面に戻って、画面の左下の、`Configration` - `Data sources` を選択します。
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_AddDataSource.png?raw=true)
2. `Configuration`画面で`Add data source`をクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_DataSources.png?raw=true)
3. `Add data source`画面の検索ボックスで`amazon timestream`を入力し、検索結果のAmazon Timestreamをクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_TimestreamDataSource.png?raw=true)
4. `Data Sources`画面で必要事項を入力します。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_AddTimestream.png?raw=true)  
   **Authentication Provider** は、**Access & secret key** を選択し、以下のように設定し、**Save & Test**をクリックします。
   | 項目 |   値  |
   | ---- | :---: |
   |Authentication Provider | Access & secret key |
   | Access Key ID | IAMユーザーのアクセスキーID |
   | Secret Access Key | IAMユーザーのシークレットアクセスキー |
   | Assume Role ARN | 空欄のまま |
   | External ID | 空欄のまま |
   | Endpoint | AWS CLIで調べたqueryエンドポイント |
   | Default Region | us-west-2 |

### ダッシュボードの作成

Timesteamにアクセスできるようになったので、以下の手順にて、ダッシュボードを作成して測定結果を表示するパネルを追加します。

1. `Dashboards` - `+ New dashboard`をクリックします。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_NewDashboard.png?raw=true)
2. `Add a new panel`をクリックします。
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_AddPanel.png?raw=true)
3. `Edit Panel`画面の下部で`Database`は`RoomCondition`を選択し、`Table`は`conditions`を選択し、`Measure`は`co2`を選択します。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_QueryPanel.png?raw=true)  
   クエリ入力部に以下を入力し、右上の`Apply`をクリックします。  
   ```sql
   SELECT time, measure_name, measure_value::double as co2 FROM "RoomCondition"."conditions" WHERE measure_name = 'co2' ORDER BY time
   ```
4. 同様に,、温度、湿度もパネルに追加すると以下のようになります。  
   ![](https://github.com/horie-t/tech-note/blob/master/images/Grafana_RoomConditionPanel.png?raw=true)
