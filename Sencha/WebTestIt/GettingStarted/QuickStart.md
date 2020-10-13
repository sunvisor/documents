クイックスタート: 最初のテストを作成する
================================

このチュートリアルでは、Sencha WebTestItを使って初めてのUIテストを作成します。このチュートリアルでは、DemoShopのサンプルの一部をステップバイステップで自動化していきます。

このチュートリアルのリソースは、TypeScript、Java、Python で用意されています。

### プロジェクトの構造

まだ新しいプロジェクトを作成していない場合は、以下の手順に従ってください。

1. **Welcome tab** の **New Project** リンクをクリックするか、**File \> New Project** を選択します
2.  **Create new project** ダイアログで、プロジェクトの名前と場所を指定し、使用するスクリプト言語を選択します。
3.  **Save** をクリックすると、Sencha WebTestIt はプロジェクトを作成し、テストを実行するために必要なコンポーネントをセットアップします。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> プログラミング言語の選択は、あなたのために設定されている自動化フレームワークを定義します。TypeScript を選択すると、Sencha WebTestIt は Protractor 環境を生成します。代わりに Java を選択すると、TestNg アプリケーションがセットアップされます。Python を選択した場合、Sencha WebTestIt は unittest 環境を生成します。

![](https://docs.sencha.com/webtestit/guides/images/project-tab.png)

**Project** タブで、新しく生成されたプロジェクトの構造を確認できます。Sencha WebTestIt は、テストスイートとページオブジェクト用のフォルダを自動的に作成します。プロジェクトのルートディレクトリにあるファイルは、テストの設定と実行環境の設定に使用されます。プロジェクト内の注目すべき設定ファイルは以下の通りです。

|Filename |Description
|----     |----
|`.editorconfig` |Sencha WebTestIt コードエディタのコードスタイル設定が含まれています。[editorconfig の詳細はこちら](https://editorconfig.org/)
|`*.endpoints.json` |テストを実行するエンドポイントの情報を格納します。[エンドポイントについての詳細はこちらをご覧ください](../RunningTests/WebdriverExecution.md)
|`*.endpoints.json` |Sencha WebTestItテストプロジェクトのプロジェクト固有のオプションが含まれています。<br />ここで定義されているすべての設定は、グローバル設定とは対照的に、単一のプロジェクトを対象としています。これはメインメニュー **File \> Preferences \> User Settings** からアクセスできます

### テスト中のサイトを分析する (SUT)

このチュートリアルでは、デモショップの自動化を行います。サイトは[ここ](https://demoshop.webtestit.com/)にあります。すぐにでも見に行ってください。

デモショップのメインページは2つの部分で構成されています。

1. メニューと検索フィールドとヘッダーエリア
2. 商品が掲載されているコンテンツエリア

![](https://docs.sencha.com/webtestit/guides/images/shop-tutorial.png)

デモショップには他にも様々なページがあります。このチュートリアルでは、商品を検索してカートに入れてください。テストケースを作成してみましょう。

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>#</strong></th>
<th><strong>タイトル</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>アイテムを探す</td>
<td>GIVEN デモショップが表示される<br />
WHEN ユーザーが検索項目として「スーパー」と入力してEnterを押すと<br />
THEN スーパークールグー」の詳細ページが表示される<br />
そして、商品名は「Super Cool Goo」に等しい<br />
そして価格は €1,500.00 である</td>
</tr>
<tr class="even">
<td>2</td>
<td>アイテムをカートに追加する</td>
<td>GIVEN スーパークールグー」の詳細ページが表示される<br />
WHEN ユーザーが「Add to cart」ボタンをクリックすると<br />
THEN 確認メッセージが表示される<br />
そして、アイテムカウンターには1が表示される<br />
そして、カートエリアの小計は、製品の価格と同じになる</td>
</tr>
</tbody>
</table>

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> テストケースの説明には、[*Gherkin syntax*](https://martinfowler.com/bliki/GivenWhenThen.html) を使用しています。すべての文には、以下の3つのキーワードが含まれています。
> 
>   - **GIVEN** は、現在の状態を記述します。*precondition*ともいいます。
>   - **WHEN** は、ユーザーの *action* を記述します。
>   - **THEN** は、これらのアクションの結果を記述します。*assertions*ともいいます。

これらのキーワードの組み合わせを *scenario* と呼びます。より複雑なシナリオを作成するために、論理演算子(および/または)を使用して、ブロック内の複数の文をリンクすることができます。

テストケースを設計するとき、アプリケーションの期待される動作のみを記述します。これからの逸脱は予期せぬものであり、テストの失敗として扱われます。

では、Sencha WebTestItを使って、これらのテストケースを自動化してみましょう!

### ページオブジェクトの作成

テストケースに必要なすべての要素にアクセスするには、それらの要素のセレクタを作成する必要があります。セレクタはページオブジェクトにまとめられています。セレクタについての詳細は[こちら](./automation/elements.html) を参照してください。
まだページオブジェクトファイルを持っていないので、ページオブジェクトを作成する必要があります。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> ページオブジェクトはセレクタだけでなく、シミュレートされたユーザ入力も含みます。実際には、ページオブジェクトのアクションだけを個々のテストケースに公開することを強く推奨します。これは*ページオブジェクトパターン*と呼ばれ、[ここで説明されています](../AdvancedTopics/StructuringYourPageObjects.md)。

1.  プロジェクトツリーツールバー内の **New Page Object file** をクリックします。Sencha WebTestIt はプロジェクトの`pageobjects`フォルダに新しいファイルを作成し、名前を付けるように促します。
2. あるいは、`pageobjects` フォルダを右クリックし、コンテキストメニューからファイルを作成します。ファイルの名前は、TypeScript を使っている場合は `HeaderPo.ts`、Javaを使っている場合は `HeaderPo.java` とし、Pythonを使っている場合はスネークケース (アンダースコア文字、文字と文字の間はスペースではなくアンダースコア) を使う必要があるので、ページオブジェクトの名前は `header_po.py` とします。

    ![](https://docs.sencha.com/webtestit/guides/images/page-objects-tab.png)

3.  新しく作成されたファイルには、`open` と `getTitle` の2つのデフォルト関数が付属しています。ページオブジェクトには、DemoShop のヘッダーに必要なセレクタとアクションがすべて含まれています。

> ![](https://docs.sencha.com/webtestit/guides/images/reference-icon.png) **Reference**
> 
> よりメンテナンスしやすいテストのためには、ウェブサイトやアプリの構造を分析し、ページオブジェクトに分類することが重要です。ページオブジェクトについての詳細は、[このガイドの第3章](../PageObjects/Introduction.md) で読むことができます。

では、以下の手順に従って、詳細ページ用の別のページオブジェクトを作成します。

1.  *Project* タブの `pageobjects` フォルダを右クリックし、コンテキストメニューから **New \> Page Object File** を選択します。
2.  新しいページオブジェクトの名前は、TypeScriptを使用している場合は `DetailPagePo.ts`、Javaを使用している場合は `DetailPagePo.java` とします。Pythonではスネークケースを使用しているので、新しいページオブジェクトの名前を `detail_page_po.py` とします。

### ページオブジェクトのアクションを作成する

すべてのセレクタがセットアップされたので、オートメーションフレームワークが実行するアクションを書くことができます。

Tシャツを検索することから始めましょう。現実の世界では、ユーザーは検索入力をクリックして検索語を入力します。テストの自動化は本質的にユーザーをシミュレートしていることを覚えておいてください。だからこそ、まったく同じことをすることになるのです。

  - **Select** プロジェクトの `pageobjects` フォルダから `HeaderPo` (`header_po`) ファイルを取り出します
  - **Drag** **Code**タブに `searchInput` (`_search_input`) を入力します
  - **Click** ポップアップメニューの**Type into element** オプションを選択します
  - **Name** 関数 `insertSearchText` (Pythonでは `insert_search_text`)

生成されたメソッドは、すぐに始められるシンプルな構成で、書かなければならないコードの量を節約することができます。

テキストを送信したら、ENTER キーをクリックして検索を開始します。これを行うために、Sencha WebTestIt は `sendKeys` (`send_keys`) パラメータを自動的にメソッドに追加します。コードは以下のようになっているはずです。

```java
public void insertSearchText(String text) {
  this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.searchInput)).sendKeys(text, Keys.RETURN);
}
```

```python
def insert_search_text(self, text):
    self.wait.until(EC.visibility_of_element_located(self._search_input)).send_keys(text)

    return self
```

```typescript
public async insertSearchText(text: string): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.searchInput)), browser.allScriptsTimeout, this.searchInput.toString());
  await element(this.searchInput).sendKeys(text, Key.ENTER);

  return this;
}
```

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> 生成されたページオブジェクトのアクションは、テンプレートから作成されます。ウェブサイトやアプリケーションの要件に応じてテンプレートをカスタマイズすることができます。詳しくは、[このガイドの第3章でテンプレートをカスタマイズする](../PageObjects/ManagingPageObjects.md)を参照してください。

`HeaderPo` ページオブジェクト内の残りの要素からテキストを取得します。上記の操作を繰り返しますが、今回はポップアップメニューの **Get element's text** オプションを使用してテキスト文字列を返すか、以下のソースコードをコピーしてください。

```java
public String getCartAmount() {
  return this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.cartAmount)).getText();
}

public String getCartCount() {
  return this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.cartCount)).getText();
}
```

```python
def get_cart_amount(self):
    cart_count_count = self.wait.until(EC.visibility_of_element_located(self._cart_count)).text

    return cart_count_count

def get_cart_count(self):
    cart_count_text = self.wait.until(EC.visibility_of_element_located(self._cart_count)).text

    return cart_count_text
```

```typescript
public async getCartAmount(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.cartAmount)), browser.allScriptsTimeout, this.cartAmount.toString());

  return await element(this.cartAmount).getText();
}

public async getCartCount(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.cartCount)), browser.allScriptsTimeout, this.cartCount.toString());

  return await element(this.cartCount).getText();
}
```

よくできました! プロジェクトの `pageobjects` フォルダから `DetailPagePo` を選択し、以下の関数を実装してください。

| **Filename**             | **Description**                                           |
| ------------------------ | --------------------------------------------------------- |
| `getProductName`         | `productName` 要素のテキストを返します。            |
| `getProductPrice`        | `productPrice`要素のテキストを返します。           |
| `addProductToCart`       | `addToCartButton` をクリックします。              |
| `getConfirmationMessage` | `cartConfirmationMessage`要素のテキストを返します。 |

*Pythonでは、関数名にそれぞれスネークケースを使用します。*

最終的には、`DetailPagePo` のメソッドは以下のようになるはずです。

```java
public String getProductName() {
  return this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.productName)).getText();
}

public String getProductPrice() {
  return this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.productPrice)).getText();
}

public void addProductToCart() {
  this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.addToCartButton)).click();
}

public String getConfirmationMessage() {
  return this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.cartConfirmationMessage)).getText();
}
```

```python
def get_product_name(self):
    product_name_text = self.wait.until(EC.visibility_of_element_located(self._product_name)).text

    return product_name_text


def get_product_price(self):
    product_price_text = self.wait.until(EC.visibility_of_element_located(self._product_price)).text

    return product_price_text

def add_product_to_cart(self):
    self.wait.until(EC.visibility_of_element_located(self._add_to_cart_button)).click()

    return self


def get_confirmation_message(self):
    cart_confirmation_message_text = self.wait.until(EC.visibility_of_element_located(self._cart_confirmation_message)).text

    return cart_confirmation_message_text
```

```typescript
public async getProductName(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.productName)), browser.allScriptsTimeout, this.productName.toString());

  return await element(this.productName).getText();
}

public async getProductPrice(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.productPrice)), browser.allScriptsTimeout, this.productPrice.toString());

  return await element(this.productPrice).getText();
}

public async addProductToCart(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.addToCartButton)), browser.allScriptsTimeout, this.addToCartButton.toString());
  await element(this.addToCartButton).click();

  return this;
}

public async getConfirmationMessage(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.cartConfirmationMessage)), browser.allScriptsTimeout, this.cartConfirmationMessage.toString());

  return await element(this.cartConfirmationMessage).getText();
}
```

これでテストケースを自動化する準備ができました。

### テストを書く

新しいテストファイルを作成するには、プロジェクトの `tests` フォルダを右クリックして、コンテキストメニューから **New \> Test file** を選択します。Sencha WebTestIt はあなたのために新しいクラスを生成し、名前を付けるのを待ちます。ファイル名は、TypeScript の場合は `Test1.ts`、Java の場合は `Test1.java` とします。今回は Python の場合はスネークケースは使いません。キャメルケースを使い、テストファイルの名前を `test1.py` とします。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> 実際のテストファイルは、Web サイトやアプリに対して実行されるアクションをオーケストレーションする必要があります。テストのメンテナンス性を確保するために、アクション自体はページオブジェクトに残しておくことをお勧めします。
> 
> 各テストケースの最も重要な部分は、ブロックの最後にあるアサーションです。これを使用して、ウェブサイトやアプリケーションの動作が期待される動作と一致しているかどうかを検証し、テストが失敗するかどうかを定義します。要素のアサーションには、さまざまな *Assertion Frameworks* が利用できます。
> 
> Sencha WebTestIt は、選択したスクリプト言語に応じてアサーションフレームワークをプロジェクトにバンドルします。Java でテストを書くことを選択した場合は、TestNG フレームワークが提供されます。TypeScript を使用している場合、Protractor と [Jasmine](https://jasmine.github.io/) がデフォルトの設定です。Python の場合は unittest 環境が提供されています。ただし、個々のコンポーネントを切り替えることは可能です。

使用する自動化フレームワークと言語に応じて、上の表のように3つのテストケースを作成します。

```java
@Test
public void SearchForItemTestCase() {
  // 1. Arrange
  // Create a new Page Object instance by right-clicking and
  // selecting "Instantiate Page Object" at the bottom
  HeaderPo header = new HeaderPo(driver);
  DetailPagePo detail = new DetailPagePo(driver);

  header.open("https://demoshop.webtestit.com");

  // 2. Act
  // Call an existing action from your initialized Page Object
  header.insertSearchText("Super");

  // 3. Assert
  // Use TestNG assertions to verify results.
  // e.g.:
  // Assert.assertEquals(title, "Test Automation for GUI Testing | Sencha");
  Assert.assertEquals(detail.getProductName(), "Super Cool Goo");
  Assert.assertEquals(detail.getProductPrice(), "€1,500.00");
}

@Test
public void AddItemToCartTestCase() {
  HeaderPo header = new HeaderPo(driver);
  DetailPagePo detail = new DetailPagePo(driver);

  detail.open("https://demoshop.webtestit.com/product/super-cool-goo/");

  detail.addProductToCart();

  Assert.assertTrue(detail.getConfirmationMessage().contains("“Super Cool Goo” has been added to your cart"));
  Assert.assertEquals(header.getCartCount(), "1 item");
  Assert.assertEquals(header.getCartAmount(), "€1,500.00");
}
```

```python
def test_search_for_item(self):

    driver = self.get_driver()
    """
    1. Arrange
    Create a new Page Object instance by right-clicking into
    the code editor and selecting "Instantiate Page Object"
    at the bottom of the context menu
    """
    header = header_po(driver)
    detail = detail_page_po(driver)

    header.open("https://demoshop.webtestit.com")
    """
    2. Act
    Call an existing action from your Page Object instance
    """
    header.insert_search_text("Super")
    """
    3. Assert
    Use unittest assertions to verify results.
    e.g.:
    self.assertEqual(title, "Test Automation for GUI Testing | Sencha")
    """
    self.assertEqual(detail.get_product_name(), "Super Cool Goo")
    self.assertEqual(detail.get_product_price(), "€1,500.00")


def test_add_item_to_cart(self):

    driver = self.get_driver()

    header = header_po(driver)
    detail = detail_page_po(driver)
    detail.open("https://demoshop.webtestit.com/product/super-cool-goo/")

    detail.add_product_to_cart()

    self.assertTrue(detail.get_confirmation_message in "“Super Cool Goo” has been added to your cart")
    self.assertEqual(header.get_cart_count(), "1 item")
    self.assertEqual(header.get_cart_ammount(), "€1,500.00")
```

```typescript
public async getProductName(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.productName)), browser.allScriptsTimeout, this.productName.toString());

  return await element(this.productName).getText();
}

public async getProductPrice(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.productPrice)), browser.allScriptsTimeout, this.productPrice.toString());

  return await element(this.productPrice).getText();
}

public async addProductToCart(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.addToCartButton)), browser.allScriptsTimeout, this.addToCartButton.toString());
  await element(this.addToCartButton).click();

  return this;
}

public async getConfirmationMessage(): Promise {
  await browser.wait(ExpectedConditions.visibilityOf(element(this.cartConfirmationMessage)), browser.allScriptsTimeout, this.cartConfirmationMessage.toString());

  return await element(this.cartConfirmationMessage).getText();
}
```

テストケースを自動化する準備はできています。

### テストを実行する

ついに、私たちの実験が機能するかどうかを確認する時が来ました。これを実現するためには、まず*エンドポイント*を作成しなければなりません。エンドポイントはテストの実行環境に関する設定のセットです。エンドポイントは、どのブラウザやオペレーティングシステムを使用するかなどを定義します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> 自動化のためのコマンドがHTTPで送信されるので、Selenium WebDriver の *JsonWireProtocol* のおかげで、ランタイム環境は実際に世界中のどこにでもあります。

ローカルのChromeエンドポイントを作成し、その上でテストを実行します。

1.  **Execution** タブに移動します
2.  **+** ボタンをクリックします
3.  **Endpoint** ダイアログに次の情報を入力します
    
    ![](https://docs.sencha.com/webtestit/guides/images/endpoint-tab.png)

4.  **Save endpoint** をクリックします

エンドポイントについては、[このガイドの第4章] (./RunningTests/WebdriverExecution.html)で詳しく説明しています。

### おめでとうございます！

このチュートリアルのすべてのステップに従っていれば、テストケースはエラーなく実行されるはずです。Sencha WebTestIt はプロジェクト内に新しいフォルダ `reports` を作成します。そこにあるXMLファイルをクリックすると、実行結果を表示する*Report* タブが開きます。

ここまでで次のことを学びました。

  - Sencha WebTestItでページオブジェクトを作成する方法
  - 要素を手動で作成する方法
  - ドラッグ＆ドロップを使ってページオブジェクトのアクションを素早く作成する方法
  - エンドポイントを作成してテストを実行する方法

### ここから先の続きはこちら

続きを読んでいただくために、いくつかの提案を作成しました。

  - [Sencha WebTestIt のセレクタと *Elements* タブについて学ぶ](../Automation/Introduction.md)
  - [ページオブジェクトを管理し、Drag&Dropでページオブジェクトのアクションを楽に作成する方法](../PageObjects/Introduction.md)
  - [Sencha WebTestItとCLIでテストを実行して素早く結果を出す](../RunningTests/Introduction.md)
  - [エンドポイントを設定し、*実行*タブでSeleniumグリッドに接続する](../RunningTests/WebdriverExecution.md)
  - [レポートを使った作業](../RunningTests/Reporting.md)
  - [診断モードを使ったエラーの発見とテストのメンテナンス](../Maintenance/DiagnosticMode.md)
HowToSetupTheJavaJdkForUseWithSenchaWebTestIt