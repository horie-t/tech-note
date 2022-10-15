---
title: "Wio Terminalで測定した、CO2データ等をTimescaleDBにアップロードしGrafanaで可視化する"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["postgesql", "grafana", "aws", "iot", "arduino", "wioterminal"]
published: true
---

[TimescaleDB](https://www.timescale.com/)とは時系列データ(例えば、IoTデバイスの測定データ等)をPostgreSQL上で扱いやすくするためのPostgreSQL拡張です。

本記事では、室内の温度、湿度、CO<sub>2</sub>濃度の測定データをAWS上で稼働するTimescaleDBに保存し、[Grafana](https://grafana.com/)で表示してみます。Grafanaとは分析や視覚化ためのオープンソースのWebアプリケーションです。


## AWS EC2インスタンス上でTimescaleDBを稼働させる

### EC2インスタンスの起動
TimescaleDBをインストールしたAMIイメージ(`TimescaleDB 2.6.0 (PostgreSQL 13) - Ubuntu 20.04 (EBS Backed)`)があるので、それを使ってセットアップします。

1. AWSの[EC2ダッシュボード](https://console.aws.amazon.com/ec2/)を開きます。
2. `イメージ` - `AMI` を選択します。
3. `パブリックイメージ` を選択し、 `TimescaleDB 2.6.0`を検索します。
4. 検索結果のAMIをチェックし、`AMIからインスタンスを起動`をクリックします。
5. `インスタンスを起動`画面で、通常のEC2インスタンスを起動するように値を設定し(インスタンスタイプはt2.microぐらいでも動きます)、`インスタンスを起動`をクリックします。

### TimescaleDBのコンフィギュレーション

起動後は、EC2インスタンスにSSHでログインして、以下のコマンドを実行してコンフィギュレーションを開始します。

```bash
sudo timescaledb-tune
# 問い合わせは、全部y
```

設定変更後に、サービスを再起動します。

```bash
sudo systemctl restart postgresql.service
```

PostgreSQLに接続します。

```bash
sudo -u postgres psql
```

`tsdb`というDatabaeを作成して、TimescaleDB拡張を追加します。

```sql
CREATE database tsdb;
\c tsdb
CREATE EXTENSION IF NOT EXISTS timescaledb;
```

追加結果の確認します。

```sql
tsdb=# \dx
                                      List of installed extensions
    Name     | Version |   Schema   |                            Description                            
-------------+---------+------------+-------------------------------------------------------------------
 plpgsql     | 1.0     | pg_catalog | PL/pgSQL procedural language
 timescaledb | 2.6.0   | public     | Enables scalable inserts and complex queries for time-series data
(2 rows)
```

Databaseアクセス用のユーザ(例: iot)を作成します。

```sql
ALTER USER iot WITH PASSWORD 'パスワード';
```

iotに権限を付与します。

```sql
GRANT ALL PRIVILEGES ON DATABASE tsdb TO iot;
GRANT ALL PRIVILEGES ON conditions TO iot;
```

## TimescaleDBへのINSERTとクエリ

TimescaleDBにテーブルを作成して、データをINSERTして、検索してみます。

まず、テーブルを作成します。

```sql
CREATE TABLE conditions (
  time        TIMESTAMPTZ       NOT NULL,
  device_id   INTEGER           NOT NULL,
  co2 	      DOUBLE PRECISION  NULL,
  temperature DOUBLE PRECISION  NULL,
  humidity    DOUBLE PRECISION  NULL
);
```

このテーブルを通常のテーブルからTimescaleDBのハイパーテーブルに変更します。

```sql
SELECT create_hypertable('conditions', 'time');
```

サンプルデータを挿入します。

```sql
INSERT INTO conditions VALUES('2022-04-04 12:00:00.000+00', 1, 500, 27, 50),
       ('2022-04-04 12:01:00.000+00', 1, 700, 28, 51),
       ('2022-04-04 12:02:00.000+00', 1, 700, 29, 52),
       ('2022-04-04 12:03:00.000+00', 1, 700, 30, 53),
       ('2022-04-04 12:04:00.000+00', 1, 800, 28, 54),
       ('2022-04-04 12:05:00.000+00', 1, 750, 27, 55),
       ('2022-04-04 12:06:00.000+00', 1, 700, 26, 53),
       ('2022-04-04 12:07:00.000+00', 1, 650, 25, 52),
       ('2022-04-04 12:08:00.000+00', 1, 600, 24, 51),
       ('2022-04-04 12:09:00.000+00', 1, 500, 23, 50),
       ('2022-04-04 12:10:00.000+00', 1, 550, 25, 49),
       ('2022-04-04 12:11:00.000+00', 1, 600, 26, 50),
       ('2022-04-04 12:12:00.000+00', 1, 650, 27, 53),
       ('2022-04-04 12:13:00.000+00', 1, 700, 29, 55);
```

以下のように5分ごとの平均を求めてみます。

```sql
tsdb=# SELECT time_bucket('5 minutes', time) AS five_min,
tsdb-#        avg(co2), avg(temperature), avg(humidity)
tsdb-# FROM conditions
tsdb-# GROUP BY five_min ORDER BY five_min;
        five_min        | avg |  avg  |  avg  
------------------------+-----+-------+-------
 2022-04-04 12:00:00+00 | 680 |  28.4 |    52
 2022-04-04 12:05:00+00 | 640 |    25 |  52.2
 2022-04-04 12:10:00+00 | 625 | 26.75 | 51.75
(3 rows)
```

以上のように平均を求められました。

## Grafanaを使って可視化

DBのセットアップができたので、Grafanaを使って可視化してみます。

まずPostgreSQL側でDBにアクセスするためのuserを作成しておく。

```sql
CREATE USER ユーザ名 WITH PASSWORD 'パスワード';
GRANT ALL PRIVILEGES ON DATABASE tsdb TO ユーザ名;
```

### Grafanaのインストール

以下のようにパッケージをインストールします。

```bash
sudo apt-get update
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_8.4.5_amd64.deb
sudo dpkg -i grafana_8.4.5_amd64.deb
```

サービスを起動します。

```bash
sudo systemctl start grafana-server
```

### Grafanaのセットアップ

手元のPCからSSHポートフォワードでトンネリングしてGrafanaサーバへlocalhostとしてアクセスできるようにします。

```bash
ssh -i .ssh/xxx-key-pair.pem -L 3000:localhost:3000 ubuntu@xxx.xxx.xxx.xxx
```

以下のようにブラウザを操作します。

1. ブラウザで`http://localhost:3000/`にアクセスします。
2. デフォルトのusername、passwordは `admin` ですので、この値でログインします。
3. ログイン後にパスワードの変更を求められますので、変更します。
4. `Configuration` の `Datasource`をクリックします。  
![](https://storage.googleapis.com/zenn-user-upload/b542c4fe5e81-20220423.png)
5. PostgreSQLを選択します。  
![](https://storage.googleapis.com/zenn-user-upload/9e3976a31a1b-20220423.png)
6. `Host`、`Database`、`User`、`Password`を入力し、`TLS/SSL Mode`は`disable`に、`PostgreSQL detail`の`Version`はTimescaleDBのベースになっているVersionを指定し、`Save & Test`をクリックします。  
![](https://storage.googleapis.com/zenn-user-upload/67b04b3407f6-20220423.png)
7. `Create` の `Dashboard` を選択します。  
![](https://storage.googleapis.com/zenn-user-upload/da192fa79d57-20220423.png)
8. `Add a new panel`を選択します。  
![](https://storage.googleapis.com/zenn-user-upload/1e2b6706edc4-20220423.png)
9. 下部の `Data source` で `PostgreSQL` を選択し `Column` を選択して、右上の `Apply` をクリックします。  
![](https://storage.googleapis.com/zenn-user-upload/0e6bc420eb71-20220423.png)
10. 上部の `Save` のアイコンをクリックして、ダイアログで `Dashboard name` を入力して、`Save`をクリックします。  
![](https://storage.googleapis.com/zenn-user-upload/f593dd377a1d-20220423.png)

以上で、Grafanaを使ってTimescaleDBのデータを分析、可視化できるようになりました。

## HTTP経由でTimescaleDBにINSERT

IoTデバイスからHTTP経由でTimescaleDBにデータをINSERTしたいので、HTTPサーバを起動させます。ここではHTTPサーバは、node.jsでサーバを立てる事にします。

node.jsをインストールし、適当なディレクトリで以下のコマンドを実行します。

```bash
npm init -y
```

`package.json` を以下のように編集します。

```javascript
{
  "name": "dbproxy",
  "version": "1.0.0",
  "description": "",
  "main": "src/main.js",
  "scripts": {
    "start": "sudo node ./src/main.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "HORIE Tetsuya",
  "license": "ISC",
  "dependencies": {
    "pg": "^8.7.3"
  }
}
```

`src/main.js` を以下の通り作成します。

```javascript
const { Client } = require("pg");

var http = require('http');
var server = http.createServer(async (req, res) => {
    const buffers = [];
    for await (const chunk of req) {
	buffers.push(chunk);
    }
    const data = JSON.parse(Buffer.concat(buffers).toString());
    await console.log(data);

    const client = new Client({
	user: "ユーザ名",
	host: "127.0.0.1",
	database: "tsdb",
	password: "パスワード",   
	port: 5432,
    });
    const sql = "INSERT INTO conditions (time, device_id, co2, temperature, humidity) VALUES($1, $2, $3, $4, $5)"
    client.connect();
    client.query(sql,
		 [new Date().toISOString(), 1, data.co2, data.temperature, data.humidity],
		 (err, result) => {
		     if (err) {
			 console.log(err);
		     }
		     client.end();
		     res.end();
		 });
}).listen(80);
```

以下のコマンドを実行し、サーバを起動します。

```bash
nohup npm start &
```

以上で、HTTPサーバを起動できました。

## IoTデバイスからTimescaleDBに測定値を記録

IoT端末として、[Wio Terminal](https://www.switch-science.com/catalog/6360/) を、センサに[SDC30](https://www.switch-science.com/catalog/7000/)を使って、室内環境をTimescaleDBに保存するようにしてみます。

Arduinoで以下のコードを作成して、Wio Terminalに書き込みします。

```cpp
#include <SCD30.h>
#include <rpcWiFi.h>
#include <HTTPClient.h>
#include <TFT_eSPI.h>

const char* ssid = "SSIDを設定";
const char* password =  "WiFIのパスワード";
const char* postUrl =  "サーバのURL";

TFT_eSPI tft;

void setup() {
  // SDC30の初期化
  Wire.begin();
  scd30.initialize();

  // シリアル出力の設定
  Serial.begin(115200);

  /*
   * WiFiのセットアップ
   */
  WiFi.begin(ssid, password);
 
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.println("Connecting..");
  }
  Serial.print("Connected to the WiFi network with IP: ");
  Serial.println(WiFi.localIP());
  
  /*
   * LCDディスプレイの固定表示部分のセットアップ
   */
  tft.begin();
  tft.setRotation(3);
 
  tft.fillScreen(TFT_DARKCYAN);

  tft.setTextColor(TFT_WHITE);
  tft.setTextSize(2);
  tft.drawString("CO2:",      40,  90);
  tft.drawString("Temp.:",    40, 115);
  tft.drawString("Humidity:", 40, 140);
  tft.drawRoundRect(15, 55, 320 - (15 * 2), 240 - (55 * 2), 15, TFT_WHITE);
}
 
void loop() {
  float result[3] = {0};

  bool scd30available = scd30.isAvailable();
  if (scd30available) {
    // SDC30の値を読み込み、LCDの表示
    scd30.getCarbonDioxideConcentration(result);
    tft.fillRect(160, 90, 140, 80, TFT_DARKCYAN);
    tft.drawString(String(result[0], 0) + " ppm",   160,  90);
    tft.drawString(String(result[1], 1) + " deg C", 160, 115);
    tft.drawString(String(result[2], 1) + " %",     160, 140);
  }

  if(scd30available) {
    /*
     * サーバへ値を送信
     */
    HTTPClient http;
 
    http.begin(postUrl);
    http.addHeader("Content-type", "application/json");

    String body = String("{\"co2\": " + String(result[0]) + ", \"temperature\":  " + String(result[1], 1) +", \"humidity\": " + String(result[2], 1) + "}");
 
    int httpResponseCode = http.POST(body);
    if (httpResponseCode > 0){
      Serial.print("HTTP Response Code: ");
    } else {
      Serial.print("Error on sending request: ");
    }
    Serial.println(httpResponseCode);
 
    http.end();  //Free resources
  } else {
    Serial.println("Unable to create client");
  }
  
  Serial.println();
  Serial.println("Waiting 1s before the next round...");
  delay(5000);
}
```

以上で、室内環境のデータがTimescaleDBに保存されるようになりました。

## データ集計する

IoT端末からのデータを集計してみます。

### SQLで集計

以下のように1日の平均値、最大値、最小値を求める事ができます。

```sql
SELECT time_bucket('1 day', time) AS a_day,
     avg(co2), max(co2), min(co2)
FROM conditions
GROUP BY a_day
ORDER BY a_day;
```

実行結果は以下の通りです。

```
         a_day          |        avg         |   max   |  min   
------------------------+--------------------+---------+--------
 2022-04-04 00:00:00+00 |                650 |     800 |    500
 2022-04-19 00:00:00+00 | 1256.9683669574706 | 2636.34 |    500
 2022-04-20 00:00:00+00 | 1211.2500919181427 | 2163.81 | 906.27
 2022-04-21 00:00:00+00 | 1123.5553359946773 | 1824.32 | 706.44
 2022-04-22 00:00:00+00 | 1236.5902098489098 | 2923.44 | 580.34
 2022-04-23 00:00:00+00 |   627.415416666667 |  951.75 | 561.21
(6 rows)
```


実はPostgreSQL自体にもこのような関数はあります。

```sql
SELECT date_trunc('day', time) AS a_day,
     avg(co2), max(co2), min(co2)
FROM conditions
GROUP BY a_day
ORDER BY a_day;
```

### Grafanaで可視化

Grafanaで可視化してみます。Grafanaで以下のようなクエリを設定します。

```sql
SELECT
  $__timeGroup("time",'1d'),
  avg(co2) AS "avg", max(co2) AS "max", min(co2) AS "min"
FROM conditions
WHERE
  $__timeFilter("time")
GROUP BY 1
ORDER BY 1
```

![](https://storage.googleapis.com/zenn-user-upload/bcbe7ad58939-20220428.png)

