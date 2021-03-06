# 红黑树

红黑树是一个平衡的二叉树，但不是一个完美的平衡二叉树。虽然我们希望一个所有查找都能在~lgN次比较内结束，但是这样在动态插入中保持树的完美平衡代价太高，所以，我们稍微放松逛一下限制，希望找到一个能在对数时间内完成查找的数据结构。这个时候，红黑树站了出来。

## 红黑树简介

红黑树\(R-B Tree\)，一种二叉查找树，但在每个结点上增加一个存储位表示结点的颜色，可以是Red或Black。  
通过对任何一条从根到叶子的路径上各个结点着色方式的限制，红黑树确保没有一条路径会比其他路径长出俩倍，因而是接近平衡的。

红黑树虽然本质上是一棵二叉查找树，但它在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为O\(log n\)。

## 红黑树的性质

1. 每个结点要么是红的要么是黑的。  
2. 根结点是黑的。  
3. 每个叶结点（叶结点即指树尾端NIL指针或NULL结点）都是黑的。  
4. 如果一个结点是红的，那么它的两个儿子都是黑的。  
5. 对于任意结点而言，其到叶结点树尾端NIL指针的每条路径都包含相同数目的黑结点。 

## reference

* [https://blog.csdn.net/v\_JULY\_v/article/details/6105630](https://blog.csdn.net/v_JULY_v/article/details/6105630)

