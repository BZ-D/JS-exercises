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





#### Object 对象

- Object() 方法可用于将任意值转换为对象

  - var a = 1;  Object(a)  =>  {1}
  - 判断一个变量是否为对象：`value === Object(value)`

- Object() 方法可用于创建新对象

  - var obj = new Object();
  - 等价于 var obj = {};

- Object.keys() 和 Object.getOwnPropertyNames()

  - 都可以用于遍历对象属性，一般情况下返回值为一个含有该对象自身（不含继承的）属性名的数组。但后者可以遍历到 **不可遍历属性**：如遍历一个数组时，会遍历到其 `length` 属性
  - 可以使用 `Object.keys(obj).length` 或 `Object.getOwnPropertyNames(obj).length` 获取对象中属性的个数

- 实例方法

  - Object.prototype.valueOf()：返回一个对象的“值”，默认返回其自身，可由用户自定义

  - Object.prototype.toString()：返回一个对象的字符串形式，如 "[object Object]"，可由用户自定义

    - 该方法也可用于判断数据类型，为了避免调用对象可能自定义的 toString() 方法，建议在类型判断时调用 Object.prototype.toString 方法：

      `Object.prototype.toString.call(value)`

  - Object.prototype.toLocaleString()：与 toString 返回结果相同，主要作用是留出一个接口，让各种不同对象实现自己版本的 toLocaleString

  - Object.prototype.hasOwnProperty()：接受一个字符串作为参数，返回一个布尔值，表示该实例对象自身是否具有该属性。`obj.hasOwnProperty('p')`



#### 属性描述对象

一种内部数据结构，用于描述对象的属性、控制它的行为，比如该属性是否可写、可遍历等。

属性描述对象中的各个属性称为元属性，是控制属性的属性

```js
{
  value: 123, 属性值，默认undefined
  writable: false, 是否可写，即对该属性重新赋值是否有效
  enumerable: true, 是否可遍历
  configurable: false, 是否可以对该属性的属性描述对象配置、重定义
  get: undefined, 取值函数，默认undefined
  set: undefined 存值函数，默认undefined
}
```

- 属性描述对象的获取：`Object.getOwnPropertyDescriptor()` 方法，但只能获取对象自身的属性，不能用于继承的属性

- 定义属性描述对象：`Object.defineProperty()` 或 `Object.defineProperties()`

- 判断是否可遍历：`Object.prototype.propertyIsEnumerable()`

- 正常模式下，对不可写对象赋值不会报错，只会默默失败；严格模式下报错，即使赋一个一模一样的值

- 如果一个属性不可遍历，即 eumerable 为 false 时，下面三个操作不会取到属性：

  - for ... in
  - Object.keys
  - JSON.stringify

- 一旦定义了存取器get和set，存取时将执行对应的函数

  ```js
  // 写法一，这里的p是不可遍历、不可配置的
  var obj = Object.defineProperty({}, 'p', {
    get: function () {
      return 'getter';
    },
    set: function (value) {
      console.log('setter: ' + value);
    }
  });
  
  obj.p // "getter"
  obj.p = 123 // "setter: 123"
  
  
  // 写法二，这里的p是可遍历的、可配置的
  var obj = {
    get p() {
      return 'getter';
    },
    set p(value) {
      console.log('setter: ' + value);
    }
  };
  ```



#### 对象的拷贝

使用 `Object.defineProperty` 方法拷贝属性，避免出现遇到存取器定义只会拷贝值的情况

```js
var extend = function (to, from) {
  for (var property in from) {
    if (!from.hasOwnProperty(property)) continue;
    Object.defineProperty(
      to,
      property,
      Object.getOwnPropertyDescriptor(from, property)
    );
  }

  return to;
}

extend({}, { get a(){ return 1 } })
// { get a(){ return 1 } })
```



#### 控制对象状态

- Object.preventExtensions()

  - 使一个对象无法添加新的属性，最弱
  - Object.isExtensible()，用于检查是否对某对象应用了Object.preventExtensions()方法
  - 是否可以为一个对象添加属性

- Object.seal()

  - 使一个对象无法添加新的属性，也无法删除旧的属性，将属性描述对象中的configurable改为false
  - 但并不会影响修改属性值
  - Object.isSealed()，用于检查是否对某对象应用了Object.seal()方法

- Object.freeze()

  - 使一个对象无法添加新属性，无法删除旧属性，也无法改变属性值，让对象成为常量
  - Object.isFrozen()

- 上面三种方法有一个漏洞：可以通过改变原型对象，为对象增加属性

  ```js
  var obj = new Object();
  Object.preventExtensions(obj);
  
  var proto = Object.getPrototypeOf(obj);
  proto.t = 'hello';
  obj.t
  // hello
  ```

  - 一种解决方案是将 obj 的原型也冻结

  - 另一个局限是：如果属性值也是对象，上面这些方法只能冻结属性指向的对象，而不能冻结对象本身内容：

    ```js
    var obj = {
      foo: 1,
      bar: ['a', 'b']
    };
    Object.freeze(obj);
    
    obj.bar.push('c');
    obj.bar // ["a", "b", "c"]
    ```



#### 包装对象

原始类型——数值、字符串、布尔值，在一定的条件下也会自动转为对象，也就是原始类型的包装对象。所谓包装对象，指的是与数值、字符串、布尔值分别相对应的 Number/String/Boolean 三个原生对象。

- Number() / String() / Boolean() 三个方法如果不作为构造函数调用（即前边加new），而是作为普通函数，则用于将任意类型的值转化为对应的值

- 这三个包装对象的valueOf方法返回包装对象实例对应的原始类型的值

  - new Number(123).valueOf()  =>  123

- 这三个包装对象的toString方法返回包装对象实例对应的字符串形式

  - new Boolean(false).toString()  =>  "false"

- 在某些场合，如调用包装对象的属性和方法时，引擎会自动将原始类型值转化为包装对象实例，在使用后立即销毁实例

  - 'abc'.length
  - 等价于 var strObj = new String('abc');  strObj.length
  - 自动转换生成的包装对象时只读的，无法修改，所以字符串无法添加新属性

- Boolean用于进行类型转换时的情况：

  ```js
  Boolean(undefined) // false
  Boolean(null) // false
  Boolean(0) // false
  Boolean('') // false
  Boolean(NaN) // false
  
  Boolean(1) // true
  Boolean('false') // 特殊 true
  Boolean([]) // true
  Boolean({}) // true
  Boolean(function () {}) // true
  Boolean(/foo/) // true
  ```

  特殊情况：对于一些特殊值，Boolean对象前面加不加new，会得到完全相反的结果：

  ```js
  if (Boolean(false)) {
    console.log('true');
  } // 无输出
  
  if (new Boolean(false)) {
    console.log('true');
  } // true，认为对象为true
  
  if (Boolean(null)) {
    console.log('true');
  } // 无输出
  
  if (new Boolean(null)) {
    console.log('true');
  } // true，认为对象为true
  ```



#### Number对象

- Number对象的静态属性
  - Number.POSITIVE_INFINITY
  - Number.NEGATIVE_INFINITY
  - Number.NaN
  - Number.MIN_VALUE: 最小的正数 `5e-324`
  - Number.MAX_SAFE_INTEGER: 能精确表示的最大整数
  - Number.MIN_SAFE_INTEGER: 能精确表示的最小整数
- Number对象的实例方法(Number.prototype.*)
  - toString()，数值转字符串；可接受一个参数，表示输出的进制
  - toFixed()，先将一个数转为指定位数的小数，然后返回这个小数对应的字符串
    - (10).toFixed(2) => "10.00"
    - 小数5的四舍五入是不确定的：
    - (10.055).toFixed(2) => "10.05"
    - (10.005).toFixed(2) => "10.01"
  - toExponential()，将一个数转为科学计数法形式，返回字符串
  - toPrecision()，将一个数转换为指定位数的有效数字，位数包含了整数位数和小数位数，但四舍五入不太可靠，跟浮点数不是精确存储有关
  - toLocaleString()，接受一个地区码座位参数，返回一个字符串，表示该数字在当地的书写形式



#### String对象

是一个类似数组的对象。

- 静态方法
  - fromCharCode()：参数为Unicode码点，返回值为这些码点组成的字符串

- 实例方法(String.prototype.*)
  - charAt()：返回指定位置字符
  - charCodeAt()：返回指定位置字符的码点
  - concat()：字符串连接操作，若参数不是字符串，先转为字符串再链接
  - slice()：取子串
  - substring()：类似于slice，但规则违反直觉，因此优先使用slice
  - indexOf(chr[, startInd]) / lastIndexOf()
  - trim()：去除字符串两端的空格，返回一个新串
  - toLowerCase / toUpperCase：将字符串全部转换为小写、大写形式
  - str.match(subStr)：确定原字符串是否匹配某个子字符串，也可以用正则表达式作参数，返回一个数组，成员为匹配的第一个字符串，如果没有找到匹配，返回null
  - search()：用法基本等同于match，返回匹配到的第一个位置，若未匹配，返回-1
  - replace()：替换掉匹配的子字符串，一般情况下只替换第一个匹配。也可以用正则表达式作参数
    - 'aaa'.replace('a', 'b')  // "baa"
  - split()：按给定规则分割字符串，接受第二个参数，表示返回数组的最大长度
  - localeCompare()：比较两个字符串，考虑自然语言的排序情况
    - 'B' > 'a'  // false
    - 'B'.localeCompare('a')  // 1



#### Math对象

- 静态属性：提供一些数学常数，如 Math.E / Math.LN2 / Math.PI 等

- 静态方法：abs/ceil/floor/max/min/sqrt/pow/log/exp/round/random 等

  - Math.round()方法对于负数中的 `.5` 会进行舍去：Math.round(-1.5) => -1

  - Math.random()方法返回一个 [0, 1) 中的伪随机数，任意范围的随机数生成函数如下：

    ```js
    // 随机数
    function getRandomArbitrary(min, max) {
      return Math.random() * (max - min) + min;
    }
    
    getRandomArbitrary(1.5, 6.5)
    // 2.4942810038223864
    
    // 随机整数
    function getRandomInt(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    }
    
    getRandomInt(1, 6) // 5
    
    // 随机字符
    function random_str(length) {
      var ALPHABET = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
      ALPHABET += 'abcdefghijklmnopqrstuvwxyz';
      ALPHABET += '0123456789-_';
      var str = '';
      for (var i = 0; i < length; ++i) {
        var rand = Math.floor(Math.random() * ALPHABET.length);
        str += ALPHABET.substring(rand, rand + 1);
      }
      return str;
    }
    
    random_str(6) // "NdQKOr"
    ```

    

#### Date对象

- 直接调用 Date 总是返回当前时间：Date()
- 作为构造函数使用时，创造一个对象实例：var today = new Date();
- 对Date实例求值时，返回一个字符串，因为默认调用 toString 而非 valueOf
- 两个日期实例对象进行减法运算时，返回间隔的毫秒数
- 静态方法
  - now()：返回距离时间零点的毫秒数
  - parse()：解析日期字符串，返回毫秒数
- 实例方法
  - valueOf()：返回距离时间零点的毫秒数，等同于getTime()
  - toString()：返回一个完整的日期字符串
  - toJSON()：返回一个符合JSON格式的ISO日期字符串
  - toDateString()：返回日期字符串，不含有小时、分、秒
  - toTimeString()：返回时间字符串，不含年月日
  - toLocaleString()：完整的本地时间
  - toLocaleDateString()：本地日期
  - toLocaleTimeString()：本地时间

- `Date`对象提供了一系列`get*`方法，用来获取实例对象某个方面的值。
  - `getTime()`：返回实例距离1970年1月1日00:00:00的毫秒数，等同于`valueOf`方法。
  - `getDate()`：返回实例对象对应每个月的几号（从1开始）。
  - `getDay()`：返回星期几，星期日为0，星期一为1，以此类推。
  - `getFullYear()`：返回四位的年份。
  - `getMonth()`：返回月份（0表示1月，11表示12月）。
  - `getHours()`：返回小时（0-23）。
  - `getMilliseconds()`：返回毫秒（0-999）。
  - `getMinutes()`：返回分钟（0-59）。
  - `getSeconds()`：返回秒（0-59）。
  - `getTimezoneOffset()`：返回当前时间与 UTC 的时区差异，以分钟表示，返回结果考虑到了夏令时因素。

  所有这些`get*`方法返回的都是整数，不同方法返回值的范围不一样。

  - 分钟和秒：0 到 59
  - 小时：0 到 23
  - 星期：0（星期天）到 6（星期六）
  - 日期：1 到 31
  - 月份：0（一月）到 11（十二月）



#### JSON对象

**格式规定：**

- 复合类型的值只能是对象或数组，不能是函数、正则对象、日期对象
- 原始类型的值只有四种：字符串、数值（十进制）、布尔值和null。其他诸如NaN、Infinity、-Infinity、undefined等均不可以
- 字符串必须用双引号表示
- 对象键名必须放在双引号里边
- 数组或对象最后一个成员后边不能加逗号

**合法格式示例：**

```js
["one", "two", "three"]

{ "one": 1, "two": 2, "three": 3 }

{"names": ["张三", "李四"] }

[ { "name": "张三"}, {"name": "李四"} ]

// null、空数组、空对象均合法
null
[]
{}
```

**不合法格式示例：**

```js
{ name: "张三", 'age': 32 }  // 属性名必须使用双引号

[32, 64, 128, 0xFFF] // 不能使用十六进制值

{ "name": "张三", "age": undefined } // 不能使用 undefined

{ "name": "张三",
  "birthday": new Date('Fri, 26 Aug 2011 07:13:10 GMT'),
  "getName": function () {
      return this.name;
  }
} // 属性值不能使用函数和日期对象
```



- 两个静态方法
  - stringify()：将一个值转为 JSON 字符串，并且可以被 JSON.parse() 方法还原
    - 注意：对原始类型的字符串，转换结果会带双引号：`JSON.stringify('foo') === "\"foo\""`
    - 会自动过滤 undefined、函数属性或 XML 对象
    - 对数组使用此方法，如果数组成员为 undefined、函数或 XML 对象，将它们转化为 null
    - 正则对象会转为空对象：JSON.stringify(/foo/)  =>  {}
    - 该方法还可以接受一个数组作为第二个参数，指定对象哪些属性需要转为字符串
