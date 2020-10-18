## GUIテストのコンポーネント

> オリジナル: [Components of a GUI test](https://docs.sencha.com/webtestit/guides/automation/components.html)

エレメントとセレクタは、完全に機能するテストを構成する主なコンポーネントです。これらは、Web サイトやアプリを自動化するための基礎となります。以下の例のように、すべてを1つのテスト機能に落とし込んで、テストを開始することができます。

#### Java
```java
@Test
public void checkCredentials() throws InterruptedException {
  By Username = By.cssSelector("#username");
  By Password = By.cssSelector("#password");
  By Login = By.cssSelector("[class='fa fa-2x fa-sign-in']");
  By ResultMessage = By.cssSelector("#flash");

  WebDriverWait wait = new WebDriverWait(driver, 10);

  // Open the Web Page
  driver.get("https://the-internet.herokuapp.com/login");

  // Attempt to log in
  wait.until(ExpectedConditions.visibilityOfElementLocated(Username)).sendKeys("username");
  wait.until(ExpectedConditions.visibilityOfElementLocated(Password)).sendKeys("password");
  wait.until(ExpectedConditions.visibilityOfElementLocated(Login)).click();

  // Capture the login message after clicking the button
  String resultMessage = wait.until(ExpectedConditions.visibilityOfElementLocated(ResultMessage)).getText().replaceAll("×", "").trim();

  Assert.assertEquals(resultMessage, "Your username is invalid!");
}
```

#### Python
```python
def test_check_credentials(self):
    driver = self.get_driver()
    username = (By.CSS_SELECTOR, "#username")
    password = (By.CSS_SELECTOR, "#password")
    login = (By.CSS_SELECTOR, "[class=\'fa fa-2x fa-sign-in\']")
    result_message = (By.CSS_SELECTOR, "#flash")
    wait = WebDriverWait(driver, 10)
    # Open the Web Page
    driver.get('https://the-internet.herokuapp.com/login')
    # Attempt to log in
    wait.until(EC.visibility_of_element_located(username)).send_keys("username")
    wait.until(EC.visibility_of_element_located(password)).send_keys("password")
    wait.until(EC.visibility_of_element_located(login)).click()
    # Capture the login message after clicking the button
    result = wait.until(EC.visibility_of_element_located(result_message)).text.replace("×", "").strip()
    self.assertEqual(result, "Your username is invalid!")
```

#### TypeScript
```typescript
it('should check credentials', async () => {
  const username = by.css('#username');
  const password = by.css('#password');
  const login = by.css('[class=\'fa fa-2x fa-sign-in\']');
  const resultMessage = by.css('#flash');

  // Open the Web Page
  await browser.driver.get('https://the-internet.herokuapp.com/login');

  // Attempt to log in
  await browser.wait(ExpectedConditions.visibilityOf(element(username)), browser.allScriptsTimeout, username.toString());
  await element(username).sendKeys('username');
  await browser.wait(ExpectedConditions.visibilityOf(element(password)), browser.allScriptsTimeout, password.toString());
  await element(password).sendKeys('password');
  await browser.wait(ExpectedConditions.visibilityOf(element(login)), browser.allScriptsTimeout, login.toString());
  await element(login).click();

  // Capture the login message after clicking the button
  await browser.wait(ExpectedConditions.visibilityOf(element(resultMessage)), browser.allScriptsTimeout, resultMessage.toString());
  const result = await element(resultMessage).getText().then((m) => m.replace(/×/g, '').trim());

  await expect(result).toEqual('Your username is invalid!');
});
}
```

自動化を始める際には、テストをシンプルに保ち、すべてを一括して自動化するだけで十分だと感じるかもしれません。しかし、テストケースの数が増えると、コードの概要を維持するのが難しいと感じるかもしれません。そのため、テストは通常複数のコンポーネントで構成されています。

![](https://docs.sencha.com/webtestit/guides/images/Test_structuring.png)

### テストファイルの構造

Sencha WebTestIt のテストファイルの実際の内容は、プロジェクトを作成する際に選択した自動化フレームワークとは異なります。しかし、用語が変わっても、コンセプトは広く同じです。

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>用語</strong></th>
<th><strong>Protractor</strong></th>
<th><strong>TestNG Annotation(s)</strong></th>
<th><strong>unittest hooks</strong></th>
<th><strong>説明</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Test Test Suite</td>
<td>Scenario</td>
<td>Suite</td>
<td>Test Suite</td>
<td>テストケースのコレクション</td>
</tr>
<tr class="even">
<td>Test case</td>
<td>Test</td>
<td>Test</td>
<td>Test</td>
<td>アサーションの後に続くアクションの単一の、自己完結したオーケストレーション。
<br>pass/failのどちらかです。</td>
</tr>
<tr class="odd">
<td>Setup</td>
<td>beforeEach()<br />
beforeAll()</td>
<td>@BeforeMethod<br />
@BeforeSuite<br />
@BeforeTest</td>
<td>setUpClass()<br />
setUp()</td>
<td>各テストケース、テストスイート、またはすべてのテストの前に実行される関数。</td>
</tr>
<tr class="even">
<td>Teardown</td>
<td>afterEach()<br />
<span>afterAll()</span></td>
<td>@AfterMethod<br />
@AfterSuite<br />
@AfterTest</td>
<td>tearDownClass()<br />
tearDown()</td>
<td>各テストケース、テストスイート、またはすべてのテストの後に実行される関数。</td>
</tr>
</tbody>
</table>

テストを実行するとき、Sencha WebTestIt は各テストファイルを 1 つのユニットとして扱います。現在のテストファイルを実行するか、プロジェクト内のすべてのテストファイルを実行するかを選択することができます。テストを構造化する際には、このことを念頭に置いてください。

> ![](https://docs.sencha.com/webtestit/guides/images/reference-icon.png) **Reference**
> 
> テスト実行の詳細については、[こちら](../PageObjects/Introduction.md) を参照してください。
