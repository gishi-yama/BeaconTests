# Evo Demo Apps の動作手順

## はじめに

ここでは、Cordova の環境を準備し、iBeaconのサンプルアプリである [Evo Demo Apps](https://github.com/divineprog/evo-demos) の[cordova-ibeacon](https://github.com/divineprog/evo-demos/tree/master/Demos2015/cordova-ibeacon)をAndroidで動作させるまでの手順を記載する。

## Mac

### git と node.js と npm をインストール

ターミナルで [Homebrew](http://brew.sh/) を使ってインストールする:

```
brew install git node npm
```

### cordova 環境をインストール

ターミナルで npm を使ってインストールする:

```
sudo npm install -g cordova
```

### Android studio のインストール


[Android studio](https://developer.android.com/sdk/index.html) からダウンロード。

デフォルト設定のままインストールする。

#### クラスパスの設定

Android SDKのクラスパスを通す:

```bash
vim ~/.bash_profile
```

下の内容を書き込んで保存する:

```
export PATH=${PATH}:${HOME}/Library/Android/sdk/tools
export PATH=${PATH}:${HOME}/Library/Android/sdk/platform-tools
```

クラスパス再読み込み:

```
source ~/.bash_profile
```


#### Android SDKの追加

Beaconに対応している4.3以上のSDKを追加インストールする。ここでは、テスト端末をAndroid 4.4 端末として想定し、 Android SDK 4.4.2（API19）と、Cordova5の必須環境として Android SDK 5.1.1（API22）をインストールする。

SDKのインストール画面を起動する:

```
android
```

SDKのインストール画面で、次のものを追加インストールする。API22は最新版のCordovaで必要。

- Android 6.0（API23）
  - Intel x86 Atom System Image
- Android 5.1.1（API22）
  - SDK Platform
  - Intel x86 Atom System Image
  - Google APIs
  - Google APIs (X86 System Image）
  - Sources for Android SDK
- Android 4.4.2（API19）
  - SDK Platform
  - Intel x86 Atom System Image
  - Google APIs (X86 System Image）
  - Sources for Android SDK
- Extras
  - Android Support Library
  - Intel x86 Emulator Accelerator（HAXM installer）

結構時間がかかる。


### Evo Demo Appsのダウンロード

[Evo Demo Apps](https://github.com/divineprog/evo-demos) をzipダウンロードして、展開する:

```
wget -O evo-demos-master.zip https://github.com/divineprog/evo-demos/archive/master.zip
unzip evo-demos-master.zip
cd evo-demos-master/Demos2015
```
フォルダ内にある既存の cordova-ieacon フォルダを移動しておく。

```
mv cordova-ibeacon cordova-ibeacon-backup
```

### Cordova プロジェクトの作成

まず、cordova コマンドで作成する:

```
cordova create cordova-ibeacon com.evothings.cordovaibeacon iBeacon
```

これにより、config.xmlのid（パッケージ名）が com.evothings.cordovaibeacon に、name（プロジェクト名）が iBeacon に設定される。

### Cordova プロジェクトの編集

#### config.xmlの設定

config.xmlを書き換える:

```
cd cordova-ibeacon
vim config.xml
```

`</widget>` の前に下のxmlタグを追加して保存する:

```xml
<preference name="DisallowOverscroll" value="true" />
```

これにより、アプリ起動時に画面外へのスクロールが抑制される。

#### www フォルダの入れ替え

cordova-ibeacon フォルダの www フォルダを削除し、cordova-ibeacon-backup フォルダから www フォルダをコピーする:

```
rm -Rf www
cp -Rf ../cordova-ibeacon-backup/www .
```

#### beaconの設定

www フォルダの app.js を書き換える:

```
vim www/app.js
```

L34〜L45のbeacon設定のうち、UUID, major, minor を書き換える。なお、[ドキュメントにあるとおり](https://github.com/divineprog/evo-demos/tree/master/Demos2015/cordova-ibeacon#quick-overview-of-how-ibeacon-ranging-and-monitoring-works)UUIDだけの記載でも動作する。

検知するbeaconが１つであれば、2つある要素のうちを１つだけを有効にしてもよい。複数あれば、増やしても良い。


#### プラグインの追加

cordova-ibeacon フォルダに ibeacon と local-notifications プラグインを追加する:

```
cordova plugin add https://github.com/petermetz/cordova-plugin-ibeacon
cordova plugin add https://github.com/katzer/cordova-plugin-local-notifications
```

#### プラットフォームの設定

cordova projectを各プラットフォーム用にコンパイルできる様にする:

```
cordova platform add android
```

Android の場合、Targetが最新版のSDKになっているので、4.4.2（API19）に修正する:

```
vim platforms/android/AndroidManifest.xml
```

`<uses-sdk>` タグの内容（SDKVersion）をAPI19想定で書き換える:

```xml
<uses-sdk android:minSdkVersion="19" android:targetSdkVersion="19" />
```

```
vim platforms/android/project.properties
```

`target`の値をAPI19想定で書き換える:

```properties
target=android-19
```

### Cordova プロジェクトのビルド

```
cordova build android
```

`platforms/android/build/outputs/apk/android-debug.apk` にビルドされる。

### プラットフォームへのインストール（Android）

#### USBでのインストールがうまくいかない端末の場合

:warning: **この設定はテスト用端末でのみ実施するか、操作完了後すぐに元に戻すこと** :warning:

設定＞ユーザ設定＞セキュリティ＞デバイス管理＞提供不明元のアプリ を許可する

**インストール手順**

1. ビルドされた `platforms/android/build/outputs/apk/android-debug.apk` をDropbox
などでリンク公開する
2. Android端末のブラウザで android-debug.apk をダウンロードする
3. ダウンロードした android-debug.apk をインストールする
4. アプリを起動し、iBeacon端末との接続を確認する。

## 備考

### ibeaconのテスト

node.js の bleacon を使うと、iBeaconの発信・受信テストが簡単（要対応端末）

```
npm install bleacon
```

以下のドキュメントを参考に、発信・受信用のjsを作成する。

- [たった6行!最も簡単にiBeaconの電波を「発信」する方法](http://qiita.com/Morikuma_Works/items/a0dd3cfcd1eef8dbd492)
- [たった5行!最も簡単にiBeaconの電波を「受信」する方法](http://qiita.com/Morikuma_Works/items/c2899e548da1c5e2c28e)

仮に push.js, recieve.js として作成した場合、 `node push.js` もしくは `node recieve.js` で起動できる。
