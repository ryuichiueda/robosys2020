## ロボットシステム学2016第13回

上田 隆一

2019年12月20日@千葉工業大学

---

<h2>ロボットと通信</h2>
<ul>
 	<li>自律分散系には必須ですね
<ul>
 	<li>そうですよね？</li>
</ul>
</li>
 	<li>使いますよね
<ul>
 	<li>リモート監視・操作等</li>
 	<li>環境に埋め込んだセンサやアクチュエータの操作</li>
 	<li>ソフトウェアのインストール</li>
</ul>
</li>
 	<li><span style="color: #ff0000;">必須 </span></li>
</ul>

---

<h2>本日の内容</h2>
<ul>
 	<li>ネットワーク関係の設定方法を一通りおさえる</li>
 	<li>イーサネット・TCP/IP IP
<ul>
 	<li>アドレス・ポート</li>
 	<li>ソケット通信については前期やったそうなので割愛</li>
</ul>
</li>
 	<li>ssh</li>
</ul>

---

<h2>IPアドレスの体系</h2>
<ul>
 	<li>計算機の住所（ただし複数持つことができる）</li>
 	<li>IPアドレス: 0-255の数字を4つドットでつないで表記
<ul>
 	<li>例: 192.168.0.1</li>
 	<li>ローカルのものとグローバルのものが存在
<ul>
 	<li>グローバル
<ul>
 	<li>世界中でその計算機しか持っていない</li>
</ul>
</li>
 	<li>ローカル（プライベートIPアドレス）
<ul>
 	<li>閉じた環境で使うアドレス</li>
</ul>
</li>
</ul>
</li>
</ul>
</li>
</ul>

---

<h2>ネットワーク部・ホスト部</h2>
<ul>
 	<li>IPアドレスを2進数で書いた時、左側の何桁かは「ネットワーク部」を表し、残りは「ホスト部」を表す</li>
 	<li>ネットワーク部
<ul>
 	<li>インターネットの中の一つのグループ</li>
</ul>
</li>
 	<li>ホスト部
<ul>
 	<li>各PC固有の番号（住所で言うと番地）</li>
</ul>
</li>
 	<li>サブネットマスク
<ul>
 	<li>どの部分がネットワーク部を表すかを示す数字</li>
 	<li>例: 255.255.255.0
<ul>
 	<li>2進数にすると11111111.11111111.11111111.00000000</li>
 	<li>ということで、左から24ビットがネットワーク部</li>
</ul>
</li>
</ul>
</li>
</ul>

---

<ul>
 	<li>表記の例
<ul>
 	<li>192.168.1.2/255.255.255.0
<ul>
 	<li>192.168.1がネットワーク部で2がホスト部</li>
 	<li>192.168.1.2/24という書き方もある
<ul>
 	<li>左側24ビットがネットワーク部</li>
</ul>
</li>
</ul>
</li>
</ul>
</li>
 	<li>コマンドで確かめてみましょう
<ul>
 	<li>
<pre><span style="color: #ffffff;">$ ip addr
...
<span class="s1">3: eth0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
</span><span class="s1">link/ether b8:27:eb:62:a8:84 brd ff:ff:ff:ff:ff:ff
</span><span class="s1">inet 192.168.0.4/24 brd 192.168.0.255 scope global eth0</span>
...
</span></pre>
</li>
</ul>
</li>
</ul>

---

<h2>ルーティング</h2>
<ul>
 	<li>別のネットワーク部にある計算機には無条件でアクセスできない
<ul>
 	<li>別のネットワークにパケットを出す設定が必要
<ul>
 	<li>計算機で設定しなければならないこと
（DHCPを使っていると自動で設定されているので気がつかない）
<ul>
 	<li>外にパケットを出すときにどのルータに送るか</li>
 	<li>どのIPアドレスが内側のものなのか</li>
</ul>
</li>
</ul>
</li>
 	<li><span style="color: #ff0000;">ルータがネットワークの境界にいて交通整理</span>
<ul>
 	<li>「ルーティング」</li>
 	<li>tracerouteを打ってみましょう（次ページ）</li>
</ul>
</li>
</ul>
</li>
</ul>

---

<pre><span style="color: #ffffff;">$ traceroute 8.8.8.8</span>
<span style="color: #ffffff;">traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets</span>
<span style="color: #ffffff;">1  133.242.186.1 (133.242.186.1)  1.073 ms  1.062 ms  1.047 ms</span>
<span style="color: #ffffff;">2  iskrt102b-rt109e.bb.sakura.ad.jp (103.10.114.73)  1.003 ms iskrt101b-rt109e.bb.sakura.ad.jp (103.10.114.65)  1.013 ms iskrt102b-rt109e.bb.sakura.ad.jp (103.10.114.73)  1.010 ms</span>
<span style="color: #ffffff;">3  iskrt1s-rt101b-2.bb.sakura.ad.jp (103.10.113.93)  0.976 ms iskrt2s-rt102b-2.bb.sakura.ad.jp (103.10.113.105)  0.974 ms iskrt1s-rt101b-1.bb.sakura.ad.jp (103.10.113.9)  0.956 ms</span>
<span style="color: #ffffff;">4  iskrt3-rt2s.bb.sakura.ad.jp (103.10.113.113)  2.061 ms iskrt3-rt1s.bb.sakura.ad.jp (103.10.113.109)  2.050 ms iskrt4-rt2s.bb.sakura.ad.jp (103.10.113.121)  0.892 ms</span>
<span style="color: #ffffff;">5  tkort3-iskrt3.bb.sakura.ad.jp (157.17.131.33)  16.704 ms tkert1-iskrt4.bb.sakura.ad.jp (157.17.131.37)  20.049 ms tkort3-iskrt3.bb.sakura.ad.jp (157.17.131.33)  16.672 ms</span>
<span style="color: #ffffff;">6  as15169.ix.jpix.ad.jp (210.171.224.96)  20.114 ms  19.862 ms tkort3-ert1.bb.sakura.ad.jp (157.17.130.113)  18.166 ms</span>
<span style="color: #ffffff;">7  as15169.ix.jpix.ad.jp (210.171.224.96)  21.424 ms 108.170.242.161 (108.170.242.161)  20.457 ms as15169.ix.jpix.ad.jp (210.171.224.96)  21.547 ms</span>
<span style="color: #ffffff;">8  209.85.255.141 (209.85.255.141)  21.071 ms 108.170.242.193 (108.170.242.193)  22.333 ms 72.14.239.193 (72.14.239.193)  21.048 ms</span>
<span style="color: #ffffff;">9  72.14.238.173 (72.14.238.173)  22.553 ms 72.14.239.31 (72.14.239.31)  19.628 ms google-public-dns-a.google.com (8.8.8.8)  17.074 ms</span></pre>

---

<ul>
 	<li>ルーティング情報の閲覧・設定
<ul>
 	<li> route(8)を使います</li>
 	<li>下の例の読み方
<ul>
 	<li>255.255.254.0でマスクをかけた時のIPアドレスが一致すればeth0から相手のIPアドレスに直接送信</li>
 	<li>それ以外の場合はeth0から
「デフォルトゲートウェイ」133.242.186.1に送信</li>
</ul>
</li>
</ul>
</li>
</ul>
<pre class="p1"><span style="color: #ffffff;"><span class="s1">ueda\@remote</span><span class="s2">:</span><span class="s3">~</span><span class="s2">$ route
</span><span class="s1">カーネルIP経路テーブル
</span><span class="s1">受信先サイト<span class="Apple-converted-space">    </span>ゲートウェイ<span class="Apple-converted-space">    </span>ネットマスク <span class="Apple-converted-space">  </span>フラグ Metric Ref 使用数 インタフェース
</span><span class="s1">default <span class="Apple-converted-space">        </span>133.242.186.1 <span class="Apple-converted-space">  </span>0.0.0.0 <span class="Apple-converted-space">        </span>UG<span class="Apple-converted-space">    </span>0<span class="Apple-converted-space">      </span>0<span class="Apple-converted-space">        </span>0 eth0
</span><span class="s1">localnet<span class="Apple-converted-space">        </span>* <span class="Apple-converted-space">              </span>255.255.254.0 <span class="Apple-converted-space">  </span>U <span class="Apple-converted-space">    </span>0<span class="Apple-converted-space">      </span>0<span class="Apple-converted-space">        </span>0 eth0</span></span></pre>

---

<h2>デフォルトゲートウェイの追加・削除</h2>
<ul>
 	<li>やってみましょう。</li>
</ul>
<pre><span style="color: #ffffff;">ueda\@ubuntu16:~$ route -n</span>
<span style="color: #ffffff;">カーネルIP経路テーブル</span>
<span style="color: #ffffff;">受信先サイト    ゲートウェイ    ネットマスク   フラグ Metric Ref 使用数 インタフェース</span>
<span style="color: #ffffff;">0.0.0.0         192.168.2.1     0.0.0.0         UG    0      0        0 enp0s3</span>
<span style="color: #ffffff;">192.168.2.0     0.0.0.0         255.255.255.0   U     0      0        0 enp0s3</span>
<span style="color: #ffffff;">ueda\@ubuntu16:~$ sudo route del default gw 192.168.2.1</span>
<span style="color: #ffffff;">ueda\@ubuntu16:~$ ping 8.8.8.8 #パケットが外に行かない</span>
<span style="color: #ffffff;">connect: Network is unreachable</span>
<span style="color: #ffffff;">ueda\@ubuntu16:~$ sudo route add default gw 192.168.2.1</span>
<span style="color: #ffffff;">ueda\@ubuntu16:~$ ping 8.8.8.8 #今度はうまくいく</span>
<span style="color: #ffffff;">PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.</span>
<span style="color: #ffffff;">64 bytes from 8.8.8.8: icmp_seq=1 ttl=54 time=4.50 ms</span>
<span style="color: #ffffff;">...</span></pre>

---

<h2>IPアドレスの設定</h2>
<ul>
 	<li>有線（Raspberry Piの場合）
<ul>
 	<li>最初からDHCPに設定されている</li>
 	<li>通常はこのままDHCPで良い
<ul>
 	<li>固定すると別の環境でログインできなくなる等、難しくなる</li>
</ul>
</li>
</ul>
</li>
 	<li>IPアドレスはルータのウェブページ、nmap等で確認可能
<ul>
 	<li>
<pre><span style="color: #ffffff;">$ nmap -sP 192.168.2.0/24</span></pre>
</li>
</ul>
</li>
 	<li>DHCPでもルータ等で固定できる</li>
</ul>

---

<h2>固定IPの設定</h2>
<ul>
 	<li>/etc/network/interfacesに設定を書く
<ul>
 	<li>下図: デフォルトの設定例（バージョンによって異なる）</li>
</ul>
</li>
</ul>
<pre><span style="color: #ffffff;">pi\@raspberrypi ~ $ cat /etc/network/interfaces</span>
<span style="color: #ffffff;">auto lo</span>
<span style="color: #ffffff;">iface lo inet loopback</span>

<span style="color: #ffffff;">iface eth0 inet dhcp </span></pre>

---

<ul>
 	<li>固定にする例
<ul>
 	<li>DHCPでもらったアドレスと同じネットワーク部を持つ別のアドレスに変えてみましょう</li>
</ul>
</li>
 	<li>
<pre><span style="color: #ffffff;">pi\@raspberrypi ~ $ cat /etc/network/interfaces</span>
<span style="color: #ffffff;">auto lo</span>

<span style="color: #ffffff;">iface lo inet loopback</span>
<span style="color: #ffffff;">#iface eth0 inet dhcp</span>
<span style="color: #ffffff;">auto eth0</span>
<span style="color: #ffffff;">iface eth0 inet static</span>
<span style="color: #ffffff;">address 192.168.1.200</span>
<span style="color: #ffffff;">netmask 255.255.255.0</span>
<span style="color: #ffffff;">gateway 192.168.1.1</span></pre>
<ul>
 	<li>dhcpの設定は#でコメントアウトを</li>
 	<li>設定後はrebootするのが素直</li>
 	<li>したくない時は</li>
 	<li>
<pre><span style="color: #ffffff;">$ sudo service networking restart</span></pre>
</li>
</ul>
</li>
</ul>

---

<h2>ポート</h2>
<ul>
 	<li>ポート= port、港</li>
 	<li>計算機が役所のようなものだとすればポートは窓口
<ul>
 	<li>窓口は65536*2個ある
<ul>
 	<li>TCP（Transmission Control Protocol）0 番～65,535 番</li>
 	<li>UDP（User Datagram Protocol）0 番～65,535 番</li>
</ul>
</li>
</ul>
</li>
 	<li>インターネット上のサービスを利用するときは
<ul>
 	<li>IPアドレスとプロトコル、ポート番号を指定している</li>
 	<li>IP アドレス: 住所</li>
 	<li>ポート: 窓口
<ul>
 	<li>窓口の後ろにサービスを提供する人（サーバ）がいる</li>
</ul>
</li>
</ul>
</li>
</ul>

---

<h2>/etc/services</h2>
<ul>
 	<li>よく使われるポート番号を表にしたもの
<ul>
 	<li>大抵のLinuxには入っている</li>
 	<li>端末からless等で読んでみましょう</li>
</ul>
</li>
 	<li>必ずしもサーバがこのポート番号を使う必要はないが、標準的なものにしておくと使うときに調べなくていい
<ul>
 	<li>HTTP: TCP80, HTTPS: TCP443, SSH: TCP22, ...</li>
</ul>
</li>
 	<li>
<p class="p1"><span class="s1">$ netstat -antuで現在使っているポートの番号を調べ、
どのサービスが通常使うポートか/etc/servicesで調査してみましょう</span></p>
</li>
</ul>

---

<h2>名前解決</h2>
<ul>
 	<li>IPアドレスとホスト名（www.yahoo.co.jp等）はどう変換される？</li>
 	<li>DNSサーバ
<ul>
 	<li>IPアドレスとホストを管理</li>
 	<li>ブラウザやpingでホスト名が指定されるとDNSサーバに問い合わせが行く
<ul>
 	<li>DNSサーバのIPアドレスは通常/etc/resolv.confに書く（DHCPを使っていると勝手に書かれている）
<ul>
 	<li>最近はresolv.confを使わない方向に</li>
</ul>
</li>
</ul>
</li>
</ul>
</li>
 	<li>/etc/hosts
<ul>
 	<li>静的にIPアドレスとホストを対応付けたい時に編集</li>
</ul>
</li>
</ul>

---

<h2>WiFiの設定</h2>
<ul>
 	<li>WiFi標準搭載のRaspberry Pi3の場合を例に</li>
 	<li>準備
<ul>
 	<li>iwconfigでwlan0があることを確認</li>
</ul>
</li>
</ul>
<pre><span style="color: #ffffff;">$ sudo apt install wireless-tools</span>
<span style="color: #ffffff;">$ sudo apt install wpasupplicant</span>
<span style="color: #ffffff;">$ iwconfig</span>
<span style="color: #ffffff;"> wlan0     IEEE 802.11bgn  ESSID:off/any</span>
<span style="color: #ffffff;">           Mode:Managed  Access Point: Not-Associated</span>
<span style="color: #ffffff;">           Retry short limit:7   RTS thr:off   Fragment thr:off</span>
<span style="color: #ffffff;">           Power Management:on</span></pre>

---

<h2>アクセスポイントに接続</h2>
<ul>
 	<li>接続したいアクセスポイントのSSID（ESSID）とパスフレーズをwpa_passphrase(1)に指定して設定ファイル（wpa_supplicant.conf）を作る</li>
</ul>
<pre><span style="color: #ffffff;">$ sudo -s</span>
<span style="color: #ffffff;"> # wpa_passphrase SSID パスフレーズ &gt; /etc/wpa_supplicant/wpa_supplicant.conf</span></pre>

---

<ul>
 	<li> /etc/network/interfacesに必要事項を書く
<ul>
 	<li>ホットプラグ</li>
 	<li>wpa_supplicant.confの場所</li>
 	<li>IPアドレス周り</li>
 	<li>パワーマネージメントを切っておくと安定</li>
</ul>
</li>
</ul>
<pre><span style="color: #ffffff;">$ sudo cat /etc/network/interfaces</span>
<span style="color: #ffffff;">（略 lo, eth0の設定）</span>
<span style="color: #ffffff;">auto wlan0</span>
<span style="color: #ffffff;">allow-hotplug wlan0</span>
<span style="color: #ffffff;">iface wlan0 inet dhcp</span>
<span style="color: #ffffff;">wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf</span>
<span style="color: #ffffff;">wireless-power off
</span></pre>
<ul>
 	<li>再起動（あるいはsudo ifup wlan0, sudo service networking restart）</li>
</ul>
