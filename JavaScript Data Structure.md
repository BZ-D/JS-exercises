## 各类算法题解决方案

### 一、二叉树

二叉树大多使用递归方式从左右子树向下递归。

#### 1、计算二叉树最大深度

```js
var maxDepth = function (root) {
    if (root === null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
};
```

#### 2、二叉树层序遍历为二维数组

```
    3               
   / \
  5   2
 / \ / \
8  3 4  1
输出结果应为 [[3], [5, 2], [8, 3, 4, 1]]
实际上就是一个二叉树层序遍历的问题，需要判断什么时候换层(i === ans.length时)
```

```js
var levelOrder = function (root) {
    let ans = [];
    helper(root, ans, 0);
    return ans;
};

function helper(node, ans, i) {
    if (node === null) return;
    if (i === ans.length) ans.push([]);
    ans[i].push(node.val);
    
    helper(node.left, ans, i+1);
    helper(node.right, ans, i+1);
}
```



### 二、可能性问题

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



### 三、查找

一般遇到的查找问题，如查找某个值，通常会用到以下方法：

- 排序算法（排序是为了方便后续查找）
- 二分查找（两数之和/三数之和为 0 的题目，可通过二分查找减小时间复杂度）
- “索引移动查找”

#### 1、查找横向和纵向都递增的二维矩阵中的某值

“索引移动查找”

```js
var searchMatrix = function (matirx, target) {
    // matrix的形式为：元素为Array的Array
    if (matrix.length === 0) return false;
    // 从右上角开始
    let row = 0, col = matrix[0].length - 1;
    while (true) {
        // 用if-elseif结构是为了每次循环只移动一次索引
        if (matrix[row][col] > target && col > 0) {
            col--;
        } else if (matrix[row][col] < target && row < matrix.length - 1) {
            row++;
        } else if (matrix[row][col] == target) {
            return true;
        } else {
            // 当前元素值大于目标值，说明目标值不存在于矩阵中
            break;
        }
    }
    return false;
};
```

#### 2、找到数组中最左边和最右边某个数字所在位置

某非递减数组中有一系列元素，找出给定目标值在数组中的开始位置和结束位置，若不存在，返回[-1, -1]。

使用二分法查找到某个目标数字的索引值，然后用索引移动法分别向左和向右查找字符，获取左右两侧索引值返回。

```js
var searchRange = function (nums, target) {
	let targetIndex = binarySearch(nums, target, 0, nums.length - 1);
    if (targetIndex === -1) return [-1, -1];
    let l = targetIndex, r = targetIndex;
    while (l > 0 && nums[l - 1] === target) {
        l--;
    }
    while (r < nums.length - 1 && nums[r + 1] === target) {
        r++;
    }
    return [l, r];
};

function binarySearch(arr, val, lo, hi) {
    if (hi < lo) return -1;
    let mid = lo + parseInt((hi - lo) / 2);
    
    if (val < arr[mid]) {
        return binarySearch(arr, val, lo, mid - 1);
    } else if (val > arr[mid]) {
        return binarySearch(arr, val, mid + 1, hi);
    } else {
        return mid;
    }
    
    /*
    或：
    while (lo <= hi) {
    	mid = lo + parseInt((hi - lo) / 2);
        if (val < arr[mid]) {
            hi = mid - 1;
        } else if (val > arr[mid]) {
            lo = mid + 1;
        } else {
            return mid;
        }
    }
    return -1;
    */
}
```



### 四、回文

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



### 五、路径

路径题可用DFS、BFS进行求解，通过递归将走过的路径进行标记来不断往前找到目标路径。

**DFS解题过程范式：**

1. 判断当前位置是否合法，如果被访问过或者越界，不用继续考虑
2. 判断当前位置是否和目标字符串的pos位置字符匹配
3. 判断匹配的pos是不是已经是最后一个位置，如果是，说明我们已经完成这次匹配，可以直接返回
4. 如果匹配我们先标记当前位置已经访问过（如给该位置设为「*」），然后继续遍历相邻的搜索空间

5. 最后清理现场，将已经访问等状态复原（将设为「*」的元素复原）

#### 1、单词搜索

给定一个 m x n 二维字符网格 board 和一个单词（字符串）列表 words，返回所有二维网格上的单词。单词必须按照字母顺序，通过 相邻的单元格 内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母在一个单词中不允许被重复使用。

<img src="assets/image-20220126234455078.png" alt="image-20220126234455078" style="zoom:67%;" />

```js
let hasWord = false;

var findWords = function (board, words) {
    let ans = [];
    let m = board.length, n = board[0].length;
    for (let word of words) {
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (board[i][j] === word[0]) {
                    hasWord = false;
                    DFS(word, board, 0, i, j, "");
                    if (hasWord) {
                        if (!ans.includes(word)) ans.push(word);
                    }
                }
            }
        }
    }
    return ans;
};

function DFS(word, board, index, i, j, subStr) {
    if (word[index] === board[i][j]) {
        subStr += board[i][j];
        // 标记已访问
        board[i][j] = "*";
        if (i < board.length - 1)
            // 向右找
            DFS(word, board, index + 1, i + 1, j, subStr);
        if (i > 0)
            // 向左找
            DFS(word, board, index + 1, i - 1, j, subStr);
        if (j < board[0].length - 1)
            // 向下找
            DFS(word, board, index + 1, i, j + 1, subStr);
        if (j > 0)
            // 向上找
            DFS(word, board, index + 1, i, j - 1, subStr);
        // 复原已访问状态
        board[i][j] = word[index];
    }
    if (index >= word.length || subStr === word) {
        hasWord = true;
    }
}
```

#### 2、矩阵中的最长递增路径

<img src="assets/image-20220127002027644.png" alt="image-20220127002027644" style="zoom:67%;" />

将已使用DFS查找过的长度放入**缓存**，若有其他元素通过DFS走到当前值，直接返回缓存最大值即可。

```js
// 用于进行dfs遍历，即分别向右、下、左、上进行深搜
const dirs = [[0, 1], [1, 0], [0, -1], [-1, 0]];

var longestIncreasingPath = function (matrix) {
    if (matrix.length === 0) return 0;
    const m = matrix.length, n = matrix[0].length;
    let max = 1;
    let cache = new Array(m);
    for (let i = 0; i < m; i++) {
        let child = new Array(n);
        child.fill(0);
        cache[i] = child;
    }
    
    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            // 对矩阵中每个元素进行一次DFS，但有缓存避免超时
            let len = dfs(matrix, i, j, m, n, cache);
            max = Math.max(max, len);
        }
    }
    return max;
};

function dfs(matrix, i, j, m, n, cache) {
    // 搜索目标：找到从当前元素matrix[i][j]开始的最长递增序列
    // cache缓存数组用于存放从当前元素matrix[i][j]开始的最长递增序列的长度
    // 当前位置cache元素不为0，意味着此位置已经遍历过，直接返回即可，避免超时
    if (cache[i][j] !== 0) return cache[i][j];
    let max = 1;
    for (let dir of dirs) {
        let x = i + dir[0], y = j + dir[1];
        if (x < 0 || x >= m || y < 0 || y >= n || matrix[x][y] <= matrix[i][j]) {
            // 当索引越界，或当前位置元素非递增时，直接退出
            continue;
        }
        
        let len = 1 + dfs(matrix, x, y, m, n, cache);
        max = Math.max(max, len);
    }
    cache[i][j] = max;
    return max;
}
```



### 六、链表

链表从 JS 的角度来说就是一串对象使用指针连接的数据结构，通过`next`指针改变指向。

#### 1、链表排序

使用自上而下的归并算法进行排序，使用`slow.next`和`fast.next.next`两种速度获取链表节点，从而获取中间值。
