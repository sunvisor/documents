サービスメソッド呼び出しとセッターインジェクション
=========================================

> [オリジナル](https://symfony.com/doc/current/service_container/calls.html)

Tip
: オートワイヤリングを使用している場合は、`@required` を使用することで[自動的にメソッドコールを設定](autowiring.md#autowiring-calls)することができます。 

通常は、コンストラクタを介して依存関係を注入することになります。しかし、特に依存関係がオプションである場合には "セッター注入" を使用したくなることもあるでしょう。例えば、以下のようになります。

```php
namespace App\Service;

use Psr\Log\LoggerInterface;

class MessageGenerator
{
    private $logger;

    public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    // ...
}
```

`setLogger` メソッドを呼び出すようにコンテナを設定するには、`calls` キーを使用します。

```yaml
# config/services.yaml
services:
    App\Service\MessageGenerator:
        # ...
        calls:
            - [setLogger, ['@logger']]
```

イミュータブルなサービスを提供するために、いくつかのクラスはイミュータブルなセッターを実装しています。このようなセッターは、呼び出されたオブジェクトを変更するのではなく、設定されたクラスの新しいインスタンスを返します。

```php
namespace App\Service;

use Psr\Log\LoggerInterface;

class MessageGenerator
{
    private $logger;

    /**
     * @return static
     */
    public function withLogger(LoggerInterface $logger)
    {
        $new = clone $this;
        $new->logger = $logger;

        return $new;
    }

    // ...
}
```

このメソッドは別のクローンされたインスタンスを返すので、このようなサービスを設定するには `withLogger` メソッドの戻り値を使用することになります (`$service = $service->withLogger($logger);` )。コンテナにそうするように指示するための設定は次のようになります。

```yaml
# config/services.yaml
services:
    App\Service\MessageGenerator:
        # ...
        calls:
            - withLogger: !returns_clone ['@logger']
```