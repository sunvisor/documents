一般的な設計上の注意事項
====================

> [オリジナル](https://api-platform.com/docs/core/design/)

API Platformは、公開するデータの構造を記述するだけなので、[「デザインファースト」で「コードファースト」](https://swagger.io/blog/api-design/design-first-or-code-first-api-development/)なAPIフレームワークです。ただし、「デザインファースト」の方法論が強く推奨されます。まず、APIエンドポイントの**公開形式(public shape)**を設計しましょう。 

そのためには、エンドポイントの入力と出力を表す古いPHPオブジェクトを書かなければなりません。これが [`@ApiResource` アノテーションでマークされた](https://api-platform.com/docs/distribution/)クラスです。
このクラスは Doctrine ORM やその他の永続化システムにマッピングされている**必要はありません**。
このクラスはシンプルでなければならず (通常は振る舞いのない、もしくは最小限の動作をするデータ構造体です)、API Platform によって [Hydra](https://api-platform.com/docs/core/extending-jsonld-context/)、[OpenAPI](https://api-platform.com/docs/core/swagger/)、[GraphQL](https://api-platform.com/docs/core/graphql/) のドキュメントやスキーマに自動的に変換されます (このクラスとそれらのドキュメントとの間は 1 対 1 にマッピングされます)。

次に、`DataProviderInterface` を実装して、このAPIリソースオブジェクトのインスタンスをハイドレートしたものをAPI Platformに供給するのは開発者の責任です。
基本的には、データプロバイダが永続化システム(RDBMS、ドキュメントやグラフDB、外部API...)に問い合わせを行い、上記のように設計されたPOPOをハイドレードして返す必要があります。

状態を更新 (POST、PUT、PATCH、DELETE HTTPメソッド) するときに、[シリアライザによってハイドレート](https://api-platform.com/docs/core/serialization/)された API Platform のリソースオブジェクトによって提供されるデータを適切に永続化するのは開発者の責任です。
そのためには、実装すべき別のインターフェイスがあります。`DataPersisterInterface` です。

このクラスはAPIリソースオブジェクト(`@ApiResource` でマークされたもの)を読み込みます。

- それを直接データベースに永続化します。
- またはDTOをハイドレートしてコマンドをトリガーします。
- またはイベントストアを生成します。
- またはその他の有用な方法でデータを永続化します。

データ永続化のロジックはアプリケーション開発者の責任であり、**API Platform の範囲外**です。

[ラピッドアプリケーション開発](https://en.wikipedia.org/wiki/Rapid_application_development)や利便性、プロトタイピングのために、`@ApiResource` でマークされたクラスが **Doctrine エンティティでもある場合に限り**、開発者は Doctrine ORM のデータプロバイダと API Platform に同梱されている永続化実装を利用することができます。

この場合、パブリック（`@ApiResource`）と内部（Doctrine エンティティ）のデータモデルが共有されます。
そうすると、API Platform は自動的にデータのクエリ、フィルタリング、ページネーション、永続化を行うことができるようになります。
このアプローチは非常に便利で効率的ですが、[CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) 以外のシステムや大規模なシステムには**向いていないかもしれません**。
繰り返しになりますが、扱うビジネスロジックに応じて、これらのビルトインデータプロバイダ/パーシスタを使用するかしないかは開発者次第です。API Platformは、カスタムデータプロバイダやパーシスタを簡単に作成することができます。
また、[Messenger Component との統合](https://api-platform.com/docs/core/messenger/)や [DTO のサポート](https://api-platform.com/docs/core/dto/)により、 [CQS](https://www.martinfowler.com/bliki/CommandQuerySeparation.html) や [CQRS](https://martinfowler.com/bliki/CQRS.html) のようなパターンを簡単に実装することができます。

最後になりましたが、[Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)ベースのシステムを作成するには、便利なアプローチがあります。

- Messenger ハンドラやカスタム[データパーシスタ](https://api-platform.com/docs/core/data-persisters/)を使って、イベントストアにデータを永続化する
- 標準的な RDBMS (PostgreSQL、MariaDB...) のテーブルやビューにプロジェクションを作成する
- これらのプロジェクションを読み取り専用の Doctrine エンティティクラスにマッピングして、これらのクラスを `@ApiResource` でマークする

これで、API Platform が提供するビルトインの Doctrine フィルタ、ソート、ページネーション、オートジョイン、そしてすべての[拡張ポイント](https://api-platform.com/docs/core/extending/)の恩恵を受けることができるようになります。

