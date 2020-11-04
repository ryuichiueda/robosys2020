# ロボットシステム学第8回<span style="font-size:50%">（第7回が簡単だった人用）</span>

上田 隆一

千葉工業大学

---

## ロボットと通信

* ロボットを扱うときは同時に通信も扱っている
    * そうですよね？<br />　
* 用途
    * リモート監視・操作等
    * 環境に埋め込んだセンサやアクチュエータの操作
    * ソフトウェアのインストール


<span style="color:red;">必須 </span>

---

## 本日の内容

* ネットワーク関係の設定方法を一通りおさえる<br />　
* イーサネット・TCP/IP
    * アドレス・ポート
    * 通信してみる<br />　
* ssh

---

## IPアドレスの体系

* 計算機の住所
    * 細かく説明するとネットワークカードに割り当て
        * 複数割り当てすることも可能<br />　
* IPアドレス（IPv4）
    * 0-255の数字を4つドットでつないで表記
        * 例: 192.168.0.1
    * ローカルのものとグローバルのものが存在
        * グローバルIPアドレス
            * 世界中でその計算機しか持っていない
        * ローカル（プライベートIPアドレス）
            * 閉じた環境で使うアドレス

---

## ネットワーク部・ホスト部

* IPアドレスを2進数で書いた時、
    * 左側の何桁かは「ネットワーク部」
        * インターネットの中の一つのグループ
    * 残りは「ホスト部」
        * 各PC固有の番号（住所で言うと番地）<br />　
* サブネットマスク
    * どの部分がネットワーク部を表すかを示す数字
        * 例: 255.255.255.0
            * 2進数にすると11111111.11111111.11111111.00000000
            * ということで、左から24ビットがネットワーク部

---

## IPアドレスの表記の例

 * 192.168.1.2/255.255.255.0
    * 192.168.1がネットワーク部で2がホスト部
    * 192.168.1.2/24という書き方もある
 	* 左側24ビットがネットワーク部という意味<br />　
 * コマンドで確かめてみましょう
    * `ip a`（`ip addr`）を利用（読み方は次ページ）

<span style="font-size:75%">

```
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
（略）
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether dc:a6:32:8d:10:ba brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.6/24 brd 192.168.1.255 scope global dynamic eth0
（略）
```

</span>

---

## <span style="text-transform:none">ip a</span>の主な項目

* `lo, eth0, wlan0`: ネットワークデバイス
  * `lo`は自分自身を指す特殊なデバイス
* `link`の横の`dc:a6:32:8d:10:ba`など: MACアドレス
* `inet, inet6`の横: IPアドレス

<span style="font-size:75%">

```
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether dc:a6:32:8d:10:ba brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.6/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 83344sec preferred_lft 83344sec
    inet6 240d:1a:a5c:6600:dea6:32ff:fe8d:10ba/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 5637sec preferred_lft 5637sec
    inet6 fe80::dea6:32ff:fe8d:10ba/64 scope link
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether dc:a6:32:8d:10:bb brd ff:ff:ff:ff:ff:ff
```

</span>

---

## ルーティング

* 別のネットワーク部にある計算機には無条件でアクセスできない
    * 別のネットワークにパケットを出す設定が必要
 	* 計算機で設定しなければならないこと（DHCPを使っていると自動で設定されているので気がつかない）
 	    * どのIPアドレスへのパケットを外に出すのか
 	    * 外にパケットを出すときにどのネットワークデバイスから送るか<br />　
* <span style="color:red">ルータがネットワークの境界にいて交通整理</span>
   * 「ルーティング」
   * tracerouteを打ってみましょう（次ページ）

---

```
$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (160.16.96.1)  0.227 ms  0.170 ms  0.160 ms
 2  tkgrt1b-grt25e.bb.sakura.ad.jp (157.17.135.1)  19.745 ms tkgrt1b-grt25e-2.bb.sakura.ad.jp (157.17.135.17)  19.720 ms tkgrt1b-grt25e.bb.sakura.ad.jp (157.17.135.1)  19.682 ms
 3  tkgrt1s-grt1b-2.bb.sakura.ad.jp (157.17.130.177)  0.292 ms  0.308 ms tkwrt1s-grt1b-2.bb.sakura.ad.jp (157.17.130.185)  0.364 ms
 4  tkort4-wrt1s-3.bb.sakura.ad.jp (157.17.131.145)  1.000 ms tkort4-wrt1s.bb.sakura.ad.jp (157.17.131.81)  0.756 ms tkort4-grt1s.bb.sakura.ad.jp (157.17.130.249)  0.646 ms
 5  as15169.ix.jpix.ad.jp (210.171.224.96)  0.986 ms  0.966 ms  0.950 ms
 6  108.170.242.97 (108.170.242.97)  1.093 ms 108.170.242.129 (108.170.242.129)  2.402 ms 108.170.242.97 (108.170.242.97)  1.227 ms
 7  72.14.234.199 (72.14.234.199)  1.220 ms 108.170.233.79 (108.170.233.79)  1.149 ms 108.170.233.15 (108.170.233.15)  1.249 ms
 8  dns.google (8.8.8.8)  1.199 ms  1.226 ms  1.362 ms
```

---

## ルーティング情報の閲覧・設定

* `routel`あるいは`ip r`（`ip route`）を使います
  * 主な項目
    * `target`: 宛先、`gateway`: どこを通るか、`source`: パケット発生元、<br />`dev`: ネットワークデバイス

<span style="font-size:80%">

```
$ routel
         target            gateway          source    proto    scope    dev tbl
        default        192.168.1.1     192.168.1.6     dhcp            eth0
   192.168.1.0/ 24                     192.168.1.6   kernel     link   eth0
    192.168.1.1                        192.168.1.6     dhcp     link   eth0
      127.0.0.0          broadcast       127.0.0.1   kernel     link     lo local
     127.0.0.0/ 8            local       127.0.0.1   kernel     host     lo local
      127.0.0.1              local       127.0.0.1   kernel     host     lo local
127.255.255.255          broadcast       127.0.0.1   kernel     link     lo local
    192.168.1.0          broadcast     192.168.1.6   kernel     link   eth0 local
    192.168.1.6              local     192.168.1.6   kernel     host   eth0 local
  192.168.1.255          broadcast     192.168.1.6   kernel     link   eth0 local
（以下、IPv6の情報）
```

</span>

---

## デフォルトゲートウェイの追加・削除

やってみましょう

```
### ip rのdefaultの項目をチェック ###
$ ip r
default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.6 metric 100
（略）
### 消す ###
$ sudo ip r del default
$ ip r
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.6
192.168.1.1 dev eth0 proto dhcp scope link src 192.168.1.6 metric 100
### 外にパケットが行かなくなる ###
$ ping 8.8.8.8
ping: connect: ネットワークに届きません
### 戻す ###
$ sudo ip r add default via 192.168.1.1 dev eth0
（ip rやpingで確認を）
```

---

## ネットワークの設定（DHCP）

* これまでの講義での`eth0`の設定
  * DHCPで設定されている
    * DHCP: dynamic host configuration protocol 
      * ローカルネットワーク上にサーバ（大抵はルータがその役をしている）がいてIPアドレスなどを勝手に設定してくれる
  * 通常はこのままDHCPでよい
    * 固定すると別の環境でログインできなくなる等、難しくなる
    * ルータ側でDHCPのままIPアドレスを固定できる

---

## IPアドレスの確認


* IPアドレスはルータのウェブページ、`nmap`、`arp`等で確認可能
  * Windowsの場合はコマンドプロンプトで`arp`を使用
    * WSLだと`nmap`や`arp`の使用には制限があるので注意
    * MACアドレスからラズパイを特定
      * ラズパイ3以前だと`B8:27:EB:...`、ラズパイ4以降だと`DC:A6:32`

```
$ arp -a
? (192.168.1.7) at 00:22:58:91:3c:9e [ether] on wlp2s0
? (192.168.1.14) at 60:84:bd:b5:94:72 [ether] on wlp2s0
? (192.168.1.6) at dc:a6:32:8d:10:ba [ether] on wlp2s0
（以下略）
```


---

## ネットワークの設定（手動）

* 新しいUbuntu（18.04以降）ではnetplanを利用
  * `/etc/netplan/99-home.yaml`などと二桁の数字をつけて設定ファイルを作成
    * `99`にしておくと、他のファイル（`01-network-manager-all.yaml`や`50-cloud-init.yaml`）などよりあとに読まれて設定が反映される
* 設定例

<span style="font-size:75%">

```
$ cat /etc/netplan/99-home.yaml     #←このファイルを作る
network:
    ethernets:
        eth0:
            dhcp4: no                          #dhcp使わない
            addresses: [192.168.1.101/24]      #IPアドレス
            gateway4: 192.168.1.1              #デフォルトゲートウェイ
            nameservers:                       #DNSサーバ（後述）
                addresses: [8.8.8.8,8.8.4.4]
            optional: true                     #接続失敗しても待たない（固まらない）
    version: 2
$ sudo netplan apply
（通信が途切れる）
$ ssh ubuntu@192.168.1.101    #←接続しなおし
```

</span>

---

## WiFiの設定（DHCP）

* 別のファイルに次のように設定
  * `access-points`: SSID、ESSID
  * `password`: キー

<span style="font-size:75%">

---

## WiFiの設定（省電力設定off）

* 省電力設定を切らないと不安定になることがある
* 手順
  * `cron`の設定を編集（エディタを選ばされるので、適当に選ぶ）
```
$ sudo crontab -e
```
    * エディタが開いたら、一番下に次のように記述
```
@reboot /sbin/iwconfig wlan0 power off
```
  * 再起動して`iwconfig`（なければインストール）で確認
    * `Power Management:off`となっていなければもう一度`sudo crontab -e`で確認
```
$ iwconfig
（略）
wlan0     IEEE 802.11  ESSID:"tako"
（略）
          Power Management:off   ←←←offを確認！！
（略）
```





---

## ポート

* ポート= port、港
* 計算機が役所のようなものだとすればポートは窓口
  * 窓口は65536*2個ある
    * TCP（Transmission Control Protocol）0 番～65,535 番
    * UDP（User Datagram Protocol）0 番～65,535 番<br />　
* インターネット上のサービスを利用しているとき
  * IPアドレスとプロトコル、ポート番号を指定している
    * 例
      * ラズパイにssh接続: ラズパイのIPアドレスとTCP22番ポート
      * ブラウザに`https://b.ueda.tech`と指定: 160.16.96.252のTCP443ポート
    * ポートの後ろにサービスを提供する人（サーバ）がいる
    * なぜURLがIPアドレスに変わるかは後述

---

## <span style="text-transform:none">/etc/services</span>

* よく使われるポート番号を表にしたもの
  * 大抵のLinuxには入っている
  * 端末からless等で読んでみましょう<br />　
* 必ずしもサーバがこのポート番号を使う必要はないが、標準的なものにしておくと使うときに調べなくてよい
  * HTTP: TCP80, HTTPS: TCP443, SSH: TCP22, ...<br />　
* 演習
  * `netstat -antu`で現在使っているポートの番号を調べ、どのサービスが通常使うポートか/etc/servicesで調査してみましょう

---

## シェルで通信してみる

* `nc`（netcat）を使う
    * Windowsだと怪しいアプリだと起こられるかもしれない
* 例: ラズパイからPCへの通信
```
### PC側 ###
$ sudo nc -l 1000
（TCP1000番を受信できる状態に）
### ラズパイ側 ###
$ echo hoge > /dev/tcp/192.168.1.19/1000 ←IPアドレスはPC側のものに変更
（PC側に文字が送信される）
```

---

## 名前解決

* IPアドレスとホスト名（www.yahoo.co.jp 等）はどう変換される？<br />　
* DNS（Domain Name System）サーバ
  * IPアドレスとホストを管理
  * ブラウザやpingなどでホスト名が指定されるとDNSサーバに問い合わせが行く
    * DNSサーバのIPアドレスは`/etc/resolv.conf`に書くが、最近は自動設定されるので編集しても上書きされる
      * netplanの設定ファイルに書いて、`/etc/resolv.conf`に反映<br />　
* `/etc/hosts`
  * 静的にIPアドレスとホストを対応付けたい時に編集
    * DNSよりこちらが優先
      * ウェブサイトを作るときにブラウザを騙す用途で使うこともある

