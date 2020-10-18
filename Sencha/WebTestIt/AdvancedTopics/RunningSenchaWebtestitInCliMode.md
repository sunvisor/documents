# CLI モードで Sencha WebTestIt を実行する

> オリジナル: [Running Sencha WebTestIt in CLI mode](https://docs.sencha.com/webtestit/guides/advanced-topics/running-sencha-webtestit-in-cli-mode.html)

継続的インテグレーション (CI) 環境などでプロジェクトのテストを実行するために、ユーザーインターフェース (UI) を使用せずに、コマンドラインインターフェース (CLI) モードで Sencha WebTestIt を起動することができます。

Sencha WebTestItをインストールしたら、以下の場所に実行ファイルがあります。

  - Windows: `%LOCALAPPDATA%\Programs\webtestit\Sencha WebTestIt.exe`
  - Mac: `/Applications/Sencha\ Webtestit.app/Contents/MacOS/Sencha\ Webtestit`
  - Linux: `webtestit`

CLIモードでSencha WebTestItプロジェクトを実行するには、CMD (Windows) またはターミナル (MacOS, Linux) を開き、次のように入力します。

`<Webtesit app path> run  [options]  <your project path>`

プロジェクトの run/open アクションについては、以下の「利用可能なコマンド」を参照してください。

### 利用可能なコマンド

  - `open <path>` プロジェクトを開く
  - `run [options] <path>` プロジェクトのすべてのテストを実行する \[options\]:
      - `--endpoints [endpoints]`  
        エンドポイントを名前でフィルタリング
      - `--endpoints-config [endpoints-config]`  
        他のエンドポイントの config ファイルのパス (絶対パスまたは相対パス)
      - `--test-file-patterns [test-file-patterns]`  
        testrun に含めるテストファイルのパターンを定義する ([glob パターン](https://en.wikipedia.org/wiki/Glob_\(programming)) をサポートします。プロジェクトディレクトリからの相対)
      - `--include-inactive-endpoints`  
        アクティブでないエンドポイントを含める
      - `--report-file-destination [report-path]`  
        代替レポートファイルの保存先 (絶対または相対)
      - `--report-file-name-pattern [file-name-pattern]`  
        ファイル名パターン
      - `--pdf-report`  
        PDFとしてレポートを直接印刷
  - `testrail-export [options] <path>` Export test results to TestRail \[options\]:
      - `--report-file-path [report-path]`  
        レポートファイルのパス - 指定されていない場合、最新のレポートが使用されます。
      - `--host [testrail-url]`  
        TestRail ホストのURL
      - `--testrail-username [username]`  
        TestRail ユーザー
      - `--testrail-password [password]`  
        TestRail のパスワードまたは API キー
      - `--project-id [project-id]`  
        TestRail プロジェクト ID
      - `--run-name [name]`  
        TestRail 実行名称

Sencha WebTestItの古いバージョンで作成されたプロジェクトをCLIモードで実行しようとしている場合があります。または、古いバージョンのアプリケーションを使用しながら、新しいバージョンのアプリケーションで作成されたプロジェクトを実行しようとしている場合もあります。この場合、次のような警告が表示されます。*"The version of Sencha WebTestIt is newer than that of your project. Please open your project in Sencha WebTestIt to migrate it to the newest version"* (Sencha WebTestItのバージョンがあなたのプロジェクトのバージョンよりも新しいです。Sencha WebTestItでプロジェクトを開いて最新バージョンに移行してください。)

メッセージ自体にもあるように、プロジェクトを移行するには、Sencha WebTestIt でプロジェクトを開くだけで、自動的に移行が行われます。

### エラーコード

実行後、Sencha WebTestItはエラーコードを返しますが、これはCI環境でビルドプロセスの失敗か成功かを判断するために使用できます。

  - 0 SUCCESS
  - 101 NO\_RUNNABLE\_PROJECT
  - 102 ROOT\_PACKAGE\_MISSING
  - 103 UNSUPPORTED\_PROJECT\_LANGUAGE
  - 301 FAILED\_TO\_READ\_ENDPOINT\_CONFIG
  - 302 MISSING\_ENDPOINT\_OBJECT\_IN\_PROVIDED\_FILE
  - 303 MISSING\_REQUIRED\_ENDPOINT\_FIELD
  - 304 WEBDRIVER\_PROBLEM
  - 305 NO\_VALID\_ENDPOINTS
  - 306 BAD\_JSON\_IN\_ENDPOINT\_CONFIG
  - 400 TEST\_FAILURES
  - 900 UNEXPECTED\_ERROR
