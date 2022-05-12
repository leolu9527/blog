---
title: "Go 标准库container/ring 循环链表"
date: 2022-05-11T14:03:01+08:00
description: ""
tags: [Go标准库, Go数据结构]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: Go学习笔记
comment: false
draft: false
---
GO标准库中提供了两种链表，双向链表container/list和循环链表container/ring，链表适合快速增删而不适合快速查询。
## 结构定义

```go
type Ring struct {
	next, prev *Ring
	Value      any // for use by client; untouched by this library
}
```
## 方法
关注Len()这个方法
```go

// Len computes the number of elements in ring r.
// It executes in time proportional to the number of elements.
//
func (r *Ring) Len() int {
	n := 0
	if r != nil {
		n = 1
		for p := r.Next(); p != r; p = p.next {
			n++
		}
	}
	return n
}
```
不同于container/list，ring没有额外的结构体去记录长度，因此计算链表长度只能循环计数。

## 代码示例

```go
package main

import (
	"container/ring"
	"fmt"
)

func main() {
	r := ring.New(5)
	n := r.Len()
	for i := 0; i < n; i++ {
		r.Value = i
		r = r.Next()
	}
	// Iterate through the ring and print its contents
	r.Do(func(p any) {
		fmt.Println(p.(int))
	})

}
```
