DependencyInjection コンポーネント
=================

> [オリジナル](https://symfony.com/doc/current/components/dependency_injection.html)

> DependencyInjection コンポーネントは、[PSR-11 互換のサービス](https://www.php-fig.org/psr/psr-11/) コンテナを実装しており、アプリケーション内でオブジェクトが構築される方法を標準化して一元化することができます。

依存性インジェクションとサービスコンテナについては、[サービスコンテナ](service_container.md)を参照してください。

<span id="installation"></span>
インストール
----------

```bash
composer require symfony/dependency-injection
```

Note
: このコンポーネントを symfony アプリケーションの外部にインストールする場合、Composer によって提供されるクラスのオートロードメカニズムを有効にするために、コード内に `vendor/autoload.php` ファイルを requre しなければなりません。詳細は[こちらの記事](https://symfony.com/doc/current/components/using_components.html)をお読みください。

<span id="basic-usage"></span>
基本的な使い方
----------

See Also
: この記事では、任意の PHP アプリケーションで独立したコンポーネントとして DependencyInjection 機能を使う方法を説明します。[サービスコンテナ](service_container.md)の記事を読んで、Symfony アプリケーションでの使い方を学びましょう。

以下の 'Mailer' のようなクラスがあって、それをサービスとして利用できるようにしたい場合があるでしょう。

```php
class Mailer
{
    private $transport;

    public function __construct()
    {
        $this->transport = 'sendmail';
    }

    // ...
}
```

これをコンテナにサービスとして登録することができます。

```php
use Symfony\Component\DependencyInjection\ContainerBuilder;

$containerBuilder = new ContainerBuilder();
$containerBuilder->register('mailer', 'Mailer');
```

クラスをより柔軟にするための改良点は、コンテナが使用するトランスポートを設定できるようにすることです。クラスを変更した場合、コンストラクタにこれが渡されます。

```php
class Mailer
{
    private $transport;

    public function __construct($transport)
    {
        $this->transport = $transport;
    }

    // ...
}
```

そして、コンテナ内の transport の選択を設定します。

```php
use Symfony\Component\DependencyInjection\ContainerBuilder;

$containerBuilder = new ContainerBuilder();
$containerBuilder
    ->register('mailer', 'Mailer')
    ->addArgument('sendmail');
```

トランスポートの選択を実装からコンテナに分離したことで、このクラスはより柔軟になりました。

どのメールトランスポートを選択したかは、他のサービスが知る必要があるかもしれません。コンテナ内のパラメータにして、`Mailer` サービスのコンストラクタの引数にこのパラメータを参照することで、複数の場所でそれを変更する必要を避けることができます。

```php
use Symfony\Component\DependencyInjection\ContainerBuilder;

$containerBuilder = new ContainerBuilder();
$containerBuilder->setParameter('mailer.transport', 'sendmail');
$containerBuilder
    ->register('mailer', 'Mailer')
    ->addArgument('%mailer.transport%');
```

`mailer` サービスがコンテナ内にあるので、他のクラスの依存関係として注入することができます。このような `NewsletterManager` クラスを持っている場合。

```php
class NewsletterManager
{
    private $mailer;

    public function __construct(\Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    // ...
}
```

`newsletter_manager` サービスを定義する際、`mailer` サービスはまだ存在しません。`Reference`クラスを使用して、コンテナがニュースレターマネージャを初期化する際に、`mailer` サービスを注入するようにコンテナに指示します。

```php
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Reference;

$containerBuilder = new ContainerBuilder();

$containerBuilder->setParameter('mailer.transport', 'sendmail');
$containerBuilder
    ->register('mailer', 'Mailer')
    ->addArgument('%mailer.transport%');

$containerBuilder
    ->register('newsletter_manager', 'NewsletterManager')
    ->addArgument(new Reference('mailer'));
```

`NewsletterManager` が `Mailer` を必要とせず、それを注入することがオプションである場合は、代わりにセッター注入を使用することができます。

```php
class NewsletterManager
{
    private $mailer;

    public function setMailer(\Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    // ...
}
```

これで `NewsletterManager` に `Mailer` を注入しないことを選択できるようになりました。もしそうしたい場合は、コンテナは `setter` メソッドを呼び出すことができます。


```php
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Reference;

$containerBuilder = new ContainerBuilder();

$containerBuilder->setParameter('mailer.transport', 'sendmail');
$containerBuilder
    ->register('mailer', 'Mailer')
    ->addArgument('%mailer.transport%');

$containerBuilder
    ->register('newsletter_manager', 'NewsletterManager')
    ->addMethodCall('setMailer', [new Reference('mailer')]);
```

このようにして、コンテナから `newsletter_manager` サービスを取得することができます。

```php
use Symfony\Component\DependencyInjection\ContainerBuilder;

$containerBuilder = new ContainerBuilder();

// ...

$newsletterManager = $containerBuilder->get('newsletter_manager');
```

<span id="avoiding-your-code-becoming-dependent-on-the-container"></span>
コードがコンテナに依存してしまうのを防ぐ
----------

コンテナから直接サービスを取得することもできますが、これは最小限にとどめておくのがベストです。例えば、`NewsletterManager` では、コンテナから `mailer` サービスを取得するのではなく、コンテナに `mailer` サービスを注入しています。コンテナに注入してそこから `mailer` サービスを取得することもできましたが、それではこの特定のコンテナに縛られてしまい、他の場所でクラスを再利用することが困難になってしまいます。

どこかの時点でコンテナからサービスを取得する必要がありますが、これはアプリケーションのエントリーポイントで可能な限り少ない回数でなければなりません。

<span id="setting-up-the-container-with-configuration-files"></span>
設定ファイルでコンテナを設定する
----------

上記のようにPHPを使ってサービスを設定するだけでなく、設定ファイルを使うこともできます。これにより、上記の例のように PHP を使用してサービスを定義するのではなく、 XML や YAML を使用してサービスの定義を記述することができます。小規模なアプリケーション以外では、サービスの定義を一つ以上の設定ファイルに移動させて整理するのが理にかなっています。そのためには Config コンポーネントをインストールする必要があります。

XML コンフィグファイルの読み込み

```php
use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Loader\XmlFileLoader;

$containerBuilder = new ContainerBuilder();
$loader = new XmlFileLoader($containerBuilder, new FileLocator(__DIR__));
$loader->load('services.xml');
```

YAML コンフィグファイルの読み込み

```php
use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Loader\YamlFileLoader;

$containerBuilder = new ContainerBuilder();
$loader = new YamlFileLoader($containerBuilder, new FileLocator(__DIR__));
$loader->load('services.yaml');
```

Note
: YAML設定ファイルをロードしたい場合は、[Yamlコンポーネント](https://symfony.com/doc/current/components/yaml.html)をインストールする必要があります。

Tip
: アプリケーションが型にはまらないファイル拡張子を使用している場合 (例えば、XML ファイルが `.config` 拡張子を持っているなど)、 `load()` メソッドの 2 番目のオプションパラメータとしてファイルタイプを渡すことができます。
    ```php
    // ...
    $loader->load('services.config', 'xml');
    ```

PHP を使用してサービスを作成したい場合は、これを別の設定ファイルに移動し、同様の方法でロードすることができます。

```php
use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Loader\PhpFileLoader;

$containerBuilder = new ContainerBuilder();
$loader = new PhpFileLoader($containerBuilder, new FileLocator(__DIR__));
$loader->load('services.php');
```

これで、設定ファイルを使って `newsletter_manager` と `mailer` サービスの設定ができるようになりました。

```yaml
parameters:
    # ...
    mailer.transport: sendmail

services:
    mailer:
        class:     Mailer
        arguments: ['%mailer.transport%']
    newsletter_manager:
        class:     NewsletterManager
        calls:
            - [setMailer, ['@mailer']]
```