# テストを実行する

Sencha WebTestIt でテスト実行を開始するのは簡単です。少なくとも1つのエンドポイントを設定したら、準備は完了です。

1.  少なくとも1つのエンドポイントを最初に設定するようにしてください。

2.  **Project** タブからテストファイルを選択します。Sencha WebTestIt は自動的に **Execution** タブも一緒に開きます。

3.  テストを実行したいエンドポイントをチェックします。

4.  **Run current test file** または **Run all test files** ボタンをクリックします。

5.  あとは座ってお楽しみください。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> すべてのテストを実行したい場合は、最初にテストファイルを開く必要はありません。

Sencha WebTestテストを実行している間はアニメーションを表示します。詳細はログパネルで確認できます。 
**Stop execution** ボタンをクリックして、テスト実行を中断します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> 自動化されている間は、ブラウザーを操作しないでください。

### ヘッドレステスト

ローカルエンドポイントには *headless* オプションがあることにお気づきかもしれません。ヘッドレスをテストする場合、ブラウザインスタンスはバックグラウンドプロセスとして起動し、GUIを使用しません（それ故に名前がついています）。これとは別に、自動化は全く同じです。ヘッドレステストはCI環境やGUIを表示できないマシン（ビルドサーバのような）では非常に便利です。

![](https://docs.sencha.com/webtestit/guides/images/headless.png)

テストをローカルでヘッドレスで実行することもできます。テスト開発のためには、プロセスを見ることをお勧めします。

### コマンドラインからの実行

アプリを起動しなくてもSencha WebTestIt でテストを実行することができます。タスクプランナーを使ってテストを計画したい場合や、CI 環境に統合したい場合に便利なオプションです。


コマンドラインからプロジェクトを実行するには、`run` パラメータを指定して Sencha WebTestIt 実行ファイルを呼び出し、プロジェクトの場所を指定する必要があります。コマンドパターンは次のようになります。`<Webtestit app path> run [options] <path/to/your/project>`。コマンドライン実行は、サポートされているすべてのオペレーティングシステムで動作します。利用可能なすべてのオプションとコマンドの概要を知るには、[専用のハウツー記事を参照してください](../AdvancedTopics/RunningSenchaWebtestitInCliMode.md)。
