#### 函数——第一等公民

- JS将函数看做一种值，与其他值地位相同，凡是可以使用值的地方，就能使用函数

  ```js
  function add(x, y) {
      return x + y;
  }
  
  var operator = add;
  
  function a(op) {
      return op;
  }
  a(add)(1, 3);  // 4
  ```

- 函数名提升，由于函数是第一等公民，因此整个函数会像变量声明一样，提升到代码头部

  ```js
  f();
  function f() {}
  // 不会报错
  ```

  但是如果采用赋值语句定义函数，JS就会报错

  ```js
  f();
  var f = function (){};
  
  // 相当于
  var f;
  f();  // 此时f为undefined，因此报错：f is not a function
  f = function (){};
  ```

  如果既采用function命令、又采用var赋值语句声明同一个函数，由于存在函数提升，最后会采用var赋值语句的定义

  ```js
  var f = function () {
      console.log(1);
  }
  function f() {
      console.log(2);
  }
  f();  // 1
  ```

- 函数的属性和方法

  - name属性：返回函数名。如果是通过赋值定义的匿名函数，返回其变量名；若为具名函数，返回function关键字之后的函数名
  - length属性：返回函数定义的参数个数
  - toString()方法：返回一个字符串，内容是函数的源码，包括注释



#### 函数变量的作用域

- 对于顶层函数来说，函数外部声明的变量就是全局变量，可以在函数内部读取；函数内部声明的变量为局部变量。函数内部定义的变量，会在局部作用域内覆盖同名全局变量

- 对于var命令来说，局部变量只能在函数内部声明。在其他区块中声明（如循环中），一律视为全局变量

  ```js
  for (var i = 0; i < 6; i++) {}
  console.log(i);  // 6
  ```

- 函数内部变量提升：与全局作用域一样，函数作用域内部也会出现变量提升现象

  ```js
  function f(x) {
      if (x > 100) {
          var tmp = x - 100;
      }
  }
  
  // similarly
  function f(x) {
      var tmp;
      if (x > 100) {
          tmp = x - 100;
      }
  }
  ```

- 函数本身作用域是独有的，与它运行时所在的作用域无关

  ```js
  var a = 1;
  var x = function () {
    console.log(a);
  };
  
  function f() {
    var a = 2;
    x();
  }
  
  f() // 1
  ```

  即函数执行时所在的作用域是定义时的作用域，而非调用时的

  ```js
  var x = function () {
    console.log(a);
  };
  
  function y(f) {
    var a = 2;
    f();
  }
  
  y(x)
  // ReferenceError: a is not defined
  ```

- 函数传参

  函数参数如果是原始类型值（数值、字符串、布尔值），传递方式是传值传递，即拷贝

  ```js
  var p = 2;
  function f(p) {
      p = 3;
  }
  f(p);
  p  // 2
  ```

  如果是复合类型值（数组、对象、其他函数），传值方式是按址传递，会影响到原始值

  ```js
  var obj = { p: 1 };
  function f(o) {
      o.p = 2;
  }
  f(obj);
  obj.p  // 2
  
  
  var arr = [1, 2, 3];
  function g(arr) {
      arr[2] = 7;
  }
  g(arr);
  arr[2];  // 7
  ```

  但如果整个替换掉了参数，不会影响到原始值，即此时是将变量指向了新地址

  ```js
  var obj = [1, 2, 3];
  
  function f(o) {
    o = [2, 3, 4];
  }
  f(obj);
  
  obj // [1, 2, 3]
  ```

- arguments对象

  - 包含了函数运行时所有参数，只有在函数体内部才能使用

    ```js
    var f = function (one) {
      console.log(arguments[0]);
      console.log(arguments[1]);
      console.log(arguments[2]);
    }
    
    f(1, 2, 3)
    // 1
    // 2
    // 3
    ```

  - 正常模式下，arguments对象可以在运行时修改，并会影响函数的参数；严格模式下，修改arguments对象也不会影响到实际的函数参数

    ```js
    var f = function(a, b) {
      arguments[0] = 3;
      arguments[1] = 2;
      return a + b;
    }
    
    f(1, 1) // 5
    
    
    var f = function(a, b) {
      'use strict'; // 开启严格模式
      arguments[0] = 3;
      arguments[1] = 2;
      return a + b;
    }
    
    f(1, 1) // 2
    ```

  - 虽然arguments看起来很像数组，但它是一个对象，数组专有方法（如slice和forEach）不能使用。若要将arguments转化为数组，有两种方法：

    - slice

      ```js
      var args = Array.prototype.slice.call(arguments);
      ```

    - 逐一填入新数组

      ```js
      var args = [];
      for (var i = 0; i < arguments.length; i++) {
          args.push(arguments[i]);
      }
      ```

  - callee属性：arguments对象带有一个callee属性，返回所对应的原函数。注意：此属性在严格模式中禁用

    ```js
    var f = function () {
        console.log(arguments.callee === f);
    }
    f();  // true
    ```



#### 闭包

要想从外部得到函数内部的变量，就需要引入闭包机制，在函数内部再定义一个函数。

```js
function f1() {
    var n = 111;
    function f2() {
        console.log(n);  // 111
    }
}
```

此时f1内部的所有变量对f2都是可见的，而f2的局部变量对f1是不可见的。

既然f2可以读取f1的局部变量，只要把f2作为返回值，就可以在f1外部读取其局部变量：

```js
function f1() {
    var n = 111;
    function f2() {
        console.log(n);
    }
    return f2;
}
var res = f1();
res();  // 111
```

这里的 **闭包** 就是函数`f2`，即能够读取其他函数内部变量的函数，闭包最大的特点就是能够记住诞生的环境，从而获取其中的局部变量（此过程会使这些变量一直保存在内存中）。此外，闭包还有封装对象私有属性和方法的作用：

```js
function Person(name) {
  var _age;
  function setAge(n) {
    _age = n;
  }
  function getAge() {
    return _age;
  }

  return {
    name: name,
    getAge: getAge,
    setAge: setAge
  };
}

var p1 = Person('张三');
p1.setAge(25);
p1.getAge() // 25
```

使用闭包的过程中需要注意：外层函数每次运行都会生成一个新的闭包，造成内存消耗很大，因此不能滥用闭包，否则造成网页性能问题。



#### 立即调用的函数表达式

```js
function() { ... }();
// 将会报错，SyntaxError: Unexpected token '('
// 报错原因是：function关键字既可以当语句又可以当表达式
            
function f() {}  // 语句
var f = function f() {}  // 表达式

// 当做表达式时，函数可以定义后直接加圆括号调用
var f = function f() { return 1; }();
f  // 1
```

函数定义后直接加圆括号调用，没有报错。原因是function作为表达式，引擎会把函数定义当做一个值。为避免解析的歧义，规定：如果function关键字出现在行首，一律解释成语句，因此引擎看到行首function关键字后，认为这一段都是函数定义，不应该以圆括号结尾，所以就报错了。

- 解决方案：
  - 注：两个语句后边的分号都是必须的，如果遇到连着两个立即调用的函数表达式（IIFE），可能就会报错

```js
(function(){ /* code */ }());
// 或
(function(){ /* code */ })();
```

- 通常情况下，只对匿名函数使用这种立即执行的函数表达式，避免全局变量污染，二是IIFE内部形成了一个单独作用域，可以封装一些私有变量

  ```js
  (function () {
      var tmp = newData;
      processData(tmp);
      storeData(tmp);
  }());
  ```

  