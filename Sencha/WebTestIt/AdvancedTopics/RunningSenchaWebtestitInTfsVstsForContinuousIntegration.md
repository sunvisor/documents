# 継続的インテグレーションのために TFS/VSTS で Sencha WebTestIt を実行する

> オリジナル: [Running Sencha WebTestIt in TFS/VSTS for Continuous Integration](https://docs.sencha.com/webtestit/guides/advanced-topics/running-sencha-webtestit-in-tfs-vsts-for-continuous-integration.html)

ここでは、Team Foundation Server（TFS）またはVisual Studio Team Services（VSTS）を使用してビルドプロセスを作成するために必要な手順を説明します。

1.  プラットフォームにカスタムエージェントをデプロイする必要があります。[公式ドキュメント](https://docs.microsoft.com/en-us/vsts/build-release/actions/agents/v2-windows?view=vsts) の指示に従ってください。
2.  エージェントが interactive モードで実行されていることを確認してください。
3.  新しく作成したエージェントに Sencha WebTestIt をインストールします。
4.  ビルドエージェント上で [Java jdkのセットアップ手順](../GettingStarted/HowToSetupTheJavaJDKForUseWithSenchaWebTestIt.md) に従ってください。
5.  ビルドエージェントにテストしたいブラウザをすべてインストールしてください。
6.  VSTS のビルドに新しいステップを追加します。

### Windows

1.  `Command line` タスクを追加します。
    
    ![](https://docs.sencha.com/webtestit/guides/images/vsts-windows.png)

2.  `Display name` に `Run Sencha WebTestIt.exe` をセットします。

3.  `Tool` に `%LOCALAPPDATA%\Programs\WebTestIt\Sencha WebTestIt.exe` をセットします。

4.  `Arguments` に `run --report-file-destination=$(Build.SourcesDirectory)\path\to\TestResults\directory $(Build.SourcesDirectory)\path\to\project` をセットします。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/vsts-windowstwo.png" width="700px"/>

5.  テスト結果をパブリッシュしたい場合は `Publish Test Results` タスクを追加します。
    
    ![](https://docs.sencha.com/webtestit/guides/images/vsts-windowsresult.png)

6.  `Test result format` に `JUnit` をセットします。

7.  `Test results file` に `*.xml` をセットします。

8.  `Search folder` に `$(Build.SourcesDirectory)\path\to\TestResults\directory` をセットします。

9.  コントロールオプションで `Run this task` を `Even if a previous task has failed, unless the build was canceled` に設定します。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/vsts-windowsrun.png" width="700px"/>

### Mac

1.  `Shell script` タスクを追加します。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/vsts-macshell.png" width="700px"/>

2.  `Display name` に `Shell Script Sencha WebTestIt` をセットします。

3.  `Script Path` に `/Applications/Sencha\ WebTestIt.app/Contents/MacOS/Sencha\ WebTestIt` をセットします。

4.  `Arguments` に `run --report-file-destination=$(Build.SourcesDirectory)/path/to/TestResults/directory $(Build.SourcesDirectory)/path/to/project` をセットします。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/vsts-macscript.png" width="700px"/>

5.  If you like to publish the test results Add a task `Publish Test Results`
    
    ![](https://docs.sencha.com/webtestit/guides/images/vsts-macpublish.png)

6.  `Test result format` に `JUnit` をセットします。

7.  `Test results file` に `*.xml` をセットします。

8.  `Search folder` に `$(Build.SourcesDirectory)/path/to/TestResults/directory` をセットします。

9.  コントロールオプションで `Run this task` を `Even if a previous task has failed, unless the build was canceled` に設定します。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/vsts-macrun.png" width="700px"/>
