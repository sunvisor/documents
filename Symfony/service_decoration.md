サービスをデコレートする方法
=======================

> [オリジナル](https://symfony.com/doc/current/service_container/service_decoration.html#control-the-behavior-when-the-decorated-service-does-not-exist)

既存の定義をオーバーライドすると、元のサービスが失われます。

```yaml
# config/services.yaml
services:
    App\Mailer: ~

    # これは、古い App\Mailer の定義を新しいものに置き換え、
    # 古い定義は失われます。
    App\Mailer:
        class: App\NewMailer
```

ほとんどの場合は、まさにその通りです。しかし、時には古いものをデコレーションしたくなることもあるでしょう (つまり [Decorator パターン](https://en.wikipedia.org/wiki/Decorator_pattern)を適用する)。この場合、古いサービスは、新しいサービスで参照できるように残しておくべきです。この設定では、`App\Mailer` を新しいものに置き換えますが、古いものの参照は `App\DecoratingMailer.inner` として保持します。

```yaml
# config/services.yaml
services:
    App\Mailer: ~

    App\DecoratingMailer:
        # `App\Mailer` サービスをオーバーライドしても、そのサービスは 
        # `App\DecoratingMailer.inner` として利用可能です。
        decorates: App\Mailer
```

`decorates` オプションは、`App\DecoratingMailer` サービスが `App\Mailer` サービスを置き換えることをコンテナに伝えます。デフォルトの `services.yaml` 構成を使用している場合、`decorating` サービスのコンストラクタが `decorating` サービスクラスで1つのタイプヒントされた引数を持っているときに `decorating` サービスが自動的に注入されます。

オートワイヤリングを使用していない場合や、decorating サービスのコンストラクタの引数がdecorating サービスクラスでタイプヒントされたものを複数持つ場合は、明示的にdecorating サービスを注入する必要があります (デコレートされたサービスのIDは自動的に`decorating_service_id + '.inner'` に変更されます)。

```yaml
# config/services.yaml
services:
    App\Mailer: ~

    App\DecoratingMailer:
        decorates: App\Mailer
        # pass the old service as an argument
        arguments: ['@App\DecoratingMailer.inner']
```

Tip
: 新サービスのエイリアスである、デコレーションされた `App\Mailer` サービスの可視性は、元の `App\Mailer` の可視性と同じになります。

Info
: 生成された内側のIDは、デコレータサービス(ここでは `App\DecoratingMailer`)のIDに基づいており、デコレータサービス(ここでは `App\Mailer`)のIDではありません。`decoration_inner_name` オプションで、内側のサービス名を制御することができます。
    ```yaml
    # config/services.yaml
    services:
        App\DecoratingMailer:
            # ...
            decoration_inner_name: App\DecoratingMailer.wooz
            arguments: ['@App\DecoratingMailer.wooz']
    ```

<span id="decoration-priority"></span>
デコレーションの優先順位
---------------------------------------------------

サービスに複数のデコレータを適用する場合は、`decoration_priority` オプションでその順番を制御することができます。この値は整数で、デフォルトは 0 で、優先度が高いほどデコレータの適用が早くなります。

```yaml
# config/services.yaml
Foo: ~

Bar:
    decorates: Foo
    decoration_priority: 5
    arguments: ['@Bar.inner']

Baz:
    decorates: Foo
    decoration_priority: 1
    arguments: ['@Baz.inner']
```

生成されるコードは以下のようになります。

```php
$this->services[Foo::class] = new Baz(new Bar(new Foo()));
```

<span id="control-the-behavior-when-the-decorated-service-does-not-exist"></span>
装飾されたサービスが存在しない場合の行動を制御する
-------------------------------------------

存在しないサービスをデコレーションする場合、decoration_on_invalid オプションで採用する動作を選択できます。

3つの異なる動作が用意されています。


- `exception`: デコレータの依存関係がないことを示す ServiceNotFoundException がスローされます。  (デフォルト)
- `ignore`: コンテナはデコレータを削除します。
- `null`: コンテナはデコレータサービスを保持し、デコレータを `null` に設定します。

```yaml
# config/services.yaml
Foo: ~

Bar:
    decorates: Foo
    decoration_on_invalid: ignore
    arguments: ['@Bar.inner']
```

Caution
: `null` を使用する場合は、デコレータのコンストラクタを更新しなければならないかもしれません。
    ```php
    namespace App\Service;

    use Acme\OptionalBundle\Service\OptionalService;

    class DecoratorService
    {
        private $decorated;

        public function __construct(?OptionalService $decorated)
        {
            $this->decorated = $decorated;
        }

        public function tellInterestingStuff(): string
        {
            if (!$this->decorated) {
                return 'Just one interesting thing';
            }

            return $this->decorated->tellInterestingStuff().' + one more interesting thing';
        }
    }
    ```

info
: 時々、その場でサービス定義を作成するコンパイラパスを追加したい場合があります。そのようなサービスをデコレーションしたい場合は、デコレーションパスが作成されたサービスを見つけられるように、コンパイラーパスが `PassConfig::TYPE_BEFORE_OPTIMIZATION` タイプに登録されていることを確認してください。 

