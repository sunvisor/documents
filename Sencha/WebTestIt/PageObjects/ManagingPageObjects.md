# Sencha WebTestIt でページオブジェクトを管理する

Sencha WebTestItは、ページオブジェクトの作成と管理のための強力なツールを提供します。Sencha WebTestItで直接*ページオブジェクトアクション*を生成するためにエレメントを使用することができます。ページオブジェクトパターンを使用して、ウェブアプリケーションの動作をシンプルなオブジェクトにモデル化し、"クリック "や "sendKeys "などのページオブジェクトアクションを実行して、論理的なユーザーインタラクションを構築することができます。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> 名前にもかかわらず、ページ オブジェクト パターンは必ずしも Web サイトやアプリ内の「ページ」全体に対応しているわけではなく、ヘッダー、メニュー、コンテンツ エリアなどのページ上の重要なコンポーネントに対応しています。ページ オブジェクトを作成する際には、ページ全体や複数のページではなく、これらのコンポーネントごとに個別に作成してください。

### ページオブジェクトの作成

ページオブジェクトは、プロジェクト内のクラスで、エレメントとそれらのエレメントを利用した一連のアクションを含んでいます。新しいページオブジェクトを作成するには、**Project**タブの`test/pageobjects`フォルダを右クリックし、コンテキストメニューから**New \> Page Object**を選択します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> デフォルトのプロジェクト構造を維持することをお勧めしますが、[**Special Headers**](./advanced-topics/about-page-object-and-test-file-headers.html) を維持していれば、好きなフォルダにページオブジェクトを作成することができます。

または、キーボードコード **Ctrl+N \> Ctrl+P** を使用することもできます。Sencha WebTestIt は新しい Page Object ファイルを作成し、名前を付けるのを待ちます。新しいクラスを作成するには **ENTER** をクリックします。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> たとえば、Java や Typescript での `DetailPagePo` や Python の `detail_page_po` のように、`Po` を接尾辞に持つページオブジェクトクラスを作成するのは良い習慣です。

各ページオブジェクトは，テンプレートシステムから派生したデフォルトの関数 `open` と `getTitle` で生成されます。

**Project**タブで選択したページオブジェクトが**アクティブページオブジェクト**になります。Sencha WebTestIt は、現在アクティブなページオブジェクトクラスに応じて、ユーザーインターフェースを適応させ、様々なタブに情報を表示します。

### Page Object タブ

**Project**タブから Page Object を選択すると、Sencha WebTestIt は**Page Object**タブを開きます。**Page Object** タブを使用して、アクティブなページオブジェクト内のアクションをナビゲートします。

![](https://docs.sencha.com/webtestit/guides/images/pageobject_tab.png)

1. **ページオブジェクト名**: 現在アクティブなページオブジェクトの名前を表示します。

2. **アクションリスト**: ページオブジェクトの *public* メソッドとその戻り値を表示します。`</>` をクリックすると、コードエディタで定義まで直接スクロールすることができます。

### ページオブジェクトのアクションを作成する

ページオブジェクトアクションを素早く作成するには、Sencha WebTestIt のテンプレートを利用することができます。

1.  アクティブなページオブジェクトで **Elements** タブを探します。

2.  アクションのベースとなるエレメントを **Code** タブにドラッグします。

3.  新しいメソッドを挿入したい場所でマウスを離します。

4.  ポップアップメニューから、メソッドをベースにするアクションを選択します。

5.  Sencha WebTestIt はアクションを反映したメソッドを作成します。

6.  アクションを自由に編集し続けることも、新しくメソッドを生成するためにさらにエレメントをドラッグすることもできます。

<img src="https://docs.sencha.com/webtestit/guides/images/actions.png" width="700px" />

この機能のおかげで、ページオブジェクトを定義するときに書かなければならないコードの量を減らすことができます。Sencha WebTestIt は、[Selenium IDE](https://www.seleniumhq.org/selenium-ide/) で使用されている Selenese コマンドのサブセットで、Webアプリケーションに関連する様々なエレメント、アクション、およびその他多くの機能性をテストするのに役立ちます。

### アクション

アクションは、ウェブサイトやアプリのエレメントとのインタラクションのためのシンプルな出発点です。以下のアクションは、Sencha WebTestIt のポップアップメニューから利用できます。

- Do
- Perform mouse
- Element by index
- Get
- Is
- Is not

#### Do

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>名前</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Click on element</td>
<td>与えられたエレメント上でクリックを実行する標準的な Selenium `click()` エレメントメソッド。</td>
</tr>
<tr class="even">
<td>Double click on element</td>
<td>Seleniumの `Actions()` クラスからマウスの `doubleclick()` メソッドを呼び出します。</td>
</tr>
<tr class="odd">
<td>Type into element</td>
<td>sendKeys(text)` メソッドを呼び出し、エレメントへの入力をシミュレートするために入力文字列を送信することができます。</td>
</tr>
<tr class="even">
<td>Clear and type into element</td>
<td><br /> 最初に `clear()` メソッドを呼び出して、入力フィールドをクリアします。
    これに続くのは `sendKeys(text)` メソッドで、入力文字列をエレメントに送信し、エレメントへの入力をシミュレートします。</td>
</tr>
<tr class="odd">
<td>Hover over element</td>
<td>マウスカーソルを与えられたエレメントの中心に移動します。 エレメントの上にカーソルを置くアクションをシミュレートします。</td>
</tr>
<tr class="even">
<td>Submit form</td>
<td>指定されたエレメントがフォームまたはフォーム内のエレメントである場合、リモートサーバーへの送信アクションを実行します。</td>
</tr>
<tr class="odd">
<td>Switch to the iframe element</td>
<td>配置された iframe に切り替えて、その iframe 内に配置されたエレメントに対してアクションを実行できるようにします。</td>
</tr>
<tr class="even">
<td>Scroll to element</td>
<td>ユーザーが指定したエレメントをブラウザウィンドウの可視領域にスクロールできるようにします。</td>
</tr>
<tr class="odd">
<td>Scroll to element with offset</td>
<td>オフセット座標（x, y）として与えられたピクセル数だけスクロールします。<br>負の値は左(-x)または上(-y)を意味します。</td>
</tr>
</tbody>
</table>

#### Perform mouse[](#page-objects-_-managing-page-objects_-_perform_mouse)

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>名前</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Down</td>
<td>マウスのクリックとホールドの動作をシミュレートします。<br>
    マウスカーソルを指定されたエレメントの中心に移動し、マウスの左ボタンをクリックしたまま（離さずに）保持します。</td>
</tr>
<tr class="even">
<td>Up</td>
<td>与えられたエレメントの中央で、クリックされたマウスボタンのリリースをシミュレートします。</td>
</tr>
<tr class="odd">
<td>Move</td>
<td>マウスカーソルを指定されたエレメントの中心に移動します。</td>
</tr>
<tr class="even">
<td>Over</td>
<td>マウスカーソルを指定されたエレメントの上に移動させることで、マウスオーバーイベントをシミュレートします。</td>
</tr>
<tr class="odd">
<td>Out</td>
<td>マウスカーソルを現在の(X,Y)位置から、指定されたオフセット分だけ離れた位置に移動します。</td>
</tr>
</tbody>
</table>

#### Element by index

**Element by index** ポップアップメニューの項目にカーソルを合わせると、列挙子ベースのアクションのセットが表示されます。これらは、リストやその他の類似したエレメントのコレクションを特に対象としたメソッドを生成します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> 列挙子ベースのエレメントの例としては、[Java](https://github.com/extjs/RxSe-java-demoshop) や [TypeScript](https://github.com/extjs/RxSe-ts-demoshop) でダウンロードできる Demo Webshop の商品リストがあります。各商品はそれぞれ独立したコンポーネントとして表示され、アイテムの数は可変です。このような場合、IDベースのセレクタでは実現できないことが多いです。

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>名前</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Click nth element</td>
<td>リストの中から n 番目の項目をクリックします。<br />
    配置されているすべてのエレメントを格納し、そのエレメントのインデックスを使用してリストから選択されたエレメントをクリックします。</td>
</tr>
<tr class="even">
<td>Get nth element</td>
<td>WebElementリストからn番目の項目を返します。<br />
    Webエレメントのリストを作成し、エレメントのインデックスを用いてリストからエレメントを選択して返します。</td>
</tr>
<tr class="odd">
<td>Get nth element’s text</td>
<td>リストから n 番目の項目のテキストを返します。<br />
Web エレメントのリストを作成し、そのエレメントのインデックスを使用してリストから選択されたエレメントのテキストを返します。</td>
</tr>
</tbody>
</table>

Selenese と共通の **Assert** や **Verify** コマンドの代わりに、Sencha WebTestIt には **Get**/**Is**/**Is not** コマンドを導入し、エレメントの値やテキスト、属性などのチェック対象のデータを取得するようにしました。これらの新しいコマンドでは、取得したデータはテストから分割されますが、これはページオブジェクトを扱う際に推奨される方法です。 つまり、ページオブジェクトには **Get**/**It**/**Is not** コマンドが含まれており、テストがデータを検証する際にデータを取得することができます。


#### Get

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>名前</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Element as variable</td>
<td>選択したエレメントを WebElement として返し、カスタムアクションやメソッドを追加できるようにします。</td>
</tr>
<tr class="even">
<td>Element’s text</td>
<td>`.getText()` メソッドを用いて可視エレメントの内部テキストを含む文字列を返します。</td>
</tr>
<tr class="odd">
<td>Element’s value</td>
<td>`getAttribute("value")` メソッドを用いて値属性 "elements" を返します。</td>
</tr>
<tr class="even">
<td>Element’s attribute</td>
<td>`getAttribute()`メソッドを用いて、値や型などの特定のエレメント属性を取得することができます。</td>
</tr>
<tr class="odd">
<td>Element’s selected value</td>
<td>与えられた ("select" 型の) エレメントから最初に選択されたオプションを返します。<br />
    WebElement が "select" 型でない場合は例外がスローされます。</td>
</tr>
<tr class="even">
<td>Element’s selected label</td>
<td>Seleniumの `Select()`クラスを使用して、指定されたselect型エレメント、例えばselectリストから最初に選択されたオプションのラベル（テキスト）を含む文字列を返します。<br />
    WebElement が "select" 型でない場合は、例外がスローされます。</td>
</tr>
</tbody>
</table>

#### Is

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>名前</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Element checked</td>
<td>与えられたエレメントがチェックされているかどうかを判定します。ブール値を返します。<br />
    チェックボックス、ラジオボタン、選択タグのオプションなどの入力エレメントにのみ適用されます。</td>
</tr>
<tr class="even">
<td>Element editable</td>
<td>与えられたエレメントが編集可能かどうかを判断します。<br />
エレメントがウェブページ内で有効になっているかどうかをチェックし、"readonly" 属性がnullかどうかをチェックした後、ブール値を返します。
</td>
</tr>
<tr class="odd">
<td>Element present</td>
<td>与えられたエレメントがWebページ上に存在するかどうかを判断します。<br />
指定されたセレクタが見つかったエレメントの数が0より大きいかどうかを確認した後、ブール値を返します。</td>
</tr>
<tr class="even">
<td>Element visible</td>
<td>与えられたエレメントがWebページ上で表示されているかどうかを判断します。<br />
    エレメントに対して `isDisplayed()` メソッドを使用した後、ブール値を返します。</td>
</tr>
</tbody>
</table>

#### Is not[](#page-objects-_-managing-page-objects_-_is_not)

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>名前</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Element checked</td>
<td>与えられたエレメントがチェックされていないかどうかを判断します。<br />
    論理値を返します。チェックボックス、ラジオボタン、選択タグのオプションなどの入力エレメントにのみ適用されます。</td>
</tr>
<tr class="even">
<td>Element editable</td>
<td>与えられたエレメントが編集可能でないかどうかを判断します。<br />
そのエレメントがウェブページ内で無効化されているか、"readonly "属性が null かどうかを確認した後、ブール値を返します。
</td>
</tr>
<tr class="odd">
<td>Element present</td>
<td>与えられたエレメントがWebページ上に存在しないかどうかを判断します。<br />
    指定されたセレクタが見つかったエレメントの数が0であるかどうかを確認した後、ブール値を返します。</td>
</tr>
<tr class="even">
<td>Element visible</td>
<td>与えられたエレメントがWebページ上で表示されていない（隠されている）かどうかを判断します。<br />
    エレメントに対して `isDisplayed()` メソッドを使用した後、ブール値を返します。</td>
</tr>
</tbody>
</table>

### ページオブジェクトの構造化

テストやページオブジェクトを設計する際には、[テストのメンテナンス](../Maintenance/Introduction.md) を念頭に置いておくことを強くお勧めします。[ページオブジェクトパターン](../AdvancedTopics/StructuringYourPageObjects.md)を実装することをお勧めします。

ページオブジェクトは、あなたのウェブサイトやアプリに固有のエレメントやアクションをカプセル化する大きな可能性を提供します。エレメントが壊れた場合（セレクタがエレメントを返さないなど）、適切なページオブジェクトでセレクタを編集することができます。テストケースは、これらのアクションをオーケストレーションし、結果を評価する責任があります。

そのため、エレメントは専用のページオブジェクトで非公開にしておくのが良い方法です。テスターであるあなたはアクションのみに興味を持っていますが、それは公開されているべきです。

リダイレクトやサイトの変更についてはどうでしょうか？あなたがフォームにデータを入力して送信すると、クライアントが別のページにリダイレクトされるとします。この場合、適切なアクションは *new* ページオブジェクトを返すべきです。

テストに必要なページオブジェクトの数は、ウェブサイトやアプリの構造に大きく依存します。全体の構造を分析し、ウェブサイトやアプリをコンポーネントに分解することをお勧めします。そして、特定したコンポーネントごとにページオブジェクトを作成してください。最初から完璧でなくても心配しないでください。ページオブジェクトをリファクタリングする可能性は常にあります。

ページオブジェクトパターンの詳細については、[ここ](https://www.martinfowler.com/bliki/PageObject.html) を参照してください。

