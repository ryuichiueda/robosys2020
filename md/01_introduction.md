# ロボットシステム学第1回

上田 隆一

千葉工業大学


---

## 今日の内容

* 動機付け（なんでこの講義があるのか）

---

## ロボットシステム

* とはなんだろう？
* この講義で言うところのロボットシステム
   * 目的を遂行するために動く
   * 高度に自動化されている
       * 例1: サッカーロボット、つくばチャレンジのロボット、ルンバ
       * <span style="color:red">例2: 新幹線、ゆりかもめ、自動ドア、エレベータ</span><br />　
* 少し拡大解釈すると、ロボット（的なもの）<br />は十分世の中に普及している
   * 趣味のものではないので、どう効率よく開発するかを大真面目に考えないといけない

---

## ロボットシステム開発の変化

* 真面目に健気にやればいいというものではない
    * ハードやソフトの多くが部品化
        * 作るより「つなぐ」機会が相対的に増加、つなぐ技術が必要
    * 使える道具が増加
        * CAD、3Dプリンタ、ROS、・・・
        * 使うか使わないかで開発速度が大きく変わる
    * インターネット上に知識が集積
        * 勉強しなくてもロボットが作れる（？）
        * 我々は何を勉強するべきなのか？<br />　

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
        * サンプルコードをうまく改造してロボットに仕事をさせる

---

## この講義

* 手っ取り早い方法を節操なく勉強する<br />　
* ただし、しっかりと基礎知識を付けて効果的に
    * 手っ取り早い方法には落とし穴が多い
    * ただウェブサイトを見てソフトをダウンロードして組み合わせる以上のことを学習

$\Longrightarrow$組み合わせることを実現する<br />コンピュータや社会の仕組みを学習

---

## 手っ取り早さの実現


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

<img width="35%" src="md/images/raspi4.jpeg" />

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
    * $\Longrightarrow$低レイヤーから高レイヤー、さらにその外側まで順番に一通り経験することで、各レイヤーの技術をつなげて考えられるようにする。<br >　
* 低レイヤー
    * OSの仕組み
    * デバイスドライバ作成
* 高レイヤー
    * ROSのモジュール作成
* 社会的レイヤー
    * 開発方法（Git、GitHub）
    * 著作権やライセンス

---

## 講義で使う道具

* Raspberry Pi
* Raspberry Piのアクセサリ（次のページ）<br />$ $
* ノートPC
    * 役割1: Raspberry Piとssh通信
    * 役割2: Raspberry Piと繋がらないときの保険
        * WSL or 仮想マシン上 or ネイティブなUbuntu
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
* 連絡: 他人に見られてもいい内容はTwitterで[@ryuichiueda](https://twitter.com/ryuichiueda)あるいは[@uedalaboratory](https://twitter.com/uedalaboratory)
    * 質問も重要なコントリビューション<br />$ $
* [この講義資料](https://github.com/ryuichiueda/robosys2020)にプルリクくれて私がマージしたら加点

