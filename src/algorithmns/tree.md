# 树：

树状结构是我们在日常开发中常见的一种数据结构，基于其分叉与特定情况下的特点，它可以在一些特定场景下的运行效率达到 `log(n)` 级别，从而减少时间上的开销。

树的相关概念：
- 高度（Height）：（从下往上度量）
  - 节点的高度：节点到叶子节点的最长路径（边数）
  - 树的高度：根节点的高度
- 深度（Depth）：根节点到这个节点所经历的边的个数（从上往下度量）
- 层（Level）：节点的深度 + 1 （计数起点是1）

## 常见的树：

### 二叉树
**定义**：每个节点最多有两个“叉”， 分别为左子节点和右子节点。

**二叉树的存储方式**：
- 链式存储法：每个节点有三个字段，其中一个存储数据，另外两个是指向左右子节点的指针
- 顺序存储法（基于数组）（只适用于完全二叉树，因为非完全二叉树会浪费很多空间）：我们把根节点存储在下标 `i = 1` 的位置，那左子节点存储在下标 `2 * i` 的位置，右子节点存储在 `2 * i + 1` 的位置


**二叉树的类型**：
- 满二叉树：叶子节点全部在最底层，除了叶子节点之外，每个节点都有左右两个子节点。
- 完全二叉树：叶子节点都在最底下两层，最后一层的叶子节点都靠左排列，并且除了最后一层，其他层的节点个数都要达到最大。

**遍历方式**:
- 前序遍历：根 -> 左 -> 右
- 中序遍历：左 -> 根 -> 右
- 后序遍历：左 -> 右 -> 根


## 常见算法：

#### 94. 二叉树的中序遍历

- 递归：
```javascript
var inorderTraversal = function(root) {
    if (!root) return [];

    const res = [];
    const inOrderTraversal = (node) => {
        if (!node) return;
        inOrderTraversal(node.left);
        res.push(node.val);
        inOrderTraversal(node.right);
    }

    inOrderTraversal(root);
    return res;
};
```
- 模拟栈:
```javascript
var inorderTraversal = function(root) {
    if (!root) return [];

    const res = [];
    const stk = [];

    while (root || stk.length) {
        while (root) {
            stk.push(root);
            root = root.left;
        }

        root = stk.pop();
        res.push(root.val);
        root = root.right;
    }

    return res;
};
```

#### lc104. 二叉树的最大深度：
**description**： 给定一个二叉树 root ，返回其最大深度。二叉树的 最大深度 是指从根节点到最远叶子节点的最长路径上的节点数。

**示例**：

```plain
输入：root = [3,9,20,null,null,15,7]
输出：3

输入：root = [1,null,2]
输出：2
```

**解决方案**：
可以通过深度优先遍历或广度优先遍历来解决这个问题，给出深度优先遍历的解法：
```javascript
/**
 * Definition for a binary tree node.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
var maxDepth = function(root) {
    if (!root) return 0;

    let maxDep = 1;

    const dfs = (tNode, currDepth) => {
        if (!tNode) return;

        maxDep = Math.max(maxDep, currDepth);
        dfs(tNode.left, currDepth + 1)
        dfs(tNode.right, currDepth + 1)
    }
    dfs(root, maxDep);
    return maxDep;
};

```

#### lc102. 二叉树的层序遍历：

**description**： 给你二叉树的根节点 root ，返回其节点值的 层序遍历 。 （即逐层地，从左到右访问所有节点）。

**示例**：

```plain
输入：root = [3,9,20,null,null,15,7]
输出：[[3],[9,20],[15,7]]

输入：root = [1]
输出：[[1]]
```

**解决方案**：
使用广度优先遍历解决这个问题：
```javascript
var levelOrder = function(root) {
    if (!root) return [];

    const res = [];
    const queue = [root];

    while (queue.length) {
        const len = queue.length;
        const path = [];
        
        for (let i = 0; i < len; i++) {
            const tNode = queue.shift();
            path.push(tNode.val);

            if (tNode.left) queue.push(tNode.left);
            if (tNode.right) queue.push(tNode.right);
        }
        res.push(path);
    }

    return res;
};
```

#### lc108. 将有序数组转换为二叉搜索树

```javascript
/**
 * @param {number[]} nums
 * @return {TreeNode}
 */
var sortedArrayToBST = function(nums) {
    return dfs(nums, 0, nums.length - 1);
};

const dfs = (nums, left, right) => {
    if (left > right) return null;

    let mid = Math.floor((left + right) / 2);

    const root = new TreeNode(nums[mid]);
    root.left = dfs(nums, left, mid - 1);
    root.right = dfs(nums, mid + 1, right);

    return root;
}
```

