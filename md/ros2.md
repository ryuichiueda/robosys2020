# ロボットシステム学第14回

上田 隆一

2020年1月10日@千葉工業大学

---

## 今日の内容

* ROS2
    * 元のROS（ROS2が出たのでROS1と呼ばれる）との違いは？
    * 参考書
        * 近藤豊: ROS2ではじめよう 次世代ロボットプログラミング, 技術評論社, 2019. 
        * 今回の内容はこの本や様々な人のブログから
* 課題の説明・テストの説明

---

## ROS1からROS2

* Ubuntu限定$\rightarrow$様々なOSへ
* rosmasterが単一故障点$\rightarrow$rosmasterをなくして分散処理
* セキュリティーの考慮
* 他、いろいろ

---

## ROS2のインストールと実行

* スクリプトにまとめました
    * https://github.com/ryuichiueda/ros2_setup_scripts_Ubuntu18.04_desktop
* 実行
    * サンプルのノードを二つ動かしてみましょう
```
### サブスクライバを持つノードを立ち上げる ###
端末1$ ros2 run demo_nodes_cpp listener
### 別の端末でパブリッシャを持つノードを立ち上げる ###
端末2$ ros2 run demo_nodes_cpp talker
[INFO] [talker]: Publishing: 'Hello World: 1'
[INFO] [talker]: Publishing: 'Hello World: 2'
・・・
### listener（端末1）の方に受け取ったデータが表示される ###
[INFO] [listener]: I heard: [Hello World: 1]
[INFO] [listener]: I heard: [Hello World: 2]
・・・
```

---

## パッケージの作成

* 未検証なのですがこちらが参考になりそうです
    * [ROS2で自分で作成したpythonプログラムを動かす方法 | @inomu](https://qiita.com/inomu/items/5bd47b4d223434d57615)

---

## 課題2（1/24まで） 

* ROSで何かを作る
  * 例（難しいものから）
    * 自分のロボットをROS化
    * LEDを光らせるノード
    * 講義のものを改造（ここまでが満点の可能性）
    * 講義のものをそのままGitHubにアップ
  * 注意
    * 課題1と同じくGitHubにプッシュ->Youtube->私まで電子メール
    * ライセンス、何をやったか分かるREADMEを書く
    * 得点の目安は課題1と同じ

---

## テスト

* [こちら](https://lab.ueda.tech/?page=robosys_2019)に過去問があります。
