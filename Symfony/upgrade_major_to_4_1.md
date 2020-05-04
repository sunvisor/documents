# Upgrading a Major Version (e.g. 3.4.0 to 4.1.0)

2年ごとに、symfonyは新しいメジャーバージョンのリリースをリリースします(最初の番号は変わります)。これらのリリースは後方互換性を壊すことが許されているので、アップグレードするには最もトリッキーです。しかしながら、symfonyはこのアップグレードプロセスを可能な限りスムーズにします。

これはメジャーリリースが実際にリリースされる前にコードのほとんどをアップデートできることを意味します。これはコードを未来互換性のあるものにすることと呼ばれます。

メジャーバージョンをアップグレードするにはいくつかのステップがあります。

1.  [<span>コードの非推奨化を防ぐ</span>](#upgrade-major-symfony-deprecations);
2.  [<span>Composer経由で新メジャー版にアップデート</span>](#upgrade-major-symfony-composer);
3.  [<span>新しいバージョンで動作するようにコードを更新する</span>](#upgrade-major-symfony-after).

## 1\) コードの非推奨化を防ぐ

メジャーリリースのライフサイクルの間に、新機能が追加されたり、メソッドのシグネチャや公開APIの使用法が変更されたりします。しかし、[*マイナーバージョン*](upgrade_minor.html)には、後方互換性のない変更が含まれてはいけません。これを達成するために、"古い "コード(関数やクラスなど)はまだ動作しますが、*deprecated*としてマークされており、将来的に削除されたり変更されたりすることを示し、使用を停止すべきであることを示しています。

メジャーバージョンがリリースされると (例: 4.1.0)、非推奨の機能や機能はすべて削除されます。ですから、メジャーの前の最後のバージョン (例: 3.4.4.\*) で非推奨の機能を使わないようにコードを更新している限り、問題なくアップグレードできるはずです。

これを助けるために、非推奨の機能を使うことになったときにはいつでも非推奨の通知が発せられます。ブラウザで [開発環境](../configuration.html#configuration-environments) でアプリケーションにアクセスすると、これらの通知がウェブ開発ツールバーに表示されます。

![../\_images/deprecations-in-profiler.png](https://symfony.com/doc/4.3/_images/deprecations-in-profiler.png)

最終的には、非推奨機能の使用をやめることを目標にするべきです。これは簡単なこともあります: 警告で何を変更すべきかが正確に示されているかもしれません。

しかし、他の場合、警告が不明確なこともあります: どこかで設定することで、より深いクラスが警告のトリガーとなることがあります。この場合、symfony は明確なメッセージを伝えるために最善を尽くしますが、警告をさらに調査する必要があるかもしれません。

ときには、サードパーティのライブラリもしくは使用しているバンドルから警告が出ることもあります。これが本当なら、これらの非推奨事項はすでに更新されている可能性が高いです。その場合は、ライブラリをアップグレードして修正してください。

非推奨の警告がすべて消えれば、自信を持ってアップグレードすることができます。

### PHPUnit の非推奨事項

PHPUnit を使ってテストを実行するとき、非推奨の通知は表示されません。ここで手助けをするために、symfony は PHPUnit ブリッジを提供します。このブリッジはテストレポートの最後にすべての非推奨の通知をまとめて表示します。

必要なのは PHPUnit ブリッジをインストールすることだけです。

    $ composer require --dev symfony/phpunit-bridge

これで、通知の修正に取り掛かることができるようになりました。

```bash
# this command is available after running &quot;composer require --dev symfony/phpunit-bridge&quot;
$ ./bin/phpunit
...

OK (10 tests, 20 assertions)

Remaining deprecation notices (6)

        The &quot;request&quot; service is deprecated and will be removed in 3.0. Add a type-hint for
        Symfony\Component\HttpFoundation\Request to your controller parameters to retrieve the
        request instead: 6x
        3x in PageAdminTest::testPageShow from Symfony\Cmf\SimpleCmsBundle\Tests\WebTest\Admin
        2x in PageAdminTest::testPageList from Symfony\Cmf\SimpleCmsBundle\Tests\WebTest\Admin
        1x in PageAdminTest::testPageEdit from Symfony\Cmf\SimpleCmsBundle\Tests\WebTest\Admin
```

全部直したら、`0`(成功)で終了です。

----

弱い非推奨モードの使用

時には、すべての非推奨版を修正できないこともあります (例えば、3.4 で非推奨版になっていたものが 3.3 をサポートする必要がある場合など)。このような場合でも、ブリッジを使って可能な限り多くの非推奨版を修正し、より多くの非推奨版を許容することでテストを再び通過させることができます。これは `SYMFONY_DEPRECATIONS_HELPER` 環境変数を使うことで可能です。

```
<!-- phpunit.xml.dist -->
<phpunit>
<!-- ... -->

<php>
<env name="SYMFONY_DEPRECATIONS_HELPER" value="max[total]=999999"/>
</php>
</phpunit>
```

のようなコマンドを実行することもできます。

$ SYMFONY_DEPRECATIONS_HELPER=max[total]=999999 php ./bin/phpunit

## 2\) コンポーザー経由で新メジャーバージョンにアップデート

コードの非推奨がなくなったら、`composer.json` ファイルを修正することで Composer を通じて symfony ライブラリをアップデートできます。

```
    "...": "...",

    "require": {
        "symfony/symfony": "^4.1",
    },
    "...": "..."
```

次に、Composer を使用して、新しいバージョンのライブラリをダウンロードします。

```bash
$ composer update symfony/symfony
```

### 依存関係エラー

依存関係のエラーが発生する場合、他のsymfonyの依存関係もアップグレードする必要があることを意味します。その場合、次のコマンドを試してみてください。

```bash
$ composer update "symfony/*" --with-all-dependencies
```

これは `symfony/symfony` とそれに依存する*すべての*パッケージを更新します。`composer.json` の中で厳密なバージョン制約を使うことで、各ライブラリがどのバージョンにアップグレードするかを制御できます。

それでもうまくいかない場合、`composer.json` ファイルは symfony の新しいバージョンと互換性のないライブラリのバージョンを指定します。この場合、`composer.json` でライブラリを新しいバージョンに更新することで問題が解決するかもしれません。

あるいは、異なるライブラリが他のライブラリのバージョンに依存している、より深い問題を抱えているかもしれません。エラーメッセージをチェックしてデバッグしてみてください。

もう一つの問題として、プロジェクトの依存関係がローカルコンピュータにはインストールされていても、リモートサーバにはインストールされていないということがあります。これは通常、各マシンでPHPのバージョンが異なる場合に起こります。解決策は、 composer.json ファイルに [platform](https://getcomposer.org/doc/06-config.md#platform) config オプションを追加して、依存関係に許可されている最高の PHP バージョンを定義することです (サーバの PHP バージョンに設定してください)。

### 他のパッケージのアップグレード

また、残りのライブラリをアップグレードした方がいいかもしれません。もし、`composer.json`の中の[バージョン制約](https://getcomposer.org/doc/01-basic-usage.md#package-versions)がうまくいっていれば、実行することで安全に実行できます。

```bash
$ composer update
```

注意
: もしあなたが `composer.json` (例: `dev-master`) の中にいくつかの固有ではない [バージョン制約](https://getcomposer.org/doc/01-basic-usage.md#package-versions) を持っている場合、これはいくつかの非Symfonyライブラリを後方互換性を破壊する変更を含む新しいバージョンにアップグレードする可能性があるので注意してください。

## 3\) 新しいバージョンで動作するようにコードを更新する

次のメジャーバージョンはBCレイヤーが常に可能性があるとは限らないので、新しいBCブレイクを含むかもしれません。注意する必要があるBCブレイクについては、Symfonyのリポジトリに含まれる `UPGRADE-X.0.md` (Xは新しいメジャーバージョン)を必ず読んでください。


## 4\) symfony 4 Flex のディレクトリ構造に更新する

symfony 4 にアップグレードするとき、Symfony Flex を利用できるように新しい symfony 4 のディレクトリ構造にアップグレードしたいと思うでしょう。これにはいくつかの作業が必要ですが、オプションです。詳細は [*既存のアプリケーションをSymfony Flexにアップグレードする*] (flex.html) をご覧ください。
