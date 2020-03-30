サービスのエイリアスを作成し、サービスをプライベートとしてマークする方法
=====================

> [オリジナル](https://symfony.com/doc/current/service_container/alias_private.html)

<span id="marking-services-as-public-private"></span>
サービスをパブリック／プライベートとしてマークする
-----------------------------------------

サービスを定義する際には、パブリックまたはプライベートにすることができます。サービスが public の場合、実行時にコンテナから直接アクセスできることを意味します。たとえば、doctrine サービスはパブリックサービスです。

```php
// パブリックサービスであればこうして利用できる
$doctrine = $container->get('doctrine');
```

しかし、典型的には、サービスは[依存性注入](service_container.md#services-constructor-injection)を使用してアクセスされます。この場合は、サービスを公開する必要はありません。

ですから、コンテナから `$container->get()` を使用して直接サービスにアクセスする*必要*がない限りは、 サービスを非公開にするのが最善の方法となります。実際、デフォルトではすべてのサービスが[非公開](service_container.md#container-public)になっています。

サービスごとに public オプションを設定することもできます。

```yaml
# config/services.yaml
services:
    # ...

    App\Service\Foo:
        public: true
```

プライベートサービスが特別なのは、コンテナがインスタンス化するかどうかとその方法を最適化できるからです。これにより、コンテナのパフォーマンスが向上します。存在しないサービスを参照しようとした場合、問題のあるコードがそのページで実行されていなくても、どのページを更新したときにも明確なエラーが表示されます。

サービスがプライベートになったので、コンテナから直接*サービスを取得してはいけません*。

```php
use App\Service\Foo;

$container->get(Foo::class);
```

簡単に言うと コードから直接アクセスしたくない場合は、サービスをプライベートとしてマークすることができます。

しかし、サービスがプライベートとしてマークされている場合でも、エイリアスを使ってそのサービスにアクセスすることができます (以下を参照)。

<span id="#aliasing"></span>
エイリアス
--------------------------------

```yaml
# config/services.yaml
services:
    # ...
    App\Mail\PhpMailer:
        public: false

    app.mailer:
        alias: App\Mail\PhpMailer
        public: true
```

つまり、コンテナを直接利用する場合は、次のように`app.mailer`サービスを get することで`PhpMailer`サービスにアクセスすることができます。

```php
$container->get('app.mailer'); // PhpMailer のインスタンスが返される
```

Tip
: YAMLでは、ショートカットを使ってサービスをエイリアス化することもできます。

    ```yaml
    # config/services.yaml
    services:
        # ...
        app.mailer: '@App\Mail\PhpMailer'
    ```


<span id="deprecating-service-aliases"></span>
サービスのエイリアスを非推奨にする
----------------------------------------------------------------------

サービスエイリアスの使用を非推奨にすることを決めた場合 (サービスエイリアスが古くなっているか、もう保守しないことにしたからです)、その定義を非推奨にすることができます。

```yaml
app.mailer:
    alias: '@App\Mail\PhpMailer'

    # これは一般的な非推奨のメッセージを表示します...
    deprecated: true

    # ...しかし、カスタムの非推奨メッセージを定義することもできます。
    deprecated: 'The "%alias_id%" alias is deprecated. Do not use it anymore.'
```

現在、このサービス・エイリアスが使用されるたびに、非推奨の警告がトリガされ、エイリアスの使用を停止または変更するように通知されます。

このメッセージは実際にはメッセージ・テンプレートで、%alias_id% プレースホルダの出現をサービス・エイリアス ID で置き換えます。テンプレート内に %alias_id% プレースホルダが少なくとも 1 つ存在する必要があります。

<span id="anonymous-services"></span>
匿名サービス
---------------------------------------

場合によっては、あるサービスが他のサービスの依存関係として使用されるのを防ぎたい場合があります。これは匿名のサービスを作成することで実現できます。これらのサービスは通常のサービスのようなものですが、ID は定義されておらず、使用される場所で作成されます。

次の例は、匿名サービスを他のサービスに注入する方法を示しています。

```yaml
# config/services.yaml
services:
    App\Foo:
        arguments:
            - !service
                class: App\AnonymousBar
```

Info
: 匿名サービスは設定で定義されたデフォルトの定義を継承しません。そのため、匿名サービスを実行する際には、明示的に`autowired`または`autoconfigure`としてサービスをマークする必要があります。
例: `inline(Foo::class)->autowire()->autoconfigure()` 

<span id="deprecating-services"></span>
サービスを非推奨にする
-----------------------------------------------------

サービスの使用を非推奨にすることを決めたら (サービスが古くなったからとか、もう保守しないことにしたからとか)、その定義を非推奨にすることができます。

```yaml
# config/services.yaml
App\Service\OldService:
    deprecated: The "%service_id%" service is deprecated since vendor-name/package-name 2.8 and will be removed in 3.0.
```

現在、このサービスが使用されるたびに、そのサービスを停止するか、そのサービスの使用を変更するように通知する非推奨警告がトリガされます。

このメッセージは実際にはメッセージ・テンプレートで、`%service_id%` プレースホルダの出現をサービスの ID で置き換えます。テンプレート内に `%service_id%` プレースホルダが 1 つ以上存在する必要があります。

Info
: 非推奨のメッセージはオプションです。設定されていない場合、Symfony は次のデフォルトのメッセージを表示します。
`The "%service_id%" service is deprecated. You should stop using it, as it will soon be removed.`

Tip
: デフォルトのメッセージは汎用的すぎるので、カスタムメッセージを定義することを強くお勧めします。良いメッセージは、このサービスがいつ廃止されたのか、いつまで保守されるのか、そして使用する代替サービスがある場合はそれを知らせます。

サービスデコレータ（[サービスのデコレーション方法](https://symfony.com/doc/current/service_container/service_decoration.html)を参照）の場合、定義が非推奨のステータスを修正しない場合、デコレーションされた定義からステータスを継承します。