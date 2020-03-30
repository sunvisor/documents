# データパーシスタ

> [オリジナル](https://api-platform.com/docs/core/data-persisters/)

[Doctrine ORM](https://www.doctrine-project.org/projects/orm.html) を使ったデータパーシスタがライブラリに含まれており、デフォルトで有効になっています。これは [Doctrine エンティティ](https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/basic-mapping.html) としてマッピングされたオブジェクトの永続化と削除を行うことができます。
[Doctrine MongoDB ODM](https://www.doctrine-project.org/projects/mongodb-odm.html) のデータパーシスタも含まれており、[MongoDB documentation](mongodb.md) を参照して有効にすることができます。

`POST`, `PUT`, `PATCH`, `DELETE` [オペレーション](operation.md)の実行中にアプリケーションの状態を変更するために、API Platform は **データパーシスタ** と呼ばれるクラスを使用します。
データパーシスタは、APIリソースとしてマークされたクラスのインスタンスを受け取ります (通常は `@ApiResource` アノテーションを使用します)。
このインスタンスには、[デシリアライズ処理](serialization.md)の際にクライアントから送信されたデータが格納されます。

しかし、あなたは以下のようなことをしたいかもしれません。

* 他の永続化レイヤー (Elasticsearch、外部のWebサービスなど) にデータを保存する
* データベースにマッピングされた内部モデルを API を通して公開したくない
* [CQRS](https://martinfowler.com/bliki/CQRS.html) のようなパターンを実装することで、[読み取りオペレーション](data-providers.md)と更新のための別のモデルを使用する

カスタム データパーシスタを使用することができます。プロジェクトには、必要に応じていくつでもデータパーシスタを含めることができます。与えられたリソースのデータを最初に永続化できるものが使用されます。

## カスタムデータパーシスタの作成

データパーシスタを作成するには、[`ContextAwareDataPersisterInterface`](https://github.com/api-platform/core/blob/master/src/DataPersister/ContextAwareDataPersisterInterface.php) を実装する必要があります。
このインタフェースは3つのメソッドのみを定義しています。

* `persist`: 与えられたデータを作成または更新する。
* `remove`: 与えられたデータを削除する。
* `supports`: 指定したデータがこのデータパーシスタでサポートされているかどうかをチェックする。

次に実装例を紹介します。

```php
namespace App\DataPersister;

use ApiPlatform\Core\DataPersister\ContextAwareDataPersisterInterface;
use App\Entity\BlogPost;

final class BlogPostDataPersister implements ContextAwareDataPersisterInterface
{
    public function supports($data, array $context = []): bool
    {
        return $data instanceof BlogPost;
    }

    public function persist($data, array $context = [])
    {
      // 永続性レイヤーを呼び出して $data を保存します 
      return $data;
    }

    public function remove($data, array $context = [])
    {
      // 永続性レイヤーを呼び出して $data を削除します
    }
}
```

サービスのオートワイヤリングとオートコンフィグレーションが有効になっていれば (デフォルトで有効になっています)、完了です。

そうではなくカスタムの依存性注入設定を使用している場合は、対応するサービスを登録して `api_platform.data_persister` タグを追加する必要があります。パーシスタの順番を決めるために`priority` 属性を使用できます。

```yaml
# api/config/services.yaml
services:
    # ...
    App\DataPersister\BlogPostDataPersister: ~
        # autoconfiguration が無効のときのみコメントを外す
        #tags: [ 'api_platform.data_persister' ]
```

データパーシスタのメソッドに `$context` を必要としない場合は、代わりに [`DataPersisterInterface`](https://github.com/api-platform/core/blob/master/src/DataPersister/DataPersisterInterface.php) を実装することに注意してください。

## ビルトインデータパーシスタをデコレートする

カスタムビジネスロジックを永続化の前後に実行したい場合は、組み込みのデータパーシスタを[デコレート](https://symfony.com/doc/current/service_container/service_decoration.html)することで実現できます。

ネイティブの Doctrine ORM データパーシスタを使ったプロジェクトで、REST の `POST` や GraphQL の `create` 操作の後に新しいユーザーにウェルカムメールを送る実装例を示します。

```php
namespace App\DataPersister;

use ApiPlatform\Core\DataPersister\DataPersisterInterface;
use ApiPlatform\Core\DataPersister\ContextAwareDataPersisterInterface;
use App\Entity\User;
use Symfony\Component\Mailer\MailerInterface;

final class UserDataPersister implements ContextAwareDataPersisterInterface
{
    private $decorated;
    private $mailer;

    public function __construct(DataPersisterInterface $decorated, MailerInterface $mailer)
    {
        $this->decorated = $decorated;
        $this->mailer = $mailer;
    }

    public function supports($data, array $context = []): bool
    {
        return $data instanceof User;
    }

    public function persist($data, array $context = [])
    {
        $result = $this->decorated->persist($data, $context);

        if (
            ($context['collection_operation_name'] ?? null) === 'post' ||
            ($context['graphql_operation_name'] ?? null) === 'create'
        ) {
            $this->sendWelcomeEmail($data);
        }

        return $result;
    }

    public function remove($data, array $context = [])
    {
        return $this->decorated->remove($data, $context);
    }

    private function sendWelcomeEmail(User $user)
    {
        // Your welcome email logic...
        // $this->mailer->send(...);
    }
}
```

サービスのオートワイヤリングとオートコンフィグレーションを有効にしても、デコレーションを設定する必要があります。

```yaml
# api/config/services.yaml
services:
    # ...
    App\DataPersister\UserDataPersister:
        decorates: 'api_platform.doctrine.orm.data_persister'
        # autoconfiguration が無効のときのみコメントを外す
        #arguments: ['@App\DataPersister\UserDataPersister.inner']
        #tags: [ 'api_platform.data_persister' ]
```
