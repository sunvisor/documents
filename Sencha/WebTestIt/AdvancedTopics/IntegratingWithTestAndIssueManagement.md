# テストと課題管理との統合

[テスト管理ツール](https://en.wikipedia.org/wiki/Test_management_tool) とは、テストしているソフトウェアのテストケースを整理するためのシステムです。テスト管理ツール内のテストケースを、Sencha WebTestItで作られた自動化されたテストケースとリンクさせることができます。テスト管理システムの構成によっては、テストケースのセットを直接実行することもできます。

Sencha WebTest は、[TestRail](https://www.gurock.com/testrail) との統合を可能にします。課題追跡ツールは、ソフトウェアの不具合を管理し、追跡するのに役立ちます。Sencha WebTestIt を [Jira](https://www.atlassian.com/software/jira) に接続して、失敗したテストケースからレポートから直接問題を生成することができます。また、再テスト後に問題を解決することもできます。

### TestRailとの接続

[Gurock の TestRail](https://www.gurock.com/testrail) を使用している場合は、そこからテストケースをインポートして、プロジェクト内のテストケースにリンクさせることができます。詳しくは [TestRailを使う](./GettingStartedWithTestrail.md) を参照してください。

### Jira との接続

課題トラッカーとして [Jira](https://www.atlassian.com/software/jira) を使用している場合、Sencha WebTestIt の設定でJiraサーバーに接続するための資格情報を提供することができます。詳細については、[Jiraを使う](./GettingStartedWithJira.md) を参照してください。
