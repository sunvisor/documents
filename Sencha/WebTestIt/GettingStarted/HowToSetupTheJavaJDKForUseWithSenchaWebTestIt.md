# Sencha WebTestItで使用する Java JDK のセットアップ方法

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> JDK バージョン 9 と 10 のサポートが終了し、それらのバージョンは Selenium のビルドをサポートしていないため、**JDK 11+** のバージョンをインストールして使用することをお勧めします。そうしないと、テストを実行しようとするとエラーが発生する可能性があります。これについての詳細な情報は [こちら](https://github.com/SeleniumHQ/selenium/wiki/Building-WebDriver) を参照してください。Sencha WebTestIt は JDK バージョン 8 でも動作します。
> 
> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> この記事では、JDKバージョン8と11の [Oracle バージョン](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html) が新しいライセンス条項の対象となっているため、代わりに[OpenJDK](https://adoptopenjdk.net/) をセットアップする方法を推奨し、紹介します。

以下に、JDKのインストールに関する重要なヒントとリンクをご紹介します。

1.  [OpenJDK](https://adoptopenjdk.net/) をダウンロードしてインストールします。Linuxの場合は、ターミナルで `apt install openjdk-11-jdk` とインストールすることもできます。
2.  Sencha WebTestIt を再起動します。

これで設定完了です。Sencha WebTestItは自動的にあなたのJDKのインストールを検出し、必要なJAVA_HOME変数のパスを内部的に設定します。複数のJDKがインストールされている場合、Sencha WebTestItは最新のJDKバージョンを使用します。

何らかの理由でJDKのインストールが見つからず、プロジェクトの作成中に変数に関する警告メッセージが表示された場合は、以下の手順に従って手動で変数を設定することができます。

### Windows での JAVA\_HOME の設定

Windows に Java をインストールしたら、Java のインストールディレクトリを指す環境変数 JAVA\_HOME を設定する必要があります。

一番簡単な方法は、openJDK 11 のインストールプロセスの中にある Set JAVA_HOME variable オプションを確認することです。

![](https://docs.sencha.com/webtestit/guides/images/open-jdk.png)

JAVA\_HOME 環境変数を設定するには、

1. Java をインストールしたディレクトリを探します (C:\\Program Files\\Javajdk-11.0.5.10-hotspot といった感じです)

2. 次のうち一つを選びます、
    
      - **Windows 7** – **My Computer** を右クリックして**Properties \> Advanced** を選択します
    
      - **Windows 8** – **Control Panel \> System \> Advanced System Settings** を開きます
    
      - **Windows 10** – 環境変数を検索し、システム環境変数の**編集**を選択します。
        
        ![](https://docs.sencha.com/webtestit/guides/images/open-sys.png)

3.  **Environment Variables** をクリック
    
    ![](https://docs.sencha.com/webtestit/guides/images/open-environment.png)

4.  **System Variables** の下の **New** をクリック
    
    ![](https://docs.sencha.com/webtestit/guides/images/open-system-variables.png)

5.  変数名のフィールドに JAVA\_HOME と入力します

6.  値のフィールドに JDK のインストールパスを入力します

7. **OK** をクリックし、プロンプトが表示されたら **変更の適用** をクリックします。

8.  Sencha WebTestIt を再起動します

[WindowsでのJAVA_HOMEの設定についての詳細はこちら](https://confluence.atlassian.com/doc/setting-the-java_home-variable-in-windows-8895.html).

### macOS で JAVA\_HOME を設定する

1.  ターミナルを開きます **Applications \>\> Utilities \>\> Terminal**

2.  *.profile* を編集します

3.  次の行をファイルの最後に追加します
    
    ```bash
    JAVA_HOME=/Library/Java/Home
    export JAVA_HOME;
    ```

4.  保存してエディタを終了します

5.  確認のため、新しいターミナルを開き `$JAVA_HOME/bin/java -version` を実行します

6.  Sencha WebTestIt を再起動します

## Linux で JAVA\_HOME を設定する

1.  nano テキストエディタがなければインストールします

2.  ターミナルを開き次のようにタイプします: `sudo nano /etc/environment`

3.  次の行を追加します
    
    ```bash
    JAVA_HOME=<path to the location where you have installed JDK>
    export JAVA_HOME
    ```

4.  ファイルを保存します

5.  最後に、次のコマンドでシステムの PATH をリロードします: `$ . /etc/environment`

6.  `echo $JAVA_HOME` とタイプして確認します

7.  Sencha WebTestIt を再起動します
