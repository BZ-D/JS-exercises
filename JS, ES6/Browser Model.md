## 链接

https://wangdoc.com/javascript/bom/engine.html

- `javascript:` 协议：常见用途是书签脚本 BookMarklet，由于浏览器书签保存的是一个网址，所以 `javascript:` 网址也可以保存在里边，当用户选择这个书签的时候，脚本也会运行。为了防止书签替换掉当前文档，可以在脚本前加上`void`，或者在脚本最后加上`void 0`。

  ```html
  <a href="javascript: void new Date().toLocaleTimeString();">点击</a>
  <a href="javascript: new Date().toLocaleTimeString();void 0;">点击</a>
  ```

  上面这两种写法，点击链接后，执行代码都不会网页跳转。



### 一、script元素

#### 工作原理

**1、网页加载流程**

- 浏览器一边下载HTML网页，一边开始解析，也就是说，不等到下载完就开始解析
- 解析过程中，如果发现了 `<script>` 元素，就暂停解析渲染，把网页渲染的控制权转交给 JS 引擎（原因：**JS代码可能修改DOM，必须将控制权交给JS引擎，否则出现线程竞赛问题**）
- 如果 `<script>` 元素引用了外部脚本，就下载该脚本再执行，否则直接执行代码
- JS引擎执行完毕该代码，控制权交还渲染引擎，恢复往下解析HTML网页



**2、浏览器假死**

**阻塞效应**是由于外部脚本一直无法完成下载导致加载时间过长的情况，为了避免这种情况，较好的方式是将 `<script>` 标签放到页面底部，即使脚本失去响应，网页主体也已经渲染完成了。而这种方式也有一个好处，在DOM结构生成之前，JS调用DOM节点会报错，如果脚本在网页尾部加载，就不存在这个问题。如果需要把脚本放在头部，且需要解决此问题，则：

- 一种解决方法是设定`DOMContentLoaded`事件的回调函数。

  ```html
  <head>
    <script>
      document.addEventListener(
        'DOMContentLoaded',
        function (event) {
          console.log(document.body.innerHTML);
        }
      );
    </script>
  </head>
  ```

  上面代码中，指定`DOMContentLoaded`事件发生后，才开始执行相关代码。`DOMContentLoaded`事件只有在 DOM 结构生成之后才会触发。

- 另一种解决方法是，使用`<script>`标签的`onload`属性。当`<script>`标签指定的外部脚本文件下载和解析完成，会触发一个`load`事件，可以把所需执行的代码，放在这个事件的回调函数里面。

  ```html
  <script src="jquery.min.js" onload="console.log(document.body.innerHTML)">
  </script>
  ```

  但是，如果将脚本放在页面底部，就可以完全按照正常的方式写，上面两种方式都不需要。



**3、多个script标签的执行顺序**

同时并行下载多个脚本，但会保证按照标签顺序从上到下执行，即使后面的脚本先下载完成。



**4、阻塞效应的避免方法**

- 如上文介绍的将脚本放在页面尾部
- 解析和执行 CSS，也会产生阻塞。Firefox 浏览器会**等到脚本前面的所有样式表，都下载并解析完，再执行脚本**；Webkit则是**一旦发现脚本引用了样式，就会暂停执行脚本，等到样式表下载并解析完，再恢复执行**。
- 此外，对于来自同一个域名的资源，比如脚本文件、样式表文件、图片文件等，浏览器一般有限制，同时最多下载6～20个资源，即**最多同时打开的 TCP 连接有限制**，这是为了防止对服务器造成太大压力。如果是来自不同域名的资源，就没有这个限制。所以，**通常把静态文件放在不同的域名之下，以加快下载速度**。

**此外的一些解决方法，见下面介绍：**



**5、defer属性**

为了解决脚本文件下载阻塞网页渲染的问题，一个方法是对`<script>`元素加入`defer`属性。它的作用是延迟脚本的执行，等到 DOM 加载生成后，再执行脚本。

```
<script src="a.js" defer></script>
<script src="b.js" defer></script>
```

上面代码中，只有等到 DOM 加载完成后，才会执行`a.js`和`b.js`。

**运行流程**

- 浏览器解析HTML网页
- 解析过程中发现带有 defer 属性的 script 元素
- 浏览器就继续往下执行 HTML 网页，同时并行下载 script 元素加载的外部脚本
- **完成解析** HTML 网页，再回过头来执行已下载完成的脚本

注：对于内置而不是加载外部资源的标签，defer 属性不起作用



**6、async属性**

解决“阻塞效应”的另一个方法是对`<script>`元素加入`async`属性。

```
<script src="a.js" async></script>
<script src="b.js" async></script>
```

`async`属性的作用是，**使用另一个进程下载脚本，下载时不会阻塞渲染**。

1. 浏览器开始解析 HTML 网页。
2. 解析过程中，发现带有`async`属性的`script`标签。
3. 浏览器继续往下解析 HTML 网页，同时并行下载`<script>`标签中的外部脚本。
4. **脚本下载完成**，浏览器暂停解析 HTML 网页，开始执行下载的脚本。
5. **脚本执行完毕**，浏览器恢复解析 HTML 网页。

`async`属性可以保证脚本下载的同时，浏览器继续渲染。需要注意的是，一旦采用这个属性，**就无法保证脚本的执行顺序。哪个脚本先下载结束，就先执行那个脚本**。另外，使用`async`属性的脚本文件里面的代码，不应该使用`document.write`方法。

`defer`属性和`async`属性到底应该使用哪一个？

一般来说，如果脚本之间没有依赖关系，就使用`async`属性，如果脚本之间有依赖关系，就使用`defer`属性。如果同时使用`async`和`defer`属性，后者不起作用，浏览器行为由`async`属性决定。



**7、脚本动态加载**

script元素还可以动态生成，生成后再插入页面，称为动态加载：

```js
['a.js', 'b.js'].forEach(function(src) {
  var script = document.createElement('script');
  script.src = src;
  document.head.appendChild(script);
});
```

这种方法的好处是，动态生成的`script`标签不会阻塞页面渲染，也就不会造成浏览器假死。但是问题在于，这种方法**无法保证脚本的执行顺序**，哪个脚本文件先下载完成，就先执行哪个。

如果想避免这个问题，**可以设置async属性为`false`**。

```js
['a.js', 'b.js'].forEach(function(src) {
  var script = document.createElement('script');
  script.src = src;
  script.async = false;  // 设置async为false
  document.head.appendChild(script);
});
```

上面的代码不会阻塞页面渲染，而且可以保证`b.js`在`a.js`后面执行。不过需要注意的是，在这段代码后面加载的脚本文件，会因此都等待`b.js`执行完成后再执行。

如果想为动态加载的脚本指定回调函数，可以使用下面的写法。

```js
function loadScript(src, done) {
  var js = document.createElement('script');
  js.src = src;
  js.onload = function() {
    done();
  };
  js.onerror = function() {
    done(new Error('Failed to load script ' + src));
  };
  document.head.appendChild(js);
}
```



**8、加载使用的协议**

如果不指定协议，浏览器默认采用 HTTP 协议下载。

```
<script src="example.js"></script>
```

上面的`example.js`默认就是采用 HTTP 协议下载，如果要采用 HTTPS 协议下载，必需写明。

```
<script src="https://example.js"></script>
```

但是有时我们会希望，根据页面本身的协议来决定加载协议，这时可以采用下面的写法。

```
<script src="//example.js"></script>
```



### 二、浏览器的组成

两部分：**渲染引擎** 和 **JavaScript 解释器（又称 JavaScript 引擎）**

#### 1、渲染引擎

- 作用：将网页代码渲染为用户视觉可感知的平面文档

- 处理阶段：

  1. 解析代码：HTML代码解析为DOM，CSS代码解析为CSSOM
  2. 对象合成：将DOM和CSSOM合成为一棵渲染树
  3. 布局：计算出渲染树的布局 layout
  4. 绘制：将渲染树绘制到屏幕

  以上四步并非严格按照顺序执行，往往第一步还没完成，第二三步就开始了



#### 2、回流和重绘

渲染树转换为网页布局，称为布局流（flow），布局显示到页面的过程称为绘制，它们都具有阻塞效应，消耗很多时间和资源。

页面生成之后，**脚本操作和样式样式表操作都会触发回流和重绘**，用户动作也会触发，如**鼠标悬停、页面滚动、输入文本、改变窗口大小**等。

回流和重绘并不一定一起发生，但**回流必然导致重绘**。如改变元素颜色只会导致重绘，改变元素布局一定发生回流和重绘。

作为开发者，应该尽量设法降低重绘的次数和成本，尽量不要变动高层的DOM元素，以底层DOM元素的变动为代替。重绘 table 布局和 flex 布局开销都较大。

**优化技巧：**

- 读取DOM或写入DOM尽量写在一起，不要混杂，不要读写交叉
- 缓存DOM信息
- 不要一项一项地改变样式，而是使用 CSS class 一次性改变
- 使用 documentFragment 操作 DOM，该对象实例不是 DOM 的一部分，在全部操作完之后，再将该实例的子节点插入原 DOM 中
- 动画使用 absolute 定位或 fixed 定位，减少对其他元素的影响
- 只在必要时才显示隐藏的元素
- 使用 `window.requestAnimationFrame()`，可以把代码推迟到下次重绘之前执行，而不是立即要求重绘
- 使用虚拟 DOM 库

下面是一个`window.requestAnimationFrame()`对比效果的例子。

```js
// 回流代价高
function doubleHeight(element) {
  var currentHeight = element.clientHeight;
  element.style.height = (currentHeight * 2) + 'px';
}

all_my_elements.forEach(doubleHeight);

// 重绘代价低
function doubleHeight(element) {
  var currentHeight = element.clientHeight;

  window.requestAnimationFrame(function () {
    element.style.height = (currentHeight * 2) + 'px';
  });
}

all_my_elements.forEach(doubleHeight);
```

上面的第一段代码，**每读一次 DOM，就写入新的值**，会造成不停的重排和重流。第二段代码**把所有的写操作，都累积在一起**，从而 DOM 代码变动的代价就最小化了。



#### 3、JavaScript 引擎

- 作用：读取网页中的 JS 代码，对其处理后运行
- JS 是一种解释型语言，不需要编译，由解释器实时运行。优点是运行和修改比较方便，刷新页面就可以重新解释；缺点是每次运行都要调用解释器，系统开销大，运行速度慢于编译型语言
- JIT：Just In Time，即时编译。为了提高运行速度，目前的浏览器将 JS 代码进行一定程度的编译，生成类似字节码的中间代码。在 JIT 下，字节码只在运行时编译，用到哪一行就编译哪一行，并把编译结果缓存。而这些字节码运行在 JavaScript 引擎上
- 例子：Chrome V8