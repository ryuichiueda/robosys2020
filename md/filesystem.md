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

## パーティションの確認1

* microSDカードの使われ方を`parted`で観察
    * ディスク: `/dev/mmcblk0`として認識されている
        * ファイル？（後述）

```bash
$ sudo parted -l
モデル: SD SA16G (sd/mmc)
ディスク /dev/mmcblk0: 15.6GB
セクタサイズ (論理/物理): 512B/512B
パーティションテーブル: msdos
ディスクフラグ:

番号  開始    終了    サイズ  タイプ   ファイルシステム  フラグ
 1    1049kB  269MB   268MB   primary  fat32             boot, lba
 2    269MB   15.6GB  15.3GB  primary  ext4
```

---

## パーティションの確認2

* 各パーティションのOSから見え方を`df`で調査
    * `/dev/mmcblk0p1`、`/dev/mmcblk0p2`と名前がついている
        * これもファイル？（後述）
    * `/dev/mmcblk0p2`がメインのストレージ
    * `/dev/mmcblk0p1`は`/boot/firmware`用

```bash
$ df -Th | grep -e Filesystem -e mmc
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/mmcblk0p2 ext4       14G  3.1G   11G  23% /
/dev/mmcblk0p1 vfat      253M   97M  157M  39% /boot/firmware
```

---

## <span style="text-transform:none">ext4</span>のブロック

* ブロックのサイズが4096バイト 
* ブロックグループには32768個のブロック

```bash
$ sudo dumpe2fs /dev/mmcblk0p2 | grep -i block | head
dumpe2fs 1.45.5 (07-Jan-2020)
Block count:              3731195
Reserved block count:     155491
Free blocks:              2955122
First block:              0
Block size:               4096
Reserved GDT blocks:      354
Blocks per group:         32768
Inode blocks per group:   495
Flex block group size:    16
Reserved blocks uid:      0 (user root)
```


---

## ブロックグループのレイアウト

* 以下のブロックが順に並んでいる（実例は次ページ）
    * ※スーパーブロック: パーティションの情報
    * ※グループディスクリプタ: あとのブロックグループの情報
    * （拡張用のブロック）
    * データブロックビットマップ: ブロックの使用状況のフラグ
    * <span style="color:red">iノード</span>ビットマップ: iノードの使用状況のフラグ
    * iノードテーブル: iノードのリスト
    * データブロック: ファイルの中身<br />　
* <span style="font-size:70%">iノードについてはあとで</span>
* <span style="font-size:70%">※: 最初のブロックグループにしかないが、バックアップのためにいくつかのブロックグループも持っている</span>
* <span style="font-size:70%">参考: https://ext4.wiki.kernel.org/index.php/Ext4_Disk_Layout</span>

---

## 最初のブロックグループ

```bash
$ sudo dumpe2fs /dev/mmcblk0p2 | sed -n '/グループ 0/,/グループ 1/p' | sed '$d'
dumpe2fs 1.45.5 (07-Jan-2020)
グループ 0: (ブロック 0-32767) csum 0x5a66 [ITABLE_ZEROED]
  Primary superblock at 0, Group descriptors at 1-2
  Reserved GDT blocks at 3-356
  Block bitmap at 357 (+357)
  Inode bitmap at 373 (+373)
  Inode table at 389-883 (+389)
  117 free blocks, 1 free inodes, 754 directories
  Free blocks: 24504, 24596-24601, 24684, 24759, 24838, 24980, （略）
  Free inodes: 746
```


---

## <span style="text-transform:none">iノード、ディレクトリ</span>

* iノード: ファイルを管理するためのデータ
  * ファイルの情報やデータブロックへのポインタを有する
  * iノード番号: ファイルの固有番号
     * `$ ls -i` で表示
     * ディレクトリもファイルなのでiノード番号を持つ<br />　
* ディレクトリ
  * ファイル名と対応するiノード番号のリスト
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

## ジャーナリング

* ストレージの書き込み途中にシステムが止まってもデータを壊さない仕組み<br />　
* ファイルシステムの更新内容を記録しておく
  * この仕組みが備わったファイルシステム: ジャーナリングファイルシステム
     * ext3の途中から備わっている<br />　
* 原理<span style="font-size:70%">（注意: ext4の方法を調べたわけではなく頭の中で考えた方法です）</span>: 
  * ファイルを書き換える前に書き換え開始を宣言し、書き換え後に書き換え終了の宣言をする
  * 書き換え終了宣言がないファイルについて書き換え前に戻す


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

* 機器との通信やOS内部の情報提供に利用<br />　
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


---

## まとめ

* ファイルシステム
  * ローカルファイルシステム
    * ストレージを有効に使う工夫
    * ジャーナリングなどデータを守る仕組み
  * VFS
    * 異なるハードウェアや疑似デバイスの抽象化<br />　
* 疑似ファイルシステム
    * ファイルをインタフェースとして使う<br />　
* デバイスファイル、デバイスドライバ
    * 次週、作成します。
