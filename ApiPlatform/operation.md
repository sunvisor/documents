# オペレーション 

> [オリジナル](https://api-platform.com/docs/core/operations/)

API Platform Coreはオペレーションの概念に依存しています。  オペレーションは、APIによって公開されているリソースに適用することができます。
実装の観点から見ると、オペレーションはリソース、ルート、および関連するコントローラ間のリンクです。

API Platform はよくある [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) オペレーションを自動的に登録し、公開されているドキュメント (Hydra と Swagger) に記述します。
また、これらのオペレーションに対応するルートを作成し、Symfony のルーティングシステムに登録します(利用可能な場合)。

組み込みのオペレーションの動作は [Getting started](get-started.md#mapping-the-entities) ガイドで簡単に説明されています。

有効なオペレーションのリストはリソースごとに設定できます。
特定のルートに対してカスタムオペレーションを作成することも可能です。

オペレーションには、コレクションオペレーションとアイテムオペレーションの 2 種類があります。

コレクションオペレーションはリソースのコレクションに対してオペレーションを行います。デフォルトでは、`POST` と `GET` の2つのルートが実装されています。
アイテムオペレーションは個々のリソースを操作します。
デフォルトでは `GET`, `PUT`, `DELETE` の3つのルートが定義されています。(仕様上、[JSON API形式](content-negotiation.md)を使用する場合は、`PATCH`もサポートしています)。

エンティティクラスに `ApiPlatform\Core\Annotation\ApiResource` アノテーションを適用した場合、以下の組み込みCRUDオペレーションが自動的に有効になります。

*Collection operations*

メソッド | 必須      | 説明
-------|-----------|------------------------------------------
`GET`  | yes       | 要素のリストを取り出す(ページングされた)
`POST` | no        | 新しいエレメントを生成する

*Item operations*

メソッド   | 必須      | 説明
---------|-----------|-------------------------------------------
`GET`    | yes       | 要素を取り出す
`PUT`    | no        | 要素を置換する
`PATCH`  | no        | 要素に部分的な変更を適用する
`DELETE` | no        | 要素を削除する

注意: `PATCH` メソッドは設定で明示的に有効にしなければなりません。
詳しくは、[コンテンツ ネゴシエーション](content-negotiation.md)の項を参照してください。

## オペレーションを有効/無効にする

オペレーションを指定しない場合、すべてのデフォルトのCRUDオペレーションが自動的に登録されます。
また、オペレーションを明示的に定義することも可能であり、大規模なプロジェクトでは推奨されています。

`collectionOperations` と `itemOperations` は独立して動作することに注意してください。
例えば、`collectionOperations` のオペレーションを明示的に設定していない場合、`itemOperations` を明示的に設定したとしても `GET` と `POST` のオペレーションは自動的に登録されます。
その逆もまた真です。

オペレーションはアノテーション、 XML または YAML を用いて設定することができます。
以下の例では、`collectionOperations` と `itemOperations` の両方に対して `GET` メソッドの組み込みオペレーションのみを有効にして、読み取り専用のエンドポイントを作成します。

`itemOperations` と `collectionOperations` はオペレーションのリストを含む配列です。各オペレーションはオペレーション名に対応するキーで定義され、値はプロパティの配列です。
オペレーションのリストが空の場合、すべてのオペレーションは無効になります。

オペレーションの名前がサポートされている HTTP メソッド (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) と一致する場合、対応する `method` プロパティが自動的に追加されます。


```php
<?php
// api/src/Entity/Book.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * ...
 * @ApiResource(
 *     collectionOperations={"get"},
 *     itemOperations={"get"}
 * )
 */
class Book
{
    // ...
}
```

先ほどの例は、明示的にメソッドを定義して書くこともできます。

```php
<?php
// api/src/Entity/Book.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * ...
 * @ApiResource(
 *     collectionOperations={"get"={"method"="GET"}},
 *     itemOperations={"get"={"method"="GET"}}
 * )
 */
class Book
{
    // ...
}
```

あるいは、YAML設定フォーマットを使用することもできます。

```yaml
# api/config/api_platform/resources.yaml
App\Entity\Book:
    collectionOperations:
        get: ~ # nothing more to add if we want to keep the default controller
    itemOperations:
        get: ~
```

またはXML設定フォーマット。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- api/config/api_platform/resources.xml -->

<resources xmlns="https://api-platform.com/schema/metadata"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="https://api-platform.com/schema/metadata
           https://api-platform.com/schema/metadata/metadata-2.0.xsd">
    <resource class="App\Entity\Book">
        <itemOperations>
            <itemOperation name="get" />
        </itemOperations>
        <collectionOperations>
            <collectionOperation name="get" />
        </collectionOperations>
    </resource>
</resources>
```

API Platform Core はメソッド名をキーに指定するか、明示的に設定された HTTP メソッドをチェックするだけで、組み込みの CRUD アクションを参照して該当する Symfony ルートを自動的に登録します。

リソースアイテムへのアクセスを許可したくない場合 (つまり `GET` アイテムオペレーションを望まない場合)、リソースアイテムを完全に省略する代わりに、リソースアイテムを IRI で識別できるように、HTTP 404 (Not Found) を返す `GET` アイテムオペレーションを宣言します。例えば、以下のようになります。

```php
<?php
// api/src/Entity/Book.php

namespace App\Entity;

use ApiPlatform\Core\Action\NotFoundAction;
use ApiPlatform\Core\Annotation\ApiResource;

/**
 * @ApiResource(
 *     collectionOperations={
 *         "get",
 *     },
 *     itemOperations={
 *         "get"={
 *             "controller"=NotFoundAction::class,
 *             "read"=false,
 *             "output"=false,
 *         },
 *     },
 * )
 */
 class Book
 {
 }
```

## オペレーションの設定

URL、メソッド、デフォルトのステータスコード (その他のオプションのうち) をオペレーションごとに設定することができます。

次の例では、`GET` と `POST` の両方のオペレーションをカスタム URL で登録しています。
これらはデフォルトで生成された URL を上書きします。
さらに、`GET` オペレーションの URL の `id` パラメータに整数を指定し、`POST` リクエストが成功した後に生成されるステータスコードを `301` に設定します。

```php
<?php
// api/src/Entity/Book.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * ...
 * @ApiResource(
 *     collectionOperations={
 *         "post"={"path"="/grimoire", "status"=301}
 *     },
 *     itemOperations={
 *         "get"={
 *             "path"="/grimoire/{id}",
 *             "requirements"={"id"="\d+"},
 *             "defaults"={"color"="brown"},
 *             "options"={"my_option"="my_option_value"},
 *             "schemes"={"https"},
 *             "host"="{subdomain}.api-platform.com"
 *         }
 *     }
 * )
 */
class Book
{
    //...
}
```

YAML では

```yaml
# api/config/api_platform/resources.yaml
App\Entity\Book:
    collectionOperations:
        post:
            path: '/grimoire'
            status: 301
    itemOperations:
        get:
            method: 'GET'
            path: '/grimoire/{id}'
            requirements:
                id: '\d+'
            defaults:
                color: 'brown'
            host: '{subdomain}.api-platform.com'
            schemes: ['https']
            options:
                my_option: 'my_option_value'
```

XML では

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- api/config/api_platform/resources.xml -->

<resources xmlns="https://api-platform.com/schema/metadata"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="https://api-platform.com/schema/metadata
           https://api-platform.com/schema/metadata/metadata-2.0.xsd">
    <resource class="App\Entity\Book">
        <collectionOperations>
            <collectionOperation name="post">
                <attribute name="path">/grimoire</attribute>
                <attribute name="status">301</attribute>
            </collectionOperation>
        </collectionOperations>
        <itemOperations>
            <itemOperation name="get">
                <attribute name="path">/grimoire/{id}</attribute>
                <attribute name="requirements">
                    <attribute name="id">\d+</attribute>
                </attribute>
                <attribute name="defaults">
                    <attribute name="color">brown</attribute>
                </attribute>
                <attribute name="host">{subdomain}.api-platform.com</attribute>
                <attribute name="schemes">
                    <attribute>https</attribute>
                </attribute>
                <attribute name="options">
                    <attribute name="color">brown</attribute>
                </attribute>
            </itemOperation>
        </itemOperations>
    </resource>
</resources>
```

これらの例では、オペレーション名と一致するため `method` 属性は省略されています。

## すべてのオペレーションのすべてのルートを接頭辞で指定する

URI に関しては、リソース全体をそれ自身の "名前空間 "に入れることも有用な場合があります。
例えば、`Book` に関連するものをすべて `library` に入れて、URIが `library/book/{id}` になるようにしたいとしましょう。
この場合、パスを設定するためにすべてのオペレーションをオーバーライドする必要はなく、代わりにエンティティ全体に対して `route_prefix` 属性を設定します。

```php
<?php
// api/src/Entity/Book.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * @ApiResource(routePrefix="/library")
 */
class Book
{
    //...
}
```

あるいは、より冗長な属性構文を使用することもできます `@ApiResource(attributes={"route_prefix"="/library"})`。