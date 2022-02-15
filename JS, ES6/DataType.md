#### 数据类型的转换

- 转换为数值：
  - 严格：只要有一个字符无法转换为数值，整个字符串就会转为 NaN
    - Number('42 cats')  =>  NaN
  - 非严格：
    - parseInt('42 cats')  =>  42
  - 二者都会忽略前导和后缀的空格
  - 当 Number() 方法的参数是对象时，除非为单个元素的数组，否则都会转为 NaN
    1. 调用对象自身的 valueOf 方法，如果返回原始类型值，则直接对该值调用 Number 函数。由于默认情况下 valueOf 方法返回对象本身，一般总会调用 toString 方法
    2. 如果返回对象，则调用对象的 toString 方法，如果返回原始类型值，则对该值使用 Number 函数
    3. 如果 toString 仍返回对象，报错

- 转换为字符串：String()
  - 原始类型值
    - 数值：对应字符串
    - 字符串：不变
    - 布尔值：转为 'true' 或 'false'
    - undefined => 'undefined'
    - null => 'null'
  - 对象
    - String({a: 1})  =>  "[object Object]"
    - String([1, 2, 3])  =>  "1,2,3"
    - 转换流程：
      1. 与 Number() 顺序不同，它先调用对象自身的 toString() 方法，如果返回原始类型值，直接转换
      2. 若返回对象，再调用源对象的 valueOf 方法，如果返回原始类型值，直接转换
      3. 否则报错
- 布尔值：Boolean()
  - undefined null 0 '' NaN，这五个值转为 false
  - 规定对象的布尔值为 `true`
- 自动转换：
  - '5' + {}  =>  '5[object Object]'
  - '5' + []  =>  '5'
  - '5' + function (){}  =>  '5function (){}'


