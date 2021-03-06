# 数据结构

**数据结构考点**：
- 数组和链表的区别；
- 链表的操作，如反转，链表环路检测，双向链表，循环链表相关操作；
- 队列，栈的应用；
- 二叉树的遍历方式及其递归和非递归的实现；
- 红黑树的旋转

## HashMap
extends Map Interface 


In Java 8, when we have too many unequal keys whih gives same hashcode(index) - when the number of items in a hash bucket grows beyond certain threshold(TREEIFY_THRESHOLD = 8), content of that bucket switches from using a linked list of Entry objects to a balanced tree. 


## AVL（平衡二叉树）
AVL树具有以下性质：它是一棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。

其高度一般都良好地维持在O(logn)，其各操作的时间复杂度在平均和最坏情况下都是O(logn)

增加和删除可能需要通过一次或多次树旋转来重新平衡这个树。

## 红黑树
自平衡的二叉查找树，为了解决二叉查找树对插入新节点而导致的不平衡现象

特性：
1. 每个节点或者是黑色，或者是红色
2. 根节点是黑色
3. 每个叶子节点（NIL）是黑色 [注意：这里说的叶子节点，是指为空（NIL 或 NULL）的叶子节点！]
4. 如果一个节点是红色的，则它的子节点必须是黑色的
5. 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑色节点

