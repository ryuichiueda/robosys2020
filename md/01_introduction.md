# ロボットシステム学第1回

上田 隆一

千葉工業大学


---

## 今日の内容

* 動機付け（なんでこの講義があるのか）

---

## ロボットシステム

* システム
    * 目的を遂行するための体系や組織<br />$ $
* ロボットシステム
    * 目的を遂行するために動く
    * 高度に自動化されている
    * 例: ルンバ、新幹線、自動ドア

なんでもロボット（視野を広く持ちましょう）

---

## ロボットシステム開発の変化

* ハードやソフトの多くが部品化
    * 作るより「つなぐ』機会が相対的に増える<br />$ $
* インターネット上に知識が集積
    * 大学の先生から何を学ぶの？<br />$ $
* 使える道具が増加$\rightarrow$開発の急速化
    * CAD、3Dプリンタ、ROS、・・・


重要なところは自作+それ以外はありあわせ


---

## 外のリソースを使う

* 例1: 小型ロボットのナビゲーション
    * 自己位置推定、SLAM、ナビゲーションに関してはプログラムを書いていない

<iframe width="560" height="315" src="https://www.youtube.com/embed/7xXnXHc0roA" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

---

* 例2: RoboCup@Home
    * チーム結成8ヶ月でハード、ソフトを組み上げ世界大会で善戦

<iframe width="560" height="315" src="https://www.youtube.com/embed/XozlE7fuTso" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

---

* 例3: 設計製作論3
    * https://twitter.com/robo_cit/status/1201399538541400064?ref_src=twsrc%5Etfw
        * サンプルコードをうまく使ってロボットに仕事をさせる

---

## 「ありあわせ」のありあわせかたをどうやって身につけるか？


* <span style="color:red">プラットフォーム</span>の存在
    * 知識やソフト等を流通させるには、それらが適用できる共通のモノが必要
    * ロボットは計算機だけの場合より多様だが存在<br />$ $
* プラットフォームの例
    * ハード: PC、Raspberry Pi、Arduinoとその互換品、各種規格品
    * ソフト: UNIX、Linux
    * サービス: GitHub
    * ロボット: TurtleBot、iCart-mini、HSR、・・・

プラットフォームを深く理解し制することが重要

---

## <span style="text-transform:none">Raspberry Pi</span>

* シングルボードコンピュータ<br />$ $
* 本講義で扱う
    * 特に講師が信者ということではない
    * 性能というよりもプラットフォームとして非常に優秀
        * 使ったことがある腕自慢の人でも、ちゃんとプラットフォームとしての視点で使っていただろうか？

---

## <span style="text-transform:none">Raspberry Pi</span>について詳しく

* イギリスで開発された教育用のシングルボードコンピュータ<br />$ $
* 開発の動機・歴史
    * 2006-2008年: 構想・試作
        * ケンブリッジ大の学生のコンピュータの腕が落ちてきた
            * 経験者と言ってもWord, Excel, HTMLくらい
            * （未ロボだと逆の人も多い）
        * 安価でハードの制御ができるようなシングルボードPCが作れないか？
    * 2009年: Raspberry Pi Foundation設立
        * Raspberry Piを販売（2016年に1000万台販売）


---

## 講義の目的

* 一つのシングルボードコンピュータを深く理解
    * 低レイヤーから高レイヤー、さらにその外側まで一通り経験
        * 低レイヤー: デバイスドライバ〜OS
        * 高レイヤー: OS〜インターネットとのやりとり
        * さらには著作権やライセンス、開発方法まで

---

## 講義の内容

全13回

* 第1週: イントロダクション（今回）
* 第2週: コンピュータボードの通信と操作
    * ラズパイのセットアップと操作
* 第3週: PC・オペレーティングシステム（OS）
* 第4週: OSのプロセス
* 第5週: ファイルシステム

---

* 第6週: デバイスドライバの仕組み
* 第7週: デバイスドライバの実装1
* 第8週: デバイスドライバの実装2
* 第9週: ソフトウェアライセンスとクリエイティブコモンズ
* 第10週: GitとGitHub
* 第11週: ROS1
* 第12週: ROS2
* 第13週: まとめ

---

## 講義で使う道具

* Raspberry Pi
* Raspberry Piのアクセサリ（次のページ）<br />$ $
* ノートPC
    * 役割1: Raspberry Piとssh通信
    * 役割2: Raspberry Piと繋がらないときの保険
        * 仮想マシン上 or ネイティブなUbuntu
        * Ubuntu 18.04 LTSあるいはUbuntu 20.04 LTSを想定

---

## ラズパイのアクセサリ

* USBケーブル
    * ノートPCからRaspberry Piに電源供給
    * Pi 3B+まで: Type A-Micro B、あるいはType C-Micro B
    * Pi 4: Type A-Type C、あるいはType C-Type C<br />$ $
* microSDカード
    * 16GBのものがおすすめ
    * 複数あるとよい<br />$ $
* LANケーブル
    * ノートPCとRaspberry Piを接続


---

## ウェブサイト等

* [講義のサイト](https://lab.ueda.tech/?page=robosys_2020)<br />$ $
* 連絡: 他人に見られてもいい内容はTwitterで[@ryuichiueda](https://twitter.com/ryuichiueda)
    * 質問も重要なコントリビューション<br />$ $
* 課題提出
    * コードをGitHubに
    * デモビデオをYouTubeかどこかに

---

## テスト・レポート等

* 課題は2回
    * 配点: 各20点（＋α）
* テストは1回
    * 配点: 60点
* 出席
    * 遅刻、早退は0.5回とカウント

* この講義資料にプルリクくれて私がマージしたら加点します。


---

## 参考図書

* 初心者の人はこれを購入のこと
    * 福田 :[これ１冊でできる！ラズベリー・パイ　超入門 改訂第6版](https://www.amazon.co.jp/dp/4800712610)、ソーテック社, 2020.<br />$ $
* Linuxの操作
    * 初心者向け
        * [Piro（結城洋志）: シス管系女子, 日経BP, 2015.](https://www.amazon.co.jp/dp/4822224961)
    * 中級者〜
        * 上田: [シェルプログラミング実用テクニック](https://www.amazon.co.jp/dp/4774173444), 技術評論社, 2015.

---

* デバドラ関係
    * Corbet 他(著), 山崎 他(翻訳): [Linuxデバイスドライバ 第3版](https://www.amazon.co.jp/dp/4873112532), オライリージャパン, 2005.
        * デバイスドライバのリファレンス
        * ちょっと古い
        * [英語ならネット上に](http://www.makelinux.net/ldd3/)<br />$ $
    * 米田: [Raspberry Piで学ぶ ARMデバイスドライバープログラミング](https://www.socym.co.jp/book/940), ソシム, 2014.

---

* OS
    * Tanenbaum: [オペレーティングシステム 第3版](https://www.amazon.co.jp/dp/4894717697), ピアソンエデュケーション, 2007.
        * 中古が出回っている
    * Bovet: [詳解 Linuxカーネル 第3版](https://www.amazon.co.jp/dp/487311313X), オライリー・ジャパン, 2007.
    * 竹内: [試して理解 Linuxのしくみ ~実験と図解で学ぶOSとハードウェアの基礎知識](https://www.amazon.co.jp/dp/477419607X)

---

## 宿題: <span style="text-transform:none">Raspberry Pi</span><br />のセットアップ

* Raspberry Piを入手して<span style="color:red">Ubuntu</span>をインストールのこと
    * 昨年までは、最初はRaspbianでしたが最初からUbuntuで
	* 方法
	    * [Install Ubuntu Server on a Raspberry Pi 2, 3 or 4](https://ubuntu.com/download/raspberry-pi)に行く
	    * 持っているRaspberry Piのバージョンに合った新しいバージョンのUbuntu（なるべく64bit）をダウンロード
	    * ダウンロードしたファイルの中身を[Etcher](https://www.balena.io/etcher/)でMicroSDカードに書き込む
	    * モニタかネットワーク越しで動作確認
    * 諸注意
        * デスクトップ環境は使いません
        * ギブアップしてもケアしますから脱落の必要はナシ

---

## <span style="text-transform:none">Raspberry Pi</span>の入出力

* 右図: Raspberry Pi 2 Model B
* 電源入力: MicroUSB
* ディスプレイへ出力: HDMI
* ストレージ: microSDカード
* 有線LANポート
* USBポート
* **40ピンのブロック**
    * 「GPIOピン」と呼ばれる
* 他: 専用カメラの取り付け端子等

<img width="35%" src="https://lab.ueda.tech/wp-content/uploads/2016/08/raspberrypi.png" />

---

## GPIOピンの構成

* Raspberry Pi 2からRaspberry Pi 3 B+まで変わってない<br />$ $
* 配置（Model Bのもの）: http://pinout.xyz
    * 電源: 3.3V、5V
    * GND
    * GPIOピン: 27
        * うち何本かがI2C、SPI、UARTも使える
    * I2C ID EEPROM用ピン<br />$ $
* A/D変換機能は付いていない


---

## CPU・メモリ・ストレージ

* Raspberry Pi 3 B+
    * CPU: 1.4GHz Cortex-A53 ARMv8 64bit（4コア）
    * DRAM: 1GB DDR2 450MHz
    * ストレージ: microSD
         * 細かい規格の制限や相性等に注意
         * 16GBまではトラブルが少ない


---

## <span style="text-transform:none">Raspberry Pi</span>のソフトウェア

* Linuxが標準
    * Raspbian
    * Ubuntu<br />$ $
* ロボットのコントローラとしてのLinux
    * 欠点: マイコンのように単純なリアルタイム制御ができない
    * 長所: 開発時、運用時にインターネットとシームレスに接続
    * これから講義でいろいろやります

---

## 次回

* 教室でRaspberry Piを使います
    * ノートPCにLANケーブルで接続してSSHから操作
