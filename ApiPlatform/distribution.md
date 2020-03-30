# API Platform を始める: ハイパーメディアと GraphQL API、管理者とプログレッシブWebアプリ

> [オリジナル](https://api-platform.com/docs/distribution/)

> *API Platform*は、どのようなフレームワークや言語の中でも、最も先進的なAPIプラットフォームです。
>
> SymfonyCon 2017、Fabien Potencier (Symfonyの生みの親)

[API Platform](https://api-platform.com) は、API駆動型プロジェクトに特化した強力で使いやすい **フルスタック** フレームワークです。
業界をリードする標準規格（[JSON-LD](https://json-ld.org/)、**[Hydra](https://www.hydra-cg.com/)**、[GraphQL](https://graphql.org/)、[OpenAPI](https://www.openapis.org/)...）をサポートした完全な機能を備えたAPIを作成するための **PHP** ライブラリが含まれています。
また、これらの API をスナップで使用するための野心的な **JavaScript** ツールを提供しています (管理者、PWA とモバイルアプリのジェネレーター、ハイパーメディアクライアント...)。

最も簡単で強力な方法は、API Platform ディストリビューションをダウンロードすることです。これには以下が含まれています。

* [サーバーサイドコンポーネント](../core/index.md)、[Symfony 4 マイクロフレームワーク](https://symfony.com/doc/current/setup/flex.html)、[Doctrine ORM](https://www.doctrine-project.org/projects/orm.html) を含む API スケルトン (オプション)
* API Platform (またはHydra API) のハイパーメディア機能を活用し、[React Admin](https://marmelab.com/react-admin/) 上に構築された、[ダイナミックJavaScriptアドミン](.../admin/)
* [クライアントジェネレーター](./client-generator/)を使用して、Hydra APIから1つのコマンドで[React](https://reactjs.org), [Vue](https://vuejs.org/), [React Native](https://facebook.github.io/react-native/), [Next.js](https://nextjs.org/), [Quasar](https://quasar.dev/), [Vuetify](https://vuetifyjs.com/)アプリをスカッフォルディングすることができます。
* [Docker](https://docker.com)ベースのセットアップで、単一のコマンドでプロジェクトをブートストラップすることができます。
  * API と JavaScript アプリのためのサーバ
  * [API Platform の組み込みの無効化キャッシュ機構](.../core/performance.md#enabling-the-built-in-http-cache-invalidation-system)を有効にする[Varnish Cache](https://varnish-cache.org/)サーバー
  * 開発用のHTTP/2とHTTPSプロキシ (例えば、提供された[サービスワーカー](https://developer.mozilla.org/fr/docs/Web/API/Service_Worker_API)をテストすることができる)
  * [Kubernetes](https://kubernetes.io/) クラスタに API をデプロイするための [Helm](https://helm.sh/) チャート

フレームワークの仕組みを知るために、本屋を運営するためのAPIを作成します。

完全な機能を備えたAPI、管理者インターフェース、プログレッシブなWebアプリを作成するには、 API の公開データモデルを設計し、*POPO (Plain Old PHP Objects)*としてそれを手作りする必要があります。

API Platformは、これらのモデルクラスを使用して、多くの組み込み機能を持つWeb APIを公開し、ドキュメント化します。

* リソースの作成、取得、更新、削除(CRUD)
* データの検証
* ページネーション
* フィルタリング
* 選別
* ハイパーメディア/[HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)とコンテンツネゴシエーションのサポート([JSON-LD](https://json-ld.org)と[Hydra](https://www.hydra-cg.com/)、[JSON:API](https://jsonapi.org/)、[HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08)...)
* [GraphQLサポート](https://graphql.org/)
* 素敵なUIとマシンリーダブルなドキュメント ([Swagger UI/OpenAPI](https://swagger.io), [GraphiQL](https://github.com/graphql/graphiql)...)
* 認証 ([Basic HTTP](https://en.wikipedia.org/wiki/Basic_access_authentication)、クッキー、[JWT](https://jwt.io/)、拡張機能による[OAuth](https://oauth.net/)など)
* [CORSヘッダ](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
* セキュリティチェックとヘッダ（[OWASP の推奨事項](https://www.owasp.org/index.php/REST_Security_Cheat_Sheet)に対してテストずみ)
* 無効化ベースのHTTPキャッシング
* 基本的には、最新のAPIを構築するために必要なすべてのもの。

始める前にもう一つ: APIプラットフォームのディストリビューションには[Symfonyフレームワーク](https://symfony.com)が含まれています。
はほとんどの [Symfony バンドル](https://flex.symfony.com) (プラグイン) と互換性があり、この強固な基盤 (イベント、DIC...) によって提供される [多数の拡張ポイント](../core/extending.md) から利益を得られます。
カスタム、サービス指向、API エンドポイント、JWT や OAuth 認証、HTTP キャッシング、メール送信や非同期ジョブのような機能を API に追加するのは簡単です。
One more thing, before we start: as the API Platform distribution includes [the Symfony framework](https://symfony.com),

## フレームワークのインストール

### API Platform ディストリビューションを利用する (推奨)

まず、[API Platformのディストリビューション `.tar.gz` ファイル](https://github.com/api-platform/api-platform/releases/latest)をダウンロードするか、[提供するテンプレートからGitHubリポジトリを生成して](https://github.com/api-platform/api-platform/generate)ください。
展開したディレクトリには、API Platformのプロジェクト構造が含まれています。その中に独自のコードや設定を追加することになります。

**注**
: [パーミッション](https://github.com/api-platform/api-platform/issues/319#issuecomment-307037562)の[問題](https://github.com/api-platform/api-platform/issues/777#issuecomment-412515342)が発生する可能性があるため。
`.zip` ファイルの使用は避けるようにしてください。

API Platformには、コンテナ化された開発環境を簡単に立ち上げることができる[Docker](https://docker.com)のセットアップが同梱されています。
まだDockerがインストールされていない場合は、[今がインストールのチャンス](https://docs.docker.com/install/)です。

Macでは[Docker for Mac](https://docs.docker.com/docker-for-mac/)のみ対応しています。
同様に、Windowsでは[Docker for Windows](https://docs.docker.com/docker-for-windows/)のみサポートされています。Docker Machineは**がサポートされていません**。

ターミナルを開き、プロジェクトのスケルトンを含むディレクトリに移動します。以下のコマンドを実行して、[Docker Compose](https://docs.docker.com/compose/)を使用してすべてのサービスを起動します。

```
$ docker-compose pull # Download the latest versions of the pre-built images
$ docker-compose up -d # Running in detached mode
```

これにより、以下のサービスが開始されます。

| 名前     | 説明                       | ポート                    | 環境                  |
|----------|--------------------------|---------------------------|----------------------|
| php      | API と PHP, PHP-FPM 7.3, Composer とセンシティブなコンフィグ | n/a     | all                                                |
| db       | PostgreSQL データベース サーバー                           | 5432    | all (prod ではマネージドサービスの使用を推奨)       |
| client   | プログレッシブWebアプリの開発サーバー                        | 443     | dev (prod では静的なウェブサイトのホスティングサービスを使う) |
| admin    | 管理者用の開発サーバー                                     | 444     | dev (prod では静的なウェブサイトのホスティングサービスを使う) |
| api      | API 用の HTTP サーバー (NGINX)                           | n/a     | all                                                |
| vulcain  | [Vulcain](https://vulcain.rocks) のゲートウェイ          | 8443    | all                                                |
| mercure  | Mercure ハブ, [リアルタイム機能のため](../core/mercure.md) | 1337    | all (prod ではマネージドバージョンの使用を推奨)     |

コンテナのログを見るには、次を実行してください。

```
$ docker-compose logs -f # follow the logs
```

プロジェクトファイルは、事前に設定された[Docker volume](https://docs.docker.com/engine/tutorials/dockervolumes/)のおかげで、ローカルホストマシンとコンテナの間で自動的に共有されます。
つまり、ローカルでお好みのIDEやコードエディタを使ってプロジェクトのファイルを編集することができ、それらはコンテナ内で透過的に考慮されます。
IDEといえば、APIプラットフォームアプリを開発するためのお気に入りのソフトウェアは、[Symfony](https://www.jetbrains.com/phpstorm/+Started+-+Symfony+Development+using+PhpStorm)と[Php Inspections](https://plugins.jetbrains.com/plugin/7622-php-inspections-ea-extended-)プラグインがある[PHPStorm](https://confluence.jetbrains.com/display/PhpStorm/Getting)です。
これらを試してみてください。品質の分析によりほとんどすべてのものを自動補完することができます。

API Platformのディストリビューションには、テスト用にダミーのエンティティが付属しています: `api/src/Entity/Greeting.php`。これは後で削除します。

PHP のエコシステムに慣れている人であれば、このテストエンティティは業界をリードする [Doctrine ORM](https://www.doctrine-project.org/projects/orm.html) ライブラリを永続化システムとして利用していることがお分かりになると思います。
これは API Platform ディストリビューションで配布されています。
Doctrine ORM は API Platform プロジェクトのデータを永続化したり、クエリを実行したりするための最も簡単な方法です。
パフォーマンスと開発の利便性のために最適化されています。
例えば Doctrine を使うと、API Platform は適切な `JOIN` 句を追加することで生成された SQL クエリを自動的に最適化することができます。
また、強力な組み込みフィルタも多数用意されています。
Doctrine ORM とそのブリッジは PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, SQLite などの一般的な RDBMS をサポートしています。
オプションで同梱の [Doctrine MongoDB ODM](https://www.doctrine-project.org/projects/mongodb-odm.html) もサポートしています。

ただし、API Platformは永続化システムとは100%独立していることを覚えておいてください。[適切なインターフェイス](./core/data-providers.md)を実装することで、必要に応じて最適なもの(NoSQLデータベースやリモートWebサービスを含む)を使用することができます。
API Platform は、同一プロジェクト内で複数の永続化システムを併用することも可能です。

### Symfony Flex と Composer を使う (上級者向け)

また、API Platformサーバーコンポーネントをローカルマシンに直接インストールすることもできます。
**この方法は、ディレクトリ構造とインストールされた依存関係を完全に制御したい上級ユーザーにのみお勧めします。**

[良い紹介のために、SymfonyCasts上で配布物なしでAPI Platformをインストールする方法をご覧ください。](https://symfonycasts.com/screencast/api-platform/install?cid=apip).

このチュートリアルの残りの部分は、公式ディストリビューションを使用してAPI Platformをインストールしたことを前提としています。その場合は次のセクションに進んでください。

API Platform には公式の Symfony Flex レシピがあります。つまり、[Composer](https://getcomposer.org/) を使えば、任意のFlex対応のSymfonyアプリケーションから簡単にインストールできます。

```bash
# Symfony Flex プロジェクトを作成する
$ composer create-project symfony/skeleton bookshop-api
# プロジェクトのフォルダに移動
$ cd bookshop-api
# API Platform のサーバーコンポーネントをインストール
$ composer req api
```

次に、データベースとスキーマを作成します。

```bash
$ bin/console doctrine:database:create
$ bin/console doctrine:schema:create
```

PHP のビルトインサーバーを起動します。

```bash
# Built-in PHP server
$ php -S 127.0.0.1:8000 -t public
```

また、すべてのJavaScriptコンポーネントは、npm または Yarn でインストール可能な[スタンドアロンライブラリとして利用可能](https://github.com/api-platform?language=javascript)です。 

**注**
: この方法でAPI Platformをインストールすると、APIは `/api/` パスで公開されます。 
APIのドキュメントを見るには `http://localhost:8000/api/` を開く必要があります。
API Platform を Apache や NGINX のウェブサーバに直接デプロイしていて、このリンクを開いたときに404エラーが出る場合は、特定のウェブサーバソフトウェア用の[書き換えルール](https://symfony.com/doc/current/setup/web_server_configuration.html)を有効にする必要があります。

## 準備ができました

プラウザで `https://localhost` を開きます。

![The welcome page](
https://api-platform.com/static/03f362d88e4214426697883fa908fce5/a7549/api-platform-2.5-welcome.png
)

フレームワークのインストール時にこのコンテナ用に生成された自己署名 TLS 証明書を受け入れるために、ブラウザにセキュリティ例外を追加する必要があります。
HTTPS 経由で利用可能な他のすべてのサービスについて、このステップを繰り返します。

後日、このウェルカム画面をプログレッシブ ウェブアプリのホームページに置き換えることになるでしょう。
プログレッシブ ウェブアプリを作成する予定がない場合は、`docker-compose.yaml` の `client/` ディレクトリと関連する行を削除することができます (今はしないでください。このチュートリアルでは後でこのコンテナを使用します)。

HTTPS API」ボタンをクリックするか、`https://localhost:8443/` に移動します。

![The API](https://api-platform.com/static/81e5bcf6ebc7b4bca54ef915b9bcae33/a7549/api-platform-2.5-api.png)

API Platformは、APIの説明を[OpenAPI](https://www.openapis.org/)形式(以前はSwaggerとして知られていました)で公開しています。
また、カスタマイズされたバージョンの [Swagger UI](https://swagger.io/swagger-ui/) も統合されており、Open API ドキュメントをレンダリングするための素晴らしいインターフェイスとなっています。
操作をクリックすると詳細が表示されます。UIから直接APIにリクエストを送ることもできます。
`POST` オペレーションを使って新しい *Greeting* リソースを作成し、`GET` オペレーションを使ってアクセスし、最後に `DELETE` オペレーションを実行して削除してみてください。
Webブラウザを使って API URL にアクセスした場合、API Platform はそれを検出し (`Accept` HTTPヘッダをスキャンすることで)、対応するAPIリクエストをUIに表示します。
試しに `http://localhost:8443/greetings` にアクセスしてみてください。
もし `Accept` ヘッダに `text/html` が指定されていない場合は、JSON-LD レスポンスが送信されます([設定可能な動作です](../core/content-negotiation.md))。

そして、生データにアクセスしたい場合は、2つの選択肢があります。

* 正しい `Accept` ヘッダを追加する (セキュリティを気にしない場合は `Accept` ヘッダを一切設定しない) - API クライアントを書く場合におすすめ。
* リソースの拡張子として必要なフォーマットを追加する - デバッグ目的のためだけに

例えば、`Greeting` リソースのリストを JSON-LD で取得するには `http://localhost:8443/greetings.jsonld` を、生の JSON でデータを取得するには `http://localhost:8443/greetings.json` を使います。

もちろん、お好きな HTTP クライアントを使って API に問い合わせをすることもできます。
私たちは[Postman](https://www.getpostman.com/)を愛用しています。それは API プラットフォームと完全にうまく動作し、ネイティブのオープン API をサポートしており、簡単に機能テストを書くことができ、良いチームコラボレーション機能を持っています。

## 自分のモデルを追加する

これでAPI Platformプロジェクトは100%機能するようになりました。
独自のデータモデルを公開してみましょう。
本屋のAPIはシンプルなものになります。
リソース型は `Book` と `Review` で構成されます。

書籍には、ID、ISBN、タイトル、説明、著者、出版日があり、レビューのリストに関連しています。
レビューには、ID、評価（0から5の間）、本文、著者、出版日があり、1冊の本に関連しています。

このデータモデルを POPO (Plain Old PHP Objects) のセットとして記述し、Doctrine ORM のアノテーションを使ってデータベースのテーブルにマッピングしてみましょう。

```php
<?php
// api/src/Entity/Book.php

namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * A book.
 *
 * @ORM\Entity
 */
class Book
{
    /**
     * @var int 本のID
     *
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @var string|null 本のISBN (ない場合は null).
     *
     * @ORM\Column(nullable=true)
     */
    public $isbn;

    /**
     * @var string 本のタイトル
     *
     * @ORM\Column
     */
    public $title;

    /**
     * @var string 本の説明
     *
     * @ORM\Column(type="text")
     */
    public $description;

    /**
     * @var string 本の著者
     *
     * @ORM\Column
     */
    public $author;

    /**
     * @var \DateTimeInterface 本の出版日
     *
     * @ORM\Column(type="datetime")
     */
    public $publicationDate;

    /**
     * @var Review[] この本のレビュー
     *
     * @ORM\OneToMany(targetEntity="Review", mappedBy="book", cascade={"persist", "remove"})
     */
    public $reviews;
    
    public function __construct()
    {
        $this->reviews = new ArrayCollection();
    }

    public function getId(): ?int
    {
        return $this->id;
    }
}
```

```php
<?php
// api/src/Entity/Review.php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * A review of a book.
 *
 * @ORM\Entity
 */
class Review
{
    /**
     * @var int レビューのID
     *
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @var int このレビューのレーティング (0 から 5 の間).
     *
     * @ORM\Column(type="smallint")
     */
    public $rating;

    /**
     * @var string レビューの本文
     *
     * @ORM\Column(type="text")
     */
    public $body;

    /**
     * @var string レビューの著者
     *
     * @ORM\Column
     */
    public $author;

    /**
     * @var \DateTimeInterface このレビューの公開日付
     *
     * @ORM\Column(type="datetime")
     */
    public $publicationDate;

    /**
     * @var Book このレビューの対象となる本
     *
     * @ORM\ManyToOne(targetEntity="Book", inversedBy="reviews")
     */
    public $book;

    public function getId(): ?int
    {
        return $this->id;
    }
}
```

**ヒント**: `--api-resource` オプションのおかげで Symfony の [MakerBundle](https://symfony.com/doc/current/bundles/SymfonyMakerBundle/index.html) を使うこともできます。

```bash
$ docker-compose exec php bin/console make:entity --api-resource
```

ご覧のように、対応する PHPDoc を持つ 2 つのよくある PHP オブジェクトがあります (エンティティやプロパティの PHPDoc に含まれる説明は、API ドキュメントに記載されていることに注意してください)。

Doctrine のアノテーションはこれらのエンティティをデータベースのテーブルにマッピングします。
アノテーションはコードと設定をグループ化できるので便利ですが、クラスとメタデータを切り離したい場合は XML や YAML のマッピングに切り替えることができます。
これらもサポートされています。

Doctrine ORM でエンティティをマッピングする方法については、[プロジェクトの公式ドキュメント](https://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/association-mapping.html) や Kévin の著書 "[Persistence in PHP with the Doctrine ORM](https://www.amazon.fr/gp/product/B00HEGSKYQ/ref=as_li_tl?ie=UTF8&camp=1642&creative=6746&creativeASIN=B00HEGSKYQ&linkCode=as2&tag=kevidung-21)" を参照してください。

シンプルにするために、この例ではパブリックプロパティを使いました (id 以外は、これについては後述)。Doctrine と同様に API Platform もアクセサメソッド (ゲッター/セッター) をサポートしています。
id はプライベートプロパティにして、ゲッターを使って、読み込み専用であることを強制しています (ID は `@ORMGeneratedValue` アノテーションのおかげで RDMS が生成してくれます)。
API Platform は UUID のファーストグレードのサポートも持っています。
[自動インクリメントされた ID ではなく](https://www.clever-cloud.com/blog/engineering/2015/05/20/why-auto-increment-is-a-terrible-idea/)、UUID を使用した方が良いでしょう。

そして、`api/src/Entity/Greeting.php`というファイルを削除します。このデモのエンティティはもう使いません。
最後に、Doctrine にデータベースのテーブル構造を新しいデータモデルと同期させるように指示します。

```bash
$ docker-compose exec php bin/console doctrine:schema:update --force
```

`php` コンテナは API アプリが存在する場所でです。
コマンドの前に `docker-compose exec php` を付けることで、このコンテナ内で指定したコマンドを実行することができます。
[エイリアスを作成する](http://www.linfo.org/alias.html)と便利です。

後日、データベースの構造を変更する際には [Doctrine Migrations](https://symfony.com/doc/current/doctrine.html#migrations-creating-the-database-tables-schema) を使うことになるでしょう。

これで、永続化とクエリが可能な作業データモデルができました。
エンティティクラスに対応するCRUD機能を持つAPIエンドポイントを作成するには、`@ApiResource`というアノテーションでマークするだけです。

```php
<?php
// api/src/Entity/Book.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * ...
 *
 * @ApiResource
 */
class Book
{
    // ...
}
```

```php
<?php
// api/src/Entity/Review.php

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;

/**
 * ...
 *
 * @ApiResource
 */
class Review
{
    // ...
}
```

**API は (ほぼ) 完成です!**
`https://localhost:8443` を開いて開発環境 (素晴らしい [Symfony profiler](https://symfony.com/blog/new-in-symfony-2-8-redesigned-profiler) を含む)をロードします。

![The bookshop API](https://api-platform.com/static/97d1114b5a8ea9445e2cb0daece78fcb/a7549/api-platform-2.5-bookshop-api.png)

2つのリソースタイプで利用できる操作がUIに表示されます。

リソースタイプ `Book` の `POST` オペレーションをクリックし、「試してみる」をクリックして、以下のJSON文書をリクエストボディとして送信します。

```json
{
  "isbn": "9781782164104",
  "title": "Persistence in PHP with the Doctrine ORM",
  "description": "This book is designed for PHP developers and architects who want to modernize their skills through better understanding of Persistence and ORM.",
  "author": "Kévin Dunglas",
  "publicationDate": "2013-12-01"
}
```

ブックショップ API を通じて新しいブックリソースが保存されました! API Platform は JSON ドキュメントを自動的に対応する PHP エンティティクラスのインスタンスに変換し、Doctrine ORM を使ってデータベースに永続化します。

デフォルトでは、`GET` (コレクションやアイテムの取得)、`POST` (作成)、`PUT` (置換)、`PATCH` (部分更新)、そして`DELETE` (自明) のHTTPメソッドがサポートされています。[不要なものは無効にする](./core/operations.md#enabling-and-disabling-operations)のを忘れずに!

コレクションに対して `GET` 操作をしてみましょう。追加した本が表示されます。コレクションに30 以上のアイテムが含まれている場合、自動的にページネーションが表示されます。
[フィルタを追加して、コレクションにソートを追加する](../core/filters.md)にも興味があるかもしれません。

生成される JSON レスポンスの中には、`@id`, `@type`, `@context` などの記号で始まるキーがあることにお気づきでしょうか?
API Platform は [JSON-LD](https://json-ld.org/) フォーマット (および、その拡張機能である [Hydra](https://www.hydra-cg.com/) を完全にサポートしています。
それはスマートクライアントを構築することを可能にし、数行で発見する API Platform Admin のような自動検出機能を備えています。
オープンデータ、SEO、相互運用性に便利で、特に[Schema.orgなどのオープンボキャブラリーと一緒に使う](http://blog.schema.org/2013/06/schemaorg-and-json-ld.html)と、[Googleに構造化データへのアクセスを与える](https://developers.google.com/search/docs/guides/intro-structured-data)や、[Apache Jena](https://jena.apache.org/documentation/io/#formats)を使って[SPARQL](https://en.wikipedia.org/wiki/SPARQL)でAPIをクエリすることができます)。

新しいAPIのデフォルトフォーマットとしては、JSON-LDが最適だと考えています。
しかし、API Platformは、[GraphQL](https://graphql.org/) (後ほど説明します)、[JSON API](https://jsonapi.org/)、[HAL](https://github.com/zircote/Hal)、生の[JSON](https://www.json.org/)、[XML](https://www.w3.org/XML/) (実験的)、さらには[YAML](https://yaml.org/)や[CSV](https://en.wikipedia.org/wiki/Comma-separated_values)など、他の多くのフォーマットをネイティブに[サポートしています](../core/content-negotiation.md)。
また、簡単に[他のフォーマットのサポートを追加する](./core/content-negotiation.md)こともできますし、どのフォーマットを有効にしてデフォルトで使用するかはあなた次第です。

さて、`Review` リソースの `POST` 操作を使って、この本のレビューを追加します。

```json
{
    "book": "/books/1",
    "rating": 5,
    "body": "Interesting book!",
    "author": "Kévin",
    "publicationDate": "September 21, 2016"
}
```

今回のリクエストについては、面白いことが2つあります。

まず、リレーションの扱い方を学びました。ハイパーメディアAPIでは、すべてのリソースは(一意の)  [IRI](https://en.wikipedia.org/wiki/Internationalized_Resource_Identifier) で識別されます。
 URL は有効な IRI であり、API Platform が使用するものです。すべての JSON-LD ドキュメントの `@id` プロパティには、それを識別する IRI が含まれています。
この IRI を使用して、他のドキュメントからこのドキュメントを参照することができます。先ほどのリクエストでは、先ほど作成した書籍の IRI を使って、作成中の `Review` とリンクさせました。
API Platform は IRI の扱いがスマートです。
ちなみに、ドキュメントを参照するのではなく、[ドキュメントを埋め込む](.../core/serialization.md)ほうがいいかもしれません (HTTPリクエストの数を減らすためなど)。[クライアントに必要なプロパティだけを選択させる](../core/filters.md#property-filter)という方法もあります。

もう一つの興味深い点は、API Platformがどのように日付(`publicationDate`プロパティ)を扱うかということです。
API Platform は、[PHPでサポートされている任意の日付フォーマット](https://www.php.net/manual/en/datetime.formats.date.php)を理解します。
本番環境では、[RFC 3339](https://tools.ietf.org/html/rfc3339) で指定されたフォーマットを使用することを強く推奨しますが、ご覧のように `September 21, 2016` を含むほとんどの一般的なフォーマットを使用することができます。

要約すると、任意のエンティティを公開したい場合は、あなたがする必要があるのは

1. バンドルの `Entity` ディレクトリに置く
2. Doctrine を使う場合はデータベースにマッピングする
3. `@ApiPlatform\Core\Annotation\ApiResource` アノテーションでマークする

簡単でしょう?!

## データのバリデーション

次に、`/books` に `POST` リクエストを発行して、以下のようなボディで別の本を追加してみましょう。

```json
{
  "isbn": "2815840053",
  "description": "Hello",
  "author": "Me",
  "publicationDate": "today"
}
```

おっと、タイトルを追加するのを忘れていました。とにかくリクエストを送信すると、以下のようなメッセージで500エラーが出るはずです。

```
An exception occurred while executing 'INSERT INTO book [...] VALUES [...]' with params [...]:
SQLSTATE[23000]: Integrity constraint violation: 1048 Column 'title' cannot be null
```

エラーは自動的に JSON-LD でシリアライズされ、Hydra Coreのエラー用語彙に準拠していることにお気づきでしたか？
これにより、クライアントはエラーから有用な情報を簡単に抽出することができます。
ともあれ、リクエストを送信するときに SQL エラーが出るのはまずいですね。
入力の検証をしていなかったことになりますし、[それは悪くて危険な行為です](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Input_Validation_Cheat_Sheet.md)。

API Platform には [Symfony Validator Component](https://symfony.com/doc/current/validation.html) とのブリッジが付属しています。
[Symfony Validator Component](https://symfony.com/doc/current/validation.html#supported-constraints) にいくつかの [多数の検証制約](https://symfony.com/doc/current/validation.html#supported-constraints) (もしくは [カスタム制約の作成](https://symfony.com/doc/current/validation/custom_constraint.html)) を追加するだけで、ユーザーが投稿したデータを検証するのに十分です。データモデルにいくつかのバリデーションルールを追加してみましょう。

```php
<?php
// api/src/Entity/Book.php

namespace App\Entity;

use Symfony\Component\Validator\Constraints as Assert;

// ...
class Book
{
    /**
     * ...
     * @Assert\Isbn
     */
    public $isbn;

    /**
     * ...
     * @Assert\NotBlank
     */
    public $title;

    /**
     * ...
     * @Assert\NotBlank
     */
    public $description;

    /**
     * ...
     * @Assert\NotBlank
     */
    public $author;

    /**
     * ...
     * @Assert\NotNull
     */
    public $publicationDate;

    // ...
}
```

```php
<?php
// api/src/Entity/Review.php

namespace App\Entity;

use Symfony\Component\Validator\Constraints as Assert;

// ...
class Review
{
    /**
     * ...
     * @Assert\Range(min=0, max=5)
     */
    public $rating;

    /**
     * ...
     * @Assert\NotBlank
     */
    public $body;

    /**
     * ...
     * @Assert\NotBlank
     */
    public $author;

    /**
     * ...
     * @Assert\NotNull
     */
    public $publicationDate;

    /**
     * ...
     * @Assert\NotNull
     */
    public $book;

    // ...
}
```

これらの `@Assert\*` アノテーションを追加してエンティティを更新した後 (Doctrine と同様に XML や YAML を使うこともできます)、前回の `POST` リクエストをもう一度試してみてください。

```json
{
  "@context": "/contexts/ConstraintViolationList",
  "@type": "ConstraintViolationList",
  "hydra:title": "An error occurred",
  "hydra:description": "isbn: This value is neither a valid ISBN-10 nor a valid ISBN-13.\ntitle: This value should not be blank.",
  "violations": [
    {
      "propertyPath": "isbn",
      "message": "This value is neither a valid ISBN-10 nor a valid ISBN-13."
    },
    {
      "propertyPath": "title",
      "message": "This value should not be blank."
    }
  ]
}
```

これで、常に Hydra エラーフォーマット([RFC 7807](https://tools.ietf.org/html/rfc7807) もサポート) を使用してシリアライズされた適切な検証エラーメッセージを取得できるようになりました。
これらのエラーはクライアント側で簡単に解析できます。適切なバリデーション制約を追加することで、提供されたISBNが有効でないことにも気付きました。

## GraphQLのサポートを追加する

API Platform は REST **と** GraphQL フレームワークではないのか？ その通りです。GraphQLのサポートはデフォルトでは有効になっていません。
追加するには、[graphql-php](https://webonyx.github.io/graphql-php/) ライブラリをインストールする必要があります。
以下のコマンドを実行してください（キャッシュを2回クリアする必要があります）。

```bash
$ docker-compose exec php composer req webonyx/graphql-php && docker-compose exec php bin/console cache:clear
```

これで GraphQL APIができました! API Platform をインストールするために Symfony Flexを使っている場合は `https://localhost:8443/graphql` (または `https://localhost:8443/api/graphql`) を開いて、API Platform に同梱されている素敵な [GraphiQL](https://github.com/graphql/graphiql) UIを使って遊んでみましょう。

![GraphQL endpoint](https://api-platform.com/static/b5897e1b639627a64bed82ada1443803/a7549/api-platform-2.5-graphql.png)

試しに greeting を作成してみてください。

```graphql
mutation {
  createGreeting(input: {name: "Test2"}) {
    greeting {
      id
      name
    }
  }
}
```

And by reading out the greeting:
```graphql
{
  greeting(id: "/greetings/1") {
    id
    name
    _id
  }
}
```

GraphQL の実装は、[クエリ](https://graphql.org/learn/queries/)、[変異](https://graphql.org/learn/queries/#mutations)、[Relayサーバ仕様の100%](https://facebook.github.io/relay/docs/en/graphql-server-specification.html)、ページネーション、[フィルタ](./core/filters.md)、[アクセス制御ルール](./core/security.md)をサポートしています。
人気のある [RelayJS](https://facebook.github.io/relay/) や [Apollo](https://www.apollographql.com/docs/react/) クライアントで使用できます。

## 管理画面

APIによって公開されたデータを管理するための管理バックエンドがあればいいと思いませんか？
待ってください... すでにあるじゃないですか!

ブラウザで `https://localhost:444` を開きます。

![The admin](https://api-platform.com/static/ac3c1b75ced7d7ed2c555fb2870718e1/a7549/api-platform-2.5-admin.png)

この [Material デザイン](https://material.io/guidelines/) の管理画面は、[API Platform Admin](./admin/index.md) で構築された[プログレッシブ ウェブアプリ](https://developers.google.com/web/progressive-web-apps/)です (内部はReact Admin、React、Redux)。
これは強力で、完全にカスタマイズ可能です。
詳細はドキュメントを参照してください。
API コンポーネントによって公開されている Hydra のドキュメントを利用して自分自身を構築します。
100% 動的です - **コード生成は発生しせん**。

## React のプログレッシブ ウェブアプリ

API Platformには、完全に動作する React/Redux や [Vue.js](https://vuejs.org/) のプログレッシブ ウェブアプリをスカッフォルドして、簡単にチューニングやカスタマイズができる素晴らしい[クライアントジェネレータ](./client-generator/index.md)も用意されています。
また、モバイルデバイスのすべての機能を活用したい場合は、[React Native](https://facebook.github.io/react-native/) もサポートしています。

ディストリビューションには、生成されたコードの React フレーバーを歓迎するためのスケルトンが付属しています。アプリを起動するには、以下を実行してください。

```bash
$ docker-compose exec client generate-api-platform-client
```

`client/src/index.js` を開き、コンソールに表示されるコピー/ペーストの指示に従います。その後、ブラウザで `https://localhost/books/` を開きます。
Open `client/src/index.js` and follow the copy/pasting instructions displayed in the console. Then open `https://localhost/books/` in your browser:

![The React Progressive Web App](https://api-platform.com/static/ac3c1b75ced7d7ed2c555fb2870718e1/a7549/api-platform-2.5-admin.png)

また、`--resource` 引数で特定のリソースのコードを生成することもできます (例: `generate-api-platform-client --resource books`)。

生成されたコードには、リスト（ページネーションを含む）、削除ボタン、作成、編集フォームが含まれています。また、障害者がアプリを利用できるようにするために、[Bootstrap 4](https://getbootstrap.com)のマークアップと[ARIAの役割](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)も含まれています。

Vue.jsの上に構築されたPWAやネイティブのモバイルアプリを生成したい場合は、[専用ドキュメント](./client-generator/index.md)を参照してください。

## 自分のビジネスロジックをフックする

基本的なことを学んだところで、[一般的な設計上の考慮事項](./core/design.md)と[API Platformの拡張方法](./core/extending.md)を読んで、API Platformがどのように設計されているか、そしてカスタムビジネスロジックをどのようにフックするかを理解してください!

## その他の機能

まずは、[アプリケーションのデプロイ方法](./deployment/index.md)を[組み込みのKubernetesインテグレーション](./deployment/kubernetes.md)を使ってクラウドにデプロイする方法を学んでみましょう。

この他にも、学ぶべき機能はたくさんあります。[ドキュメント全文](../core/index.md)を読んで、それらの使い方や API Platform を自分のニーズに合わせて拡張する方法を見つけてください。
API Platform はプロトタイピングやラピッドアプリケーション開発 (RAD) には非常に効率的ですが、フレームワークのほとんどは、単純な CRUD アプリをはるかに超えた複雑な API 駆動型プロジェクトを作成するために設計されています。
また、[**強力な拡張ポイント**](./core/extending.md)の恩恵を受け、[パフォーマンス](./core/performance.md)のために**継続的に最適化**されています。
多数の高トラフィックのウェブサイトにパワーを供給しています。

API PlatformにはHTTPキャッシュ無効化システムが組み込まれており、API Platformアプリを高速に動作させることができ、デフォルトでは[Varnish](https://varnish-cache.org/)を使用しています。
詳細は [API Platform Core Library: ビルトイン HTTP キャッシュ無効化システムを有効にする](../core/performance.md#enabling-the-built-in-http-cache-invalidation-system)の章を参照してください。。

好みのクライアントサイド技術が使えることを覚えておきましょう。API Platform では React や Vue.js のコンポーネントを提供していますが、 Angular や Ionic、Swift などお好きなクライアントサイドの技術を使うことができます。
HTTP リクエストを送信できる言語であれば何でもOKです（COBOLでもOK）。

さらに、API Platformチームは、シリアライゼーショングループの活用、ユーザー管理、JWTやOAuth認証など、より高度なユースケースを示すデモアプリケーションを公開しています。
デモコードのソースをGitHubでチェックアウトする](https://github.com/api-platform/demo)と、[オンラインで閲覧する](https://demo.api-platform.com)があります。

## スクリーンキャスト

<p align="center" class="symfonycasts"><a href="https://symfonycasts.com/tracks/rest?cid=apip#api-platform"><img src="https://api-platform.com/static/15d4f39d6b8a59d2f18851c95aac069d/b4640/symfonycasts-player.png" alt="SymfonyCasts, API Platform screencasts"></a></p>

API Platformの使い方を学ぶ最も簡単で楽しい方法は、[SymfonyCastsで公開されている60以上のスクリーンキャスト](https://symfonycasts.com/tracks/rest?cid=apip#api-platform)を見ることです!