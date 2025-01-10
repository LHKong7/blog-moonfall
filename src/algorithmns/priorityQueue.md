# 优先队列 Priority Queue

## 定义：
特殊的队列，队列中的每个元素有一个特殊的 `priority value` 优先值。高优先级的元素在队列的前面。（方便弹出）

Implementation of PQ：
- 数组
- linked list 链表
- heap 堆

通过不同数据结构生成优先队列：
Operations	peek	insert	delete
Linked List	O(1)	O(n)	O(1)
Binary Heap	O(1)	O(log n)	O(log n)
Binary Search Tree	O(1)	O(log n)	O(log n)


## 操作：

### Insertion
- 将节点插入到队尾
- heapify

### Deletion
- 将将要删除的元素与最后一个元素交换
- 删除最后一个元素
- heapify



