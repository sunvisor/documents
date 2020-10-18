# WebDriver の実行についての説明

> オリジナル: [The WebDriver execution explained](https://docs.sencha.com/webtestit/guides/running-tests/webdriver-execution.html)

Selenium WebDriverは、自動化フレームワークをドライバと通信させるJSON-on-Wireプロトコル(RESTサービスに匹敵)を利用します。ドライバは、Web ブラウザや最終的には Web サイトやアプリケーションを自動化するインターフェイスや実行ファイルです。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> WebDriverのリモートコントロールインターフェースは、現在[ W3C による勧告状態](https://www.w3.org/TR/webdriver1/) にあります。デスクトップ、タブレット、モバイルの主要なWebブラウザはすべてWebDriverをサポートしています。

テストを実行すると、Selenium WebDriver は IP 経由で設定されたホストに接続します。自動化フレームワークは、Web サイトやアプリの DOM から情報を取得し、実行するアクションを返します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> テスト (すなわち、エレメントを評価し、アクションをオーケストレーションする) を実行しているコンピュータは、必ずしも SUT を自動化するブラウザのホストとして機能しているコンピュータとは限りません。

### ローカルおよびリモート実行

ウェブサイトやアプリケーションのテストとは、その仕様に沿った正しい動作を保証することを意味します。ウェブベースの技術に関しては、さまざまなプラットフォームやブラウザ上での動作を確認する必要があります。インターネットには標準が存在しますが、その実装は常に個々のブラウザメーカーに依存しています。さらに、さまざまなブラウザの新バージョンが短いサイクルでリリースされ、それに合わせて実装が変更されています。テストの設計に使用するコンピュータは、通常、多大な努力をしなければ、同じブラウザの複数のバージョンを検証することはできません。

どのプラットフォーム/OSを使用しているかにかかわらず、単一のデバイス上で利用可能なすべてのブラウザをテストする方法もありません。したがって、最もシンプルな解決策は、ウェブサイトやアプリを自動化するために、複数のプラットフォームとブラウザのバージョンを手元に用意しておくことです。Selenium WebDriver はネットワークを介してブラウザドライバと通信するので、ローカルとリモートの実行をシームレスに切り替えることができます。異なる環境で同じテストを実行することができます。

### Selenium サーバー

**Selenium サーバー**は、テスト専用のマシン上で動作するホストアプリケーションです。デフォルトの設定では、ポート 4444 をリッスンし、テストを実行するための自動化フレームワークからのリクエストを受け入れます。これは、テストと Web ブラウザの間のブリッジとして機能します。複数のマシンでローカルネットワーク環境を設定し、個別にターゲットにすることができます。テストをローカルで実行すると、Sencha WebTestIt はローカルの Selenium サーバー インスタンスを起動し、それに接続します。

### Selenium グリッド

[Selenium グリッド](https://github.com/SeleniumHQ/selenium/wiki/Grid2) は、Selenium サーバーのグループを *hub* と呼ばれる単一のインターフェースに抽象化する技術です。グリッドとは、テストの準備ができている OS/ブラウザの組み合わせを持つコンピュータのネットワークです。Selenium グリッドを使うと、多数の環境でテストをスケールアップしたり、並列化することができます。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
>
> [独自のグリッドを設定](https://www.seleniumhq.org/docs/07_selenium_grid.jsp#installation) することもできますし、**グリッドプロバイダ**に接続することもできます。グリッドプロバイダとは、あなたのために Selenium のグリッドを運用し、洗練されたテスト風景を表現するために必要なハードとソフトウェアをすべて引き受けてくれるサービスです。

Sencha WebTestIt はグリッドプロバイダとして [Sauce Labs](http://saucelabs.com/) と統合されています。Sauce Labsのユーザー名とアクセスキーを提供して、利用を開始することができます。別のプロバイダを使用している場合は、機能を通して必要な認証を手動で設定することができます。

### エンドポイント

Sencha WebTestIt では、テストを実行するための複数の設定を定義することができます。これらのオプションのセットは **endpoint** に保存されます。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> エンドポイントは、設定の集合を表すJSON構造体です。これらの設定は主にプラットフォーム、ブラウザ、ブラウザのバージョンで構成されています。ただし、これらの属性を絞り込んだり、ブラウザ固有のオプションを設定したりすることもできます。

### ケイパビリティについて

エンドポイントは、テスト対象のシステムの設定とオプションのセットをホストします。
これらの構成は、**ケイパビリティ**と呼ばれています。
すべてのテストシナリオは、いくつかの特定のテスト環境で実行されなければなりません。
ユースケースに応じて、必要なケイパビリティと望まれるケイパビリティを区別することができます。
必要なケイパビリティは、テストスクリプトで使用する環境を定義するのに役立ちます。
自動化フレームワークは、望まれるケイパビリティを満たすように WebDriver に問い合わせを行い、それに応じてターゲット環境を設定します。
例えば、ブラウザとして Google Chrome を指定した場合、ターゲットホストは Chrome のインスタンスを検索し、テストを自動化するために Chrome を起動します。
ローカルまたはリモートホストでケイパビリティが利用できない場合、テストの実行は失敗します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> Sencha WebTestItでは、検出されたWebDriversとブラウザインスタンスのみでローカルエンドポイントを設定することができます。これにより、ローカルで作業する際の設定エラーを回避することができます。

Selenium WebDriver がサポートするすべてのケイパビリティの概要は[こちら](https://github.com/SeleniumHQ/selenium/wiki/DesiredCapabilities) で確認できます。

`{ "platform": "macOS 10.12", "browserName": "safari", "version": "10.0" }`

macOS 10.12およびSafariバージョン10でテストする機能を定義するための例

Seleniumグリッド上で自動化する場合、ケイパビリティの処理は通常、より柔軟になります。
複数のデバイスがハブに接続されている場合、環境はフィルタリングされ、要求されたケイパビリティを満たすように絞り込まれます。
ケイパビリティのためのグリッドの振る舞いは、多くの柔軟性を与えてくれます。
Web サイトやアプリケーションがグリッドに対してすべてのブラウザで動作するようにするには、ブラウザを定義し、ハブに対してバージョンに関係なく実行する必要があります。
環境要件に加えて、ターゲット環境の設定を定義するためにケイパビリティを使用することもできます。
その中には、拡張機能を有効にしたり無効にしたりするオプションのように、ブラウザ固有のものもあります。
ブラウザ固有の機能の概要は[こちら](https://github.com/SeleniumHQ/selenium/wiki/DesiredCapabilities#browser-specific-capabilities) を参照してください。


### Sencha WebTestItでエンドポイントを管理する

エンドポイントは、コンピュータ上の単一のブラウザインスタンスをターゲットにすることも、リモートにあるマシンをターゲットにすることもできます。Sencha WebTestIt は、作成して管理できる様々なタイプのエンドポイントを区別しています。プロジェクトに設定されたエンドポイントは **Execution** タブで確認できます。

![](https://docs.sencha.com/webtestit/guides/images/managing-endpoints.png)

1. **エンドポイントファイル**: 現在選択されているエンドポイント設定ファイル。プロジェクトごとに複数のエンドポイントファイルを作成して編集することができます。

2. **アクションメニュー**: 新しいエンドポイントファイルを作成したり、現在のエンドポイントファイルを削除したりすることができます。

3. **+ ボタン**: ここをクリックして、現在の設定ファイルに新しいエンドポイントを追加します。

4. **リストセレクター**: ここをクリックして、実行するすべてのエンドポイントを選択または選択解除します。

5. **エンドポイント**: 一つのエンドポイント。

6. **エンドポイントオプション**: エンドポイントを編集または削除するには、ここをクリックします。

7. **Run current test file**: このボタンをクリックすると、選択したすべてのエンドポイントで現在アクティブなテスト・ファイルが実行されます。このボタンは、テスト・ファイルが開かれている場合にのみ有効になります。

*8* **Run all test files**: このボタンをクリックすると、選択したすべてのエンドポイントでプロジェクト内のすべてのテストファイルが実行されます。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> エンドポイントは `.endpoints.json` で終わるファイルに格納されます。エンドポイントはプロジェクト構造のどこにでも置くことができます。Sencha WebTestIt はエンドポイントを見つけて **Execution** タブにリストアップしてくれます。

### エンドポイントの追加と編集

プロジェクトに新しいエンドポイントを追加するには、**Execution** タブの **+** ボタンをクリックします。エンドポイントを編集する場合は、リスト上のエンドポイントの横にある歯車のアイコンをクリックします。ポップアップダイアログで、作成または編集するエンドポイントのタイプに切り替えます。以下に説明するようにフィールドを記入し、**Save endpoint** をクリックして変更を保存します。

#### ローカルエンドポイント

テストを書いているのと同じコンピュータ上にテスト用のローカルエンドポイントを作成します。Sencha WebTestItはインストールされているブラウザを自動的に検出します。また、adbを介してAndroidデバイスに接続することもできます。

> ![](https://docs.sencha.com/webtestit/guides/images/reference-icon.png) **Reference**
> 
> モバイル端末でテストしたい方はこちらから ["Android端末の接続方法"](../AdvancedTopics/HowToConnectAnAndroidDevice.md) をご覧ください。

<img src="https://docs.sencha.com/webtestit/guides/images/local_endpoint.png" height="700px" />

<p></p>
<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>フィールド</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Name</td>
<td>エンドポイントに固有の名前を指定します。<br />
    これはエンドポイントのリストに表示され、エンドポイントの識別に役立ちます。</td>
</tr>
<tr class="even">
<td>Platform &amp; Browser</td>
<td>エンドポイントは、テストするプラットフォーム（オペレーティングシステム）とブラウザによって定義されます。<br />
    デスクトッププラットフォームとモバイルプラットフォームを選択することができます。<br />
    Sencha WebTestIt はインストールされているブラウザを自動的に評価し、`Browser` ドロップダウンにリストアップします。<br />
    注意: モバイルテストを行うには、Android Developer Toolsとadbがコンピュータにインストールされている必要があります。</td>
</tr>
<tr class="odd">
<td>Desired capabilities</td>
<td>環境設定の追加オプションを指定したい場合は、このフィールドにJSONコードとして入力することができます。</td>
</tr>
</tbody>
</table>

### リモートブラウザと Selenium グリッド

ネットワーク上にSelenium ServerやGridを実行しているテストマシンがある場合、このオプションを使用してテストを接続して実行することができます。これにより、コンピュータにインストールされているものとは別のオペレーティングシステムやブラウザのバージョンにアクセスすることができます。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> 1台のリモートマシンに接続する場合は、**Selenium Grid** オプションを選択し、末尾の`/wd/hub`を省略してコンピュータに到達するアドレスを指定します。

<img src="https://docs.sencha.com/webtestit/guides/images/remote_endpoint.png" height="700px" />
<p></p>
<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>フィールド</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Name</td>
<td>エンドポイントに固有の名前を指定します。<br />
    これはエンドポイントのリストに表示され、エンドポイントの識別に役立ちます。</td>
</tr>
<tr class="even">
<td>Provider Selector</td>
<td>接続するハブのプロバイダを選択します。<br />
    Sauce Labsでは、追加の認証が必要です。<br />
    プロバイダーとして Sauce Labs を選択した場合は、認証情報の入力を求められます。</td>
</tr>
<tr class="odd">
<td>Desired capabilities</td>
<td>設定したい環境を選択します。<br />
    Sencha WebTestItは、*Fetch available capabilities*ボタンをクリックすると、利用可能なオプションを自動的に埋めます。</td>
</tr>
</tbody>
</table>

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> リモートインスタンスのURLと認証情報を入力したら、**Fetch available capabilities** ボタンをクリックします。Sencha WebTestIt は、利用可能な OS/ブラウザの組み合わせをエンドポイントに照会します。

グリッド上で並行して実行したい場合は、同じ URL に対して異なるケイパビリティを持つ複数のエンドポイントを定義することができます。

#### カスタムエンドポイント

リモートシステムに期待する機能のセットを手動で提供することで、カスタムエンドポイントを定義することができます。カスタムエンドポイントを使用すると、必要に応じてリモートマシンを柔軟に構成することができます。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> Selenium グリッドの中に異なるマシンのセットがあることを想像してみてください。ハブをFirefox上で実行するように要求すると、Firefoxのインスタンスがアクティブな最初の利用可能なマシンが使われます。ケイパビリティを指定していないので、OS と OS のバージョンは考慮されません。実際のテスト実行が Windows 10 または Windows 7 マシンで実行されるかどうかは保証されません。

<img src="https://docs.sencha.com/webtestit/guides/images/custom_endpoint.png" height="700px" />
