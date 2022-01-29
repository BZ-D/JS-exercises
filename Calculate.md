注意事项：如果使用累加、累乘等常数级别增长，可能出现超时，因此需要指数级别增长来找到结果。

#### 1、计算 $x^n$

- 累乘：$x\times x\times \dots\times x$
- 转换为指数级增长：$2^9=2\times4^4=2\times16^2=2\times256=512$，对于 $x^n$：
  - 若 n % 2 !== 0，即 n 为奇数，则 $x^n = x \times(x^2\times x^{\lfloor\frac{n}{2}\rfloor})$
  - 否则，$x^n=x^2\times x^{\frac{n}{2}}$

```js
var xPow = function (x, n) {
    if (n === 0) return 1;
    if (n < 0) {
        n = -n;
        x = 1 / x;
    }
    return (n % 2 === 0) ?
        xPow(x * x, n / 2) :
    	x * xPow(x * x, parseInt(n / 2));
}
```

#### 2、求 $\sqrt{x}$

使用二分法。

```js
var xSqrt = function (x) {
    let l = 0, r = x;
    while (true) {
        let mid = l + parseInt(l + (r - l) / 2);
        if (mid * mid > x) {
            r = mid - 1;
        } else if (mid * mid < x) {
            if ( (mid + 1) * (mid + 1) > x )  return mid;
            l = mid + 1;
        } else return mid;
    }
}
```

