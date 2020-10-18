# 既存のプロジェクトで Sencha WebTestIt を使用する

> オリジナル: [Using Sencha WebTestIt with existing projects](https://docs.sencha.com/webtestit/guides/advanced-topics/using-sencha-webtestit-with-existing-projects.html)

多くのシナリオでは、自動テストは、開発中のウェブサイトやアプリケーションのソースコードの一部です。これらのプロジェクトは、IntelliJ IDEAやVisual Studioのような統合開発環境（IDE）によって作成されます。

いくつかの基本的なルールに従う限り、Sencha WebTestItを使って他のIDEから既存のプロジェクトを開いて、その中のテストケースを編集することができます。

> ![](https://docs.sencha.com/webtestit/guides/images/note-icon.png) **Note**
> 
> 既存のプロジェクトでのテストの編集は、テスト環境がサポートされているテストフレームワークのいずれかを使用している場合にのみ機能します。
> テスト実行に異なるフレームワークを使用している場合、Sencha WebTestIt は統合できません。

開始するには、Sencha WebTestIt で開発プロジェクトのルートフォルダを開きます。次のダイアログでは、新しい Sencha WebTestIt 設定ファイルを作成するためにいくつかの情報を提供する必要があります。これは主にプロジェクト名と使用する言語に関係しています。

<img src="https://docs.sencha.com/webtestit/guides/images/open-existing-project.png" width="700px" />
<p></p>

設定ファイルを追加した後、**Project** タブを使用してファイルを閲覧することができます。空のプロジェクトの場合と同じように、新しいページオブジェクトとテストファイルを追加することができます。

### Sencha WebTestIt で既存のファイルを動作させる方法

Sencha WebTestItは、ソースファイル内の *contextual comments* に依存して、それらをコアオブジェクトとして識別します。コアオブジェクトはテストファイルやページオブジェクトファイルであり、Sencha WebTestIt UI を適応させます。

Drag&Dropのような機能を利用するためには、プロジェクト内の既存のページオブジェクトファイルを変更する必要があります。

ページオブジェクトファイルをそのように検出できるようにするには、**Project** タブの**ファイルの最初の行に**以下のコメントを追加する必要があります。
`// Sencha WebTestItt Page Object File`
ファイルのアイコンが変わり、Sencha WebTestIttで作成したように見えます。

エディタは、別の色で特別なコメントを表示し、ユーザーインターフェイスは、コードを記述する必要がなく、セレクタを作成することができました。

![](https://docs.sencha.com/webtestit/guides/images/existing-files-project.png)

テストファイルでは、ファイルの最初の行に以下のコメントをつけて指定する必要があります。`Sencha WebTestIt Test File`

このように変更したテストファイルをクリックすると、Sencha WebTestIt は **Execution** タブを開き、設定したエンドポイントで直接実行できるようにします。

### 新しいエレメントの追加

以下の手順で、Sencha WebTestItt プロジェクトの場合と同じように、新しいページオブジェクトファイルとテストファイルを既存のプロジェクトに追加することができます。

1.  **Project** タブに移動し、新しいファイルを作成するフォルダを選択 (または作成) します。
2. フォルダを右クリックして、新しいページオブジェクトを追加するには **New \> Page Object file** を選択し、新しいテストを追加するには **New \> Test file** を選択します。
3. Sencha WebTestIt は、**Execute**tab で **Run all test files** をクリックすると、プロジェクトインフラストラクチャのテストを自動的にスキャンして実行します。
