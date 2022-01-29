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

