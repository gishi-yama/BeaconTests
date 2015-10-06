Evothings Studio Example の動作手順
=====

## 1. はじめに

[Evo Demo Apps の動作手順](EvoDemoApps.md)で紹介したEvo Demo Appsは、[Evo Thing Studio](https://evothings.com/) のJSライブラリを利用している。
この Evo Thing Studio のexampleに入っている [iBeacon-Scan](https://evothings.com/doc/examples/ibeacon-scan.html) の[プログラムが、よりシンプルでわかりやすい](https://github.com/evothings/evothings-examples/blob/master/examples/ibeacon-scan/app.js)ので、動作させてみる。

[Evo Demo Apps の動作手順](EvoDemoApps.md) とは、手順3,4のみ若干異なるので、以下ではその差し替え部分だけを記載している。


## 3. Evo Thing Studio のダウンロード

[Evo Thing Studio](https://evothings.com/download/) をzipダウンロードして、展開する:

```
wget -O EvothingsStudio.zip  https://s3-eu-west-1.amazonaws.com/evothings-download/EvothingsStudio_Mac_64_1.2.0.zip
unzip EvothingsStudio.zip
```

Windowsの時にダウンロードするのは、MacではなくWinでも良い（本稿に必要なファイルの中身は変わらないはず）


## 4. Cordova プロジェクトの準備

### プロジェクトの作成

まず、cordova コマンドで作成する。今回は、EvothingsStudio フォルダの外で作成する:

```
cordova create ibeacon-scan com.evothings.cordovaibeacon iBeacon-scan
```

### config.xmlの設定

config.xmlを書き換える:

```
cd ibeacon-scan
vim config.xml
```

`</widget>` の前に下のxmlタグを追加して保存する:

```xml
<preference name="DisallowOverscroll" value="true" />
```

これにより、アプリ起動時に画面外へのスクロールが抑制される。

### www フォルダの入れ替え

cordova-ibeacon/www フォルダの中身を削除し、ダウンロードして展開していた EvothingsStudio/examples/ibeacon-scan フォルダの中身をコピーする:

```
rm -Rf www/*
cp -Rf ../EvothingsStudio/examples/ibeacon-scan/* .
```

### beaconの設定

www フォルダの app.js を書き換える:

```
vim www/app.js
```

L7からの `var regions` の配列内の定義に、認識させたいデバイスのUUIDを追記する。


### プラグインの追加

cordova-ibeacon フォルダに ibeacon  プラグインのみを追加する:

```
cordova plugin add https://github.com/petermetz/cordova-plugin-ibeacon
```

以降は [Evo Demo Apps の動作手順](EvoDemoApps.md) と同様の手順で、プラットフォームで動作させる。

## その他

app.js の後半、displayBeaconList関数の中身を書き換えることで、距離を表示できるようにする。

```js
var element = $(
'<li>'
  +  '<strong>UUID: ' + beacon.uuid + '</strong><br />'
  +  'Major: ' + beacon.major + '<br />'
  +  'Minor: ' + beacon.minor + '<br />'
  +  'Proximity: ' + beacon.proximity + '<br />'
  +  'Distance: ' + beacon.accuracy + '<br />'
  +  'RSSI: ' + beacon.rssi + '<br />'
  +  '<div style="background:rgb(255,128,64);height:20px;width:'
  +   rssiWidth + '%;"></div>'
+ '</li>'
);
```
