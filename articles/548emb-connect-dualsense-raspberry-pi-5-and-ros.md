---
title: "Raspberry Pi 5にPlayStation 5のDualSenseコントローラを接続してROSのTurtleを動かす"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi", "ros", "ros2"]
published: true
---
[以前の記事](https://zenn.dev/thorie/articles/548emb-install-ros20-on-raspberry-pi-os)でRaspberry Pi 5にROS2(Jazzy Jalisco)をインストールしました。今回はこの環境にPlayStation 5のDualSenseコントローラを接続して、ROSのTurtleシミュレータの亀を動かします。

## DualSenseコントローラの入力をROSのメッセージに変換する

### ROSのjoyパッケージのインストール・セットアップ

以下のコマンドを実行して開発用パッケージをインストールします

```bash
sudo apt-get install libspnav-dev libbluetooth-dev libcwiid-dev
```

次に以下のようにROSのパッケージのソースコードをチェックアウトして、ビルドします。

```bash
cd <ROSのワークスペース>/src    # 例 cd ~/ros_ws/src
git clone https://github.com/ros/diagnostics.git -b ros2               # ROS2のブランチを指定する
git clone https://github.com/ros-drivers/joystick_drivers.git -b ros2  # ROS2のブランチを指定する

cd ..
colcon build
source ./install/setup.bash
```

### DualSenseコントローラの接続

以下の手順でRaspberry Piとコントローラをペアリングします。

1. コントローラプレイヤーランプ(長いLEDバー)を消灯させます。  
   点灯している場合はPSボタンを長押し(10秒くらい)します。
2. クリエイトボタン(十字ボタンの右上のボタン)を押しながらPSボタンを押し続けて、青いライトバーが点灯してペアリングモードになるのを待ちます。
3. Raspberry Pi側でBluetoothのデバイスを接続します。  
   タスクバーの右側の「Bluetoothアイコン」- 「Add Device」をクリックして、「Add New Device」ウィンドウでDualSenseコントローラを選択します。

### joyノードを起動してコントローラの入力を確認する

以下のコマンドでjoyノードを起動します。

```bash
ros2 run joy joy_node
```

以下のコマンドでコントローラからの入力値を確認します。

```
$ ros2 topic echo /joy
header:
  stamp:
    sec: 1719271678
    nanosec: 787959569
  frame_id: joy
axes:
- -0.0
- -0.0
- 0.9999999403953552
- -0.0
- -0.0
- 0.9999999403953552
- 0.0
- 0.0
buttons:
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
```

コントローラの操作とメッセージの対応は以下の通りです。

**axis**

| Index | コントローラ | 値の範囲 |
| ----- | ------------ | -------- |
| 0     | 左側スティック（左右方向） | 左:1, 右:-1 |
| 1     | 左側スティック（上下方向） | 上:1, 下:-1 |
| 2     | L2                         | 押していない: 1、押し込む: -1 |
| 3     | 右側スティック（左右方向） | 左:1, 右:-1 |
| 4     | 右側スティック（上下方向） | 上:1, 下:-1 |
| 5     | R2                         | 押していない:1, 押し込む: -1 |
| 6     | 十字方向ボタン（左右方向） | 左:1, 右:-1 |
| 7     | 十字方向ボタン（上下方向） | 上:1, 下:-1 |

**buttons**

| Index | コントローラ |
| ----- | ------------ |
| 0     | ×ボタン |
| 1     | 〇ボタン |
| 2     | △ボタン |
| 3     | □ボタン |
| 4     | L1ボタン |
| 5     | R1ボタン |
| 6     | L2ボタン |
| 7     | R2ボタン |
| 8     | クリエイトボタン |
| 9     | オプションボタン |
| 10    | PSボタン         |
| 11    | L3ボタン         |
| 12    | R3ボタン         |

以上でコントローラの入力をメッセージとしてトピックに発行する事が確認できました。

## joyのトピックをTwist型のトピックに変換する

次にjoyノードのメッセージをTurtleシミュレータで受け取るTwist型のメッセージに変換します。

### teleope_twist_joyパッケージのインストール・セットアップ

次に以下のようにROSのパッケージのソースコードをチェックアウトして、ビルドします。

```bash
cd <ROSのワークスペース>/src    # 例 cd ~/ros_ws/src
git clone https://github.com/ros2/teleop_twist_joy.git

cd ..
colcon build
source ./install/setup.bash
```

### teleop_twist_joyノードを起動する

デフォルトでは、joyパッケージはPS3やXBoxに対応していますが、PS5のDualSenseコントローラには対応しています。以下のコントローラのマッピングファイルを作成してください。

```yml:ps5.conf.yaml
teleop_twist_joy_node:
  ros__parameters:
    axis_linear:
      x: 1
      y: 0
    scale_linear:
      x: 0.7
      y: 0.7
    scale_linear_turbo:
      x: 1.5
      y: 1.5

    axis_angular:
      yaw: 3
    scale_angular:
      yaw: 0.4

    enable_button: 4  # L1 shoulder button
    enable_turbo_button: 5  # R1 shoulder button
```

以下のコマンドを実行してノードを起動します。

```bash
ros2 launch teleop_twist_joy teleop-launch.py config_filepath:=<yamlファイルへのパス>/ps5.conf.yaml
```

以下のように変換されたトピックを確認します。コントローラのスティックを動かしながらL1もしくはR1ボタンを押します。押している間だけメッセージが発行されます。

```bash
$ ros2 topic echo /cmd_vel
linear:
  x: 0.7
  y: -0.05850413963198661
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.4
```

## Turtleシミュレータで亀を動かす

以下のコマンドでTurtleシミュレータを起動して亀を動かします。(teleop_twist_joyノードから発行されるメッセージを購読するようにトピックを再マッピングしています。)

```bash
ros2 run turtlesim turtlesim_node --ros-args --remap /turtle1/cmd_vel:=/cmd_vel
```

![TurtleSim](https://github.com/horie-t/tech-note/blob/master/images/turtle-sim.png?raw=true)
