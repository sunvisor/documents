# ドラッグ＆ドロップの実行方法

> オリジナル: [How to perform the Drag And Drop action](https://docs.sencha.com/webtestit/guides/advanced-topics/how-to-perform-the-drag-and-drop-action.html)

新しいアウトオブザボックスアクションを利用するには、エレメントをコードエディタにドラッグして、ドロップダウンメニューから目的のアクションを選択するだけです。

> ![](https://docs.sencha.com/webtestit/guides/images/hint-icon.png) **Hint**
> 
> Actions クラスを使用する "伝統的な方法" も、以下のようにサポートされています。

### **dragAndDrop() Action**

Web アプリケーションの中には、Web エレメントをドラッグして定義された領域やエレメントにドロップする機能を持っているものがあります。Selenium Webdriverを使用して、そのようなエレメントのドラッグ＆ドロップを自動化することができます。

ドラッグ＆ドロップの自動化を可能にする方法はいくつかあります。

以下の例では、`Actions.dragAndDrop(sourceLocator, destinationLocator)` を使用しています。

`dragAndDrop` メソッドには、2つのパラメータがあります。 

  - 最初のパラメーター: "WebElement source" はドラッグする要素です。

  - 2番目のパラメーター: "WebElement target" は、ソース要素をドロップする必要がある要素です。

#### ワークフロー

  - ソースとターゲットの WebElements を定義して生成します。

    ```java
    WebElement source = this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.DragMe));
    WebElement target = this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.DropHere));
    ```

  - アクションクラスをインスタンス化します。

    ```java
    Actions act = new Actions(driver);
    ```

  - dragAndDrop アクションを実行します。

    ```java
    act.dragAndDrop(source, target).perform();
    ```

**全体のアクションステップは次のようになります。:**

```java
public itemsPo dragAndDrop() {

  // 要素を探す
  WebElement sourceElement = this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.source));
  WebElement targetElement = this.wait.until(ExpectedConditions.visibilityOfElementLocated(this.target));

  // Actions クラスのインスタンス化
  Actions act = new Actions(driver);

  // dragAndDrop() メソッドを実行
  act.dragAndDrop(sourceElement, targetElement).perform();

  return this;
}
```

**またはワンライナーで**

```java
(new Actions(driver).dragAndDrop(source, target)).perform();
```

dragAndDropアクションを実行するもう一つの方法は `dragAndDropBy(source, xOffset, yOffset)` メソッドです。ここには3つのパラメータがあります。

  - source (WebElement) - アクションを実行している要素
  - xOffset (Integer) - 水平移動オフセット
  - yOffset (Integer) - 垂直移動オフセット

要素を特定の位置にドラッグしたい場合、例えば水平方向に100px、垂直方向に50pxといったように、このアプローチを使うことができます。

```java
(new Actions(driver).dragAndDrop(source, 100, 50)).perform();
```
