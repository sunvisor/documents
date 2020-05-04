# Upgrading Existing Applications to Symfony Flex

デフォルトでFlexが使われているSymfony 4においても、SymfonyのFlexの利用はオプションです。しかしながら、Flex はとても便利で生産性を向上させるので、既存のアプリケーションを Flex にアップグレードすることが強く推奨されます。

symfony Flexはアプリケーションが以下のディレクトリ構造を使うことを推奨しています。これはSymfony 4でデフォルトで使われているものと同じですが、[<span>いくつかのディレクトリをカスタマイズすることができます</span>](#flex-customize-paths)。

```
your-project/
├── assets/
├── bin/
│   └── console
├── config/
│   ├── bundles.php
│   ├── packages/
│   ├── routes.yaml
│   └── services.yaml
├── public/
│   └── index.php
├── src/
│   ├── ...
│   └── Kernel.php
├── templates/
├── tests/
├── translations/
├── var/
└── vendor/
```

これは `symfony/flex` 依存関係をアプリケーションにインストールするだけでは十分ではないことを意味します。ディレクトリ構造を上で示されたものにアップグレードしなければなりません。このアップグレードを行う自動ツールはありませんので、これらの手動のステップに従わなければなりません。

1.  プロジェクトの依存関係としてFlexをインストールします。

```bash
$ composer require symfony/flex
```

2.  プロジェクトの `composer.json` ファイルに `symfony/symfony` 依存関係がある場合、symfony 4 で利用できなくなった Symfony Standard Edition に依存したままです。まず、この依存関係を削除します。

```bash
$ composer remove symfony/symfony
```

ここで、プロジェクトの `composer.json` ファイルの `conflict` セクションに `symfony/symfony` パッケージを追加します [スケルトンプロジェクトの例](https://github.com/symfony/skeleton/blob/8e33fe617629f283a12bbe0a6578bd6e6af417af/composer.json#L44-L46)。これで再びインストールされることはありません。

```diff
{
    "require": {
        "symfony/flex": "^1.0",
+     },
+     "conflict": {
+         "symfony/symfony": "*"
    }
}
```

プロジェクトに必要なすべてのsymfonyの依存関係を `composer.json` に追加しなければなりません。手っ取り早い方法は、以前の `symfony/symfony` 依存関係に含まれていたすべてのコンポーネントを追加して、後で本当に必要のないものを削除することです。

```bash
$ composer require annotations asset orm-pack twig \
  logger mailer form security translation validator
$ composer require --dev dotenv maker-bundle orm-fixtures profiler
```

3.  3. プロジェクトの `composer.json` ファイルに `symfony/symfony` 依存関係が含まれていない場合、Flex が要求するように、すでに依存関係を明示的に定義しています。すべての依存関係を再インストールして、Flex が `config/`, に設定ファイルを生成するように強制します。これがアップグレードプロセスの最も面倒な部分です。

```bash
$ rm -rf vendor/*
$ composer install
```

4.  前の手順のどれに従ったかに関わらず。この時点で、`config/`に新しいコンフィグファイルがたくさんあります。これらのファイルには symfony によって定義されたデフォルトの設定が含まれているので、`app/config/` の元のファイルをチェックして新しいファイルに必要な変更を加えなければなりません。Flex config は設定ファイルにサフィックスを使わないので、古い `app/config/config_dev.yml` は `config/packages/dev/*.yaml` などに移動します。

5.  最も重要な設定ファイルは `app/config/services.yml` で、現在は `config/services.yaml` にあります。[デフォルトのservices.yamlファイル](https://github.com/symfony/recipes/blob/master/symfony/framework-bundle/3.3/config/services.yaml)の内容をコピーして、独自のサービス設定を追加してください。後でこのファイルを再検討することができます。なぜならSymfonyの[*autowiring機能*](./service_container/3.3-di-changes.html)のおかげでほとんどのサービス設定を削除することができるからです。

Info
: 以前の設定ファイルに `Kernel::configureContainer()` メソッドや `Kernel::configureRoutes()` メソッドによって既にロードされたリソースを指す `imports` 宣言がないことを確認してください。


6.  以下のように `app/` の残りの内容を移動します（その後、`app/` ディレクトリを削除します）。

- `app/Resources/views/` -\> `templates/`
- `app/Resources/translations/` -\> `translations/`
- `app/Resources/<BundleName>/views/` -\> `templates/bundles/<BundleName>/`
- rest of `app/Resources/` files -\> `src/Resources/`

7.  バンドル固有のファイル(`AppBundle.php` や `DependencyInjection/` など)を除き、元の PHP ソースコードを `src/AppBundle/*` から `src/` に移動します。

ファイルの移動に加えて、`composer.json` ファイルの `autoload` と `autoload-dev` の値を [この例で示す](https://github.com/symfony/skeleton/blob/8e33fe617629f283a12bbe0a6578bd6e6af417af/composer.json#L24-L33) のように更新して、アプリケーションの名前空間として `App\` と `App\Tests` を使用するようにします (高度な IDE は自動的にこれを行うことができます)。

複数のバンドルを使ってコードを整理している場合は、`src/`に再編成する必要があります。例えば、`src/UserBundle/Controller/DefaultController.php` と `src/ProductBundle/Controller/DefaultController.php` があれば、`src/Controller/UserController.php` と `src/Controller/ProductController.php` に移動させることができる。

8.  8. 画像やコンパイルされたCSS/JSファイルなどのパブリックアセットを `src/AppBundle/Resources/public/` から `public/` (例: `public/images/`) に移動します。

9.  9. アセットのソース(SCSSファイルなど)を`assets/`に移動し、[*Webpack Encore*](./frontend.html)を使用して管理・コンパイルします。

10. 環境変数 `SYMFONY_DEBUG` と `SYMFONY_ENV` を `APP_DEBUG` と `APP_ENV` に置き換えました。これらの値を新しい変数にコピーしてから、前の変数を削除してください。

11. Create the new `public/index.php` front controller [copying Symfony's index.php source](https://github.com/symfony/recipes/blob/master/symfony/framework-bundle/3.4/public/index.php) and, if you made any customization in your `web/app.php` and `web/app_dev.php` files, copy those changes into the new file. You can now remove the old `web/` dir.

12. Update the `bin/console` script [copying Symfony's bin/console source](https://github.com/symfony/recipes/blob/master/symfony/console/3.4/bin/console) and changing anything according to your original console script.

13. Remove `src/AppBundle/`.

14. Move the original source code from `src/{App,...}Bundle/` to `src/` and update the namespaces of every PHP file to be `App\...` (advanced IDEs can do this automatically).

15. Remove the `bin/symfony_requirements` script and if you need a replacement for it, use the new [Symfony Requirements Checker](https://github.com/symfony/requirements-checker).

16. Update the `.gitignore` file to replace the existing `var/logs/` entry by `var/log/`, which is the new name for the log directory.

<div id="customizing-flex-paths" class="section">

<span id="flex-customize-paths"></span>

## Customizing Flex Paths[¶](#customizing-flex-paths "Permalink to this headline")

The Flex recipes make a few assumptions about your project's directory structure. Some of these assumptions can be customized by adding a key under the `extra` section of your `composer.json` file. For example, to tell Flex to copy any PHP classes into `src/App` instead of `src`:

<div class="literal-block notranslate">

<div class="highlight-json">

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr class="odd">
<td><div class="linenodiv">
<pre><code>1
2
3
4
5
6
7</code></pre>
</div></td>
<td><div class="highlight">
<pre><code>{
    &quot;...&quot;: &quot;...&quot;,

    &quot;extra&quot;: {
        &quot;src-dir&quot;: &quot;src/App&quot;
    }
}</code></pre>
</div></td>
</tr>
</tbody>
</table>

</div>

</div>

The configurable paths are:

  - `bin-dir`: defaults to `bin/`
  - `config-dir`: defaults to `config/`
  - `src-dir` defaults to `src/`
  - `var-dir` defaults to `var/`
  - `public-dir` defaults to `public/`

If you customize these paths, some files copied from a recipe still may contain references to the original path. In other words: you may need to update some things manually after a recipe is installed.

</div>

</div>

</div>

<div class="m-t-30">

This work, including the code samples, is licensed under a [Creative Commons BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) license.

</div>

</div>

