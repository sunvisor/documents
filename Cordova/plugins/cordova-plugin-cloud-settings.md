Cordova Cloud Settings plugin [![Latest Stable Version](https://img.shields.io/npm/v/cordova-plugin-cloud-settings.svg)](https://www.npmjs.com/package/cordova-plugin-cloud-settings)
=====================================================================================================================

> [オリジナル: Cordova Cloud Settings plugin](https://www.npmjs.com/package/cordova-plugin-cloud-settings)

Android & iOS用のCordovaプラグインで、デバイスやインストールをまたいでクラウドストレージにユーザー設定を永続化することができます。

<!-- doctoc README.md --maxlevel=2 -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**目次**

- [概要](#概要)
  - [Android](#android)
  - [iOS](#ios)
- [インストール](#インストール)
  - [プラグインのインストール](#プラグインのインストール)
  - [リモートビルド環境](#リモートビルド環境)
- [利用ライフサイクル](#利用ライフサイクル)
- [API](#api)
  - [`exists()`](#exists)
  - [`save()`](#save)
  - [`load()`](#load)
  - [`onRestore()`](#onrestore)
  - [`enableDebug()`](#enabledebug)
- [テストをする](#testing)
  - [Android でテストする](#android-でテストする)
  - [iOS でテストする](#ios-でテストする)
- [サンプルプロジェクト](#サンプルプロジェクト)
- [GDPR に準拠したアナリティクスでの使用](#gdpr-に準拠したアナリティクスでの使用)
  - [GDPR の背景](#gdpr-の背景)
  - [アナリティクスにおけるユーザー追跡への影響](#アナリティクスにおけるユーザー追跡への影響)
  - [このプラグインを使うメリット](#benefits-of-using-this-plugin)
- [Authors](#authors)
- [Licence](#licence)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 概要

このプラグインは、ユーザーがアプリを再インストールしたり、別のデバイスにインストールしたりしても、設定は復元され、新しいインストールで利用できるように、JSON構造の形でキー/バリューのアプリ設定をクラウドストレージに保存する仕組みを提供します。

主な機能:

- 設定は、プラットフォームが提供する無料のネイティブクラウドストレージソリューションを使用して保存されます。
    - 追加のSDKを必要とせず、すぐに使えるクラウドストレージです。
- ユーザー認証が不要なので、ユーザーにログインを求めることなく、すぐに設定を読み込んで保存することができます。
    - このため、このプラグインは[GDPR対応クロスインストールユーザートラッキング](#use-in-GDPR-compliant-analytics) に便利です。
- そのため、保存された設定はアプリの起動時にすぐに利用できるようになります（ユーザーのログインは必要ありません）。

Note:
- Android と iOS の間で設定を共有することはできません。

## Android

プラグインでは、[Androidのデータバックアップサービス](http://developer.android.com/guide/topics/data/backup.html) を利用して、設定をファイルに保存しています。

- Android v2.2.2 (APIレベル8)以上をサポートしています。
- Android 6 以上では、[Auto Backup](http://androiddoc.qiniudn.com/training/backup/autosyncapi.html) の仕組みを使用しています。
- On Android 5 and below, the [manual Backup API](http://androiddoc.qiniudn.com/training/backup/backupapi.html) is used.
    - APIキーを取得するためには、Google Data Backup を利用するための[アプリ登録](https://developer.android.com/google/backup/signup.html?csw=1) が必要です。

## iOS

プラグインは設定を保存するために [iCloud](https://support.apple.com/en-gb/HT207428) を使用しており、特にネイティブの Key/Value ストアに設定を保存するために [NSUbiquitousKeyValueStoreクラス](https://developer.apple.com/documentation/foundation/nsubiquitouskeyvaluestore) を使用しています。

Note:
 - iOS v5.0以上をサポート
 - 利用可能なストレージ容量は、アプリごとに1ユーザーあたり1MBとなっています。
 - [Apple Member Centre](https://developer.apple.com/membercenter/index.action) で App Identifier の iCloud を有効にして、Xcode プロジェクトで iCloud 機能を有効にする必要があります。
    - Xcode で App ID に iCloud を追加して iCloud 機能を有効化する方法は [こちら](https://theblog.github.io/post/swift-icloud-key-value-store/) を参照してください。


# インストール

プラグインは [cordova-plugin-cloud-settings](https://www.npmjs.org/package/cordova-plugin-cloud-settings) として npm に公開されています。

## プラグインのインストール

```sh
cordova plugin add cordova-plugin-cloud-settings --variable ANDROID_BACKUP_SERVICE_KEY="<API_KEY>"
```

## リモートビルド環境

iOS プラットフォームでは、Cordova で生成された XCode プロジェクトを手動で編集するか、本プラグインの[フックスクリプト](https://cordova.apache.org/docs/en/latest/guide/appdev/hooks/) を削除する必要があります。
そのため、このプラグインは iOS プラットフォームのリモート ("クラウド") ビルド環境をサポートしていない環境では**動作しません**。

- [Phonegap Build](https://github.com/phonegap/build/issues/425)
- [Intel XDK](https://software.intel.com/en-us/xdk/docs/add-manage-project-plugins)
- [Telerik Appbuilder](http://docs.telerik.com/platform/appbuilder/cordova/using-plugins/using-custom-plugins/using-custom-plugins)
- [Ionic Cloud](https://docs.ionic.io/services/package/#hooks)

他のクラウドベースのCordova/Phonegapビルドサービスを使用していて、このプラグインが動作しないことに気付いた場合、その原因もおそらく同じです。

# 利用ライフサイクル

典型的なライフサイクルは以下の通りです。
 - ユーザーが初めてアプリをインストールする
 - アプリが起動して `exists()` を呼び出し、既存の設定がないことを確認する
 - ユーザがアプリを使用して設定を生成する: アプリが `save()` を呼び出して設定をクラウドにバックアップする
 - さらなる使用、さらなるバックアップ...
 - ユーザーが新しいデバイスにアプリをダウンロード
 - アプリが起動して `exists()` を呼び出し、既存の設定があることを確認する
 - アプリが `load()` を呼び出して既存の設定にアクセスする
 - ユーザーは途中で中断したところから続行

# API

プラグインのJS APIはグローバル名前空間 `cordova.plugin.cloudsettings` の下にあります。

## `exists()`

`cordova.plugin.cloudsettings.exists(successCallback);`

現在のユーザーに保存されているクラウド設定が存在するかどうかを示します。

### パラメータ
- {function} successCallback - コールバック関数。
ユーザ用のストアの設定が存在するかどうかを示す boolean フラグが渡されます。

```javascript
cordova.plugin.cloudsettings.exists(function(exists){
    if(exists){
        console.log("Saved settings exist for the current user");
    }else{
        console.log("Saved settings do not exist for the current user");
    }
});
```

## `save()`

`cordova.plugin.cloudsettings.save(settings, [successCallback], [errorCallback], [overwrite]);`

設定をクラウドバックアップに保存します。

Notes:
- ネストした JSON 構造を渡すことができ、プラグインはそれを既存のものに保存/マージします。
    - しかし、構造が [stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) (つまり、関数や循環参照などがない) になることを確認してください。
- クラウドへのバックアップは、Android や iOS ではすぐには起こらないかもしれませんが、次のスケジュールされたバックアップが行われたときに発生します。

### パラメータ
- {object} settings - クラウドバックアップに保存するユーザー設定を表すJSON構造体。
- {function} successCallback - (optional) 設定の保存やバックアップのスケジューリングが成功した際に呼び出すコールバック関数です。保存された設定を JSON オブジェクトとして含む一つのオブジェクト引数が渡されます。
- {function} errorCallback - (optional) 設定の保存やバックアップのスケジュールに失敗した場合に呼び出すコールバック関数。エラーの説明を含む一つの文字列引数を渡します。
- {boolean} overwrite - (optional) true の場合、既存の設定は更新されずに置き換えられます。デフォルトは false です。
    - false の場合、既存の設定はこの関数に渡された新しい設定とマージされます。

```javascript
var settings = {
  user: {
    id: 1678,
    name: 'Fred',
    preferences: {
      mute: true,
      locale: 'en_GB'
    }
  }
}

cordova.plugin.cloudsettings.save(settings, function(savedSettings){
    console.log("Settings successfully saved at " + (new Date(savedSettings.timestamp)).toISOString());
}, function(error){
    console.error("Failed to save settings: " + error);
}, false);
```

## `load()`

`cordova.plugin.cloudsettings.load(successCallback, [errorCallback]);`

現在の設定をロードします。

Note: 設定は、クラウドバックアップから直接ではなくローカルのディスクから読み込まれるため、クラウドバックアップの最新の設定であることを保証するものではありません。
ローカルとクラウドの設定の競合を解決する必要がある場合は、設定をロード/保存する際にタイムスタンプを保存し、競合を解決するために使用してください。

### パラメータ
- {function} successCallback - (optional) 設定の読み込みに成功した際に呼び出すコールバック関数です。引数には、現在の設定を JSON オブジェクトとして含む一つのオブジェクトを渡します。
- {function} errorCallback - (optional) 設定の読み込みに失敗した場合に呼び出すコールバック関数です。エラーの説明を含む一つの文字列を引数に渡します。

```javascript
cordova.plugin.cloudsettings.load(function(settings){
    console.log("Successfully loaded settings: " + console.log(JSON.stringify(settings)));
}, function(error){
    console.error("Failed to load settings: " + error);
});
```

## `onRestore()`

`cordova.plugin.cloudsettings.onRestore(successCallback);`

クラウドから端末の設定が更新された場合に呼び出される機能を登録します。

これは、アプリの実行中にクラウドから更新されたために現在の設定が変更された場合に、アプリに通知することを目的としています。
これは例えば、ユーザーがアプリをインストールした2つのデバイスを持っている場合に発生する可能性があり、一方のデバイスの設定を変更すると、もう一方のデバイスに同期されます。

`save()` を呼び出すと、`timestamp` キーが保存された設定オブジェクトに追加され、設定が保存された日時を示します。
必要に応じて、これを使ってコンフリクトを解決することができます。

### パラメータ
- {function} successCallback - クラウドからデバイスの設定が更新された際に呼び出すコールバック関数。

```javascript
cordova.plugin.cloudsettings.onRestore(function(){
    console.log("Settings have been updated from the cloud");
});
```

## `enableDebug()`

`cordova.plugin.cloudsettings.enableDebug(successCallback);`

ネイティブプラグインコンポーネントから JS コンソールに詳細なログメッセージを出力します。

### パラメータ
- {function} successCallback - デバッグモードが有効になったときに呼び出すコールバック関数

```javascript
cordova.plugin.cloudsettings.enableDebug(function(){
    console.log("Debug mode enabled");
});
```

# テストをする

## Android でテストする

### バックアップのテスト
設定のバックアップをテストするには、手動でバックアップマネージャーを起動して（[Androidドキュメント](https://developer.android.com/guide/topics/data/testingbackup) の指示に従って）、更新された値を強制的にバックアップする必要があります。

まず、バックアップマネージャーが有効になっていて、詳細なロギングができるように設定されていることを確認してください。

```bash
    $ adb shell bmgr enabled
    $ adb shell setprop log.tag.GmsBackupTransport VERBOSE
    $ adb shell setprop log.tag.BackupXmlParserLogging VERBOSE
```

バックアップをテストする方法は、ターゲットデバイスで実行しているAndroidのバージョンに依存します。

#### Android 7 以降

以下のコマンドを実行してバックアップを実行します。

```bash
    $ adb shell bmgr backupnow <APP_PACKAGE_ID>
```

#### Android 6

* 以下のコマンドを実行してください。
```bash
    $ adb shell bmgr backup @pm@ && adb shell bmgr run
```

* 前のステップのコマンドが終了するまで、`adb logcat` を監視して以下の出力を確認してください。
```
    I/BackupManagerService: Backup pass finished.
```

* 以下のコマンドを実行してバックアップを実行します。
```bash
    $ adb shell bmgr fullbackup <APP_PACKAGE_ID>
```


#### Android 5 以前
以下のコマンドを実行してバックアップを実行します。

```bash
    $ adb shell bmgr backup <APP_PACKAGE_ID>
    $ adb shell bmgr run
```

### restore のテスト

手動でレストアを開始するには、次のコマンドを実行します。
```bash
    $ adb shell bmgr restore <TOKEN> <APP_PACKAGE_ID>
```

* バックアップトークンを調べるには、`adb shell dumpsys backup`を実行します。
* トークンはラベル `Ancestral:` と `Current:` の後に続く16進数の文字列です。
    * ancestral トークンは、デバイスを最初にセットアップしたとき (デバイスセットアップウィザードを使用したとき) に、デバイスのリストアに使用したバックアップデータセットを参照します。
    * current トークンは、デバイスの現在のバックアップデータセット（デバイスが現在バックアップデータを送信しているデータセット）を参照します。
* 正規表現を使用して、アプリIDだけの出力をフィルタリングすることができます。例えば、アプリのパッケージIDが `io.cordova.plugin.cloudsettings.test` の場合です。
```bash
    $ adb shell dumpsys backup | grep -P '^\S+\: | \: io\.cordova\.plugin\.cloudsettings\.test'
```

また、adb または Google Play ストアアプリを使って、アプリをアンインストールして再インストールすることで、アプリの自動復元をテストすることもできます。

### バックアップデータを消去する
アプリのバックアップデータを消去するには

```bash
    $ adb shell bmgr list transports
    # note the one with an * next to it, it is the transport protocol for your backup
    $ adb shell bmgr wipe [* transport-protocol] <APP_PACKAGE_ID>
```

## iOS でテストする

### バックアップのテスト

iCloud のバックアップは定期的に行われるため、プラグイン経由でデータを保存するとディスクに書き込まれますが、すぐに iCloud に同期されない場合があります。

iCloud ですぐに強制的にバックアップを取るには（iOS 11の場合）。

- 設定アプリを開く
- アカウントとパスワード > iCloud > iCloudバックアップ > 今すぐバックアップ を選択します

### レストアのテスト

レストアをテストするには、アンインストールしてからアプリを再インストールして、iCloudからデータを同期して戻します。

2台のiOSデバイスにアプリをインストールすることもできます。片方のデバイスにデータを保存すると、もう片方のデバイスで `onRestore()` コールバックが呼び出されるはずです。

# サンプルプロジェクト[](#example-project)

このプラグインの使用方法を説明/検証するプロジェクトの例は、ここにあります。[https://github.com/dpa99c/cordova-plugin-cloud-settings-test](https://github.com/dpa99c/cordova-plugin-cloud-settings-test)

# GDPR に準拠したアナリティクスでの使用

## GDPR の背景
- EUの[一般データ保護規則(GDPR)](https://www.eugdpr.org/) が2018年5月25日から施行されました。
- アプリやウェブサイトによる個人データの利用に関する厳格な規制を導入しています。
- GDPR では、3種類のデータとそれに付随する義務を区別しています([参考](https://iapp.org/media/pdf/resource_center/PA_WP2-Anonymous-pseudonymous-comparison.pdf)) 。

    - 個人を特定できる情報（PII)
        - 直接個人を特定する
        - 例：暗号化されていないデバイスID
        - GDPR に基づく完全義務化
    - 擬擬態化データ
        - 間接的に個人が特定される
        - 例: 顧客IDの暗号化された [一方向ハッシュ](https://support.google.com/analytics/answer/6366371?hl=en&ref_topic=2919631)
        - GDPR に基づく一部義務化
    - 匿名データ
        - 個人とは無縁
        - 例：ランダムに生成されたGUID
        - GDPR 義務なし

## アナリティクスにおけるユーザー追跡への影響
- [アプリのインストールをまたいでユーザーIDを追跡するのに便利](https://support.google.com/analytics/answer/3123663)
- ただし、GDPR は、Google Analytics や Firebase などの分析プラットフォームに送信された個人情報にも適用されます。
- PII または仮名化されたデータ（ユーザーIDなど）を使用する場合、ユーザーに以下のメカニズムを提供することが GDPR によって義務付けられています。
    - データを送信する前にアナリティクスにオプトインする（暗黙の同意は GDPR ではもはや受け入れられません）
    - オプトアウト
    - 分析データの検索または削除を要求する

## このプラグインを使うメリット[](#benefits-of-using-this-plugin)
- このプラグインを使用して、ランダムに生成されたユーザー ID を保存することができます。
- これを分析サービスに渡すことで、アプリのインストールやデバイス間でユーザーを (匿名で) 追跡することができます。
- GUID はランダムで、ユーザーの個人情報とは関連性がないため、GDPR では**匿名データ**として分類され、義務づけられていません。
- つまり、オプトイン/オプトアウトを提供する必要はなく、分析データの検索や削除を提供する義務はありません。
    - これはもちろん、アナリティクスに他のPIIや仮名データを送信していない場合にのみ適用されます。

# Authors

- [Dave Alden](https://github.com/dpa99c)

Major code contributors:
- [Daniel Jackson](https://github.com/cloakedninjas)
- [Alex Drel](https://github.com/alexdrel)

Based on the plugins:
- https://github.com/cloakedninjas/cordova-plugin-backup
- https://github.com/alexdrel/cordova-plugin-icloudkv
- https://github.com/jcesarmobile/FilePicker-Phonegap-iOS-Plugin

# Licence

The MIT License

Copyright (c) 2018, Dave Alden (Working Edge Ltd.)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.