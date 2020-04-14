# Filters

API Platform Core は、コレクションにフィルタやソート基準を適用するための汎用的なシステムを提供します。Doctrine ORM、MongoDB ODM、ElasticSearch 用の便利なフィルタが提供されています。

また、特定のニーズに合わせてカスタムフィルタを作成することもできます。ライブラリが提供するインターフェイスを実装することで、カスタム[データプロバイダ](data-providers.md)にフィルタリング機能を追加することもできます。

デフォルトでは、すべてのフィルタは無効になっています。明示的に有効にする必要があります。
デフォルトでは、すべてのフィルタが無効になっています。 明示的に有効にする必要があります。

フィルタが有効になると、コレクション レスポンスの `hydra:search` プロパティとして自動的に文書化されます。 また、[NelmioApiDocのマニュアル](nelmio-api-doc.md) が利用可能であれば自動的に表示されます。

## Doctrine ORM Filters

### 基礎知識

フィルタはサービスです（[カスタムフィルタ](#creating-custom-filters)のセクションを参照）。これらのフィルタは2つの方法でリソースにリンクできます。

1.  `ApiResource` 宣言を介して、`filters` 属性として。
たとえば、次のフィルタサービス宣言があるとします。

```yaml
# api/config/services.yaml
services:
    # ...
    offer.date_filter:
        parent: 'api_platform.doctrine.orm.date_filter'
        arguments: [ { dateProperty: ~ } ]
        tags:  [ 'api_platform.filter' ]
        # 以下は _defaults セクションが定義されている場合のみ必須です。
        # フィルタを追加しないように専用のファイルで分離したい場合があります。
        autowire: false
        autoconfigure: false
        public: false
```

`offer.date_filter` フィルタを `@ApiResource` アノテーションでリンクしています、

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * @ApiResource(attributes={"filters"={"offer.date_filter"}})
 */
class Offer
{
    // ...
}
```

または、YAML を使うと、

```yaml
# api/config/api_platform/resources.yaml
App\Entity\Offer:
    collectionOperations:
        get:
            filters: ['offer.date_filter']
    # ...
```

XML では、

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- api/config/api_platform/resources.xml -->

<resources xmlns="https://api-platform.com/schema/metadata"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="https://api-platform.com/schema/metadata
           https://api-platform.com/schema/metadata/metadata-2.0.xsd">
    <resource class="App\Entity\Offer">
        <collectionOperations>
            <collectionOperation name="get">
                <attribute name="filters">
                    <attribute>offer.date_filter</attribute>
                </attribute>
            </collectionOperation>
            <!-- ... -->
        </collectionOperations>
    </resource>
</resources>
```

2. `@ApiFilter` アノテーションを使って

このアノテーションは自動的にサービスを宣言し、必要なフィルタクラスを使用するだけです。

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\DateFilter;

/**
 * @ApiResource
 * @ApiFilter(DateFilter::class, properties={"dateProperty"})
 */
class Offer
{
    // ...
}
```

[ApiFilterアノテーション](filters.md#apifilter-annotation)の仕組みについて詳しくは、こちらをご覧ください。

一貫性を保つため、以下のドキュメントではアノテーションを使用しています。

### Search Filter

Doctrine ORMサポートが有効になっている場合、フィルターを追加するのは `api/config/services.yaml`にフィルターサービスを登録するのと同じくらい簡単です

The search filter supports `exact`, `partial`, `start`, `end`, and `word_start` matching strategies:
検索フィルタは `exact`、`partial`、 `start`、`end`、 `word_start` のマッチング戦略をサポートしています：

* `partial`戦略は`LIKE％text％ `を使ってテキストを含むフィールドを検索します。
* `start`戦略は`LIKE text％ `を使ってテキストで始まるフィールドを検索します。
* `end`戦略は`LIKE％text`を使ってテキストで終わるフィールドを検索します。
* `word_start`戦略は` LIKE text％OR LIKE％text％ `を使用して、` text`で始まる単語を含むフィールドを検索します。

大文字小文字を区別しないようにするには、フィルタに文字「i」を付加します。 たとえば `ipartial`や` iexact`などです。
これは `LOWER` 関数を使用し、[適切なインデックスがない場合](performance.md#search-filter)はパフォーマンスに影響を与えることがあることに注意してください。

大文字と小文字の区別は、使用されている[照合順序(Collation)](https://en.wikipedia.org/wiki/Collation)に応じてデータベースレベルで既に適用されている可能性があります。
MySQLを使用している場合、一般に使用される `utf8_unicode_ci`照合（およびその兄弟の` utf8mb4_unicode_ci`）は、名前に `_ci`部分で示されるように、大文字と小文字を区別しないことに注意してください。

次の例では、eコマース注文リストのフィルタリングをする方法を見ていきます。

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\SearchFilter;

/**
 * @ApiResource()
 * @ApiFilter(SearchFilter::class, properties={"id": "exact", "price": "exact", "name": "partial"})
 */
class Offer
{
    // ...
}
```

`http://localhost:8000/api/offers?price=10`は価格が正確に`10`であるすべてのオファーを返します。
`http：// localhost：8000 / api / offers？name = shirt`は、単語" shirt "を含むすべての注文を返します。


フィルターは組み合わせて使えます: `http://localhost:8000/api/offers?price=10&name=shirt`

リレーションについてもフィルタリングすることは可能です、`Offer` が `Product` を所有している時、

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * @ApiResource()
 * @ApiFilter(SearchFilter::class, properties={"product": "exact"})
 */
class Offer
{
    // ...
}
```

このサービス定義では、特定のIRIによって特定された製品に属するすべての注文を見つけることができます。
`http://localhost:8000/api/offers?product=/api/products/12` を試してください。
`http://localhost:8000/api/offers?product=12` のように数値ID も使えます。

前のURLでは、JSON-LD識別子として次のIRIを持つ製品のすべての注文を返します
(`@id`): `http://localhost:8000/api/products/12`.

### Date Filter

日付フィルタを使用すると、日付間隔でコレクションをフィルタリングできます。

文法: `?property[<after|before|strictly_after|strictly_before>]=value`

この値は、
[`\DateTime` constructor](http://php.net/manual/en/datetime.construct.php).
でサポートされている任意の日付形式を使うことができます。

`after`と` before` フィルタは値を含めてフィルタリングしますが、 `strictly_after`と` strictly_before` は値を除外してフィルタリングします。

他のフィルタリングをするときに、日付フィルタを明示的に有効にする必要があります。

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\DateFilter;

/**
 * @ApiResource
 * @ApiFilter(DateFilter::class, properties={"createdAt"})
 */
class Offer
{
    // ...
}
```

It will return all offers where `createdAt` is superior or equal to `2018-03-19`.
`createdAt` が `2018-03-19` よりも後か等しいすべての注文を返します。

#### `null` 値を扱う

date フィルタは、`null` 値を有する日付プロパティを扱うことができます。
フィルタのプロパティレベルでは、次の4つの動作を使用できます。


説明                                | セットする Strategy
-------------------------------------|------------------------------------------------------------------------------------
DBMS の規定の動作を使う             | `null`
アイテムを除外する                  | `ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\DateFilter::EXCLUDE_NULL` (`exclude_null`)
アイテムを最も古いと扱う            | `ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\DateFilter::INCLUDE_NULL_BEFORE` (`include_null_before`)
アイテムを最も新しいと扱う          | `ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\DateFilter::INCLUDE_NULL_AFTER` (`include_null_after`)

たとえば、次のサービス定義を使用すると、プロパティ値が `null` のエントリを除外します。

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\DateFilter;

/**
 * @ApiResource
 * @ApiFilter(DateFilter::class, properties={"dateProperty": DateFilter::EXCLUDE_NULL})
 */
class Offer
{
    // ...
}
```

### Boolean Filter

Boolean フィルタでは、Boolean 値のフィールドと値を検索できます。

文法: `?property=<true|false|1|0>`

フィルターを有効にするには次のようにします

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\BooleanFilter;

/**
 * @ApiResource
 * @ApiFilter(BooleanFilter::class, properties={"isAvailableGenericallyInMyCountry"})
 */
class Offer
{
    // ...
}
```

コレクションのエンドポイントが `/offers` であるとすると、`/offers?isAvailableGenericallyInMyCountry=true` というクエリでブール値で注文をフィルタリングできます。

`isAvailableGenericallyInMyCountry` が `true` である注文が返されます。

### Numeric Filter

数値フィルタを使用すると、数値フィールドと値を検索できます。

文法: `?property=<int|bigint|decimal...>`

フィルターを有効にするには次のようにします

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\NumericFilter;

/**
 * @ApiResource
 * @ApiFilter(NumericFilter::class, properties={"sold"})
 */
class Offer
{
    // ...
}
```

コレクションのエンドポイントが `/offers` であるとすると、`/offers?sold=1` というクエリでブール値でオファーをフィルタリングできます。

`sold` が `1` の注文が返されます。

### Range Filter

範囲フィルタを使用すると、より小さい、より大きい、以下、以上、および2つの値の間の値でフィルタリングできます。

文法: `?property[<lt|gt|lte|gte|between>]=value`

フィルターを有効にするには次のようにします

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\RangeFilter;

/**
 * @ApiResource
 * @ApiFilter(RangeFilter::class, properties={"price"})
 */
class Offer
{
    // ...
}
```

Given that the collection endpoint is `/offers`, you can filters the price with the following query: `/offers?price[between]=12.99..15.99`.
コレクションのエンドポイントが `/offers` であるとすれば、`/offers?price[between]=12.99..15.99` というクエリで価格をフィルタリングできます。

`price` が 12.99 から 15.99 までの注文を返します。

2つの値をつなげて注文をフィルタリングできます。例えば: `/offers?price[gt]=12.99&price[lt]=19.99`

### Exists Filter

存在フィルタを使用すると、null可能なフィールド値に基づいて項目を選択できます。

文法: `?property[exists]=<true|false|1|0>`

フィルターを有効にするには次のようにします

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\ExistsFilter;

/**
 * @ApiResource
 * @ApiFilter(ExistsFilter::class, properties={"transportFees"})
 */
class Offer
{
    // ...
}
```

コレクションのエンドポイントが `/offers` であるとすれば、`/offers?transportFees[exists]=true` というクエリでヌル可能フィールドの注文をフィルタリングできます。

It will return all offers where 
`transportFees` が `null` ではない注文を返します。

### Order Filter (ソート)

順序フィルタを使用すると、指定されたプロパティに対してコレクションをソートできます。

文法: `?order[property]=<asc|desc>`

フィルターを有効にするには次のようにします

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\OrderFilter;

/**
 * @ApiResource
 * @ApiFilter(OrderFilter::class, properties={"id", "name"}, arguments={"orderParameterName"="order"})
 */
class Offer
{
    // ...
}
```

コレクションのエンドポイントが `/offers`　であるとするとば、注文を名前順に昇順に、次にIDを降順に並べ替えることができます： `/offers?order[name]=desc&order[id]=asc`

デフォルトでは、クエリが方向を明示的に指定していない場合（たとえば、`/offers?order[name]&order[id]`）、デフォルトの指示方向を使用するように設定しない限り、フィルタは適用されません。

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\OrderFilter;

/**
 * @ApiResource
 * @ApiFilter(OrderFilter::class, properties={"id": "ASC", "name": "DESC"})
 */
class Offer
{
    // ...
}
```

#### Null 値との比較

順序付けに使用されるプロパティーに `null`値が含まれる場合、` null`値がどのように扱われるかを指定することができます

説明                            | セットする Strategy
--------------------------------|---------------------------------------------------------------------------------------
DBMS の規定の動作を使う         | `null`
アイテムを最小として扱う        | `ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\OrderFilter::NULLS_SMALLEST` (`nulls_smallest`)
アイテムを最大として扱う        | `ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\OrderFilter::NULLS_LARGEST` (`nulls_largest`)

例えば、次のサービス定義を使用して、プロパティ値が `null` のエントリを最小値として扱います。


```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\OrderFilter;

/**
 * @ApiResource
 * @ApiFilter(OrderFilter::class, properties={"validFrom": { "nulls_comparison": OrderFilter::NULLS_SMALLEST, "default_direction": "DESC" }})
 */
class Offer
{
    // ...
}
```

#### カスタムオーダークエリパラメータ名の使用

`order` が検索フィルタが有効になっているプロパティの名前でもある場合、競合が発生します。
幸いにも、使用するクエリパラメータ名は設定可能です：

```yaml
# api/config/packages/api_platform.yaml
api_platform:
    collection:
        order_parameter_name: '_order' # the URL query parameter to use is now "_order"
```

### ネストされたプロパティのフィルタリング

場合によっては、いくつかのリンクされたリソース（リレーションの反対側）に基づいてフィルタリングを実行できる必要があります。
次のように、すべてのビルトインフィルタは、ドット（`.`）構文を使用してネストされたプロパティをサポートします。

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\OrderFilter;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\SearchFilter;

/**
 * @ApiResource
 * @ApiFilter(OrderFilter::class, properties={"product.releaseDate"})
 * @ApiFilter(SearchFilter::class, properties={"product.color": "exact"})
 */
class Offer
{
    // ...
}
```

上記のようすると、それぞれの製品の色で注文を見つけることができます： `http://localhost:8000/api/offers?product.color=red`
または製品のリリース日で注文を並べ替えます
`http://localhost:8000/api/offers?order[product.releaseDate]=desc`

### リソースのすべてのプロパティのフィルタを有効にする

前の例で見たように、フィルタを適用できるプロパティは明示的に宣言する必要があります。
セキュリティとパフォーマンス（アクセスが制限されているAPIなど）を気にかけない場合は、すべてのプロパティに組み込みのフィルタを有効にすることもできます。

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\OrderFilter;

/**
 * @ApiResource
 * @ApiFilter(OrderFilter::class)
 */
class Offer
{
    // ...
}
```

**注意：ネストされたプロパティのフィルタは明示的に有効にする必要があります。**

このオプションに関係なく、フィルタは次の場合にのみプロパティに適用できます。

* そのプロパティが存在する
* その値がサポートされている (例: order フィルターにおける `asc` や `desc`）

これは、プロパティが次の場合、フィルタが**暗黙**で無視されることを意味します。

* 存在しない
* 有効でない
* 不正な値を持つ

## Serializer Filters

### Group Filter

グループフィルタを使用すると、シリアライゼーショングループをフィルタリングできます。

文法: `?groups[]=<group>`

必要な数のグループを追加できます。

フィルターを有効にするには次のようにします

```php
<?php

// api/src/Entity/Book.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Serializer\Filter\GroupFilter;

/**
 * @ApiResource
 * @ApiFilter(GroupFilter::class, arguments={"parameterName": "groups", "overrideDefaultGroups": false, "whitelist": {"allowed_group"}})
 */
class Book
{
    // ...
}
```

フィルターを設定する3つの引数が使えます。
- `parameterName` はクエリー パラメーターの名前です (デフォルトは `groups`)
- `overrideDefaultGroups` で既定のシリアル化グループをオーバーライドできます (デフォルトは `false`)
- `whitelist` 管理されていないデータエクスポージャーを避けるためのホワイトリストグループ (デフォルトは `null` で全てのグループを有効にします)

コレクションのエンドポイントが `/books` である場合、`/books?groups[]=read&groups[]=write` というクエリを使ってシリアライズ グループをフィルタリングすることができます。

### Property filter

プロパティフィルタは、シリアル化するプロパティを選択する可能性を追加します（まばらなフィールドセット）。

文法: `?properties[]=<property>&properties[<relation>]=<property>`

必要な数のプロパティを追加できます。

フィルターを有効にするには次のようにします

```php
<?php

// api/src/Entity/Book.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Serializer\Filter\PropertyFilter;

/**
 * @ApiResource
 * @ApiFilter(PropertyFilter::class, arguments={"parameterName": "properties", "overrideDefaultProperties": false, "whitelist": {"allowed_property"}})
 */
class Book
{
    // ...
}
```

フィルターを設定する3つの引数が使えます。
- `parameterName` はクエリー パラメーターの名前です (デフォルトは `groups`)
- `overrideDefaultGroups` で既定のシリアル化グループをオーバーライドできます (デフォルトは `false`)
- `whitelist` 管理されていないデータの暴露を避けるためのホワイトリストのプロパティ (デフォルトは `null` で全てのプロパティを有効にします)

Given that the collection endpoint is `/books`, you can filter the serialization properties with the following query: `/books?properties[]=title&properties[]=author`.
コレクションエンドポイントが `/books` である場合、`/books?properties[]=title&properties[]=author` というクエリでシリアライズ プロパティをフィルタリングできます。
ネストされた "author"ドキュメントのいくつかのプロパティを含めるには、
`/books?properties[]=title&properties[author]=name`
を使います。

## カスタムフィルターを作成する

カスタムフィルタは
`ApiPlatform\Core\Api\FilterInterface`
インタフェースを実装することで記述できます。

APIプラットフォームは、Doctrine ORMフィルタを作成する便利な方法を提供します。
[カスタムデータプロバイダ](data-providers.md)を使用している場合でも、前述のインターフェイスを実装してフィルタを作成できますが、API Platform は永続システムの内部を認識しないため、自分でフィルタリングロジックを作成する必要があります。

### カスタム Doctrine ORM フィルタを作成する

Doctrineフィルタは、HTTPリクエスト（Symfonyの `Request`オブジェクト）とデータベースからデータを取得するために使用される `QueryBuilder` インスタンスにアクセスできます。
それらはコレクションにのみ適用されます。 アイテムを取得するために生成されたDQLクエリを処理する場合、またはHTTPリクエストにアクセスする必要がない場合は、[extensions](extensions.md) を使用します。

Doctrine ORMフィルタは基本的に
`ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\FilterInterface`
を実装するクラスです。
APIプラットフォームには、このインタフェースを実装し、ユーティリティメソッドを提供する便利な抽象クラスが含まれています: `ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\AbstractFilter`

次の例では、プロパティに正規表現を適用してコレクションをフィルタリングするクラスを作成します。
この例で使用される `REGEXP` DQL関数は、[`DoctrineExtensions`](https://github.com/beberlei/DoctrineExtensions) ライブラリにあります。
この例を使用するには、このライブラリを適切にインストールして登録する必要があります（MySQLでのみ動作します）。

```php
<?php
// api/src/Filter/RegexpFilter.php

namespace App\Filter;

use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\AbstractFilter;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use Doctrine\ORM\QueryBuilder;

final class RegexpFilter extends AbstractFilter
{
    protected function filterProperty(string $property, $value, QueryBuilder $queryBuilder, QueryNameGeneratorInterface $queryNameGenerator, string $resourceClass, string $operationName = null)
    {
        $parameterName = $queryNameGenerator->generateParameterName($property); // Generate a unique parameter name to avoid collisions with other filters
        $queryBuilder
            ->andWhere(sprintf('REGEXP(o.%s, :%s) = 1', $property, $parameterName))
            ->setParameter($parameterName, $value);
    }

    // This function is only used to hook in documentation generators (supported by Swagger and Hydra)
    public function getDescription(string $resourceClass): array
    {
        if (!$this->properties) {
            return [];
        }

        $description = [];
        foreach ($this->properties as $property => $strategy) {
            $description["regexp_$property"] = [
                'property' => $property,
                'type' => 'string',
                'required' => false,
                'swagger' => [
                    'description' => 'Filter using a regex. This will appear in the Swagger documentation!',
                    'name' => 'Custom name to use in the Swagger documentation',
                    'type' => 'Will appear below the name in the Swagger documentation',
                ],
            ];
        }

        return $description;
    }
}
```

次に、このフィルタをサービスとして登録します。

```yaml
# api/config/services.yaml
services:
    # ...
    'App\Filter\RegexpFilter':
        # Uncomment only if autoconfiguration isn't enabled
        #tags: [ 'api_platform.filter' ]
```

前の例では、任意のプロパティにフィルタを適用できます。 しかし、 `AbstractFilter` クラスのおかげで、いくつかのプロパティに対しても有効にすることができます：

```yaml
# api/config/services.yaml
services:
    'App\Filter\RegexpFilter':
        arguments: [ '@doctrine', '@request_stack', '@?logger', { email: ~, anOtherProperty: ~ } ]
        # Uncomment only if autoconfiguration isn't enabled
        #tags: [ 'api_platform.filter' ]
```

最後に、フィルタリングするリソースにこのフィルタを追加します。

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;
use App\Filter\RegexpFilter;

/**
 * @ApiResource(attributes={"filters"={RegexpFilter::class}})
 */
class Offer
{
    // ...
}
```

あるいは、 `ApiFilter` アノテーションを使って：

```php
<?php
// api/src/Entity/Offer.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use App\Filter\RegexpFilter;

/**
 * @ApiResource
 * @ApiFilter(RegexpFilter::class)
 */
class Offer
{
    // ...
}
```

`http://example.com/offers?regexp_email=^[FOO]`.
のようなURLを使ってこのフィルタを有効にすることができます。
この新しいフィルターは、SwaggerやHydraの文書にも表示されます。

### Doctrine フィルターを使う

Doctrine は、開発者がクエリの条件節にSQLを追加できるようにする[フィルタシステム](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/filters.html)を備えています。
これらはSQLが生成される場所（DQLクエリや関連エンティティのロードなど）に関係なく、コレクションやアイテムに適用されるため、非常に便利です。

SymfonyのDoctrineフィルタに固有の以下の情報は、[MichaëlPerrinのブログに投稿された素晴らしい記事](http://blog.michaelperrin.fr/2014/12/05/doctrine-filters/)に基づいています。

Suppose we have a `User` entity and an `Order` entity related to the `User` one. A user should only see his orders and no others's ones.
`User` エンティティと `User` エンティティに関連する `Order` エンティティがあるとします。
ユーザーは自分の注文のみを表示し、他のユーザーは表示しません。

```php
<?php
// api/src/Entity/User.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * @ApiResource
 */
class User
{
    // ...
}
```

```php
<?php
// api/src/Entity/Order.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ApiResource
 */
class Order
{
    // ...

    /**
     * @ORM\ManyToOne(targetEntity="User")
     * @ORM\JoinColumn(name="user_id", referencedColumnName="id")
     **/
    private $user;
}
```

全体のアイデアは、注文テーブルのどのクエリでも `WHERE user_id =：user_id` という条件を追加する必要があるということです。

まず制限付きエンティティをマークするためのカスタムアノテーションを作成します。

```php
<?php
// api/Annotation/UserAware.php

namespace App\Annotation;

use Doctrine\Common\Annotations\Annotation;

/**
 * @Annotation
 * @Target("CLASS")
 */
final class UserAware
{
    public $userFieldName;
}
```

次に、 `Order` エンティティを 「User を理解する」エンティティとしてマークしましょう。

```php
<?php
// api/src/Entity/Order.php

namespace App\Entity;

use App\Annotation\UserAware;

/**
 * @UserAware(userFieldName="user_id")
 */
class Order {
    // ...
}
```

ここで、Doctrineフィルタクラスを作成します。

```php
<?php
// api/src/Filter/UserFilter.php

namespace App\Filter;

use App\Annotation\UserAware;
use Doctrine\ORM\Mapping\ClassMetaData;
use Doctrine\ORM\Query\Filter\SQLFilter;
use Doctrine\Common\Annotations\Reader;

final class UserFilter extends SQLFilter
{
    private $reader;

    public function addFilterConstraint(ClassMetadata $targetEntity, string $targetTableAlias): string
    {
        if (null === $this->reader) {
            return throw new \RuntimeException(sprintf('An annotation reader must be provided. Be sure to call "%s::setAnnotationReader()".', __CLASS__));
        }

        // The Doctrine filter is called for any query on any entity
        // Check if the current entity is "user aware" (marked with an annotation)
        $userAware = $this->reader->getClassAnnotation($targetEntity->getReflectionClass(), UserAware::class);
        if (!$userAware) {
            return '';
        }

        $fieldName = $userAware->userFieldName;
        try {
            // Don't worry, getParameter automatically escapes parameters
            $userId = $this->getParameter('id');
        } catch (\InvalidArgumentException $e) {
            // No user id has been defined
            return '';
        }

        if (empty($fieldName) || empty($userId)) {
            return '';
        }

        return sprintf('%s.%s = %s', $targetTableAlias, $fieldName, $userId);
    }

    public function setAnnotationReader(Reader $reader): void
    {
        $this->reader = $reader;
    }
}
```

ここで、Doctrine フィルタを設定する必要があります。

```yaml
# api/config/packages/api_platform.yaml
doctrine:
    orm:
        filters:
            user_filter:
                class: App\Filter\UserFilter
```

そして、リクエストごとにバンドルサービス宣言ファイルを現在のユーザで Doctrine フィルタを初期化するリスナーを追加します。

```yaml
# api/config/services.yaml
services:
    # ...
    'App\EventListener\UserFilterConfigurator':
        tags:
            - { name: kernel.event_listener, event: kernel.request, priority: 5 }
        # Autoconfiguration must be disabled to set a custom priority
        autoconfigure: false
```

[この問題](https://github.com/api-platform/core/issues/1185)にフラグが立てられているように、優先順位を `ApiPlatform\Core\EventListener\ReadListener` の優先順位より高く設定することは重要です。
さもなくば `PaginatorExtension` はDoctrineフィルタを無視し、不正な` totalItems`と `page`（first/last/next）データを返します。

最後に、コンフィギュレータクラスを実装します。

```php
<?php
// api/EventListener/UserFilterConfigurator.php

namespace App\EventListener;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;
use Doctrine\Common\Persistence\ObjectManager;
use Doctrine\Common\Annotations\Reader;

final class UserFilterConfigurator
{
    private $em;
    private $tokenStorage;
    private $reader;

    public function __construct(ObjectManager $em, TokenStorageInterface $tokenStorage, Reader $reader)
    {
        $this->em = $em;
        $this->tokenStorage = $tokenStorage;
        $this->reader = $reader;
    }

    public function onKernelRequest(): void
    {
        if (!$user = $this->getUser()) {
            throw new \RuntimeException('There is no authenticated user.');
        }

        $filter = $this->em->getFilters()->enable('user_filter');
        $filter->setParameter('id', $user->getId());
        $filter->setAnnotationReader($this->reader);
    }

    private function getUser(): ?UserInterface
    {
        if (!$token = $this->tokenStorage->getToken()) {
            return null;
        }

        $user = $token->getUser();
        return $user instanceof UserInterface ? $user : null;
    }
}
```

完了：Doctrineはすべての "UserAware"エンティティを自動的にフィルタリングします！

### リクエストのプロパティ抽出をオーバーライドする

リクエストからフィルタパラメータを抽出する方法を変更することができます。
これは、 `extractProperties(\Symfony\Component\HttpFoundation\Request $request)` メソッドをオーバーライドすることで行うことができます。

次の例では、order フィルタの構文を完全に変更して次のようにします： `?filter[order][property]`

```php
<?php
// api/src/Filter/CustomOrderFilter.php

namespace App\Filter;

use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\OrderFilter;
use Symfony\Component\HttpFoundation\Request;

final class CustomOrderFilter extends OrderFilter
{
    protected function extractProperties(Request $request): array
    {
        return $request->query->get('filter[order]', []);
    }
}
```

最後に、カスタムフィルタを登録します。

```yaml
# api/config/services.yaml
services:
    # ...
    'App\Filter\CustomOrderFilter': ~
        # Uncomment only if autoconfiguration isn't enabled
        #tags: [ 'api_platform.filter' ]
```

## ApiFilter アノテーション

アノテーションは `property` や `class` で使用できます。

アノテーションがプロパティに与えられている場合、そのプロパティにフィルタが設定されます。
例えば、 `name` や ` colors` リレーションの `prop` プロパティに検索フィルタを追加しましょう：

```php
<?php

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\SearchFilter;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ApiResource
 */
class DummyCar
{
    /**
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string")
     * @ApiFilter(SearchFilter::class, strategy="partial")
     */
    private $name;

    /**
     * @ORM\OneToMany(targetEntity="DummyCarColor", mappedBy="car")
     * @ApiFilter(SearchFilter::class, properties={"colors.prop": "ipartial"})
     */
    private $colors;
    
    // ...
}

```

最初のプロパティ `name` は、簡単です。
最初のアノテーション引数はフィルタクラス、2番目のオプションを指定します。ここでは strategy です：
```
@ApiFilter(SearchFilter::class, strategy="partial")
```

2番目の注釈では、フィルタを適用する `properties` を指定します。
`colors` をフィルタリングするのではなく、` colors` リレーションのプロパティ `prop` をフィルタリングしたいので、必要になります。
与えられたプロパティごとに strategy を指定することに注意してください。

```
@ApiFilter(SearchFilter::class, properties={"colors.prop": "ipartial"})
```

`ApiFilter` アノテーションもクラスで設定できます。
プロパティを指定しないと、クラスのすべてのプロパティに作用します。

例えば、3つのデータフィルタ（ `DateFilter`、` SearchFilter`と `BooleanFilter`）と` DummyCar` クラスの2つのシリアライズ フィルタ（`PropertyFilter` と ` GroupFilter`）を定義しましょう：

```php
<?php
// api/src/Entity/DummyCar.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Annotation\ApiResource;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\BooleanFilter;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\DateFilter;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\SearchFilter;
use ApiPlatform\Core\Serializer\Filter\GroupFilter;
use ApiPlatform\Core\Serializer\Filter\PropertyFilter;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ApiResource
 * @ORM\Entity
 * @ApiFilter(BooleanFilter::class)
 * @ApiFilter(DateFilter::class, strategy=DateFilter::EXCLUDE_NULL)
 * @ApiFilter(SearchFilter::class, properties={"colors.prop": "ipartial", "name": "partial"})
 * @ApiFilter(PropertyFilter::class, arguments={"parameterName": "foobar"})
 * @ApiFilter(GroupFilter::class, arguments={"parameterName": "foobargroups"})
 */
class DummyCar
{
    // ...
}

```

`BooleanFilter` はクラスのすべての ` Boolean` プロパティに適用されます。
実際、各コアフィルタでは、Doctrine 型をチェックします。
これは、フィルタークラスを使用してのみ記述されます。

```
@ApiFilter(BooleanFilter::class)
```

ここに与えられた `DateFilter` は ` DummyCar` クラスの `Date` プロパティに ` DateFilter::EXCLUDE_NULL` ストラテジーで適用されます：

```
@ApiFilter(DateFilter::class, strategy=DateFilter::EXCLUDE_NULL)
```

ここの `SearchFilter` はプロパティを追加します。 結果は、プロパティの注釈を使用した例とまったく同じです。

```
@ApiFilter(SearchFilter::class, properties={"colors.prop": "ipartial", "name": "partial"})
```

すべてのフィルタで `properties` 引数を指定できることに注意してください。

次のフィルタは、データの取り込み方法とは関係なく、シリアライズの方法に関係なく、 `arguments` オプション（[利用可能な引数についてはこちらを参照してください](#serializer-filters)を与えることができます。

```
@ApiFilter(PropertyFilter::class, arguments={"parameterName": "foobar"})
@ApiFilter(GroupFilter::class, arguments={"parameterName": "foobargroups"})
```