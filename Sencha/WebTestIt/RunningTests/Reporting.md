# 結果を得る

> オリジナル: [Getting results](https://docs.sencha.com/webtestit/guides/running-tests/reporting.html)

Sencha WebTestItはテスト実行ごとにレポートを作成します。テストを実行した後、**Project** タブの中にある `reports` フォルダを見つけます。そこにあるレポートファイルをクリックして、**Report** タブを開きます。

> ![](https://docs.sencha.com/webtestit/guides/images/images/hint-icon.png) **Hint**
> 
> 最新のテスト結果をすぐに見たい場合は、ステータスバーの**Show report**ボタンをクリックします。<br />
>![](https://docs.sencha.com/webtestit/guides/images/show-report.png)

**Report** タブには、完了したテスト実行の詳細な概要が表示されます。

<img src="https://docs.sencha.com/webtestit/guides/images/report.png" width="700px" />

1. **日時**: テスト実行が開始された正確な日時。

2. **Duration**: テスト走行にかかった時間。

3. **Test suites カウンター**: 実行されたテストスイート（テストファイル）の数。

4. **Test case カウンター**: すべてのテストスイートで実行されたテストケースの総数。

5. **エンドポイントフィルター**: 詳細ビューをフィルタリングするエンドポイントにチェックを入れたり外したりします。

6. **詳細ビュー**: 各テストケースの個別の結果を表示します。

7. **Upload report**: 現在のレポートをレポートサーバーにアップロードするには、クリックします。

8. **Save report**: クリックしてレポートをPDFファイルとして保存します。

9. **Print**: クリックしてレポートを印刷します。

10. **Refresh button**: クリックしてレポートを更新します。

レポート ファイルのレイアウトは、好みに合わせてカスタマイズすることができます。テンプレートを変更することで行うことができます。[ハウツー記事はこちら](../AdvancedTopics/AdjustingTheReportTemplate.md)

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **テストスイートとテストケースのカウントに関する注意点**
> 
> カウンタは常に個々のテストスイートとテストケースの実行数を示します。3つのテストケースを含む1つのテストスイートを2つのエンドポイントで実行した場合、テストケースの総数は6になりますが、テストスイートの総数は2になります。

テスト実行中にエラーが発生した場合は、**Jump to test case** ボタンをクリックすることで、コード内の問題のあるテストケースに直接ジャンプすることができます。

![](https://docs.sencha.com/webtestit/guides/images/error.png)

**Report** タブの上部にあるボタンをクリックして、失敗したテストケースを再実行することができます。

![](https://docs.sencha.com/webtestit/guides/images/rerun.png)

失敗したテストケースをさらに調査するために、診断を開始することができます。診断モードについての詳細は、[このガイドの次の章](../Maintenance/Introduction.md) を参照してください。
