raspi-doorlock
====

## 必要なもの
+ Raspberry pi Zero WH https://www.switch-science.com/catalog/3646/
+ microSDカード（32GBまで） https://www.dospara.co.jp/5shopping/detail_parts.php?bg=7&br=219&sbr=1043&ic=369719&lf=0
+ 変換名人 USBMCH-20LL（100均でも買えます） https://www.yodobashi.com/product/100000001003247877/
+ NFCリーダ Sony RC-S380 https://kakaku.com/item/K0000426530/
+ microUSBケーブル（100均のでもOK） http://akizukidenshi.com/catalog/g/gC-09314/
+ 360°サーボ http://akizukidenshi.com/catalog/g/gM-01723/

## Raspbianのダウンロード
http://ftp.jaist.ac.jp/pub/raspberrypi/raspbian_lite/images/
から最新のliteイメージをダウンロード（2019/10/3現在最新2019-09-26-raspbian-buster-lite.zip）

## イメージの書き込み
[Rufus](https://rufus.ie/)を利用してダウンロードしたzipファイルを書き込み


## シリアルコンソールの有効化
書き込んだbootドライブのconfig.txtの最終行に追記
`dtoverlay=pi3-miniuart-bt`

参考：https://www.usagi1975.com/201907061439/

## シリアル接続
USBシリアルモジュールのTX、RX、GNDを写真のようにpi0wと接続する。
シリアルモジュールのTXとpi0wのRX（10ピン）、シリアルモジュールのRXとpi0wのTX（8ピン）を接続することに注意。

![シリアル接続](https://github.com/iwax2/raspi-fiap/blob/master/pi0w-serial.jpg "pi0w-serial")

TeraTermなどでシリアル接続、シリアル設定のうちボーレートを115200に設定する。

## pi0wの設定
電源を入れて、pi / raspberryでログインする。

~~~
pi@raspberrypi:~$ sudo raspi-config
> 1 Change User Password Change password for the current user
> 2 Network Options      Configure network settings
> N2 Wi-fi                   Enter SSID and passphrase
> JP Japan
> Ok
> Please enter SSID -> WiFi アクセスポイントのSSIDを入力
> Please enter passphrase. Leave it empty if none. -> パスワードを入力
> 4 Localisation Options Set up language and regional settings to match your
> I2 Change Timezone        Set up timezone to match your location
> Asia
> Tokyo
> Finish
pi@raspberrypi:~$ ip a
2: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:27:eb:d8:02:ea brd ff:ff:ff:ff:ff:ff
    inet 192.168.7.189/24 brd 192.168.7.255 scope global noprefixroute wlan0
pi@raspberrypi:~$ sudo apt update
pi@raspberrypi:~$ sudo apt upgrade -y
pi@raspberrypi:~$ sudo reboot
pi@raspberrypi:~$ sudo apt install -y git python3-pip build-essential vim
~~~

## Node-REDの設定
https://nodered.jp/docs/getting-started/raspberrypi を参考に

~~~
pi@raspberrypi:~$ bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
Are you really sure you want to do this ? [y/N] ? y
Would you like to install the Pi-specific nodes ? [y/N] ? y
12分かかった
pi@raspberrypi:~$ sudo systemctl enable nodered
pi@raspberrypi:~$ sudo systemctl start nodered
~~~

http://x.x.x.x:1880/ にアクセスして、三本線のパレット管理からnode-red-dashboardのノードを追加

![Node-RED Webドアロック](https://github.com/iwax2/raspi-doorlock/blob/master/nodered-web.png "nodered-webver")

これでWebブラウザ経由で鍵の開け閉めはできる．

## nfcpy
FeLiCaでドア開けたいよね．開けたくない？
python2系列で動かす悪夢

参考：https://qiita.com/mascii/items/ec79ad5a7026f771d181

![NFC接続](https://github.com/iwax2/raspi-doorlock/blob/master/pi0w-nfc.jpg "pi0w-nfc")

~~~
pi@raspberrypi:~$ sudo apt install -y python-pip python-dev libusb-dev python-usb
pi@raspberrypi:~$ sudo pip install -U nfcpy nfcpy-id-reader
pi@raspberrypi:~$ lsusb
pi@raspberrypi:~$ python -m nfc
pi@raspberrypi:~$ sudo sh -c 'echo SUBSYSTEM==\"usb\", ACTION==\"add\", ATTRS{idVendor}==\"054c\", ATTRS{idProduct}==\"06c3\", GROUP=\"plugdev\" >> pi@raspberrypi:~$ /etc/udev/rules.d/nfcdev.rules'
sudo udevadm control -R # then re-attach device
pi@raspberrypi:~$ sudo reboot
pi@raspberrypi:~$ python -m nfc
~~~

node-red-contrib-nfcpy-idノードを追加して，
入力グループのnfcpy idノードを追加してプログラミング

なんかXperia XZ3のIDmが08000000しか見れなくてむーりー

~~~
pi@raspberrypi:~$ sudo pip3 install -U nfcpy ndef
pi@raspberrypi:~$ python3 -m nfc
pi@raspberrypi:~$ git clone https://github.com/nfcpy/nfcpy.git
pi@raspberrypi:~$ cd nfcpy/examples
pi@raspberrypi:~/nfcpy/examples $ python3 tagtool.py



pi@raspberrypi:~$ 
pi@raspberrypi:~$ 
pi@raspberrypi:~$ 
~~~

