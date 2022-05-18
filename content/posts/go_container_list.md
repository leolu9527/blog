---
title: "Go数据结构 标准库container/list 双向链表"
date: 2021-09-11T11:41:08+08:00
description: "Go标准库 container/list 的使用笔记"
tags: [Go标准库, Go数据结构]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: Go学习笔记
comment: false
draft: false
---

Golang 标准库提供了一个双向链表container/list，看下源码是如何实现的.

## Element结构体
```go
type Element struct {
	// Next and previous pointers in the doubly-linked list of elements.
	// To simplify the implementation, internally a list l is implemented
	// as a ring, such that &l.root is both the next element of the last
	// list element (l.Back()) and the previous element of the first list
	// element (l.Front()).
	next, prev *Element

	// The list to which this element belongs.
	list *List

	// The value stored with this element.
	Value any
}
```
上面的结构体作为链表的元素结构，相较于简单结构的链表这里多了一个`*List`.
## List结构体
我们通过New()方法来创建一个新的链表:
```go
// List represents a doubly linked list.
// The zero value for List is an empty list ready to use.
type List struct {
    root Element // sentinel list element, only &root, root.prev, and root.next are used
    len  int     // current list length excluding (this) sentinel element
}

// Init initializes or clears list l.
func (l *List) Init() *List {
    l.root.next = &l.root
    l.root.prev = &l.root
    l.len = 0
    return l
}

// New returns an initialized list.
func New() *List { return new(List).Init() }
```
可以看到，New()调用了Init()方法，主要作用就是初始化一个链表的根元素.
实际上，这个list是一个环状结构，初始化后root作为根元素起到连接首尾的作用，但是不存储数据。


![链表元素结构](/images/list_elements.png "链表元素结构")


下面*List的两个方法Front()和Back()可以清楚的看出根元素的作用：
```go
// Front returns the first element of list l or nil if the list is empty.
func (l *List) Front() *Element {
	if l.len == 0 {
		return nil
	}
	return l.root.next
}

// Back returns the last element of list l or nil if the list is empty.
func (l *List) Back() *Element {
	if l.len == 0 {
		return nil
	}
	return l.root.prev
}
```
## 使用示例
```go
package main

import (
	"container/list"
	"fmt"
)

func main() {
	l := list.New()
	e4 := l.PushBack(4)
	e1 := l.PushFront(1)
	l.InsertBefore(3, e4)
	l.InsertAfter(2, e1)

	fmt.Println(l.Back().Value)

	// Iterate through list
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Println(e.Value)
	}
}
```
