マイクロローダー
===============

"Microloader "は、JavaScriptとCSSのためのSenchaのデータ駆動型ダイナミックローダーの名前です。Microloader は、生成されたアプリケーションの一部として Sencha Cmd によって提供されます。このガイドでは、マイクロローダが何をするのか、そして特定の要件に合わせてマイクロローダを調整する方法をしっかりと理解していただけると思います。

Ext 6 で使用されている Microloader は、現在 Ext JS 5 や Sencha Touch で使用されているものとは異なる実装であることを明確にすることが重要です。Ext JS 6 のマイクロローダは、Sencha Touch のマイクロローダが提供していたものと同じ機能をすべて提供しますが、`"app.json"`ファイルで利用可能ないくつかの改良と新しいコントロールがあり、それについてはこのドキュメントで後述します。すべてのマイクロローダの実装は Sencha Cmd によって提供され、マイクロローダへのアップグレードは "sencha app upgrade" プロセスによって適用されます。

マニフェスト
------------

### app.json

このファイルは、アプリケーションの詳細を指定する場所です。この設定ファイルは、最初にSencha Cmdのビルドプロセスによって消費されます。Sencha Cmdは `"app.json"` の内容を変換し、結果のマニフェストをマイクロローダーに渡して実行時に使用します。最後に、Ext JS自身もランタイムマニフェストを参照して設定オプションを確認します。

#### Ext.manifest

アプリケーションを起動すると、`"app.json"` の処理内容が "Ext.manifest" として読み込まれます。Ext JS 6では、指定したExt.manifestプロパティを使用して、Compatibility Layerを有効にするなどの処理を行います。このオブジェクトを読み込ませるためにマイクロローダがサポートしている様々なオプションがありますが、これについては後ほど説明します。

#### スクリプトタグ

マイクロローダを使用するには、あなたのページに以下のscriptタグが必要です。

    <script id="microloader" data-app="12345" type="text/javascript" src="bootstrap.js"></script>

デフォルトでは、このスクリプトタグはビルドプロセスによって置き換えられますが、開発中にアプリケーションをロードするために使用されます。この *data-app* atribute は、アプリのスキャフォールド中に生成されているはずです。これは、データの衝突を防ぐためにローカルストレージで使用される一意のIDです。

Defaults and Customization
デフォルトとカスタマイズ
-----------------------

Sencha Cmdによって生成された `"app.json"` ファイルには、調整したい多くのプロパティが含まれています。これらのプロパティは、それぞれが何を制御するかを説明するためにインラインでドキュメント化されています。

プロジェクトをアップグレードしている場合、`"app.json"`にはすべてのオプションが含まれていない可能性があります。アップグレードを実行した後、不足しているプロパティのデフォルト値を ".sencha/app/app.defaults.json" に見つけることができます。

以下は、最も頻繁に注目されるプロパティです。

### indexHtmlPath

これはアプリケーションのHTMLドキュメントへのパスです（`"app.json"`ファイルからの相対パス）。デフォルトでは、このプロパティは "index.html "に設定されています。サーバーがPHP、ASP、JSP、その他の技術を使用している場合は、この値を以下のように適切なファイルを指すように変更することができます。

    "indexHtmlPath": "../page.jsp"

この設定を変更した場合、「出力」オプションも変更する必要があるでしょう（以下を参照）。

### framework

フレームワークの名前; "ext" または "touch"。例えば

    "framework": "ext"

### theme

Ext JSアプリケーションの場合、これはテーマパッケージの名前です。例えば、以下のようになります。

    "theme": "ext-theme-crisp"

### toolkit

Ext JS 6 アプリケーションの場合、これはツールキット パッケージの名前です。例えば、以下のようになります。

    "toolkit": "classic"

### js

ロードする JavaScript コードファイルの説明の配列。デフォルトでは、Ext JS 6 アプリケーションはこのようなものを持ちます。

    "js": [{
        "path": "app.js",
        "bundle": true
    }]

このエントリは、アプリケーションのエントリポイントを指定します。"bundle" フラグは、"sencha app build" を実行したときに、このエントリをビルドの連結出力で置き換えることを示しています。この配列には、以下のように他のファイルを追加することができます。

    "js": [{
        "path": "library.js",
        "includeInBundle": true
    },{
        "path": "app.js",
        "bundle": true
    }]

余分なファイルを追加する場合、ビルドプロセスがそれらをどのように扱うかを示すオプションのプロパティがいくつかあります。例えば、上記のケースでは、"library.js "は連結されたビルド出力に含まれ、ランタイムマニフェストから削除されます。

代わりに "includeInBundle "を削除した場合、"library.js "はアプリフォルダにあると見なされ、ビルド出力フォルダにコピーされます。このエントリはマニフェストに残り、マイクロローダによって個別にロードされます。

マイクロローダがロードする（ビルド プロセスでコピーされない）エントリを単純に渡すには、以下のように "remote" プロパティを追加します。

    "js": [{
        "path": "library.js",
        "remote": true
    },{
        "path": "app.js",
        "bundle": true
    }]

この配列にエントリを追加することもできますが、ほとんどの依存関係はコードの中の "require" ステートメントや `"app.json"` (パッケージを要求するための) に記述されます。

**注**. Sencha Touchの場合、通常は "js "配列に "sencha-touch-debug.js "が含まれています。Ext JS 6では "framework "の設定で十分なので、この項目は必要ありません。

### css

CSSアセットの扱いはJavaScriptとは少し異なります。これは、標準のSencha Cmdアプリケーションでは、CSSは `.scss` ソースからコンパイルされるからです。css "プロパティの初期内容は以下のようになります。

    "css": [{
        "path": "boostrap.css",
        "bootstrap": true
    }],

このCSSファイルは、`"sass"`フォルダのビルドをインポートするシンプルなスタブです。"bootstrap" フラグは、このエントリが開発に使用されるが、ビルド出力から削除されるべきであることを示しています。ビルドの場合、コンパイルされたCSSファイルは、生成されたマニフェストの "css "配列に追加されます。

最初は「bootstrap.css」がフレームワークからテーマをインポートして、こんな感じになります。

    @import "..../ext-theme-neptune/build/ext-theme-neptune-all.css";

アプリをビルドすると、このファイルは最近ビルドされたCSSファイルを指すように更新されます。例えば、"sencha app build "を実行すると、本番用のCSSファイルは "bootstrap.css "によって以下のようにインポートされます。

    @import "..../build/..../app-all.css";

css "エントリは "remote "プロパティもサポートしています。remote が設定されていない場合、これらのエントリは "js" エントリと同じように動作し、ビルド出力フォルダにコピーされます。

### requires

この配列は、必要なパッケージの名前を保持します。Sencha Cmdがこのリストを処理すると、不足しているパッケージは自動的にダウンロードされ、ワークスペースに抽出されます。パッケージは `"package.json"` の中に `requires` を含むことができます。これらも必要に応じてダウンロードされ、抽出されます。

これらのパッケージ名には、バージョン要件を含めることもできます。バージョン番号の指定については、[Sencha Cmd Packages](./cmd_packages/cmd_packages.html)を参照してください。

### output

output" オブジェクトは、ビルド出力がどこでどのように生成されるかを制御する機能を提供します。このオブジェクトはビルド出力の多くの側面を制御することができます。上記の `indexHtmlPath` の使い方では、ページのソースが `".../page.jsp"` であることを Sencha Cmd に伝えました。この処理を完了させるために、`output`を使ってSencha Cmdにビルドされたページが、ビルドされたJavaScriptとCSSからの相対的な位置にあることを伝えます。ソースツリーと同じ相対配置を維持するために、`"app.json"`にこれを追加します。

    "output": {
        "page": {
            "path": "../page.jsp",
            "enable": false
        },
        appCache: {
            "enable": false
        },
        "manifest": {
            "name": "bootstrap.js"
        }
    }

今回は、"enable "を追加してfalseにしています。この組み合わせは、Sencha Cmdに最終ページがどこにあるかを伝えますが、ソースをコピーしてページを生成しないようにもします(indexHtmlPageで指定されているように)。

ページを生成しないので、Microloaderスクリプトタグには同じ "src "値の "bootstrap.js "が含まれます。上の "manifest "オプションは、同じ名前を使ってビルドされたMicroloaderを生成するようにSencha Cmdに指示します。これは、JSPやASPのようなサーバーサイドのテンプレート環境ではもちろん、その他の環境でもよくあることです。

### Application Cache

アプリケーションキャッシュは、オフラインアクセスのためにブラウザが保存すべきアセットを決定するために使用されるマニフェストです。これを有効にするには、出力オブジェクト内の `appCache` プロパティの `enable` フラグをトグルするだけです。例えば、以下のようになります。

    "output": {
        "page": "index.html",
        "appCache": {
            "enable": true
        }
    }

ここに表示される `appCache` プロパティは、ビルド・プロセスがアプリケーション・キャッシュ・マニフェスト・ファイルを生成するかどうかを決定するために使用されます。これを true に設定すると、マニフェストは app.json のトップレベルの `appCache` コンフィグ・オブジェクトから生成されます。マニフェストは以下のようになります。

    "appCache": {
        "cache": [
            "index.html"
        ],
        "network": [
            "*"
        ],
        "fallback": []
    }

### Local Storage Cache

ローカルストレージキャッシングシステムは、ブラウザに内蔵されたアプリケーションキャッシュとは別のオフラインキャッシングシステムです。資産は一意のキーを介してローカルストレージに保存され、起動時にはリモートロードを試みる前に最初にこれらの資産が要求されます。これにより、インターネットに接続していなくても、アプリケーションを非常に高速にロードすることができます。このキャッシュはまた、アセットのデルタパッチを可能にします。つまり、アセットの変更されたビット（CSSやJS）のみがネットワーク経由でロードされます。これにより、アセットの変更されたビットのみがネットワーク経由で読み込まれ、ファイルはローカルストレージでパッチされてユーザーに配信されます。

このキャッシュはアセットごとに `update` プロパティで有効になります。これは `"full"` または `"delta"` のいずれかに設定することができます。以下は、ローカルストレージキャッシュを有効にした JS と CSS アセットの例です。

    // app.js は更新時にデルタパッチが適用されます。
    "js": [
        {
            "path": "app.js",
            "bundle": true,
            "update": "delta"
        }
    ],

    // アップデートでapp.cssが完全にダウンロードされます。
    "css": [
        {
            "path": "app.css",
            "update": "full"
        }
    ]

アセットでローカルストレージキャッシュを有効にしたら、グローバルに `cache` 設定 `"app.json"` を有効にしなければなりません。通常、開発中のビルドではこれを false に設定し、運用中のビルドではこれを true に設定します。

    "cache": {
        "enable": false
    }

deltas のパスと生成を設定することもできます。`deltas` プロパティが真の値に設定されている場合、Local Storage Caching を使用している全てのアセットはビルドの `"deltas"` フォルダに detas が生成されます。`deltas` が文字列に設定されている場合は、その値がすべてのパッチファイルが保存されるフォルダ名として使用されます。delta トグルは `enable` が false に設定されている場合は効果がありません。

    "cache": {
        "enable": true,
        "deltas": true
    }

アプリケーションキャッシュとローカルストレージキャッシュの両方とも、オフラインアクセス用のファイルを即座にロードします。このため、更新されたファイルはユーザーの現在のアプリケーション体験にはロードされません。マイクロローダが新しいアプリケーションキャッシュまたはローカルストレージファイルを検出してロードすると、グローバルイベントがディスパッチされ、最新のアップデートのためにアプリケーションを再ロードするようにユーザーに促すことができます。以下のコードをアプリケーションに追加することで、このイベントをリッスンすることができます。

    Ext.application({
    name: 'MyApp',
    mainView: 'MyMainView',
    onAppUpdate: function () {
        Ext.Msg.confirm('Application Update', 'This application has an update, reload?',
            function (choice) {
                if (choice === 'yes') {
                    window.location.reload();
                }
            }
        );
    }

### More

ここまで見てきたプロパティの多くはビルドプロセスへの指示であり、他のプロパティはマイクロローダーによって処理されます。前述したように、マニフェストはまた、Ext JS の機能の一部を設定する手段としても理解されています。これらのプロパティや他のプロパティの詳細については、`"app.json"`のコメントを参照してください。

### Build Profiles

アプリケーションに複数のバリエーションがある場合、`"app.json"`に "builds "オブジェクトを追加して、以下のようなビルドを記述することができます(Kitchen Sinkから抜粋)。

    "builds": {
        "classic": {
            "theme": "ext-theme-classic"
        },
        "gray": {
            "theme": "ext-theme-gray"
        },
        "access": {
            "theme": "ext-theme-access"
        },
        "crisp": {
            "theme": "ext-theme-crisp"
        },
        "neptune": {
            "theme": "ext-theme-neptune"
        },
        "neptune-touch": {
            "theme": "ext-theme-neptune-touch"
        }
    }

ビルド」の各キーは「ビルドプロファイル」と呼ばれます。値はプロパティのオーバーライドのセットで、以下に説明するように出力マニフェストを生成するために `"app.json"` のベースコンテンツに適用されます。この場合、各ビルドプロファイルに対して変更されるのは "theme "プロパティだけです。

さらに、アプリケーションのビルドで共通のバリエーションである他の2つのオプションのプロパティがあります。"locales" と "themes" です。これらは、最終的なビルドプロファイルを生成するプロセスを自動化するために使用されます。例えば、Kitchen Sinkは "locales "を使用しています。

    "locales": [
        "en",
        "he"
    ],

locales」や「themes」が与えられると、それぞれの値が「builds」の各エントリと結合され、最終的なマニフェストが生成されます。この場合、"neptune-en"、"neptune-he"、"crisp-en "などが最終的なビルドプロファイル名となります。

マニフェストの生成
------------------

前述のように、`"app.json"` は、実行時に "Ext.manifest" として表示される前に、ビルドプロセスの間に変換を受けます。このプロセスは、[Ext.merge](.../.../.../extjs/7.2.0/api/Ext.html#merge)のようにオブジェクトをマージすることで構成されていますが、ひねりがあります: 2つの配列が "マージ "されるとき、それらは連結されます。

この変換の最初のステップは、希望するビルド "環境 "の設定をマージすることです (例えば、"production" や "testing" など)。これに続いて、ビルドプロファイルが使用されている場合は、その内容がマージされます。ルートまたはビルドプロファイルのどちらかが "ツールキット"（"クラシック "または "モダン"）を指定している場合は、そのオブジェクトのプロパティがマージされます。最後に、パッケージャー ("cordova" や "phonegap") が設定されている場合は、そのプロパティがマージされます。

擬似コードでは次のようになります。

    var manifest = load('app.json');

    // These would come from the "sencha app build" command:
    var environment = 'production';
    var buildProfile = 'native';

    mergeConcat(manifest, manifest[environment]);
    if (buildProfile) {
        mergeConcat(manifest, manifest.builds[buildProfile]);
    }
    if (manifest.toolkit) {
        mergeConcat(manifest, manifest[manifest.toolkit]);
    }    
    if (manifest.packager) {
        mergeConcat(manifest, manifest[manifest.packager]);
    }

パッケージ
----------

マニフェストを作成する最後のステップは、必要なパッケージを追加することです。

必須パッケージがその package.json で "js" や "css" エントリを指定すると、これらはパッケージの依存関係順に連結されて、すでに生成された配列の前に置かれます。これにより、パッケージ自身がそのような依存関係を扱うことができるようになります。

js" と "css" エントリに加えて、各必須パッケージの package.json ファイルの内容が少しクリーンアップされ、パッケージ名でキーを設定した最終マニフェストの "packages" オブジェクトに追加されます。もし `"app.json"` が既に "packages" オブジェクトを含んでいる場合、そのオブジェクトは対応する package.json ファイルの内容とマージされます。`"app.json"` が優先されるのは、`"app.json"` のプロパティがパッケージへの設定オプションとして通過できるようにするためです (後述)。

擬似コードでは、`"app.json"` と package.json の内容は以下のようにマージされます。
The final step in producing the manifest is to add any required packages.

    var manifest;  // from above

    manifest.packages = manifest.packages || {};

    var js = [], css = [];

    // Expand required packages and sort in dependency order
    expandAndSort(manifest.requires).forEach(function (name) {
        var pkg = load('packages/' + name + '/package.json');

        js = js.concat(pkg.js);
        css = css.concat(pkg.css);
        manifest.packages[name] = merge(pkg, manifest.packages[name]);
    });

    manifest.css.splice(0, 0, css);

    var k = isExtJS ? 0 : manifest.js.indexOf('sencha-touch??.js');
    manifest.js.splice(k, 0, js);

これにより、以下のようなExt.manifestが生成されます。

    {
        "name": "MyApp",
        "packages": {
            "ext": {
                "type": "framework",
                "version": "5.0.1.1255"
            },
            "ext-theme-neptune": {
                "type": "theme",
                "version": "5.0.1.1255"
            },
            ...
        },
        "theme": "ext-theme-neptune",
        "js": [{
            "path": "app.js"
        }],
        "css": [{
            "path": "app.css"
        }],
    }

このマージの結果、"foo" パッケージは package.json でいくつかのグローバルオプション (例えば "bar" など) を提供し、デフォルト値を設定できるようになりました。foo パッケージを使うアプリケーションであれば、このオプションを `"app.json"` のように設定することができます。

    "packages": {
        "foo": {
            "bar": 42
        }
    }

パッケージはこの値を次のように取得します。

    console.log('bar: ' + Ext.manifest.packages.foo.bar);

### ロード順

開発中にアプリケーションをロードする場合とビルドからアプリケーションをロードする場合の大きな違いの 1 つは、個々のファイルがブラウザに表示される順序です。以前のリリースでは、app.js ファイルはブラウザによって処理されるほぼ最初のファイルでした。要件が記述されると、より多くのファイルが読み込まれ、その要件が発見されるなどしていました。

しかし、ビルドではこの順序が逆になります。app.js ファイルは、ほとんどの場合、ビルド出力の最後のファイルになります。これは、開発ではコードが動作していても、本番では失敗するような状況に陥りやすく、明らかに望ましくない結果となります。

Ext JS 5 では、ビルドに使用されるロード順序がマニフェストに追加され、同じ順序でファイルをロードするために使用されます。これによりマニフェストはかなり大きくなりますが、開発中にのみ使用されます。

マニフェストのロード
--------------------

マイクロローダは、`"app.json"` に記述されている通りにアプリケーションをロードし、Ext.manifest で渡されます。これを行うには、マイクロローダは最初にマニフェストをロードしなければなりません。これを行うには、3つの基本的な方法があります。

### マニフェストの埋め込み

一般的なアプリケーションでは、テーマ、ロケール、フレームワークの選択が1つで、その結果、1つのマニフェストが作成されます。このマニフェストは、ダウンロード時間を最適化するために、出力マークアップファイルに配置することができます。

このオプションを有効にするには、`"app.json"`に以下を追加します。

    "output": {
        "manifest": {
            "embed": true
        }
    }

この設定は、マークアップ ファイルにマニフェストとマイクロローダを埋め込みます。

### 名前付きマニフェスト

ビルドプロファイルを使用している場合、マニフェストの埋め込みはオプションではありません。その代わりに、マイクロローダはロード時に名前を指定してマニフェストを要求することができます。デフォルトでは、生成された `"app.json"` ファイルはマークアップファイルと同じフォルダに配置され、これがデフォルトのマニフェスト名になります。別の名前を指定するには、次のようにします。

    <script type="text/javascript">
        var Ext = Ext || {};
        Ext.manifest = 'foo';  // loads "./foo.json" relative to your page
    </script>

このアプローチは、ビルドプロファイルをハードコーディングするのではなく、名前を生成することでサーバー側でビルドプロファイルを管理するのに便利な方法です。

### 動的マニフェスト

クライアント側でビルドプロファイルを選択したい場合もあるでしょう。これを簡単にするために、マイクロローダは "Ext.beforeLoad" というフックメソッドを定義しています。このメソッドを以下のように定義すると、Microloader が処理する前に "Ext.manifest" の名前や内容を制御しながら、プラットフォーム検出を活用することができます。

    <script type="text/javascript">
        var Ext = Ext || {};
        Ext.beforeLoad = function (tags) {
            var theme = location.href.match(/theme=([\w-]+)/),
                locale = location.href.match(/locale=([\w-]+)/);

            theme  = (theme && theme[1]) || 'crisp';
            locale = (locale && locale[1]) || 'en';

            Ext.manifest = theme + "-" + locale;
        };
    </script>

上記はExt JS 5 Kitchen Sinkの例から引用しています。この例は、いくつかのテーマと2つのロケール(英語の "en "とヘブライ語の "he")でビルドされています。このメソッドを提供することで、ビルドプロファイル名が "crisp-en" のような値に変更され、マイクロローダに `"app.json"` の代わりに "crisp-en.json" マニフェストをロードするように指示します。

ビルドプロファイルの選択プロセスは、アプリケーションが必要とするものであれば何でも構いません。サーバサイドのフレームワークは、ページがレンダリングされるときにこの選択を行うことを選択するかもしれません。これはクッキーやその他の個人データに基づいている可能性があります。上記のケースでは、純粋にURLに基づいています。

#### プラットフォーム検出 / レスポンシブ

多くの場合、デバイスやブラウザが重要な選択基準となるため、マイクロローダは "tags "と呼ばれるオブジェクトをbeforeLoadに渡します。このオブジェクトには、検出した様々なプラットフォームタグがプロパティとして含まれています。すべてのタグのセットは以下の通りです。

-   phone
-   tablet
-   desktop
-   touch
-   ios
-   android
-   blackberry
-   safari
-   chrome
-   ie10
-   windows
-   tizen
-   firefox

beforeLoadメソッドは、これらのタグを使用したり、変更したりすることができます。このオブジェクトはその後、[platformConfig](.../.../.../extjs/7.2.0/api/Ext.Class.html#cfg-platformConfig)によって設定されたランタイム設定値と同様に、アセット(マニフェストに記述されているjsやcssのアイテム)のフィルタリングを制御するために使用されます。このフックを使って、これらのフィルターが何にマッチするかを制御することができます。しかし、タグを変更することは、主にアプリケーションに意味のある新しいタグを追加することを目的としています。カスタムタグの前には "ux-" を付けてください。Senchaが提供するタグは、このプレフィックスでは始まりません。

#### マニフェストで指定されるタグ

マニフェストは、利用可能なタグを設定するための "tags" プロパティも提供します。このプロパティは、文字列の配列である

    "tags": ["ios", "phone", "fashion"]

またはタグ名をキーにしたオブジェクトリテラルとして、その値がそのタグの値となります。

    "tags": {
        "ios": true,
        "phone": true,
        "desktop": false,
        "fashion": true
    }

提供された場合、このタグは自動検出された値よりも優先されます。これらの値はマニフェストによって供給されるので、"beforeLoad "メソッドでは利用できず、タグ名の衝突が発生した場合にそのメソッドによって行われたタグへの更新はすべて上書きされます。
