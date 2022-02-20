## GlobalEventHandler接口

参见：https://wangdoc.com/javascript/events/globaleventhandlers.html



## EventTarget接口

DOM 节点的事件操作（监听和触发），都定义在`EventTarget`接口。所有节点对象都部署了这个接口，其他一些需要事件通信的浏览器内置对象（比如，`XMLHttpRequest`、`AudioNode`、`AudioContext`）也部署了这个接口。

### 实例方法

- addEventListener(type, listener[, useCapture])

  - type：事件名称，大小写敏感

  - listener：监听函数

    - 也可以是一个具有handleEvent方法的对象：

      ```js
      btnElement.addEventListener('click', {
          handleEvent: function()
          {
              ...
          }
      });
      ```

  - useCapture：如果为true，表示监听函数将捕获阶段触发，否则在冒泡阶段触发

    - 还可以是一个对象，包括如下属性：
      - capture：作用同useCapture
      - once：只监听一次
      - passive：布尔，true时表示禁止监听函数调用preventDefault方法
      - signal

  - 如果希望向监听函数传参，可以用匿名函数包装监听函数：

    `el.addEventListener('click', function (x) {...}, false);`

  - 监听函数内部的this指向当前事件所在的对象：

    ```js
    / HTML 代码如下
    // <p id="para">Hello</p>
    var para = document.getElementById('para');
    para.addEventListener('click', function (e) {
      console.log(this.nodeName); // "P"
    }, false);
    ```

    上面代码中，监听函数内部的`this`指向事件所在的对象`para`。

- removeEventListener(type, listener, useCapture)，用法和上面一样
- dispatchEvent(event)：在当前节点上触发指定事件，从而触发监听函数，该方法返回一个布尔值，只要有一个监听函数调用了 preventDefault，则返回值为 false



## 事件模型

浏览器的事件模型，就是通过监听函数（listener）对事件做出反应。事件发生后，浏览器监听到了这个事件，就会执行对应的监听函数。这是事件驱动编程模式（event-driven）的主要编程方式。

### 绑定监听函数的方法

- HTML 的 on- 属性，如 onclick="xxx()"，注意立即执行括号不要少
  - 只会在冒泡阶段触发
  - 可以通过 setAttribute 函数设置此属性
- 元素节点的事件属性，如 `div.onclick = funtion() {...}`
- EventTarget.addEventListener 方法
  - 推荐使用，优点：
  - 同一个事件可以添加多个监听器
  - 能指定在哪个阶段触发监听函数
  - 是JS统一的监听函数接口

### 事件的传播

#### 一个事件发生后传播的三个阶段：

1. 从window对象传导到目标节点：捕获阶段
2. 在目标节点上触发事件：目标阶段
3. 从目标节点传导回window对象：冒泡阶段

这种三阶段的传播模型，使得同一个事件会在多个节点上触发。

```html
<div>
  <p>点击</p>
</div>
```

上面代码中，`<div>`节点之中有一个`<p>`节点。

如果对这两个节点，都设置`click`事件的监听函数（每个节点的捕获阶段和冒泡阶段，各设置一个监听函数），共计设置四个监听函数。然后，对`<p>`点击，`click`事件会触发四次。

```js
var phases = {
  1: 'capture',
  2: 'target',
  3: 'bubble'
};

var div = document.querySelector('div');
var p = document.querySelector('p');

div.addEventListener('click', callback, true);
p.addEventListener('click', callback, true);
div.addEventListener('click', callback, false);
p.addEventListener('click', callback, false);

function callback(event) {
  var tag = event.currentTarget.tagName;
  var phase = phases[event.eventPhase];
  console.log("Tag: '" + tag + "'. EventPhase: '" + phase + "'");
}

// 点击以后的结果
// Tag: 'DIV'. EventPhase: 'capture'
// Tag: 'P'. EventPhase: 'target'
// Tag: 'P'. EventPhase: 'target'
// Tag: 'DIV'. EventPhase: 'bubble'
```

上面代码表示，`click`事件被触发了四次：`<div>`节点的捕获阶段和冒泡阶段各1次，`<p>`节点的目标阶段触发了2次。

1. 捕获阶段：事件从`<div>`向`<p>`传播时，触发`<div>`的`click`事件；
2. 目标阶段：事件从`<div>`到达`<p>`时，触发`<p>`的`click`事件；
3. 冒泡阶段：事件从`<p>`传回`<div>`时，再次触发`<div>`的`click`事件。

其中，`<p>`节点有两个监听函数（`addEventListener`方法第三个参数的不同，会导致绑定两个监听函数），因此它们都会因为`click`事件触发一次。所以，`<p>`会在`target`阶段有两次输出。

注意，**浏览器总是假定`click`事件的目标节点，就是点击位置嵌套最深的那个节点**（本例是`<div>`节点里面的`<p>`节点）。所以，**`<p>`节点的捕获阶段和冒泡阶段，都会显示为`target`阶段**。

事件传播的最上层对象是`window`，接着依次是`document`，`html`（`document.documentElement`）和`body`（`document.body`）。也就是说，上例的事件传播顺序，在捕获阶段依次为`window`、`document`、`html`、`body`、`div`、`p`，在冒泡阶段依次为`p`、`div`、`body`、`html`、`document`、`window`。



### 事件的代理

由于事件会**在冒泡阶段向上传播到父节点**，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件。这种方法叫做事件的代理（delegation）。比如，按钮存在嵌套结构时，将监听函数放在外层节点上

```js
var ul = document.querySelector('ul');

ul.addEventListener('click', function (event) {
  if (event.target.tagName.toLowerCase() === 'li') {
    // some code
  }
});
```

上面代码中，`click`事件的监听函数定义在`<ul>`节点，但是实际上，它处理的是子节点`<li>`的`click`事件。这样做的好处是，只要定义一个监听函数，就能处理多个子节点的事件，而不用在每个`<li>`节点上定义监听函数。而且以后再添加子节点，监听函数依然有效。

如果希望事件到某个节点为止，不再传播，可以使用事件对象的`stopPropagation`方法。

```js
// 事件传播到 p 元素后，就不再向下传播了
p.addEventListener('click', function (event) {
  event.stopPropagation();
}, true);  // 捕获阶段触发

// 事件冒泡到 p 元素后，就不再向上冒泡了
p.addEventListener('click', function (event) {
  event.stopPropagation();
}, false);  // 冒泡阶段触发
```

上面代码中，`stopPropagation`方法分别在捕获阶段和冒泡阶段，阻止了事件的传播。

但是，`stopPropagation`方法只会阻止事件的传播，不会阻止该事件触发`<p>`节点的其他`click`事件的监听函数。也就是说，不是彻底取消`click`事件。

```js
p.addEventListener('click', function (event) {
  event.stopPropagation();
  console.log(1);
});

p.addEventListener('click', function(event) {
  // 会触发
  console.log(2);
});
```

上面代码中，`p`元素绑定了两个`click`事件的监听函数。`stopPropagation`方法只能阻止这个事件的传播，不能取消这个事件，因此，第二个监听函数会触发。输出结果会先是1，然后是2。

如果想要彻底取消该事件，不再触发后面所有`click`的监听函数，可以使用`stopImmediatePropagation`方法。

```js
p.addEventListener('click', function (event) {
  event.stopImmediatePropagation();
  console.log(1);
});

p.addEventListener('click', function(event) {
  // 不会被触发
  console.log(2);
});
```

上面代码中，`stopImmediatePropagation`方法可以彻底取消这个事件，使得后面绑定的所有`click`监听函数都不再触发。所以，只会输出1，不会输出2。



### Event对象

`Event`构造函数接受两个参数。第一个参数`type`是字符串，表示事件的名称；第二个参数`options`是一个对象，表示事件对象的配置。该对象主要有下面两个属性。

- `bubbles`：布尔值，可选，默认为`false`，表示事件对象是否冒泡。
- `cancelable`：布尔值，可选，默认为`false`，表示事件是否可以被取消，即能否用`Event.preventDefault()`取消这个事件。一旦事件被取消，就好像从来没有发生过，不会触发浏览器对该事件的默认行为。

```js
var ev = new Event(
  'look',
  {
    'bubbles': true,
    'cancelable': false
  }
);
document.dispatchEvent(ev);
```

上面代码新建一个`look`事件实例，然后使用`dispatchEvent`方法触发该事件。

注意，如果不是显式指定`bubbles`属性为`true`，生成的事件就只能在“捕获阶段”触发监听函数。

```js
// HTML 代码为
// <div><p>Hello</p></div>
var div = document.querySelector('div');
var p = document.querySelector('p');

function callback(event) {
  var tag = event.currentTarget.tagName;
  console.log('Tag: ' + tag); // 没有任何输出
}

div.addEventListener('click', callback, false);

var click = new Event('click');
p.dispatchEvent(click);
```

上面代码中，`p`元素发出一个`click`事件，该事件默认不会冒泡。`div.addEventListener`方法指定在冒泡阶段监听，因此监听函数不会触发。如果写成`div.addEventListener('click', callback, true)`，那么在“捕获阶段”可以监听到这个事件。

另一方面，如果这个事件在`div`元素上触发。

```
div.dispatchEvent(click);
```

那么，不管`div`元素是在冒泡阶段监听，还是在捕获阶段监听，都会触发监听函数。因为这时`div`元素是事件的目标，不存在是否冒泡的问题，`div`元素总是会接收到事件，因此导致监听函数生效。

#### 属性

- bubbles：布尔，表示当前事件是否会冒泡，默认不冒泡
- eventPhase：整数常量，表示事件所处阶段。（0：未发生；1：捕获阶段；2：目标阶段；3：冒泡阶段）
- cancelable：事件是否可以取消，如果不可以，调用 preventDefault 是无效的
- cancelBubble：如果将其设为true，相当于执行 stopPropagation，可以阻止事件传播
- defaultPrevented：布尔，表示该事件是否调用过 preventDefault 方法
- currentTarget：事件当前所在的节点，总是等同于监听函数内部的 this
- target：原始触发事件的节点
- type：返回事件类型，可以是系统提供的类型，也可以是用户自定义类型
- timeStamp：毫秒时间戳，表示事件发生的时间
- isTrusted：布尔，表示该事件是否由真人触发



#### 实例方法

- preventDefault：取消浏览器对当前事件的默认行为，如跳转页面、按空格键滚动页面等行为，调用此方法的前提是，事件对象的 cancelable 属性为真。此方法不会阻断事件传播。可用于阻止用户选中、输入文本等行为
- stopPropagation：阻止事件在DOM中继续传播，防止触发别的节点上定义的监听器。不会阻止事件在当前节点的传播
- stopImmediatePropagation：阻止同一个事件的其他监听函数被调用，比上者阻止得更彻底
- composedPath：返回一个数组，成员是事件的最底层节点和依次冒泡经过的所有上层节点



## 其他常见事件

- `beforeunload`事件在窗口、文档、各种资源将要卸载前触发。它可以用来防止用户不小心卸载资源。

  如果该事件对象的`returnValue`属性是一个非空字符串，那么浏览器就会弹出一个对话框，询问用户是否要卸载该资源。但是，用户指定的字符串可能无法显示，浏览器会展示预定义的字符串。如果用户点击“取消”按钮，资源就不会卸载。

  ```
  window.addEventListener('beforeunload', function (event) {
    event.returnValue = '你确定离开吗？';
  });
  ```

  上面代码中，用户如果关闭窗口，浏览器会弹出一个窗口，要求用户确认。

- `unload`事件在窗口关闭或者`document`对象将要卸载时触发。它的触发顺序排在`beforeunload`、`pagehide`事件后面。

  `unload`事件发生时，文档处于一个特殊状态。所有资源依然存在，但是对用户来说都不可见，UI 互动全部无效。这个事件是无法取消的，即使在监听函数里面抛出错误，也不能停止文档的卸载。

- `load`事件在页面或某个资源加载成功时触发。注意，页面或资源从浏览器缓存加载，并不会触发`load`事件。

  ```
  window.addEventListener('load', function(event) {
    console.log('所有资源都加载完成');
  });
  ```

  `error`事件是在页面或资源加载失败时触发。`abort`事件在用户取消加载时触发。

  这三个事件实际上属于进度事件，不仅发生在`document`对象，还发生在各种外部资源上面。

- `popstate`事件在浏览器的`history`对象的当前记录发生显式切换时触发。注意，调用`history.pushState()`或`history.replaceState()`，并不会触发`popstate`事件。该事件只在用户在`history`记录之间显式切换时触发，比如鼠标点击“后退/前进”按钮，或者在脚本中调用`history.back()`、`history.forward()`、`history.go()`时触发。

- `hashchange`事件在 URL 的 hash 部分（即`#`号后面的部分，包括`#`号）发生变化时触发。该事件一般在`window`对象上监听。

  `hashchange`的事件实例具有两个特有属性：`oldURL`属性和`newURL`属性，分别表示变化前后的完整 URL。

  ```js
  // URL 是 http://www.example.com/
  window.addEventListener('hashchange', myFunction);
  
  function myFunction(e) {
    console.log(e.oldURL);
    console.log(e.newURL);
  }
  
  location.hash = 'part2';
  // http://www.example.com/
  // http://www.example.com/#part2
  ```

- `scroll`事件在文档或文档元素滚动时触发，主要出现在用户拖动滚动条。

  ```
  window.addEventListener('scroll', callback);
  ```

  该事件会连续地大量触发，所以它的监听函数之中不应该有非常耗费计算的操作。推荐的做法是使用`requestAnimationFrame`或`setTimeout`控制该事件的触发频率，然后可以结合`customEvent`抛出一个新事件。

  改用`setTimeout()`方法，可以放置更大的时间间隔。

  ```js
  (function() {
    window.addEventListener('scroll', scrollThrottler, false);
  
    var scrollTimeout;
    function scrollThrottler() {
      if (!scrollTimeout) {
        scrollTimeout = setTimeout(function () {
          scrollTimeout = null;
          actualScrollHandler();
        }, 66);
      }
    }
  
    function actualScrollHandler() {
      // ...
    }
  }());
  ```

  上面代码中，每次`scroll`事件都会执行`scrollThrottler`函数。该函数里面有一个定时器`setTimeout`，每66毫秒触发一次（每秒15次）真正执行的任务`actualScrollHandler`。

  下面是一个更一般的`throttle`函数的写法。

  ```js
  function throttle(fn, wait) {
    var time = Date.now();
    return function() {
      if ((time + wait - Date.now()) < 0) {
        fn();
        time = Date.now();
      }
    }
  }
  
  window.addEventListener('scroll', throttle(callback, 1000));
  ```

  上面的代码将`scroll`事件的触发频率，限制在一秒一次。

  `lodash`函数库提供了现成的`throttle`函数，可以直接使用。

  ```
  window.addEventListener('scroll', _.throttle(callback, 1000));
  ```

  本书前面介绍过`debounce`的概念，`throttle`与它区别在于，`throttle`是“节流”，确保一段时间内只执行一次，而`debounce`是“防抖”，要连续操作结束后再执行。以网页滚动为例，`debounce`要等到用户停止滚动后才执行，`throttle`则是如果用户一直在滚动网页，那么在滚动过程中还是会执行。



## 焦点事件

焦点事件发生在元素节点和`document`对象上面，与获得或失去焦点相关。它主要包括以下四个事件。

- `focus`：元素节点获得焦点后触发，该事件不会冒泡。
- `blur`：元素节点失去焦点后触发，该事件不会冒泡。
- `focusin`：元素节点将要获得焦点时触发，发生在`focus`事件之前。该事件会冒泡。
- `focusout`：元素节点将要失去焦点时触发，发生在`blur`事件之前。该事件会冒泡。

这四个事件的事件对象都继承了`FocusEvent`接口。`FocusEvent`实例具有以下属性。

- `FocusEvent.target`：事件的目标节点。
- `FocusEvent.relatedTarget`：对于`focusin`事件，返回失去焦点的节点；对于`focusout`事件，返回将要接受焦点的节点；对于`focus`和`blur`事件，返回`null`。

由于`focus`和`blur`事件不会冒泡，只能在捕获阶段触发，所以`addEventListener`方法的第三个参数需要设为`true`。

```js
form.addEventListener('focus', function (event) {
  event.target.style.background = 'pink';
}, true);

form.addEventListener('blur', function (event) {
  event.target.style.background = '';
}, true);
```

上面代码针对表单的文本输入框，接受焦点时设置背景色，失去焦点时去除背景色。



## 鼠标事件

参见：https://wangdoc.com/javascript/events/mouse.html



## 键盘事件

参见：https://wangdoc.com/javascript/events/keyboard.html



## 进度事件

参见：https://wangdoc.com/javascript/events/progress.html



## 表单事件

参见：https://wangdoc.com/javascript/events/form.html



## 触摸事件

参见：https://wangdoc.com/javascript/events/touch.html



## 拖拉事件

参见：https://wangdoc.com/javascript/events/drag.html

