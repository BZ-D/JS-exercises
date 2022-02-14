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

  

#### 数组

- 数组的本质是一种特殊的对象，`typeof` 运算符会返回数组的类型是 `object`

  - 特殊性体现在：它的键名是按次序排列的一组整数(0,1,2,...)
  - `Object.keys(arr)  // ['0', '1', '2']，数组的键名其实也是字符串`

- 清空数组的有效方法：将 length 属性设为 0

- 如果数组键名是添加超出范围的数值，该键名自动转为字符串

  ```js
  var arr = [];
  arr[-1] = 'a';
  arr[Math.pow(2, 32)] = 'b';
  
  arr.length  // 0
  arr[-1]  // 'a'
  arr[4294967296]  // 'b'
  ```

- in 运算符

  - 检查某个**键名**是否存在

    ```js
    var arr = ['a', 'b', 'c'];
    2 in arr  // true
    '2' in arr  // true
    4 in arr  // false
    ```

  - 若某个位置为空位，返回false

    ```js
    var arr = [];
    arr[100] = 1;
    
    100 in arr  // true
    1 in arr  // false
    ```

- 数组的空位：
  - var a = [1, , 3]  a.length === 3，即空位只在中间有效
  - var a = [1, 2, 3,]  a.length === 3，即空位在结尾无效
  - 若读取空位，返回undefined
  - 若用delete删除数组元素，会形成空位，但不影响length属性
  - forEach、for...in、Object.keys遍历数组时，自动跳过空位。注意若某位元素显示赋值为undefined，则遍历时不跳过



#### 类似数组的对象

如果一个对象所有键名都是非负整数，且有length属性，那么这个对象就很像数组，语法上称为类似数组的对象。(array-like object)

```js
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
};

obj[0] // 'a'
obj[1] // 'b'
obj.length // 3
obj.push('d') // TypeError: obj.push is not a function
```

类似数组的对象根本特征就是具有length属性，只要有它，就可以认为这个对象类似于数组，但length不是动态值，不会随着成员变化而变化。典型的类似数组的对象是函数的 `arguments` 对象，以及大多数 DOM 元素集，还有字符串。

- 数组的 slice 方法可以将类似数组的对象变成真正的数组：

  ```js
  var arr = Array.prototype.slice.call(arrayLike);
  ```

- 除了转为真正数组，类似数组的对象还有一个方法可以使用数组的方法，就是通过 `call()` 把数组的方法放到对象上面：

  ```js
  function print(value, index) {
    console.log(index + ' : ' + value);
  }
  
  Array.prototype.forEach.call(arrayLike, print);
  ```

  本来 arrayLike 是不可以用 forEach 方法的，但是通过 call，可以调用，对于字符串来讲也可以这样做：

  ```js
  Array.prototype.forEach.call('abc', function (chr) {
      console.log(chr);
  })
  ```

  注意：此种方法比原生的 forEach 要慢，因此最好还是将类似数组的对象先转化为数组，再调用数组的 forEach 方法



#### 对象

- 对象的所有键名都是字符串，可以不加引号。

- 如果不同变量名指向同一个对象，则它们都是此对象的引用，即指向同一内存地址

  ```js
  var o1 = {};
  var o2 = o1;
  o2.a = 1;
  o1.a  // 1
  
  // 如果取消某一变量对于原对象的引用，不会影响到另一个变量
  o1 = 1;
  o2  // { a: 1 }
  ```

- 对象属性的读取

  ```js
  var obj = {
      p: 'Hello'
  };
  
  obj.p  // 'Hello'
  obj['p']  // 'Hello'
  ```

- 属性的查看: 使用Object.keys方法

  ```js
  var obj = {
      key1: 1,
      key2: 2
  };
  
  Object.keys(obj);  // ['key1', 'key2']
  ```

- 属性的删除：delete命令

  ```js
  var obj = { p: 1 };
  delete obj.p;  // 返回true
  obj.p  // undefined
  ```

  注意：删除一个不存在的属性，delete不会报错，且返回true

- 查看属性是否存在：in 运算符

  ```js
  var obj = { p: 1 };
  'p' in obj  // true
  'toString' in obj  // true，因为toString属性继承自Object类
  ```

  



#### 字符串

- 可视为字符数组，因此可以使用数组方括号运算符，返回某位置的字符，若下标不符，则返回undefined。

- 但无法改变字符串之中的单个字符。

```js
var str = '12345';
str[0] = '6';
console.log(str);  // 12345

// Base64转码
btoa() 任意值转为Base64编码
atob() Base64编码转为原值
但这两个方法不适合非ASCII码字符，如中文字符
```



#### 判断NaN数据类型的可靠方法

NaN不等于任何值，也是唯一一个不等于自身值的数据，因此：

```js
function myIsNaN(value) {
	return value !== value;
}

// 也可以：
function myIsNaN(value) {
    return typeof value === 'number' && isNaN(value);
    // 对于字符串、对象和数组，isNaN将会返回true，因此最好先判断一下数据类型
}
```



#### Set 的一个雷区

```js
let set = new Set();
set.add([1, 2]);
set.has([1, 2]);  // 输出false
```

原因：对于数组而言，添加数组在Set中后该数组所指向的内存地址就发生了变化，所以用has()获取不到



#### Array.sort

- 该排序方法默认为字典序排序，即若对 Number 数组排序，则 25 比 100 大
- 若参数为 (a, b) => a - b，则为升序
- 若参数为 (a, b) => b - a，则为降序
- 其他排序规则可自行定义



#### 数组扁平化

将一个多维数组变成一个一维数组。

```js
const arr = [1, [2, [3, [4, 5]]], 6];  // => [1, 2, 3, 4, 5, 6]

/*
方法1：
使用Array.prototype.flat()，该函数会按照一个可指定的深度遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回
*/
const res1 = arr.flat(Infinity);

/*
方法2：
使用reduce和concat
reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])
其中initialValue可选，作为第一次调用callback函数时第一个参数的值
const reducer = (previousValue, currentValue) => previousValue + currentValue;
arr.reduce(reducer)
对arr数组中每个元素执行一个提供的reducer函数（升序执行），将结果汇总为单个返回值
*/
const res2 = arr.reduce((acc, val) => acc.concat(val), []);
// 使用扩展运算符...
const res3 = arr => [].concat(...arr);

/*
方法3：
利用正则
String.prototype.replace(regex, newSubStr)
e.g. str.replace(/Dog/i, 'subStr');
*/
const res4 = JSON.stringify(arr).replace(/\[|\]/g, '').split(',');
const res5 = JSON.parse('[' + JSON.stringify(arr).replace(/\[|\]/g, '') + ']');

/*
方法4：
使用reduce和递归
const flatten = arr => {
	return arr.reduce((pre, cur) => {
		return pre.concat(Array.isArray(cur) ? flatten(cur) : cur);
	}, []);
}
*/
const res6 = flatten(arr);
```



#### 数组去重

```js
const arr = [1, 1, '1', 17, true, true, false, false, 'true', 'a', {}, {}];
// => [1, '1', 17, true, false, 'true', 'a', {}];

// 方法1：利用Set
const res1 = Array.from(new Set[arr]);

// 方法2：两层for+splice
const unique1 = arr => {
    let len = arr.length;
    for (let i = 0; i < len; i++) {
        for (let j = i + 1; j < len; j++) {
            if (arr[i] === arr[j]) {
                arr.splice(j, 1);
                len--;
                j--;
            }
        }
    }
    return arr;
};

// 方法3：利用indexOf或includes
const unique2 = arr => {
    const res = [];
    for (let i = 0; i < arr.length; i++) {
        if (res.indexOf(arr[i]) === -1)  res.push(arr[i]);
        // 判断条件也可以是 !res.includes(arr[i])
    }
    return res;
};

// 方法4：利用filter
const unique3 = arr => {
    return arr.filter((item, index) => {
        return arr.indexOf(item) === index;
    });
};

// 方法5：利用Map
const unique4 = arr => {
    const map = new Map();
    const res = [];
    for (let i = 0; i < arr.length; i++) {
        if (!map.has(arr[i])) {
            map.set(arr[i], 1);
            res.push(arr[i]);
        }
    }
    return res;
}
```



#### 类数组转化为数组

类数组具有length属性，但不具有数组原型上的方法，常见类数组有arguments、DOM操作方法返回的结果。

```js
// 方法1：Array.from
// from()方法用于通过拥有length属性或可迭代对象来返回数组
Array.from(document.querySelectorAll('div'));

// 方法2：Array.prototype.slice.call
Array.prototype.slice.call(document.querySelectorAll('div'));

// 方法3：扩展运算符
[...document.querySelectAll('div')];

// 方法4：利用concat
// apply()方法会调用一个函数，apply方法的第一个参数作为被调用函数的this值，第二个参数（一个数组或类数组的对象）会被作为调用对象的arguments值
Array.prototype.concat.apply([], document.querySelectAll('div'));
```



#### Array.prototype.filter()

```js
filter(callback(element[, index[, array]])[, thisArg])
callback: 用来测试数组每个元素的函数，返回true表示通过测试，保留元素，否则不保留
callback接受三个参数：
1）element，数组中当前正在处理的元素
2）index，可选，正在处理的元素在数组中的索引
3）array，可选，调用了filter的数组本身
4）thisArg，可选，执行callback时用于this的值
```

```js
Array.prototype.filter = function(callback, thisArg) {
    if (this == undefined) {
        throw new TypeError('this is null or not undefined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError(callback + 'is not a function');
    }
    
    const res = [];
    // 让O为回调函数的对象传递（强制转换对象）
    const O = Object(this);
    // >>> 0 实际上是无符号右移0位，且浮点数做位运算时小数部分直接丢弃
    // js不同于java，当>>>0时，会将符号位替换为0
    // 因此丢弃小数部分、替换符号位为0，就能够保证正整数
    // >>> 0 保证 len 为 Number，且为正整数
    const len = O.length >>> 0;
    for (let i = 0; i < len; i++) {
        // 检查 i 是否在 O 的属性，会检查原型链
        if (i in O) {
            // 回调函数调用传参
            if (callback.call(thisArg, O[i], i, O)) {
                res.push(O[i]);
            }
        }
    }
    return res;
}
```



#### Array.prototype.map()

```js
参数：
callback：生成新数组元素的函数，使用三个参数：
1）currentValue：callback数组中正在处理的当前元素
2）index，可选，正在处理当前元素的索引
3）array，可选，map方法调用的数组

thisArg，可选，执行callback函数时，值被用作this
```

```js
Array.prototype.map = function(callback, thisArg) {
    if (this == undefined) {
        throw new TypeError('this is null or not defined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError(callback + ' is not a function');
    }
    const res = [];
    const O = Object(this);
    const len = O.length >>> 0;
    for (let i = 0; i < len; i++) {
        if (i in O) {
            // 调用回调函数并传入新数组
            res[i] = callback.call(thisArg, O[i], i, this);
        }
    }
    return res;
}
```



#### Array.prototype.forEach()

```js
参数：
callback：为数组中每个元素执行的函数，接受一到三个参数：
1）currentValue
2）index，可选，当前处理元素的索引
3）array，正在操作的数组

thisArg：可选，用作this
```

```js
Array.prototype.forEach = function(callback, thisArg) {
    if (this == null) {
        throw new TypeError('this is null or undefined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError(callback + ' is not a function');
    }
    const O = Object(this);
    const len = O.length >>> 0;
    let k = 0;
    while (k < len) {
        if (k in O) {
            callback.call(thisArg, O[k], k, O);
        }
        k++;
    }
}
```



#### Array.prototype.reduce()

```js
参数：
callback，包含四个参数：
1）accumulator，累计器累计回调的返回值
2）currentValue
3）index：可选，索引
4）array：可选，调用reduce的数组

initialValue：可选，第一个参数值
```

```js
Array.prototype.reduce = function(callback, initialValue) {
    if (this == undefined) {
        throw new TypeError('this is null or not defined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError(callback + ' is not a function');
    }
    const O = Object(this);
    const len = this.length >>> 0;
    let accumulator = initialValue;
    let k = 0;
    
    // 如果第二个参数initialValue为undefined，则数组第一个有效值作为累计器初值
    if (accumulator === undefined) {
        while (k < len && !(k in O)) {
            k++;
        }
        // 如果超出数组界限还没找到累加器初始值，抛出异常
        if (k >= len) {
            throw new TypeError('Reduce of empty array with no initial value');
        }
        accumulator = O[k++];
    }
    
    while (k < len) {
        if (k in O) {
            accumulator = callback(accumulator, O[k], k, O);
        }
        k++;
    }
    return accumulator;
}
```



#### Function.prototype.apply()

```js
第一个参数是绑定的this，默认为window，第二个参数是数组或类数组
Function.prototype.apply = function(context = window, args) {
    if (typeof this !== 'function') {
        throw new TypeError('Type Error');
    }
    // Symbol是ES6新增的数据类型，可以作为对象的属性标识符使用
    const fn = Symbol('fn');
    context[fn] = this;
    const res = context[fn](...args);
    delete context[fn];
    return res;
}
```



#### Function.prototype.call

```js
Function.prototype.call = function(context = window, ...args) {
    // 接受参数列表...args
    if (typeof this !== 'function') {
        throw new TypeError('Type Error');
    }
    const fn = Symbol('fn');
    context[fn] = this;
    
    const res = context[fn](...args);
    delete context[fn];
    return res;
}
```



#### Function.prototype.bind

```js
Function.prototype.bind = function(context, ...args) {
	if (typeof this !== 'function') {
		throw new TypeError('Type Error');
	}
    var self = this;
    return function F() {
        if (this instanceof F) {
            return new self(...args, ...arguments);
        }
        return self.apply(context, [...args, ...arguments]);
    }
}
```

