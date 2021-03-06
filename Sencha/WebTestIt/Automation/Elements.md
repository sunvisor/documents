# エレメント

> オリジナル: [Elements](https://docs.sencha.com/webtestit/guides/automation/elements.html)

各Webサイトやアプリケーションは、UIのビルディングブロックまたは**エレメント**で構成されています。画像、テキスト、リンク、ボタンなど、さまざまな形式のエレメントがあります。UIロジックは、これらのエレメントとのインタラクションの可能性を定義します。ウェブサイトやアプリケーションの**ビジネスロジック**は、どの条件下でエレメントが表示されるとか、クリックできるとか、非表示にするとかを定義します。

オートメーションフレームワークは、検査したいエレメントを識別し、それらを操作したり、アサートしたりすることを担当していることは、すでに導入部でお気づきの通りです。これを実現するために、各エレメントは***セレクター**によって識別されます。

### セレクター

セレクタは、Web サイトやアプリのビジュアル構造内のエレメントロケータです。ほとんどの最新の UI 記述言語 (HTML のような) では、この構造は階層的です。セレクタを自動化フレームワークに渡すと、そのルールを現在アクティブなページに適用し、それによって記述されたエレメントを見つけようとします。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> エレメントが見つからない場合、自動的にテストに失敗する必要はありません。しかし、信頼性の高いテストを実行するためには、エレメントに対して堅牢なセレクタを持つことが非常に重要です。

Sencha WebTestIt は **Elements** タブでセレクタを使った作業をサポートしており、ページオブジェクトのセレクタを定義して整理することができます。

各セレクタは、特定のルールセットに従う文字列値として表現されます。これらのルールセットは *strategies* と呼ばれています。

### セレクターの種類

ウェブサイトやアプリケーションの中でオブジェクトを確実に見つけるためには何が必要でしょうか？ 最も簡単な方法は、ユニークなIDを通して各エレメントを参照することだと思うかもしれません。 この点については、私たちもあなたの意見に全面的に同意します。 しかし、実際には、これが機能しない状況がしばしばあります。

  - 多くのウェブサイトやアプリケーションでは、ID はすべてのエレメントに提供されているわけではありません。
  - 一部の UI フレームワークでは、自動生成されたランダムな ID を各エレメントに割り当てています。
  - アプリケーションで動的に生成されたオブジェクトに固定の ID を持たせることはできません。

IDだけに頼ることができない場合には、UI階層内での配置や、そのエレメントが持つ他の属性によってエレメントを見つけるのに役立つセレクタ戦略が用意されています。Sencha WebTestIt は以下のセレクタ戦略をサポートしています。

  - **XPath –** [XPath](https://en.wikipedia.org/wiki/XPath) ストラテジーは、HTML文書を階層的にたどることを可能にします。これは、HTML構造の各レベルがファイルシステムのフォルダ構造に似ているという事実に基づいています。このため、エレメントを見つけるために文書をトップダウンでたどることができます。[XPathの例とチュートリアルはこちらをご覧ください](https://www.w3schools.com/xml/xpath_intro.asp)
  - **CSS –** エレメントのスタイル記述を使用して、UI 内でそれを見つけることができます。CSS セレクタは、基本的にはウェブサイトやアプリケーションのスタイルを定義するために使用するのと同じです。CSS の例は [こちら](https://www.w3schools.com/cssref/css_selectors.asp) を参照してください。
  - **link-text –** 非常に単純な戦略は、リンクテキストです。これは、アプリケーションの\<a\>タグの内容をスキャンして、完全または部分的に一致するものを検索します。これはハイパーリンクに対してのみ機能することに注意してください。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> 推奨されるエレメントセレクタの順序は、`id` \> `name` \> `CSS` \> `XPath` です。テスト用の堅牢なセレクタを生成するために、これを心に留めておいてください。

セレクターを作成するには、さまざまなアプローチを使うことができます。Google Chrome のようなブラウザでは、開発ツールからクリップボードにセレクタをコピーすることができます。

### Sencha WebTestIt の Element タブ

**Elements** タブは、ページオブジェクト内のエレメントとセレクタを管理するために使用されます。現在開いているページオブジェクトと同期します。

![](https://docs.sencha.com/webtestit/guides/images/elements-tab.png)

1. **+ button** クリックして新しいエレメントを作成する**+ボタン**、クリックして新しいエレメントを作成します。

2. **Element:** 名前をクリックすると、エレメントの属性が展開されます。

    - **Name:** コードで参照されるときのエレメントの名前。
    - **Selector:** エレメントの位置を特定するために使用されるセレクタ。
    - **Strategy:** エレメントの位置を特定するために使用されるセレクタストラテジー

3. **Screenshot Indicator:** 有効にすると、このエレメントで利用可能なスクリーンショットがあることを示します。このインジケータの上にマウスを置くと、スクリーンショットが表示されます。クリックしてスクリーンショットを開きます。

4. **Code reference:** クリックすると、コード内のエレメントの定義にジャンプします。

5. **Delete button:** クリックしてエレメントを削除します。これは、そのエレメントがプロジェクトのどこにも使用されていない場合にのみ機能します。

6. **Close button:** クリックしてエレメントの詳細を閉じます。

### エレメントを手動で追加する

現在アクティブなページオブジェクトファイルに新しいエレメントを追加するには、以下の指示に従います。

1.  **+** ボタンをクリックします。

2.  新しく作成されたエレメントスタブで、
    
    1.  **Name** を指定します。
    2.  **Selector**を記入します。
    3.  使用する **Strategy** を選択します。
    4.  **Close** ボタンをクリックします。

### CSS ストラテジーのための複数のセレクタを管理する

If you are using CSS as a strategy, you can specify multiple selectors to identify your element within your SUT. You will notice there is an **Additional Selector** field located. You can click **+** to add even more selectors if needed. You can click **–** to remove the selector in that respective line.

ストラテジーとしてCSSを使用している場合、SUT内でエレメントを識別するために複数のセレクタを指定することができます。**Additional Selector** フィールドがあることにお気づきでしょう。必要に応じて、**+** をクリックしてさらにセレクタを追加することができます。**-** をクリックすると各行のセレクタを削除できます。

![](https://docs.sencha.com/webtestit/guides/images/css-multiple.png)

### ファイルの同期

Sencha WebTestItの**Elements**タブは、ページオブジェクトファイルと同期しています。UIを使用する代わりに、ページオブジェクトファイルを直接編集することができます。変更は入力中に **Elements** タブに反映されます。

### スクリーンショットを手動で提供する

手動および自動で生成されるエレメントのために、独自のスクリーンショットを提供することができます。プロジェクト構造の中に `screenshots` フォルダがあり、そこには自動的に生成されるすべてのスクリーンショットが含まれています。このフォルダに独自のスクリーンショットを提供することも、別の場所を指定することもできます。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> プロジェクトに `screenshots` フォルダがない場合は、*Project* タブのスクリーンショットボタンをクリックしてすべてのファイルを表示してください。それでもフォルダが表示されない場合は、まだスクリーンショットがないためです。この場合は手動でフォルダを作成してください。

#### エレメントにスクリーンショットを追加するには

1.  ページオブジェクトのソースコード内のエレメントを探します。

2.  その上にデコレーターのコメントを追加します: `// Additional data: {"img":"your/path/your_filename.png"}`.

3.  `your/path/your_filename.png` をスクリーンショットの実際の場所に置き換えてください。

4.  画像ファイルへのパスが正常に解決されると、Sencha WebTestItは、編集したエレメントのスクリーンショットインジケータを直ちにアクティブにします。
