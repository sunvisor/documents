# Node.js と np の設定方法

> オリジナル: [How to setup Node.js and npm for use with Sencha WebTestIt](https://docs.sencha.com/webtestit/guides/advanced-topics/how-to-setup-node-js-and-npm-for-use-with-sencha-webtestit.html)

TypeScriptプロジェクトを作成するには、[Node.js](https://nodejs.org/en/) とデフォルトのパッケージマネージャ[npm](https://www.npmjs.com/) がインストールされていて、グローバルに利用できる状態になっている必要があります。

### 解説

1. [Node.js](https://nodejs.org/en/download/) の min バージョン 7.x.x (Java プロジェクト用) または 8.16.x (TypeScript プロジェクト用) をダウンロードしてください。
2.  セットアップの指示に従って、聞かれたら必ずPATH環境変数をインストールするようにしてください。
    
    > ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
    > 
    >  `/usr/local/bin` が PATH 環境変数に含まれていない場合は、次の設定方法が参考になるでしょう
    > - [macOS](https://www.cyberciti.biz/faq/appleosx-bash-unix-change-set-path-environment-variable/) 
    > - [Linux](https://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path) 

3.  インストール後にターミナル/コマンドラインを開く

4.  `node -v` と入力すると、インストールされた Node.js のバージョンを確認できます。

5.  `npm -v` と入力すると、インストールされた NPM のバージョンを確認できます。

6.  Sencha WebTestItを再起動します。
