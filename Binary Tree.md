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

#### 3、由前序和中序构建二叉树

<img src="assets/image-20220216203936642.png" alt="image-20220216203936642" style="zoom:67%;" />

```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {number[]} preorder
 * @param {number[]} inorder
 * @return {TreeNode}
 */
var buildTree = function(preorder, inorder) {
  // 先序遍历：根左右，第一个节点为根节点
  // 中序遍历：左右根，可根据先序遍历找到根节点左右子树
  let rootMap = new Map();
  for (let i = 0; i < inorder.length; i++) {
    // 存放根节点的值在inorder中对应的位置，避免反复扫描
    rootMap.set(inorder[i], i);
  }

  var func = function (root, l, r) {
    // root: 从先序中取根节点值的索引
    // l: inorder 的左边界
    // r: inorder 的右边界
    if (l > r)  return null;

    // rootInInd: 当前根在中序数组的位置
    // leftPreInd/rightPreInd：左右子树根节点在前序数组中的位置
      
    // preorder[root]是当前根节点的值，并由此获取当前根节点在中序数组中的位置
    var rootInInd = rootMap.get(preorder[root]);
    
    // 获取到了根节点在中序数组中的位置，就能找到其左右两侧的左右子树的节点数目
    // 进而在先序数组中找到左右子树根节点的索引
    var leftCnt = rootInInd - l, leftPreInd = root + 1;
    var rightCnt = r - rootInInd, rightPreInd = root + leftCnt + 1;

    var rootNode = new TreeNode(preorder[root]);

    // 递归地
    rootNode.left = func(leftPreInd, l, rootInInd - 1);
    rootNode.right = func(rightPreInd, rootInInd + 1, r);

    return rootNode;
  }

  // 初始时，根节点即为先序第一个，此时需要扫描整个中序数组，因此 l = 0, r = len - 1
  return func.call(func, 0, 0, inorder.length - 1);
};

```

#### 4、树的子结构

<img src="assets/image-20220218143017039.png" alt="image-20220218143017039" style="zoom: 95%;" />

<img src="assets/image-20220218143325988.png" alt="image-20220218143325988" style="zoom:98%;" />

![image-20220218143344098](assets/image-20220218143344098.png)

```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} A
 * @param {TreeNode} B
 * @return {boolean}
 */
var isSubStructure = function(A, B) {
    if (A === null || B === null) {
        return false;
    }

    const recur = (A, B) => {
        // 判断B是否是A的子结构，依次左右节点递归地对比
        if (B === null) {
            return true;
        }
        if (A === null) {
            return false;
        }
        if (A.val === B.val) {
            return recur(A.left, B.left) && recur(A.right, B.right);
        } else {
            return false;
        }
    }
    // 对A和B先整体判断是否是子结构，再递归地判断A的左子树和A的右子树
    return recur(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
};
```

