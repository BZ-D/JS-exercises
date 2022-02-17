#### 1、ThreeNum

三数之和，双指针移动。答案中三元组不能重复，因此需要进行去重。

<img src="assets/image-20220205145833731.png" alt="image-20220205145833731" style="zoom:67%;" />

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var threeSum = function(nums) {
    if (nums.length < 3) return [];

    let ans = [];
    nums.sort((a,b) => a-b);  // 升序

    // 升序排序后，从数组左侧开始，若nums[i]大于0，则三数之和不可能为0,
    // 否则，取左指针l=i+1，右指针r=len-1
    // 若nums[i]+nums[l]+nums[r]>0，r--
    // 若nums[i]+nums[l]+nums[r]<0，l++
    // 若nums[i]+nums[l]+nums[r]=0，ans中添加一个答案数组，然后l++, r--
    for (let i = 0; i < nums.length && nums[i] <= 0; i++) {
        // 去重
        if (i >= 1 && nums[i] === nums[i - 1]) {
            continue;
        }

        let l = i + 1, r = nums.length - 1;
        while (l < r) {
            if (nums[i] + nums[l] + nums[r] < 0) {
                l++;
            } else if (nums[i] + nums[l] + nums[r] > 0) {
                r--;
            } else {
                ans.push([nums[l], nums[r], nums[i]]);
                while (nums[l] === nums[l + 1]) l++;	// 去重
                while (nums[r] === nums[r - 1]) r--;	// 去重
                l++;
                r--;
            }
        }
    }

    return ans;
};
```



#### 2、区间合并

<img src="assets/image-20220211205849607.png" alt="image-20220211205849607" style="zoom:67%;" />

**排序**：按区间下界升序的原则进行排序，若区间下界相等，则按区间上界降序的原则排序。排序后，遍历intervals的每个元素，若当前元素的下界大于之前合并的区间的上界，则将之前合并好的区间加入ans中。

```js
/**
 * @param {number[][]} intervals
 * @return {number[][]}
 */
var merge = function(intervals) {
    if (intervals.length === 1) return intervals;

    let ans = [];
    // 排序
    intervals.sort((a, b) => a[0] === b[0] ? b[1] - a[1] : a[0] - b[0]);

    let tmp = [intervals[0][0], intervals[0][1]];
    for (let i = 1; i < intervals.length; i++) {
        // 若之间合并好的区间与当前元素区间无交集，则将之前合并好的区间加入ans
        if (tmp[1] < intervals[i][0]) {
            ans.push(tmp);
            // tmp设为当前区间
            tmp = [intervals[i][0], intervals[i][1]];
            continue;
        }

        tmp = [tmp[0], Math.max(tmp[1], intervals[i][1])];
    }
    ans.push(tmp);

    return ans;
};
```



#### 3、双栈实现队列

```js
var CQueue = function() {
    this.stack1 = [];  // 入队栈
    this.stack2 = [];  // 出队栈
};

CQueue.prototype.appendTail = function(value) {
    this.stack1.push(value);
};

CQueue.prototype.deleteHead = function() {
    if (!this.stack2.length) {
        while (this.stack1.length) {
            this.stack2.push(this.stack1.pop());
        }
    }
    return this.stack2.pop() ?? -1;
};

/**
 * Your CQueue object will be instantiated and called as such:
 * var obj = new CQueue()
 * obj.appendTail(value)
 * var param_2 = obj.deleteHead()
 */
```

