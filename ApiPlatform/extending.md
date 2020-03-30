# API Platform を拡張する

> [オリジナル](https://api-platform.com/docs/core/extending/)

API インフラストラクチャを作成するという複雑で退屈で反復的なタスクを処理してくれるので、API Platform を使用することで、エンドユーザーにとって最も重要なこと、つまりビジネスロジックに集中することができます。
そのために、API Platform には、独自のコードをフックするために使用できる多くの拡張ポイントが用意されています。
これらの拡張ポイントは、REST サブシステムと [GraphQL](graphql.md) サブシステムの両方で考慮されます。

以下の表は、やりたいことに応じてどの拡張ポイントを使うかをまとめたものです。

| Extension Point    | Usage     |
|--------------|-----------------|
| [Data Providers](data-providers.md) | カスタム永続化レイヤー、仮想フィールド、カスタムハイドレーション用のアダプタ |
| [Denormalizers](serialization.md)|HTTP リクエストボディで送信されたペイロードから作成されたオブジェクトをポストプロセスする|
| [Voters](security.md#hooking-custom-permission-checks-using-voters) |カスタム認証ロジック |
| [Validation constraints](validation.md) | カスタム バリデーション ロジック |
| [Data Persisters](data-persisters) | 永続性の前後にトリガーするカスタムビジネスロジックと計算 (例: メール、外部APIへの呼び出しなど)|
| [Normalizers](serialization.md#decorating-a-serializer-and-adding-extra-data)| クライアントに送信されるリソースをカスタマイズする (JSON ドキュメントにフィールドを追加したり、コードや日付をエンコードしたり)|
| [Filters](filters.md) | コレクション用のフィルタを作成し、自動的にドキュメント化 (OpenAPI, GraphQL, Hydra) |
| [Serializer Context Builders](serialization.md#changing-the-serialization-context-dynamically) | Serialization コンテキスト（グループなど）を動的に変更する |
| [Messenger Handlers](messenger.md) | 100% カスタム、RPC、非同期、サービス指向のエンドポイントを作成する (メッセンジャーの統合は REST と GraphQL の両方に対応していますが、カスタムコントローラは REST でしか動作しないため、カスタムコントローラの代わりに使用する必要があります) |
| [DTOs and Data Transformers](dto.md) | 特定のクラスを使用して、操作に関連する入力または出力データ構造を表現する |
| [Kernel Events](events.md) | HTTP リクエストまたはレスポンスをカスタマイズする (REST のみ。可能であれば他の拡張ポイントを優先します) |

## Doctrine 固有の拡張ポイント

| Extension Point                                            | Usage                                                                                     |
|------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| [Extensions](extensions.md)                                | DQL クエリを変更するためのクエリビルダへのアクセス                                                |
| [Filters](filters.md#doctrine-orm-and-mongodb-odm-filters) | フィルタのドキュメント（OpenAPI、GraphQL、Hydra）を追加し、DQLクエリに自動的に適用する               |

## コンポジションを利用した組込みインフラの活用 

ほとんどの API Platform クラスは `final` としてマークされていますが、組み込みのサービスは [Composition](https://en.wikipedia.org/wiki/Composition_over_inheritance) で再利用やカスタマイズが簡単にできます 。

たとえば、リソースがパースされた後にメールを送りたいが、Doctrine ORM のネイティブの [データパーシスタ](data-persisters.md) の恩恵を受けたい場合、[Decorator デザインパターン](https://en.wikipedia.org/wiki/Decorator_pattern#PHP) を使ってネイティブのデータパーシスタを独自のクラスでラップしてメールを送ります。

既存の API Platform サービスをデコレータで置き換えるには、[サービスのデコレーション方法](../Symfony/service_decoration.md)を確認してください。