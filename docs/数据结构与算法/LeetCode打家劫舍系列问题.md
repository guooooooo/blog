---
title: LeetCode打家劫舍系列问题
date: 2019-12-21
tags:
 - 数据结构与算法
categories:
 - 数据结构与算法
---

## 1.相当于在一列数组中取出一个或多个不相邻数，使其和最大。

 ```js
输入: [2,7,9,3,1]
输出: 12
解释: 偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
     偷窃到的最高金额 = 2 + 9 + 1 = 12 。
 ```

> *f*(*k*) = 从前 *k* 个房屋中能抢劫到的最大数额，A<sub>i</sub> = 第 i 个房屋的钱数。

> 面对每个房屋，只有抢或者不抢两个选择。

状态转移方程：

> *f*(*k*) = max(*f*(*k* – 2) + A<sub>k</sub>, *f*(*k* – 1))

### DP数组

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var rob = function(nums) {
  const maxMoney = []
  if (nums.length === 0) {
    return 0
  }
  if (nums.length === 1) {
    return nums[0]
  }
  maxMoney[0] = nums[0]
  maxMoney[1] = Math.max(nums[0], nums[1])
  for (let i = 2; i < nums.length; i++) {
    maxMoney[i] = Math.max(maxMoney[i-1], maxMoney[i-2] + nums[i])
  }
  return maxMoney[maxMoney.length-1]
};
```

### 优化空间复杂度

状态转移只和*f*(*k*)前面的两个状态有关，所以可以进一步优化，将空间复杂度降低到 O(1)。

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var rob = function(nums) {
  let prevMax = 0, curMax = 0
  for (let i = 0; i < nums.length; i++) {
    let temp = curMax
    curMax = Math.max(prevMax + nums[i], curMax)
    prevMax = temp
  }
  return curMax
};
```

## 2.相当于数组变成了环形，首尾相连。

因为首尾相连了，所以第一家和最后一家只能抢其中的一家，或者都不抢，那这里变通一下，如果把第一家和最后一家分别去掉，各算一遍能抢的最大值，然后比较两个值取其中较大的一个即为所求。

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var rob = function(nums) {
  if (nums.length === 0) {
    return 0
  }
  if (nums.length === 1) {
    return nums[0]
  }
  function helper(nums) {
    let prevMax = 0, curMax = 0
    for (let i = 0; i < nums.length; i++) {
      let temp = curMax
      curMax = Math.max(prevMax + nums[i], curMax)
      prevMax = temp
    }
    return curMax
  }
  return Math.max(helper(nums.slice(0, nums.length-1)), helper(nums.slice(1)))
};
```

## 3.数组变成了二叉树

```js
输入: [3,4,5,1,3,null,1]

     3
    / \
   4   5
  / \   \ 
 1   3   1

输出: 9
解释: 小偷一晚能够盗取的最高金额 = 4 + 5 = 9.
```

### 暴力递归

状态转移方程：

> result(root) = max((root.val+result(孙子节点) ), (result(左子节点)+result(右子节点)))

```js
var rob = function(root) {
  if (!root) {
    return 0
  }
  let max = root.val
  if (root.left) {
    max += (rob(root.left.left) + rob(root.left.right))
  }
  if (root.right) {
    max += (rob(root.right.left) + rob(root.right.right))
  }
  const res = Math.max(max, rob(root.left) + rob(root.right))
  return res
};
```

### 记忆化递归

我们发现爷爷节点在计算自己能偷多少钱的时候，同时计算了4个孙子节点能偷多少钱，也计算了2个儿子节点能偷多少钱。这样在儿子节点当爷爷节点时，就会产生重复计算一遍孙子节点。

可以使用Map来做缓存优化，避免重复计算。

```js
let memo = new Map()
var rob = function(root) {
  if (!root) {
    return 0
  }
  if (memo.has(root)) {
    return memo.get(root)
  }
  let max = root.val
  if (root.left) {
    max += (rob(root.left.left) + rob(root.left.right))
  }
  if (root.right) {
    max += (rob(root.right.left) + rob(root.right.right))
  }
  const res = Math.max(max, rob(root.left) + rob(root.right))
  memo.set(root, res)
  return res
};
```

### DP + 递归

使用一个长度为2的数组来表示当前节点的偷到的最大钱 0代表不偷，1代表偷
**任何一个节点能偷到的最大钱的状态可以定义为**

- 当前节点选择不偷: 当前节点能偷到的最大钱数 = 左孩子能偷到的钱 + 右孩子能偷到的钱
- 当前节点选择偷: 当前节点能偷到的最大钱数 = 左孩子选择自己不偷时能得到的钱 + 右孩子选择不偷时能得到的钱 + 当前节点的钱数

公式表示：

```js
root[0] = Math.max(rob(root.left)[0], rob(root.left)[1]) + Math.max(rob(root.right)[0], rob(root.right)[1])
root[1] = rob(root.left)[0] + rob(root.right)[0] + root.val;
```

代码：

```js
var rob = function(root) {
  let result = dp(root)
  return Math.max(result[0], result[1])
}

function dp(root) {
  if (!root) {
    return [0,0]
  }
  let result = []

  let left = dp(root.left)
  let right = dp(root.right)

  result[0] = Math.max(left[0], left[1]) + Math.max(right[0], right[1])
  result[1] = root.val + left[0] + right[0]

  return result
}
```

