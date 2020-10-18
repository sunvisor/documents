# Android 端末の接続方法

> オリジナル: [How to connect an Android device](https://docs.sencha.com/webtestit/guides/advanced-topics/how-to-connect-an-android-device.html)

Sencha WebTestIt は USB 接続された Android デバイス上でのテストの実行をサポートします。

### 手順

以下の手順に従って、モバイルテストを開始してください。

1.  ADB (Android Debug Bridge) がシステムにインストールされていることを確認してください。お使いのオペレーティングシステムに合った適切なダウンロードをここで見つけることができます。
    
      - [Android SDK tools for Windows](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)
      - [Android SDK tools for MacOS](https://dl.google.com/android/repository/platform-tools-latest-darwin.zip)
      - [Android SDK tools for Linux](https://dl.google.com/android/repository/platform-tools-latest-linux.zip)

2.  ターミナルやコマンドラインから直接アクセスできるように、ADB の実行ファイルを PATH に登録します。

3.  デバイスを接続して、**Settings \> DeveloperSettings \> USB Debugging** メニューで  *"USB Debugging "* を ON にしてください。

    > ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
    > 
    > 最近のAndroid端末では、USBデバッグがデフォルトで非表示になっています。[ここ](https://developer.android.com/studio/debug/dev-options) でデバイス上の開発者オプションを設定する方法を見ることができます。

4.  `adb devices` とタイプするとデバイスIDが表示されるはずです。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/adb-devices.png" width="700px" />

5.  Sencha WebTestIt を再起動します。

6.  目的のプロジェクトを開き、新しいエンドポイントを追加します。
    
    ![](https://docs.sencha.com/webtestit/guides/images/new-endpoint.png)

これで、`local endpoints` セクションの下部に Android デバイスが選択可能なオプションとして表示されるはずです。**Save endpoint** ボタンをクリックすると、すべての設定が完了します。


![]()
<img src="https://docs.sencha.com/webtestit/guides/images/execution-endpoint.png" height="700px" />

エンドポイントを追加した後は、Android 版 Chrome でテストを実行できるようになります。
