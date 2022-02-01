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
}
```

