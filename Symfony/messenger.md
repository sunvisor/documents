Messenger: 同期およびキューに入ったメッセージの処理
=========================================

メッセンジャーは、メッセージを送信する機能を備えたメッセージバスを提供し アプリケーション内ですぐに処理するか、トランスポートを介して送信してください。 (キューなど)を後から処理するようにします。より深く知りたい場合は `Messenger コンポーネントのドキュメント</components/messenger>`。

インストール
------------

ref:<span class="title-ref">Symfony Flex &lt;symfony-flex&gt;</span> を使っているアプリケーションでは、このコマンドを次のように実行します。 messenger をインストールします。

```     
$ composer require symfony/messenger
```

メッセージとハンドラーの作成
----------------------------

メッセンジャーの中心となるのは、作成する2つの異なるクラスです。(1) データを保持するメッセージクラスと、(2) メッセージがディスパッチされたときに呼び出されるハンドラクラスです。ハンドラクラスはメッセージクラスを読み込んで何らかのタスクを実行します。

メッセージクラスに特別な要件はありませんが、シリアル化が可能であることを除いては、次のようになります。

```php
// src/Message/SmsNotification.php namespace AppMessage;

class SmsNotification { private $content;

    public function __construct(string $content) { $this->content = $content; }

    public function getContent(): string { return $this->content; }

}
```

メッセージハンドラは、PHPの callable で、それを作成するための推奨される方法は、`Symfony\Component\\\Messenger\\Handler\MessageHandlerInterface` を実装し、メッセージクラス（またはメッセージインターフェイス）とタイプヒントされている `__invoke()` メソッド持つするクラスを作成することです。

```php
// src/MessageHandler/SmsNotificationHandler.php namespace AppMessageHandler;

use AppMessageSmsNotification; use SymfonyComponentMessengerHandlerMessageHandlerInterface;

class SmsNotificationHandler implements MessageHandlerInterface { public function \_\_invoke(SmsNotification $message) { // ... do some work - like sending an SMS message! } }
```

<span class="title-ref">autoconfiguration &lt;services-autoconfigure&gt;</span> と `SmsNotification` のタイプヒントのおかげで、`SmsNotification` メッセージがディスパッチされたときにこのハンドラが呼び出されるべきであることを symfony は知っています。ほとんどの場合、これが必要なすべてです。しかし、`手動でメッセージハンドラを設定することもできます <messenger-handler-config>` 。設定されたすべてのハンドラを見るには、実行してください。`autoconfiguration <services-autoconfigure>` と `SmsNotification` のタイプヒントのおかげで、`SmsNotification` メッセージがディスパッチされたときにこのハンドラが呼び出されることを symfony は知っています。ほとんどの場合、これが必要なすべてです。しかし、`手動でメッセージハンドラを設定することもできます <messenger-handler-config>` 。設定されたすべてのハンドラを見るには、以下を実行してください。

    $ php bin/console debug:messenger

Dispatching the Message
-----------------------

これで準備完了です! メッセージをディスパッチする (そしてハンドラを呼び出す) には、コントローラのように `MessageBusInterface` を介して `message_bus` サービスを注入します。

```php
    // src/Controller/DefaultController.php
    namespace App\Controller;

    use App\Message\SmsNotification;
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Messenger\MessageBusInterface;

    class DefaultController extends AbstractController
    {
        public function index(MessageBusInterface $bus)
        {
            // will cause the SmsNotificationHandler to be called
            $bus->dispatch(new SmsNotification('Look! I created a message!'));

            // or use the shortcut
            $this->dispatchMessage(new SmsNotification('Look! I created a message!'));

            // ...
        }
    }
```

Transports: Async/Queued Messages
---------------------------------

既定では、メッセージはディスパッチされるとすぐに処理されます。メッセージを非同期に処理したい場合は、トランスポートを設定することができます。トランスポートはメッセージを (例えばキューイングシステムに) 送信してから、`ワーカー <messenger-worker>` を介してメッセージを受け取ることができます。Messenger は `複数のトランスポート <messenger-transports-config>` をサポートしています。

Note

サポートされていないトランスポートを使いたい場合は、KafkaやAmazon SQS、Google Pub/Subのようなものをサポートしている[Enqueueのトランスポート](https://github.com/sroze/messenger-enqueue-transport)をチェックしてみてください。

トランスポートは "DSN" を使って登録されます。MessengerのFlexレシピのおかげで、`.env`ファイルにはすでにいくつかの例があります。

    # MESSENGER_TRANSPORT_DSN=amqp://guest:guest@localhost:5672/%2f/messages
    # MESSENGER_TRANSPORT_DSN=doctrine://default
    # MESSENGER_TRANSPORT_DSN=redis://localhost:6379/messages

必要なトランスポートをアンコメントしてください (あるいは `.env.local` で設定してください)。詳細は `messenger-transports-config` を参照のこと。

次に、`config/packages/messenger.yaml`で、この設定を利用する `async` というトランスポートを定義します。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            transports:
                async: "%env(MESSENGER_TRANSPORT_DSN)%"

                # or expanded to configure more options
                #async:
                #    dsn: "%env(MESSENGER_TRANSPORT_DSN)%"
                #    options: []

```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:transport name="async">%env(MESSENGER_TRANSPORT_DSN)%</framework:transport>

                <!-- or expanded to configure more options -->
                <framework:transport name="async"
                    dsn="%env(MESSENGER_TRANSPORT_DSN)%"
                >
                    <option key="...">...</option>
                </framework:transport>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'transports' => [
                'async' => '%env(MESSENGER_TRANSPORT_DSN)%',

                // or expanded to configure more options
                'async' => [
                   'dsn' => '%env(MESSENGER_TRANSPORT_DSN)%',
                   'options' => []
                ],
            ],
        ],
    ]);
```

### Routing Messages to a Transport

トランスポートを設定したので、メッセージをすぐに処理するのではなく、トランスポートに送信するように設定することができます。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            transports:
                async: "%env(MESSENGER_TRANSPORT_DSN)%"

            routing:
                # async is whatever name you gave your transport above
                'App\Message\SmsNotification':  async

```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:routing message-class="App\Message\SmsNotification">
                    <!-- async is whatever name you gave your transport above -->
                    <framework:sender service="async"/>
                </framework:routing>
            </framework:messenger>
        </framework:config>
    </container>
```

```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'routing' => [
                // async is whatever name you gave your transport above
                'App\Message\SmsNotification' => 'async',
            ],
        ],
    ]);
```

このおかげで、`App\MessageSmsNotification` は `async` トランスポートに送られ、そのハンドラはすぐに呼び出されません。`routing` の下でマッチしなかったメッセージはすぐに処理されます。

親クラスやインターフェイスを使ってクラスをルーティングすることもできます。また、複数のトランスポートにメッセージを送ることもできます。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            routing:
                # route all messages that extend this example base class or interface
                'App\Message\AbstractAsyncMessage': async
                'App\Message\AsyncMessageInterface': async

                'My\Message\ToBeSentToTwoSenders': [async, audit]

```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <!-- route all messages that extend this example base class or interface -->
                <framework:routing message-class="App\Message\AbstractAsyncMessage">
                    <framework:sender service="async"/>
                </framework:routing>
                <framework:routing message-class="App\Message\AsyncMessageInterface">
                    <framework:sender service="async"/>
                </framework:routing>
                <framework:routing message-class="My\Message\ToBeSentToTwoSenders">
                    <framework:sender service="async"/>
                    <framework:sender service="audit"/>
                </framework:routing>
            </framework:messenger>
        </framework:config>
    </container>
```
```php

    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'routing' => [
                // route all messages that extend this example base class or interface
                'App\Message\AbstractAsyncMessage' => 'async',
                'App\Message\AsyncMessageInterface' => 'async',
                'My\Message\ToBeSentToTwoSenders' => ['async', 'audit'],
            ],
        ],
    ]);
```

### Doctrine Entities in Messages

メッセージの中で Doctrine のエンティティを渡す必要がある場合、オブジェクトの代わりにエンティティの主キー (あるいはハンドラが実際に必要とする `email` などの関連情報) を渡したほうがよいでしょう。

```php
    // src/Message/NewUserWelcomeEmail.php
    namespace App\Message;

    class NewUserWelcomeEmail
    {
        private $userId;

        public function __construct(int $userId)
        {
            $this->userId = $userId;
        }

        public function getUserId(): int
        {
            return $this->userId;
        }
    }
```

そして、ハンドラの中で、新しいオブジェクトをクエリすることができます。

```php
    // src/MessageHandler/NewUserWelcomeEmailHandler.php
    namespace App\MessageHandler;

    use App\Message\NewUserWelcomeEmail;
    use App\Repository\UserRepository;
    use Symfony\Component\Messenger\Handler\MessageHandlerInterface;

    class NewUserWelcomeEmailHandler implements MessageHandlerInterface
    {
        private $userRepository;

        public function __construct(UserRepository $userRepository)
        {
            $this->userRepository = $userRepository;
        }

        public function __invoke(NewUserWelcomeEmail $welcomeEmail)
        {
            $user = $this->userRepository->find($welcomeEmail->getUserId());

            // ... send an email!
        }
    }
```

これは、エンティティに新鮮なデータが含まれていることを保証します。

### メッセージを同期的に処理する

メッセージが `どのルーティングルール<messenger-routing>` にもマッチしない場合、どのトランスポートにも送られず、すぐに処理されます。いくつかのケース( [ハンドラを異なるトランスポートにバインドする場合](#binding-handlers-to-ifferent-transports)のように)では、明示的にこれを扱う方が簡単で柔軟性があります: `sync` トランスポートを作成してそこにメッセージを「送信」してすぐに処理されるようにします。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            transports:
                # ... other transports

                sync: 'sync://'

            routing:
                App\Message\SmsNotification: sync

```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <!-- ... other transports -->

                <framework:transport name="sync" dsn="sync://"/>

                <framework:routing message-class="App\Message\SmsNotification">
                    <framework:sender service="sync"/>
                </framework:routing>
            </framework:messenger>
        </framework:config>
    </container>
```

```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'transports' => [
                // ... other transports

                'sync' => 'sync://',
            ],
            'routing' => [
                'App\Message\SmsNotification' => 'sync',
            ],
        ],
    ]);
```

### Creating your Own Transport

サポートされていないものからメッセージを送受信する必要がある場合は、独自のトランスポートを作成することもできます。詳しくは `/messenger/custom-transport` を参照してください。

Consuming Messages (Running the Worker)
---------------------------------------

メッセージがルーティングされたら、たいていの場合、メッセージを「消費」する必要があります。これは `messenger:consume` コマンドで行うことができます。

    $ php bin/console messenger:consume async

    # use -vv to see details about what's happening
    $ php bin/console messenger:consume async -vv

最初の引数は受信機の名前(カスタムサービスにルーティングした場合はサービスID)です。デフォルトでは、このコマンドは永遠に実行されます: トランスポート上の新しいメッセージを探して、それを処理します。このコマンドは「ワーカー」と呼ばれる。

### Deploying to Production

本番では、いくつか重要なことがあります。

**ワーカーの稼働を維持するために Supervisor を使う**
1つ以上の「ワーカー」を常時稼働させたいと思うでしょう。これを行うには、`Supervisor <messenger-supervisor>` のようなプロセス制御システムを使用します。

**ワーカーを永遠に走らせてはいけない**
サービスによっては (Doctrine の EntityManager のように) 時間の経過とともにメモリを消費するものもあります。そこで、ワーカーを永遠に実行させる代わりに、`messenger:consume --limit=10` のようなフラグを使って、ワーカーが終了する前に 10 個のメッセージだけを処理するように指示します (そうすると Supervisor は新しいプロセスを作成します)。他にも `--memory-limit=128M` や `--time-limit=3600` のようなオプションもあります。

**デプロイ時にワーカーを再起動する**
デプロイするたびに、すべてのワーカープロセスを再起動して、新しくデプロイされたコードを見るようにする必要があります。これを行うには、デプロイ時に `messenger:stop-workers` を実行してください。これは各ワーカーに、現在処理しているメッセージを終了して、 優雅にシャットダウンするように合図します。その後、Supervisor は新しいワーカープロセスを作成します。このコマンドは内部的に `app <cache-configuration-with-frameworkbundle>` キャッシュを使用します。

### 優先順位付けされたトランスポート

時には、あるタイプのメッセージはより高い優先度を持ち、他のメッセージよりも先に処理されるべきである。これを可能にするために、複数のトランスポートを作成して、異なるメッセージをそれらにルーティングすることができます。例えば、以下のようになります。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            transports:
                async_priority_high:
                    dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                    options:
                        # queue_name is specific to the doctrine transport
                        queue_name: high

                        # for AMQP send to a separate exchange then queue
                        #exchange:
                        #    name: high
                        #queues:
                        #    messages_high: ~
                        # or redis try "group"
                async_priority_low:
                    dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                    options:
                        queue_name: low

            routing:
                'App\Message\SmsNotification':  async_priority_low
                'App\Message\NewUserWelcomeEmail':  async_priority_high
```

```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:transport name="async_priority_high" dsn="%env(MESSENGER_TRANSPORT_DSN)%">
                    <framework:options>
                        <framework:queue>
                            <framework:name>Queue</framework:name>
                        </framework:queue>
                    </framework:options>
                </framework:transport>
                <framework:transport name="async_priority_low" dsn="%env(MESSENGER_TRANSPORT_DSN)%">
                    <option key="queue_name">low</option>
                </framework:transport>

                <framework:routing message-class="App\Message\SmsNotification">
                    <framework:sender service="async_priority_low"/>
                </framework:routing>
                <framework:routing message-class="App\Message\NewUserWelcomeEmail">
                    <framework:sender service="async_priority_high"/>
                </framework:routing>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'transports' => [
                'async_priority_high' => [
                    'dsn' => '%env(MESSENGER_TRANSPORT_DSN)%',
                    'options' => [
                        'queue_name' => 'high',
                    ],
                ],
                'async_priority_low' => [
                    'dsn' => '%env(MESSENGER_TRANSPORT_DSN)%',
                    'options' => [
                        'queue_name' => 'low',
                    ],
                ],
            ],
            'routing' => [
                'App\Message\SmsNotification' => 'async_priority_low',
                'App\Message\NewUserWelcomeEmail' => 'async_priority_high',
            ],
        ],
    ]);
```

その後、各トランスポートに対して個々のワーカーを実行したり、1人のワーカーに優先順位の高い順にメッセージを処理するように指示したりすることができます。

    $ php bin/console messenger:consume async_priority_high async_priority_low

ワーカーは常に `async_priority_high` で待機しているメッセージを最初に探します。
もしそれがなければ、`async_priority_low` からのメッセージを消費します。

### Supervisor の設定

Supervisor は、ワーカープロセスが*常に*実行されていることを保証するための優れたツールです (たとえそれが失敗したり、メッセージ制限に達したり、`messenger:stop-workers`のおかげで終了してしまったとしても)。例えば、Ubuntu にインストールするには、以下のようにします。

```
    $ sudo apt-get install supervisor
```

スーパバイザ設定ファイルは通常 `/etc/supervisor/conf.d` ディレクトリにあります。例えば、そこに新しい `messenger-worker.conf` ファイルを作成して、`messenger:consume` の 2 つのインスタンスが常に実行されていることを確認することができます。

    ;/etc/supervisor/conf.d/messenger-worker.conf
    [program:messenger-consume]
    command=php /path/to/your/app/bin/console messenger:consume async --time-limit=3600
    user=ubuntu
    numprocs=2
    startsecs=0
    autostart=true
    autorestart=true
    process_name=%(program_name)s_%(process_num)02d

引数 `async` をトランスポート (またはトランスポート) の名前を使用するように変更し、`user` をサーバ上の Unix ユーザに変更します。次に、Supervisor に設定を読み込んでワーカーを起動するように指示します。

```
    $ sudo supervisorctl reread

    $ sudo supervisorctl update

    $ sudo supervisorctl start messenger-consume:*
```

詳しくは [Supervisor のドキュメント](http://supervisord.org/) を参照。

リトライとエラー
------------------

トランスポートからメッセージを消費している間に例外が投げられた場合、それは自動的にトランスポートに再送されて再試行されます。デフォルトでは、メッセージは破棄されるか `失敗トランスポート <messenger-failure-transport>` に送られる前に3回再試行されます。失敗が一時的な問題によるものであった場合、各再試行は遅延されます。これらはすべてトランスポートごとに設定可能です。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            transports:
                async_priority_high:
                    dsn: '%env(MESSENGER_TRANSPORT_DSN)%'

                    # default configuration
                    retry_strategy:
                        max_retries: 3
                        # milliseconds delay
                        delay: 1000
                        # causes the delay to be higher before each retry
                        # e.g. 1 second delay, 2 seconds, 4 seconds
                        multiplier: 2
                        max_delay: 0
                        # override all of this with a service that
                        # implements Symfony\Component\Messenger\Retry\RetryStrategyInterface
                        # service: null
```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:transport name="async_priority_high" dsn="%env(MESSENGER_TRANSPORT_DSN)%?queue_name=high_priority">
                    <framework:retry-strategy max-retries="3" delay="1000" multiplier="2" max-delay="0"/>
                </framework:transport>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'transports' => [
                'async_priority_high' => [
                    'dsn' => '%env(MESSENGER_TRANSPORT_DSN)%',

                    // default configuration
                    'retry_strategy' => [
                        'max_retries' => 3,
                        // milliseconds delay
                        'delay' => 1000,
                        // causes the delay to be higher before each retry
                        // e.g. 1 second delay, 2 seconds, 4 seconds
                        'multiplier' => 2,
                        'max_delay' => 0,
                        // override all of this with a service that
                        // implements Symfony\Component\Messenger\Retry\RetryStrategyInterface
                        // 'service' => null,
                    ],
                ],
            ],
        ],
    ]);
```

### リトライの回避

ときおり、メッセージの処理に失敗した場合に、それが恒久的なものであり、再試行すべきではないことがわかっている場合があります。
`Symfony\\Component\\Messenger\\Exception\\UnrecoverableMessageHandlingException`
例外を投げると、メッセージは再試行されません。

### 失敗したメッセージの保存と再試行

メッセージが失敗した場合、メッセージは複数回再試行され(`max_retries`)、その後破棄されます。このような事態を避けるために、代わりに `failure_transport` を設定することができます。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            # after retrying, messages will be sent to the "failed" transport
            failure_transport: failed

            transports:
                # ... other transports

                failed: 'doctrine://default?queue_name=failed'

```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <!-- after retrying, messages will be sent to the "failed" transport -->
            <framework:messenger failure-transport="failed">
                <!-- ... other transports -->

                <framework:transport name="failed" dsn="doctrine://default?queue_name=failed"/>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            // after retrying, messages will be sent to the "failed" transport
            'failure_transport' => 'failed',

            'transports' => [
                // ... other transports

                'failed' => [
                    'dsn' => 'doctrine://default?queue_name=failed',
                ],
            ],
        ],
    ]);
```

この例では、メッセージの処理に 3 回失敗した場合 (デフォルトの `max_retries`)、そのメッセージは `failed` トランスポートに送られます。通常のトランスポートのように `messenger:consume failed` を使ってこれを消費することもできますが、通常は手動で失敗トランスポートのメッセージを見て、再試行することを選択することになります。

    # 障害トランスポート内のすべてのメッセージを見る
    $ php bin/console messenger:failed:show

    # 特定の障害の詳細を見る
    $ php bin/console messenger:failed:show 20 -vv

    # メッセージを1つずつ表示して再試行する
    $ php bin/console messenger:failed:retry -vv

    # 特定のメッセージを再試行する
    $ php bin/console messenger:failed:retry 20 30 --force

    # 再試行せずにメッセージを削除する
    $ php bin/console messenger:failed:remove 20

    # メッセージを再試行せずに削除し、削除する前に各メッセージを表示する
    $ php bin/console messenger:failed:remove 20 30 --show-messages

5.1

`--show-messages` オプションは symfony 5.1 で導入されました。

メッセージが再び失敗した場合、通常の `retry ルール <messenger-retries-failures>` により、失敗トランスポートに再送されます。最大リトライ回数に達すると、メッセージは永久に破棄されます。

トランスポートの設定
-----------------------

Messengerは、独自のオプションを持つ多くの異なるトランスポートタイプをサポートしています。

### AMQP Transport

5.1

symfony 5.1から、AMQPトランスポートは別のパッケージに移動しました。実行してインストールしてください。

    $ composer require symfony/amqp-messenger

amqp` トランスポートの設定は以下のようになります。

    # .env
    MESSENGER_TRANSPORT_DSN=amqp://guest:guest@localhost:5672/%2f/messages

symfony の組み込みの AMQP トランスポートを使うには、AMQP PHP エクステンションが必要です。

Note

デフォルトでは、トランスポートは、必要とされるすべてのエクスチェンジ、キュー、 およびバインディングキーを自動的に生成する。これは無効にすることができますが、いくつかの機能は正しく動作しないかもしれません (遅延キューのような)。

トランスポートには、交換を構成する方法、キューに鍵をバインドする方法など、多くの他のオプションがある。`Symfony\Component\Messenger\Bridge\Amqp\TransportTransportConnection` のドキュメントを参照のこと。

また、`Symfony\Component\Messenger\Bridge\Amqp\Transport\AmqpStamp` をエンベロープに追加することで、メッセージにAMQP固有の設定を行うこともできます。

```php
    use Symfony\Component\Messenger\Bridge\Amqp\Transport\AmqpStamp;
    // ...

    $attributes = [];
    $bus->dispatch(new SmsNotification(), [
        new AmqpStamp('custom-routing-key', AMQP_NOPARAM, $attributes)
    ]);
```

Caution

コンシューマは管理パネルに表示されない。これは、このトランスポートがブロッキングされている `\AmqpQueue::consume()` に依存していないからである。受信機がブロッキングされていると、`messenger:consume` コマンドの `--time-limit/--memory-limit` オプションや `messenger:stop-workers` コマンドの `--time-limit/--memory-limit` オプションは非効率になります。consume ワーカーは、処理すべきメッセージを受信するまで、および/または停止条件のいずれかに到達するまで繰り返し処理を行う責任があります。したがって、ワーカーの停止ロジックは、ブロッキングコールでスタックしている場合には到達できません。

### Doctrine トランスポート

5.1

Starting from Symfony 5.1, the Doctrine transport has moved to a separate package. Install it by running:

    $ composer require symfony/doctrine-messenger

The Doctrine transport can be used to store messages in a database table.

    # .env
    MESSENGER_TRANSPORT_DSN=doctrine://default

The format is `doctrine://<connection_name>`, in case you have multiple connections and want to use one other than the "default". The transport will automatically create a table named `messenger_messages` (this is configurable) when the transport is first used. You can disable that with the `auto_setup` option and set the table up manually by calling the `messenger:setup-transports` command.

Tip

To avoid tools like Doctrine Migrations from trying to remove this table because it's not part of your normal schema, you can set the `schema_filter` option:

    # config/packages/doctrine.yaml
    doctrine:
        dbal:
            schema_filter: '~^(?!messenger_messages)~'

The transport has a number of options:

    # config/packages/messenger.yaml
    framework:
        messenger:
            transports:
                async_priority_high: "%env(MESSENGER_TRANSPORT_DSN)%?queue_name=high_priority"
                async_normal:
                    dsn: "%env(MESSENGER_TRANSPORT_DSN)%"
                    options:
                        queue_name: normal_priority

    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:transport name="async_priority_high" dsn="%env(MESSENGER_TRANSPORT_DSN)%?queue_name=high_priority"/>
                <framework:transport name="async_priority_low" dsn="%env(MESSENGER_TRANSPORT_DSN)%">
                    <framework:options>
                        <framework:queue>
                            <framework:name>normal_priority</framework:name>
                        </framework:queue>
                    </framework:options>
                </framework:transport>
            </framework:messenger>
        </framework:config>
    </container>

    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'transports' => [
                'async_priority_high' => 'dsn' => '%env(MESSENGER_TRANSPORT_DSN)%?queue_name=high_priority',
                'async_priority_low' => [
                    'dsn' => '%env(MESSENGER_TRANSPORT_DSN)%',
                    'options' => [
                        'queue_name' => 'normal_priority'
                    ]
                ],
            ],
        ],
    ]);

Options defined under `options` take precedence over ones defined in the DSN.

<table><thead><tr class="header"><th>Option</th><th>Description</th><th>Default</th></tr></thead><tbody><tr class="odd"><td>table_name</td><td>Name of the table</td><td>messenger_messages</td></tr><tr class="even"><td><p>queue_name</p></td><td><p>Name of the queue (a column in the table, to use one table for multiple transports)</p></td><td><p>default</p></td></tr><tr class="odd"><td><p>redeliver_timeout</p><p>auto_setup</p></td><td><p>Timeout before retrying a message that's in the queue but in the "handling" state (if a worker died for some reason, this will occur, eventually you should retry the message) - in seconds. Whether the table should be created automatically during send / get.</p></td><td><p>3600</p><p>true</p></td></tr></tbody></table>

### Redis Transport

5.1

Starting from Symfony 5.1, the Redis transport has moved to a separate package. Install it by running:

    $ composer require symfony/redis-messenger

The Redis transport uses [streams](https://redis.io/topics/streams-intro) to queue messages.

    # .env
    MESSENGER_TRANSPORT_DSN=redis://localhost:6379/messages
    # Full DSN Example
    MESSENGER_TRANSPORT_DSN=redis://password@localhost:6379/messages/symfony/consumer?auto_setup=true&serializer=1&stream_max_entries=0&dbindex=0
    # Unix Socket Example
    MESSENGER_TRANSPORT_DSN=redis:///var/run/redis.sock

5.1

The Unix socket DSN was introduced in Symfony 5.1.

To use the Redis transport, you will need the Redis PHP extension (&gt;=4.3) and a running Redis server (^5.0).

A number of options can be configured via the DSN or via the `options` key under the transport in `messenger.yaml`:

<table><thead><tr class="header"><th>Option</th><th>Description</th><th>Default</th></tr></thead><tbody><tr class="odd"><td>stream</td><td>The Redis stream name</td><td>messages</td></tr><tr class="even"><td>group</td><td>The Redis consumer group name</td><td>symfony</td></tr><tr class="odd"><td>consumer</td><td>Consumer name used in Redis</td><td>consumer</td></tr><tr class="even"><td><p>auto_setup auth</p></td><td><p>Create the Redis group automatically? The Redis password</p></td><td><p>true</p></td></tr><tr class="odd"><td><p>serializer</p></td><td><p>How to serialize the final payload in Redis (the <code>Redis::OPT_SERIALIZER</code> option)</p></td><td><p><code>Redis::SERIALIZER_PHP</code></p></td></tr><tr class="even"><td><p>stream_max_entries</p></td><td><p>The maximum number of entries which the stream will be trimmed to. Set it to a large enough number to avoid losing pending messages</p></td><td><p><code>0</code> (which means "no trimming")</p></td></tr><tr class="odd"><td>tls</td><td>Enable TLS support for the connection</td><td>false</td></tr></tbody></table>

### In Memory Transport

`in-memory` トランスポートは実際にメッセージを配送しない。代わりに、リクエスト中にメモリに保持するので、テストに便利です。例えば、`async_priority_normal` トランスポートがある場合、`test` 環境でそれをオーバーライドしてこのトランスポートを使うことができます。

```yaml
    # config/packages/test/messenger.yaml
    framework:
        messenger:
            transports:
                async_priority_normal: 'in-memory://'

```
```xml
    <!-- config/packages/test/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:transport name="async_priority_normal" dsn="in-memory://"/>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/test/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'transports' => [
                'async_priority_normal' => [
                    'dsn' => 'in-memory://',
                ],
            ],
        ],
    ]);
```

そうすれば、テストの間、メッセージは実際のトランスポートに配送されません。さらに良いことに、テストでは、リクエスト中に正確に1つのメッセージが送られたかどうかをチェックできます。

```php
    // tests/DefaultControllerTest.php
    namespace App\Tests;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
    use Symfony\Component\Messenger\Transport\InMemoryTransport;

    class DefaultControllerTest extends WebTestCase
    {
        public function testSomething()
        {
            $client = static::createClient();
            // ...

            $this->assertSame(200, $client->getResponse()->getStatusCode());

            /* @var InMemoryTransport $transport */
            $transport = self::$container->get('messenger.transport.async_priority_normal');
            $this->assertCount(1, $transport->getSent());
        }
    }
```

Note

すべての `in-memory` トランスポートは、`KernelTestCase` や `WebTestCase` を拡張したテストクラスの各テストの後に自動的にリセットされます。

### Serializing Messages

メッセージがトランスポートに送信 (およびトランスポートから受信) されるとき、PHP のネイティブの `serialize()` & `unserialize()` 関数を使用してシリアライズされます。これをグローバルに (または各トランスポートに) 変更できるのは、`Symfony\Component\Messenger\Transport\Serialization\SerializerInterface` を実装したサービスです。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            serializer:
                default_serializer: messenger.transport.symfony_serializer
                symfony_serializer:
                    format: json
                    context: { }

            transports:
                async_priority_normal:
                    dsn: # ...
                    serializer: messenger.transport.symfony_serializer
```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:serializer default-serializer="messenger.transport.symfony_serializer">
                    <framework:symfony-serializer format="json">
                        <framework:context/>
                    </framework:symfony-serializer>
                </framework:serializer>

                <framework:transport name="async_priority_normal" dsn="..." serializer="messenger.transport.symfony_serializer"/>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'serializer' => [
                'default_serializer' => 'messenger.transport.symfony_serializer',
                'symfony_serializer' => [
                    'format' => 'json',
                    'context' => [],
                ],
            ],
            'transports' => [
                'async_priority_normal' => [
                    'dsn' => // ...
                    'serializer' => 'messenger.transport.symfony_serializer',
                ],
            ],
        ],
    ]);
```

`messenger.transport.symfony_serializer` は `Serializer component </serializer>` を使う組み込みのサービスで、いくつかの方法で設定できます。symfony のシリアライザを使うことを *do* 選ぶ場合、`Symfony\\Component\\\Messenger\\Stamp\SerializerStamp` ( [Envelopes & Stamps](#envelopes-stamps) を通じてケースバイケースでコンテキストをコントロールできます。)

Tip

他のアプリケーションとの間でメッセージを送受信する場合、シリアライズ処理をより制御する必要があるかもしれません。カスタムシリアライザーを使うことでこのコントロールを提供します。詳細は [SymfonyCasts のメッセージシリアライザーチュートリアル](https://symfonycasts.com/screencast/messenger/transport-serializer) をご覧ください。

ハンドラーのカスタマイズ
--------------------

### ハンドラを手動で設定する

通常、symfony は自動的に `<messenger-handler>` のハンドラを見つけて登録します。しかし、ハンドラサービスに `messenger.message_handler` というタグをつけることで、ハンドラを手動で設定し、追加の設定を渡すこともできます。

```yaml
    # config/services.yaml
    services:
        App\MessageHandler\SmsNotificationHandler:
            tags: [messenger.message_handler]

            # またはオプションを付けて
            tags:
                -
                    name: messenger.message_handler
                    # タイプヒントで推測できない場合にだけ必要
                    handles: App\Message\SmsNotification
```
```xml
    <!-- config/services.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd">

        <services>
            <service id="App\MessageHandler\SmsNotificationHandler">
                 <!-- handles is only needed if it can't be guessed by type-hint -->
                 <tag name="messenger.message_handler"
                      handles="App\Message\SmsNotification"/>
            </service>
        </services>
    </container>
```
```php
    // config/services.php
    use App\Message\SmsNotification;
    use App\MessageHandler\SmsNotificationHandler;

    $container->register(SmsNotificationHandler::class)
        ->addTag('messenger.message_handler', [
            // only needed if can't be guessed by type-hint
            'handles' => SmsNotification::class,
        ]);
```

タグで設定できるオプションは以下の通りです。

-   `bus`
-   `from_transport`
-   `handles`
-   `method`
-   `priority`

### ハンドラー サブスクライバとオプション

ハンドラクラスは複数のメッセージを扱ったり、`MessageSubscriberInterface` を実装して自身を設定したりすることができる。

```php
    // src/MessageHandler/SmsNotificationHandler.php
    namespace App\MessageHandler;

    use App\Message\OtherSmsNotification;
    use App\Message\SmsNotification;
    use Symfony\Component\Messenger\Handler\MessageSubscriberInterface;

    class SmsNotificationHandler implements MessageSubscriberInterface
    {
        public function __invoke(SmsNotification $message)
        {
            // ...
        }

        public function handleOtherSmsNotification(OtherSmsNotification $message)
        {
            // ...
        }

        public static function getHandledMessages(): iterable
        {
            // handle this message on __invoke
            yield SmsNotification::class;

            // also handle this message on handleOtherSmsNotification
            yield OtherSmsNotification::class => [
                'method' => 'handleOtherSmsNotification',
                //'priority' => 0,
                //'bus' => 'messenger.bus.default',
            ];
        }
    }
```

### ハンドラーを異なるトランスポートにバインドする

各メッセージは複数のハンドラを持つことができ、メッセージが消費されたときにすべてのハンドラが呼び出されます。しかし、特定のトランスポートから受信したときにのみ呼び出されるように ハンドラを設定することもできます。これにより、各ハンドラが、異なるトランスポートを消費する異なる「ワーカー」によって 呼び出される単一のメッセージを持つことができる。

2つのハンドラを持つ `UploadedImage` メッセージがあるとします。

- `ThumbnailUploadedImageHandler`: これを `image_transport` というトランスポートで処理したい場合に使用します。
- `NotifyAboutNewUploadedImageHandler`: これを `async_priority_normal` というトランスポートで処理したい場合に使用します。

これを行うには、各ハンドラに `from_transport` オプションを追加します。例えば、以下のようになります。

```php
    // src/MessageHandler/ThumbnailUploadedImageHandler.php
    namespace App\MessageHandler;

    use App\Message\UploadedImage;
    use Symfony\Component\Messenger\Handler\MessageSubscriberInterface;

    class ThumbnailUploadedImageHandler implements MessageSubscriberInterface
    {
        public function __invoke(UploadedImage $uploadedImage)
        {
            // do some thumbnailing
        }

        public static function getHandledMessages(): iterable
        {
            yield UploadedImage::class => [
                'from_transport' => 'image_transport',
            ];
        }
    }
```

そして同様に、

```php
    // src/MessageHandler/NotifyAboutNewUploadedImageHandler.php
    // ...

    class NotifyAboutNewUploadedImageHandler implements MessageSubscriberInterface
    {
        // ...

        public static function getHandledMessages(): iterable
        {
            yield UploadedImage::class => [
                'from_transport' => 'async_priority_normal',
            ];
        }
    }
```

それから、あなたのメッセージを*両方の*トランスポートに「ルート」することを確認してください。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            transports:
                async_priority_normal: # ...
                image_transport: # ...

            routing:
                # ...
                'App\Message\UploadedImage': [image_transport, async_priority_normal]
```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:transport name="async_priority_normal" dsn="..."/>
                <framework:transport name="image_transport" dsn="..."/>

                <framework:routing message-class="App\Message\UploadedImage">
                    <framework:sender service="image_transport"/>
                    <framework:sender service="async_priority_normal"/>
                </framework:routing>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'transports' => [
                'async_priority_normal' => '...',
                'image_transport' => '...',
            ],
            'routing' => [
                'App\Message\UploadedImage' => ['image_transport', 'async_priority_normal']
            ]
        ],
    ]);
```

それだ！これで各トランスポートを消費できるようになりました。

```php
    # はメッセージを処理するときにのみ ThumbnailUploadedImageHandler を呼び出します。
    $ php bin/console messenger:consume image_transport -vv

    $ php bin/console messenger:consume async_priority_normal -vv
```

Caution

ハンドラが `from_transport` 設定を持っていない場合は、メッセージを受信したすべてのトランスポートに対して実行されます。

メッセンジャーを拡張する
-------------------

### エンベロープ & スタンプ

メッセージは、任意の PHP オブジェクトにすることができます。AMQP 内での処理方法や、メッセージが処理されるまでの遅延時間を追加するなど、 メッセージについて何か特別な設定をしなければならないことがあるかもしれません。このような場合は、メッセージに "スタンプ" を追加することで設定することができます。

    use Symfony\Component\Messenger\Envelope;
    use Symfony\Component\Messenger\MessageBusInterface;
    use Symfony\Component\Messenger\Stamp\DelayStamp;

    public function index(MessageBusInterface $bus)
    {
        $bus->dispatch(new SmsNotification('...'), [
            // wait 5 seconds before processing
            new DelayStamp(5000)
        ]);

        // or explicitly create an Envelope
        $bus->dispatch(new Envelope(new SmsNotification('...'), [
            new DelayStamp(5000)
        ]));

        // ...
    }

内部的には、各メッセージは `Envelope` に包まれており、メッセージとスタンプを保持しています。これは手動で作成することもできますし、メッセージバスで作成することもできます。スタンプは目的に応じて様々な種類があり、内部的にはメッセージに関する情報を追跡するために使用されます - メッセージを処理しているメッセージバスや、失敗した後に再試行されているかどうかなど。

### ミドルウェア

メッセージバスにメッセージをディスパッチしたときに何が起こるかは、そのミドルウェアのコレクションとその順序に依存します。デフォルトでは、各バスに設定されたミドルウェアは次のようになります。

1.  `add_bus_name_stamp_middleware` - メッセージがどのバスにディスパッチされたかを記録するためのスタンプを追加します。
2.  `dispatch_after_current_bus`- `/messenger/message-recorder` を参照してください。
3.  `failed_message_processing_middleware` - `failed_message_processing_middleware` - `failure transport <messenger-failure-transport>` 経由で再試行中のメッセージを処理し、元のトランスポートから受信したかのように正しく機能するようにします。
4.  あなた自身の [ミドルウェア](#middleware) のコレクション。
5.  `send_message` - ルーティングがトランスポートに設定されている場合、そのトランスポートにメッセージを送り、ミドルウェアチェーンを停止します。
6.  `handle_message` - 与えられたメッセージのメッセージハンドラを呼び出す。

Note

これらのミドルウェア名は実際にはショートカット名です。実際のサービス ID の前には `messenger.middleware.` が付けられます (例: `messenger.middleware.handle_message`)。

ミドルウェアはメッセージがディスパッチされたときに実行されますが、ワーカーを介してメッセージを受信したときにも実行されます (トランスポートに送信されたメッセージを非同期に処理するために)。独自のミドルウェアを作成する場合は、この点に注意してください。

このリストに独自のミドルウェアを追加することもできますし、デフォルトのミドルウェアを完全に無効にして、独自のミドルウェアだけを含めることもできます。

```yaml
    # config/packages/messenger.yaml
    framework:
        messenger:
            buses:
                messenger.bus.default:
                    middleware:
                        # Symfony\Component\Messenger\Middleware\MiddlewareInterface を実装したサービスID
                        - 'App\Middleware\MyMiddleware'
                        - 'App\Middleware\AnotherMiddleware'

                    #default_middleware: false
```
```xml
    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:middleware id="App\Middleware\MyMiddleware"/>
                <framework:middleware id="App\Middleware\AnotherMiddleware"/>
                <framework:bus name="messenger.bus.default" default-middleware="false"/>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'buses' => [
                'messenger.bus.default' => [
                    'middleware' => [
                        'App\Middleware\MyMiddleware',
                        'App\Middleware\AnotherMiddleware',
                    ],
                    'default_middleware' => false,
                ],
            ],
        ],
    ]);
```

Note

ミドルウェアサービスが抽象化されている場合、バスごとに異なるインスタンスが作成されます。

### Doctrine 用のミドルウェア

1.11

The following Doctrine middleware were introduced in DoctrineBundle 1.11.

If you use Doctrine in your app, a number of optional middleware exist that you may want to use:

    # config/packages/messenger.yaml
    framework:
        messenger:
            buses:
                command_bus:
                    middleware:
                        # each time a message is handled, the Doctrine connection
                        # is "pinged" and reconnected if it's closed. Useful
                        # if your workers run for a long time and the database
                        # connection is sometimes lost
                        - doctrine_ping_connection

                        # After handling, the Doctrine connection is closed,
                        # which can free up database connections in a worker,
                        # instead of keeping them open forever
                        - doctrine_close_connection

                        # wraps all handlers in a single Doctrine transaction
                        # handlers do not need to call flush() and an error
                        # in any handler will cause a rollback
                        - doctrine_transaction

                        # or pass a different entity manager to any
                        #- doctrine_transaction: ['custom']

    <!-- config/packages/messenger.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

        <framework:config>
            <framework:messenger>
                <framework:bus name="command_bus">
                    <framework:middleware id="doctrine_transaction"/>
                    <framework:middleware id="doctrine_ping_connection"/>
                    <framework:middleware id="doctrine_close_connection"/>

                    <!-- or pass a different entity manager to any -->
                    <!--
                    <framework:middleware id="doctrine_transaction">
                        <framework:argument>custom</framework:argument>
                    </framework:middleware>
                    -->
                </framework:bus>
            </framework:messenger>
        </framework:config>
    </container>

    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            'buses' => [
                'command_bus' => [
                    'middleware' => [
                        'doctrine_transaction',
                        'doctrine_ping_connection',
                        'doctrine_close_connection',
                        // Using another entity manager
                        ['id' => 'doctrine_transaction', 'arguments' => ['custom']],
                    ],
                ],
            ],
        ],
    ]);

### メッセンジャーのイベント

ミドルウェアに加えて、Messenger はいくつかのイベントをディスパッチします。`イベントリスナー</event_dispatcher>`を作成して、プロセスの様々な部分にフックすることができます。それぞれに対して、イベントクラスはイベント名です。

-   `Symfony\Component\Messenger\Event\WorkerStartedEvent`
-   `Symfony\Component\Messenger\Event\WorkerMessageReceivedEvent`
-   `Symfony\Component\Messenger\Event\SendMessageToTransportsEvent`
-   `Symfony\Component\Messenger\Event\WorkerMessageFailedEvent`
-   `Symfony\Component\Messenger\Event\WorkerMessageHandledEvent`
-   `Symfony\Component\Messenger\Event\WorkerRunningEvent`
-   `Symfony\Component\Messenger\Event\WorkerStoppedEvent`

複数のバス、コマンド＆イベントバス
-------------------------------------

Messengerは、デフォルトで単一のメッセージバスサービスを提供しています。しかし、「コマンド」、「クエリ」、「イベント」バスを作成し、それらのミドルウェアを制御することで、必要なだけ多くのバスを設定することができます。詳しくは `/messenger/multiple_buses` を参照してください。
