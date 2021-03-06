# Ext Direct 仕様

概要
--------

Ext Directは、プラットフォームと言語にとらわれないリモートプロシージャコール(RPC)プロトコルです。Ext Directは、Ext JSアプリケーションのクライアント側と仕様に準拠した任意のサーバープラットフォーム間のシームレスな通信を可能にします。Ext Directはステートレスで軽量で、APIディスカバリ、コールバッチ、サーバーからクライアントへのイベントなどの機能をサポートしています。

規約
-----------

このドキュメントのキーワード「MUST」、「MUST NOT」、「REQUIRED」、「SHALL」、「SHALL NOT」、「SHOULD」、「SHOULD NOT」、「RECOMMENDED」、「MAY」、「OPTIONAL」は、RFC 2119に記述されているように解釈されるべきである。

Ext Direct は部分的に JSON に基づいており (http://www.json.org または RFC 4627 を参照)、HTML フォームベースのファイルアップロード (RFC 1867, RFC 2388 ) を利用しています。

APIディスカバリー
-------------

サーバーは、API ディスカバリ機構を介してクライアントに API を公開することをサポートしている場合があります。API ディスカバリがサポートされている場合、サーバーはあらかじめ設定された URI で HTTP GET リクエストに応答し、ブラウザがこのドキュメントを JavaScript コードとして解釈するために正しいコンテンツタイプのドキュメントを返さなければなりません。

### API 宣言

サーバーAPI宣言は、後にクライアントアプリケーションのExt Direct初期化コードに渡すことができる変数に代入された入れ子になったオブジェクトのセットを作成する有効なJavaScriptコードでなければなりません。JavaScript以外の実装がAPI宣言をJSONとしてパースしようとする可能性があるため、コードはJSONオブジェクト構文のより厳しいルールにも準拠することをお勧めします。

APIの宣言コードの例です。

```js
    var Ext = Ext || {};
    Ext.REMOTING_API = {
        "id": "provider1",
        "url": "ext/direct/router",
        "type": "remoting",
        "actions": {
            "Album": [{
                "name": "getAll",
                "len": 0
            }, {
                "name": "add",
                "params": ["name", "artist"],
                "strict": false
            }, {
                "name": "delete",
                "len": 1
            }]
        }
    };
```

### API 宣言プロパティ

API 宣言の JavaScript オブジェクトには、以下の必須プロパティを含める必要があります。

- **`url`** - このAPIのサービスURI。
- **`type`** - Remoting API の場合は `remoting`、Polling API の場合は `polling` のいずれかでなければなりません。
- **`actions`** - 与えられた Remoting API で利用可能なすべてのアクションとメソッドをリストアップした JavaScript オブジェクト。

API 宣言には、以下のオプションのプロパティも含めることができます。

- `id`** - リモーティング API プロバイダの識別子。これは複数のAPIが使用されている場合に便利です。
- **`namespace`** - 指定した Remoting API の名前空間。
- **`timeout`** - この Remoting API の各メソッド呼び出しのタイムアウトとして使用するミリ秒数。

その他のプロパティはオプションです。

### アクションとメソッドの宣言

API宣言の `actions` プロパティ内の各アクションは、メソッドを表すオブジェクトの配列である。アクションはそれ自体がプロパティではありません。つまり、1つのアクションにはメソッドだけでなく他のアクションも含まれている可能性があります。

メソッド宣言は、以下のプロパティを持つ必要があります。

- **`name`** - このメソッドの名前は、アクション内で一意でなければなりません。

各メソッドは、アクション名とメソッド名をドット文字(.)で連結し、プレフィックスとしてオプションのNamespaceを付けることで完全に修飾されます。

    [Namespace.]Foo.Bar.baz

ここで、`Foo` は外側のアクション名、`Bar` は内側のアクション名、`baz` はメソッド名です。

#### メソッドの呼び出し規則

メソッドの宣言には、メソッドの呼び出し規則を記述する、以下の相互に排他的なプロパティのうちの**1つ**を持たなければなりません。

- **`len`** - 順序付きメソッドに必要な引数の数。 引数を受け付けないメソッドの場合、この数は 0 になることがあります。

- **`params`** - 名前付きメソッドでサポートされるパラメータの配列。 この配列は空の場合もあります。

- **`formHandler`** - JavaScript のブール値 `true` は、このメソッドがフォームの送信を受け入れることを示します。

Ordered メソッドは常にその呼び出し規約に従わなければなりません。Remoting Method プロキシ関数が Ordered Method に対して呼び出される場合、そのメソッドが要求するのと全く同じ順序で、 必要な引数の数を正確に供給しなければならない。必要な引数数よりも少ない数の引数が渡された場合、ルータは、実際のメソッドを呼び出さずに Exception を返すことを選択することができる。

名前付きメソッドは、メソッドが呼び出された際に、引数がどのようにチェックされ、サーバに渡されるかを制御するために **`strict`** プロパティを使用することができる。

- strict` が `true` に設定されている場合、`params` 配列にリストされた名前の引数のみがサーバーに送信され、それ以外の引数は破棄されます。これがデフォルトの動作です。
- strict` が `false` に設定されている場合、すべての引数がサーバに渡される。ルータは、すべての引数を実際のメソッドに渡すべきである。

ルータは、リストされたパラメータの一部が欠落している場合には例外を返し、実際のメソッ ドの呼び出しを省略することを選択することができる。params` Array が空で、`strict` プロパティが `false` に設定されている場合、ルータは引数チェックを行わず、すべての引数を呼び出したメ ソッドに渡さなければならない。

すべてのオプションのパラメータを持つ名前付きメソッドの例。

    "actions": {
        "TestAction": [{
            "name": "named_no_strict",
            "params": [],
            "strict": false
        }]
    }

#### コールメタデータ

メソッド宣言には、オプションの **`metadata`** プロパティを含めることができます。メソッド宣言に `metadata` プロパティが存在しない場合、クライアントはそのメソッドの呼び出しに対してコールメタデータをサーバーに送信してはいけません。

`metadata` プロパティが存在する場合、以下のプロパティを持つ JavaScript オブジェクトでなければなりません。

- **`len`** - 位置によるコールメタデータに必要な引数の数。この数は 0 より大きくする必要があります。
- **`params`** - 名前によるコールメタデータでサポートされる名前の配列。この配列は空でもよい。
- **`strict`** - コールメタデータ引数のチェック方法を制御するJavaScriptのブール値。

存在する場合、コールメタデータの引数は、その呼び出し規約に従わなければなりません。メタデータの呼び出し規約は、メインのメソッド引数の呼び出し規約とは異なる場合があります。例えば、順序付けられたメソッドは名前による呼び出しメタデータを受け入れたり、名前付きメソッドは位置による呼び出しメタデータを受け入れたりします。
引数チェックは、メイン・メソッド引数と同じ方法で実行されます。

コール・メタデータを受け入れるメソッド宣言の例をいくつか挙げます。

    "actions": {
        "TestAction": [{
            "name": "meta1",
            "len": 0,
            "metadata": {
                "len": 1
            }
        }, {
            "name": "meta2",
            "len": 1,
            "metadata": {
                "params": ["foo", "bar"],
                "strict": false
            }
        }, {
            "name": "meta3",
            "params": [],
            "strict": false,
            "metadata": {
                "len": 3
            }
        }, {
            "name": "meta4",
            "params": ["foo", "bar"],
            "metadata": {
                "params": ["baz", "qux"]
            }
        }]
    }

### Polling API 宣言

Polling API の宣言は任意です。サーバーが複数のイベントプロバイダを実装している場合、Polling API 宣言を Remoting API 宣言と一緒に同じ JavaScript ドキュメントに含めることが推奨されます。

1つのRemoting APIと1つのPolling APIによるAPI宣言の例。

```js
    var Ext = Ext || {};
    Ext.REMOTING_API = {
        "id": "provider1",
        "type": "remoting",
        "url": "ext/direct/router",
        "actions": {
            "User": [{
                "name": "read",
                "len": 1
            }, {
                "name": "create",
                "params": ["id", "username"]
            }]
        }
    };
    Ext.POLLING_API = {
        "id": "provider2",
        "type": "polling",
        "url": "ext/direct/events"
    };
```

Remoting 関数呼び出し
-----------------------

リモーティングコールはRequestオブジェクトまたは複数のRequestオブジェクトをサーバに送信することで表現されます。リクエストはJSONでエンコードされ、API宣言で`url`としてアドバタイズされたサービスURIへのHTTP POSTリクエストで生のペイロードとして送信されます。HTTP POSTには、1つ以上のリクエストを含む有効なJSONドキュメント以外のデータが存在してはいけません。POSTリクエストのHTTPヘッダの中には、値が「application/json」のContent-Typeヘッダが存在しなければなりません。クライアントは、リクエストドキュメントに UTF-8 文字エンコーディングを使用しなければなりません。

### リモーティング関数の呼び出しリクエスト

リクエストは、以下のメンバを持つオブジェクトです。

-   **`type`** – 文字列 "rpc" でなければなりません。
-   **`tid`** –このリクエストのトランザクション ID は、1 つのバッチ内のリクエスト間で一意の整数でなければなりません。
-   **`action`** – 呼び出されたメソッドが属するアクションを指定する必要があります。
-   **`method`** – 呼び出される Remoting Method を指定する必要があります。
-   **`data`** – 0 (ゼロ) パラメータを受け取るメソッドの場合は `null`、順序付きメソッドの場合は配列、名前付きメソッドの場合はオブジェクトのいずれかでなければなりません。
-   **`metadata`** - (オプション) クライアントから提供された場合、呼び出された Remoting メソッドが利用できるようにするためのメタ引数のセット。呼び出しにメタデータが関連付けられていない場合、このメンバはリクエストに存在してはいけません。

Orderedメソッドの典型的なJSONエンコードされたリクエストオブジェクトは次のようになります。

    {"type":"rpc","tid":1,"action":"Foo","method":"bar","data":[42,"baz"]}

Named Methodの典型的なJSONエンコードされたRequestオブジェクトは次のようになります。

    {"type":"rpc","tid":2,"action":"Foo","method":"qux","data":{"foo":"bar"}}

呼び出しメタデータを名前で指定した Ordered Method の典型的な JSON エンコードされた Request オブジェクトは、次のようになります。

    {"type":"rpc","tid":3,"action":"Foo","method":"fred","data":[0],
     "metadata":{"borgle":"throbbe"}}

その場合、リクエストは一意の `tid` メンバを持つリクエストオブジェクトの配列として送信されなければなりません。

    [
        {"type":"rpc","tid":3,"action":"Foo","method":"frob","data":["foo"]},
        {"type":"rpc","tid":4,"action":"Foo","method":"blerg","data":["qux"]}
    ]

サーバーはリクエストのバッチングをサポートしていなければならず、呼び出しの結果または例外を同じ順序で返すことを試みるべきである[SHOULD]。

### Remoting レスポンス

リモーティングコールへの応答には、リクエストごとに結果か例外のいずれかを含まなければなりません。レスポンスは JSON でエンコードされ、Content-Type ヘッダが「application/json」で文字エンコーディングが UTF-8 の生の HTTP レスポンスペイロードとしてクライアントに返されます。

リクエストがバッチとして送信された場合、サーバは、すべてのリクエストがルータで処理された後にのみ、 対応するレスポンスを返さなければならず、レスポンスは元のリクエストと同じ順序に従うべきである [SHOULD NOT]。各レスポンスに対して、元のリクエストの対応する `tid` メンバは、サーバによって変更されずに渡されなければなりません。

#### Remoting コールリザルト

リザルトは、以下のメンバを持つオブジェクトです。

-   **`type`** — は文字列 "rpc" でなければなりません。
-   **`tid`** — このレスポンスのトランザクションIDは、オリジナルのリクエストと同じでなければなりません。
-   **`action`** — 呼び出されたメソッドが属するアクションは、オリジナルのRequestと同じでなければなりません。
-   **`method`** — 呼び出された Remoting Method の名前は、元のリクエストと同じでなければなりません。
-   **`result`** — メソッドが返すデータはレスポンスオブジェクトに存在しなければなりませんが、データを返さないメソッドの場合は `null` にすることができます。

典型的な JSON エンコードされた Array of Response オブジェクトは次のようになります。

    [
        {"type":"rpc","tid":1,"action":"Foo","method":"bar","result":0},
        {"type":"rpc","tid":2,"action":"Foo","method":"baz","result":null}
    ]

#### Remoting 呼び出しの例外

例外は、以下のメンバを持つオブジェクトです。

-   **`type`** — は文字列 "exception" でなければなりません。
-   **`tid`** — このレスポンスのトランザクションIDは、オリジナルのリクエストと同じでなければなりません。
-   **`action`** — 呼び出されたメソッドが属するアクションは、オリジナルのRequestと同じでなければなりません。
-   **`method`** — 呼び出された Remoting Method の名前は、元のリクエストと同じでなければなりません。
-   **`message`** — エラーメッセージが表示されている必要があります。
-   **`where`** — (オプション) 例外が発生した場所の説明。スタックトレースや追加情報を含むことができます。

典型的な JSON エンコードされた Exception は次のようになります。

    {
        "type": "exception",
        "tid": 3,
        "action": "Foo",
        "method": "frob",
        "message": "Division by zero",
        "where": "... stack trace here ..."
    }

### Remoting フォーム送信

リモーティングフォームの呼び出しは、HTMLフォームをHTTP POSTリクエストで送信することで表現されます。フォームの内容は (HTML 4.01 仕様)に準拠している必要があります。サーバーは、下位互換性のために「application/x-www-form-urlencoded」と「multipart/form-data」の両方のコンテンツタイプをサポートしなければなりません。クライアントは、"multipart/form-data" エンコーディングのみを実装することを選択することができます。

送信されたフォームにつき、メソッドの呼び出しは 1 つだけでなければなりません。クライアントは Remoting API で宣言されているフォームハンドラの呼び出し規約を使用して、各メソッドにフォーム送信を使用しなければなりません。

#### Remoting フォームリクエスト

フォームには、以下のフィールドが含まれているものとします。

-   **`extType`** - 文字列 "rpc" である必要があります。
-   **`extTID`** - このリクエストのトランザクション ID は、1 つのバッチ内のリクエスト間で一意の整数でなければなりません。
-   **`extAction`** - 呼び出されたメソッドが属するアクション。 指定する必要があります。
-   **`extMethod`** - 呼び出される Remoting Method。指定する必要があります。
-   **`extUpload`** - 
文字列化されたブール値 (`"true"` または `"false"`) は、ファイルのアップロードがこのフォーム送信に添付されていることを示します。

フォームは `metadata` フィールドを含むことができます。

フォームは上記の必須フィールド以外にも他のフィールドを含むことができます。これらの追加フィールドは名前付き引数として呼び出されたメソッドに渡されなければなりません。

フォームがファイルをアップロードするために使われる場合、エンコーディングの型は "multipart/form-data "でなければなりません。

#### Remoting フォームレスポンス

サーバーは、4.2.1 節および 4.2.2.2 節で説明するように、有効な Response または Exception オブジェクトを含む JSON ドキュメントでフォーム送信に応答しなければなりません。ドキュメントのコンテンツタイプは「application/json」で、文字エンコーディングはUTF-8でなければなりません。

#### Remoting ファイルアップロードレスポンス

フォーム リクエストに 1 つ以上のファイル アップロードが含まれていた場合、サーバーは、ブラウザがこのドキュメントを HTML として解釈するために、正しいコンテンツ タイプを持つ有効な HTML ドキュメントを返さなければなりません。ドキュメントは UTF-8 文字エンコーディングでなければなりません。

HTML文書は、4.2.1項および4.2.2項で説明したように、有効なJSONエンコードされたResponseまたはExceptionをHTML `<textarea>`タグで囲んで含まなければなりません。

ファイルのアップロード応答の例は次のようになります。

    <!DOCTYPE html>
    <html>
        <head>
            <title>File upload response</title>
        </head>
        <body>
            <textarea>{JSON RESPONSE HERE}</textarea>
        </body>
    </html>

イベントポーリング
------------------

サーバーは、オプションのイベントポーリング機構を実装することができます。イベントポーリングは、サーバーに対して定期的にHTTP GETリクエストを行うことで実行されます。
サーバーごとに複数のイベントプロバイダがあるかもしれません。その場合、各イベントプロバイダは別個のサービスURIを持たなければなりません。

### イベントポーリングリクエスト

基本的な形でのイベント・ポーリング・リクエストは、ポーリングされたイベント・プロバイダのサービスURIへのHTTP GETリクエストです。サーバーは、各イベント・プロバイダに対してアクティブな Poll ハンドラ・メソッドのリストを維持し、ポーリング・リクエストが行われるたびに各 Poll ハンドラ・メソッドを呼び出さなければなりません。サーバーは、HTTP GET リクエスト URI を介して Poll ハンドラメソッドに引数を渡すことをサポートしているかもしれませんが、必須ではありません。

#### イベントポーリングレスポンス

イベントポーリングのレスポンスは、ブラウザがこのドキュメントを JSON として解釈するためには、正しいコンテンツタイプを持つ有効な JSON ドキュメントでなければなりません。ドキュメントはUTF-8文字エンコーディングを使用しなければなりません。

イベントポーリングレスポンスは、イベントオブジェクトの配列を含むべきです。指定されたリクエストに対してイベントが保留されていない場合は、この配列は空であってもかまいません。サーバーは、レスポンス配列に Exception オブジェクトを含めてはいけません。

#### イベントオブジェクト

イベントオブジェクトには、以下のプロパティを含める必要があります。

-   **`type`** - は文字列 "event" でなければなりません。
-   **`name`** - イベント名は文字列でなければなりません。
-   **`data`** - イベントデータは、有効なJSONデータであるべきです（SHOULD）。

典型的なイベントオブジェクトの例。

    {
        "type": "event",
        "name": "progressupdate",
        "data": {
            "processId": 42,
            "progress": 100
        }
    }

Terminology
-----------------------------------------------------------------------------------------------------

Ext Direct makes use of the following terms:

-   **Server** — クライアントがリモートで実行する機能を提供するコンピューティングシステム。
-   **Client** — サーバーの機能をリモートで呼び出し、結果や例外を受け取るコンピューティングシステム。
-   **Server API** — クライアントに公開されているサーバー機能の説明
-   **API Declaration** — サーバー API をコード化する JavaScript チャンクです。API 宣言は通常、サーバーによって動的に生成され、起動時にクライアントによって取得されます。
-   **Remoting API** — API 宣言の主な部分は、クライアントが利用できるサーバーアクションとメソッド、呼び出し規則、その他のパラメータを列挙しています。
-   **Polling API** — API 宣言のオプション部分は、サーバーイベントプロバイダの存在とその資格情報を宣言するために使用することができます。
-   **API Discovery URI** — クライアントが API 宣言を取得するために HTTP GET 要求を送信するサーバー上の事前に設定された URI。このURIはオプションで、特定のアプリケーションでダイナミックAPIディスカバリを使用しない場合は省略することができます。
-   **Service URI** — 
クライアントがルータにリモーティングリクエストをディスパッチするために使用するサーバ上 の事前設定された URI、またはイベントプロバイダにポーリングリクエストをディスパッチ するために使用する URI。各ルータやイベントプロバイダは、そのルータやイベントプロバイダに固有のサービス URI を公開しなければなりません。
-   **Namespace** — 対応する Remoting メソッドのスタブを持つネストされた Action オブジェクトが作成されるグローバルオブジェクト。API 宣言で定義されていない場合は、デフォルトの `window` オブジェクトが使われます。
-   **Action** — 
名前空間ユニット。アクションは入れ子にすることができ、グローバル名前空間に配置することもできます。アクションは、メソッドを公開するクラスと考えることができます。
-   **Remoting Method** — クライアントからリモートで呼び出すことができるサーバー関数。メソッドは常にアクションに属し、名前空間、アクション、メソッド名を組み合わせて完全に修飾されます: `[Namespace.]Action.Method`.
-   **Remoting Method proxy** — サーバー上にのみ存在する実際の Method の代わりに Ext Direct クライアント側のスタックによって作成されるスタブ JavaScript 関数。パラメータのシグネチャが Method の API 宣言に準拠したもので、それぞれの Method に対して別個のプロキシ関数が作成されます。
-   **Ordered Method** — 0個以上のパラメータを順番に、または位置ごとに(リストのように)受け付けるリモーティング・メソッド。
-   **Named Method** — 連想配列 (hash, Object, map) のように名前でパラメータを受け付ける Remoting メソッド。
-   **Form Handler Method** — フォームの送信を受け付けるリモーティングメソッドです。これは XMLHttpRequest Level 2 仕様に準拠していないブラウザでファイルをアップロードするために使用できるレガシーなメカニズムです。 フォームハンドラのサポートを実装することはサーバーにとってオプションであり、そのようなサポートが提供されていない場合、サーバーは、フォームハンドラの呼び出し規則を使用するメソッドを宣伝してはならない。
-   **Poll Handler** — 
定期的なイベントポーリングの結果としてクライアントに渡されるイベントのリストを返すために、イベントプロバイダによって呼び出されるサーバ関数です。
-   **Call Metadata** - メソッドに渡される可能性のある引数のオプションの追加セット。メタデータは主な引数に関連していますが、同じものではなく、別々に扱われます。一般的な使用例は、CRUD操作のためにテーブル名などのデータベース情報を渡すことです。
-   **Router** — サービス URI でリモーティング関数の呼び出し要求を受け取り、実際の関数を実行し、クライアントに返される結果や例外を収集するサーバーコンポーネント。
-   **Event Provider** — サービスURIで定期的にポーリング要求を受信し、このイベントプロバイダに登録されたポーリングハンドラ機能を実行し、ポーリングハンドラが返すイベントを収集し、クライアントにイベントの一覧を返すサーバコンポーネント。
-   **Request** — クライアントから要求されたリモーティングメソッドの呼び出しで、メソッドの呼び出し規約に準拠した引数を渡します。
-   **Response** — リクエストの完了に関する情報。結果または例外のいずれかである必要があります。
-   **Result** — 呼び出しが成功したか失敗したかに関わらず、リモーティング・メソッドが返すデータ。結果はオプションであり、メソッドがデータを返すことが想定されていない場合は省略することができます。
-   **Exception** — 致命的なエラー、アプリケーション・ロジック・エラー、またはアプリケーション・コード内のその他の回復不可能なイベント。
-   **Event** — Poll ハンドラによって生成され、クライアントに渡される通知。イベントは、アプリケーションのステータスの更新、進捗インジケータ、その他の予測可能な条件やイベントに便利です。
-   **Batching** - 各リクエストを別々に送信するのではなく、複数のリクエストを1つのパケットにまとめてサーバーに送信します。
