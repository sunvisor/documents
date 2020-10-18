# CI環境への統合

> オリジナル: [Integrating in CI environments](https://docs.sencha.com/webtestit/guides/advanced-topics/integrating-in-ci-environments.html)

Sencha WebTestItを開発ワークフローに組み込むことができます。開発ワークフローに [Continuous Integration (CI)](https://martinfowler.com/articles/continuousIntegration.html) を組み込むと、ビルドパイプラインの追加ステップとして自動テストを設定することができます。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> 継続的インテグレーションは、チームが一定のソフトウェア品質と迅速なフィードバックを確保するのに役立ちます。アジャイル手法と組み合わせることで、生産性を最大化することができます。

市場には数多くのCIツールが存在します。私たちは、JenkinsとTeam Foundation Server/Visual Studio Team Servicesのためのガイドを設定して、あなたが始めることができます。

| **CI Tool**                | **Link**                                                                                                                                                   |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Jenkins**                | [継続的インテグレーションのために Jenkins で Sencha WebTestIt を実行する](./RunningSenchaWebtestitInJenkinsForContinuousIntegration.md)   |
| **Team Foundation Server** | [継続的インテグレーションのために TFS/VSTS でSencha WebTestIt を実行する](./RunningSenchaWebtestitInTfsVstsForContinuousIntegration.md) |

使用している CI 環境がリストにない場合、テストを実行するように設定することができます。[コマンドラインモードの使い方](../RunningTests/HowTo.md#running-tests-_-how-to_-_running_from_the_command_line) と [ヘッドレステストに関する情報](../RunningTests/HowTo.md#running-tests-_-how-to_-_headless_testing) を参照して、別のプロセスからテストを開始する方法を確認してください。
