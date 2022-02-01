[一般用按位运算符和二进制转换 `Number.parseInt()` 和 `Number.prototype.toString()` 解决。]()

#### 1、将一个32位数字的二进制进行倒序

**利用库函数：**

```js
var reverseBits = function(n) {
    let t = n.toString(2).split("");	// 转为数组
    while (t.length < 32) t.unshift("0");  // 数组头部添0凑够32位
    return parseInt(t.reverse().join(""), 2);
}
```

**位运算分治：**

<img src="assets/image-20220129000208809.png" alt="image-20220129000208809" style="zoom:50%;" />

- 若要翻转一个二进制串，可以将其均分成左右两部分，对每部分递归执行翻转操作，然后将左半部分拼在右半部分的后面，即完成了翻转。


- 由于左右两部分的计算方式是相似的，利用位掩码和位移运算，我们可以自底向上地完成这一分治流程。


- 对于递归的最底层，我们需要交换所有奇偶位：
  - 取出所有奇数位和偶数位；
  - 将奇数位移到偶数位上，偶数位移到奇数位上。
  - 类似地，对于倒数第二层，每两位分一组，按组号取出所有奇数组和偶数组，然后将奇数组移到偶数组上，偶数组移到奇数组上。以此类推。

- 需要注意的是，在某些语言（如 Java）中，没有无符号整数类型，因此对 n 的右移操作应使用逻辑右移。


```js
var reverseBits = function(n) {
    const M1 = 0x55555555; // 01010101010101010101010101010101
    const M2 = 0x33333333; // 00110011001100110011001100110011
    const M4 = 0x0f0f0f0f; // 00001111000011110000111100001111
    const M8 = 0x00ff00ff; // 00000000111111110000000011111111

    // >>> 为以0填充的右位移
    n = n >>> 1 & M1 | (n & M1) << 1;
    n = n >>> 2 & M2 | (n & M2) << 2;
    n = n >>> 4 & M4 | (n & M4) << 4;
    n = n >>> 8 & M8 | (n & M8) << 8;
    return (n >>> 16 | n << 16) >>> 0;
};
```



#### 2、巧用位运算

**例题：最长的美好子字符串**

<img src="assets/image-20220201101602715.png" alt="image-20220201101602715" style="zoom:67%;" />

```js
// 方法1：暴力遍历所有子串
// 时间复杂度：O(n^2)，空间复杂度：O(1)
var longestNiceSubstring = function(s) {
    const n = s.length;
    let maxPos = 0;
    let maxLen = 0;
    for (let i = 0; i < n; ++i) {
        let lower = 0;
        let upper = 0;
        for (let j = i; j < n; ++j) {
            if ('a' <= s[j] && s[j] <= 'z') {
                // charCodeAt方法返回字符的ASCII码
                // 下面的运算实际是在二进制中为每个字符分配了一个位
                // 如'a'是第0位，'z'是第25位
                // 某一位为1，则说明子串中出现了该位表示的字符
                lower |= 1 << (s[j].charCodeAt() - 'a'.charCodeAt());
            } else {
                // 使用二进制表示'A'~'Z'
                upper |= 1 << (s[j].charCodeAt() - 'A'.charCodeAt());
            }
            if (lower === upper && j - i + 1 > maxLen) {
                maxPos = i;
                maxLen = j - i + 1;
            }
        }
    }
    return s.slice(maxPos, maxPos + maxLen);
};

// 方法2：分治
// 如果字符串本身就是一个美好字符串，最长的美好字符串就是本身
// 否则，该字符串中一定包含某些字符 chr，对于这些 chr 没有对应的大写形式或小写形式出现
// 因此可以利用分治思想将字符串从这些非法字符 chr 处切分成若干段，则满足要求的最长子串一定出现在某个被切分的段内，而不能跨越一个或多个段

"use strict";  // 开启严格模式优化尾递归
var maxPos = 0;
var maxLen = 0;
var longestNiceSubString = function (s) {
    dfs(s, 0, s.length - 1);
    return s.slice(maxPos, maxPos + maxLen);
}

function dfs(s, start, end) {
    if (start >= end) return;
    let lower = 0, upper = 0;
    for (let i = start; i <= end; i++) {
        if (s[i] >= 'a' && s[i] <= 'z') {
            lower |= 1 << (s[i].charCodeAt() - 'a'.charCodeAt());
        } else {
            upper |= 1 << (s[i].charCodeAt() - 'A'.charCodeAt());
        }
    }
    if (lower === upper) {
        if (end - start + 1 > maxLen) {
            maxPos = start;
            maxLen = end - start + 1;
        }
        return;
    }
    let valid = lower & upper;  // 保留所有兼有大小写形式的字符
    let pos = start;
    while (pos <= end) {
        start = pos;
        while (pos <= end && (valid & ( 1 << s[pos].toLowerCase().charCodeAt() - 'a'.charCodeAt() ) !== 0)) {
            // 循环终止条件：遇到了非法字符
            pos++;
        }
        dfs(s, start, pos - 1);
        ++pos;  // 跳过上边遇到的非法字符
    }
}
```

