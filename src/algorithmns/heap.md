# Heap/binary heap

## 定义：
一个完全二叉树的数据结构，满足heap的特性；给定一个heap的node节点：
- 如果是Max heap：父级节点永远大于其所有的子节点。
- 如果是Min heap：父级节点永远小于其所有的子节点。

## Heap操作：

### Heapify
用来创建一个大顶堆/小顶堆的方法。其中重要的步骤有：
1. 从第一个非叶子节点的值 `(n/2) - 1` n为元素长度
2. 对比其与叶子节点的关系 `2i + 1 & 2i + 2`
3. 

### Insertion
1. 将节点插入到数组中的最后
2. 调用 heapify


### Deletion 删除
1. 将选中的节点与最后一个节点调换
2. 删除最后一个节点
3. heapify



