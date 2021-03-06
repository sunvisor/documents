基礎
====

> オリジナル: [Fundamentals](https://docs.sencha.com/webtestit/guides/getting-started/fundamentals.html)

この章では、ウェブサイトやアプリをテストする際に必要となる一般的な用語や定義を紹介します。すぐに Sencha WebTestIt を使い始めたい場合は、この章をスキップして、最初のテストを作成するところから始めても構いません。

なぜテスト自動化なのか？
-------------------

あなたは典型的なソフトウェア開発プロセスに精通しているかもしれません。開発者はある種の仕様に基づいてコードを書き、テスターとして、それが意図した通りに動作するかどうかを検証するのがあなたの仕事です。通常、ウェブサイトやアプリの期待される動作を記述する[テストケース](https://en.wikipedia.org/wiki/Test_case)を作成することから始めます。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
>
> 他の目的の中でも、テストケースは、アプリの機能がどのように検証されたかを文書化する役割を果たします。テストケースは、あなたが最初に定義したのと同じ方法でソフトウェアをテストするために、組織内の他の人が使用することができます。
 
 古典的なソフトウェア開発の時代には、*テスト*とは、ある時点ですべてのテストケースを取得し、ウェブサイトやアプリケーションに対して検証しなければならないことを意味していました。今日、私たちはアジャイルプロセスで仕事をしていますが、これはリリースの高速化と反復回数の短縮に焦点を当てたもので、テストケースをより頻繁に適応させ、検証しなければならないことを意味します。これに伴い、いくつかの深刻な問題が発生します。

- 同じテストケースを何度も何度も検証しなければならないので、作業が単調になります。
- 日常的に何かを見落としてしまう可能性が高くなります。
- ソフトウェアが複雑になり、テストケースの数が増えると、時間通りに結果を出すことが難しくなります。
- 大規模なプロジェクトでは、多くの場合、ソフトウェアの修正された部分のみがテストされますが、その中のコンポーネントの依存性を評価することは困難です。  
  これは、技術的な負債の増加を発見できずに放置してしまうリスクを助長しています。
 
上記の問題は、自動化することで簡単に回避することができます。機械は反復的な作業を得意としており、シンプルなロジックに基づいています。そのおかげで、すべてのテストケースが毎回同じ方法で実行されるので安心できます。テスト自動化の世界に入ってみましょう。

Sencha WebTestItとは？
--------------------

Sencha WebTestItは、WebサイトやWebアプリのUIテストを作成・実行するために最適化された軽量なエディタとツールセットです。Sencha WebTestItで書かれたテストは、ブラウザを制御するアプリであり、ウェブサイトやアプリ上のユーザーをシミュレートします。 

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
>
>  ブラウザを遠隔操作するプロセスはブラウザの自動化と呼ばれ、[WebDriver](https://www.w3.org/TR/webdriver1/) と呼ばれる API によって有効化されています。最近のすべてのブラウザは、デスクトップ、タブレット、スマホで WebDriver をサポートしています。

自動テストの助けを借りて、多くの時間を節約することができ、手動でテストケースを何度も何度も繰り返す必要はありません。Sencha WebTestItは、テストを実行するための環境を設定し、[ページオブジェクト](https://docs.sencha.com/webtestit/guides/page-objects/introduction.html)を管理し、実際のテストケースをスクリプト化するための強力なエディタを提供します。

自動化フレームワークと言語
---------------------

機械でも人間でも読めるテストケースを作成するには、テストプログラムへのステップの適切な翻訳を保証するアプリケーション・プログラミング・インターフェース (API) を利用する必要があります。テストを書くために、さまざまな自動化フレームワークやプログラミング言語を使うことができます。Sencha WebTestIt はテスト自動化のためのフレームワークとして Selenium、Protractor、unittest をサポートしています。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
>
> Sencha WebTestIt は環境を設定し、テスト開発を直接開始するための準備をします。最初にマシンの設定や設定を心配する必要はありません。

Sencha WebTestIt でテストやページオブジェクトを作成するには、サポートされているプログラミング言語のいずれかの知識が必要です。Java, TypeScript, Python のチュートリアルは以下のリンクから見つけることができます。

- Java は [Oracle のドキュメント](https://docs.oracle.com/javase/tutorial/)
- TypeScript は [aQuick start guide](https://www.typescriptlang.org/docs/tutorial.html)
- [Python 3](https://docs.python.org/3/tutorial/)

セレクタとページオブジェクト
----------------------

ウェブサイトやアプリは、ブラウザによって画面上に表示される HTML コンポーネントのセットで構成されています。ユーザーは、それらのエレメントをクリックしたり、キーボードから値を入力したりすることで、それらのエレメントと対話します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
>
>  ウェブデザインの努力と何十年にもわたるコンピュータの使用のおかげで、人間はウェブサイトやアプリ内の様々なエレメントを簡単に識別することができるようになりました。私たちは、デザインが変化しても、入力エレメントとの相互作用の仕方や、それらをどこに配置するかについて、長い時間をかけて自分自身を訓練してきました。

テストの中で同じエレメントと対話するためには、それらを適切に識別してアクセスする方法を自動化フレームワークに伝える必要があります。これを行うには、セレクタを通してそれらを参照します。セレクタによって提供される情報によって、自動化フレームワークは、ウェブサイトやアプリの UI 構造内のエレメントを見つけます。

Sencha WebTestIt には、セレクタを[ページオブジェクト](https://github.com/SeleniumHQ/selenium/wiki/PageObjects) に整理して管理するための強力なツールが付属しています。ページオブジェクトについては、[このガイドの第3章](../PageObjects/Introduction.md) で詳しく説明しています。

エンドポイント
-----------

エンドポイントとは、テストを実行するための設定のセットです。テスターとしては、Web サイトやアプリがさまざまなブラウザやオペレーティング システムで動作することを確認したいと思います。エンドポイントの助けを借りれば、それが可能になります。

Sencha WebTestIt は、テストを実行するためのエンドポイントの設定と管理を支援します。エンドポイントについては、[このガイドの第4章](../RunningTests/Introduction.md)で詳しく説明しています。

テストを書くために必要なもの
----------------------

上記で説明したサポートされているプログラミング言語の基本的なプログラミング知識が必要です。それに加えて、サポートされているテストフレームワークの一つに精通していることをお勧めします。[Protractor](https://www.protractortest.org/#/tutorial), [Selenium WebDriver](https://www.seleniumhq.org/docs/03_webdriver.jsp#introducing-the-selenium-webdriver-api-by-example), [unittest](https://docs.python.org/3/library/unittest.html)) のチュートリアルは公式サイトで見ることができます。