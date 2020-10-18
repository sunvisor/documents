# Structuring your Page Objects

> オリジナル: [Structuring your Page Objects](https://docs.sencha.com/webtestit/guides/advanced-topics/structuring-your-page-objects.html)

> *“テストメソッドに WebDriver API を使用している場合、それは間違っています。”* - サイモン・スチュワート

Selenium プロジェクトのリードコミッターであるサイモン・スチュワート氏は、Selenium の初期ビジョンの中で、ページオブジェクトは WebDriver の上にある必須レイヤーであることを共有しています。Sencha WebTestIt はこの事実を念頭に置いて作られました。

Sencha WebTestIt を効率的に使うためには、ページオブジェクトパターンを理解する必要があります。

### ページオブジェクトとは？

**ページオブジェクト**パターンは、カプセル化の典型的な例です。これは、GUIのデータを見つけて操作するために必要な仕組みを、アプリケーション固有のAPIで包み込んでいます。ページオブジェクトの基本的な経験則は、ソフトウェアクライアントが人間ができることを何でもできるようにすることです。

ページオブジェクトという名前ですが、ページごとにページオブジェクトを作成しないでください。ページオブジェクトは、ヘッダーやメニュー、コンテンツリージョンなど、ページ上の重要な構成エレメントに対して作成してください。

#### サンプル

ページオブジェクトの導出方法の一例として、弊社の[デモショップ](http://demoshop.webtestit.com/) を使用しています。

最初のスタートページを、少なくとも3つのページオブジェクトに分割することができます。

1.  **HeaderPo**: これはすべてのページにありますので、これをページオブジェクトとして抽出するのは理にかなっています。

2.  **ItemsOverviewPo**: このコンポーネントには、販売可能なすべてのアイテムが含まれています。

3.  **ShoppingCartPo**: このショッピングカートのサマリーはヘッダーの外に展開しますが、独自のページオブジェクトにするのに十分な機能とエレメントが含まれています。
    
    <img src="https://docs.sencha.com/webtestit/guides/images/demoshop-page.png" width="700px" />

### ページオブジェクトのアクション

さて、ページをページオブジェクトに分割したので、テストでどのように使用できるかを見てみましょう。

ページオブジェクトパターンの意図は、*Application* と *Assertion API* を *WebDriver API* から分離することです。これは、テストが HTML エレメントや WebDriver を直接使用してはならず、 *Actions* と呼ばれる公開 (パブリック) メソッドのみを使用することを意味します。

これらのアクションは、WebDriver APIとページオブジェクトに関連付けられたエレメントを内部的に使用して、*"ボタンをクリック "*、*"ログイン "*、*"フォームを送信 "* などのユーザインタラクションの単位をカプセル化します。

そのため、アクションメソッドを作成する際には、以下のようなベストプラクティスを念頭に置きましょう。

1.  **エレメントを直接返さない**, その代わりに:
      - 基本的な型 (文字列、整数、日付、...) の値を返します。
      - 単にエレメントを操作しているだけの場合は、ページオブジェクトのインスタンス自体 (`this`) を返します。
      - リンクをクリックした後やフォームを送信した後など、コンテキストスイッチを示すために別のページオブジェクトを返します。
      - これにより、ページオブジェクトのアクションを *チェイン* して最終的な値を返すことができ、それをアサーションで使用することができます。
2.  **ページオブジェクトでアサーションを使用しない**, その代わりに:
      - エレメントの関連するプロパティ、例えば text を返します。
      - あるいは、比較値を引数として渡してブール値を返すこともできます。
      - これらの結果をテストファイルのアサーションに使用してください。
3.  **優れたページオブジェクトのアクションは再利用可能です**, テストの構成エレメントとしてイメージしてみてください。

#### サンプル

上記のページオブジェクトを使って、先ほど学んだルールで簡単なテストを作ってみましょう。テストは以下のようにします。1. カートに1つのアイテムを追加する、2. カートに移動する、3. 合計価格が正しいことを確認します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> この例では Java を使用していますが、TypeScript プロジェクトにも同じ原理が適用されます。

1.  このアクションは `ItemsOverviewPo` で行われ、カートにアイテムを追加します。カートにアイテムを追加するだけでは、どこにもナビゲートしてくれませんし、他のテストでは `ItemsOverviewPo` の中でもっと多くのことをしたいかもしれません。なので、`this`を返します。
    
    ```java
    public ItemsOverviewPo addItemToCart() {
        this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.addItemToCartButton)).click();
        return this;
    }
    ```

2.  同じページオブジェクトの中で、*View cart* ボタンをクリックすると、*Cart* ページに遷移することを期待しているので、対応するページオブジェクトクラス `CartPo` の新しいインスタンスを返すことになります。
    
    ``` java
    public CartPo viewCart() {
        this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.viewCartButton)).click();
        return new CartPo(this.driver);
    }
    ```

3.  最後に、価格が正しく表示されているかどうかを確認する準備ができました。ページオブジェクトのアクションにアサーションを入れないでください。代わりに、合計価格を保持するエレメントのテキストを返すメソッドを作成します。
    
    ``` java
    public String getTotalPrice() {
        String totalPriceText = this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.totalPrice)).getText();
        return totalPriceText;
    }
    ```
    
    これで、これらのアクションをテストで使用することができます。アクションでページオブジェクトのインスタンスを返すおかげで、このようにアクションメソッドを*チェイン*することができ、テストの可読性を向上させることができることに注意してください。
    
    ```java
    @Test
    public void checkTotalPrice() {
        // 1. Arrange
        WebDriver driver = getDriver();
        ItemsOverviewPo itemsOverviewPo = new ItemsOverviewPo(driver)
                .open("https://demoshop.webtestit.com");
    
        // 2. Act
        String totalPrice = itemsOverviewPo
                .addItemToCart()
                .viewCart()
                .getTotalPrice();
    
        // 3. Assert
        Assert.assertEquals(totalPrice, "€1,500.00");
    }
    ```

### まとめ

この記事を読んだ後は、ページオブジェクトのパターン、複雑なウェブサイトを個々のページオブジェクトに分割する方法、ベストプラクティスを使用してページオブジェクトのアクションを構成する方法について学びました。

PageObjectsについてもっと詳しく知りたい方は、[マーティン・ファウラー氏のページオブジェクトに関する記事](https://martinfowler.com/bliki/PageObject.html) を必ずお読みください。
