## DOM

DOM 是 JavaScript 操作网页的接口，全称为“文档对象模型”（Document Object Model）。它的作用是将网页转为一个 JavaScript 对象，从而可以用脚本进行各种操作（比如增删内容）。

浏览器会根据 DOM 模型，将结构化文档（比如 HTML 和 XML）解析成一系列的节点，再由这些节点组成一个树状结构（DOM Tree）。所有的节点和最终的树状结构，都有规范的对外接口。

DOM 只是一个**接口规范**，可以用各种语言实现。所以严格地说，DOM 不是 JavaScript 语法的一部分，但是 DOM 操作是 JavaScript 最常见的任务，离开了 DOM，JavaScript 就无法控制网页。另一方面，JavaScript 也是最常用于 DOM 操作的语言。后面介绍的就是 JavaScript 对 DOM 标准的实现和用法。

## 节点

DOM 的最小组成单位叫做节点（node）。文档的树形结构（DOM 树），就是由各种不同类型的节点组成。每个节点可以看作是文档树的一片叶子。

节点的类型有七种。

- `Document`：整个文档树的顶层节点
- `DocumentType`：`doctype`标签（比如`<!DOCTYPE html>`）
- `Element`：网页的各种HTML标签（比如`<body>`、`<a>`等）
- `Attr`：网页元素的属性（比如`class="right"`）
- `Text`：标签之间或标签包含的文本
- `Comment`：注释
- `DocumentFragment`：文档的片段

浏览器提供一个原生的节点对象`Node`，**上面这七种节点都继承了`Node`**，因此具有一些共同的属性和方法。

## 节点树

一个文档的所有节点，按照所在的层级，可以抽象成一种树状结构。这种树状结构就是 DOM 树。它有一个顶层节点，下一层都是顶层节点的子节点，然后子节点又有自己的子节点，就这样层层衍生出一个金字塔结构，又像一棵树。

浏览器原生提供`document`节点，代表整个文档。

```
document
// 整个文档树
```

文档的第一层有两个节点，第一个是文档类型节点（`<!doctype html>`），第二个是 HTML 网页的顶层容器标签`<html>`。后者构成了树结构的根节点（root node），其他 HTML 标签节点都是它的下级节点。

除了根节点，其他节点都有三种层级关系。

- 父节点关系（parentNode）：直接的那个上级节点
- 子节点关系（childNodes）：直接的下级节点
- 同级节点关系（sibling）：拥有同一个父节点的节点

DOM 提供操作接口，用来获取这三种关系的节点。比如，子节点接口包括`firstChild`（第一个子节点）和`lastChild`（最后一个子节点）等属性，同级节点接口包括`nextSibling`（紧邻在后的那个同级节点）和`previousSibling`（紧邻在前的那个同级节点）属性。



## Document 节点

`document`节点对象代表整个文档，每张网页都有自己的`document`对象。`window.document`属性就指向这个对象。只要浏览器开始载入 HTML 文档，该对象就存在了，可以直接使用。

- `document`对象有不同的办法可以获取。
  - 正常的网页，直接使用`document`或`window.document`。
  - `iframe`框架里面的网页，使用`iframe`节点的`contentDocument`属性。
  - Ajax 操作返回的文档，使用`XMLHttpRequest`对象的`responseXML`属性。
  - 内部节点的`ownerDocument`属性。

- `document.defaultView`属性：返回`document`对象所属的`window`对象。如果当前文档不属于`window`对象，该属性返回`null`。

- 对于 HTML 文档来说，`document`对象一般有两个子节点。第一个子节点是`document.doctype`，指向`<DOCTYPE>`节点，即文档类型（Document Type Declaration，简写DTD）节点。HTML 的文档类型节点，一般写成`<!DOCTYPE html>`。如果网页没有声明 DTD，该属性返回`null`。

  ```js
  var doctype = document.doctype;
  doctype // "<!DOCTYPE html>"
  doctype.name // "html"
  ```

  `document.firstChild`通常就返回这个节点。

- `document.documentElement`属性：返回当前文档的根元素节点（root）。它通常是`document`节点的第二个子节点，紧跟在`document.doctype`节点后面。HTML网页的该属性，一般是`<html>`节点。

- `document.body`属性指向`<body>`节点，`document.head`属性指向`<head>`节点。

  这两个属性总是存在的，如果网页源码里面省略了`<head>`或`<body>`，浏览器会自动创建。另外，这两个属性是可写的，如果改写它们的值，相当于移除所有子节点。

- `document.scrollingElement`属性：返回文档的滚动元素。也就是说，当文档整体滚动时，到底是哪个元素在滚动。

  标准模式下，这个属性返回的文档的根元素`document.documentElement`（即`<html>`）。兼容（quirk）模式下，返回的是`<body>`元素，如果该元素不存在，返回`null`。

  ```
  // 页面滚动到浏览器顶部
  document.scrollingElement.scrollTop = 0;
  ```

- `document.activeElement`属性：返回获得当前焦点（focus）的 DOM 元素。通常，这个属性返回的是`<input>`、`<textarea>`、`<select>`等表单元素，如果当前没有焦点元素，返回`<body>`元素或`null`。

- `document.fullscreenElement`属性返回当前**以全屏状态展示的** DOM 元素。如果不是全屏状态，该属性返回`null`。

  ```js
  if (document.fullscreenElement.nodeName == 'VIDEO') {
    console.log('全屏播放视频');
  }
  ```

  上面代码中，通过`document.fullscreenElement`可以知道`<video>`元素有没有处在全屏状态，从而判断用户行为。



## 节点属性集合

以下属性返回一个`HTMLCollection`实例，表示文档内部特定元素的集合。这些集合都是动态的，原节点有任何变化，立刻会反映在集合中。

**（1）document.links**

`document.links`属性返回当前文档所有设定了`href`属性的`<a>`及`<area>`节点。

```
// 打印文档所有的链接
var links = document.links;
for(var i = 0; i < links.length; i++) {
  console.log(links[i]);
}
```

**（2）document.forms**

`document.forms`属性返回所有`<form>`表单节点。

```
var selectForm = document.forms[0];
```

上面代码获取文档第一个表单。

除了使用位置序号，`id`属性和`name`属性也可以用来引用表单。

```
/* HTML 代码如下
  <form name="foo" id="bar"></form>
*/
document.forms[0] === document.forms.foo // true
document.forms.bar === document.forms.foo // true
```

**（3）document.images**

`document.images`属性返回页面所有`<img>`图片节点。

```
var imglist = document.images;

for(var i = 0; i < imglist.length; i++) {
  if (imglist[i].src === 'banner.gif') {
    // ...
  }
}
```

上面代码在所有`img`标签中，寻找某张图片。

**（4）document.embeds，document.plugins**

`document.embeds`属性和`document.plugins`属性，都返回所有`<embed>`节点。

**（5）document.scripts**

`document.scripts`属性返回所有`<script>`节点。

```
var scripts = document.scripts;
if (scripts.length !== 0 ) {
  console.log('当前网页有脚本');
}
```

**（6）document.styleSheets**

`document.styleSheets`属性返回网页内嵌或引入的 CSS 样式表集合

**（7）小结**

除了`document.styleSheets`属性，以上的其他集合属性返回的都是`HTMLCollection`实例。`document.styleSheets`属性返回的是`StyleSheetList`实例。



## 文档静态信息属性

以下属性返回文档信息。

**（1）document.documentURI，document.URL**

`document.documentURI`属性和`document.URL`属性都返回一个字符串，表示当前文档的网址。不同之处是它们继承自不同的接口，**`documentURI`继承自`Document`接口，可用于所有文档；`URL`继承自`HTMLDocument`接口，只能用于 HTML 文档。**

```
document.URL
// http://www.example.com/about

document.documentURI === document.URL
// true
```

如果文档的锚点（`#anchor`）变化，这两个属性都会跟着变化。

**（2）document.domain**

**`document.domain`属性返回当前文档的域名，不包含协议和端口**。比如，网页的网址是`http://www.example.com:80/hello.html`，那么`document.domain`属性就等于`www.example.com`。如果无法获取域名，该属性返回`null`。

`document.domain`基本上是一个只读属性，只有一种情况除外。次级域名的网页，可以把`document.domain`设为对应的上级域名。比如，当前域名是`a.sub.example.com`，则`document.domain`属性可以设置为`sub.example.com`，也可以设为`example.com`。修改后，`document.domain`相同的两个网页，可以读取对方的资源，比如设置的 Cookie。

另外，**设置`document.domain`会导致端口被改成`null`**。因此，如果通过设置`document.domain`来进行通信，双方网页都必须设置这个值，才能保证端口相同。

**（3）document.location**

`Location`对象是浏览器提供的原生对象，提供 URL 相关的信息和操作方法。通过`window.location`和`document.location`属性，可以拿到这个对象。

关于这个对象的详细介绍，请看《浏览器模型》部分的《Location 对象》章节。

**（4）document.lastModified**

`document.lastModified`属性返回一个字符串，表示当前文档最后修改的时间。不同浏览器的返回值，日期格式是不一样的。

```
document.lastModified
// "03/07/2018 11:18:27"
```

注意，`document.lastModified`属性的值是字符串，所以不能直接用来比较。`Date.parse`方法将其转为`Date`实例，才能比较两个网页。

```
var lastVisitedDate = Date.parse('01/01/2018');
if (Date.parse(document.lastModified) > lastVisitedDate) {
  console.log('网页已经变更');
}
```

如果页面上有 JavaScript 生成的内容，`document.lastModified`属性返回的总是当前时间。

**（5）document.title**

`document.title`属性返回当前文档的标题。默认情况下，返回`<title>`节点的值。但是该属性是可写的，一旦被修改，就返回修改后的值。

```
document.title = '新标题';
document.title // "新标题"
```

**（6）document.characterSet**

`document.characterSet`属性返回当前文档的编码，比如`UTF-8`、`ISO-8859-1`等等。

**（7）document.referrer**

`document.referrer`属性返回一个字符串，表示当前文档的访问者来自哪里。

```
document.referrer
// "https://example.com/path"
```

如果无法获取来源，或者用户直接键入网址而不是从其他网页点击进入，`document.referrer`返回一个空字符串。

`document.referrer`的值，总是与 HTTP 头信息的`Referer`字段保持一致。但是，`document.referrer`的拼写有两个`r`，而头信息的`Referer`字段只有一个`r`。

**（8）document.dir**

`document.dir`返回一个字符串，表示文字方向。它只有两个可能的值：`rtl`表示文字从右到左，阿拉伯文是这种方式；`ltr`表示文字从左到右，包括英语和汉语在内的大多数文字采用这种方式。

**（9）document.compatMode**

`compatMode`属性返回浏览器处理文档的模式，可能的值为`BackCompat`（向后兼容模式）和`CSS1Compat`（严格模式）。

一般来说，如果网页代码的第一行设置了明确的`DOCTYPE`（比如`<!doctype html>`），`document.compatMode`的值都为`CSS1Compat`。



## 文档状态属性

**（1）document.hidden**

`document.hidden`属性返回一个布尔值，表示当前页面是否可见。如果**窗口最小化、浏览器切换了 Tab**，都会导致导致页面不可见，使得`document.hidden`返回`true`。

这个属性是 Page Visibility API 引入的，一般都是配合这个 API 使用。

**（2）document.visibilityState**

`document.visibilityState`返回文档的可见状态。

它的值有四种可能。

> - `visible`：页面可见。注意，页面可能是部分可见，即不是焦点窗口，前面被其他窗口部分挡住了。
> - `hidden`：页面不可见，有可能窗口最小化，或者浏览器切换到了另一个 Tab。
> - `prerender`：页面处于正在渲染状态，对于用户来说，该页面不可见。
> - `unloaded`：页面从内存里面卸载了。

**这个属性可以用在页面加载时，防止加载某些资源；或者页面不可见时，停掉一些页面功能。**

**（3）document.readyState**

`document.readyState`属性返回当前文档的状态，共有三种可能的值。

- `loading`：加载 HTML 代码阶段（尚未完成解析）
- `interactive`：加载外部资源阶段
- `complete`：加载完成

这个属性变化的过程如下。

1. 浏览器开始**解析 HTML 文档**，`document.readyState`属性等于`loading`。
2. 浏览器遇到 HTML 文档中的`<script>`元素，并且**没有`async`或`defer`属性**，就暂停解析，开始**执行脚本**，这时`document.readyState`属性还是等于`loading`。
3. HTML 文档**解析完成**，`document.readyState`属性**变成`interactive`。**
4. 浏览器等待图片、样式表、字体文件等外部资源加载完成，一旦全部加载完成，`document.readyState`属性变成`complete`。

下面的代码用来检查网页是否加载成功。

```js
// 基本检查
if (document.readyState === 'complete') {
  // ...
}

// 轮询检查
var interval = setInterval(function() {
  if (document.readyState === 'complete') {
    clearInterval(interval);
    // ...
  }
}, 100);
```

另外，每次状态变化都会触发一个`readystatechange`事件。



## Document对象方法

- `document.open`方法清除当前文档所有内容，使得文档处于可写状态，供`document.write`方法写入内容。

  `document.close`方法用来关闭`document.open()`打开的文档。

  ```
  document.open();
  document.write('hello world');
  document.close();
  ```

- `document.write`方法用于向当前文档写入内容。

  在网页的首次渲染阶段，只要页面没有关闭写入（即没有执行`document.close()`），`document.write`写入的内容就会追加在已有内容的后面。

  ```
  // 页面显示“helloworld”
  document.open();
  document.write('hello');
  document.write('world');
  document.close();
  ```

  注意，`document.write`会**当作 HTML 代码解析**，不会转义。

  ```
  document.write('<p>hello world</p>');
  ```

  上面代码中，`document.write`会将`<p>`当作 HTML 标签解释。

  如果页面已经解析完成（`DOMContentLoaded`事件发生之后），再调用`write`方法，它会先调用`open`方法，擦除当前文档所有内容，然后再写入。

- `document.querySelector`方法接受一个 CSS 选择器作为参数，返回匹配该选择器的元素节点。如果有多个节点满足匹配条件，则返回第一个匹配的节点。如果没有发现匹配的节点，则返回`null`。

  ```
  var el1 = document.querySelector('.myclass');
  var el2 = document.querySelector('#myParent > [ng-click]');
  ```

  `document.querySelectorAll`方法与`querySelector`用法类似，区别是返回一个**`NodeList`**对象，包含所有匹配给定选择器的节点。

  ```
  elementList = document.querySelectorAll('.myclass');
  ```

  这两个方法的参数，可以是逗号分隔的多个 CSS 选择器，返回匹配其中一个选择器的元素节点，这与 CSS 选择器的规则是一致的。

  ```
  var matches = document.querySelectorAll('div.note, div.alert');
  ```

  上面代码返回`class`属性是`note`或`alert`的`div`元素。

  这两个方法都支持复杂的 CSS 选择器。

  ```js
  // 选中 data-foo-bar 属性等于 someval 的元素
  document.querySelectorAll('[data-foo-bar="someval"]');
  
  // 选中 myForm 表单中所有不通过验证的元素
  document.querySelectorAll('#myForm :invalid');
  
  // 选中div元素，那些 class 含 ignore 的除外
  document.querySelectorAll('DIV:not(.ignore)');
  
  // 同时选中 div，a，script 三类元素
  document.querySelectorAll('DIV, A, SCRIPT');
  ```

  但是，它们**不支持 CSS 伪元素的选择器**（比如`:first-line`和`:first-letter`）和伪类的选择器（比如`:link`和`:visited`），即无法选中伪元素和伪类。

  如果`querySelectorAll`方法的参数是字符串 `*`，则会返回文档中的所有元素节点。另外，`querySelectorAll`的返回结果不是动态集合，不会实时反映元素节点的变化。

  最后，这两个方法除了定义在`document`对象上，还定义在元素节点上，即在元素节点上也可以调用。

- `document.getElementsByTagName()`方法：搜索 HTML 标签名，返回符合条件的元素。它的返回值是一个**类似数组对象（`HTMLCollection`实例）**，可以实时反映 HTML 文档的变化。如果没有任何匹配的元素，就返回一个空集。

  ```
  var paras = document.getElementsByTagName('p');
  paras instanceof HTMLCollection // true
  ```

  上面代码返回当前文档的所有`p`元素节点。

  HTML 标签名是**大小写不敏感**的，因此`getElementsByTagName()`方法的参数也是大小写不敏感的。另外，返回结果中，各个成员的顺序就是它们在文档中出现的顺序。

  如果传入`*`，就可以返回文档中所有 HTML 元素。注意，**元素节点本身**也定义了`getElementsByTagName`方法，返回该元素的后代元素中符合条件的元素。

- `document.getElementsByClassName()`方法：返回一个类似数组的对象（`HTMLCollection`实例），包括了所有`class`名字符合指定条件的元素，元素的变化实时反映在返回结果中。

  ```
  var elements = document.getElementsByClassName(names);
  ```

  由于`class`是保留字，所以 JavaScript 一律使用`className`表示 CSS 的`class`。

  参数可以是多个`class`，它们之间使用空格分隔。

  ```
  var elements = document.getElementsByClassName('foo bar');
  ```

  上面代码返回**同时具有**`foo`和`bar`两个`class`的元素，`foo`和`bar`的顺序不重要。

  注意，正常模式下，CSS 的`class`是**大小写敏感**的。（`quirks mode`下，大小写不敏感。）

  与`getElementsByTagName()`方法一样，`getElementsByClassName()`方法不仅可以在`document`对象上调用，也可以在任何元素节点上调用。

  ```
  // 非document对象上调用
  var elements = rootElement.getElementsByClassName(names);
  ```

- `document.getElementsByName()`方法：用于选择拥有`name`属性的 HTML 元素（比如`<form>`、`<radio>`、`<img>`、`<frame>`、`<embed>`和`<object>`等），返回一个类似数组的的对象（`NodeList`实例），因为`name`属性相同的元素可能不止一个。

  ```
  // 表单为 <form name="x"></form>
  var forms = document.getElementsByName('x');
  forms[0].tagName // "FORM"
  ```

- `document.getElementById()`方法返回匹配指定`id`属性的元素节点。如果没有发现匹配的节点，则返回`null`。效率较高

  ```
  var elem = document.getElementById('para1');
  ```

  注意，该方法的参数是大小写敏感的。比如，如果某个节点的`id`属性是`main`，那么`document.getElementById('Main')`将返回`null`。

- `document.createElement`方法用来生成元素节点，并返回该节点。

  ```
  var newDiv = document.createElement('div');
  ```

  `createElement`方法的参数为元素的标签名，即元素节点的`tagName`属性，对于 HTML 网页大小写不敏感，即参数为`div`或`DIV`返回的是同一种节点。如果参数里面包含尖括号（即`<`和`>`）会报错。

- `document.createTextNode`方法用来生成文本节点（`Text`实例），并返回该节点。它的参数是**文本节点的内容**。

  ```js
  var newDiv = document.createElement('div');
  var newContent = document.createTextNode('Hello');
  newDiv.appendChild(newContent);
  ```

  上面代码新建一个`div`节点和一个文本节点，然后将文本节点插入`div`节点。

  这个**方法可以确保返回的节点，被浏览器当作文本渲染，而不是当作 HTML 代码渲染**。因此，可以用来展示用户的输入，避免 XSS 攻击（攻击载荷：标签中的 ">" 可以用 "//" 代替）。`createTextNode`方法对大于号和小于号进行转义，从而保证即使用户输入的内容包含恶意代码，也能正确显示。

  需要注意的是，该方法不对单引号和双引号转义，所以**不能用来对 HTML 属性赋值**。

  ```js
  function escapeHtml(str) {
    var div = document.createElement('div');
    div.appendChild(document.createTextNode(str));
    return div.innerHTML;
  };
  
  var userWebsite = '" onmouseover="alert(\'derp\')" "';
  var profileLink = '<a href="' + escapeHtml(userWebsite) + '">Bob</a>';
  var div = document.getElementById('target');
  div.innerHTML = profileLink;
  // <a href="" onmouseover="alert('derp')" "">Bob</a> 这里onmouseover中的双引号仍然存在
  ```

  上面代码中，由于`createTextNode`方法不转义双引号，导致`onmouseover`方法被注入了代码。



## Node接口

https://wangdoc.com/javascript/dom/node.html

### 一些Node的属性

- Node.prototype.baseURI：返回当前网页的绝对路径，可以通过使用 `<base href="xxx">` 改变该属性的值

- Node.prototype.nextSibling：返回紧跟在当前节点后面的第一个同级节点，可用于遍历所有子节点，包括文本节点和注释节点

- Node.prototype.previousSibling：返回当前节点前面的同级节点，包括文本节点和注释节点

- Node.prototype.parentNode：返回当前节点的父节点，对于一个节点来说，其父节点只有可能是三种类型：元素节点（element）、文档节点（document）和文档片段节点（documentfragment）

  ```js
  if (node.parentNode) {
      node.parentNode.removeChile(node);
  }
  ```

  以上代码是从父节点中移除当前节点的方法。

- Node.prototype.parentElement：当前节点的父元素节点。由于父节点只可能是三种类型：元素节点、文档节点（document）和文档片段节点（documentfragment），`parentElement`属性相当于把后两种父节点都排除了。

- Node.prototype.firstChild：返回当前节点的第一个子节点，也可能返回文本节点和注释节点

- Node.prototype.lastChild：返回当前节点的最后一个子节点，也可能返回文本节点和注释节点

- Node.prototype.childNodes：返回一个类似数组的对象，即 NodeList 集合，可通过其遍历所有子节点

  ```
  var div = document.getElementById('div1');
  var children = div.childNodes;
  
  for (var i = 0; i < children.length; i++) {
    // ...
  }
  ```

  文档节点（document）就有两个子节点：文档类型节点（docType）和 HTML 根元素节点。

  ```
  var children = document.childNodes;
  for (var i = 0; i < children.length; i++) {
    console.log(children[i].nodeType);
  }
  // 10
  // 1
  ```

  上面代码中，文档节点的第一个子节点的类型是10（即文档类型节点），第二个子节点的类型是1（即元素节点）。

- Node.prototype.isConnected：返回一个布尔值，表示当前节点是否在文档中



### 一些Node的实例方法

- `appendChild()`方法：接受一个节点对象作为参数，将其作为最后一个子节点，插入当前节点。该方法的返回值就是插入文档的子节点。如果参数节点是 DOM 已经存在的节点，`appendChild()`方法会将其从原来的位置，移动到新位置。

- `hasChildNodes`方法：返回一个布尔值，表示当前节点是否有子节点。包括所有类型的节点，并不仅仅是元素节点。哪怕节点只包含一个空格，`hasChildNodes`方法也会返回`true`。

  判断一个节点有没有子节点，有许多种方法，下面是其中的三种。

  - `node.hasChildNodes()`
  - `node.firstChild !== null`
  - `node.childNodes && node.childNodes.length > 0`

  `hasChildNodes`方法结合`firstChild`属性和`nextSibling`属性，可以遍历当前节点的所有后代节点。

  ```js
  function DOMComb(parent, callback) {
    if (parent.hasChildNodes()) {
      for (var node = parent.firstChild; node; node = node.nextSibling) {
        DOMComb(node, callback);
      }
    }
    callback(parent);
  }
  
  // 用法
  DOMComb(document.body, console.log)
  ```

  上面代码中，`DOMComb`函数的第一个参数是某个指定的节点，第二个参数是回调函数。这个回调函数会依次作用于指定节点，以及指定节点的所有后代节点。

- `cloneNode`方法：用于克隆一个节点。它接受一个布尔值作为参数，表示是否同时克隆子节点。它的返回值是一个克隆出来的新节点。

  （1）克隆一个节点，会拷贝该节点的所有属性，但是会丧失`addEventListener`方法和`on-`属性（即`node.onclick = fn`），添加在这个节点上的事件回调函数。

  （2）该方法返回的节点不在文档之中，即没有任何父节点，必须使用诸如`Node.appendChild`这样的方法添加到文档之中。

  （3）克隆一个节点之后，DOM 有可能出现两个有相同`id`属性（即`id="xxx"`）的网页元素，这时应该修改其中一个元素的`id`属性。如果原节点有`name`属性，可能也需要修改。

- `insertBefore`方法：用于将某个节点插入父节点内部的指定位置。

  ```
  var insertedNode = parentNode.insertBefore(newNode, referenceNode);
  ```

  `insertBefore`方法接受两个参数，第一个参数是所要插入的节点`newNode`，第二个参数是父节点`parentNode`内部的一个子节点`referenceNode`。`newNode`将插在`referenceNode`这个子节点的前面。返回值是插入的新节点`newNode`。

  由于不存在`insertAfter`方法，如果新节点要插在父节点的某个子节点后面，可以用`insertBefore`方法结合`nextSibling`属性模拟。

  ```
  parent.insertBefore(s1, s2.nextSibling);
  ```

- `removeChild`方法：接受一个子节点作为参数，用于从当前节点移除该子节点。返回值是移除的子节点。下面是如何移除当前节点的所有子节点。

  ```
  var element = document.getElementById('top');
  while (element.firstChild) {
    element.removeChild(element.firstChild);
  }
  ```

  被移除的节点依然存在于内存之中，但不再是 DOM 的一部分。所以，一个节点移除以后，**依然可以使用它**，比如插入到另一个节点下面。

- `replaceChild`方法：用于将一个新的节点，替换当前节点的某一个子节点。

  ```
  var replacedNode = parentNode.replaceChild(newChild, oldChild);
  ```

- `contains`方法：返回一个布尔值，表示参数节点是否满足以下三个条件之一。

  - 参数节点为当前节点。
  - 参数节点为当前节点的子节点。
  - 参数节点为当前节点的后代节点。

  ```
  document.body.contains(node)
  ```

  上面代码检查参数节点`node`，是否包含在当前文档之中。

  注意，当前节点传入`contains`方法，返回`true`。

  ```
  nodeA.contains(nodeA) // true
  ```

- `isEqualNode`方法：返回一个布尔值，用于检查两个节点是否相等。所谓相等的节点，指的是两个节点的类型相同、属性相同、子节点相同。
- `isSameNode`方法：返回一个布尔值，表示两个节点是否为同一个节点。

- `normalize`方法：用于清理当前节点内部的所有文本节点（text）。它会去除空的文本节点，并且将毗邻的文本节点合并成一个，也就是说不存在空的文本节点，以及毗邻的文本节点。

- `getRootNode()`方法：返回当前节点所在文档的根节点`document`，与`ownerDocument`属性的作用相同。

  ```
  document.body.firstChild.getRootNode() === document
  // true
  document.body.firstChild.getRootNode() === document.body.firstChild.ownerDocument
  // true
  ```

  该方法可用于`document`节点自身，这一点与`document.ownerDocument`不同。

  ```
  document.getRootNode() // document
  document.ownerDocument // null
  ```



## NodeList接口

- `NodeList`可以包含各种类型的节点，`HTMLCollection`只能包含 HTML 元素节点。

- `NodeList`实例是一个**类似数组的对象**（可以使用length属性和forEach方法，但不能push、pop等），它的成员是节点对象。通过以下方法可以得到`NodeList`实例。
  - `Node.childNodes`
  - `document.querySelectorAll()`等节点搜索方法

- 若要让它使用数组方法，可以将其转换为真正的数组：`Array.prototype.slice.call(nodeList)`

- 注意，NodeList 实例可能是动态集合，也可能是静态集合。所谓动态集合就是一个活的集合，DOM 删除或新增一个相关节点，都会立刻反映在 NodeList 实例。目前，只有**`Node.childNodes`返回的是一个动态集合**，其他的 NodeList 都是静态集合:

  ```js
  var children = document.body.childNodes;
  children.length // 18
  document.body.appendChild(document.createElement('p'));
  children.length // 19
  ```



### 实例方法

- `item`方法：接受一个整数值作为参数，表示成员的位置，返回该位置上的成员。

  ```
  document.body.childNodes.item(0)
  ```

  上面代码中，`item(0)`返回第一个成员。

  如果参数值大于实际长度，或者索引不合法（比如负数），`item`方法返回`null`。如果省略参数，`item`方法会报错。

  所有类似数组的对象，都可以使用方括号运算符取出成员。一般情况下，都是使用方括号运算符，而不使用`item`方法。

  ```js
  document.body.childNodes[0]
  ```

- NodeList.prototype.keys()，NodeList.prototype.values()，NodeList.prototype.entries()方法：

  这三个方法都返回一个 ES6 的遍历器对象，可以通过`for...of`循环遍历获取每一个成员的信息。区别在于，`keys()`返回键名的遍历器，`values()`返回键值的遍历器，`entries()`返回的遍历器同时包含键名和键值的信息。

  ```js
  var children = document.body.childNodes;
  
  for (var key of children.keys()) {
    console.log(key);
  }
  // 0
  // 1
  // 2
  // ...
  
  for (var value of children.values()) {
    console.log(value);
  }
  // #text
  // <script>
  // ...
  
  for (var entry of children.entries()) {
    console.log(entry);
  }
  // Array [ 0, #text ]
  // Array [ 1, <script> ]
  // ...
  ```



## HTMLCollection接口

- `HTMLCollection`是一个节点对象的集合，只能包含元素节点（element），不能包含其他类型的节点。它的返回值是一个类似数组的对象，但是与`NodeList`接口不同，`HTMLCollection`没有`forEach`方法，只能使用`for`循环遍历。

- 返回`HTMLCollection`实例的，主要是一些`Document`对象的集合属性，比如`document.links`、`document.forms`、`document.images`等。

- `HTMLCollection`实例都是**动态集合**，节点的变化会实时反映在集合中。

  如果元素节点有`id`或`name`属性，那么`HTMLCollection`实例上面，可以使用`id`属性或`name`属性引用该节点元素。如果没有对应的节点，则返回`null`。

  ```js
  // HTML 代码如下
  // <img id="pic" src="http://example.com/foo.jpg">
  
  var pic = document.getElementById('pic');
  document.images.pic === pic // true
  ```

- 实例方法：`namedItem`方法的参数是一个字符串，表示`id`属性或`name`属性的值，返回当前集合中对应的元素节点。如果没有对应的节点，则返回`null`。

  ```js
  // HTML 代码如下
  // <img id="pic" src="http://example.com/foo.jpg">
  
  var pic = document.getElementById('pic');
  document.images.namedItem('pic') === pic // true
  ```

  `Collection.namedItem('value')`等同于`Collection['value']`。



## ParentNode接口

如果当前节点是父节点，就会混入了（mixin）`ParentNode`接口。由于只有元素节点（element）、文档节点（document）和文档片段节点（documentFragment）拥有子节点，因此只有这三类节点会拥有`ParentNode`接口。

### 属性

- `children`属性返回一个`HTMLCollection`实例，成员是当前节点的所有元素子节点。该属性只读。
- `firstElementChild`属性返回当前节点的第一个元素子节点。如果没有任何元素子节点，则返回`null`。
- `lastElementChild`属性返回当前节点的最后一个元素子节点，如果不存在任何元素子节点，则返回`null`。
- `childElementCount`属性返回一个整数，表示当前节点的所有元素子节点的数目。如果不包含任何元素子节点，则返回`0`。

### 实例方法

- **ParentNode.append()**

  `append()`方法为当前节点追加一个或多个子节点，位置是最后一个元素子节点的后面。

  该方法不仅可以添加元素子节点（参数为元素节点），还可以添加文本子节点（参数为字符串）。

  ```js
  var parent = document.body;
  
  // 添加元素子节点
  var p = document.createElement('p');
  parent.append(p);
  
  // 添加文本子节点
  parent.append('Hello');
  
  // 添加多个元素子节点
  var p1 = document.createElement('p');
  var p2 = document.createElement('p');
  parent.append(p1, p2);
  
  // 添加元素子节点和文本子节点
  var p = document.createElement('p');
  parent.append('Hello', p);
  ```

  该方法没有返回值。

  注意，该方法与`Node.prototype.appendChild()`方法有三点不同。

  - `append()`允许**字符串**作为参数，`appendChild()`只允许子**节点**作为参数。
  - `append()`没有返回值，而`appendChild()`返回添加的子节点。
  - `append()`可以添加**多个子节点和字符串**（即允许多个参数），`appendChild()`只能添加一个节点（即只允许一个参数）。



## ChildNode接口

如果一个节点有父节点，那么该节点就拥有了`ChildNode`接口。

### 实例方法

- `remove()`方法用于从父节点移除当前节点。

  ```
  el.remove()
  ```

  上面代码在 DOM 里面移除了`el`节点。

- **（1）ChildNode.before()**

  `before()`方法用于在当前节点的前面，插入一个或多个同级节点。两者拥有相同的父节点。

  注意，该方法不仅可以插入元素节点，还可以插入文本节点。

  ```js
  var p = document.createElement('p');
  var p1 = document.createElement('p');
  
  // 插入元素节点
  el.before(p);
  
  // 插入文本节点
  el.before('Hello');
  
  // 插入多个元素节点
  el.before(p, p1);
  
  // 插入元素节点和文本节点
  el.before(p, 'Hello');
  ```

  **（2）ChildNode.after()**

  `after()`方法用于在当前节点的后面，插入一个或多个同级节点，两者拥有相同的父节点。用法与`before`方法完全相同。

- `replaceWith()`方法使用参数节点，替换当前节点。参数可以是元素节点，也可以是文本节点。

  ```js
  var span = document.createElement('span');
  el.replaceWith(span);
  ```

  上面代码中，`el`节点将被`span`节点替换。