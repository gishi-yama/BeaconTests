Evo Demo Apps の動作手順
======

## 1. はじめに

ここでは、Cordova の環境を準備し、iBeaconのサンプルアプリである [Evo Demo Apps](https://github.com/divineprog/evo-demos) の[cordova-ibeacon](https://github.com/divineprog/evo-demos/tree/master/Demos2015/cordova-ibeacon)をAndroid, iOSで動作させるまでの手順を記載する。

## 2. 環境設定：OSX

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

## 3. Evo Demo Appsのダウンロード

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

## 4. Cordova プロジェクトの準備

### プロジェクトの作成

まず、cordova コマンドで作成する:

```
cordova create cordova-ibeacon com.evothings.cordovaibeacon iBeacon
```

これにより、config.xmlのid（パッケージ名）が com.evothings.cordovaibeacon に、name（プロジェクト名）が iBeacon に設定される。

### config.xmlの設定

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

### www フォルダの入れ替え

cordova-ibeacon フォルダの www フォルダを削除し、cordova-ibeacon-backup フォルダから www フォルダをコピーする:

```
rm -Rf www
cp -Rf ../cordova-ibeacon-backup/www .
```

### beaconの設定

www フォルダの app.js を書き換える:

```
vim www/app.js
```

L34〜L45のbeacon設定のうち、UUID, major, minor を書き換える。なお、[ドキュメントにあるとおり](https://github.com/divineprog/evo-demos/tree/master/Demos2015/cordova-ibeacon#quick-overview-of-how-ibeacon-ranging-and-monitoring-works)UUIDだけの記載でも動作する。

検知するbeaconが１つであれば、2つある要素のうちを１つだけを有効にしてもよい。複数あれば、増やしても良い。


### プラグインの追加

cordova-ibeacon フォルダに ibeacon と local-notifications プラグインを追加する:

```
cordova plugin add https://github.com/petermetz/cordova-plugin-ibeacon
cordova plugin add https://github.com/katzer/cordova-plugin-local-notifications
```

## 5. プラットフォーム設定とテスト：Android

### Android studio のインストール


[Android studio](https://developer.android.com/sdk/index.html) からダウンロード。

デフォルト設定のままインストールする。

### クラスパスの設定

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

### Android SDKの追加

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

### CordovaのAndroidプラットフォームの追加と設定

cordova projectをAndroidプラットフォーム用にコンパイルできる様にする:

```
cordova platform add android
```

Android の場合、TargetがAndroid-14〜22になっているので、beaconが対応するAndroid-19〜22に修正する。

```
vim platforms/android/AndroidManifest.xml
```

`<uses-sdk>` タグの内容（SDKVersion）をAPI19想定で書き換える:

```xml
<uses-sdk android:minSdkVersion="19" android:targetSdkVersion="22" />
```

### Cordova プロジェクトのビルド

```
cordova build android
```

`platforms/android/build/outputs/apk/android-debug.apk` にビルドされる。

### 動作テスト

#### 端末の準備

:warning: **この設定はテスト用端末でのみ実施するか、動作テスト完了後すぐに元に戻すこと** :warning:

1. 設定＞ユーザ設定＞セキュリティ＞デバイス管理＞提供不明元のアプリ を許可する
2. 設定＞システム＞開発者向けオプション＞USB デバッグ を許可する。
3. 端末をUSBでPCと接続する。
  - Windows PCの場合、ドライバのインストールが必要？
4. USBデバッグを許可するか？という画面が表示されたら、許可する。

#### テストの開始

```
cordova build android
cordova run android --device
```

実機にアプリが表示され、操作できる。

UUIDなどが一致するiBeacon端末の電源が入っていれば、接続が確認できる。

#### USB経由でのテストがうまくいかない端末の場合

1. ビルドされた `platforms/android/build/outputs/apk/android-debug.apk` をDropbox
などでリンク公開する
2. Android端末のブラウザで android-debug.apk をダウンロードする
3. ダウンロードした android-debug.apk をインストールする
4. アプリを起動し、UUIDなどが一致するiBeacon端末との接続を確認する。


## 6. プラットフォーム設定とテスト：iOS

### プラットフォームの追加と設定

cordova projectをiOSプラットフォーム用にコンパイルできる様にする:

```
cordova platform add ios
```

### Cordova プロジェクトのビルド

```
cordova build ios
```

`platforms/ios/build/emulator/iBeacon.app` にビルドされる。


### 動作テスト

#### 端末の準備

Xcode（本稿時点で7.0.1）を起動する。

Apple IDが登録されていなければ、Preference > Accounts で登録する。

Cordovaアプリを実機テストするには、Apple IDにApp ID（Bundle Identifer）が割り振られていなければいけない（本来は、Apple Developer Programの有料プログラムで登録する）。

Xcode 7からは、Apple IDで作成したことのある Bundle Identifer であれば（期限つきで？）実機テストを行える。これを利用するため、ダミーアプリを作成して Bundle Identifer を登録する。

File > New > Project で、iOS > Application > Single View Application　を選択して作成し、次の値を入力したプロジェクトを保存する。

- Product Name : iBeacon
- Organization Identifer : com.evothings.cordova

これにより、Bundle Identifer : com.evothings.cordova.ibeacon が作成される。

FIle > 開く で、`platforms/ios/iBeacon.xcodeproj`　を開く。

iBeaconプロジェクト選択すると、Bundle Identifer が `com.evothings.cordovaibeacon` になっているので、ピリオドを追加して `com.evothings.cordova.ibeacon` に修正する。

Team は自分のApple IDを選択する。

Deployment Target は 7.1 以上を選択する。

エラーが表示されていないことを確認して、USBでiOS端末を接続する。

画面上部のデバイスリストから、接続したiOS端末を選択する。（`iBeacon > デバイス名` になってればOK）

#### テストの開始

Xcodeの再生（ビルド＆テスト）ボタンを押す。

もし、`Could not launch "iBeacon" process .... faild : security` というダイアログが表示されたら、iOS端末の設定が必要。

1. 設定＞一般＞プロファイル
2. デベロッパAPP に AppleIDのプロファイルが表示されている
3. 選択して、 "Apple ID" を信頼をタップ
4. "信頼" をタップ
5. アプリを起動する

実機にアプリが表示され、操作できる。

UUIDなどが一致するiBeacon端末の電源が入っていれば、接続が確認できる。

:warning: **不要になったら、プロファイルは削除しておくこと。** :warning:


## 7. 備考

### ibeaconのテスト

node.js の bleacon を使うと、iBeaconの発信・受信テストが簡単（要対応端末）

```
npm install bleacon
```

以下のドキュメントを参考に、発信・受信用のjsを作成する。

- [たった6行!最も簡単にiBeaconの電波を「発信」する方法](http://qiita.com/Morikuma_Works/items/a0dd3cfcd1eef8dbd492)
- [たった5行!最も簡単にiBeaconの電波を「受信」する方法](http://qiita.com/Morikuma_Works/items/c2899e548da1c5e2c28e)

仮に push.js, recieve.js として作成した場合、 `node push.js` もしくは `node recieve.js` で起動できる。
