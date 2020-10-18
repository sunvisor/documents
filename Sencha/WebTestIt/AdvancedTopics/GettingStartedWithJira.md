# Jira を使う

> オリジナル: [Getting started with Jira](https://docs.sencha.com/webtestit/guides/advanced-topics/getting-started-with-jira.html)

### Jira インスタンスへの接続を設定する

メインメニューから `Preferences` ダイアログにアクセスし、必要な項目を入力します。入力が完了したら、“Test connection” をクリックして、Jira アカウントへの接続をテストします。

![](https://docs.sencha.com/webtestit/guides/images/jira-one.png)

Jira接続を設定するもう一つの方法は、プロジェクトの *webtestit.json* ファイルを開き、**jiraCredentials** セクションにパラメータを設定することです。

```json
    "host": "jira-sandbox",
    "issueType": "Task",
    "password": "",
    "projectKey": "WTP",
    "protocol": "http",
    "statusCompletedName": "Done",
    "statusToDoName": "To Do",
    "strictSSL": false,
    "username": ""
  },
```

**username**: Jiraサーバー版の場合は、Jiraユーザー名を使用してください。Jira クラウドでは、この [Atlassian Community post](https://community.atlassian.com/t5/Jira-Software-questions/How-to-login-with-JIRA-username/qaq-p/730716) で説明されているように、専用のメールアドレスを使用します。

**Password**: サーバー版ではJiraアカウントのパスワードを使用します。Jira クラウド版では、API トークンを作成してパスワードフィールドで使用する必要があります。API トークンを作成するには、以下の手順に従ってください。

1.  [https://id.atlassian.com/manage/api-tokens](https://id.atlassian.com/manage/api-tokens?_ga=2.90269405.632423060.1557916489-2099514475.1539341538) にログインします。

2.  **Create API token** をクリックします。

3.  表示されたダイアログから、トークンの **Label** を簡潔に入力し、**Create**をクリックします。

4.  **Copy to clipboard** をクリックしてから、トークンをスクリプトに貼り付けるか、他の場所に貼り付けて保存します。 

    ![](https://docs.sencha.com/webtestit/guides/images/jira-two.png)

### Sencha WebTestIt を Jira で使う方法

テストケースは一度に1つの Jira issue にしか接続できません。リンク情報はテストケースの横にコメントとして保存され、必要な情報がすべて含まれており、必要に応じて編集することができます。一番良いのは、気にしないでコメントをそのままにしておくことです。そうすれば Sencha WebTestIt がすべてを代行してくれます。

![](https://docs.sencha.com/webtestit/guides/images/jira-three.png)

コメントがある場合、レポートは複数のアクションを提供しています。

  - **Create Issue** テストケースが失敗した場合、新しい Jira issue を作成することができます。そのため、レポートを開き、失敗したテストケースに移動し、それを展開して、*"Create Issue "*ボタンをクリックしてください。Sencha WebTestIt は、Jira プロジェクトをスパム化しないようにするために、新しい Jira issue を自動的に作成しません。

  - **Show Issue in Browser** テストケースが Jira の issue にリンクされると、レポートには *"Show Issue in Browser"* ボタンが表示され、デフォルトのブラウザで Jira の issue が開きます。

  - **Resolve Issue** 次にテストケースが成功すると、レポートで問題を解決することができます。ボタン *"Resolve Issue "* をクリックするだけで、*webtestit.json* ファイルに定義されている完了ステータス `statusCompletedName` で Jira の issue がクローズされます。

  - **Reopen Issue** もちろん、バグが再び現れることもあります。その場合は、レポート内の *"Reopen Issue"* ボタンをクリックすることで、issue をオープン再オープンすることができます。Jira プロジェクトがこのワークフロー移行をサポートしていることを確認してください。

#### その他のボタン

上のスクリーンショットでお気づきのように、私たちはより多くのオプションを導入しました。その目的は、ユーザーがテストプロセスを非常に効率的で時間のかからないものにするのを助けることです。

  - **Jump to test case**
    
    スタックトレースから直接エラーを含むテストケースにジャンプできるようになりました。テストはエディタで開かれ、エラーを含む行はハイライトされ、フォーカスされます。これにより、デバッグやテストプロセス自体がより速く、より効率的になりました。

  - **Run test in diagnostics mode**
    
    テスト実行後にブラウザを開いたままにしておくことで、ユーザーがテストをデバッグするのに役立つ診断モードでテストを実行するオプションを導入しました。この方法により、失敗したテストを分析し、特定のテストステップでテスト中のアプリケーションの状態を調べることができます。
