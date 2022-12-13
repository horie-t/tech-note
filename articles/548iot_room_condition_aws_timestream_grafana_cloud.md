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