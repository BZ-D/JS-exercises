#### 1、找到给定字符串中某段最长的回文子串

中心扩展法：从每一个位置出发向两边扩展，遇到不是回文的时候结束。

扩展的时候，需要分别考虑单个中心字符和两个相同中心字符分别扩展的情况。

若不想单独考虑，则也可以首先向左、向右分别扩散寻找与当前中心字符相同的字符，直到遇到不相等的为止，然后再左右同时扩散找回文串。

##### 法1：中心扩展，分别考虑单中心字符和双中心字符

```js
var longestPalidrome = funstion (s) {
    let maxLength = 0, left = 0, right = 0;
    for (let i = 0; i < s.length; i++) {
        let singleCharLength = getPalLenByCenterChar(s, i, i);
        let doubleCharLength = getPalLenByCenterChar(s, i, i+1);
        let max = Math.max(singleCharLength, doubleCharLength);
        if (max > maxLength) {
            maxLength = max;
            left = i - parseInt((max - 1) / 2);
            right = i + parseInt(max / 2);
        }
    }
    return s.slice(left, right + 1);
};

function getPalLenByCenterChar(s, left, right) {
    // 中间值为两个字符，确保两个字符相等
    if (s[left] !== s[right]) return right - left;
    while (left > 0 && right < s.length - 1) {
        left--;
        right++;
        if (s[left] !== s[right]) {
            return right - left - 1;
        }
    }
    return right - left + 1;
}
```

##### 法2：中心扩展，不单独考虑单中心字符和双中心字符：

```js
var longestPalidrome = funstion (s) {
    if (s.length <= 1) return s;
    let left = 0, right = 0, len = 1, maxStart = 0, maxLen = 0;
    
    for (let i = 0; i < s.length; i++) {
        left = i - 1;
        right = i + 1;
        while (left >= 0 && s[left] === s[i]) {
            len++;
            left--;
        }
        while (right < s.length && s[right] === s[i]) {
            len++;
            right++;
        }
        while (left >= 0 && right < s.length && s[left] === s[right]) {
            len += 2;
            left--;
            right++;
        }
        if (len > maxLen) {
            maxLen = len;
            maxStart = left + 1;
        }
        len = 1;
    }
    return s.slice(maxStart, maxStart + maxLen);
};
```

##### 法3：动态规划：空间换时间

回文天然具有状态转移的性质，即一个长度严格大于2的回文去掉头尾字符后，剩下部分仍然是回文。反之如果一个字符串头尾两个字符都不相等，则它一定不是回文。

- **第一步：定义状态**

  `dp[i][j]`表示子串`s[i..j]`是否为回文子串

- **第二步：思考状态转移方程**

  根据头尾字符是否相等，需要讨论：

  `dp[i][j] = (s[i] === s[j]) AND dp[i + 1][j - 1]`

  即当两端字符都相等，且掐头去尾后的子串也为回文时，`s[i..j]`才为回文。

  **说明：**

  - 动态规划的自底向上求解问题的思路，很多时候是在填写一张二维表，由于`s[i..j]`表示`s`的一个子串，因此`i`和`j`的关系是`i <= j`，因此只需要填写此二维表的对角线以上部分。
  - 看到`dp[i + 1][j - 1]`就需要考虑特殊情况：如果去掉`s[i..j]`头尾两个字符，子串`s[i+1 .. j-1]`的长度严格小于2，即 `j - 1 - (i + 1) + 1 < 2`，整理得`j - 1 < 3`，此时`s[i..j]`是否是回文只取决于`s[i]`和`s[j]`是否相等，结论也比较直观：`j-i<3`等价于`j-i+1<4`，即当子串`s[i..j]`的长度等于2或3时，`s[i..j]`是否是回文与`s[i]`与`s[j]`是否相等决定。

- **第三步：考虑初始化**

  单个字符一定是回文，因此把对角线先初始化为`true`，根据第二步的说明，当`s[i..j]`长度为2时只需要判断`s[i]`是否等于`s[j]`，所以二维表对角线上的数值不会被参考，不设置`dp[i][i] = true`也能得到正确结论。

- **第四步：考虑输出**

  一旦得到`dp[i][j] = true`，就记录子串的『长度』和『起始位置』

```js
var longestPalidrome = funstion (s) {
    if (s.length <= 1) return s;
    
    let maxLen = 1, begin = 0;
    let dp = new Array(s.length);
    for (let i = 0; i < s.length; i++) {
        dp[i] = new Array(s.length);
    }
    
    for (let i = 0; i < s.length; i++) {
        dp[i][i] = true;
    }
    for (let j = 1; j < s.length; j++) {
        for (let i = 0; i < j; i++) {
            if (s[i] !== s[j]) {
                dp[i][j] = false;
            } else {
                if (j - i < 3) dp[i][j] = true;
                else dp[i][j] = dp[i + 1][j - 1];
            }
            
            // 只要dp[i][j]为true，就表示s[i..j]是回文，此时记录回文长度及起始位置
            if (dp[i][j] && j - i + 1 > maxLen) {
                maxLen = j - i + 1;
                begin = i;
            }
        }
    }
    return s.slice(begin, begin + maxLen);
};
```

