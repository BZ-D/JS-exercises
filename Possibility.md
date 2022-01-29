这类题一般是告诉一组数据，然后求出可能性或最值。

#### 1、给定几种面额的硬币和一个总额，用最少的硬币凑成这个总额

```js
var coinChange = function (coins, amount) {
    let max = amount + 1;
    let dp = new Array(amount + 1);
    dp.fill(max);
    dp[0] = 0;
    
    for (let i = 1; i < max; i++) {
        for (let j = 0; j < coins.length; j++) {
            if (coins[j] <= i) {
                dp[i] = Math.min(dp[i], dp[i - coins[j]] + 1);
            }
        }
    }
    
    return dp[amount] > amount ? -1 : dp[amount];
};
```

使用了动态规划，将从 0 到目标额度所需的最小硬币数都列出来。

#### 2、求出从矩阵左上角走到右下角，且只能向右、向下移动，一共有多少走法

```js
var uniquePaths = function (m, n) {
    const pos = new Array(m);
    for (let i = 0; i < m; i++) {
        pos[i] = new Array[n];
    }
    for (let i = 0; i < n; i++) {
        pos[0][i] = 1;
    }
    for (let i = 0; i < m; i++) {
        pos[i][0] = 1;
    }
    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            pos[i][j] = pos[i-1][j] + pos[i][j-1];
        }
    }
    return pos[m-1][n-1]
};
```

使用动态规划逐步列出每一格的可能性，最后返回最右下角的可能性。

#### 3、获取给定数组连续元素累加最大值

```js
var maxSubArray = function (nums) {
    let count = nums[0], maxCount = nums[0];
    for (let i = 1; i < nums.length; i++) {
        count = Math.max(count + nums[i], nums[i]);
        maxCount = Math.max(maxCount, count);
    }
    return maxCount;
};
```

通过不断对比最大值并保存最大值。