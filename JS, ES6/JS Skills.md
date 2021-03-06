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

