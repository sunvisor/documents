メッセンジャーコンポーネント
=======================

> メッセンジャーコンポーネントは、アプリケーションが他のアプリケーションとの間で、またはメッセージキューを介してメッセージを送受信するのに役立ちます。
>
> このコンポーネントは、Matthias Noback氏の一連の[コマンドバスに関するブログ記事](https://matthiasnoback.nl/tags/command%20bus/)と [SimpleBusプロジェクト](http://docs.simplebus.io/en/latest/)に大きく触発されています。

この記事では、Messenger の機能を PHP アプリケーションの独立したコンポーネントとして使う方法を説明します。symfony アプリケーションでメッセンジャーを使う方法を学ぶには `/messenger` の記事を読んでください。

インストール
------------

    $ composer require symfony/messenger

コンセプト
--------

<object data="../_images/components/messenger/overview.svg" type="image/svg+xml"></object>

**Sender**。 
何か*にメッセージをシリアライズして送信する責任があります。この何かとは、例えば、メッセージブローカーやサードパーティのAPIなどです。

**Receiver**: メッセージを取得してシリアライズし、ハンドラに転送する責任があります。 
メッセージの取得、シリアライズ、ハンドラへの転送を担当します。これは、メッセージキューのプラーやAPIエンドポイントなどになります。

**Handler**。 
メッセージに適用されるビジネスロジックを使用してメッセージを処理する責任があります。ハンドラは `HandleMessageMiddleware` ミドルウェアによって呼び出されます。

**Middleware**:  
ミドルウェアは、バスを介してディスパッチされている間、メッセージとそのラッパー(エンベロープ)にアクセスすることができます。文字通り「中間のソフトウェア」*ですが、これらはアプリケーションの中核的な問題(ビジネスロジック)ではありません。その代わりに、アプリケーション全体に適用され、メッセージバス全体に影響を与える、横断的な問題です。例えば、ロギング、メッセージの検証、トランザクションの開始などです。ミドルウェアはまた、チェーンの次のミドルウェアを呼び出す責任があります。つまり、エンベロープにスタンプを追加したり、交換したりして、エンベロープを微調整したり、ミドルウェアチェーンを中断したりすることができます。ミドルウェアは、メッセージが最初にディスパッチされたときと、メッセージがトランスポートから受信されたときの両方に呼び出されます。

**Envelope**:  
メッセンジャーに特化したコンセプトで、メッセージバスの中にメッセージを包み込むことで、メッセージバスの内部に完全な柔軟性を与え、*envelopeスタンプ*を介して内部に有用な情報を追加することができます。

**Envelope Stamps**:  
メッセージに添付する必要がある情報の一部: トランスポートに使用するシリアライザコンテキスト、受信したメッセージを識別するマーカー、またはミドルウェアやトランスポートレイヤーが使用する可能性があるメタデータの種類。

バス
---

バスはメッセージをディスパッチするために使用されます。バスの動作は、その順序付けされたミドルウェアスタックにあります。コンポーネントには利用できるミドルウェアのセットが付属しています。

Symfony の FrameworkBundle でメッセージバスを使う場合、次のミドルウェアが設定されます。

1.  `Symfony\Component\Messenger\Middleware\SendMessageMiddleware` (は非同期処理を有効にし、ロガーを渡した場合にメッセージの処理をログに記録します。)
2.  `Symfony\Component\Messenger\Middleware\HandleMessageMiddleware` (登録されたハンドラを呼び出します。)

例:

```php
    use App\Message\MyMessage;
    use App\MessageHandler\MyMessageHandler;
    use Symfony\Component\Messenger\Handler\HandlersLocator;
    use Symfony\Component\Messenger\MessageBus;
    use Symfony\Component\Messenger\Middleware\HandleMessageMiddleware;

    $handler = new MyMessageHandler();

    $bus = new MessageBus([
        new HandleMessageMiddleware(new HandlersLocator([
            MyMessage::class => [$handler],
        ])),
    ]);

    $bus->dispatch(new MyMessage(/* ... */));
```

Note

すべてのミドルウェアは `Symfony\Component\Messenger\Middleware\MiddlewareInterface` を実装する必要があります。

Handlers
--------

バスにディスパッチされると、メッセージは "メッセージハンドラ" によって処理されます。メッセージハンドラとは、メッセージに必要な処理を行う PHP の callable (つまり、関数やクラスのインスタンス) のことです。

```php
    namespace App\MessageHandler;

    use App\Message\MyMessage;

    class MyMessageHandler
    {
        public function __invoke(MyMessage $message)
        {
            // Message processing...
        }
    }
```

メッセージにメタデータを追加する  (エンベロープ)
---------------------------------------

メッセージにメタデータや設定を追加する必要がある場合、`Symfony\Component\Messenger\Envelope` クラスでメッセージをラップしてスタンプを追加します。たとえば、メッセージがトランスポートレイヤーを通過するときに使われるシリアライゼーショングループを設定するには、`SerializerStamp` スタンプを使います。

```php
    use Symfony\Component\Messenger\Envelope;
    use Symfony\Component\Messenger\Stamp\SerializerStamp;

    $bus->dispatch(
        (new Envelope($message))->with(new SerializerStamp([
            // groups are applied to the whole message, so make sure
            // to define the group for every embedded object
            'groups' => ['my_serialization_groups'],
        ]))
    );
```

現時点では、Symfony Messengerには以下のような封筒のスタンプが組み込まれています。

1.  `Symfony\Component\Messenger\Stamp\SerializerStamp`, トランスポートが使用するシリアライゼーショングループを設定する
2.  `Symfony\Component\Messenger\Stamp\ValidationStamp`, を使用して、検証ミドルウェアが有効になっているときに使用する検証グループを設定します。
3.  `Symfony\Component\Messenger\Stamp\ReceivedStamp`, トランスポートから受信したメッセージをマークする内部スタンプ。
4.  `Symfony\Component\Messenger\Stamp\SentStamp`, 特定の送信者によって送信されたメッセージをマークするスタンプ。Symfony\Component\Messenger\TransportSender\SendersLocator`から利用できる場合、送信者FQCNとエイリアスへのアクセスを許可する。
5.  `Symfony\Component\Messenger\Stamp\HandledStamp`, メッセージを特定のハンドラで処理されたものとしてマークするスタンプ。ハンドラの戻り値とハンドラ名にアクセスできます。

ミドルウェアで直接メッセージを扱うのではなく、エンベロープを受け取ります。そのため、封筒の中身やスタンプを検査したり、何かを追加したりすることができます。

```php
    use App\Message\Stamp\AnotherStamp;
    use Symfony\Component\Messenger\Envelope;
    use Symfony\Component\Messenger\Middleware\MiddlewareInterface;
    use Symfony\Component\Messenger\Middleware\StackInterface;
    use Symfony\Component\Messenger\Stamp\ReceivedStamp;

    class MyOwnMiddleware implements MiddlewareInterface
    {
        public function handle(Envelope $envelope, StackInterface $stack): Envelope
        {
            if (null !== $envelope->last(ReceivedStamp::class)) {
                // Message just has been received...

                // You could for example add another stamp.
                $envelope = $envelope->with(new AnotherStamp(/* ... */));
            } else {
                // Message was just originally dispatched
            }

            return $stack->next()->handle($envelope, $stack);
        }
    }
```

上記の例では、メッセージがちょうど受信された(すなわち、少なくとも1つの `ReceivedStamp` スタンプを持っている)場合、メッセージを追加のスタンプで次のミドルウェアに転送します。Symfony\Component\Messenger\Stamp\StampInterface` を実装することで、独自のスタンプを作成することができます。

エンベロープ上のすべてのスタンプを調べたい場合は、`$envelope->all()`メソッドを使ってください。あるいは、このメソッドの最初のパラメータに FQCN を指定することで、特定のタイプのスタンプを繰り返し処理することもできます (例: `$envelope->all(ReceivedStamp::class)`)。

Note

任意のスタンプは `Symfony\Component\Messenger\Transport\Serialization\Serializer` ベースのシリアライザを使ってトランスポートを通す場合、symfony Serializer コンポーネントを使ってシリアライズ可能でなければなりません。

Transports
----------

メッセージを送受信するためには、トランスポートを設定する必要があります。トランスポートはメッセージブローカやサードパーティとの通信を担当します。

### Your own Sender

既に `ImportantAction` メッセージがメッセージバスを通過し、ハンドラによって処理されていることを想像してみてください。さて、このメッセージをメールとして送信したいとします (`Mime </components/mime>` と `Mailer </components/mailer>` コンポーネントを使用します)。

Symfony\Component\Messenger\Transport\SenderSenderInterface`を使えば、独自のメッセージ送信者を作ることができます。

```php
    namespace App\MessageSender;

    use App\Message\ImportantAction;
    use Symfony\Component\Mailer\MailerInterface;
    use Symfony\Component\Messenger\Envelope;
    use Symfony\Component\Messenger\Transport\Sender\SenderInterface;
    use Symfony\Component\Mime\Email;

    class ImportantActionToEmailSender implements SenderInterface
    {
        private $mailer;
        private $toEmail;

        public function __construct(MailerInterface $mailer, string $toEmail)
        {
            $this->mailer = $mailer;
            $this->toEmail = $toEmail;
        }

        public function send(Envelope $envelope): Envelope
        {
            $message = $envelope->getMessage();

            if (!$message instanceof ImportantAction) {
                throw new \InvalidArgumentException(sprintf('This transport only supports "%s" messages.', ImportantAction::class));
            }

            $this->mailer->send(
                (new Email())
                    ->to($this->toEmail)
                    ->subject('Important action made')
                    ->html('<h1>Important action</h1><p>Made by '.$message->getUsername().'</p>')
            );

            return $envelope;
        }
    }
```

### あなた自身のレシーバー

レシーバはソースからメッセージを取得してアプリケーションにディスパッチする役割を果たします。

アプリケーションで `NewOrder` メッセージを使ってすでにいくつかの「注文」を処理したとします。サードパーティやレガシーアプリケーションと統合したいとしますが、APIを使うことができず、新しい注文を含む共有のCSVファイルを使う必要があります。

このCSVファイルを読み込んで `NewOrder` メッセージを送信します。必要なのは、独自のCSVレシーバを書くことだけです。

```php
    namespace App\MessageReceiver;

    use App\Message\NewOrder;
    use Symfony\Component\Messenger\Envelope;
    use Symfony\Component\Messenger\Exception\MessageDecodingFailedException;
    use Symfony\Component\Messenger\Transport\Receiver\ReceiverInterface;
    use Symfony\Component\Serializer\SerializerInterface;

    class NewOrdersFromCsvFileReceiver implements ReceiverInterface
    {
        private $serializer;
        private $filePath;

        public function __construct(SerializerInterface $serializer, string $filePath)
        {
            $this->serializer = $serializer;
            $this->filePath = $filePath;
        }

        public function get(): iterable
        {
            // Receive the envelope according to your transport ($yourEnvelope here),
            // in most cases, using a connection is the easiest solution.
            if (null === $yourEnvelope) {
                return [];
            }

            try {
                $envelope = $this->serializer->decode([
                    'body' => $yourEnvelope['body'],
                    'headers' => $yourEnvelope['headers'],
                ]);
            } catch (MessageDecodingFailedException $exception) {
                $this->connection->reject($yourEnvelope['id']);
                throw $exception;
            }

            return [$envelope->with(new CustomStamp($yourEnvelope['id']))];
        }

        public function ack(Envelope $envelope): void
        {
            // Add information about the handled message
        }

        public function reject(Envelope $envelope): void
        {
            // In the case of a custom connection
            $this->connection->reject($this->findCustomStamp($envelope)->getId());
        }
    }
```

### 同じバスにある Receiver と Sender


同じバス上でメッセージの送受信を可能にし、無限ループを防ぐために、メッセージバスはメッセージエンベロープに `Symfony\Component\MessengerMessenger\Stamp\ReceivedStamp` スタンプを追加し、`Symfony\Component\MessengerMiddleware\SendMessageMiddleware` ミドルウェアは、これらのメッセージをトランスポートに再度ルーティングしてはならないことを知っているだろう。

