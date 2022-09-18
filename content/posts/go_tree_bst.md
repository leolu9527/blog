---
title: "Go数据结构 二叉搜索树(BST)"
date: 2022-05-31T17:39:27+08:00
description: "二叉搜索树的基础"
tags: [数据结构, 二叉树]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: Go学习笔记
comment: false
draft: false
---

接上篇[Go数据结构 树(tree) 二叉树基础](/posts/go_tree/)内容，这篇介绍二叉搜索树。

## 二叉搜索树的定义

**二叉搜索树**（Binary Search Tree），也称为**二叉查找树**、**有序二叉树**（ordered binary tree）或**排序二叉树**（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

1. 若任意节点的左子树不空，则左子树上所有节点的值均小于它的根节点的值；
2. 若任意节点的右子树不空，则右子树上所有节点的值均大于它的根节点的值；
3. 任意节点的左、右子树也分别为二叉搜索树；
4. **没有键值相等的节点**

二叉搜索树示例：[5,1,7,null,null,6,9]
```
    5
   / \
  1   7
     / \
    6   9
```

非二叉搜索树示例：[5,1,7,null,null,4,9]
```
    5
   / \
  1   7
     / \
    4   9
```
解释: 节点4小于根节点

二叉搜索树相比于其他数据结构的优势在于查找、插入的时间复杂度较低。为$O(\log n)$。

## 二叉搜索树的判断

根据二叉搜索树的三个性质，我们容易想到使用递归来逐个节点判断是否满足 `左节点 < 节点值 < 右节点` 条件，但是会忽略
**子树上的所有节点值都需要小于（左子树）或者大于（右子树）该节点**这个规则。于是需要在判断每个节点时引入上界与下界，即当前节点允许的范围，超出
范围即表示不是一个二叉搜索树了。

总结就是每次递归中，我们除了**进行左右节点的校验，还需要与上下界进行判断**。代码如下：

```go
// IsBST 判断是否是二叉搜索树
func IsBST(root *Node) bool {
	if root == nil {
		return true
	}
	return isBSTRecursion(root, math.MinInt64, math.MaxInt64)
}

func isBSTRecursion(root *Node, min, max int) bool {
	if root == nil {
		return true
	}
	if min >= root.Val || max <= root.Val {
		return false
	}
	return isBSTRecursion(root.Left, min, root.Val) && isBSTRecursion(root.Right, root.Val, max)
}
```

## 二叉搜索树查找

二叉搜索树的查找过程：

1. 如果val小于当前结点的值，转向其左子树继续搜索； 
2. 如果val大于当前结点的值，转向其右子树继续搜索； 
3. 如果已找到，则返回当前结点。

下面是迭代和递归两种解法：

```go

// SearchBSTRecursion 递归查找二叉搜索树
func SearchBSTRecursion(root *Node, v int) *Node {
	if root == nil {
		return nil
	}
	if root.Val > v {
		return SearchBSTRecursion(root.Left, v)
	} else if root.Val < v {
		return SearchBSTRecursion(root.Right, v)
	} else {
		return root
	}
}

// SearchBSTIterate 迭代查找二叉搜索树
func SearchBSTIterate(root *Node, v int) *Node {
	for root != nil {
		if root.Val == v {
			return root
		} else if root.Val > v {
			root = root.Left
		} else {
			root = root.Right
		}
	}

	return nil
}

```

使用：

```go
root := tree.BuildTree([]int{5, 1, 7, 0, 0, 6, 9})
if tree.IsBST(root) {
    fmt.Println(tree.SearchBSTRecursion(root, 7))
    fmt.Println(tree.SearchBSTIterate(root, 7))
}
```

输出：
```shell
&{7 0xc0000040c0 0xc0000040d8}
&{7 0xc0000040c0 0xc0000040d8}
```


## 二叉搜索树删除

二叉搜索树在删掉某个节点后需要维持它的性质不变。

leetcode题目 [450. 删除二叉搜索树中的节点](https://leetcode.cn/problems/delete-node-in-a-bst/)

删除节点分为两个步骤： 首先找到需要删除的节点，如果找到了，删除它。

删除节点有三种情况：

>第一种，**待删除的节点是叶子节点**。这样的情况只需要将该结点的父结点指向空即可。

示例：[5,3,6,2,4,null,7]

删除节点：7
```
    5                     5
   / \                   / \
  3   6        =>       3   6
 / \   \               / \   
2   4   7             2   4   
```

>第二种，**待删除的节点有一个子树(左子树或右子树)。** 此时删除该结点的方法就是让该结点的父结点指向该结点的子树即可。

示例：[5,3,6,2,null,null,9,null,null,null,null,null,null,7,10]

删除节点：6
```
    5                     5
   / \                   / \
  3   6        =>       3   9
 /     \               /   / \
2       9             2   7  10
       / \
      7  10
```

>第三种，待删除的节点的左右子树都不为空。这个时候我们要采取的方法就是：从左子树里找一个最大值（前驱）来代替，或者从右子树里找一个最小值（后继）来代替。

示例：[15,  10,16,7,11,null,17,6,9,null,13,null,null,null,null,null,null,8,null,null,null,12]

删除节点：10

```
            15                        15
           /  \                      /  \
          10   16                   11   16
         /  \    \                 /  \    \
        7   11   17      =>       7    13   17
       / \   \                   / \   /
      6   9  13                 6  9  12
         /   /                    /
        8   12                   8
```

下面代码使用后继节点来替换被删除节点：

```go
//DeleteBSTNode 二叉搜索树删除（后继节点替代被删除节点）
func DeleteBSTNode(root *Node, key int) *Node {
	if root == nil {
		return nil
	}
	if key < root.Val {
		root.Left = DeleteBSTNode(root.Left, key)
		return root
	}
	if key > root.Val {
		root.Right = DeleteBSTNode(root.Right, key)
		return root
	}
	//已经查找到目标
	if root.Right == nil {
		//右子树为空
		return root.Left
	}
	if root.Left == nil {
		//左子树为空
		return root.Right
	}
	minNode := root.Right
	for minNode.Left != nil {
		//查找后继
		minNode = minNode.Left
	}
	root.Val = minNode.Val
	root.Right = deleteMinNode(root.Right)
	return root
}

func deleteMinNode(root *Node) *Node {
	if root.Left == nil {
		pRight := root.Right
		root.Right = nil
		return pRight
	}
	root.Left = deleteMinNode(root.Left)
	return root
}
```

调用：
```go
root := tree.BuildTree([]int{15, 10, 16, 7, 11, 0, 17, 6, 9, 0, 13, 0, 0, 0, 0, 0, 0, 8, 0, 0, 0, 12})
newRoot := tree.DeleteBSTNode(root, 10)
if tree.IsBST(newRoot) {
    fmt.Println(tree.ConvertToArr(newRoot))
}
```

输出：
```shell
[15 11 16 7 13 0 17 6 9 12 0 0 0 0 0 0 0 8]
```

## 附录
代码：[https://github.com/leolu9527/go-algorithms/tree/main/tree](https://github.com/leolu9527/go-algorithms/tree/main/tree)

## References
1. https://zhuanlan.zhihu.com/p/136758152