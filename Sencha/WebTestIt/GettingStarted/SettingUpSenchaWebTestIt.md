Sencha WebTestItの設定
=====================

以下の手順に従って、Sencha WebTestIt をインストールし、最初のプロジェクトを作成します。

## 1. Sencha WebTestItをダウンロードしてインストールする

以下の手順でSencha WebTestItをインストールします。

1. https://www.sencha.com/products/webtestit/ からインストーラをダウンロードします。
2. 提供されたフォームに必要事項を記入して Sencha WebTestIt のインストーラを入手し、**Download Now** をクリックします。
3. インストールプロセスを完了するためのすべてのパッケージのリンクが記載されたメールが届きます。

## 2. Javaをダウンロードしてインストール

プロジェクトに Selenium をセットアップするには Java が必要です。Sencha WebTestIt は、新しいプロジェクトを作成する際に、有効なJavaがインストールされているかどうかをチェックします。正しいJavaフレームワークがインストールされていないとプロジェクトを作成することはできません。Java がまだコンピュータにインストールされていない場合は、[Java SEランタイム環境](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) をダウンロードしてインストールする必要があります。Java でテストを書く場合は、Java SE Development Kit (JDK) をダウンロードする必要があります。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
>
> Java プロジェクトの場合。JDK バージョン 9 や 11 は現在 Selenium のビルドをサポートしていないため、JDK 8 バージョンをインストールして使用することを強くお勧めします。そうしないと、テストを実行しようとしたときにエラーが発生する可能性があります。ただし、[Oracle バージョン 8 の JDK](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) は新しいライセンス条項の対象となっています（有料オプションとして提供されています）ので、代わりに [OpenJDK](https://adoptopenjdk.net/) を使用してください。

[Sencha WebTestIt で使用するための Java JDK の設定方法](HowToSetupTheJavaJDKForUseWithSenchaWebTestIt.md)についての詳細な説明をお読みください。

## 3. 新しいプロジェクトを作成する

1. **Welcome** タブで、**New Project** リンクをクリックします。
<img src="https://docs.sencha.com/webtestit/guides/images/welcome-screen-new-project.png" width="700px">
2. **Create new project** ダイアログで、プロジェクト名を入力します。最初のプロジェクトでは、古典的な「HELLO WORLD」という名前を選びました。しかし、お好きな名前を選んでください。
3. プロジェクトを保存する場所、スクリプト言語、選択した自動化フレームワークを選択します。
4. **Save** をクリックしてプロジェクトを作成します。Sencha WebTestItはあなたの環境をセットアップし、必要な依存関係をすべてインストールします。

<img src="https://docs.sencha.com/webtestit/guides/images/welcome_screen-create_proj_dialog.png" width="700px">
![]()

## 4.全て順調です！

これで、最初のテストを作成する準備が整いました。これですべての設定が完了したので、最初のテストを作成する準備ができました!

次の章では、UIとウェブサイトの自動化について説明します。