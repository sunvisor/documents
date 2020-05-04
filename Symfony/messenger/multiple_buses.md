複数のバス
==============

アプリケーションを構築する際の一般的なアーキテクチャは、コマンドとクエリを分離することです。コマンドは何かを行うアクションであり、クエリはデータを取得します。これを CQRS (Command Query Responsibility Segregation) と呼びます。詳しくは Martin Fowler の [CQRS に関する記事](https://martinfowler.com/bliki/CQRS.html) を参照してください。このアーキテクチャは、複数のバスを定義することでMessengerコンポーネントと併用することができます。

コマンドバスはクエリバスとは少し異なります。例えば、コマンドバスは通常、結果を提供しませんし、クエリバスは非同期であることがほとんどありません。ミドルウェアを使用することで、これらのバスとそのルールを設定することができます。

また、イベントバスを導入することで、アクションとリアクションを分離するのも良いアイデアかもしれません。イベントバスは0以上のサブスクライバーを持つことができます。

```yaml
    framework:
        messenger:
            # The bus that is going to be injected when injecting MessageBusInterface
            default_bus: command.bus
            buses:
                command.bus:
                    middleware:
                        - validation
                        - doctrine_transaction
                query.bus:
                    middleware:
                        - validation
                event.bus:
                    default_middleware: allow_no_handlers
                    middleware:
                        - validation
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
            <!-- The bus that is going to be injected when injecting MessageBusInterface -->
            <framework:messenger default-bus="command.bus">
                <framework:bus name="command.bus">
                    <framework:middleware id="validation"/>
                    <framework:middleware id="doctrine_transaction"/>
                <framework:bus>
                <framework:bus name="query.bus">
                    <framework:middleware id="validation"/>
                <framework:bus>
                <framework:bus name="event.bus" default-middleware="allow_no_handlers">
                    <framework:middleware id="validation"/>
                <framework:bus>
            </framework:messenger>
        </framework:config>
    </container>
```
```php
    // config/packages/messenger.php
    $container->loadFromExtension('framework', [
        'messenger' => [
            // The bus that is going to be injected when injecting MessageBusInterface
            'default_bus' => 'command.bus',
            'buses' => [
                'command.bus' => [
                    'middleware' => [
                        'validation',
                        'doctrine_transaction',
                    ],
                ],
                'query.bus' => [
                    'middleware' => [
                        'validation',
                    ],
                ],
                'event.bus' => [
                    'default_middleware' => 'allow_no_handlers',
                    'middleware' => [
                        'validation',
                    ],
                ],
            ],
        ],
    ]);
```

これにより、3つの新サービスが誕生します。

-   `command.bus`: `Symfony\Component\Messenger\MessageBusInterface` タイプヒントでオートワイヤリングできる (これが `default_bus` なので);
-   `query.bus`: `MessageBusInterface $queryBus` でオートワイヤリングできる
-   `event.bus`: `MessageBusInterface $eventBus` でオートワイヤリングできる

バスごとにハンドラを制限する
-------------------------

デフォルトでは、各ハンドラはすべてのバスのメッセージを扱うことができます。間違ったバスにメッセージをディスパッチしてもエラーにならないようにするために、`messenger.message_handler` タグを使って各ハンドラを特定のバスに限定することができます。

```yaml
    # config/services.yaml
    services:
        App\MessageHandler\SomeCommandHandler:
            tags: [{ name: messenger.message_handler, bus: command.bus }]
            # prevent handlers from being registered twice (or you can remove
            # the MessageHandlerInterface that autoconfigure uses to find handlers)
            autoconfigure: false
```
```xml
    <!-- config/services.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd">

        <services>
            <service id="App\MessageHandler\SomeCommandHandler">
                <tag name="messenger.message_handler" bus="command.bus"/>
            </service>
        </services>
    </container>
```
```php
    // config/services.php
    $container->services()
        ->set(App\MessageHandler\SomeCommandHandler::class)
        ->tag('messenger.message_handler', ['bus' => 'command.bus']);
```

この方法では、`App\MessageHandler\SomeCommandHandler` ハンドラは、`command.bus` バスによってのみ知られています。

また、命名規則に従い、ハンドラサービスをすべて正しいタグで登録することで、このタグをいくつかのクラスに自動的に追加することもできます。

```yaml
    # config/services.yaml

    # put this after the "App\" line that registers all your services
    command_handlers:
        namespace: App\MessageHandler\
        resource: '%kernel.project_dir%/src/MessageHandler/*CommandHandler.php'
        autoconfigure: false
        tags:
            - { name: messenger.message_handler, bus: command.bus }

    query_handlers:
        namespace: App\MessageHandler\
        resource: '%kernel.project_dir%/src/MessageHandler/*QueryHandler.php'
        autoconfigure: false
        tags:
            - { name: messenger.message_handler, bus: query.bus }
```
```xml
    <!-- config/services.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            https://symfony.com/schema/dic/services/services-1.0.xsd">

        <services>
            <!-- command handlers -->
            <prototype namespace="App\MessageHandler\" resource="%kernel.project_dir%/src/MessageHandler/*CommandHandler.php" autoconfigure="false">
                <tag name="messenger.message_handler" bus="command.bus"/>
            </prototype>
            <!-- query handlers -->
            <prototype namespace="App\MessageHandler\" resource="%kernel.project_dir%/src/MessageHandler/*QueryHandler.php" autoconfigure="false">
                <tag name="messenger.message_handler" bus="query.bus"/>
            </prototype>
        </services>
    </container>
```
```php
    // config/services.php

    // Command handlers
    $container->services()
        ->load('App\MessageHandler\', '%kernel.project_dir%/src/MessageHandler/*CommandHandler.php')
        ->autoconfigure(false)
        ->tag('messenger.message_handler', ['bus' => 'command.bus']);

    // Query handlers
    $container->services()
        ->load('App\MessageHandler\', '%kernel.project_dir%/src/MessageHandler/*QueryHandler.php')
        ->autoconfigure(false)
        ->tag('messenger.message_handler', ['bus' => 'query.bus']);
```

バスのデバッグ
-------------------

コマンド `debug:messenger` はバスごとに利用可能なメッセージとハンドラをリストアップします。引数にバス名を指定することで、リストを特定のバスに限定することもできます。


    $ php bin/console debug:messenger

      Messenger
      =========

      command.bus
      -----------

       The following messages can be dispatched:

       ---------------------------------------------------------------------------------------
        App\Message\DummyCommand
            handled by App\MessageHandler\DummyCommandHandler
        App\Message\MultipleBusesMessage
            handled by App\MessageHandler\MultipleBusesMessageHandler
       ---------------------------------------------------------------------------------------

      query.bus
      ---------

       The following messages can be dispatched:

       ---------------------------------------------------------------------------------------
        App\Message\DummyQuery
            handled by App\MessageHandler\DummyQueryHandler
        App\Message\MultipleBusesMessage
            handled by App\MessageHandler\MultipleBusesMessageHandler
       ---------------------------------------------------------------------------------------
