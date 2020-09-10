# ロボットシステム学第6回

上田 隆一

千葉工業大学

---

## 今日の内容

* ファイルシステムとデバイス

---

## ファイルシステム

* OSの最重要機能の一つ<br />　
* 機能
  * 誰にでも分かりやすいもの
    * HDDを使えるようにする
    * USBメモリを使えるようにする
  * 人によっては意外なもの
    * 外部の機器（センサ等）を使えるようにする
    * OS と通信する

---

## ローカルファイルシステム

* ストレージ側のファイルシステム
  * 何百万、何百兆のゼロイチの並びのどこにファイル・ディレクトリ、その他メタデータ等を置くかを規定・実現<br />　
* 主なファイルシステム
  * ext4, ext3, (extended file system): Linux
  * UFS2 (UNIX File System2): FreeBSD
  * APFS (Apple File System), HFS+: Mac OS X, macOS
  * NTFS (NT File System), FAT16, FAT32, exFAT: Windows<br />　
* 特殊なファイルシステム
  * procfs, sysfs, tmpfs, スワップファイルシステム

---

## ストレージにデータを記録する仕組み

* ストレージがあったら、いくつかのパーティションに分け、それぞれの形式でファイルシステムをフォーマットして使う
  * ストレージの区分け（上位レイヤーから順に）
    * パーティション
    * ローカルファイルシステム
    * ブロックグループ
    * ブロック
  * 注意: HDD 自体にもセクタ、トラック、シリンダーという階層構造があるが、それは別の話

---

## パーティションの確認

```bash
$ sudo parted -l
Model: SD SA16G (sd/mmc)
Disk /dev/mmcblk0: 15.6GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type      File system  Flags
1      4194kB  1089MB  1085MB  primary   fat32        lba
2      1091MB  15.6GB  14.5GB  extended
5      1095MB  1158MB  62.9MB  logical   fat16        lba
6      1162MB  15.6GB  14.5GB  logical   ext4
3      15.6GB  15.6GB  33.6MB  primary   ext4
```

---

## <span style="text-transform:none">iノード、ディレクトリ

* iノード: ファイルを管理するためのデータ
  * iノード番号: ファイルやディレクトリ（これもファイル）の固有番号
  * `$ ls -i` で、iノード番号が表示できる
  * ディレクトリもファイルなので当然iノード番号を持つ<br />　
* ディレクトリ
  * ディレクトリエントリのリストの管理
  * ディレクトリエントリ
  * ファイルのiノードと名前
    * （ファイル名を変えても変わるのはファイル本体ではない）

---

## 仮想ファイルシステム（VFS）

* 主な役割: ローカルファイルシステムの違いを吸収
  * write,read 等のシステムコールが呼ばれると各フォーマット用の関数に変換される
  * iノードという概念がないローカルファイルシステムもiノードをなんとか再現<br />　
* 機能
  * メモリ上にiノード（inode 構造体）やディレクトリエントリ（dentry 構造体）を展開してユーザプログラムとやりとり
  * データブロックの内容をメモリ上に保持（キャッシュ）

---

## パーティションの作成（1/2）

* partedコマンドを利用
  * 例/dev/sdcに見えるUSBメモリにext4のパーティションを作る

```bash
$ sudo parted /dev/sdc
GNU Parted 3.2
Using /dev/sdc
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt                                                      
(parted) unit GB                                                          
(parted) print                                                            
Model: TOSHIBA TransMemory-Mx (scsi)
Disk /dev/sdc: 62.4GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags
``` 

---

## パーティションの作成（2/2）


* パーティションを作ったらext4でフォーマット

```bash
(parted) mkpart
Partition name?  []? data1                                                
File system type?  [ext2]? ext4
Start? 0                                                                  
End? 62.4                                                                 
(parted)   q
$ mkfs.ext4 /dev/sdc1 
```

---

## マウント・アンマウント

* 既存のディレクトリにファイルシステムをぶら下げる
  * どこにでもぶら下げられる
    * 例: `~/tmp`にさっきのUSBメモリのパーティションをぶら下げる

```bash
$ echo aaa > ~/tmp/file1    #元のところにファイルを作っておきましょう
$ ls ~/tmp
 file1
$ cd 
$ sudo mount /dev/sdc1 ~/tmp   #マウント
$ ls ~/tmp/             #~/tmpが差しているディレクトリが変わる
 lost+found
$ df
ファイルシス   1K-ブロック      使用     使用可 使用% マウント位置
（中略）
/dev/sdc1         59871340     53064   56753856    1% /home/ueda/tmp
$ sudo umount ~/tmp     #アンマウント
$ ls ~/tmp/             #元に戻る
file1
```

---

## 擬似ファイルシステム

* 機器の機能やOS内部の情報を提供<br />　
* 主要なもの
  * devtmpfs（/dev/）: デバイスファイル置き場
  * procfs（/proc/）: プロセスの情報＋その他OS の情報
  * sysfs（/sys/）: OSの情報（/proc/が混乱したのでこちらに移動）
  * tmpfs（/run/shm/）: DRAM 上にファイルを置く仕組み

---

## デバイスドライバ・<br />デバイスファイル

* デバイスドライバ
  * 計算機に接続された機器を操作するためのプログラム
  * HDD、ディスプレイ、カメラ、マイク、端末、 ...<br />　
* デバイスファイル
  * デバイスドライバのインタフェース
  * ファイルとして表現されている
    * 機械と言えども、結局データをin/outするという点でファイルとして抽象化可能
  * `/dev/`の下に存在

---

## <span style="text-transform:none">/dev/下のファイル

* `$ ls -l /dev/`と打って表示を観察しましょう
  * 日付右側の数字（10, 58 等）: デバイスの番号
    * メジャー番号（左側）: デバイスドライバ固有の番号
    * マイナー番号（右側）: デバイスドライバの中で与えられる番号
  * ブロックデバイス、キャラクタデバイスの判別
    * パーミッション前の文字（b: ブロック、c: キャラクタ）
    * ブロックデバイス: データをブロック（塊）単位で読み書き
    * キャラクタデバイス: データをシーケンシャルに出し入れ
    * もう一つ、「ネットワークデバイス」（ eth0等のあれ）があるがLinuxでは別扱い

---

## デバイスドライバ

* カーネルの一部として動作
  * Cで言えばmainを持たない関数の塊
  * 本来はカーネルと同時にコンパイルされるべき
    * ただしそれをやってしまうと時間や手間の無駄<br />　
* Linuxにはカーネルの一部を動的に脱着する仕組みが存在
  * カーネルの一部: カーネルモジュール（拡張子 .ko）

---

## カーネルモジュールの調査

* ファイルの検索
  * `  sudo find / | grep '\.ko$'`
  * 今使われているカーネルモジュールはlsmodで調査

```bash
$ lsmod
Module                  Size  Used by
binfmt_misc             6388  1 
bnep                   10340  2 
hci_uart               17943  1 
btbcm                   5929  1 hci_uart
bluetooth             326067  22 bnep,btbcm,hci_uart
（以下略）
```

---

## カーネルモジュールの脱着

* insmod, rmmod, 他にmodplobe
  * 例:

    ```bash
    $ lsmod
    （中略）
    i2c_dev                 5859  0 
    （略）
    $ sudo rmmod i2c_dev
    $ lsmod | grep i2c
    $ 
    $ sudo insmod /usr/src/linux/drivers/i2c/i2c-dev.ko 
    ### ↑ここにない場合は自分でfindで探しましょう ###
    $ lsmod
    Module                  Size  Used by
    i2c_dev                 5859  0 
    （以下略）
    ``` 
* insmodすると/sys/下で情報が見られるようになるので興味のある人は調査を
  * あとでカーネルモジュールを書くときにやります

