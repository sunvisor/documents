# Sencha Cmdを使ったプログレッシブ Web アプリの構築

> [オリジナル: Building Progressive Web Apps with Sencha Cmd](https://docs.sencha.com/cmd/7.3.0/guides/progressive_web_apps.html)

Cmd 6.5+は、Ext JSアプリをプログレッシブなWebアプリに変えるのに役立ちます。

プログレッシブウェブアプリは、最新のウェブ技術を使用してネイティブアプリの体験を提供します。Google はプログレッシブWebアプリの優れた入門書を提供しています: [Your First Progressive Web App](https://codelabs.developers.google.com/codelabs/your-first-pwapp/#0)

Cmdは特に開発者に以下のことを可能にします。

  - Androidユーザーをホーム画面にアプリをインストールするように誘うアプリバナーを表示します。
  - この機能をサポートするブラウザのために、サービスワーカーベースのキャッシングを介してオフラインサポートを提供します。

**注:**CmdプログレッシブWebアプリを使用するには、まず [Node JS](https://nodejs.org/en/) がローカルにインストールされている必要があります。

## 必要条件

プログレッシブ Web アプリの機能を使用するには、Node JS 6 以上が必要です。

## ホーム画面のバナーに追加

「ホーム画面に追加」バナーをアプリに追加するには、`progressive` config オブジェクトを app.json に追加します。プログレッシブコンフィグオブジェクトには、`manifest` と `serviceWorker` の2つの項目があります。`manifest` config には、[Web アプリのマニフェスト](https://developer.mozilla.org/en-US/docs/Web/Manifest) のデータが含まれています。この config が配置されると、ビルド時に必要な `<link rel="manifest"/>` タグが自動的にアプリの `index.html` ファイルに追加されます。

Webアプリのマニフェストが存在するからといって、サポートされているブラウザがホーム画面に追加したバナーをすぐに表示するとは限りません。たとえば、Chromeは、サービスワーカーの使用、SSLステータス、訪問頻度ヒューリスティックを含む一連の基準を使用して、いつ[バナーを表示](https://codelabs.developers.google.com/codelabs/your-first-pwapp/#3) するかを判断します。

Example:

```json
"progressive": {
    "manifest": {

        "name": "My App",
        "short_name": "My App",
        "icons": [{
            "src": "resources/icon-small.png",
            "sizes": "96x96"
        }, {
            "src": "resources/icon-medium.png",
            "sizes": "192x192"
        }, {
            "src": "resources/icon-large.png",
            "sizes": "256x256"
        }],
        "theme_color": "#054059",
        "background_color": "#054059",
        "display": "standalone",
        "orientation": "portrait",
        "start_url": "/index.html"
    }
}
```

## Service Worker でのキャッシング

**注**. sw-precache の使用は、Workbox の代わりに非推奨となりました。Sencha Cmd は今後のリリースで Workbox をサポートする予定です。

おそらく、プログレッシブウェブアプリの最も重要な機能は、オフライン機能を提供することです。一般的に、これはアプリシェル（アプリを表示するために必要なHTML、JavaScript、CSS、フォント、および画像アセット）とコンテンツをキャッシュすることで行われ、通常はREST APIへのAJAX呼び出しを介して取得されます。

プログレッシブなウェブアプリでは、キャッシュを提供するために [service workers](https://developers.google.com/web/fundamentals/getting-started/primers/service-workers) を使用します。`progressive` config が `app.json` にある場合、Cmd はビルド時に [sw-precache](https://github.com/GoogleChromeLabs/sw-precache) を使ってサービスワーカーを作成し、アプリシェルを自動的にキャッシュします。コード内に@sw-cacheコメントを追加することで、サービスワーカーにAJAXリクエストをキャッシュさせることもできます。


## // @sw-cache

サーバーから予定されているイベントのリストを取得するストアを想像してみてください。URLに `@sw-cache` コメントを追加することで、Cmdによって生成されたサービスワーカーにAPIコールをキャッシュさせ、オフライン時に結果を利用できるようにすることができます。

```javascript
Ext.define('App.store.UpcomingEvents', {
    extend: 'Ext.data.Store',
    proxy: {
        type: 'ajax',

        // @sw-cache
        url: '/api/upcoming-events.json',
        reader: {
            type: 'json'
        }
    }
});
```

コメント `@sw-cache` は、任意の文字列やオブジェクトのプロパティの上に置くことができます。このコメントがアタッチされる文字列は、キャッシュされる URL でなければなりません。 URL が動的な場合 (例えば、パスやクエリ文字列に id を含むなど)、どのリクエストがキャッシュされるかを制御するために "urlPattern " config をオプションで指定することができます。たとえば、特定のイベントに対する AJAX リクエストなどです。

```javascript
Ext.define('App.model.Event', {
    extend: 'Ext.data.Model',
    fields: ['id', 'name', 'date'],

    proxy: {
        type: 'rest',

        // @sw-cache { urlPattern: "\\/api\\/events\\/\\d+" }
        url : '/events'
    }
});
```

## キャッシュの動作を制御する

コメント `@sw-cache` にはいくつかのオプションを指定することができます。

  - **urlPattern**: 正規表現です。これにマッチしたときにレスポンスがキャッシュされるようになります。

  - **handler**: デフォルトは "networkFirst" です。以下のいずれかになります。
    
      - **networkFirst** ネットワークからフェッチしてリクエストを処理しようとします。成功した場合は、キャッシュにレスポンスを格納します。そうでなければ、キャッシュからリクエストを処理しようとします。これは基本的なリードスルーキャッシュに使う戦略です。データが利用可能なときには常に最新のデータが欲しいが、データがないよりは古いデータの方がいいというような API リクエストにも適しています。
    
      - **cacheFirst** リクエストがキャッシュエントリにマッチする場合は、それで応答します。そうでなければ、ネットワークからリソースを取得しようとします。ネットワークからのリクエストが成功した場合、キャッシュを更新します。このオプションは、変化しないリソースや他の更新メカニズムを持っているリソースに適しています。
    
      - **fastest** キャッシュとネットワークの両方からリソースを並行してリクエストします。どちらか先に返ってくる方で応答します。通常、キャッシュされたバージョンがあれば、それが返されます。一方で、このストラテジーはリソースがキャッシュされている場合でも、常にネットワークリクエストを行います。一方で、ネットワークリクエストが完了するとキャッシュが更新され、将来のキャッシュリードがより最新のものになるようにします。
    
      - **cacheOnly** キャッシュからのリクエストを解決するか、失敗します。このオプションは、モバイルでのバッテリー節約など、ネットワークリクエストを行わないことを保証する必要がある場合に適しています。
    
      - **networkOnly** ネットワークからURLをフェッチしようとすることでリクエストを処理します。フェッチに失敗した場合は、リクエストを失敗させます。基本的には、URL のルートを全く作成しないのと同じです。

  - **options**: 以下のプロパティを持つオブジェクト
    
      - **debug** \[Boolean\] 追加情報をブラウザのコンソールに記録するかどうかを決定します。
    
      - **networkTimeoutSeconds** \[Number\] `toolbox.networkFirst` 組み込みハンドラに適用されるタイムアウト。`networkTimeoutSeconds` が設定されている場合、その時間よりも長い時間を要するネットワークリクエストは、キャッシュされたレスポンスが存在する場合には自動的にキャッシュされたレスポンスにフォールバックします。`networkTimeoutSeconds` が設定されていない場合は、ブラウザのネイティブネットワークタイムアウトロジックが適用されます。
      - **cache** \[Object\]
        
          - **name** \[String\] レスポンス オブジェクトの保存に使用するキャッシュの名前。固有の名前を使用することで、キャッシュの最大サイズとエントリの寿命をカスタマイズすることができます。
          - **maxEntries** \[Number\] さまざまな組み込みハンドラを介してキャッシュされたエントリに対して、最も最近使用されたキャッシュの期限切れポリシーを適用します。これは、自然な制限のない動的なリソースセットのエントリを保存するためのキャッシュで使用できます。`cache.maxEntries` を例えば 10 に設定すると、11番目のエントリがキャッシュされた後、最も最近使用されたエントリは自動的に削除されます。キャッシュは `cache.maxEntries` エントリを超えて成長してはいけません。このオプションは `cache.name` が設定されている場合にのみ有効です。このオプションは単独で使用することも `cache.maxAgeSeconds` と組み合わせて使用することもできます。
          - **maxAgeSeconds** \[Number\] キャッシュエントリの寿命を秒単位で設定します。これは、自然な制限のない動的なリソースセットのエントリを保存することに特化したキャッシュで使用することができます。`cache.maxAgeSeconds` を例えば 86400 に設定すると、一日より古いエントリは自動的に削除されます。このオプションは `cache.name` が設定されている場合にのみ有効です。このオプションは単独でも、`cache.maxEntries` と組み合わせてでも使用することができます。

### サンプル: API コールに対するキャッシュされたレスポンスの数を制限する

上記の例で最近アクセスされた10件のイベントのみをキャッシュしたい場合は、次のようにしてキャッシュします。

```javascript
options: { cache: { name: ‘events’, maxEntries: 10 } }
```

`@sw-cache` コメントにセットします。次のようになります。

```javascript
Ext.define('App.model.Event', {
    extend: 'Ext.data.Model',
    fields: ['id', 'name', 'date'],

    proxy: {
        type: 'rest',
        // @sw-cache { urlPattern: "\\/api\\/events\\/\\d+", options: { cache: { name: 'events', maxEntries: 10 } } }
        url : '/events'
    }
});
```

> **Note:** `@sw-cache` のコメントを追加したり `app.json` に変更を加えたりした後は、`sencha app build` を実行して変更内容をテストする必要があります。サービスワーカーはアプリの `production` ビルドにのみ含まれます。これは、`development` ビルドでは各コードファイルを個別のリクエストとして読み込むため、`development` ビルドではアプリシェルに必要なすべてのアセットを事前にキャッシュできないからです。キャッシングの動作をテストするには、必ず `production` ビルドを使用してください。

> **Note:** ビルドされると、アプリの最初のリロード時にプログレッシブ マニフェストとキャッシングの変更が検出されます。ブラウザでアプリをリロードすると、サービスワーカーが非同期にフェッチされてインストールされるため、アプリの起動中に行われた API 呼び出しが最初にキャッシュされない場合があります。起動中に行われたAPIコールをキャッシュするには、ブラウザを2回リロードする必要がある場合があります。

## serviceWorker Config

コメントの `@sw-cache` に加えて、`app.json` の `progressive.serviceWorker` セクションに `runtimeCaching` プロパティを追加することで、サービスワーカーのキャッシュリクエストを設定することができます。上記で説明したすべてのオプションがサポートされています。 
例えば、最近閲覧した10件のイベントのデータだけでなく、今後のイベントのリストをキャッシュするには、store クラスと model クラスに `@sw-cache` コメントを追加する代わりに、以下を `app.json` に追加することができます。

マニフェストと `serviceWorker` の両方の config を持つ完全な例を以下に示します。

```json
"progressive": {

    "manifest": {

        "name": "My App",
        "short_name": "My App",

        "icons": [{
            "src": "resources/icon-small.png",
            "sizes": "96x96"
        }, {
            "src": "resources/icon-medium.png",
            "sizes": "192x192"
        }, {
            "src": "resources/icon-large.png",
            "sizes": "256x256"
        }],

        "theme_color": "#054059",
        "background_color": "#054059",
        "display": "standalone",
        "orientation": "portrait",
        "start_url": "/index.html"
    },

    "serviceWorker": {

        "runtimeCaching": [
            {
                "urlPattern": "\\/api\\/events"
            }, 
            {
                "urlPattern": "\\/api\\/events\\/\d+",
                "options": {
                    "cache": {
                        "name": "events",
                        "maxEntries": 10
                    }
                }
            }
        ]
    }
}
```

> **Note**: キャッシングの動作を指定するために `app.json` の `runtimeCaching` config と `@sw-cache` コメントの両方を使用することができますが、`@sw-cache` コメントが優先されます。このため、両方を混在させるのではなく、どちらか一方の方法をアプリに選択することをお勧めします。
