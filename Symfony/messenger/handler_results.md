ハンドラーから結果を得る
=================================

メッセージが処理されると、`Symfony\Component\Messenger\Middleware\HandleMessageMiddleware` は、メッセージを処理した各オブジェクトに、`Symfony\Component\MessengerStamp\HandledStamp` を追加します。これを使用して、ハンドラによって返された値を取得することができます。

```php
    use Symfony\Component\Messenger\MessageBusInterface;
    use Symfony\Component\Messenger\Stamp\HandledStamp;

    $envelope = $messageBus->dispatch(SomeMessage());

    // 最後のメッセージハンドラが返した値を取得します。
    $handledStamp = $envelope->last(HandledStamp::class);
    $handledStamp->getResult();

    // またはすべてのハンドラの情報を取得する
    $handledStamps = $envelope->all(HandledStamp::class);
```

コマンドとクエリバスの操作
----------------------------------

Messenger コンポーネントは、コマンドとクエリバスがアプリケーションの中心部分である CQRS アーキテクチャで使用できます。
Martin Fowler氏の[CQRSに関する記事](https://martinfowler.com/bliki/CQRS.html)を読んでみてください。また、複数のバスを設定する方法については、</messenger/multiple_buses> を参照してください。

クエリは通常同期的に処理され、一度処理されることが期待されるので、ハンドラから結果を取得することは一般的な必要性です。

同期的に処理するときにハンドラの結果を取得するために `Symfony\Component\Messenger\HandleTrait` が存在します。これはまた、正確に1つのハンドラが登録されていることを保証します。`HandleTrait` は `$messageBus` プロパティを持つ任意のクラスで使うことができます。

```php
    // src/Action/ListItems.php
    namespace App\Action;

    use App\Message\ListItemsQuery;
    use App\MessageHandler\ListItemsQueryResult;
    use Symfony\Component\Messenger\HandleTrait;
    use Symfony\Component\Messenger\MessageBusInterface;

    class ListItems
    {
        use HandleTrait;

        public function __construct(MessageBusInterface $messageBus)
        {
            $this->messageBus = $messageBus;
        }

        public function __invoke()
        {
            $result = $this->query(new ListItemsQuery(/* ... */));

            // Do something with the result
            // ...
        }

        // Creating such a method is optional, but allows type-hinting the result
        private function query(ListItemsQuery $query): ListItemsResult
        {
            return $this->handle($query);
        }
    }
```

したがって、この特性を使ってコマンドやクエリバスクラスを作成することができます。例えば、特別な `QueryBus` クラスを作成し、`MessageBusInterface` の代わりにクエリバスの振る舞いが必要な場所に注入することができます。

```php
    // src/MessageBus/QueryBus.php
    namespace App\MessageBus;

    use Symfony\Component\Messenger\Envelope;
    use Symfony\Component\Messenger\HandleTrait;
    use Symfony\Component\Messenger\MessageBusInterface;

    class QueryBus
    {
        use HandleTrait;

        public function __construct(MessageBusInterface $messageBus)
        {
            $this->messageBus = $messageBus;
        }

        /**
         * @param object|Envelope $query
         *
         * @return mixed The handler returned value
         */
        public function query($query)
        {
            return $this->handle($query);
        }
    }
```
