# Python と Sencha WebTestIt を使い始める

> オリジナル: [Getting started with Python and Sencha WebTestIt](https://docs.sencha.com/webtestit/guides/getting-started/getting-started-with-python-and-sencha-webtestit.html)

Sencha WebTestIt は、Java、TypeScript、Pythonプログラミング言語をサポートする Protractor と Selenium を使用したテストのためのテスト自動化ツールセットとしてよく知られています。

Sencha WebTestItはPythonを使ったテストのために [unittest](https://docs.python.org/3/library/unittest.html) フレームワークを利用しています。

unittest は Python でのテスト自動化を支援するためのツールをふんだんに盛り込んだ Python の標準ライブラリのモジュールです。主にユニットテストを目的としていますが、unittest フレームワークは Web GUI テストにも利用できます。最も重要なことは、Java や TypeScript プロジェクト用の TestNg や Protractor 環境を生成するのと同じように、Sencha WebTestIt がテストやフレームワークの設定をすべて代行してくれるので、テストやフレームワークの設定を丸ごと心配する必要がないことです。

## Python をインストールする

### Windows

1. <https://www.python.org/> に行き Python の最新版をダウンロードします

2.  [Python をインストール](https://wiki.python.org/moin/BeginnersGuide/Download) して PATH に Python を追加します
    
    ![](https://docs.sencha.com/webtestit/guides/images/python-install.png)

3.  コマンドラインウィンドウで **python** と入力してインストールを確認します。  
    
    <img src="https://docs.sencha.com/webtestit/guides/images/python-cmd.png" width="700px" />

4.  WebTestIt を再起動します

### macOS

1. <https://www.python.org/> に行き macOS 用の Python の最新版をダウンロードします

2.  [Python をインストール](https://wiki.python.org/moin/BeginnersGuide/Download) して [PATH に Python を追加](https://bic-berkeley.github.io/psych-214-fall-2016/using_pythonpath.html) します
    
    ![](https://docs.sencha.com/webtestit/guides/images/python-mac.png)

3.  ターミナルで **python** と入力してインストールを確認します。  
    
    ![](https://docs.sencha.com/webtestit/guides/images/python-terminal.png)

4.  WebTestIt を再起動します

### Linux

1.  ターミナルを開きます

2.  次のコマンドをタイプします
    
    `sudo apt update`
    
    `sudo apt install software-properties-common`
    
    `sudo add-apt-repository ppa:deadsnakes/ppa`
    
    `sudo apt update`
    
    `sudo apt install python3.7`

3.  ターミナルで **python3.7** と入力してインストールを確認します。  
    
    ![](https://docs.sencha.com/webtestit/guides/images/python-linux.png)

4.  ターミナルに以下のコマンドを入力して、必ずPipパッケージ管理システムをインストールしてください
    
    `sudo apt install python3-pip`

5.  仮想環境が適切にサポートされるように、以下をインストールします
    
    `sudo apt install python3-venv`

6.  WebTestIt を再起動します

## 新しいプロジェクトを開く

Sencha WebTestItを開いたら、以下の手順で新しいプロジェクトを作成します。

1.  プロジェクト名を入力します。

2.  言語オプションとフレームワークとして **Python** と unittest を選択します。

3.  **Save**をクリックします。Sencha WebTestIt はあなたのための作業環境を準備します。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/python-new-project.png" width="700px" />
    
## ページオブジェクトの作成

PythonでWeb GUIをテストするには、セレクタを使ってWebページのエレメントを探す必要があります。Sencha WebTestItでは、推奨されている[Page Object pattern](./advanced-topics/structuring-your-page-objects.html)を利用してセレクタを整理しています。ヘッダーやメニュー、コンテンツ領域など、Webページ上の重要なコンポーネントに対してページオブジェクトを作成することで、コンポーネントごとのWebエレメントの扱いが格段に楽になり、テストのメンテナンス性が向上します。

Sencha WebTestItでPythonプロジェクトを作成したら、次はページオブジェクトを作成します。

ページオブジェクトの取得方法については、[demoshopページ](http://demoshop.webtestit.com/) を参考にしてください。

ここでは、ヘッダー、アイテム概要、ショッピングカートの3つ以上のPage Objectをこのページから作成できることがわかります。下の画像をチェックしてみてください。

<img src="https://docs.sencha.com/webtestit/guides/images/python-create-page-objects.png" width="700px" />

ここで、ヘッダーコンポーネントのページオブジェクトを作成します。**New Page Object file** をクリックするか、**pageobjects**フォルダを右クリックして、**New \> Page Object file** を選択します。

<img src="https://docs.sencha.com/webtestit/guides/images/python-page-object.png" width="700px" />

接尾辞として 'Po' を持つページオブジェクトクラスを作成するのは良い習慣です。Python を使用しているので、スネークケース、つまりページオブジェクトの名前を付ける際には、文字と文字の間にスペースを入れずにアンダースコアを付けるようにすることをお勧めします。

Header コンポーネントのページオブジェクトの名前は `header_po.py` とします。

![](https://docs.sencha.com/webtestit/guides/images/python-header.png)

デモショップページの他のコンポーネントのためにページオブジェクトを作成するときにも同じ戦略を使用します。つまり、`items_overview_po.py`, `shopping_cart_po.py`などの名前が付けられます。

しかし、今回のデモでは、これらを作成しません。["Super Cool Goo 詳細ページ](https://demoshop.webtestit.com/product/super-cool-goo) 用のページオブジェクトが必要です。それを作成して `detail_page_po.py` という名前にしてみましょう。

## ページオブジェクトのアクションを作成する

すべてのエレメントをページオブジェクトに追加したので、それらに対していくつかのアクションを起こしてみましょう。

**Elements** タブからページオブジェクトのコードにエレメントをドラッグ＆ドロップします。Sencha WebTestIt はアクションを反映したメソッドを作成します。

まず `_search_input` を `header_po` のコードにドラッグします。マウスを離すとドロップダウンメニューが表示されるので、メソッドに反映させたいアクションを選択することができます。このエレメントの場合は、ポップアップメニューから `Do > Type into element` を選択してテキストを送信します。

<img src="https://docs.sencha.com/webtestit/guides/images/python-gif-page.gif" width="700px" />

関数名を `insert_search_text` とすると、コードは以下のようになります。

```python
def insert_search_text(self, text):
    self.wait.until(EC.visibility_of_element_located(self.search_input)).send_keys(text)

    return self
```

残りのエレメントについては `header_po` の中のドラッグ＆ドロップを繰り返しますが、今回はポップアップメニューから `Get > Element's text` を選択します。コードは以下のようになります。

```python
def get_cart_amount(self):
    cart_count_count = self.wait.until(EC.visibility_of_element_located(self._cart_count)).text

    return cart_count_count

def get_cart_count(self):
    cart_count_text = self.wait.until(EC.visibility_of_element_located(self._cart_count)).text

    return cart_count_text
```

同様に、すべてのエレメントを `detail_page_po` のコードに追加し、それぞれに適切なアクションを選択します。
例えば、ボタンの場合は `Do > Click on element`、テキストエレメントの場合は `Get > Element's text` などです。 

You can view the list of all actions on elements available in Sencha WebTestIt [here](../page-objects/managing-page-objects.html).

Sencha WebTestIt で利用可能なエレメントに対するすべてのアクションのリストは[ここ](../PageObjects/ManagingPageObjects.md) で見ることができます。

最終的には、`detail_page_po` のメソッドは以下のようになります。

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

自由にすべてのアクションを編集でき、エレメントをドラッグしてメソッドを新しく生成できることを覚えておいてください。

## テストを書く

すべてのエレメントを追加してアクションを開始したら、テストを作成します。

1.  プロジェクトツリーの **Tests** フォルダを右クリックして、**New \> Test file** を選択します。

2.  ファイル名を `tc1.py` とします。ご覧の通り、テストファイルの名前にはスネークケースを使用していません。

3.  新しいテストファイルは、AAA (Arrange, Act, Assert) パターンを使ったテストの書き方を指示する空のスタブで生成されます。これは、あなたのテストを構造化された、きれいで、読みやすいものに保つのに役立ちます。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/python-writing-test.png" width="700px" />

Sencha WebTestIt はまた、テストをメソッドとしてクラスに入れたり、組み込みのアサート文の代わりに `unittest.TestCase` クラスで特別なアサートメソッドを使ったりするなど、unittest フレームワークがテストを書いて実行するために持っているいくつかの重要な要件にも対応しています。

最終的には、テストは次のようになります。

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

    self.assertTrue(detail.get_confirmation_message in ""Super Cool Goo" has been added to your cart")
    self.assertEqual(header.get_cart_count(), "1 item")
    self.assertEqual(header.get_cart_ammount(), "€1,500.00")
```

## テストを実行する

テストを実行するには、エンドポイントを作成する必要があります。エンドポイントとは、ブラウザ、オペレーティングシステム、使用するデバイスなど、テストの実行時環境に関する設定のセットです。Sencha WebTestIt では、テストを実行するためのローカル、リモート、カスタムのエンドポイントを定義することができます。 

**エンドポイントを作成する**

1.  **Execution** タブの **Add endpoint** をクリックします。

2.  必要なパラメータをすべて追加して、**Save endpoint** をクリックします。

3.  Click either **Run current test file** or **Run all test files** to execute your tests.
3.  **Run current test file** または **Run all test files** をクリックして、テストを実行します。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/python-execute.gif" width="700px" />

テストの実行が終了すると、Sencha WebTestIt はプロジェクト内に新しいフォルダ **Reports** を作成します。テストのレポートが自動的に **Report** タブで開かれ、テスト実行の結果が表示されます。また **Reports** フォルダ内のXMLファイルをクリックしてレポートを開くこともできます。この記事のすべての手順に従っていれば、テストケースはエラーなく実行されるはずです。しかし、テストが失敗することもあるかもしれません。その場合、Sencha WebTestItは[Jiraとの統合](../AdvancedTopics/GettingStartedWithJira.md)のおかげで、テストレポートからバグチケットを作成したり、問題を解決したりする機能を提供しています。
