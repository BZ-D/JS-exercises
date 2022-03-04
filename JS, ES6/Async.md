#### 单线程模型

JavaScript 只在一个线程上运行，其他任务都必须在后面排队等待。但这并不代表 JS 引擎只有一个线程，实际上它有多个线程，单个脚本只能在一个线程上运行，其他线程都是在后台配合。之所以不采用多线程，和历史有关系，单线程也是这门语言的核心特征之一。

- 好处：实现起来比较简单、执行环境相对单纯
- 坏处：只要有任意一个任务耗时很长，后边的任务都必须排队，导致浏览器假死、页面卡顿等问题。JS 语言本身并不慢，慢的是读写外部数据。
- 解决方案：事件循环机制，CPU 可以在任务进行 IO 操作时将其挂起，先运行排在后边的任务，待执行完成后再回头执行挂起的任务。单线程模型一方面是一种限制，另一方面让程序不会阻塞，这就是 Node JS 可以用很少的资源应付大流量访问的原因



#### 同步任务和异步任务

- 同步任务：没有被引擎挂起，在主线程上排队执行的任务，需要依次执行
- 异步任务：被引擎挂起、不进入主线程而进入任务队列的任务，只有引擎认为某个异步任务可以执行了（如 Ajax 操作从服务器得到了数据），该任务（采用回调函数的形式）才会进入主线程执行。而排在异步任务后面的代码，不用等待异步任务结束会马上运行，也就是说异步任务不具有阻塞效应



#### 任务队列和事件循环

JavaScript 运行时，除了一个正在运行的主线程，引擎还提供一个任务队列（task queue），里面是各种需要当前程序处理的异步任务。（实际上，根据异步任务的类型，存在多个任务队列。为了方便理解，这里假设只存在一个队列。）

首先，**主线程会去执行所有的同步任务**。等到同步任务全部执行完，就会去看任务队列里面的异步任务。**如果满足条件，那么异步任务就重新进入主线程开始执行，这时它就变成同步任务了**。等到执行完，下一个异步任务再进入主线程开始执行。一旦任务队列清空，程序就结束执行。

**异步任务的写法通常是回调函数。一旦异步任务重新进入主线程，就会执行对应的回调函数**。如果一个异步任务没有回调函数，就不会进入任务队列，也就是说，不会重新进入主线程，因为没有用回调函数指定下一步的操作。

JavaScript 引擎怎么知道异步任务有没有结果，能不能进入主线程呢？答案就是**引擎在不停地检查，一遍又一遍，只要同步任务执行完了，引擎就会去检查那些挂起来的异步任务**，是不是可以进入主线程了。这种循环检查的机制，就叫做事件循环（Event Loop）。[维基百科](https://en.wikipedia.org/wiki/Event_loop)的定义是：“事件循环是一个程序结构，用于等待和发送消息和事件（a programming construct that waits for and dispatches events or messages in a program）”。



#### 异步操作的模式

**1、回调函数**

回调函数是异步操作最基本的方法。

下面是两个函数`f1`和`f2`，编程的意图是`f2`必须等到`f1`执行完成，才能执行。

```js
function f1() {
  // ...
}

function f2() {
  // ...
}

f1();
f2();
```

上面代码的问题在于，如果`f1`是异步操作，`f2`会立即执行，不会等到`f1`结束再执行。

这时，可以考虑改写`f1`，把`f2`写成`f1`的回调函数。

```js
function f1(callback) {
  // ...
  callback();
}

function f2() {
  // ...
}

f1(f2);
```

回调函数的优点是简单、容易理解和实现，缺点是不利于代码的阅读和维护，各个部分之间高度[耦合](https://en.wikipedia.org/wiki/Coupling_(computer_programming))（coupling），使得程序结构混乱、流程难以追踪（尤其是多个回调函数嵌套的情况），而且每个任务只能指定一个回调函数。



**2、事件监听**

另一种思路是采用事件驱动模式。异步任务的执行不取决于代码的顺序，而取决于某个事件是否发生。

还是以`f1`和`f2`为例。首先，为`f1`绑定一个事件（这里采用的 jQuery 的[写法](https://api.jquery.com/on/)）。

```js
f1.on('done', f2);
```

上面这行代码的意思是，当`f1`发生`done`事件，就执行`f2`。然后，对`f1`进行改写：

```js
function f1() {
  setTimeout(function () {
    // ...
    f1.trigger('done');
  }, 1000);
}
```

上面代码中，`f1.trigger('done')`表示，执行完成后，立即触发`done`事件，从而开始执行`f2`。

这种方法的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以“[去耦合](https://en.wikipedia.org/wiki/Decoupling)”（decoupling），有利于实现模块化。缺点是**整个程序都要变成事件驱动型**，运行流程会变得很不清晰。阅读代码的时候，很难看出主流程。



**3、发布/订阅**

事件完全可以理解成“信号”，如果存在一个“信号中心”，某个任务执行完成，就向信号中心“发布”（publish）一个信号，其他任务可以向信号中心“订阅”（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做”[发布/订阅模式](https://en.wikipedia.org/wiki/Publish-subscribe_pattern)”（publish-subscribe pattern），又称“[观察者模式](https://en.wikipedia.org/wiki/Observer_pattern)”（observer pattern）。

这个模式有多种[实现](https://msdn.microsoft.com/en-us/magazine/hh201955.aspx)，下面采用的是 Ben Alman 的 [Tiny Pub/Sub](https://gist.github.com/661855)，这是 jQuery 的一个插件。

首先，`f2`向信号中心`jQuery`订阅`done`信号。

```js
jQuery.subscribe('done', f2);
```

然后，`f1`进行如下改写。

```js
function f1() {
  setTimeout(function () {
    // ...
    jQuery.publish('done');
  }, 1000);
}
```

上面代码中，`jQuery.publish('done')`的意思是，`f1`执行完成后，向信号中心`jQuery`发布`done`信号，从而引发`f2`的执行。

`f2`完成执行后，可以取消订阅（unsubscribe）。

```js
jQuery.unsubscribe('done', f2);
```

这种方法的性质与“事件监听”类似，但是明显优于后者。因为可以通过查看“消息中心”，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。



#### 异步操作的流程控制

如果有多个异步操作，就存在一个流程控制的问题：如何确定异步操作执行的顺序，以及如何保证遵守这种顺序。

```js
function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1秒后返回结果');
  setTimeout(function () { callback(arg * 2); }, 1000);
}
```

上面代码的`async`函数是一个异步任务，非常耗时，每次执行需要1秒才能完成，然后再调用回调函数。

如果有六个这样的异步任务，需要全部完成后，才能执行最后的`final`函数。请问应该如何安排操作流程？

```js
function final(value) {
  console.log('完成: ', value);
}

async(1, function (value) {
  async(2, function (value) {
    async(3, function (value) {
      async(4, function (value) {
        async(5, function (value) {
          async(6, final);
        });
      });
    });
  });
});
// 参数为 1 , 1秒后返回结果
// 参数为 2 , 1秒后返回结果
// 参数为 3 , 1秒后返回结果
// 参数为 4 , 1秒后返回结果
// 参数为 5 , 1秒后返回结果
// 参数为 6 , 1秒后返回结果
// 完成:  12
```

上面代码中，六个回调函数的嵌套，不仅写起来麻烦，容易出错，而且难以维护。



**1、串行执行**

我们可以编写一个流程控制函数，让它来控制异步任务，一个任务完成以后，再执行另一个。这就叫串行执行。

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];

function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1秒后返回结果');
  setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
  console.log('完成: ', value);
}

function series(item) {
  if(item) {
    async( item, function(result) {
      results.push(result);
      return series(items.shift());
    });
  } else {
    return final(results[results.length - 1]);
  }
}

series(items.shift());
```

上面代码中，函数`series`就是串行函数，它会依次执行异步任务，所有任务都完成后，才会执行`final`函数。`items`数组保存每一个异步任务的参数，`results`数组保存每一个异步任务的运行结果。

注意，上面的写法需要六秒，才能完成整个脚本。



**2、并行执行**

流程控制函数也可以是并行执行，即所有异步任务同时执行，等到全部完成以后，才执行`final`函数。

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];

// 定义一个异步任务
function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1秒后返回结果');
  setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
  console.log('完成: ', value);
}

items.forEach(function(item) {
  // forEach若传入异步任务，则并行执行
  async(item, function(result){
    results.push(result);
    if(results.length === items.length) {
      final(results[results.length - 1]);
    }
  })
});
```

上面代码中，`forEach`方法会同时发起六个异步任务，等到它们全部完成以后，才会执行`final`函数。

相比而言，上面的写法只要一秒，就能完成整个脚本。这就是说，并行执行的效率较高，比起串行执行一次只能执行一个任务，较为节约时间。但是问题在于如果并行的任务较多，很容易耗尽系统资源，拖慢运行速度。因此有了第三种流程控制方式。



**3、并行与串行结合**

所谓并行与串行的结合，就是设置一个门槛，每次最多只能并行执行`n`个异步任务，这样就避免了过分占用系统资源。

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];
var running = 0;
var limit = 2;

function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1秒后返回结果');
  setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
  console.log('完成: ', value);
}

function launcher() {
  while(running < limit && items.length > 0) {
    var item = items.shift();
    async(item, function(result) {
      results.push(result);
      running--;
      if(items.length > 0) {
        launcher();
      } else if(running == 0) {
        final(results);
      }
    });
    running++;
  }
}

launcher();
```

上面代码中，最多只能同时运行两个异步任务。变量`running`记录当前正在运行的任务数，只要低于门槛值，就再启动一个新的任务，如果等于`0`，就表示所有任务都执行完了，这时就执行`final`函数。

这段代码需要三秒完成整个脚本，处在串行执行和并行执行之间。通过调节`limit`变量，达到效率和资源的最佳平衡。



#### 定时器

JavaScript 提供定时执行代码的功能，叫做定时器（timer），主要由`setTimeout()`和`setInterval()`这两个函数来完成。它们向任务队列添加定时任务。

**setTimeout()**

`setTimeout`函数用来指定某个函数或某段代码，**在多少毫秒之后执行**。它返回一个整数，表示定时器的编号，以后可以用来取消这个定时器。

```js
var timerId = setTimeout(func|code, delay);
```

上面代码中，`setTimeout`函数接受两个参数，第一个参数`func|code`是将要推迟执行的函数名或者一段代码，第二个参数`delay`是推迟执行的毫秒数。

```js
console.log(1);
setTimeout('console.log(2)',1000);
console.log(3);
// 1
// 3
// 2
```

上面代码会先输出1和3，然后等待1000毫秒再输出2。注意，`console.log(2)`必须以字符串的形式，作为`setTimeout`的参数。

如果推迟执行的是函数，就直接将函数名，作为`setTimeout`的参数。

```js
function f() {
  console.log(2);
}

setTimeout(f, 1000);
```

`setTimeout`的第二个参数如果省略，则默认为0。

```js
setTimeout(f)
// 等同于
setTimeout(f, 0)
```

除了前两个参数，`setTimeout`还允许更多的参数。它们将依次传入推迟执行的函数（回调函数）。

```js
setTimeout(function (a,b) {
  console.log(a + b);
}, 1000, 1, 1);
```

上面代码中，`setTimeout`共有4个参数。最后那两个参数，将在1000毫秒之后回调函数执行时，作为回调函数的参数。

还有一个需要注意的地方，**如果回调函数是对象的方法，那么`setTimeout`使得方法内部的`this`关键字指向全局环境，而不是定义时所在的那个对象**。

```js
var x = 1;

var obj = {
  x: 2,
  y: function () {
    console.log(this.x);
  }
};

setTimeout(obj.y, 1000) // 1
```

上面代码输出的是1，而不是2。因为当`obj.y`在1000毫秒后运行时，`this`所指向的已经不是`obj`了，而是全局环境。

为了防止出现这个问题，一种解决方法是将`obj.y`放入一个函数。

```js
var x = 1;

var obj = {
  x: 2,
  y: function () {
    console.log(this.x);
  }
};

setTimeout(function () {
  obj.y();
}, 1000);
// 2
```

上面代码中，`obj.y`放在一个匿名函数之中，这使得`obj.y`在`obj`的作用域执行，而不是在全局作用域内执行，所以能够显示正确的值。

另一种解决方法是，使用`bind`方法，将`obj.y`这个方法绑定在`obj`上面。

```js
var x = 1;

var obj = {
  x: 2,
  y: function () {
    console.log(this.x);
  }
};

setTimeout(obj.y.bind(obj), 1000)
// 2
```

## setInterval()

`setInterval`函数的用法与`setTimeout`完全一致，区别仅仅在于`setInterval`指定某个任务每隔一段时间就执行一次，也就是无限次的定时执行。

```js
var i = 1
var timer = setInterval(function() {
  console.log(2);
}, 1000)
```

上面代码中，每隔1000毫秒就输出一个2，会无限运行下去，直到关闭当前窗口。

与`setTimeout`一样，除了前两个参数，`setInterval`方法还可以接受更多的参数，它们会传入回调函数。

下面是一个通过`setInterval`方法实现网页动画的例子。

```js
var div = document.getElementById('someDiv');
var opacity = 1;
var fader = setInterval(function() {
  opacity -= 0.1;
  if (opacity >= 0) {
    div.style.opacity = opacity;
  } else {
    clearInterval(fader);
  }
}, 100);
```

上面代码每隔100毫秒，设置一次`div`元素的透明度，直至其完全透明为止。

`setInterval`的一个常见用途是实现轮询。下面是一个轮询 URL 的 Hash 值是否发生变化的例子。

```js
var hash = window.location.hash;
var hashWatcher = setInterval(function() {
  if (window.location.hash != hash) {
    updatePage();
  }
}, 1000);
```

`setInterval`指定的是**“开始执行”之间的间隔**，并不考虑每次任务执行本身所消耗的时间。因此实际上，两次执行之间的间隔会小于指定的时间。比如，`setInterval`指定每 100ms 执行一次，每次执行需要 5ms，那么第一次执行结束后95毫秒，第二次执行就会开始。如果某次执行耗时特别长，比如需要105毫秒，那么它结束后，下一次执行就会立即开始。

为了确保两次执行之间有固定的间隔，可以不用`setInterval`，而是**每次执行结束后，使用`setTimeout`指定下一次执行的具体时间**。

```js
var i = 1;
var timer = setTimeout(function f() {
  // ...
  timer = setTimeout(f, 2000);
}, 2000);
```

上面代码可以确保，下一次执行总是在本次执行结束之后的2000毫秒开始。

## clearTimeout()，clearInterval()

`setTimeout`和`setInterval`函数，都返回一个整数值，表示计数器编号。将该整数传入`clearTimeout`和`clearInterval`函数，就可以取消对应的定时器。

```js
var id1 = setTimeout(f, 1000);
var id2 = setInterval(f, 1000);

clearTimeout(id1);
clearInterval(id2);
```

上面代码中，回调函数`f`不会再执行了，因为两个定时器都被取消了。

`setTimeout`和`setInterval`返回的整数值是连续的，也就是说，第二个`setTimeout`方法返回的整数值，将比第一个的整数值大1。

```js
function f() {}
setTimeout(f, 1000) // 10
setTimeout(f, 1000) // 11
setTimeout(f, 1000) // 12
```

上面代码中，连续调用三次`setTimeout`，返回值都比上一次大了1。

利用这一点，可以写一个函数，取消当前所有的`setTimeout`定时器。

```js
(function() {
  // 每轮事件循环检查一次
  var gid = setInterval(clearAllTimeouts, 0);

  function clearAllTimeouts() {
    var id = setTimeout(function() {}, 0);
    while (id > 0) {
      if (id !== gid) {
        clearTimeout(id);
      }
      id--;
    }
  }
})();
```

上面代码中，先调用`setTimeout`，得到一个计算器编号，然后把编号比它小的计数器全部取消。

## 实例：debounce 函数 ※

有时，我们不希望回调函数被频繁调用。比如，用户填入网页输入框的内容，希望通过 Ajax 方法传回服务器，jQuery 的写法如下。

```js
$('textarea').on('keydown', ajaxAction);
```

这样写有一个很大的缺点，就是如果用户连续击键，就会连续触发`keydown`事件，造成大量的 Ajax 通信。这是不必要的，而且很可能产生性能问题。**正确的做法应该是，设置一个门槛值，表示两次 Ajax 通信的最小间隔时间。如果在间隔时间内，发生新的`keydown`事件，则不触发 Ajax 通信，并且重新开始计时**。如果过了指定时间，没有发生新的`keydown`事件，再将数据发送出去。

这种做法叫做 debounce（防抖动）。假定两次 Ajax 通信的间隔不得小于2500毫秒，上面的代码可以改写成下面这样。

```js
$('textarea').on('keydown', debounce(ajaxAction, 2500));

function debounce(fn, delay){
  var timer = null; // 声明计时器
  return function() {
    var context = this;
    var args = arguments;
    clearTimeout(timer);
    timer = setTimeout(function () {
      fn.apply(context, args);
    }, delay);
  };
}
```

上面代码中，只要在2500毫秒之内，用户再次击键，就会取消上一次的定时器，然后再新建一个定时器。这样就保证了回调函数之间的调用间隔，至少是2500毫秒。

## 运行机制

`setTimeout`和`setInterval`的运行机制，是**将指定的代码移出本轮事件循环，等到下一轮事件循环，再检查是否到了指定时间**。如果到了，就执行对应的代码；如果不到，就继续等待。

这意味着，`setTimeout`和`setInterval`指定的回调函数，**必须等到本轮事件循环的所有同步任务都执行完，才会开始执行**。由于前面的任务到底需要多少时间执行完，是不确定的，**所以没有办法保证，`setTimeout`和`setInterval`指定的任务，一定会按照预定时间执行。**

```js
setTimeout(someTask, 100);
veryLongTask();
```

上面代码的`setTimeout`，指定100毫秒以后运行一个任务。但是，如果后面的`veryLongTask`函数（同步任务）运行时间非常长，过了100毫秒还无法结束，那么被推迟运行的`someTask`就只有等着，等到`veryLongTask`运行结束，才轮到它执行。

再看一个`setInterval`的例子。

```js
setInterval(function () {
  console.log(2);
}, 1000);

sleep(3000);

function sleep(ms) {
  var start = Date.now();
  while ((Date.now() - start) < ms) {
  }
}
```

上面代码中，`setInterval`要求每隔1000毫秒，就输出一个2。但是，紧接着的`sleep`语句需要3000毫秒才能完成，那么`setInterval`就必须推迟到3000毫秒之后才开始生效。注意，生效后`setInterval`不会产生累积效应，即不会一下子输出三个2，而是只会输出一个2。

## setTimeout(f, 0)

### 含义

`setTimeout`的作用是将代码推迟到指定时间执行，如果指定时间为`0`，即`setTimeout(f, 0)`，那么会立刻执行吗？

**答案是不会。因为上一节说过，必须要等到当前脚本的同步任务，全部处理完**以后，才会执行`setTimeout`指定的回调函数`f`。也就是说，`setTimeout(f, 0)`会**在下一轮事件循环一开始就执行**。

**优先级：同步 > 任务队列（排队中的异步任务）**

```js
setTimeout(function () {
  console.log(1);
}, 0);
console.log(2);
// 2
// 1
```

上面代码先输出`2`，再输出`1`。因为`2`是同步任务，在本轮事件循环执行，而`1`是下一轮事件循环执行。

总之，`setTimeout(f, 0)`这种写法的目的是，**尽可能早地执行`f`**，但是并不能保证立刻就执行`f`。

实际上，`setTimeout(f, 0)`不会真的在0毫秒之后运行，不同的浏览器有不同的实现。以 Edge 浏览器为例，会等到4毫秒之后运行。如果电脑正在使用电池供电，会等到16毫秒之后运行；如果网页不在当前 Tab 页，会推迟到1000毫秒（1秒）之后运行。这样是为了节省系统资源。

### 应用

`setTimeout(f, 0)`有几个非常重要的用途。它的一大应用是，可以调整事件的发生顺序。比如，网页开发中，某个事件先发生在子元素，然后冒泡到父元素，即**子元素的事件回调函数，会早于父元素的事件回调函数触发**。如果，想让父元素的事件回调函数先发生，就要用到`setTimeout(f, 0)`。

```js
// HTML 代码如下
// <input type="button" id="myButton" value="click">

// 属于事件监听型异步任务
var input = document.getElementById('myButton');

input.onclick = function A() {
  // 看起来是立即执行，实际上是降低了其执行的优先级
  setTimeout(function B() {
    input.value +=' input';
  }, 0)
};

document.body.onclick = function C() {
  input.value += ' body'
};
```

上面代码在点击按钮后，先触发回调函数`A`，然后触发函数`C`。函数`A`中，`setTimeout`将函数`B`推迟到下一轮事件循环执行，这样就起到了，先触发父元素的回调函数`C`的目的了。

另一个应用是，**用户自定义的回调函数，通常在浏览器的默认动作之前触发**。比如，用户在输入框输入文本，`keypress`事件会在浏览器接收文本之前触发。因此，下面的回调函数是达不到目的的。

```js
// HTML 代码如下
// <input type="text" id="input-box">

document.getElementById('input-box').onkeypress = function (event) {
  this.value = this.value.toUpperCase();
}
```

上面代码想在用户每次输入文本后，立即将字符转为大写。但是实际上，它只能将本次输入前的字符转为大写，因为浏览器此时还没接收到新的文本，所以`this.value`取不到最新输入的那个字符。只有用`setTimeout`改写，上面的代码才能发挥作用。

```js
document.getElementById('input-box').onkeypress = function() {
  var self = this;
  // 使用setTimeOut(f, 0)进行改写
  setTimeout(function() {
    self.value = self.value.toUpperCase();
  }, 0);
}
```

上面代码将代码放入`setTimeout`之中，就能使得它在浏览器接收到文本之后触发。

由于`setTimeout(f, 0)`实际上意味着，**将任务放到浏览器最早可得的空闲时段执行**，所以那些计算量大、耗时长的任务，常常会被放到几个小部分，分别放到`setTimeout(f, 0)`里面执行。

```js
var div = document.getElementsByTagName('div')[0];

// 写法一
for (var i = 0xA00000; i < 0xFFFFFF; i++) {
  div.style.backgroundColor = '#' + i.toString(16);
}

// 写法二
var timer;
var i=0x100000;

function func() {
  timer = setTimeout(func, 0);
  div.style.backgroundColor = '#' + i.toString(16);
  if (i++ == 0xFFFFFF) clearTimeout(timer);
}

timer = setTimeout(func, 0);
```

上面代码有两种写法，都是改变一个网页元素的背景色。写法一会造成浏览器“堵塞”，因为 JavaScript 执行速度远高于 DOM，会造成大量 DOM 操作“堆积”，而写法二就不会，这就是`setTimeout(f, 0)`的好处。

另一个使用这种技巧的例子是代码高亮的处理。如果代码块很大，一次性处理，可能会对性能造成很大的压力，那么将其分成一个个小块，一次处理一块，比如写成`setTimeout(highlightNext, 50)`的样子，性能压力就会减轻。



### 让多个Promise实例串行执行

- 在 promiseChain 方法中，主体为递归函数 cur
- 采用递归的原因：保证函数执行顺序，保证 promiseList 数组中的 promise 实例都能串行执行
- 如果用 for、forEach 等方法进行遍历，实际上是让多个 promise 实例并行执行了

```js
const promise1 = Promise.resolve(1);
const promise2 = new Promise(resolve => {
  setTimeout(() => {
    resolve(2);
  }, 3000);
});
const promise3 = new Promise(resolve => {
  setTimeout(() => {
    resolve(3);
  }, 2000);
});
const promiseList = [promise1, promise2, promise3];

function promiseChain(promiseList) {
  let index = 0;
  const results = [];
  return new Promise((resolve => {
    const recur = () => {  // 递归函数主体
      promiseList[index].then(res => {  // 对当前promise实例调用then()
        console.log(res);  // 打印res
        index++;
        results.push(res);
        if (index === promiseList.length) {  // 所有promise实例都执行完毕
          return resolve(results);
        }
        recur();  // 递归调用，保证promise实例是串行执行的
      })
    }
    recur();
  }))
}

let p = promiseChain(promiseList)
p.then(res => {
  console.log('所有 Promise 执行完毕。');
  console.log(...res)
});
```

