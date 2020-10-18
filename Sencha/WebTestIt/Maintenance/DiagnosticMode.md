# 診断モードでの実行

> オリジナル: [Running in diagnostic mode](https://docs.sencha.com/webtestit/guides/maintenance/diagnostic-mode.html)

診断モードはインタラクティブなテスト実行です。Sencha WebTestIt は開発者ツールを開いた状態でブラウザを開き、テスト実行に問題があった瞬間に実行を停止します。失敗したテスト実行のレポートを開くことで、診断モードを開始することができます。

失敗した行を探して、**Run test in diagnostic mode** をクリックします。または、テストファイルの中で、特定のテスト方法を右クリックしてコンテキストメニューから同じことを行うこともできます。

<img src="https://docs.sencha.com/webtestit/guides/images/jumptotestcase.png" width="700px" />

このボタンをクリックすると、Sencha WebTestItは特別な実行モードに入ります。

  - テスト実行は、元の実行と同じ構成で繰り返される。
  - サポートされている場合は、ブラウザの開発者ツールがSUTと並んで開かれます。
  - エラーが発生すると実行が停止し、正確なポイントで調査を開始することができます。

**Execution** タブで、**Show report** をクリックすると、これまでに記録された進行中のレポートを表示することができます。

ここには、エラーの詳細が表示されます。エラーが発生したコードの場所に直接切り替えるには、**Jump to test case** をクリックしてください。

修正が完了したら、**Stop diagnostic mode** をクリックしてテストの実行を終了します。すべてが正常に動作することを確認するために、テストの再実行を試みることができます。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> 診断モードでは、最初のエラーが発生するまで実行が継続します。エラーの可能性のある原因を修正した後、テストを再実行する必要があります。
