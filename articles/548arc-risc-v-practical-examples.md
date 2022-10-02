---
title: "RISC-Vの実用例等の現状について(2022年10月版)"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["riscv", "コンピュータアーキテクチャ"]
published: true
---

[RISC-V](https://ja.wikipedia.org/wiki/RISC-V)がコンピュータ業界である程度の注目([*1](https://www.jasa.or.jp/etwest/2019/conference2019/confpage-ts05.html), [*2](https://interface.cqpub.co.jp/wp-content/uploads/if1912_034.pdf))を集めるようになってから数年経過しています。

開発用のボードはアマチュアでもスイッチサイエンスで[多くの機種](https://www.switch-science.com/search?type=article%2Cpage%2Cproduct&q=RISC-V*)が入手できるようになり、海外では[ポータブルコンピュータも購入](https://www.clockworkpi.com/product-page/devterm-kit-r01)できます。また、保守的なイメージがある日本の半導体メーカーでも[RISC-Vベースのマイクロコントローラ](https://www.renesas.com/jp/ja/products/microcontrollers-microprocessors/risc-v)を発売しています。

教育関連ではコンピュータ・アーキテクチャの教科書の和書も[ヘネパタ](https://www.amazon.co.jp/dp/4434264001/)等([*4](https://www.amazon.co.jp/dp/4434305220/))がRISC-Vを題材として発行されています。[パタネネ](https://www.amazon.co.jp/dp/0128122757)も和訳が待ち望まれています。(一体、誰が翻訳をサボっているのだろう…)

研究でも[カーボンナノチューブ（CNT）トランジスタを使ったマイクロプロセッサの開発](https://eetimes.itmedia.co.jp/ee/articles/1909/13/news029.html)のような最先端の研究に利用されたりしている。

このように注目を集めているRISC-Vですが実用レベルではどうなの？って疑問になってきます。ここでは、RISC-Vの実用例をいくつか取り上げてみます。

## GPU内部のコントローラ

RICS-Vの初期からRISC-Vへのコミットを宣言していた半導体メーカーにNVIDIAがあります。NVIDIAはGPU内部のコントローラとしてFalconと呼ばれるMCUを自社開発していたのですが、ハードウェア側が複雑化するソフトウェアの要求に答えられない状態になり([ここの絵](https://riscv.org/wp-content/uploads/2017/05/Tue1345pm-NVIDIA-Sijstermans.pdf#page=4)が笑える)、RISC-Vを[2nd coreとして採用](https://riscv.org/wp-content/uploads/2016/07/Tue1100_Nvidia_RISCV_Story_V2.pdf#page=9)しました。純粋なFalconは[2017年にEOL](https://youtu.be/7Lx3692cbAg?t=27)になりました。

NVIDIAが[Linux用のGPUカーネル・モジュールをオープンソースとして公開](https://developer.nvidia.com/blog/nvidia-releases-open-source-gpu-kernel-modules/)しているので、そのソースコードを解析して、どこでRISC-Vが使用されているのかを調べた[ブログ](https://vengineer.hatenablog.com/entry/2022/05/16/090000)もあります。

このように、独自のCPUを開発して自社製品に使用していたメーカーが、独自CPUのエコシステムの開発コストに耐えきれず、RISC-Vに切り替える事例が増えそうです。

## 創薬専用スパコン

理化学研究所は創薬専用スパコンとして[「MDGRAPE-4A」を開発](https://www.riken.jp/press/2019/20191118_2/index.html)しました。RISC-Vは、内部のいくつかあるプロセッサの中で[General-Purpus Cores](https://www.bdr.riken.jp/ja/research/labs/taiji-m/mdgrape4.html)として使用されています。

MDGRAPEは[GRAPE](https://ja.wikipedia.org/wiki/GRAPE)と呼ばれる日本独自の重力多体問題専用計算機から派生した分子動力学用計算機の最新モデルです。GRAPEの初期開発の様子は[『スーパーコンピューターを20万円で創る』](https://www.amazon.co.jp/dp/4087203956)に描かれています。

RISC-VはArmとは異なりオープンソースなので、独自の実装や拡張が許されているため、専用のシステムを作るのにも向いています。

## ダイソーの1,100円「完全ワイヤレスイヤホン」

ダイソーで販売されているワイヤレスイヤホンの[分解記事](https://el.jibun.atmarkit.co.jp/thousandiy/2022/02/3_bluetooth.html)ではメインプロセッサとしてRISC-VのSoCを見つけています。

RISC-Vはライセンス料が無料なので、低価格帯製品に使用するのに適しています。このようにまずローエンド市場でシェアを確保して、コストパフォーマンスを維持しながら高性能化していき、段々とハイエンド製品の市場を侵食していくのは、破壊的イノベーションのセオリー通りの動きです。

Armも初期は性能が低い組み込み製品だけでの利用でしたが、スマートフォン向けの製品で性能を上げ続け、ついにArmv8.2-A SVEに準拠したスーパーコンピュータの[富岳](https://www.fujitsu.com/jp/about/businesspolicy/tech/fugaku/)が開発され、PC用途でも[Apple M1](https://ja.wikipedia.org/wiki/Apple_M1)として使用されるようになりました。

## 将来的な動き

上記の他にも、Western Digitalが[製品での使用する予定](https://www.westerndigital.com/ja-jp/company/innovation/open-source/risc-v)だと表明しています。SEAGATEも対抗するように[RISC-Vを組み込んだ半導体チップを設計](https://www.seagate.com/jp/ja/innovation/risc-v/)しています。
