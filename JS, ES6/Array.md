#### 数组 Array

- 在新建数组时，new Array() 中的参数是一个正整数，返回数组的成员都是 **空位**，虽然读取的时候读到 undefined，但该位置没有任何值

- 识别数组：Array.isArray 方法

- 数组的 toString 方法返回数组的字符串形式：[1,2,3] => "1,2,3"

- 如果数组成员有 undefined 或 null，在 join 时会转为空串

- 数组的join方法可以用于字符串或类似数组的对象

  - Array.prototype.join.call('hello', '-')  =>  "h-e-l-l-o"
  - var obj = { 0: 'a', 1: 'b', length: 2 };
    - Array.prototype.join.call(obj, '-')  =>  'a-b'

- concat 方法将值添加到目标数组尾部。如果数组成员包括对象，concat 方法返回当前数组的一个浅拷贝，所谓浅拷贝，就是新数组拷贝的是对象的引用

  ```js
  var obj = { a: 1 };
  var oldArray = [obj];
  
  var newArray = oldArray.concat();
  
  obj.a = 2;
  newArray[0].a // 2
  ```

- reverse 方法用于颠倒数组元素，将改变原数组

- slice方法类似于substr，若slice没有参数，则返回原数组的一个深拷贝

  - 若参数为负数，则从尾部开始计算，负几就代表第几个，和正常的下标不同

  - 该方法的一个重要应用是将类似数组的对象转换为真正的数组：

    ```js
    Array.prototype.slice.call({ 0: 'a', 1: 'b', length: 2 })
    // ['a', 'b']
    
    Array.prototype.slice.call(document.querySelectorAll("div"));
    Array.prototype.slice.call(arguments);
    ```

- splice方法用于删除原数组的一部分成员，并可以在删除的位置添加新的数组成员，返回值为被删除的元素，该方法会改变原数组：

  - arr.splice(start, count, ...addElements)

    ```js
    var a = ['a', 'b', 'c', 'd', 'e', 'f'];
    a.splice(4, 2) // ["e", "f"]
    a // ["a", "b", "c", "d"]
    
    var a = ['a', 'b', 'c', 'd', 'e', 'f'];
    a.splice(4, 2, 1, 2) // ["e", "f"]
    a // ["a", "b", "c", "d", 1, 2]
    
    起始位置为负数时，从末位开始计算开始删除的下标，然后从左往右删除。负几就代表第几个，和正常的下标不同
    var a = ['a', 'b', 'c', 'd', 'e', 'f'];
    a.splice(-4, 2) // ["c", "d"]
    ```

- map方法将所有成员依次传入参数函数，然后把每一次执行结果组成一个**新数组**返回，注意，原数组不会被改变。可以接受第二个参数，将回调函数内部的this对象指向arr数组

- forEach方法与map方法很相似，但不返回值，只用来操作数据，可以接受第二个对象thisArg。如果要使用返回值，用map，否则用forEach

- filter方法用于过滤数组成员，满足条件的成员组成一个**新数组**返回，注意原数组不会被改变

- some方法和every方法类似于断言，判断数组元素是否满足某种条件，some方法返回true，当某一元素满足条件；every方法返回true，当所有元素满足条件。这两个方法都可以接受第二个参数thisArg

  ```js
  var arr = [1,2,3,4,5]
  arr.some(function (el, i, arr) {
      return el >= 3;
  });
  // true
  ```

  注意：对于空数组，some方法返回false，every方法返回true，回调函数都不会执行。

- reduce方法和reduceRight方法依次处理数组每个成员，最终累计为一个值，前者从左到右处理，后者从右到左处理。

  ```js
  [1,2,3,4,5].reduce(function (accumulator, curVal) {
      return accumulator + curVal;
  })
  ```

  reduce方法接受第二个参数，即accumulator的初始值。建议总是添加此初始值

- indexOf方法返回某元素在数组中第一次出现的位置，若没有出现则返回-1，它可以接受第二个参数，表示开始搜索的位置。lastIndexOf方法返回某元素在数组中最后一次出现的位置。这两个方法不能用来搜索NaN的位置

- 对于上述某些返回数组的方法，可以进行链式调用

  ```js
  var users = [
    {name: 'tom', email: 'tom@example.com'},
    {name: 'peter', email: 'peter@example.com'}
  ];
  
  users
  .map(function (user) {
    return user.email;
  })
  .filter(function (email) {
    return /^t/.test(email);  // 过滤出以t开头的地址
  })
  .forEach(function (email) {
    console.log(email);
  });
  // "tom@example.com"
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