サービスの依存関係を自動的に定義する (オートワイヤリング)
================================================

> [オリジナル](https://symfony.com/doc/current/service_container/autowiring.html)

オートワイヤリングを使うと、最小限の設定でコンテナ内のサービスを管理することができます。これはコンストラクタ(もしくは他のメソッド)の型のヒントを読み、正しいサービスをそれぞれのメソッドに自動的に渡します。
Symfony のオートワイヤリングは予測可能なように設計されています: どの依存関係が渡されるべきかが絶対的に明確でない場合、アクション可能な例外が表示されます。

Tip
: symfony のコンパイル済みコンテナのおかげで、オートワイヤリングを使うためのランタイムオーバーヘッドはありません。

<span id="#an-autowiring-example"></span>
オートワイヤリングのサンプル
-----------------------

アルファベットの13文字を前方にシフトさせる楽しいエンコーダーである[ROT13](https://en.wikipedia.org/wiki/ROT13)で難読化されたTwitterフィード上でステータスを公開するAPIを構築しているとしましょう。

ROT13 トランスフォーマークラスの作成から始めます。

```php
namespace App\Util;

class Rot13Transformer
{
    public function transform($value)
    {
        return str_rot13($value);
    }
}
```

そして、このトランスフォーマークラスを使ったTwitterクライアントが登場しました。

```php
namespace App\Service;

use App\Util\Rot13Transformer;

class TwitterClient
{
    private $transformer;

    public function __construct(Rot13Transformer $transformer)
    {
        $this->transformer = $transformer;
    }

    public function tweet($user, $key, $status)
    {
        $transformedStatus = $this->transformer->transform($status);

        // ... connect to Twitter and send the encoded status
    }
}
```

デフォルトの[services.yamlの設定](./service_container.md#service-container-services-load-example)を使用している場合、**両方のクラスは自動的にサービスとして登録され、オートワイヤリングされます**。つまり、**何も**設定しなくてもすぐに使えるようになります。

しかし、オートワイヤリングをよりよく理解するために、以下の例では両方のサービスを明示的に設定しています。

```yaml
# config/services.yaml
services:
    _defaults:
        autowire: true
        autoconfigure: true
    # ...

    App\Service\TwitterClient:
        # defaults のおかげで不要ですが、値は各サービスでオーバーライド可能です。
        autowire: true

    App\Util\Rot13Transformer:
        autowire: true
```

これで、コントローラ内ですぐに `TwitterClient` サービスを利用できるようになりました。

```php
namespace App\Controller;

use App\Service\TwitterClient;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class DefaultController extends AbstractController
{
    /**
     * @Route("/tweet", methods={"POST"})
     */
    public function tweet(TwitterClient $twitterClient)
    {
        // fetch $user, $key, $status from the POST'ed data

        $twitterClient->tweet($user, $key, $status);

        // ...
    }
}
```

これは自動的に動作します。コンテナは `TwitterClient` サービスを作成する際の第一引数に `Rot13Transformer` サービスを渡すことを知っています。

<span id="#autowiring-logic-explained"></span>
オートワイヤリングロジックの解説
---------------------------

オートワイヤリングは `TwitterClient` の `Rot13Transformer` *タイプヒント*を読み込むことで動作します。

```php
// ...
use App\Util\Rot13Transformer;

class TwitterClient
{
    // ...

    public function __construct(Rot13Transformer $transformer)
    {
        $this->transformer = $transformer;
    }
}
```

オートワイヤリングシステムは、id がタイプヒントと正確に一致するサービスを探します。この場合、それは存在します。`Rot13Transformer` サービスを設定したときは、完全修飾されたクラス名を id として使用しました。オートワイヤリングは魔法ではなく、id がタイプヒントと一致するサービスを探します。[サービスを自動的にロードする場合](../service_container.html#service-container-services-load-example)、各サービスの id はそのクラス名になります。

id がタイプに正確にマッチするサービスが*ない*場合、明確な例外が投げられます。

オートワイヤリングは設定を自動化するための素晴らしい方法で、Symfony は可能な限り*予測可能*で明確なものにしようとします。

<span id="service-autowiring-alias" tabindex="-1"
style="outline: none;"></span>

<span ="#using-aliases-to-enable-autowiring"></span>
エイリアスを使用してオートワイヤリングを可能にする
---------------------------------

オートワイヤリングを設定する主な方法は、サービスの id がクラスと正確に一致するサービスを作成することです。前の例では、サービスの ID は `App\UtilUtilRot13Transformer` で、このタイプを自動的にオートワイヤリングすることができます。

これは、[エイリアス](alias_private.md#services-alias)を使っても実現できます。
何らかの理由で、サービスのIDが`app.rot13.transformer`ではなく、`app.rot13.transformer`になっていたとします。この場合、クラス名(`App\Util\Rot13Transformer`)でタイプヒントされた引数はオートワイヤリングされなくなります。

問題ありません。これを修正するには、サービスエイリアスを追加することで、IDがクラスと一致するサービスを*作成*することができます。

```yaml
# config/services.yaml
services:
    # ...

    # the id is not a class, so it won't be used for autowiring
    app.rot13.transformer:
        class: App\Util\Rot13Transformer
        # ...

    # but this fixes it!
    # the ``app.rot13.transformer`` service will be injected when
    # an ``App\Util\Rot13Transformer`` type-hint is detected
    App\Util\Rot13Transformer: '@app.rot13.transformer'
```

これは、サービスの「エイリアス」を作成し、そのIDは`App\UtilRot13Transformer`です。このおかげで、オートワイヤリングはこれを見て、`Rot13Transformer` クラスがタイプヒントされているときにそれを使用します。

Tip
: エイリアスは、サービスをオートワイヤリングできるようにするためにコアバンドルで使用されます。
例えば、MonologBundle は ID が `logger` であるサービスを作成します。
しかし、`logger` サービスを指す `Psr\Log\LoggerInterface` というエイリアスも追加します。これが、`Psr\LogLog\LoggerInterface` でタイプヒントされた引数をオートワイヤリングできる理由です。

<span id="#working-with-interfaces"></span>
インターフェースを使う
------------------

また、具体的なクラスの代わりに抽象化 (例: インターフェイス) をタイプ・ヒンティングしていることに気づくかもしれません。

このベストプラクティスに従うために、`TransformerInterface`を作成することにしたとします。

```php
namespace App\Util;

interface TransformerInterface
{
    public function transform($value);
}
```

そして、`Rot13Transformer` を更新して実装します。

```php
// ...
class Rot13Transformer implements TransformerInterface
{
    // ...
}
```

これでインターフェースができたので、これをタイプヒントにしましょう。

```php
class TwitterClient
{
    public function __construct(TransformerInterface $transformer)
    {
        // ...
    }

    // ...
}
```

しかし、今では、タイプヒント(`App\Util\TransformerInterface`)がサービスのID(`App\Util\Rot13Transformer`)と一致しなくなりました。つまり、引数をオートワイヤリングできなくなりました。

これを修正するには、[エイリアス](#service-autowiring-alias)を追加します。

```yaml
# config/services.yaml
services:
    # ...

    App\Util\Rot13Transformer: ~

    # `App\Util\TransformerInterface` タイプヒントが見つかった時に
    # `App\Util\Rot13Transformer` サービスが注入されます
    App\Util\TransformerInterface: '@App\Util\Rot13Transformer'
```

`App\UtilTransformerInterface`エイリアスのおかげで、オートワイヤリングサブシステムは、`TransformerInterface`を扱うときに`App\UtilRot13Transformer`サービスが注入されるべきであることを知っています。

Tip
    :[サービス定義のプロトタイプ](https://symfony.com/blog/new-in-symfony-3-3-psr-4-based-service-discovery)を使うとき、インターフェイスを実装したサービスが1つだけ発見され、そのインターフェイスも同じファイルで発見される場合、エイリアスを設定することは必須ではなく、Symfonyは自動的にエイリアスを作成します。 

<span id="#dealing-with-multiple-implementations-of-the-same-type"></span>
同じタイプの複数の実装への対応
--------------------------

`TransformerInterface` を実装した 2 番目のクラス `UppercaseTransformer` を作成したとします。

```php
namespace App\Util;

class UppercaseTransformer implements TransformerInterface
{
    public function transform($value)
    {
        return strtoupper($value);
    }
}
```

これをサービスとして登録すると、`App\UtilTransformerInterface`タイプを実装する2つのサービスができました。オートワイヤリング・サブシステムは、どちらを使用するかを決定することができません。オートワイヤリングは魔法ではないことを覚えておいてください。したがって、タイプから正しいサービス ID へのエイリアスを作成して、そのサービスを選択する必要があります（「[インターフェイスを使う](#autowiring-interface-alias)」を参照してください）。さらに、ある実装を使用したい場合や、別の実装を使用したい場合には、いくつかの名前付きのオートワイヤリングエイリアスを定義することができます。

たとえば、`TransformerInterface` インターフェースがタイプヒンティングされているときはデフォルトで `Rot13Transformer` 実装を使用し、特定のケースでは `UppercaseTransformer` 実装を使用したい場合があります。そのためには、`TransformerInterface` インターフェースから `Rot13Transformer` への通常のエイリアスを作成し、インターフェースを含む特別な文字列と、インジェクションを行う際に使用する変数名に一致する変数名から、名前付きオートワイヤリングエイリアスを作成することができます。

```php
namespace App\Service;

use App\Util\TransformerInterface;

class MastodonClient
{
    private $transformer;

    public function __construct(TransformerInterface $shoutyTransformer)
    {
        $this->transformer = $shoutyTransformer;
    }

    public function toot($user, $key, $status)
    {
        $transformedStatus = $this->transformer->transform($status);

        // ... Mastodonに接続して変換されたステータスを送信する
    }
}
```

```yaml
# config/services.yaml
services:
    # ...

    App\Util\Rot13Transformer: ~
    App\Util\UppercaseTransformer: ~

    # `$shoutyTransformer` が引数にある `App\Util\TransformerInterface`
    # タイプヒントが見つかった場合は `App\Util\UppercaseTransformer` 
    # サービスが注入さる
    App\Util\TransformerInterface $shoutyTransformer: '@App\Util\UppercaseTransformer'

    # インジェクションに使用される引数が一致しないが、
    # タイプヒントが一致する場合、`App\UtilRot13Transformer` 
    # サービスが注入される。
    App\Util\TransformerInterface: '@App\Util\Rot13Transformer'

    App\Service\TwitterClient:
        # Rot13Transformerが$transformerの引数として渡される
        autowire: true

        # デフォルトではないサービスを選択し、名前付きのオートワイヤリング
        # エイリアスを使用したくない場合は、手動でワイヤリングする
        #     $transformer: '@App\Util\UppercaseTransformer'
        # ...
```

`App\UtilUtilTransformerInterface`エイリアスのおかげで、このインターフェースで暗示された任意の引数タイプは、`App\UtilUtilRot13Transformer`サービスに渡されます。引数の名前が`$shoutyTransformer`の場合は、代わりに`App\UtilUppercaseTransformer`が使われます。しかし、引数キーの下で引数を指定することで、他のサービスを手動でワイヤリングすることもできます。

<span id="#fixing-non-autowireable-arguments"></span>
自動化できない引数を修正する
-----------------------

オートワイヤリングは、引数が*オブジェクト*の場合にのみ機能します。しかし、スカラの引数 (たとえば文字列) を持つ場合、これはオートワイヤリングできません。Symfonyは明確な例外を投げます。

これを修正するには、[問題のある引数を手動でワイヤリングします](service_container.md#services-manually-wire-args)。難しい引数をワイヤリングすると、Symfonyは残りの部分を処理します。

<span id="#autowiring-other-methods-e-g-setters"></span>
他のメソッドをオートワイヤリングする (例えば Setter)
--------------------------------------------

サービスに対してオートワイヤリングが有効になっている場合、インスタンス化されたときにクラスのメソッドを呼び出すようにコンテナを設定することもできます。たとえば、`logger` サービスに注入したい場合、セッター注入を使用することにしたとします。

```php
namespace App\Util;

class Rot13Transformer
{
    private $logger;

    /**
     * @required
     */
    public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function transform($value)
    {
        $this->logger->info('Transforming '.$value);
        // ...
    }
}
```

オートワイヤリングは、その上に `@required` アノテーションを付けたメソッドを自動的に呼び出し、各引数をオートワイヤリングします。引数の一部を手動でメソッドにワイヤリングする必要がある場合は、 [メソッドの呼び出しを明示的に設定](calls.md)することができます。

<span id="#autowiring-controller-action-methods"></span>
コントローラーのアクションメソッドをオートワイヤリングする
------------------------------------------------

symfony フレームワークを利用している場合、コントローラーのアクションメソッドに引数をオートワイヤリングすることもできます。これは利便性のために存在するオートワイヤリングのための特別なケースです。詳細は[サービスの取得](controller.html#controller-accessing-services)を参照してください。


<span id="#performance-consequences"></span>
パフォーマンスの影響
-------------------------------------------------------

Symfony のコンパイル済みコンテナのおかげで、オートワイヤリングを使うことによるパフォーマンスのペナルティは*ありません*。しかしながら、クラスを修正するときにコンテナがより頻繁にリビルドされるかもしれないので、開発環境では小さなパフォーマンスペナルティがあります。コンテナのリビルドが遅い場合(非常に大きなプロジェクトでありえます)、オートワイヤリングを利用できないかもしれません。


<span id="#public-and-reusable-bundles"></span>
公開された再利用可能なバンドル
-------------------------------------------------------------------

パブリックバンドルは明示的にサービスを設定し、オートワイヤリングに依存しないようにしてください。オートワイヤリングはコンテナ内で利用可能なサービスに依存しており、バンドルはそのコンテナに含まれるアプリケーションのサービスコンテナを制御することはできません。企業内で再利用可能なバンドルを構築する際には、すべてのコードを完全に制御できるため、オートワイヤリングを使用することができます。