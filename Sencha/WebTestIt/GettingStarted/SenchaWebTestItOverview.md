Sencha WebTestIt の概要
======================

Sencha WebTestItは、プロジェクト内の現在アクティブなファイルに応じて動的に適応するタブ型のユーザーインターフェースを特徴としています。すべての機能はタブに配置されており、編集中のファイルの種類に応じて自動的にアクティブになります。私たちは、あなたが道を見つけるのに役立つように概要を作成しました。

<img src="https://docs.sencha.com/webtestit/guides/images/overview-test-development.png" width="700px" />

1. **Project タブ:** プロジェクトのテストと設定ファイルを表示する
2. **Page Object タブ:** 現在開いているページオブジェクトの内容を表示します
3. **Elements タブ:** 現在開いているページオブジェクトのセレクタを管理するのに役立ちます
4. **Code タブ:** 現在選択されているファイルのソースを表示します

<img src="https://docs.sencha.com/webtestit/guides/images/overview-test-execution.png" width="700px" />

1. **Execution タブ:** エンドポイントを管理してテストを実行
2. **Log タブ:** オートメーションフレームワークのログ出力を表示します
3. **Report タブ:** 現在開いているテストレポートを表示します

> **Hint:**
> 
> すべてのタブのラベルをマウスでドラッグすることで、アプリケーション内でタブの位置やサイズを自由に配置することができます。デフォルトのレイアウトをリセットするには、[見た目を調整してデフォルトのレイアウトに戻す](../AdvancedTopics/AdjustingTheLooksAndRestoringTheDefaultLayout.md)にチェックを入れます。

## プロジェクト内のファイルを操作する

Sencha WebTestItコードエディタは、プロジェクト内のテキストベースのファイルを開いて表示することができます。ファイルを編集するには、**Project** タブ内でクリックするだけです。ほとんどの場合、UIを可能な限り最善の方法でワークフローをサポートするために適応させる3つの特殊なファイルタイプのうちの1つを使用して作業します。

#### Page Object File

Page Object file をクリックすると、Sencha WebTestIt はその内容を解析し、*Page Object* と *Elements*タブに表示します。

#### Test File

If you click a test file within your project, the UI will switch into execution mode. The **Execution** and **Log** tabs are brought into view, displaying your configured Endpoints.
プロジェクト内のテストファイルをクリックすると、UI が実行モードに切り替わります。**Execution** と **Log** タブが表示され、設定したエンドポイントが表示されます。

#### Report File

If you select an XML file from within your project’s `reports` folder, the file’s contents will be parsed and displayed in a clear and open summary table within the **Report** tab.
プロジェクトの `reports` フォルダ内のXMLファイルを選択すると、そのファイルの内容が解析され、**Report** タブ内の明確でオープンなサマリーテーブルに表示されます。
