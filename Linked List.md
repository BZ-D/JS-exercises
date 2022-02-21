链表从 JS 的角度来说就是一串对象使用指针连接的数据结构，通过`next`指针改变指向。

#### 1、链表排序

使用自上而下的归并算法进行排序，使用`slow.next`和`fast.next.next`两种速度获取链表节点，从而获取中间值。

```js
var sortList = function (head) {
    if (head === null || head.next === null) return head;
    
    let prev = null, slow = head, fast = head;
    while (fast !== null && fast.next !== null) {
        prev = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    prev.next = null;
    
    let l1 = sortList(head);
    let l2 = sortList(slow);
    
    return merge(l1, l2);
};

function merge(l1, l2) {
    let l = new ListNode(0), p = l;
    while (l1 !== null && l2 !== null) {
        if (l1.val < l2.val) {
            p.next = l1;
            l1 = l1.next;
        } else {
            p.next = l2;
            l2 = l2.next;
        }
        p = p.next;
    }
    
    if (l1 !== null) p.next = l1;
    if (l2 !== null) p.next = l2;
    return l.next;
}
```

#### 2、链表倒序

```js
// 迭代：时间复杂度O(n)
// 遍历时，将当前节点的next指针改为指向前一节点
// 由于节点没有引用其前一个节点，因此必须事先存储其前一个节点
// 在更改引用之前，还需要存储后一个节点
// 最后返回新的头引用
var reverseList = function (head) {
    // 一开始prev设为null是因为原链表的头结点在反转后的链表中是尾结点
    // 因此原链表头结点在反转后应指向null
    let prev = null, cur = head;
    while (cur !== null) {
        // 将当前节点指向前一节点
        let next = cur.next;
        cur.next = prev;
        // 挪到后一位置
        prev = cur;
        cur = next;
    }
    return prev;
};
```

#### 3、双指针法

<img src="assets/image-20220206133804497.png" alt="image-20220206133804497" style="zoom:67%;" />

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
var removeNthFromEnd = function(head, n) {
    const dummy = new ListNode(0, head);
    let first = head;
    let second = dummy;

    // 让右指针领先左指针n步，这样当右指针到头时，左指针指向的节点.next即为要删除的倒数第N个节点
    for (let i = 0; i < n; i++) {
        first = first.next;
    }

    while (first !== null) {
        first = first.next;
        second = second.next;
    }

    second.next = second.next.next;
    return dummy.next;
};
```

#### 4、复杂链表的深拷贝

<img src="assets/image-20220219143907214.png" alt="image-20220219143907214" style="zoom:67%;" />

**重点：**

- 深拷贝，对每个节点都要重新 new 一个新节点
- 含有 random 属性，在所有节点都遍历完之前，没法获知每个节点的 random 信息，可能出现第一个节点的 random 指向最后一个节点，但此时最后一个新节点并没有 new 出来

**思路：**

- 哈希表，既然可能出现对 random 属性赋值时对应节点没 new 出来的情况，那第一遍遍历时干脆不处理 next 和 random 属性，而是先建立一个 `原节点 -> 新节点` 的映射表
- 第二次遍历时，所需的 next 和 random 节点都已经在第一遍遍历中 new 出来了，直接根据映射关系赋值即可

```js
var copyRandomList = function(head) {
    if (head === null) {
        return null;
    }

    let node = head;
    let map = new Map();
    
    // 先建立源节点->新节点的映射
    while (node) {
        map.set(node, new Node(node.val));
        node = node.next;
    }
	
    // 第二次遍历时即可赋值
    node = head;
    while (node) {
        map.get(node).next = map.get(node.next) ? map.get(node.next) : null;
        map.get(node).random = map.get(node.random) ? map.get(node.random) : null;
        node = node.next;
    }

    return map.get(head);
}

```



#### 5、找出两链表的第一个公共节点

<img src="assets/image-20220221204752871.png" alt="image-20220221204752871" style="zoom:67%;" />

- 设A的总长为a，B的总长为b，公共长度为c，则在公共节点之前，A部分长度为(a-c)，B部分长度为(b-c)
- 当A走到头时，让A从B的头部开始走，当B走到头时，让B从A的头部开始走，则相遇时，A走了(a+b-c)，B走了(b+a-c)，可见两者长度一致，因此会相遇

```js
var getIntersectionNode = function(headA, headB) {
  if (headA === null || headB === null) return null;

  let ha = headA, hb = headB;
  while (ha !== hb) {
    ha = ha === null ? headB : ha.next;
    hb = hb === null ? headA : hb.next;
  }
  return ha;
};
```

