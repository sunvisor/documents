# 継続的インテグレーションのために Jenkins で Sencha WebTestIt を実行する

> オリジナル: [Running Sencha WebTestIt in Jenkins for Continuous Integration](https://docs.sencha.com/webtestit/guides/advanced-topics/running-sencha-webtestit-in-jenkins-for-continuous-integration.html)

以下では、Jenkinsでビルドプロセスを作成するために必要な手順を説明します。

Webtestitで使用する依存関係をインストールした後に新しいコマンドプロンプト/ターミナルを開き、新しい環境変数が認識されるようにエージェントを再起動していることを確認してください。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> ビルドを開始する前にSencha WebTestItアプリケーションを開いている場合は、必ず閉じてください。そうしないと、"Cannot run multiple instances" エラーが発生します。

### Windows
1. ユーザーアカウントでJenkinsサービスを実行する必要があります。[公式ドキュメント](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+as+a+Windows+service) の指示に従ってください。あるいは、[対話型モードでJenkinsスレーブ](https://wiki.jenkins.io/display/JENKINS/Distributed+builds) を使用することもできます。詳細は[公式ドキュメント](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins) を参照してください。

2.  Sencha WebTestIt をインストールし、Jenkins サービスを起動する際に使用したのと同じユーザーアカウントを使用します。

3. 必ずJenkinsサーバ/スレーブの「[Java jdkのセットアップ手順](../GettingStarted/HowToSetupTheJavaJDKForUseWithSenchaWebTestIt.md)」に従ってください。

4.  Jenkinsサーバ/スレーブにテストしたいブラウザをすべてインストールしてください。

5.  NEWコマンドプロンプトウィンドウを使用してエージェントを再起動します。

6.  ニーズに合った新しい Jenkins ジョブを作成します。

7.  テスト結果をパブリッシュしたい場合は、必ず [JUnit-Plugin](https://wiki.jenkins.io/display/JENKINS/JUnit+Plugin) をインストールしてください。.

8.  ジョブに `Execute Windows batch command` ビルドステップを追加します。

9.  次のコマンドを追加します。
    
    `"%LOCALAPPDATA%\Programs\webtestit\Sencha WebTestIt.exe" run --report-file-destination="%WORKSPACE%\testresults"` `--endpoints-config="/Users/mgrundner/Desktop\endpoints.json" "%WORKSPACE%\path\to\your\project"`

10. *Advanced Project Options* で、*Use custom workspace* チェックボックスにチェックを入れます。*Directory* フィールドにプロジェクトの場所を入力します。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-two.png)

11. ビルド後アクションに `Publish JUnit test result report` を追加します。

12. *Test report XMLs* に `testresults\*.xml` が含まれるようにします。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-three.png)

13. これで、ビルドを実行して結果を見ることができます。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-four.png)

### MacOS

1.  JenkinsエージェントをLaunchエージェントとして実行する必要があります。[公式ドキュメント](https://jenkins.io/doc/book/installing/) を参照してください。

2.  Jenkins エージェントを起動したユーザーで Sencha WebTestIt をインストールします。

3. 必ずJenkinsサーバ/スレーブの「[Java jdkのセットアップ手順](../GettingStarted/HowToSetupTheJavaJDKForUseWithSenchaWebTestIt.md)」に従ってください。

4.  Jenkinsサーバ/スレーブにテストしたいブラウザをすべてインストールしてください。

5.  NEWコマンドプロンプトウィンドウを使用してエージェントを再起動します。

6.  ニーズに合った新しい Jenkins ジョブを作成します。

7.  テスト結果をパブリッシュしたい場合は、必ず [JUnit-Plugin](https://wiki.jenkins.io/display/JENKINS/JUnit+Plugin) をインストールしてください。.

8.  ジョブに `Execute shell` ビルドステップを追加します。

9.  次のコマンドを追加します。
    
    `/Applications/Sencha\ Webtestit.app/Contents/MacOS/Sencha\ Webtestit run` `--report-file-destination=/users/yourUsername/yourProjectPath/reports` `--endpoints-config=/Users/yourUsername/locationOfEndpointFile Users/YourUsername/yourProjectPath`

10. *Advanced Project Options* で、*Use custom workspace* チェックボックスにチェックを入れます。*Directory* フィールドにプロジェクトの場所を入力します。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-six.png)

11. ビルド後アクションに `Publish JUnit test result report` を追加します。

12. *Test report XMLs* に `testresults\*.xml` が含まれるようにします。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-seven.png)

13. これで、ビルドを実行して結果を見ることができます。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-eight.png)

### Linux

1.  Jenkinsエージェントを Launch エージェントとして実行する必要があります。こちらの優れた[ブログ記事](https://medium.com/@ved.pandey/setting-up-jenkins-on-mac-osx-50d8fe16df9f) の指示に従ってください。あるいは、[Jenkinsスレーブ](https://embeddedartistry.com/blog/2017/12/22/jenkins-configuring-a-linux-slave-node) を使用することもできます。詳細は [公式ドキュメント](https://jenkins.io/doc/book/installing/#macos) を参照してください。
2.  Jenkins エージェントを起動したユーザーで Sencha WebTestIt をインストールします。

3. 必ずJenkinsサーバ/スレーブの「[Java jdkのセットアップ手順](../GettingStarted/HowToSetupTheJavaJDKForUseWithSenchaWebTestIt.md)」に従ってください。

4.  Jenkinsサーバ/スレーブにテストしたいブラウザをすべてインストールしてください。

5.  NEWコマンドプロンプトウィンドウを使用してエージェントを再起動します。

6.  ニーズに合った新しい Jenkins ジョブを作成します。

7.  テスト結果をパブリッシュしたい場合は、必ず [JUnit-Plugin](https://wiki.jenkins.io/display/JENKINS/JUnit+Plugin) をインストールしてください。.

8.  ジョブに `Execute shell` ビルドステップを追加します。

9.  次のコマンドを追加します。
    `"/opt/Sencha WebTestIt/webtestit"" run --report-file-destination="${WORKSPACE$}/testresults"` `--endpoints-config="$C:\Users\mgrundner\Desktop\endpoints.json" "${WORKSPACE$}/path/to/your/project"`

10. *Advanced Project Options* で、*Use custom workspace* チェックボックスにチェックを入れます。*Directory* フィールドにプロジェクトの場所を入力します。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-ten.png)

11. ビルド後アクションに `Publish JUnit test result report` を追加します。

12. *Test report XMLs* に `testresults\*.xml` が含まれるようにします。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-eleven.png)

13. これで、ビルドを実行して結果を見ることができます。
    
    ![](https://docs.sencha.com/webtestit/guides/images/jenkins-twelve.png)
