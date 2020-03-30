# データプロバイダー

> [オリジナル](https://api-platform.com/docs/core/data-providers/)

API が公開しているデータを取得するために、API Platform は **データプロバイダー** と呼ばれるクラスを利用します。
データベースからデータを取得するための [Doctrine ORM](https://www.doctrine-project.org/projects/orm.html) を利用したデータプロバイダ、ドキュメントデータベースからデータを取得するための [Doctrine MongoDB ODM](https://www.doctrine-project.org/projects/mongodb-odm.html) を利用したデータプロバイダ、Elasticsearch クラスタからデータを取得するための [Elasticsearch-PHP](https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/index.html) を利用したデータプロバイダがライブラリに含まれています。
最初のものはデフォルトで有効になっています。
これらのデータプロバイダは、ページ化されたコレクションとフィルタをネイティブにサポートしています。
これらはそのまま使用することができ、一般的な用途に完全に適しています。

しかし、別の永続化レイヤーやウェブサービスなど、他のソースからデータを取得したい場合もあります。
そのような場合には、カスタムデータプロバイダを使用することができます。
プロジェクトには、必要なだけのデータプロバイダを含めることができます。与えられたリソースのデータを最初に取得できるものが使用されます。

与えられたリソースに対して、2種類のインターフェースを実装することができます。

* [`ColllectionDataProviderInterface`](https://github.com/api-platform/core/blob/master/src/DataProvider/CollectionDataProviderInterface.php)
  はコレクションを取得する際に使用されます。
* [`ItemDataProviderInterface`](https://github.com/api-platform/core/blob/master/src/DataProvider/ItemDataProviderInterface.php)
  はアイテムを取得する際に使用します。

どちらの実装も、単一のリソースやオペレーションに効果を限定したい場合は、['RestrictedDataProviderInterface'](https://github.com/api-platform/core/blob/master/src/DataProvider/RestrictedDataProviderInterface.php)と呼ばれる3つ目の、オプションのインターフェースを実装することができます。

以下の例では、`App\Entity\BlogPost`というエンティティクラスのカスタムデータプロバイダを作成します。
Doctrine 関連のエンティティではない場合、正しく動作するためには、`@ApiProperty(identifier=true)`を使って識別子プロパティにフラグを立てる必要があることに注意してください( [Entity Identifier Case](serialization.md#entity-identifier-case)も参照してください)。

## カスタム コレクション データプロバイダ

まず、`BlogPostCollectionDataProvider`は、[`CollectionDataProviderInterface`](https://github.com/api-platform/core/blob/master/src/DataProvider/CollectionDataProviderInterface.php)を実装しなければなりません。

`getCollection` メソッドは、`array`, `Traversable` または [`ApiPlatformCoreDataProvider\PaginatorInterface`](https://github.com/api-platform/core/blob/master/src/DataProvider/PaginatorInterface.php) インスタンスを返す必要があります。
データがない場合は、空の配列を返します。

```php
<?php
// api/src/DataProvider/BlogPostCollectionDataProvider.php

namespace App\DataProvider;

use ApiPlatform\Core\DataProvider\CollectionDataProviderInterface;
use ApiPlatform\Core\DataProvider\RestrictedDataProviderInterface;
use ApiPlatform\Core\Exception\ResourceClassNotSupportedException;
use App\Entity\BlogPost;

final class BlogPostCollectionDataProvider implements CollectionDataProviderInterface, RestrictedDataProviderInterface
{
    public function supports(string $resourceClass, string $operationName = null, array $context = []): bool
    {
        return BlogPost::class === $resourceClass;
    }

    public function getCollection(string $resourceClass, string $operationName = null): \Generator
    {
        // Retrieve the blog post collection from somewhere
        yield new BlogPost(1);
        yield new BlogPost(2);
    }
}
```

デフォルトの設定を使用した場合、[オートワイヤリング](https://symfony.com/doc/current/service_container/autowiring.html)により、対応するサービスが自動的に登録されます。
サービスを明示的に宣言したり、カスタム優先度を設定したりするには、以下のスニペットを使用します。

```yaml
# api/config/services.yaml
services:
    # ...
    'App\DataProvider\BlogPostCollectionDataProvider':
        tags: [ { name: 'api_platform.collection_data_provider', priority: 2 } ]
        # Autoconfiguration must be disabled to set a custom priority
        autoconfigure: false
```

サービスに `api_platform.collection_data_provider` というタグを付けると、API Platform Core が自動的にこのデータプロバイダを登録して利用できるようになります。オプションの属性 `priority` は、データプロバイダを呼び出す順番を定義することができます。
最初に `ApiPlatformCore\\ExceptionResourceClassNotSupportedException` が発生しなかったデータプロバイダが使用されます。

## カスタム アイテム データプロバイダ

アイテムデータプロバイダの場合も同様の処理を行います。アイテムデータプロバイダは、[`ItemDataProviderInterface`](https://github.com/api-platform/core/blob/master/src/DataProvider/ItemDataProviderInterface.php)インターフェイスを実装した`BlogPostItemDataProvider`を作成します。

`getItem` メソッドは結果が見つからなかった場合に `null` を返すことができます。

```php
<?php
// api/src/DataProvider/BlogPostItemDataProvider.php

namespace App\DataProvider;

use ApiPlatform\Core\DataProvider\ItemDataProviderInterface;
use ApiPlatform\Core\DataProvider\RestrictedDataProviderInterface;
use ApiPlatform\Core\Exception\ResourceClassNotSupportedException;
use App\Entity\BlogPost;

final class BlogPostItemDataProvider implements ItemDataProviderInterface, RestrictedDataProviderInterface
{
    public function supports(string $resourceClass, string $operationName = null, array $context = []): bool
    {
        return BlogPost::class === $resourceClass;
    }

    public function getItem(string $resourceClass, $id, string $operationName = null, array $context = []): ?BlogPost
    {
        // Retrieve the blog post item from somewhere then return it or null if not found
        return new BlogPost($id);
    }
}
```

サービスのオートワイヤリングとオートコンフィグレーションが有効になっていれば (デフォルトではそうなっています) 完了です。

それ以外の場合は、カスタムの依存性注入設定を使用する場合は、対応するサービスを登録して `api_platform.item_data_provider` タグを追加する必要があります。
コレクションのデータ プロバイダのように、`priority` 属性を使ってプロバイダの順番を決めることができます。

```yaml
# api/config/services.yaml
services:
    # ...
    'App\DataProvider\BlogPostItemDataProvider': ~
        # autoconfigration が無効になっている場合のみコメントを外してください。
        #tags: [ 'api_platform.item_data_provider' ]
```

## `ItemDataProvider` にシリアライザを注入する

場合によっては、`DataProvider` に `Serializer` を注入する必要があるかもしれません。
`CollectionDataProvider` は問題ありませんが、 `ItemDataProvider` に注入すると `CircularReferenceException` がスローされます。
このため、`SerializerAwareDataProviderInterface`を実装します。

```php
<?php
// api/src/DataProvider/BlogPostItemDataProvider.php

namespace App\DataProvider;

use ApiPlatform\Core\DataProvider\ItemDataProviderInterface;
use ApiPlatform\Core\DataProvider\SerializerAwareDataProviderInterface;
use ApiPlatform\Core\DataProvider\SerializerAwareDataProviderTrait;
use ApiPlatform\Core\Exception\ResourceClassNotSupportedException;
use App\Entity\BlogPost;

final class BlogPostItemDataProvider implements ItemDataProviderInterface, SerializerAwareDataProviderInterface
{
    use SerializerAwareDataProviderTrait;

    public function supports(string $resourceClass, string $operationName = null, array $context = []): bool
    {
        return BlogPost::class === $resourceClass;
    }

    public function getItem(string $resourceClass, $id, string $operationName = null, array $context = []): ?BlogPost
    {
        // 好きな場所からカスタムフォーマットでデータを取得
        $data = '...';

        // シリアライザを使用してデータをデシリアライズする
        return $this->getSerializer()->deserialize($data, BlogPost::class, 'custom');
    }
}
```

## 拡張機能（ページネーション、フィルタ、EagerLoading など）の注入

API Platformには、カスタム DataProvider で再利用できる拡張機能がいくつか用意されています。
拡張機能にはいくつかの種類があり、それらについては [ドキュメントの独自の章] (extensions.md) で詳しく説明されています。
拡張機能はタグ付けされたサービスなので、[タグ付けされたサービスのインジェクション](https://symfony.com/blog/new-in-symfony-3-4-simpler-injection-of-tagged-services)を使用することができます。

```yaml
services:
    'App\DataProvider\BlogPostItemDataProvider':
        arguments:
          $itemExtensions: !tagged api_platform.doctrine.orm.query_extension.item
```

XML では

```xml
<services>
    <service id="App\DataProvider\BlogPostItemDataProvider">
        <argument key="$itemExtensions" type="tagged" tag="api_platform.doctrine.orm.query_extension.item" />
    </service>
</services>
```

これであなたのデータプロバイダはコア拡張機能にアクセスできるようになります。

```php
<?php
// api/src/DataProvider/BlogPostItemDataProvider.php

namespace App\DataProvider;

use ApiPlatform\Core\Bridge\Doctrine\Orm\Util\QueryNameGenerator;
use ApiPlatform\Core\DataProvider\ItemDataProviderInterface;
use ApiPlatform\Core\DataProvider\RestrictedDataProviderInterface;
use ApiPlatform\Core\Exception\ResourceClassNotSupportedException;
use App\Entity\BlogPost;
use Doctrine\Common\Persistence\ManagerRegistry;

final class BlogPostItemDataProvider implements ItemDataProviderInterface, RestrictedDataProviderInterface
{
    private $itemExtensions;
    private $managerRegistry;

    public function __construct(ManagerRegistry $managerRegistry, iterable $itemExtensions)
    {
      $this->managerRegistry = $managerRegistry;
      $this->itemExtensions = $itemExtensions;
    }

    public function supports(string $resourceClass, string $operationName = null, array $context = []): bool
    {
        return BlogPost::class === $resourceClass;
    }

    public function getItem(string $resourceClass, $id, string $operationName = null, array $context = []): ?BlogPost
    {
        $manager = $this->managerRegistry->getManagerForClass($resourceClass);
        $repository = $manager->getRepository($resourceClass);
        $queryBuilder = $repository->createQueryBuilder('o');
        $queryNameGenerator = new QueryNameGenerator();
        $identifiers = ['id' => $id];

        foreach ($this->itemExtensions as $extension) {
            $extension->applyToItem($queryBuilder, $queryNameGenerator, $resourceClass, $identifiers, $operationName, $context);
            if ($extension instanceof QueryResultItemExtensionInterface && $extension->supportsResult($resourceClass, $operationName, $context))                 {
                return $extension->getResult($queryBuilder, $resourceClass, $operationName, $context);
            }
        }

        return $queryBuilder->getQuery()->getOneOrNullResult();
    }
}

```

## コミュニティ データプロバイダー

組み込みの Doctrine システムを使いたくない場合は、API Platform との統合を提供する別のアプローチもあります。
* [Pomm Data Provider](https://github.com/pomm-project/pomm-api-platform): ([Pomm](http://www.pomm-project.org/) は PostgreSQL データベースに特化したデータベースアクセスフレームワークです。
