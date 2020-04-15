# ロボットシステム学第7回

上田 隆一

2019年11月1日@千葉工業大学

---

## 今日の内容

* デバイスドライバを作る
  * 今回はPCの中で完結する話
  * RaspbianとUbuntuでは少しやり方が違う
* 次回はGPIOの操作に挑戦します
  * Raspberry Pi必須
* リポジトリ
  * https://github.com/ryuichiueda/robosys_device_drivers

---

## 回路の例

* GPIO25とGNDの間にLEDを接続
  * GPIO25: 22番ピン
  * GND: 39番ピン
  * LEDのアノード（足の長い方）<br />をGPIO25に
* 抵抗をつなぐ場合は<br />200-300Ω程度
  * つないだ方がよい

<img width="42%" src="./md/images/gpio25.jpg" />

---

## 最初のコード

* ファイル名は「myled.c」にしましょう
  * 適当なディレクトリを作ってその中に置く

```c
#include <linux/module.h>

static int __init init_mod(void) //カーネルモジュールの初期化
{
        return 0;
}

static void __exit cleanup_mod(void) //後始末
{
}

module_init(init_mod);     // マクロで関数を登録
module_exit(cleanup_mod);  // 同上
```

---

## <span style="text-transform:none">Makefileを書く

* Makefileという名前のファイルを作って次のように書く
* インデントはタブで
* Makefileのごくごく簡単な説明
  * `<作りたいファイル or 行いたい処理>: <必要なファイル>`と書いてその下に実行するコマンドを書く
  * コロンの左側をターゲットと言う
  * makeと打つと最初に書いてあるターゲットの作成が試みられる
  * 必要なファイルがないと、ないファイルのターゲットの処理が試みられる
      
```Makefile
obj-m:= myled.o                           #オブジェクトファイルの名前を指定（拡張子はo）

myled.ko: myled.c                
        make -C /usr/src/linux M=`pwd` V=1 modules     #makeと打つと実行される
clean:
        make -C /usr/src/linux M=`pwd` V=1 clean        #make cleanで実行
```

---

## コンパイル、インストール、<br />実行

* insmodでインストールできる
* `/dev/`等にはまだ何も出てこない

```bash
$ make       #Makefileとmyled.cが正しければこれでOK
$ ls         #確認
Makefile  Module.symvers  modules.order  myled.c  myled.ko  myled.mod.c  myled.mod.o  myled.o
$ sudo insmod myled.ko
$ lsmod
Module                  Size  Used by
myled                    735  0 
...
$ sudo rmmod myled
```

---

## ログを吐くようにする

* `init_mod`と`cleanup_mod`に「printk」という関数を追加
* `KERN_INFO`: ログのレベルを示すマクロ
* `__FILE__`: ソースコードのファイル名

```c
static int __init init_mod(void)
{
        printk(KERN_INFO "%s is loaded.\n",__FILE__);
        return 0;
}

static void __exit cleanup_mod(void)
{
        printk(KERN_INFO "%s is unloaded.\n",__FILE__);
}
```

---

## 動作確認

* `/var/log/messages`にカーネルモジュールの脱着が記録される

```bash
$ make
$ sudo insmod myled.ko
pi@raspberrypi:~/myled_lecture $ tail /var/log/messages
...
Oct 23 11:42:48 raspberrypi kernel: [ 5639.631142] /home/pi/myled_lecture/myled.c is loaded.
$ sudo rmmod myled 
pi@raspberrypi:~/myled_lecture $ tail /var/log/messages
...
Oct 23 11:44:23 raspberrypi kernel: [ 5734.994105] /home/pi/myled_lecture/myled.c is unloaded.
```

---

## ヘッダにモジュールの情報を記述

* 作者、何のモジュール化、ライセンスは何か、バージョン
* ライセンス
  * 基本的にはGPL（後日説明）

```c
#include <linux/module.h>
MODULE_AUTHOR("Ryuichi Ueda");
MODULE_DESCRIPTION("driver for LED control");
MODULE_LICENSE("GPL");
MODULE_VERSION("0.1");

static int __init init_mod(void)
（以下略）
```

---

## 情報の確認

* modinfoというコマンドを利用

```c
$ modinfo myled.ko
filename:       /home/pi/myled_lecture/myled.ko
version:        0.1
license:        GPL
description:    driver for LED control
author:         Ryuichi Ueda
srcversion:     1278C67A0C932CB5D86D367
depends:        
vermagic:       4.4.27-v7+ SMP mod_unload modversions ARMv7 
```

---

## デバイス番号の取得（1/2）

```c
#include <linux/module.h>
#include <linux/fs.h>
（中略）（MODULE_AUTHOR〜MODULE_VERSION）
static dev_t dev;

static int __init init_mod(void)
{
	int retval;
	retval =  alloc_chrdev_region(&dev, 0, 1, "myled");
	if(retval < 0){
		printk(KERN_ERR "alloc_chrdev_region failed.\n");
		return retval;
	}
（次ページに続く）
```

* `alloc_chrdev_region`: デバイス番号の取得
  * 引数: dev（番号の入れ物）のアドレス、0番から1個マイナー番号が欲しい、デバイスの名前はmyled
* `MAJOR`: devからメジャー番号を取り出すマクロ

---

## デバイス番号の取得（2/2）


```
	printk(KERN_INFO "%s is loaded. major:%d\n",__FILE__,MAJOR(dev));
	return 0;
}

static void __exit cleanup_mod(void)
{
	unregister_chrdev_region(dev, 1);
	printk(KERN_INFO "%s is unloaded. major:%d\n",__FILE__,MAJOR(dev));
}
（以下略）
```

* `unregister_chrdev_region`: デバイス番号の解放
  * 引数: dev、マイナー番号を1個返す
  * これを怠るとinsmodのたびに番号が増えていくので実験すると面白い

---

## メジャー番号の確認

```bash
$ make
$ sudo insmod myled.ko
$ tail /var/log/messages
（略）
Oct 23 12:26:56 raspberrypi kernel: [  492.932021] /home/pi/myled_lecture/myled.c is loaded. major:243
$ cat /proc/devices | grep myled
243 myled
$ sudo rmmod myled
```

---

## キャラクタ型デバイスを作る（1/4）

* ヘッダに以下を追加
  * ヘッダファイル: `linux/cdev.h`のinclude
  * キャラクタデバイスの情報を格納する構造体`static struct cdev cdv`: 

```c
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
（中略）（MODULE_AUTHOR〜MODULE_VERSION）
static dev_t dev;
static struct cdev cdv;
（続く）
```

---

## キャラクタ型デバイスを作る（2/4）

* キャラクタデバイスの挙動の記述と登録
  * `led_write`: デバイスファイルに書き込みがあった時の挙動
  * `static struct file_operations led_fops`: 挙動を書いた関数のポインタを格納する構造体

```c
static ssize_t led_write(struct file* filp, const char* buf, size_t count, loff_t* pos)
{
        printk(KERN_INFO "led_write is called\n");
        return 1; //読み込んだ文字数を返す（この場合はダミーの1）
}

static struct file_operations led_fops = {
        .owner = THIS_MODULE,
        .write = led_write
};
（続く）
```

---

## キャラクタ型デバイスを作る（3/4）

* cdev_init: キャラクタデバイスの初期化
  * file_operations渡す
* cdev_add: キャラクタデバイスをカーネルに登録

```c
static int __init init_mod(void)
{
        （略。デバイス番号を取得する部分）
        printk(KERN_INFO "%s is loaded. major:%d\n",__FILE__,MAJOR(dev));

        cdev_init(&cdv, &led_fops);
        retval = cdev_add(&cdv, dev, 1);
        if(retval < 0){
                printk(KERN_ERR "cdev_add failed. major:%d, minor:%d",MAJOR(dev),MINOR(dev));
                return retval;
        }
（続く）
```

---

## キャラクタ型デバイスを作る（4/4）

* cdev_del:キャラクタデバイスの破棄

```c
        return 0;
}

static void __exit cleanup_mod(void)
{
        cdev_del(&cdv);
        unregister_chrdev_region(dev, 1);
        printk(KERN_INFO "%s is unloaded. major:%d\n",__FILE__,MAJOR(dev));
}
（以下略）
```

---

## 動作確認（1/2）

* `mknod c 243 0`でデバイスファイルを手動作成
  * `c`: キャラクタデバイス、243: メジャー番号（場合により変わる）、0: マイナー番号

```bash      
$ sudo insmod myled.ko
$ tail /var/log/messages
（略）
Oct 23 12:59:08 raspberrypi kernel: [ 2424.243775] /home/pi/myled_lecture/myled.c
 is loaded. major:243
$ sudo mknod /dev/myled0 c 243 0
$ sudo chmod 666 /dev/myled0 
$ ls -l /dev/myled0 
crw-rw-rw- 1 root root 243, 0 10月 23 13:00 /dev/myled0
```

---

## 動作確認（2/2）


* デバイスファイルに4文字（abcと改行記号）書き込む
  * ログに4回、led_writeに仕掛けたprintkの出力が残る

```bash
$ echo abc > /dev/myled0 
$ tail /var/log/messages
（略）
Oct 23 12:59:08 raspberrypi kernel: [ 2424.243775] /home/pi/myled_lecture/myled.c is loaded. major:243
Oct 23 13:03:08 raspberrypi kernel: [ 2664.510510] led_write is called
Oct 23 13:03:08 raspberrypi kernel: [ 2664.510533] led_write is called
Oct 23 13:03:08 raspberrypi kernel: [ 2664.510543] led_write is called
Oct 23 13:03:08 raspberrypi kernel: [ 2664.510551] led_write is called
$ sudo rm /dev/myled0    #後始末
$ sudo rmmod myled
```

---

## クラスの作成と削除（1/2）

* `/sys/class`下にこのデバイスの情報を置く
* `class_create`で作成
  * `THIS_MODULE`: このモジュールを管理する構造体のポインタ

```c
（略。ヘッダファイル）
#include <linux/device.h>    //追加
（中略）（MODULE_*とグローバル変数）
static struct class *cls = NULL;  //追加
（中略）
static int __init init_mod(void)
{      （中略）
        cls = class_create(THIS_MODULE,"myled");   //ここから追加
        if(IS_ERR(cls)){
                printk(KERN_ERR "class_create failed.");
                return PTR_ERR(cls);
        }
        return 0;
}
（続く）
```


---

## クラスの作成と削除（2/2）

* `class_destroy`で削除

```c
static void __exit cleanup_mod(void)
{
        cdev_del(&cdv);
        class_destroy(cls);  //追加
（以下略）
```

---

## <span style="text-transform:none">/sys下の確認

* `/sys/class`下に`myled`というディレクトリができる
  * まだ中身は空

```bash      
$ sudo insmod myled.ko
$ ls -l /sys/class/myled/       #ディレクトリができている
合計 0
```

---

## クラスへの情報書き込み（1/2）

* `device_create`: デバイス情報の作成
  * 引数（NULLのところ以外）: クラス、デバイス、デバイスの名前、 デバイスのマイナー番号

```c
static int __init init_mod(void)
{       （中略）
        cls = class_create(THIS_MODULE,"myled");
        if(IS_ERR(cls)){
                printk(KERN_ERR "class_create failed.");
                return PTR_ERR(cls);
        }
        device_create(cls, NULL, dev, NULL, "myled%d",MINOR(dev));
        return 0;
}
（続く）
```


---

## クラスへの情報書き込み（2/2）

* `device_destroy`: デバイス情報の削除

```c
static void __exit cleanup_mod(void)
{
        cdev_del(&cdv);
        device_destroy(cls, dev);
        class_destroy(cls);
（以下略）
```

---

## <span style="text-transform:none">/sys下の確認

```bash
$ ls -l /sys/class/myled/
合計 0
lrwxrwxrwx 1 root root 0 10月 23 13:40 myled0 -> ../../devices/virtual/myled/myled0
$ ls -l /sys/class/myled/myled0/
合計 0
-r--r--r-- 1 root root 4096 10月 23 13:40 dev
drwxr-xr-x 2 root root    0 10月 23 13:40 power
lrwxrwxrwx 1 root root    0 10月 23 13:40 subsystem -> ../../../../class/myled
-rw-r--r-- 1 root root 4096 10月 23 13:39 uevent
$ cd /sys/class/myled/myled0/
$ cat dev
243:0                #これがメジャー番号とマイナー番号
```

---

## <span style="text-transform:none">/dev下の確認

* udevというサービスが`/sys/class/myled/myled0/dev`を見てデバイスファイルを作成

```bash    
$ ls -l /dev/myled0 
crw------- 1 root root 243, 0 10月 23 13:39 /dev/myled0
$ sudo rmmod myled
$ ls -l /dev/myled0        #rmmodで自動的にデバイスファイルが消える
ls: /dev/myled0 にアクセスできません: そのようなファイルやディレクトリはありません
```

---

## デバイスファイルからの字の<br />読み込み

* ユーザランドからの字の書き込みをカーネルに読み込む
  * アドレス空間が違う
  * `copy_from_user`という関数を使用

```c
#include <linux/uaccess.h>    //ヘッダに追加

//led_writeを次のように書き換え
static ssize_t led_write(struct file* filp, const char* buf, size_t count, loff_t* pos)
{
 char c;   //読み込んだ字を入れる変数
 if(copy_from_user(&c,buf,sizeof(char)))
 return -EFAULT;

 printk(KERN_INFO "receive %c\n",c);
 return 1;
}
```

---

## 動作確認

（細かい手順はもう書きません）

```bash
$ echo abc > /dev/myled0 
$ tail /var/log/messages
（中略）
Oct 23 14:10:45 raspberrypi kernel: [ 1437.805316] /home/pi/robosys_device_drivers/myled.c is loaded. major:243
Oct 23 14:11:10 raspberrypi kernel: [ 1462.960887] receive a
Oct 23 14:11:10 raspberrypi kernel: [ 1462.960911] receive b
Oct 23 14:11:10 raspberrypi kernel: [ 1462.960920] receive c
Oct 23 14:11:10 raspberrypi kernel: [ 1462.960929] receive
```

---

## デバイスファイルからの出力

* catすると寿司を表示する無駄機能の実装
  * `copy_to_user`でカーネルからユーザランドへ文字を送ります

```c      
static ssize_t sushi_read(struct file* filp, char* buf, size_t count, loff_t* pos)
{
    int size = 0;
     char sushi[] = {0xF0,0x9F,0x8D,0xA3,0x0A}; //寿司の絵文字のバイナリ
     if(copy_to_user(buf+size,(const char *)sushi, sizeof(sushi))){
        printk( KERN_INFO "sushi : copy_to_user failed\n" );
     return -EFAULT;
     }
     size += sizeof(sushi);
    return size;
}

static struct file_operations led_fops = {
     .owner = THIS_MODULE,
     .write = led_write,
     .read = sushi_read
};
```

---

## GPIOの操作

* ピンの操作は特定のメモリアドレスへゼロイチを書き込むことで行う
  * メモリマップドI/O
  * 番地をどう調べるか（なかなか難しい問題）
* Raspberry Piの場合
  * Raspberry Piのページのbcm2835に関するページで調査
    * 「Peripheral specification」をクリック
  * Raspberry Pi 3のbcm2837とはレジスタのアドレスの開始位置が違うので読み替え

---

## レジスタアドレスのリマップ

* GPIOのレジスタを配列にマッピング
  * マッピング: アドレスとアドレスを対応付けること
  * 0x3f200000: GPIOのレジスタの最初のアドレス
    * bcm2835のpdfには書いてない
    * このアドレスの後のレジスタの配置は同じ（Peripheral specification90,91ページ）
  * 0xA0: 必要なアドレスの範囲（91ページを読んで設定）

```c
...
#include <linux/io.h>　　　　　　　　　 //ヘッダファイルを追加
...
static volatile u32 *gpio_base = NULL;  //アドレスをマッピングするための配列をグローバルで定義
...
static int __init init_mod(void)
{
 int retval;

 gpio_base = ioremap_nocache(0x3f200000, 0xA0); //追加
...
```

---

## GPIOピンを出力にする

* ピンを出力にするレジスタに1を書き込む
  * ほかのレジスタの値はそのままにしないといけない
* Peripheral specificationのp. 90-92あたり
  * GPIO25の機能はGPFSEL2にある
  * GPFSEL2の17,16,15番目のビットを001に設定

```c
static int __init init_mod(void)
{
    int retval;

    gpio_base = ioremap_nocache(0x3f200000, 0xA0); //0x3f..:base address, 0xA0: region to map

    const u32 led = 25;
    const u32 index = led/10;//GPFSEL2
    const u32 shift = (led%10)*3;//15bit
    const u32 mask = ~(0x7 << shift);//11111111111111000111111111111111
    gpio_base[index] = (gpio_base[index] & mask) | (0x1 << shift);//001: output flag
    //11111111111111001111111111111111
...
```

---

## ON/OFFの書き込み

p. 90, 95
ON: 「GPFSET0」のGPIO25に対応するところに1を書き込む
OFF: 「GPCLR0」のGPIO25に対応するところに1を書き込む

```c
static ssize_t led_write(struct file* filp, const char* buf, size_t count, loff_t* pos)
{
    char c;
    if(copy_from_user(&c,buf,sizeof(char)))
        return -EFAULT;

    if(c == '0')
        gpio_base[10] = 1 << 25;
    else if(c == '1')
        gpio_base[7] = 1 << 25;

    return 1;
}
```

---

## 補足

* 「ただLEDを光らせるだけでこんなに苦労しなければならないのか」と質問をいただいたので・・・
* そんなことはない
  * `/sys/class/gpio`の操作
  * WiringPi
* デバドラを書いた意味
  * 機械からユーザランドまでの仕組みの一部を理解した
  * デバイスドライバで利用できる他の機能の学習の機会を得た
    * タイマー
    * メモリをもっと使った処理を含むデバイスドライバ

