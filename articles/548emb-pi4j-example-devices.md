---
title: "「pi4j-example-devices」を使ってPi4Jのサンプルコードを実行する"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["java", "raspberrypi", "pi4j"]
published: true
---

[Pi4J](https://pi4j.com/)は、Raspberry PiのGPIOピンをJavaで制御するためのライブラリです。Pi4Jの公式リポジトリには、様々なサンプルコードが含まれていますが、その中でも「[pi4j-example-devices](https://github.com/Pi4J/pi4j-example-devices)」は、LEDやセンサーなどのデバイスを制御するためのサンプルコードが含まれており、初心者にとって非常に役立ちます。今回は、この「pi4j-example-devices」を使って、Pi4Jのサンプルコードを実行する方法を紹介します。

## 1. ハードウェアの接続

「pi4j-example-devices」リポジトリには、MCP3008というアナログ-デジタルコンバータを使用して、可変抵抗器の値を読み取るサンプルコードが含まれています。以下のように、Raspberry PiとMCP3008と可変抵抗器を接続してください。

![MCP3008の接続図](/images/Pi5_MCP3008_Potentiometer.png)

## 2. リポジトリのクローン
まずは、GitHubから「pi4j-example-devices」リポジトリをクローンします。ターミナルで以下のコマンドを実行してください。

```bash
git clone https://github.com/Pi4J/pi4j-example-devices.git
```

## 3. プロジェクトのビルド
クローンしたリポジトリのディレクトリに移動し、Mavenを使ってプロジェクトをビルドします。以下のコマンドを実行してください。

```bash
cd pi4j-example-devices
mvn clean package
```

## 4. サンプルコードの実行
ビルドが成功したら、`target`ディレクトリに生成されたJarファイルをRaspberry Pi上で実行します。以下のコマンドを実行してください。実行しながら、可変抵抗器のつまみを操作して、MCP3008の値が変化する様子を確認できます。

```bash
cd target/distribution
./runMcp3008.sh
```

実行結果

```bash
$ ./runMcp3008.sh
[main] INFO com.pi4j.Pi4J - New auto context
[main] INFO com.pi4j.Pi4J - New context builder
[main] INFO com.pi4j.Pi4J - Pi4J library build info:
[main] INFO com.pi4j.Pi4J -     Branch: develop
[main] INFO com.pi4j.Pi4J -     Commit ID: 126fb93
[main] INFO com.pi4j.Pi4J -     Version: 4.1.0-SNAPSHOT
[main] INFO com.pi4j.Pi4J -     Timestamp: 2026-03-17T07:15:08Z
[main] INFO com.pi4j.boardinfo.util.BoardInfoHelper - Detected OS: Name: Linux, version: 6.6.74+rpt-rpi-2712, architecture: aarch64
[main] INFO com.pi4j.boardinfo.util.BoardInfoHelper - Detected Java: Version: 25.0.2, runtime: 25.0.2+10-LTS, vendor: Eclipse Adoptium, vendor version: Temurin-25.0.2+10
[main] INFO com.pi4j.boardinfo.util.BoardInfoHelper - Detected board type MODEL_5_B by code: d04170
# (中略)初期メッセージ
[main] INFO com.pi4j.util.Console - ************************************************************
[main] INFO com.pi4j.util.Console - ************************************************************
[main] INFO com.pi4j.util.Console -
[main] INFO com.pi4j.util.Console -            <-- The Pi4J V2 Project Extension  -->
[main] INFO com.pi4j.util.Console -                          MCP3008App
[main] INFO com.pi4j.util.Console -
[main] INFO com.pi4j.util.Console - ************************************************************
[main] INFO com.pi4j.util.Console - ************************************************************
[main] INFO com.pi4j.util.Console -
[main] INFO com.pi4j.util.Console - ----------------------------------------------------------
[main] INFO com.pi4j.util.Console - PI4J PROVIDERS
[main] INFO com.pi4j.util.Console - ----------------------------------------------------------
PROVIDERS: [5] "I/O Providers" <com.pi4j.provider.impl.DefaultProviders>
├─DIGITAL_INPUT: [1] <com.pi4j.io.gpio.digital.DigitalInputProvider>
│ └─PROVIDER: "GpioD Digital Input (GPIO) Provider" {gpiod-digital-input} <com.pi4j.plugin.gpiod.provider.gpio.digital.GpioDDigitalInputProviderImpl> {com.pi4j.plugin.gpiod.provider.gpio.digital.GpioDDigitalInputProviderImpl}
├─SPI: [1] <com.pi4j.io.spi.SpiProvider>
│ └─PROVIDER: "LinuxFS SPI Provider" {linuxfs-spi} <com.pi4j.plugin.linuxfs.provider.spi.LinuxFsSpiProviderImpl> {com.pi4j.plugin.linuxfs.provider.spi.LinuxFsSpiProviderImpl}
├─I2C: [1] <com.pi4j.io.i2c.I2CProvider>
│ └─PROVIDER: "LinuxFS I2C Provider" {linuxfs-i2c} <com.pi4j.plugin.linuxfs.provider.i2c.LinuxFsI2CProviderImpl> {com.pi4j.plugin.linuxfs.provider.i2c.LinuxFsI2CProviderImpl}
├─ANALOG_OUTPUT: [0] <com.pi4j.io.gpio.analog.AnalogOutputProvider>
├─ANALOG_INPUT: [0] <com.pi4j.io.gpio.analog.AnalogInputProvider>
├─DIGITAL_OUTPUT: [1] <com.pi4j.io.gpio.digital.DigitalOutputProvider>
│ └─PROVIDER: "GpioD Digital Output (GPIO) Provider" {gpiod-digital-output} <com.pi4j.plugin.gpiod.provider.gpio.digital.GpioDDigitalOutputProviderImpl> {com.pi4j.plugin.gpiod.provider.gpio.digital.GpioDDigitalOutputProviderImpl}
└─PWM: [1] <com.pi4j.io.pwm.PwmProvider>
  └─PROVIDER: "LinuxFS PWM Provider" {linuxfs-pwm} <com.pi4j.plugin.linuxfs.provider.pwm.LinuxFsPwmProviderImpl> {com.pi4j.plugin.linuxfs.provider.pwm.LinuxFsPwmProviderImpl}
----------------------------------------------------------
[main] TRACE com.pi4j.plugin.linuxfs.provider.spi.LinuxFsSpi - [INITIALIZE] -> SPI_BUFFSIZ=4096
[main] TRACE com.pi4j.plugin.linuxfs.provider.spi.LinuxFsSpi - invoked 'open()'
[main] INFO com.pi4j.plugin.linuxfs.provider.spi.LinuxFsSpi - Opening SPI bus 0, address 0
[main] INFO com.pi4j.util.Console -

[main] INFO com.pi4j.util.Console - ************************************************************
[main] INFO com.pi4j.util.Console - ************************************************************
[main] INFO com.pi4j.util.Console -
[main] INFO com.pi4j.util.Console -                   <-- The Pi4J Project -->
[main] INFO com.pi4j.util.Console -           SPI test program using MCP3008 AtoD Chip
[main] INFO com.pi4j.util.Console -
[main] INFO com.pi4j.util.Console - ************************************************************
[main] INFO com.pi4j.util.Console - ************************************************************
[main] INFO com.pi4j.util.Console -
[main] INFO com.pi4j.util.Console - ------------------------------
[main] INFO com.pi4j.util.Console - |    PRESS CTRL-C TO EXIT    |
[main] INFO com.pi4j.util.Console - ------------------------------
[main] INFO com.pi4j.util.Console -
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 0   Bytes read : 0  Value : 940   # *1
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 1   Bytes read : 0  Value : 13
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 2   Bytes read : 0  Value : 2
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 3   Bytes read : 0  Value : 0
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 4   Bytes read : 0  Value : 0
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 5   Bytes read : 0  Value : 10
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 6   Bytes read : 0  Value : 26
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 7   Bytes read : 0  Value : 52
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 0   Bytes read : 0  Value : 251   # *2
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 1   Bytes read : 0  Value : 2
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 2   Bytes read : 0  Value : 21
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 3   Bytes read : 0  Value : 20
[main] INFO com.pi4j.devices.mcp3008.MCP3008 - Channel : 4   Bytes read : 0  Value : 19
^C[Thread-1] INFO com.pi4j.util.Console -
[Thread-1] INFO com.pi4j.util.Console - ************************************************************
[Thread-1] INFO com.pi4j.util.Console -                            GOODBYE
[Thread-1] INFO com.pi4j.util.Console - ************************************************************
[Thread-1] INFO com.pi4j.util.Console -
$ 
```

*1: MCP3008の0番チャンネルから読み取った値が940であることを示しています。
*2: 可変抵抗器のつまみを操作した結果、MCP3008の0番チャンネルから読み取った値が251であることを示しています。