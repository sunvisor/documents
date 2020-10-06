# Cordova または PhoneGap との統合

Original: <https://docs.sencha.com/cmd/7.3.0/guides/cordova_phonegap.html>

## Cordova

[Apache Cordova](http://cordova.apache.org/) は、Android、iOS、BlackBerry、Windows Phone デバイス用のアプリケーションを作成するAPIとパッケージングツールを提供します。Sencha Cmd 5+ は、複数のビルドプロファイルをサポートするビルドシステムを提供します。ネイティブパッケージに最適です。Cordova Native Packager コンポーネントは、この機能をフルに活用できるようにアップデートされています。

## PhoneGap

PhoneGap は Cordova の上に構築されており、どちらも Cordova のAPIにアクセスすることができます。PhoneGap と Cordova は、パッケージングツールの実装方法が異なります。PhoneGap は [Adobe PhoneGap Build](https://build.phonegap.com/) でリモートビルドインターフェースを提供しており、クラウド上の単一プラットフォーム向けのアプリケーションをパッケージ化してエミュレートすることができます。PhoneGap Native Packager コンポーネントは、この機能をフルに活用できるように更新されています。

## パッケージャーを選ぶ

Cordova と PhoneGap のどちらかを選択する際には、いくつかの点に注意することが重要です。

- Cordova のパッケージングツールでは、マシン上で複数のプラットフォーム向けに同時にビルドすることができます。
- Cordovaはオープンソースコミュニティによって常にアップデートされていますが、PhoneGapのアップデートはAdobeによって調整されています。
- PhoneGapはリモートビルドアプリケーションを提供し、ローカルビルドの必要性を排除します。


## Cordova や PhoneGap で Sencha Cmd を使う

Sencha Cmdで生成されたすべてのアプリは、これらのサービスを介してネイティブプラットフォーム向けにビルドする機能を持っています。Sencha Cmd は、アプリをビルドしたり、Cordova や PhoneGap 用の適切な場所にアプリを配置したり、適切なコマンドを実行してアプリをビルドしたり、エミュレートしたり、実行したりするなどの反復的なタスクをすべて処理してくれます。

ネイティブ開発を始めるための詳細な情報については、[Apache Cordova プラットフォームガイド](http://cordova.apache.org/docs/en/3.0.0/guide_platforms_index.md.html#Platform%20Guides) を参照してください。これらのガイドには、システムを起動して実行するために必要な情報と前提条件が記載されています。

### 重要な注意事項

1. packager.json は完全に削除されました。プロジェクトにこのファイルが残っている場合は、念の為削除してください。このファイルは Sencha のレガシーな Native Packager でのみ使用されていました。

2. Cordova と PhoneGap の場合、Sencha Cmd はアプリケーションのDEBUGバージョンのみを作成することができ、リリース(アプリストア用)バージョンは作成できません。リリースバージョンを作成するには、Android の場合は Eclipse や IntelliJ、iOS の場合はXcode などのパッケージャーを使用する必要があります。

3. Sencha Cmd Cordovaコマンドが正常に動作するためには、[Java JRE 1.8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) が必要です。Windows と Mac OS X用の Sencha Cmd 6 以降は、デフォルトのインストーラーに最新のJRE を入れてダウンロードすることができますが、CordovaツールはSencha Cmd の内部 JRE を見つけて利用することはできません。

## 環境のセットアップ

Cordova や PhoneGap　アプリケーションを開発する前に、以下のソフトウェアで環境を整える必要があります。

1. [Javaのインストール](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
    JRE バージョン 8 から OpenJDK 11 に対応しています。
2. [Node.jsのインストール](http://nodejs.org/download/)
3. Cordova でパッケージ化してエミュレートする場合は、以下のコマンドでインストールします。
    ```
    npm install -g cordova
    ```

    Macの場合は、インストールを完了させるために "sudo "を含める必要があるかもしれません。
    コマンドは次のようになります。

    ```
    sudo npm install -g cordova
    ```

    **注意**：Cordovaをインストールしてアプリをパッケージ化してエミュレートするかどうかに関わらず、Cordova API はアプリで利用できます。Cordova のパッケージ化とエミュレーションは、Cordova または PhoneGap アプリでの Cordova API の使用には影響しません。
4. PhoneGapでパッケージ化してエミュレートしたい場合は、以下のコマンドでインストールします。

    ```
    npm install -g phonegap
    ```

    Macの場合は、インストールを完了させるために "sudo "を含める必要があるかもしれません。
    コマンドは次のようになります。

    ```
    sudo npm install -g phonegap
    ```

    **注意**：PhoneGap リモートビルドサイトにアクセスするには、無料の [Adobe PhoneGapビルド](https://build.phonegap.com/)サイトに登録し、ユーザー名とパスワードを取得する必要があります。

5. Sencha Cmdをダウンロードしてインストールします。
    このプロセスの詳細については、Cmd 入門ガイドをご確認ください。

6. ターゲットデバイスのプラットフォーム固有の要件を確認し、それを満たす
    **Android**: [Android SDK Manager](http://developer.android.com/sdk/index.html) をダウンロードしてインストールします。

    **Blackberry**: [BlackBerry Native SDK](http://developer.blackberry.com/native/) を確認し、BlackBerry Keys Order Formでアプリに署名するために登録します。

    **iOS**: [Apple iOS provisioning profiles portal](https://developer.apple.com/account/ios/profile/profileLanding.action) でiOSプロビジョニングのすべての手順を完了します (Apple ID、パスワード、購入した開発者ライセンスが必要です)。このサイトを使用して証明書を取得し、デバイスを識別し、AppIDを取得します。

    さらに、無料のXcodeソフトウェアをダウンロードしてインストールします。デバイスにインストールする前に、Xcodeシミュレータを使ってiOSアプリをデバッグすることができます。

## config.xml

PhoneGapまたはCordovaアプリを作成すると、両方ともプロジェクト用の `config.xml` ファイルを作成します。

Cordova は `config.xml` を "app\_root/cordova "フォルダに保存し、PhoneGap は `config.xml` ファイルを "app\_root/cordova/www" フォルダに保存します。これは www フォルダがコンパイルされたビルドの保存先になっているためです。この www フォルダが削除されて `config.xml` ファイルが消去されてしまう可能性が高いです。

Sencha Cmdでこの問題を解決するには、PhoneGap アプリケーションが作成されたときに `config.xml` ファイルを PhoneGap プロジェクトのルートにコピーします。ビルドを行うたびに、コンパイルされたアプリケーションと一緒に www フォルダにコピーします。これは、どのフレームワークを使用しているかに関わらず、`config.xml` ファイルが Cordova または PhoneGap フォルダのルートにあることをユーザーが期待できることを意味します。

PhoneGap と Cordova の設定の詳細については、以下のリソースを参照してください。

- [PhoneGap example config.xml file](https://github.com/phonegap/phonegap-start/blob/master/www/config.xml)
- [Configuration Reference](http://docs.phonegap.com/en/3.0.0/config_ref_index.md.html#Configuration%20Reference)

## Sencha コマンドアプリの生成とCordova または PhoneGapアプリの初期化

次のようにCmdのgenerateコマンドを使ってアプリケーションを作成します。

```
sencha -sdk /path/to/extjs-framework generate app MyApp /path/to/MyApp
```

CordovaやPhoneGapアプリをこのように初期化します。

```
新しく生成されたプロジェクトフォルダに移動し、次のコマンドのいずれかを実行します。
(app_idとapp_nameの引数はオプション)。

sencha phonegap init com.mycompany.MyApp MyApp

または

sencha cordova init com.mycompany.MyApp MyApp

上記のコマンドが完了した後、プロジェクトフォルダ内に phonegap または cordova ディレクトリが表示されているはずです。
```

## Cordova アプリの開発

### ユニバーサル・アプリの場合

Ext JS 6+ のユニバーサルアプリケーションを作成した場合、生成されたアプリケーションにはすでに`app.json` に `builds` ブロックが含まれています。アプリケーションが既に `builds` ブロックを含んでいる場合、`sencha phonegap/cordova init` コマンドは `app.json` を変更することができません。Cordovaのサポートを追加するには、アプリケーションの`app.json` の `builds` オブジェクトを変更して Cordova の統合を追加してください。以下の例は、`modern` ビルドプロファイルを変更する方法を示しています。

```json
"builds": {
    "classic": {
        "toolkit": "classic",
        "theme": "theme-triton"
    },

    "modern": {
        "toolkit": "modern",
        "theme": "theme-cupertino",
        "packager": "cordova",
        "cordova": {
            "config": {
                "platforms": "ios",
                "id": "com.mydomain.MyApp"
            }
        }
    }
}
```

既存のプロファイルを変更することができます。また、新しいプロファイルを作成して、好きな名前にすることもできます。以下は、Cordova と連携するカスタムビルドプロファイルの例です。

```json
"cordova_android": {
    "packager": "cordova",
    "toolkit": "modern",
    "theme": "theme-neptune",
    "cordova" : {
        "config": {
            "platforms": "android",
            "id": "com.mycompany.test",
            "name": "test"
        }
    }
}
```

### ユニバーサルではないアプリの場合

ユニバーサルアプリケーションを生成していない場合、`app.json` に `builds` オブジェクトが表示されます。このファイルは`sencha cordova init` コマンドを実行した時に作成されました。

これで、プロジェクトの `app.json` ファイルに PhoneGap や Cordova が統合されたビルドプロファイルが表示されるはずです。`app.json` はプロジェクトのルートにあることに注意してください。

`app.json` ファイルの中に以下のようなビルドオブジェクトが存在することを確認してください。

```json
"builds": {
    "native": {
        "packager": "cordova",
        "cordova" : {
            "config": {
                "platforms": "android",
                "id": "com.mydomain.MyApp"
            }
        }
    }
}
```

#### カスタムビルドプロファイルの使用

独自のカスタム ビルド プロファイルを作成するか、または生成された `native` プロファイルを使用する場合、アプリのマニフェストの作成で使用するプロファイルを手動で指定する必要があります。これを行うには、次のようにします。

1. プロジェクトのルートで **index.html** を開く
2. **Ext.beforeLoad** フックを探します。
3. このフック内のコードの終わり近くに、`Ext.manifest = profile`という行があることに注意してください。
4. カスタムビルドのプロファイルをフック内のプロファイル変数に手動で設定するコードを追加します。

```js
    // Add this line
    profile = 'your_custom_profile_name here';

    // Add your code above this line
    Ext.manifest = profile;
```

**注意**: ここで使用するプロファイルを、次のセクションでリストアップされている Sencha コマンドで使用するようにしてください。

### ビルドプロファイルとビルド名

これらのビルドプロファイルについて少しお話しましょう。

後者の例では、"native "という単語は単にビルド名であり、好きなようにつけられるということに注意してください。ビルド名は単一の文字列でなければなりません。例えば、「ios」、「android」、「iphone」、「ipad」などのようなビルド名を指定することができます。ユニバーサルアプリを生成した場合、「modern」や「classic」などの他のビルドプロファイルが表示されます。ビルド、エミュレート、パッケージなどを行う際には、ビルドプロファイルの名前を使用します。

**注意**: ビルド名には何を選んでも構いませんが、"production"、"testing"、"development "という名前は Sencha Cmd によって予約されており、この文脈では使用すべきではありません。

**注意**：ビルドやエミュレートなどのコマンドを実行する際には、どのビルドプロファイルをビルドするかを指定する必要があります。

### プラットフォーム

プラットフォームオブジェクトには、任意のプラットフォーム、またはプラットフォームの組み合わせを設定することができます。Cordovaの場合は、"ios android "のようにスペースで区切られたリストを指定することができます。

### ID

"id" プロパティは Cordova アプリケーションが最初に生成されたときに使用されます。これはアプリケーションの識別子になります。このプロパティが正しく選択されていることを確認してください。もし変更する必要がある場合は、プロジェクトから Cordova フォルダを削除して、Cmd に ID プロパティを変更した後に Cordova フォルダを再生成させてください。これはiOSアプリケーションでは特に重要なことで、これは Bundle Identifier と一致する必要があります。

#### Cordova アプリのビルド

ターミナルまたはコマンドラインから以下のコマンドを実行します。

```
sencha app build {build-profile-name}
```

ここで `{build-profile-name}` は `app.json` ファイルのビルドオブジェクトで定義された名前の一つです。例えば、cordovaのビルドプロファイルを "android "と呼ぶ場合、コマンドは次のようになります。

```
sencha app build android
```

これで完了です。Sencha Cmd はこれで Cordova アプリケーションを作成し、`app.json` の platform プロパティで指定したプラットフォームをビルドします。ビルドを成功させるためには、適切なSDKとサポートツール（Android の場合は Android Studio、Gradle など、iOS の場合は Xcode など）が必要です。Cordovaは依存関係を見逃している場合、親切に説明してくれます。

## Sencha Crdova コマンド

Cordova とこの新しいビルドオブジェクトで使用できるコマンドは4つあります。

**注意**：`{build-profile-name}` には Cordova のビルドプロファイル名を設定してください。

### Build

```
sencha app build {build-profile-name}
```

"build" コマンドは、Sencha アプリケーションと、ビルドプロファイルで指定されたプラットフォーム用のネイティブアプリケーションをビルドします。

### Run

```
sencha app run {build-profile-name}
```

"run" は Sencha アプリケーションとネイティブアプリケーションをビルドします。そして、接続されたデバイス上で実行しようとします。

### Emulate

```
sencha app emulate {build-profile-name}
```

"emulate" は Sencha アプリケーションとネイティブアプリケーションをビルドします。その後、エミュレータで実行しようとします。

**注意**: Android Studio / ADV / Emulatorを使った連続したデプロイで問題が発生した場合、このコマンドの実行時に生成された APK へのパスを常にメモしておくことができます。あなたのエミュレータにインストールして実行するために ADB を使用することができます。あなたの apk をインストールするには、次のコマンドを実行します。

```
adb install /path/to/apk/app-debug.apk
```

### Prepare

```
sencha app prepare {build-profile-name}
```

"prepare" は Sencha アプリケーションをビルドし、ビルドされたアプリケーションをネイティブアプリに準備します。しかし、ネイティブ部分はビルドされません。これは、Sencha のコンパイル後、ネイティブビルドの前にアプリケーションに何かを注入する必要がある場合に良いでしょう。

## PhoneGap アプリの開発

PhoneGap は Cordova と非常によく似ています。実際、PhoneGap のクラウドサービス経由でビルドしていない場合は、プロセスはほぼ同じです。

ユニバーサルアプリに取り組んでいる場合は、既存のビルドプロファイルを修正して PhoneGap をサポートするか、独自のファイルを作成することができます。上記の Cordova のセクションに記載されているのと同じ手順を踏むことができます。

上の Cordova プロジェクトと同じ PhoneGap プロジェクトを作成するための`app.json` のスニペットは以下の通りです。

```json
"builds": {
    "native": {
        "packager": "phonegap",
        "phonegap" : {
            "config": {
                "platform": "ios",
                "id": "com.mydomain.MyApp"
            }
        }
    }
}
```

些細なことですが、重要な違いがあります。

- 名前が "cordova" から "phonegap" に変わる。
- "platform" は "platforms" ではなく単数形です。PhoneGap は一度に1つのローカルプラットフォームを構築することができますが、Cordova は同時に複数のプラットフォームを構築することができます。

これらの小さな違いを認識してしまえば、前と同じコマンドです。単純に `sencha app build native` か `sencha app run native` を実行すれば、Cordova と同じ道を歩むことになります。

## Sencha PhoneGap コマンド

PhoneGapとこの新しいビルドオブジェクトで使用できるコマンドは3つあります。PhoneGap は Cordova で見つかった prepare コマンドをサポートしていません。他の3つは上記の Cordova コマンドと同じです。

### Build

```
sencha app build {build-profile-name}
```

"build "はsenchaアプリケーションをビルドしてからネイティブアプリケーションをビルドします。

### Run

```
sencha app run {build-profile-name}
```
"run" を実行すると、sencha アプリケーションとネイティブアプリケーションがビルドされます。そして、接続されたデバイス上で実行しようとします。

### Emulate

```
sencha app emulate {build-profile-name}
```

"emulate" は Sencha アプリケーションとネイティブアプリケーションをビルドします。その後、エミュレータで実行しようとします。

## リモートPhoneGapアプリの開発

PhoneGap アプリケーションをクラウドで構築することには、多くの利点があります。お使いのマシン用のSDKやツールをすべてダウンロードするという通常の手間をかける必要はありません。WebアプリケーションをPhoneGapサーバーに送信するだけで、Adobeがネイティブアプリケーションを生成してくれます。

Sencha Cmdとリモートビルドを始める前に、<http://build.phonegap.com> にアクセスして無料アカウントにサインアップしてください。

`index.html` ファイルを含むZIPをアップロードするだけで、ビルドプロセスをテストできます。PhoneGap Remote Buildがパッケージ化します。

iOS用にビルドする場合は、サイトの指示に従って、アプリケーションをビルドするための適切な資格情報を追加する必要があります。

アカウントの設定が完了し、Adobeにアップロードされたすべての正しいファイルを確認したら、Senchaのワークフローにリモートビルドを素早く追加することができます。

リモートデプロイ用のビルドオブジェクトを見てみましょう。

```json
"builds": {
    "native": {
        "packager": "phonegap",
        "phonegap" : {
            "config": {
                "platform": "ios",
                "remote": true,
                "id": "com.mydomain.MyApp"
            }
        }
    }
}
```

ご覧のように、ローカルビルドからリモートビルドへの切り替えは非常に簡単です。必要なのは `app.json` ビルドブロックにリモートフラグを追加するだけです。

次のステップは、ユーザー名とパスワードを入力して、Sencha Cmdが自動的にPhoneGapリモートサーバーにログインしてプロジェクトをアップロードできるようにすることです。

これを行うには、アプリケーションのルートに `local.properties` ファイルを追加することをお勧めします。セキュリティ上の理由から、このプロパティファイルはすべてのバージョン管理で無視されるべきです。

このファイルに次の行を追加します。

```
phonegap.username=myname@domain.com
phonegap.password=s3nch@isgr3@t
```

ユーザー名とパスワードを設定したら、ビルドの送信は簡単です。`sencha app build native` を実行するだけで、アプリケーションがコンパイルされてクラウドに出荷され、Adobe がネイティブアプリケーションを生成します。

ビルドが完了したら、最終的なパッケージファイルに PhoneGap ポータルからアクセスしてデバイス上でテストしたり、該当するアプリストアにアップロードしたりすることができます。
