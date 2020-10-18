# TestRail を使う

> オリジナル: [Getting started with TestRail](https://docs.sencha.com/webtestit/guides/advanced-topics/getting-started-with-testrail.html)

**前提条件: TestRail で Administration \> Site settings \> API の下の “Enable API” を有効にします**

TestRail アカウントを Sencha WebTestIt に接続するには、プロジェクトを開き、メインメニューから ***Sencha WebTestIt/File \> Preferences \> TestRail integration*** をクリックします。

1.  環境設定ダイアログで、TestRail のプロファイル URL を入力します。

2.  **Connect** をクリックし、プロンプトが表示されたら資格情報を入力します。URL と認証情報が有効であれば、ダイアログに緑色で ***Connected*** ステータスが表示されます。
    
    ![](https://docs.sencha.com/webtestit/guides/images/testrail-one.png)

3.  ダイアログでプロジェクトIDを指定することができ、このフィールドを空白のままにしておくと、インポートダイアログで利用可能なTestRailプロジェクトのリストからプロジェクトを選択します。

4.  変更を保存すると、プロジェクトツリーの上にインポートアイコンが表示されます。このアイコンをクリックすると、インポートダイアログが表示され、プロジェクトのインポートに進みます。このアイコンは、TestRail アカウントを Sencha WebTestIt に接続した場合にのみ使用できます。
    
    ![](https://docs.sencha.com/webtestit/guides/images/testrail-two.png)

5.  **Import** ボタンをクリックすると、インポートダイアログが開き、ドロップダウンからプロジェクトを選択できます。テストはグループ化され、プロジェクトスイートの、セクション、サブセクションに含まれています。 
   
    ![](https://docs.sencha.com/webtestit/guides/images/testrail-three.png)
    
    インポートされたテストファイルには、テストケースを TestRail にリンクするための caseID と suiteID に関する追加情報と、ブラウザでテストケースを開くためのダイレクトリンクが含まれています。利用可能なテストステップがある場合、それらはフォーマットされ、コメントとしてテストファイルに表示されます。
    
    ![](https://docs.sencha.com/webtestit/guides/images/testrail-four.png)

6.  テストの実行が完了し、結果を TestRail アカウントにエクスポートしたい場合は、レポートを開き、**Export to TestRail**をクリックします。すでに TestRail プロジェクトにリンクされているテストケースだけがエクスポートできることに注意してください。
    
    ![](https://docs.sencha.com/webtestit/guides/images/testrail-five.png)

7.  プロジェクトの新しいテストランでテストケースをエクスポートすることができます。希望のフォーマットを選択し、**Upload** ボタンをクリックして完了です。
    
    ![](https://docs.sencha.com/webtestit/guides/images/testrail-six.png)
