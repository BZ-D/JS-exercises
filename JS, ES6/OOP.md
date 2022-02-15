#### new 命令

其作用就是执行构造函数并返回实例对象。如果不使用 new 命令而直接调用构造函数，则构造函数变成了普通函数，并不会生成实例对象。

```js
var Vehicle = function () {
    this.price = 1000;
};

var v = Vehicle();
v  // undefined
price  // 1000
```

上面代码中没有使用new命令，导致 price 成为全局变量，而 v 不是实例对象。函数内部 this 指向全局。为了保证构造函数必须与 new 命令一起使用，一个解决办法是：构造函数内部使用严格模式，这样调用时不加 new 就会报错。另一个解决方法是，构造函数内部判断是否使用 new 命令，如果发现没有使用，则直接返回一个实例对象：

```js
function Fubar(foo, bar) {
 	if (!(this instanceof Fubar)) {
        return new Fubar(foo, bar);
    }
    this._foo = foo;
    this._bar = bar;
}
```

函数内部可以使用 new.target 属性，如果当前函数是 new 命令调用，new.target 指向当前函数，否则为 undefined：

```js
function f() {
    console.log(new.target === f);
}
f();  // false
new f();  // true
```



#### new 命令的原理

- 使用 new 命令时，其后面的函数依次执行下面的步骤：

1. 创建一个空对象，作为将要返回的对象实例
2. 将这个空对象的原型指向构造函数的 prototype 属性
3. 调用构造函数，并将这个空对象赋值给函数内部的 this 关键字
4. 开始执行构造函数内部的代码

- 也就是说，构造函数内部，this 指向一个新生成的空对象，所有针对 this 的操作，都会发生在这个空对象上。构造函数的目的就是操作一个空对象 this，将其构造为需要的样子。如果构造函数内部有 return 语句，且后面跟着一个对象，new 命令会返回 return 语句指定的对象，否则就不会管 return 语句，而返回 this 对象

- 手写一个 new 函数：

  ```js
  // 写法1
  function _new(constructor, ...args) {
      // 创建一个空对象
      const obj = {};
      // 将这个空对象原型指向构造函数的prototype属性
      obj._proto_ = constructor.prototype;
      // 调用构造函数
      const res = constructor.apply(obj, args);
      // 如果构造函数没有显式return，那么直接返回obj
      // 如果构造函数有return，且返回的不是一个对象，那么还是返回obj
      // 如果构造函数显式返回一个对象，那么返回此对象res
      return res instanceof Object ? res : obj;
  }
  
  
  // 写法2
  function _new(constructor, params) {
      // 将arguments对象转为数组
      var args = [].slice.call(arguments);
      // 取出构造函数
      var constructor = args.shift();
      // 创建一个空对象，继承构造函数的prototype属性
      var context = Object.create(constructor.prototype);
      // 执行构造函数
      var result = constructor.apply(context, args);
      return (typeof result === 'object' && result != null) ? result : context;
  }
  
  var actor = _new(Person, '张三', 28);
  ```

- 如果普通函数使用 new 命令，则返回一个空对象



#### Object.create() 创建实例对象

当拿不到构造函数、只能拿到一个现有的对象时，希望以现有对象为模板生成新的实例对象时，可以使用此方法：

```js
var person1 = {
  name: '张三',
  age: 38,
  greeting: function() {
    console.log('Hi! I\'m ' + this.name + '.');
  }
};

var person2 = Object.create(person1);

person2.name // 张三
person2.greeting() // Hi! I'm 张三.
```



#### this 关键字 ※

非常重要的语法点。

简单来说：this 就是属性或方法”当前“所在的对象。`this.property` 中，this 就代表 property 属性当前所在的对象。

```js
var person = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
    // this指向person
  }
};

person.describe()
// "姓名：张三"
```

- 由于对象的属性可以赋给另一个对象，所以属性所在的当前对象是可变的，即this的指向是可变的：

  ```js
  var A = {
    name: '张三',
    describe: function () {
      return '姓名：'+ this.name;
    }
  };
  
  var B = {
    name: '李四'
  };
  
  B.describe = A.describe;
  B.describe()
  // "姓名：李四"
  ```

  只要函数被复制给另一个变量，this 的指向就会改变。如果将对象成员函数赋值给一个全局变量，则函数中的 this 将指向顶层对象。

- **this 关键字的实质**

  JS 之所以有 this 的设计，与内存中的数据结构有关。当创建一个对象实例 obj 时，obj 实际上是指向该对象的内存地址，而原始对象以字典结构保存，每个属性名都对应一个属性描述对象：

  ```json
  {
    foo: {
      [[value]]: 5	// 保存了属性的值
      [[writable]]: true
      [[enumerable]]: true
      [[configurable]]: true
    }
  }
  ```

  若属性的值是一个函数，这时，引擎会将函数单独保存在内存中，然后再将函数地址赋给 foo 属性的 value 属性，即 `[[value]]` 值为函数的地址。由于函数是一个单独的值，所以可以在不同环境执行，因此需要有一种机制能够让函数体内部获得当前的运行环境，所以，this 出现了，用于指代函数当前运行环境。

- **this 的使用场合**

  - 全局环境：用于指向顶层对象 window

  - 构造函数：指向实例对象，在构造函数内部定义成员属性时，用 `this.xxx = vvv` 的形式

  - 对象的方法：如果对象的方法里包含 this，其指向就是方法运行时所在的对象，该方法赋给另一个对象时就会改变 this 的指向。但此条规则不容易把握：

    ```js
    var obj = {
        foo: function () {
            console.log(this);
        }
    }
    obj.foo()  // obj
    (obj.foo = obj.foo)()  // window
    (false || obj.foo)()  // window
    (1, obj.foo)()  // window
    ```

    三种打印出 window 的情况中，obj.foo 就是一个值，这个值真正调用的时候，运行环境已经不是 obj 了，而是全局环境。在引擎内部，obj 和 obj.foo 存储在两个内存地址中，称作地址1和地址2：

    - obj.foo() 这样调用时，是从地址1调用地址2，因此地址2的运行环境是地址1，this 指向obj
    - 但是 obj.foo 这种形式都是直接取出地址2进行调用，这样的话，运行环境就是全局环境

  - 如果 this 所在的方法不在对象的第一层，这时 this 只是指向当前一层的对象，不会继承更上面的层：

    ```js
    var a = {
      p: 'Hello',
      b: {
        m: function() {
          console.log(this.p);
        }
      }
    };
    
    a.b.m() // undefined
    ```

    如果将嵌套对象内部的函数赋值给一个全局变量，this 依然指向全局对象。为了避免这个问题，可以只将函数所在的对象赋值给全局变量，在调用函数时，this 的指向不会改变。

- **多层 this 嵌套问题**

  ```js
  var o = {
    f1: function () {
      console.log(this);
      var f2 = function () {
        console.log(this);
      }();
    }
  }
  
  o.f1()
  // Object
  // Window
  ```

  一个解决方法是在第二层改用一个指向外层 this 的变量

  ```js
  var o = {
    f1: function() {
      console.log(this);
      var that = this;
      var f2 = function() {
        console.log(that);
      }();
    }
  }
  
  o.f1()
  // Object
  // Object
  ```

  事实上，使用一个变量固定 this 的值，然后在内层函数调用此变量的操作很常见。

  **注：**严格模式下内部 this 指向全局对象时就会报错

- **避免数组处理方法中的 this**

  - 数组的 map 和 forEach 方法，允许提供一个函数作为参数，此函数内部不应该使用 this，因为此时内层 this 指向顶层对象：

    ```js
    var o = {
      v: 'hello',
      p: [ 'a1', 'a2' ],
      f: function f() {
        this.p.forEach(function (item) {
          // this指向顶层，因此访问不到 v
          console.log(this.v + ' ' + item);
        });
      }
    }
    
    o.f()
    // undefined a1
    // undefined a2
    ```

    解决方案仍然是用中间变量固定 this：

    ```js
    var o = {
      v: 'hello',
      p: [ 'a1', 'a2' ],
      f: function f() {
        var that = this;
        this.p.forEach(function (item) {
          console.log(that.v+' '+item);
        });
      }
    }
    
    o.f()
    // hello a1
    // hello a2
    ```

    另一种方法是将 this 作为 forEach 函数的第二个参数，固定其运行环境：

    ```js
    var o = {
      v: 'hello',
      p: [ 'a1', 'a2' ],
      f: function f() {
        this.p.forEach(function (item) {
          console.log(this.v + ' ' + item);
        }, this);
      }
    }
    
    o.f()
    // hello a1
    // hello a2
    ```

- 避免回调函数中的 this，因为往往会改变指向：

  ```js
  var o = new Object();
  o.f = function () {
      console.log(this === o);
  }
  
  $('#button').on('click', o.f);  // false
  ```

  此时 this 指向按钮的 DOM 对象，而非 o

- **绑定 this 的方法**

  - **Function.prototype.call()**，函数实例方法，可以指定函数内部 this 的指向，然后再所指定的作用域中调用该函数：`Function.prototype.call(obj, ...args)`

    ```js
    var obj = {};
    var f = function () {
        return this;
    };
    
    f() === window  // true
    f.call(obj) === obj
    ```

    call 方法参数应该是一个对象，如果参数为空、null 或 undefined，则默认传入 window。如果参数是一个原始值，则将原始值自动转成包装对象，然后传入 call 方法。后面一系列参数 args 是该函数调用时需要的一系列参数。

    - call 的一个典型应用是调用对象的原生方法

  - **Function.prototype.apply()**，与 call 类似，唯一区别是：接受一个数组作为函数执行的参数：`Function.prototype.apply(thisValue, [arg1, arg2, ...])`

    利用 apply 方法第二个参数需为数组的特点，可以进行一些应用：

    - 找出数组最大元素：`Math.max.apply(null, arr)`

    - 将数组的空元素变为 undefined：`Array.apply(null, ['a', ,'b'])`，利用 Array 构造函数将空元素变为 undefined

    - 转换类似数组的对象：`Array.prototype.slice.apply({ 0: 1, length: 1 })` => [1]

    - 绑定回调函数的对象：

      ```js
      var o = new Object();
      
      o.f = function () {
        console.log(this === o);
      }
      
      var f = function (){
        o.f.apply(o);
        // 或者 o.f.call(o);
      };
      
      // jQuery 的写法
      $('#button').on('click', f);  // true
      ```

      由于 apply 和 call 方法不仅绑定函数执行时所在对象，还会立即执行函数，因此不得不把绑定语句写在一个函数体内，更简洁的写法是下面介绍的 bind 函数

  - **Function.prototype.bind()**，用于将函数体内的 this 绑定到某个对象，然后返回一个 **新函数**：

    ```js
    var d = new Date();
    d.getTime() // 1481869925657
    
    var print = d.getTime;
    print() // Uncaught TypeError: this is not a Date object.
    ```

    getTime 方法内部的 this 先是绑定到 d 上，而后又绑定到了 window 上，而用 bind 可解决此问题：

    ```js
    var print = d.getTime.bind(d);
    print();
    ```

    将 getTime 方法内部的 this 绑定到 d 对象上，这时可以安全地将此方法赋值给其他变量了。bind 方法的参数就是所需要绑定 this 的对象，下面是一个更清晰的例子：

    ```js
    var counter = {
      count: 0,
      inc: function () {
        this.count++;
      }
    };
    
    var func = counter.inc.bind(counter);
    func();
    counter.count // 1
    
    // 也可以绑定到其他对象
    var obj = {
      count: 100
    };
    var func = counter.inc.bind(obj);
    func();
    obj.count // 101
    ```

    bind 还可以接受更多参数，作为绑定原函数的参数：

    ```js
    var add = function (x, y) {
      return x * this.m + y * this.n;
    }
    
    var obj = {
      m: 2,
      n: 2
    };
    
    var newAdd = add.bind(obj, 5);
    newAdd(5) // 20
    ```

    上面代码中，bind 方法除了绑定 this 对象，还将 add 函数的第一个参数 x 绑定为 5，然后返回一个新函数 newAdd，这个函数只接受一个参数 y 就能运行了。如果 bind 方法第一个参数是 null 或 undefined，等于将 this 绑定到 window 上

    - bind 方法使用的注意点

      - 每运行一次就会产生一个新函数，会产生一些问题，如监听事件的时候，不能写成：

        `el.addEventListener('click', o.m.bind(o));`

        click 事件绑定 bind 方法生成的一个匿名函数(o.m.bind(o))，会导致无法取消绑定，正确的写法应该是：

        `var listener = o.m.bind(o);`

        `el.addEventListener('click', listener);`

        `el.removeEventListener('click', listener);`

      - 结合回调函数使用，很常见的错误是将包含 this 的方法直接当作回调函数，解决办法是使用 bind 方法进行绑定：

        ```js
        var counter = {
          count: 0,
          inc: function () {
            'use strict';
            this.count++;
          }
        };
        
        function callIt(callback) {
          callback();
        }
        
        callIt(counter.inc.bind(counter));
        counter.count // 1
        ```

        将 counter.inc() 绑定 counter 上，使得 this 总是指向 counter

      - 某些数组方法可以接受一个函数当作参数，这些函数的内部 this 指向也很可能出错：

        ```js
        var obj = {
          name: '张三',
          times: [1, 2, 3],
          print: function () {
            this.times.forEach(function (n) {
              console.log(this.name);
            });
          }
        };
        
        obj.print()
        // 没有任何输出
        ```

        forEach 方法回调函数内部的 this 指向全局，没有办法取到值。为了解决这个问题，也是通过 bind 进行绑定：

        ```js
        obj.print = function () {
          this.times.forEach(function (n) {
            console.log(this.name);
          }.bind(this));
        };
        
        obj.print()
        // 张三
        // 张三
        // 张三
        ```

      - 结合 call 方法进行使用

        ```js
        [1, 2, 3].slice(0, 1) // [1]
        // 等同于
        Array.prototype.slice.call([1, 2, 3], 0, 1) // [1]
        ```

        call 方法实质是调用 Function.prototype.call 方法，而 bind 方法将会返回一个新函数，因此可以改写为：

        ```js
        var slice = Function.prototype.call.bind(Array.prototype.slice);
        slice([1, 2, 3], 0, 1) // [1]
        ```

        类似地，还有其他数组方法：

        ```js
        var push = Function.prototype.call.bind(Array.prototype.push);
        var pop = Function.prototype.call.bind(Array.prototype.pop);
        
        var a = [1 ,2 ,3];
        push(a, 4)
        a // [1, 2, 3, 4]
        
        pop(a)
        a // [1, 2, 3]
        ```

        再进一步，将 Function.prototype.call 方法内的 this 绑定到 Function.prototype.bind 对象，就意味着 bind 的调用形式也可以被改写：

        ```js
        function f() {
          console.log(this.v);
        }
        
        var o = { v: 123 };
        var bind = Function.prototype.call.bind(Function.prototype.bind);
        bind(f, o)() // 123
        ```

        上面的代码就是把 Function.prototype.bind 方法绑定到 Function.prototype.call 的 this 上，所以 bind 方法就可以直接使用，不需要在函数实例上使用

