## JS操作CSS的方法

### 操作HTML属性

通过 `el.setAttribute()` 方法，为 style 属性进行赋值

### CSSStyleDeclaration接口

```
// style 属性的值实际上是一个CSSStyleDeclaration实例，其包含的属性与CSS规则一一对应，但名字需要改成驼峰式，对于一些js保留字，则加上css前缀，如float->cssFloat
var divStyle = document.querySelector('div').style;
divStyle.backgroundColor = 'red';
```

#### 实例属性

- `CSSStyleDeclaration.cssText`属性用来读写当前规则的所有样式声明文本。

  ```
  var divStyle = document.querySelector('div').style;
  
  divStyle.cssText = 'background-color: red;'
    + 'border: 1px solid black;'
    + 'height: 100px;'
    + 'width: 100px;';
  ```

  注意，`cssText`的属性值不用改写 CSS 属性名。

  删除一个元素的所有行内样式，最简便的方法就是设置`cssText`为空字符串。

  ```
  divStyle.cssText = '';
  ```

- `CSSStyleDeclaration.length`属性返回一个整数值，表示当前规则包含多少条样式声明。
- `CSSStyleDeclaration.parentRule`属性返回当前规则所属的那个样式块（CSSRule 实例）。如果不存在所属的样式块，该属性返回`null`。

#### 实例方法

- `CSSStyleDeclaration.getPropertyPriority`方法接受 CSS 样式的属性名作为参数，返回一个字符串，表示有没有设置`important`优先级。如果有就返回`important`，否则返回空字符串。

  ```html
  // HTML 代码为
  // <div id="myDiv" style="margin: 10px!important; color: red;"/>
  var style = document.getElementById('myDiv').style;
  style.margin // "10px"
  style.getPropertyPriority('margin') // "important"
  style.getPropertyPriority('color') // ""
  ```

  上面代码中，`margin`属性有`important`优先级，`color`属性没有。

- `CSSStyleDeclaration.getPropertyValue`方法接受 CSS 样式属性名作为参数，返回一个字符串，表示该属性的属性值。

  ```html
  // HTML 代码为
  // <div id="myDiv" style="margin: 10px!important; color: red;"/>
  var style = document.getElementById('myDiv').style;
  style.margin // "10px"
  style.getPropertyValue("margin") // "10px"
  ```

- `CSSStyleDeclaration.removeProperty`方法接受一个属性名作为参数，在 CSS 规则里面移除这个属性，返回这个属性原来的值。

  ```js
  // HTML 代码为
  // <div id="myDiv" style="color: red; background-color: white;">
  //   111
  // </div>
  var style = document.getElementById('myDiv').style;
  style.removeProperty('color') // 'red'
  // HTML 代码变为
  // <div id="myDiv" style="background-color: white;">
  ```

  上面代码中，删除`color`属性以后，字体颜色从红色变成默认颜色。

- `CSSStyleDeclaration.setProperty`方法用来设置新的 CSS 属性。该方法没有返回值。

  该方法可以接受三个参数。

  - 第一个参数：属性名，该参数是必需的。
  - 第二个参数：属性值，该参数可选。如果省略，则参数值默认为空字符串。
  - 第三个参数：优先级，该参数可选。如果设置，唯一的合法值是`important`，表示 CSS 规则里面的`!important`。

  ```js
  // HTML 代码为
  // <div id="myDiv" style="color: red; background-color: white;">
  //   111
  // </div>
  var style = document.getElementById('myDiv').style;
  style.setProperty('border', '1px solid blue', 'important');
  ```

  上面代码执行后，`myDiv`元素就会出现蓝色的边框。

### CSS对象

浏览器原生提供，两个静态方法：

- CSS.escape()：转义CSS选择器中的特殊字符。如 `<div id="foo#bar">`，如果用 document 对象进行操作，只能写成 `document.querySelector('#foo\\#bar')`，使用此方法则写成 `document.querySelector('#' + CSS.escape('foo#bar'))`

- CSS.supports()：返回一个布尔值，表示当前环境是否支持某一句CSS规则：

  它的参数有两种写法，一种是第一个参数是属性名，第二个参数是属性值；另一种是整个参数就是一行完整的 CSS 语句。

  ```
  // 第一种写法
  CSS.supports('transform-origin', '5px') // true
  
  // 第二种写法
  CSS.supports('display: table-cell') // true
  ```

  注意，第二种写法的参数结尾不能带有分号，否则结果不准确。

  ```
  CSS.supports('display: table-cell;') // false
  ```



### window.getComputedStyle()

**行内样式（inline style）具有最高的优先级**，改变行内样式，通常会立即反映出来。但是，网页元素最终的样式是综合各种规则计算出来的。因此，如果想得到元素实际的样式，只读取行内样式是不够的，**需要得到浏览器最终计算出来的样式规则**。

`window.getComputedStyle`方法，就用来**返回浏览器计算后得到的最终规则**。它接受一个节点对象作为参数，**返回一个 CSSStyleDeclaration 实例**，包含了指定节点的最终样式信息。所谓“最终样式信息”，指的是各种 CSS 规则叠加后的结果。

```
var div = document.querySelector('div');
var styleObj = window.getComputedStyle(div);
styleObj.backgroundColor
```

上面代码中，得到的背景色就是`div`元素真正的背景色。

注意，**CSSStyleDeclaration 实例是一个活的对象，任何对于样式的修改，会实时反映到这个实例上面**。另外，这个实例是**只读的**。

`getComputedStyle`方法还可以接受第二个参数，表示**当前元素的伪元素**（比如`:before`、`:after`、`:first-line`、`:first-letter`等）。

```
var result = window.getComputedStyle(div, ':before');
```

也可以使用 CSSStyleDeclaration 实例的 getPropertyValue 方法获取伪元素属性

下面的例子是如何获取元素的高度。

```
var elem = document.getElementById('elem-container');
var styleObj = window.getComputedStyle(elem, null)
var height = styleObj.height;
// 等同于
var height = styleObj['height'];
var height = styleObj.getPropertyValue('height');
```

上面代码得到的`height`属性，是浏览器最终渲染出来的高度，比其他方法得到的高度更可靠。由于`styleObj`是 CSSStyleDeclaration 实例，所以可以使用各种 CSSStyleDeclaration 的实例属性和方法。

有几点需要注意。

- CSSStyleDeclaration 实例返回的 CSS 值都是**绝对单位**。比如，长度都是像素单位（返回值包括`px`后缀），颜色是`rgb(#, #, #)`或`rgba(#, #, #, #)`格式。
- **CSS 规则的简写形式无效**。比如，想读取`margin`属性的值，不能直接读，只能读`marginLeft`、`marginTop`等属性；再比如，`font`属性也是不能直接读的，只能读`font-size`等单个属性。
- **如果读取 CSS 原始的属性名，要用方括号运算符**，比如`styleObj['z-index']`；如果读取骆驼拼写法的 CSS 属性名，可以直接读取`styleObj.zIndex`。
- 该方法返回的 CSSStyleDeclaration 实例的`cssText`属性无效，返回`undefined`。



## StyleSheet接口

`StyleSheet`接口代表网页的一张样式表，包括`<link>`元素加载的样式表和`<style>`元素内嵌的样式表。

`document`对象的`styleSheets`属性，可以返回当前页面的所有`StyleSheet`实例（即所有样式表）。它是一个类似数组的对象。

## CSSRuleList 接口

CSSRuleList 接口是一个类似数组的对象，表示一组 CSS 规则，成员都是 CSSRule 实例。

获取 CSSRuleList 实例，一般是通过`StyleSheet.cssRules`属性。

```
// HTML 代码如下
// <style id="myStyle">
//   h1 { color: red; }
//   p { color: blue; }
// </style>
var myStyleSheet = document.getElementById('myStyle').sheet;
var crl = myStyleSheet.cssRules;
crl instanceof CSSRuleList // true
```

CSSRuleList 实例里面，每一条规则（CSSRule 实例）可以通过`rules.item(index)`或者`rules[index]`拿到。CSS 规则的条数通过`rules.length`拿到。还是用上面的例子。

```
crl[0] instanceof CSSRule // true
crl.length // 2
```

注意，添加规则和删除规则不能在 CSSRuleList 实例操作，而要在它的父元素 StyleSheet 实例上，通过`StyleSheet.insertRule()`和`StyleSheet.deleteRule()`操作。

## CSSRule 接口

### 概述

一条 CSS 规则包括两个部分：CSS 选择器和样式声明。下面就是一条典型的 CSS 规则。

```css
.myClass {
  color: red;
  background-color: yellow;
}
```

JavaScript 通过 CSSRule 接口操作 CSS 规则。一般通过 CSSRuleList 接口（`StyleSheet.cssRules`）获取 CSSRule 实例。

```html
// HTML 代码如下
// <style id="myStyle">
//   .myClass {
//     color: red;
//     background-color: yellow;
//   }
// </style>
var myStyleSheet = document.getElementById('myStyle').sheet;
var ruleList = myStyleSheet.cssRules;
var rule = ruleList[0];
rule instanceof CSSRule // true
```

### CSSRule 实例的属性

**（1）CSSRule.cssText**

`CSSRule.cssText`属性返回当前规则的文本，还是使用上面的例子。

```
rule.cssText
// ".myClass { color: red; background-color: yellow; }"
```

如果规则是加载（`@import`）其他样式表，`cssText`属性返回`@import 'url'`。

**（2）CSSRule.parentStyleSheet**

`CSSRule.parentStyleSheet`属性返回当前规则所在的样式表对象（StyleSheet 实例），还是使用上面的例子。

```
rule.parentStyleSheet === myStyleSheet // true
```

**（3）CSSRule.parentRule**

`CSSRule.parentRule`属性返回包含当前规则的父规则，如果不存在父规则（即当前规则是顶层规则），则返回`null`。

父规则最常见的情况是，当前规则包含在`@media`规则代码块之中。

```js
// HTML 代码如下
// <style id="myStyle">
//   @supports (display: flex) {
//     @media screen and (min-width: 900px) {
//       article {
//         display: flex;
//       }
//     }
//  }
// </style>
var myStyleSheet = document.getElementById('myStyle').sheet;
var ruleList = myStyleSheet.cssRules;

var rule0 = ruleList[0];
rule0.cssText
// "@supports (display: flex) {
//    @media screen and (min-width: 900px) {
//      article { display: flex; }
//    }
// }"

// 由于这条规则内嵌其他规则，
// 所以它有 cssRules 属性，且该属性是 CSSRuleList 实例
rule0.cssRules instanceof CSSRuleList // true

var rule1 = rule0.cssRules[0];
rule1.cssText
// "@media screen and (min-width: 900px) {
//   article { display: flex; }
// }"

var rule2 = rule1.cssRules[0];
rule2.cssText
// "article { display: flex; }"

rule1.parentRule === rule0 // true
rule2.parentRule === rule1 // true
```

**（4）CSSRule.type**

`CSSRule.type`属性返回一个整数值，表示当前规则的类型。

最常见的类型有以下几种。

- 1：普通样式规则（CSSStyleRule 实例）
- 3：`@import`规则
- 4：`@media`规则（CSSMediaRule 实例）
- 5：`@font-face`规则

### CSSStyleRule 接口

如果一条 CSS 规则是普通的样式规则（不含特殊的 CSS 命令），那么除了 CSSRule 接口，它还部署了 CSSStyleRule 接口。

CSSStyleRule 接口有以下两个属性。

**（1）CSSStyleRule.selectorText**

`CSSStyleRule.selectorText`属性返回当前规则的选择器。

```
var stylesheet = document.styleSheets[0];
stylesheet.cssRules[0].selectorText // ".myClass"
```

注意，这个属性是可写的。

**（2）CSSStyleRule.style**

`CSSStyleRule.style`属性返回一个对象（CSSStyleDeclaration 实例），代表当前规则的样式声明，也就是选择器后面的大括号里面的部分。

```
// HTML 代码为
// <style id="myStyle">
//   p { color: red; }
// </style>
var styleSheet = document.getElementById('myStyle').sheet;
styleSheet.cssRules[0].style instanceof CSSStyleDeclaration
// true
```

CSSStyleDeclaration 实例的`cssText`属性，可以返回所有样式声明，格式为字符串。

```
styleSheet.cssRules[0].style.cssText
// "color: red;"
styleSheet.cssRules[0].selectorText
// "p"
```