# Adjusting the report template

> オリジナル: [Adjusting the report template](https://docs.sencha.com/webtestit/guides/advanced-topics/adjusting-the-report-template.html)

Sencha WebTestItは、[Handlebars テンプレート](https://handlebarsjs.com/) から構成された高度にモジュール化されたレポートを持っており、デフォルトでは`reports`フォルダ内にある生成されたレポートファイルに移動すると、アプリ内で直接レンダリングされます。

このガイドでは、レポート テンプレートをニーズに合わせてカスタマイズする方法を紹介します。以下の手順に従ってください。

1.  グローバルテンプレートとプロジェクト固有のテンプレートを理解する
2.  プロジェクト固有のオーバーライドの作成
3.  レポートの修正
4.  会社のロゴを追加する
5.  テンプレートが間違っている場合はどうすればいいですか？どのようにして元に戻すことができますか？
6.  他にどのような変数が利用できますか？

### グローバルテンプレートとプロジェクト固有のテンプレートを理解する

レポートはテンプレートから生成されます。デフォルトのテンプレートは、ユーザーのフォルダ内にあるグローバルな `.webtestit` フォルダにインストールされています。すぐに移動するには、アプリメニューからテンプレートフォルダを開いてください。

<img src="https://docs.sencha.com/webtestit/guides/images/global-vs-project.png" width="700px" />

`report` という名前のフォルダにハンドルバーのファイルがいくつか入っています。

![](https://docs.sencha.com/webtestit/guides/images/template.png)

ご覧のように、これらは `report.hbs` と呼ばれる基本テンプレートといくつかの [partials](https://handlebarsjs.com/#partials) に基づいて分割されており、サブセクションを別のファイルに分割することを意味します。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> これらのテンプレートを調整すると、作業しているすべての WebTestIt プロジェクトに対して有効になります。プロジェクトごとにレポートをカスタマイズしたい場合は、プロジェクトのローカルオーバーライドが必要です。

### プロジェクト固有のオーバーライドの作成

プロジェクト内にトップレベルのフォルダ `templates` を作成し、そこにグローバルテンプレートから `report` フォルダをコピーします。

次に、先に作成したテンプレートフォルダがプロジェクト内で実際に使用されるようにします。そのためには、環境設定メニューからプロジェクトの設定を開き、テンプレートの上書きフォルダを探して設定します。

![](https://docs.sencha.com/webtestit/guides/images/preferences.png)

### レポートの修正

ここでどんなテンプレートファイルがあるのか調べてみましょう。`report.hbs` はレポートのレンダリングエントリです。このファイルは `@media print` クエリセレクタに適用されるインラインCSSルールから始まり、PDFに印刷された場合にのみレンダリングされます。レポートの外観をカスタマイズする必要がある場合は、ここを利用するとよいでしょう。例として、`test-summary` ヘッダの背景色を変えてパディングを増やすように調整したいとします。

```html
<style>
  .test-summary {
    background-color: #FB9836; padding: 10px;
  }
 ...
```

保存したら、レポートに移動します。レポートパネルのヘッダーには、新しい調整でレポートを再表示する refresh ボタンがあります。

![](https://docs.sencha.com/webtestit/guides/images/refresh.png)

新しいレポートは以下のようになります。

![](https://docs.sencha.com/webtestit/guides/images/test-new-report.png)

### 会社のロゴを追加する

すべてのレポートヘッダーに会社のロゴを追加したいと想像してみてください。まず、`templates/report` フォルダの中に `images` フォルダを作成します。そこに会社のロゴを `logo.jpg` という名前で配置します。

``` prettyprint
{{> summary.hbs}} {{> report-filter.hbs}}
```

このディレクティブは、レポートのこの特定の場所に `summary.hbs` という名前のパーシャルを含めるように指示します。このファイルを開くと、レポートの最上部にある `test-summary` `div` を含む `div` が見つかります。ロゴを表示するために、このように微調整することができます。

``` html
<style>
   .test-summary__company-logo {
      width: 200px;
      float: left;
  }

  .test-summary__title {
    text-align: right;
  }

 .donut__hole {
    fill: #FB9836;
  }
</style>

<div class="test-summary">
  <div>
    <img class="test-summary__company-logo" src="{{projectRoot}}/reports/images/logo.jpg" />
    <div class="test-summary__title">Summary</div>
  </div>
  ...
```

test-summary__title` を `img.test-summary__title` タグと一緒に新しい div に移動します。

ロゴを含む以前に作成された `images` フォルダへのマッピングを反映させるために、テンプレート変数 `{{projectRoot}}` を使用します。これはプロジェクトのルートを正確に指しています。そこから、最終的な保存先である `templates/report/images/logo.jpg` を連結します。そして、上記の `style` タグに少し余分な CSS を追加することで、結果として以下のようなものが得られます。

![](https://docs.sencha.com/webtestit/guides/images/test-report-two.png)

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> なぜクラスの名前をあんな風に付けているのでしょうか？
  Sencha WebTestIt は [Block-Element-Modifier (BEM) メソドロジー](https://en.bem.info/methodology/quick-start/) を利用して健全で一貫性のある CSS ルールを実現しています。

### テンプレートが間違っている場合はどうすればいいですか？どうすれば元に戻せますか？

プロジェクト固有のテンプレートについて話している限り、テンプレートを新しいもので上書きするか、完全に削除することができます。Sencha WebTestItはプロジェクトのローカルテンプレートを好み、単一の部分テンプレートであっても見つからない場合は、代わりにグローバルバージョンを使用します。

### 他にどんな変数がありますか？

ここまでは、レンダリング中にプロジェクトのルートへのアクセスを得るために `{{projectRoot}}` 変数を使用してきました。しかし、他にもありますので、以下のリストをチェックしてください。

#### endpoints

実行されたエンドポイントごとに以下のプロパティを持つインターフェイスが含まれています。

``` prettyprint
export interface EndpointData {
  name: string;
  time: number;
  timestamp: Date;
  readableTimestamp: string;
  testSuites: TestSuite[];
  failures: number;
  readableDuration: string;
  testCaseCount: number;
  testSuiteCount: number;
  isVisible: boolean;
  skipped: number;
  iconPath: string;
  id: string;
  headless?: boolean;
}
```

#### testSuites

`endpoints` の中から、特定のエンドポイントで実行された `TestSuite` にアクセスすることができます。

``` prettyprint
export interface TestSuite {
  errors: string;
  expanded: boolean;
  failures: string;
  id: string;
  endpoint: Endpoint;
  name: string;
  skipped: string;
  testCases: TestCase[];
  tests: string;
  time: string;
  readableDuration: string;
  timestamp: Date;
  readableTimeStamp: string;
  extraElements?: ExtraElements[];
  isVisible: boolean;
}
```

#### testCases

`testSuites` の中から、特定のテストスイートで実行される `TestCase` にアクセスすることができます。

``` prettyprint
export interface TestCase {
  className: string;
  expanded?: boolean;
  fileName: string;
  filePath?: string;
  lineNumber?: number;
  failure: boolean | TestFailure;
  loading?: boolean;
  skipped: boolean;
  endpointName?: string;
  endpointBrowserName?: string;
  name: string;
  time: string;
  id: string;
  jiraIssue?: JiraData;
  testrailData?: { case_id?: number, suite_id?: number };
  testCaseNotFound?: boolean;
  screenshotFound?: boolean;
  extraElements?: ExtraElements[];
  formattedTimestamp?: string;
  screenshot?: string;
  isVisible: boolean;
}
```

#### extraElements

extraElementValue ヘルパーを使用して、レポートから追加のエレメントの値にアクセスします。
2つの引数を受け付けます: extraElements 配列とエレメントの名前です。

``` prettyprint
export interface ExtraElements{
  name: string;
  value: string | null;
  attributes?: ExtraAttributes;
}
```

#### extraAttributes

``` prettyprint
export interface ExtraAttributes { [name: string]: string | null; }
```

#### Jira issue

Jira issue は、指定されたテストケースにリンクされた Jira issue の詳細を表示します。

``` prettyprint
export interface JiraData {
  id: string;
  key: string;
  isCompleted: boolean;
  testCaseNotFound?: boolean;
}
```

#### failure

``` prettyprint
export interface TestFailure {
  message: string;
  trace: string;
  attributes?: ExtraAttributes;
}
```

#### jiraAvailable

これは、テストが Jira に接続されていて、資格情報を設定しているかどうかを示します。

#### isVisible

選択されたフィルターに基づいて計算されます。

#### @root

テンプレートからルート変数にアクセスするには @root (etc: `@root.jiraAvailable`, `@root.projectRoot`...) を使用します。
