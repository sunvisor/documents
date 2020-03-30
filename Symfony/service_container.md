Service Container
=================

> [オリジナル](https://symfony.com/doc/current/service_container.html)

アプリケーションには便利なオブジェクトがたくさんあります。「メーラー」オブジェクトはメールの送信を助け、別のオブジェクトはデータベースへの保存を助けます。アプリが「行う」ほぼすべてのことは、実際にはこれらのオブジェクトの1つによって行われます。そして、新しいバンドルをインストールするたびに、さらに多くのオブジェクトにアクセスできます!

Symfony において、これらの便利なオブジェクトはサービスと呼ばれ、それぞれのサービスはサービスコンテナと呼ばれる非常に特別なオブジェクトのなかに住みます。コンテナはオブジェクトが構築される方法を一元化することを可能にします。これはあなたの生活をより簡単にし、強力なアーキテクチャを促進し、超高速です!

<span id="fetching-and-using-services"></span>
サービスの取得と利用
-----------------

Symfony アプリを起動した瞬間、コンテナにはすでに多くのサービスが含まれています。これらはツールのようなものです: それらを利用するためにあなたを待っています。コントローラーのなかで、サービスのクラスもしくはインターフェイスの名前を引数にタイプヒントを与えることで、コンテナからサービスを「リクエスト」することができます。何かをログに記録したいですか？問題ありません。

```php
// src/Controller/ProductController.php
namespace App\Controller;

use Psr\Log\LoggerInterface;

class ProductController
{
    /**
     * @Route("/products")
     */
    public function list(LoggerInterface $logger)
    {
        $logger->info('Look! I just used a service');

        // ...
    }
}
```

他にはどんなサービスがありますか？走って調べてみましょう。

```bash
php bin/console debug:autowiring

  # this is just a *small* sample of the output...

  Describes a logger instance.
  Psr\Log\LoggerInterface (monolog.logger)

  Request stack that controls the lifecycle of requests.
  Symfony\Component\HttpFoundation\RequestStack (request_stack)

  Interface for the session.
  Symfony\Component\HttpFoundation\Session\SessionInterface (session)

  RouterInterface is the interface that all Router classes must implement.
  Symfony\Component\Routing\RouterInterface (router.default)

  [...]
```

これらのタイプヒントをコントローラーのメソッドもしくは[独自のサービス](#service-container-creating-service)のなかで使うとき、symfony は自動的にそのタイプにマッチするサービスオブジェクトを渡します。

ドキュメント全体を通して、コンテナ内に存在する多くの異なるサービスを使う方法を見ることができます。

Tip
: コンテナには実際にはもっとたくさんのサービスがあり、それぞれのサービスはコンテナ内では`session`や `router.default`のように固有のIDを持っています。完全なリストを見るには、`php bin/console debug:container` を実行してください。しかし、たいていの場合はこのことを気にする必要はないでしょう。[特定のサービスを選択する](#services-wire-specific-service) を参照してください。
[サービスコンテナをデバッグする方法とサービスの一覧](https://symfony.com/doc/current/service_container/debug.html)を参照してください。

<span id="service-container-creating-service"></span>
コンテナでのサービスの作成/設定
--------------------------

また、独自のコードをサービスに整理することもできます。例えば、ユーザーにランダムでハッピーなメッセージを表示する必要があるとします。このコードをコントローラに入れてしまうと、再利用することができません。その代わりに、新しいクラスを作成することにします。

```php
// src/Service/MessageGenerator.php
namespace App\Service;

class MessageGenerator
{
    public function getHappyMessage()
    {
        $messages = [
            'You did it! You updated the system! Amazing!',
            'That was one of the coolest updates I\'ve seen all day!',
            'Great work! Keep going!',
        ];

        $index = array_rand($messages);

        return $messages[$index];
    }
}
```

おめでとうございます! 最初のサービスクラスができました! コントローラの中ですぐに使うことができます。

```php
use App\Service\MessageGenerator;

public function new(MessageGenerator $messageGenerator)
{
    // タイプヒントのおかげでコンテナーは MessageGenerator の
    // インスタンスを作って渡してくれます
    // ...

    $message = $messageGenerator->getHappyMessage();
    $this->addFlash('success', $message);
    // ...
}
```

`MessageGenerator` サービスを要求すると、コンテナは新しい `MessageGenerator` オブジェクトを構築してそれを返します (下記のサイドバーを参照してください)。しかし、サービスを要求しない場合は、サービスは構築されません。ボーナスとして、`MessageGenerator` サービスは一度しか作成されません。

### services.yaml での自動サービス読み込み

このドキュメントでは、以下のサービス設定を使用していることを前提としています。

```yaml
# config/services.yaml
services:
    # このファイル内のサービスのデフォルト設定
    _defaults:
        autowire: true      # サービスに依存関係を自動的に注入します。
        autoconfigure: true # コマンドやイベントサブスクライバーなどのサービスを自動的に登録します。

    # src/ にあるクラスをサービスとして利用できるようにします。
    # これはクラスごとに作成され、id は完全修飾されたクラス名になります
    App\:
        resource: '../src/*'
        exclude: '../src/{DependencyInjection,Entity,Migrations,Tests,Kernel.php}'

    # ...
```

Tip
: `resource` および `exclude` オプションの値には、任意の有効な[グロブパターン](https://en.wikipedia.org/wiki/Glob_(programming))を指定できます。`exclude` オプションの値は、グロブパターンの配列にすることもできます。

この設定のおかげで、手動で設定しなくても `src/` ディレクトリにある任意のクラスを自動的にサービスとして利用することができます。これについては後ほど、[リソースを使って多くのサービスを一度にインポートする](https://symfony.com/doc/current/service_container.html#service-psr4-loader) を参照してください。

サービスを手動でワイヤリングしたい場合、それは完全に可能です: [明示的にサービスと引数を設定する](#services-explicitly-configure-wire-services) を参照してください。


<span id="injecting-services-config-into-a-service"></span>
サービスへのサービス/設定の注入
--------------------------

`MessageGenerator` 内から 'logger' サービスにアクセスする必要がある場合はどうすればいいですか？問題ありません! `LoggerInterface` タイプヒントを持つ `$logger` 引数を持つ `__construct()` メソッドを作成します。これを新しい `$logger` プロパティに設定し、後で使用します。

```php
// src/Service/MessageGenerator.php
namespace App\Service;

use Psr\Log\LoggerInterface;

class MessageGenerator
{
    private $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function getHappyMessage()
    {
        $this->logger->info('About to find a happy message!');
        // ...
    }
}
```

これで完了です。コンテナは `MessageGenerator` をインスタンス化するときに、`logger` サービスを渡すことを*自動的*に知っています。どうやって知っているのでしょうか？オートワイヤリングです。キーとなるのは、`__construct()` メソッドの `LoggerInterface` タイプヒントと `services.yaml` の `autowire: true` コンフィグです。引数にタイプヒントを与えると、コンテナは自動的に一致するサービスを見つけます。もし見つからなかった場合は、役立つ提案が表示された明確な例外が表示されます。

ところで、このように `__construct()` メソッドに依存性を追加する方法は、*依存性注入*と呼ばれています。単純な概念にしては怖い言葉ですね。

`LoggerInterface`を型ヒントに使うことをどうやって知ったらいいのでしょうか？あなたが使用している機能のドキュメントを読むか、または実行してオートワイアブルなタイプヒントのリストを取得することができます。

```bash
php bin/console debug:autowiring

  # this is just a *small* sample of the output...

  Describes a logger instance.
  Psr\Log\LoggerInterface (monolog.logger)

  Request stack that controls the lifecycle of requests.
  Symfony\Component\HttpFoundation\RequestStack (request_stack)

  Interface for the session.
  Symfony\Component\HttpFoundation\Session\SessionInterface (session)

  RouterInterface is the interface that all Router classes must implement.
  Symfony\Component\Routing\RouterInterface (router.default)

  [...]
```

<sapn id="handling-multiple-services"></span>
複数のサービスの取り扱い
--------------------

また、サイトの更新が行われるたびにサイト管理者にメールを送りたいとします。そのためには、新しいクラスを作成します。

```php
// src/Updates/SiteUpdateManager.php
namespace App\Updates;

use App\Service\MessageGenerator;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;

class SiteUpdateManager
{
    private $messageGenerator;
    private $mailer;

    public function __construct(MessageGenerator $messageGenerator, MailerInterface $mailer)
    {
        $this->messageGenerator = $messageGenerator;
        $this->mailer = $mailer;
    }

    public function notifyOfSiteUpdate()
    {
        $happyMessage = $this->messageGenerator->getHappyMessage();

        $email = (new Email())
            ->from('admin@example.com')
            ->to('manager@example.com')
            ->subject('Site update just happened!')
            ->text('Someone just updated the site. We told them: '.$happyMessage);

        $this->mailer->send($email);

        // ...
    }
}
```

これには `MessageGenerator` と `Mailer` サービスが必要です。それは問題ありません、私たちはそれらのクラス名とインタフェース名をヒントにしてタイプによってそれらを尋ねます これで、この新しいサービスを使用する準備が整いました。例えば、コントローラの中で、新しい `SiteUpdateManager` クラスをタイプヒントして使用することができます。

```php
// src/Controller/SiteController.php

// ...
use App\Updates\SiteUpdateManager;

public function new(SiteUpdateManager $siteUpdateManager)
{
    // ...

    if ($siteUpdateManager->notifyOfSiteUpdate()) {
        $this->addFlash('success', 'Notification mail was sent successfully.');
    }

    // ...
}
```

オートワイヤリングと `__construct()` の型のヒントのおかげで、コンテナは SiteUpdateManager オブジェクトを作成し、それに正しい引数を渡します。ほとんどの場合、これは完璧に動作します。


<sapn id="">manually-wiring-arguments</span>
手動で引数をワイヤリングする
----------------

しかし、サービスへの引数がオートワイヤリングできない場合がいくつかあります。例えば、管理者用のメールを設定可能にしたいとします。

```diff
// src/Updates/SiteUpdateManager.php
// ...

class SiteUpdateManager
{
    // ...
+    private $adminEmail;

-    public function __construct(MessageGenerator $messageGenerator, \Swift_Mailer $mailer)
+    public function __construct(MessageGenerator $messageGenerator, \Swift_Mailer $mailer, $adminEmail)
    {
        // ...
+        $this->adminEmail = $adminEmail;
    }

    public function notifyOfSiteUpdate()
    {
        // ...

        $message = \Swift_Message::newInstance()
            // ...
-            ->setTo('manager@example.com')
+            ->setTo($this->adminEmail)
            // ...
        ;
        // ...
    }
}
```

この変更をして更新するとエラーになります。

> サービス "AppUpdatesSiteUpdateManager" を自動生成できません: メソッド "__construct()" の引数 "$adminEmail" は、型のヒントを持つか、明示的に値を与えなければなりません。

それは理にかなっています! ここに渡したい値をコンテナが知っているわけがありません。問題ありません! 設定では、この引数を明示的に設定することができます。

```yaml
# config/services.yaml
services:
    # ... same as before

    # same as before
    App\:
        resource: '../src/*'
        exclude: '../src/{DependencyInjection,Entity,Migrations,Tests,Kernel.php}'

    # explicitly configure the service
    App\Updates\SiteUpdateManager:
        arguments:
            $adminEmail: 'manager@example.com'
```

このおかげで、コンテナは `SiteUpdateManager` サービスを作成する際に `__construct` の `$adminEmail` 引数に `manager@example.com` を渡します。他の引数はそのままオートワイヤリングされます。

しかし、これは壊れやすいのではないでしょうか？幸いなことに、そんなことはありません。もし `$adminEmail` の引数の名前を他のものに変更した場合、例えば `$mainEmail` のように、次のページをリロードしたときに明確な例外が発生します (そのページがこのサービスを使用していなくても)。

<sapn id="service-parameters"></span>
サービスパラメータ
---------------

サービスオブジェクトを保持することに加えて、コンテナは**パラメータ**と呼ばれる設定も保持します。Symfony のConfigに関する主な記事では[設定パラメータ](https://symfony.com/doc/current/configuration.html#configuration-parameters)について詳しく説明し、すべての型(文字列、ブール値、配列、バイナリ、PHP 定数パラメータ)を示します。

しかしながら、サービスに関連する別のタイプのパラメータがあります。YAMLの設定では、@で始まる文字列は通常の文字列の代わりにサービスのIDとみなされます。XMLの設定では、パラメータに`type="service"` 型を使い、PHPの設定ではReferenceクラスを使います。

```yaml
# config/services.yaml
services:
    App\Service\MessageGenerator:
        # これは文字列ではなく、'logger' と呼ばれるサービスへの参照です。
        arguments: ['@logger']

        # 文字列パラメータの値が '@' で始まる場合は、エスケープする必要があります。
        # '@' を追加することで、symfony がサービスとみなさないようにします。
        # (これは文字列 '@securepassword' としてパースされます)
        mailer_password: '@@securepassword'
```

コンテナのパラメータの操作は、コンテナのアクセサメソッドを使って簡単に行うことができます。

```php
// パラメータが定義されているかどうかをチェックします (パラメータ名は大文字と小文字を区別します)。
$container->hasParameter('mailer.transport');

// パラメータの値を取得します。
$container->getParameter('mailer.transport');

// 新しいパラメータを追加します。
$container->setParameter('mailer.transport', 'sendmail');
```

Caution
: 上記で使われている `.` ノーテーションはパラメータを読みやすくするための [Symfony の規約](https://symfony.com/doc/current/contributing/code/standards.html#service-naming-conventions)です。パラメータはフラットなキーと値の要素で、ネストされた配列に編成することはできません。

Info
: パラメータを設定できるのはコンテナがコンパイルされる前だけで、ランタイムではありません。コンテナのコンパイルについては、[コンテナのコンパイル](https://symfony.com/doc/current/components/dependency_injection/compilation.html)を参照してください。

<sapn id="choose-a-specific-service"></span>
特定のサービスを選択する
--------------------

先に作成した `MessageGenerator` サービスには `LoggerInterface` 引数が必要です。

```php
// src/Service/MessageGenerator.php
namespace App\Service;

use Psr\Log\LoggerInterface;

class MessageGenerator
{
    private $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
    // ...
}
```

しかし、コンテナ内には`logger`、`monolog.logger.request`、`monolog.logger.php`など、`LoggerInterface`を実装したサービスが*複数*存在します。コンテナはどのサービスを使用するかをどのようにして知ることができるのでしょうか？

このような状況では、コンテナは通常、サービスのいずれかを自動的に選択するように設定されています - この場合は `logger` (理由については、[オートワイヤリングを有効にするためのエイリアスの使用](#service-autowiring-alias)を参照)。しかし、これを制御して別のロガーを渡すこともできます。

```yaml
# config/services.yaml
services:
    # ... same code as before

    # サービスを明示的に設定します
    App\Service\MessageGenerator:
        arguments:
            # '@' 記号は重要です。 これはコンテナに、
            # 'monolog.logger.request' という文字列ではなく、
            # ID が 'monolog.logger.request' である service を
            # 渡したいことを伝えるものです。
            $logger: '@monolog.logger.request'
```

これは、コンテナに、`__construct` の `$logger` 引数へは、id が `monolog.logger.request` であるサービスを使用するように指示します。

コンテナ内のすべての可能なサービスの完全なリストは、次を実行してください。

```bash
 php bin/console debug:container
```

<sapn id="binding-arguments-by-name-or-type"></span>
名前またはタイプによるバインド引数
-----------------------------

`bind` キーワードを使用して、特定の引数を名前や型でバインドすることもできます。

```yaml
# config/services.yaml
services:
    _defaults:
        bind:
            # この値を、このファイルで定義されているサービスの 
            # $adminEmail 引数に渡します (コントローラの引数を含む)。
            $adminEmail: 'manager@example.com'

            # このファイルで定義されているサービスの任意の 
            # $requestLogger 引数にこのサービスを渡します。
            $requestLogger: '@monolog.logger.request'

            # このファイルで定義されている任意の LoggerInterface タイプの
            # ヒントに対して、このサービスを渡します。
            Psr\Log\LoggerInterface: '@monolog.logger.request'

            # オプションで、マッチする引数の名前と型の両方を定義することができます。
            string $adminEmail: 'manager@example.com'
            Psr\Log\LoggerInterface $requestLogger: '@monolog.logger.request'
            iterable $rules: !tagged_iterator app.foo.rule

    # ...
```

`defaults` の下に `bind` キーを置くことで、このファイルで定義されているサービスの引数の値を指定することができます。引数を名前 (例: `$adminEmail`)、タイプ (例: `PsrR\Log\LogLoggerInterface`)、または両方 (例: `PsrR\LogLogLoggerInterface $requestLogger`) でバインドすることができます。

`bind` 設定は、特定のサービスに適用したり、多くのサービスを一度に読み込む場合にも適用できます (例: [リソースを使って多くのサービスを一度に読み込む](https://symfony.com/doc/current/service_container.html#service-psr4-loader))。

<sapn id="the-autowire-option"></span>
autowire オプション
-----------------

上記の例では、`services.yaml` ファイルの `_defaults` セクションに `autowire: true` を設定し、そのファイルで定義されているすべてのサービスに適用するようにしています。この設定をすることで、サービスの `__construct()` メソッドにヒントとなる引数を入力することができ、コンテナは自動的に正しい引数を渡してくれます。このエントリ全体では、オートワイヤリングを中心に書いてきました。

オートワイヤリングの詳細については、[サービスの依存関係を自動的に定義する (オートワイヤリング)](autowiring.md)をご覧ください。

<sapn id="the-autoconfigure-option"></span>
autoconfigure オプション
----------------------

上記の例では、`services.yaml` ファイルの `_defaults` セクションに `autoconfigure: true` を設定し、そのファイルで定義されているすべてのサービスに適用するようにしています。この設定をすると、コンテナはサービスのクラスに基づいて自動的に特定の設定をサービスに適用します。これは主にサービスの*自動タグ付け (auto-tag)* に使われます。

例えば、Twigの拡張機能を作成するには、クラスを作成し、それをサービスとして登録し、`twig.extension`でタグ付けする必要があります。

しかし、`autoconfigure: true`を使うと、タグは必要ありません。実際、デフォルトの`services.yaml`設定を使っている場合、何もする必要はありません。そして、`autoconfigure`はあなたのクラスが`Twig\ExtensionExtensionExtensionInterface`を実装しているので、あなたのために`twig.extension`タグを追加します。`autowire`のおかげで、設定なしでコンストラクタの引数を追加することもできます。

<sapn id="linting-service-definitions"></span>
サービス定義の確認
---------------

`lint:container` コマンドは、サービスに注入された引数がその型宣言と一致するかどうかをチェックします。アプリケーションを本番環境にデプロイする前に実行すると便利です (例: 継続的インテグレーションサーバ)。

```bash
php bin/console lint:container
```

コンテナがコンパイルされるたびにすべてのサービス引数の型をチェックすると、パフォーマンスが低下する可能性があります。そのため、この型チェックは `CheckTypeDeclarationsPass` と呼ばれる[*コンパイラパス*](https://symfony.com/doc/current/service_container/compiler_passes.html)で実装されています。パフォーマンスの低下を気にしないのであれば、アプリケーションでコンパイラパスを有効にしてください。

<sapn id="public-versus-private-services"></span>
パブリックサービスとプライベートサービス
---------------------------------

Symfony 4.0 からは、定義されたすべてのサービスはデフォルトでプライベートになります。

これは何を意味するのでしょうか？サービスが公開されているとき、コンテナオブジェクトから直接アクセスできます。これは遅延して(lazily)サービスを取得したい場合に便利です。

```php
namespace App\Generator;

use Psr\Container\ContainerInterface;

class MessageGenerator
{
    private $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function generate(string $message, string $template = null, array $context = []): string
    {
        if ($template && $this->container->has('twig')) {
            // コンテナにはパブリックな "twig" サービスがある
            $twig = $this->container->get('twig');

            return $twig->render($template, $context + ['message' => $message]);
        }

        // template が渡されなかった場合は "twig" サービスはロードされない

        // ...
    }
```

ベストプラクティスとしては、プライベートなサービスのみを作成するべきです。また、`$container->get()` メソッドを使用して公開サービスを取得してはいけません。

しかし、サービスを公開する必要がある場合は、`public` 設定をオーバーライドしてください。

Note
: コンテナを注入するのではなく、代わりに[サービスロケータ](https://symfony.com/doc/current/service_container/service_subscribers_locators.html#service-locators)を使うことを検討すべきです。

<sapn id="importing-many-services-at-once-with-resource"></span>
リソースを使って多くのサービスを一度にインポートする
-------------------------------------------

リソースキーを使うことで多くのサービスを一度にインポートできることはすでに見てきました。たとえば、デフォルトのSymfonyの設定にはこれが含まれています。

```yaml
# config/services.yaml
services:
    # ... 上記と同じ

    # makes classes in src/ available to be used as services
    # this creates a service per class whose id is the fully-qualified class name

    # src/ のクラスをサービスとして利用できるようにします
    # クラスごとにサービスを作成します
    # id は完全修飾されたクラス名です
    App\:
        resource: '../src/*'
        exclude: '../src/{DependencyInjection,Entity,Migrations,Tests,Kernel.php}'
```

Tip
: `resource` および `exclude` オプションの値は、任意の有効な[グロブパターン](https://en.wikipedia.org/wiki/Glob_(programming))にすることができます。

これを使用して、多くのクラスをサービスとして利用可能にし、いくつかのデフォルト設定を適用することができます。各サービスの 'id' は、その完全修飾されたクラス名です。
以下の id (クラス名) を使用することで、インポートされたサービスをオーバーライドすることができます (例: [手動で引数をワイヤリングする]()を参照)。サービスをオーバーライドした場合、オプション (例えば `public`) はインポートから継承されません (ただし、オーバーライドされたサービスは `_defaults` から継承されます)。

また、特定のパスを `exclude` することもできます。除外されたパスは追跡されないので、変更してもコンテナが再構築されることはありません。

Note
: 待って、これは `src/` の*すべての*クラスがサービスとして登録されているということですか？モデルクラスも？実は違います。インポートしたサービスを [private](https://symfony.com/doc/current/service_container.html#container-public) にしておく限り、サービスとして明示的に利用されていない `src/` のすべてのクラスは最終的なコンテナから自動的に削除されます。実際には、インポートは手動で設定しなくてもすべてのクラスが「サービスとして利用できる」ことを意味します。

<sapn id="multiple-service-definitions-using-the-same-namespace"></span>
同じ名前空間を使用する複数のサービス定義
---------------------------------

YAML 設定フォーマットを使ってサービスを定義する場合、PHP の名前空間がそれぞれの設定のキーとして使われるので、同じ名前空間下のクラスに対して異なるサービス設定を定義することはできません。

```yaml
# config/services.yaml
services:
    App\Domain\:
        resource: '../src/Domain/*'
        # ...
```

複数の定義を持つためには、名前空間オプションを追加し、各サービス設定のキーとして任意の一意の文字列を使用します。

```yaml
# config/services.yaml
services:
    command_handlers:
        namespace: App\Domain\
        resource: '../src/Domain/*/CommandHandler'
        tags: [command_handler]

    event_subscribers:
        namespace: App\Domain\
        resource: '../src/Domain/*/EventSubscriber'
        tags: [event_subscriber]
```

<sapn id="explicitly-configuring-services-and-arguments"></span>
明示的にサービスと引数を設定する
--------------------------

Symfony 3.3 以前では、すべてのサービスと(一般的に)引数は明示的に設定されていました。[自動的にサービスをロード](https://symfony.com/doc/current/service_container.html#service-container-services-load-example)することはできず、[オートワイヤリング](https://symfony.com/doc/current/service_container.html#services-autowire)はあまり一般的ではありませんでした。

これらの機能は両方ともオプションです。これらの機能を使うとしても、サービスを手動でワイヤリングしたい場合があるかもしれません。例えば、`SiteUpdateManager`クラスに2つのサービスを登録したいとします。この場合、それぞれが一意のサービス ID を持つ必要があります。

```yaml
# config/services.yaml
services:
    # ...

    # this is the service's id
    site_update_manager.superadmin:
        class: App\Updates\SiteUpdateManager
        # you CAN still use autowiring: we just want to show what it looks like without
        autowire: false
        # manually wire all arguments
        arguments:
            - '@App\Service\MessageGenerator'
            - '@mailer'
            - 'superadmin@example.com'

    site_update_manager.normal_users:
        class: App\Updates\SiteUpdateManager
        autowire: false
        arguments:
            - '@App\Service\MessageGenerator'
            - '@mailer'
            - 'contact@example.com'

    # Create an alias, so that - by default - if you type-hint SiteUpdateManager,
    # the site_update_manager.superadmin will be used
    App\Updates\SiteUpdateManager: '@site_update_manager.superadmin'
```

この場合、`site_update_manager.superadmin`と`site_update_manager.normal_users`の2つのサービスが登録されています。エイリアスのおかげで、`SiteUpdateManager`をタイプヒントとして指定すると、最初のサービス(`site_update_manager.superadmin`)が渡されます。2番目を渡したい場合は、サービスを[手動でワイヤリング](https://symfony.com/doc/current/service_container.html#services-wire-specific-service)する必要があります。

Caution
: エイリアスを作成せずに [src/ からすべてのサービスをロード](https://symfony.com/doc/current/service_container.html#service-container-services-load-example)している場合、3 つのサービスが作成され (自動サービス + 2 つのサービス)、自動的にロードされたサービスは `SiteUpdateManager` をタイプヒントにしたときに - デフォルトで - 渡されます。そのため、エイリアスを作成するのは良いアイデアです。

