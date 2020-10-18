# Javaでのデータ駆動型テストのサンプル

> オリジナル: [Data-driven testing in Java sample](https://docs.sencha.com/webtestit/guides/advanced-topics/data-driven-testing-in-java-sample.html)

システムがどのように動作するかを確認したり、期待される結果をテストしたり、システムが奇妙な方法で壊れるかどうかを確認したりするために、さまざまな入力を使ってテストしたい機能に出くわすことがあります。

これらの組み合わせをすべて自動化されたテストのセットに配線するのは、かなりの作業であり、これらのテストを長期間にわたって維持するのは難しい傾向があります。幸いにも、この分野で物事をより簡単にする方法があります（Sencha WebTestIt に関してはすでに慣れていると思いますが）。

**データ駆動型テスト**

このすっきりとしたテスト設計戦略により、ハードコードされた値を使用するのではなく、Excel シート、CSV ファイル、データベースなどのデータソースからテストデータを読み込んでインポートするテストスクリプトを作成することができます。この記事では、`@DataProvider` を使って CSV ファイルからテストデータを読み込むデータ駆動型のサンプルプロジェクトについて説明します。基本的には、[このログインフォーム](https://the-internet.herokuapp.com/login) を CSV ファイルに格納された3つの認証情報のセットでテストします。目的は、CSVファイルからデータを読み込み、実際のテストで利用することです。

  - プロジェクトツリーに`data`という名前のフォルダを作成して、実際のCSVファイルを保存し、データプロバイダクラスファイルを作成します。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> インターネット上には、DataProvider を実際のテストファイルに配置するように指示しているサンプルをよく見かけます。これは、ぱっと作ったデモのためには良いのですが、複数のテストファイルでの懸念事項や再利用性をきれいに分離するためには避けた方が良いでしょう。
>
テスト要件に応じて、CSV ファイルの形式はテストごとに若干異なる場合があります。ファイルにはヘッダレコードが含まれているかもしれないし、含まれていないかもしれないし、値はシングルクォーテーションまたはダブルクォーテーションの下に配置することができるので、実際のデータを取得するためにいくつかの追加の操作が必要になるかもしれません。私たちの場合は、データを一行ずつ読み込んで、インデックスを使って ArrayList に保存しています。

![](https://docs.sencha.com/webtestit/guides/images/data-driven-folder.png)

  - **CredentialsDataProvider.java クラスを作成する**: 
  それでは、データフォルダに `CredentialsDataProvider.java` クラスファイルを作成してみましょう。CSVReader を使用して、CSV ファイルからデータを読み込みます。ファイルの各行は、1つのクレデンシャルの組み合わせを表しています。

DataProviderは2次元の `Object[][]` を必要とするので、CSVReaderから取得した `ArrayList<Object[]>` を変換しなければなりません。

![](https://docs.sencha.com/webtestit/guides/images/create-credentials.png)

これでCSVReaderと実際の `@DataProvider` の設定が完了しました。これで、テストでCSVデータを使用してユーザー名とパスワードフィールドを入力し、ログインの成功/失敗メッセージをアサートします。

データ・プロバイダは、テスト・メソッドに2次元オブジェクトを返し、テスト・メソッドは、M \* N型のオブジェクト配列でM回呼び出されます。例えば、DataProviderが3\*3のオブジェクトの配列を返す場合、対応するテストケースは3つのパラメータで3回呼び出されます。

  - **テストファイルの作成** - 実際のテストに入って、新しいテスト・ファイルを作成し、`@Test` アノテーションで "Credentials" という名前のデータ・プロバイダと、dataProviderClass のタイプ (`CredentialsDataProvider.class`) を指定しています。
    
    ![](https://docs.sencha.com/webtestit/guides/images/create-test-file.png)

  - **レポートにテストケース名を表示する** 実装されている `ITest` インターフェースを用いて、`@BeforeMethod` アノテーションの下に以下のメソッドでテスト名をフォーマットします。
    
    ```java
      public void BeforeMethod(Method method, Object[] testData) {
      testName.set(String.format("%s [Username: '%s'; Pass: '%s']", method.getName(), testData[0], testData[1]));
    }
    ```
    
    もちろんそれはカスタマイズすることができます。これに `@Override` のアノテーションを追加して、カスタマイズしたテストケース名を完成させることを忘れないでください。
    
    ```java
    @Override
      public String getTestName() {
          return testName.get();
      }
    ```
    
    私たちのCSVファイルには3つの認証情報のセットが含まれているので、このテストは3回実行されます。最初の認証情報は正しいもので、他の2つは欠陥があります。すべてのクレデンシャルを試してみて、適切なメッセージが表示されていることをアサートすることで、ログインフォームをテストします。

  - **カスタムアサートメッセージの作成**: `org.testng.Assert.assertEquals(String actual, String expected, String message)`メソッドを使用して、テストに失敗した場合のカスタムメッセージ (*iteration indicator*) を作成します。このメッセージはレポートに表示され、ログイン中にどのような資格情報のセットが、そのテスト実行のために不適切なログイン確認メッセージをもたらしたかについての情報を提供します。
    
    ![](https://docs.sencha.com/webtestit/guides/images/assert-message.png)

### まとめ

テストシナリオは3つあります。最初のものは正しい認証情報がフォームに送信され、成功メッセージがアサートされている場合、他の2つは正しくない認証情報が送信され、ログイン失敗メッセージがアサートされている場合です。

また、テスト レポートに表示されるカスタム アサート メッセージは、どのクレデンシャルのセットがテストの失敗の原因となったかを認識するのに役立ちます。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> このサンプルプロジェクトは、アプリケーション内の「ようこそ」画面の「サンプル」カテゴリから直接ダウンロードしてください。このサンプルプロジェクトは、[Gitリポジトリからダウンロードしてください](https://github.com/extjs/RxSe-java-data-driven-sample) 。
