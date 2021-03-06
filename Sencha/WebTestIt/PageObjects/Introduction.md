# ページオブジェクト

> オリジナル: [Page objects](https://docs.sencha.com/webtestit/guides/page-objects/introduction.html)

ウェブサイトやアプリのテストには、慎重な計画と準備が必要です。製品が成熟し成長するにつれ、要件の変更や複雑さの増大を考慮する必要があります。自動テストの実装を成功させる鍵は、テストの実行前にテストのセットアップを計画することです。自動化フレームワークは、異なるワークフローでテストを構成するのに役立つ一連の構成を提供します。テストスイートとテストケースを使用して、必要なビットを論理的なグループに整理します。テスト構造の詳細については、[こちら](../automation/components.html) をご覧ください。

エレメントについてはどうでしょうか？ウェブサイトやアプリケーションの特定のボタンを自動化するために、同じセレクタを使用する複数のテストファイルがあることを想像してみてください。各テストファイルでセレクタを定義すると、セレクタが変更された場合、各テストファイルでもセレクタを変更しなければならないことになります。**ページオブジェクト**を使用すると、セレクタを一元的に整理し、テストファイルに抽象化したレイヤーを導入することができます。ページオブジェクトを使用すると、エレメントごとにセレクタを定義し、その都度再利用することができます。ロケータが壊れてセレクタを変更しなければならない場合は、中央の一箇所で変更するだけで済みます。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> Webサイトやアプリのテストを計画する際には、エレメントとテスト操作を別のドメインに分けることを検討しましょう。

しかし、ページオブジェクトの概念の背後には、単なるエレメントの集まり以上のものがあります。エレメントの代わりに*アクション*を管理するのに役立つ[Page Object Pattern](../AdvancedTopics/StructuringYourPageObjects.md)を確認してください。Sencha WebTestには、この章で学ぶページオブジェクトを中心としたワークフローが付属しています。
