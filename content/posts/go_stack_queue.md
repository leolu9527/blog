---
title: "Go 栈 Stack 和队列 Queue的简单实现"
date: 2022-05-13T11:35:48+08:00
description: ""
tags: [Go数据结构]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: Go学习笔记
comment: false
draft: false
---

## 一、栈 Stack 和队列 Queue
栈和队列跟链表一样都属于线性表，它们都是一组“数据元素”按照一定的顺序依次排列的结构。

栈和队列在删除和访问数据时有如下特点：
1. 栈：先进后出，先进队的数据最后才出来。
2. 队列：先进先出，先进队的数据先出来。

我们可以用不同的数据结构 `数组（Array）` 或者 `链表（Linked List）` 来实现 `栈（stack`） 和 `队列 (queue)`。

下面会展示栈的简单实现代码， 首先我们约定一个栈的基本结构：

```go
// Stack 栈
type Stack interface {
    Push(s string)         //入栈
    Pop() (string, error)  //出栈
    Peek() (string, error) //获取栈顶元素,不做数据删除
}
```

## 二、数组栈 `ArrayStack`

数组栈的存储形式如下图所示：

![ArrayStack存储形式](/images/ArrayStack.png "ArrayStack")

考虑到栈的长度不是固定的，我们使用Go的slice切片来做底层:

```go
// ArrayStack 数组栈
type ArrayStack struct {
	array   []string   //切片作为底层
	maxSize int        //栈大小
	lock    sync.Mutex //并发安全锁
}
```
下面我们实现3个操作。

### 2.1 入栈 `Push`

```go
// Push 入栈
func (stack *ArrayStack) Push(s string) {
	stack.lock.Lock()
	defer stack.lock.Unlock()
	//插入数据到栈顶
	stack.array = append(stack.array, s)
	stack.maxSize++
}
```
### 2.2 出栈 `Pop`

```go
// Pop 出栈
func (stack *ArrayStack) Pop() (string, error) {
	stack.lock.Lock()
	defer stack.lock.Unlock()
	if stack.maxSize == 0 {
		return "", errors.New("empty")
	}
	s := stack.array[stack.maxSize-1]

    // 切片收缩
    //stack.array = stack.array[0 : stack.maxSize-1]

    //这里使用copy函数，copy函数会丢弃原切片中较短最后一个元素
	destArray := make([]string, stack.maxSize-1, stack.maxSize-1)
	copy(destArray, stack.array)
	stack.array = destArray

	//循环
	//newArray := make([]string, stack.maxSize-1, stack.maxSize-1)
	//for i := 0; i < stack.maxSize-1; i++ {
	//	newArray[i] = stack.array[i]
	//}
	//stack.array = newArray
	
	// 栈中元素数量-1
	stack.maxSize--
	return s, nil
}
```
入栈和出栈操作都加上了互斥锁。

当栈大小 `maxSize=0` 时为空栈，此时出栈会有`empty` error抛出，但不影响栈的继续运行。

栈顶元素取出后，可以有三种方式缩容：
1. 切片偏移量向前移动 stack.array[0 : stack.maxSize-1]，此时，切片被缩容
2. 使用内置的`copy`函数来建一个新的切片
3. 循环赋值一个新的切片

Benchmark测试三种方式差别很小。

### 2.3 获取栈顶元素 `Peek`

```go
// Peek 获取栈顶元素
func (stack *ArrayStack) Peek() (string, error) {
	stack.lock.Lock()
	defer stack.lock.Unlock()
	if stack.maxSize == 0 {
		return "", errors.New("empty")
	}
	return stack.array[stack.maxSize-1], nil
}
```
考虑到 `maxSize`在出栈和入栈中都有联动，需要加锁保证并发安全。

## 三、链表栈 `LinkStack`

链表栈的存储形式如下图所示：

![LinkStack存储形式](/images/LinkStack.png "LinkStack")

定义结构如下：
```go
// LinkStack 单链表栈
type LinkStack struct {
	root *LinkNode  //栈顶
	size int        //栈深
	lock sync.Mutex //并发锁
}

type LinkNode struct {
	value string
	Next  *LinkNode
}
```

`LinkStack`结构持有栈顶元素`root`及栈的大小`size`。

### 3.1 入栈 `Push`

```go
// Push 入栈
func (stack *LinkStack) Push(s string) {
	stack.lock.Lock()
	defer stack.lock.Unlock()
	lastNode := stack.root
	node := new(LinkNode)
	node.value = s
	node.Next = lastNode
	stack.root = node
	stack.size++
}
```

### 3.2 出栈 `Pop`

```go
// Pop 出栈
func (stack *LinkStack) Pop() (string, error) {
	stack.lock.Lock()
	defer stack.lock.Unlock()
	if stack.root == nil {
		return "", errors.New("empty")
	}
	s := stack.root.value
	nextNode := stack.root.Next
	stack.root = nextNode
	stack.size--
	return s, nil
}
```

链表栈的操作主要是栈顶元素的变更。

### 3.3 获取栈顶元素 `Peek`

```go
// Peek 获取栈顶元素
func (stack *LinkStack) Peek() (string, error) {
	stack.lock.Lock()
	defer stack.lock.Unlock()
	if stack.size == 0 {
		return "", errors.New("empty")
	}
	s := stack.root.value

	return s, nil
}
```

## 四、测试代码

`stack_test.go`
```go
package stack

import (
	"math/rand"
	"sync"
	"testing"
)

const letterBytes = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

const stringLen = 16

const stackSize = 1000

func RandStringBytes(n int) string {
	l := len(letterBytes)
	b := make([]byte, n)
	for i := range b {
		b[i] = letterBytes[rand.Intn(l)]
	}
	return string(b)
}

// test stack Push by interface
func testStackPush(t *testing.T, stack Stack) {
	for i := 0; i < stackSize; i++ {
		s := RandStringBytes(stringLen)
		stack.Push(s)
		p, err := stack.Peek()
		if err != nil || p != s {
			t.Errorf("push:%s != peek: %s", s, p)
		}
	}
}

// test stack Pop by interface
func testStackPop(t *testing.T, stack Stack) {
	var storage [stackSize]string
	for i := 0; i < stackSize; i++ {
		s := RandStringBytes(stringLen)
		storage[i] = s
		stack.Push(s)
		p, err := stack.Peek()
		if err != nil || p != s {
			t.Errorf("push:%s, peek: %s", s, p)
		}
	}

	for i := 0; i < stackSize; i++ {
		p, err := stack.Pop()
		if err != nil {
			s := storage[stackSize-1-i]
			if p != s {
				t.Errorf("storage:%s != pop: %s", s, p)
			}
		}
	}
}

// test stack On Goroutines by interface
func testStackOnGoroutine(t *testing.T, stack Stack) {
	wg := sync.WaitGroup{}
	for i := 0; i < stackSize; i++ {
		s := RandStringBytes(stringLen)
		wg.Add(1)
		go func() {
			stack.Push(s)
		}()
	}

	go func() {
		for {
			_, err := stack.Pop()
			if err == nil {
				wg.Done()
			}
		}
	}()

	wg.Wait()
}

func TestArrayStackPush(t *testing.T) {
	stack := new(ArrayStack)
	testStackPush(t, stack)
}

func TestArrayStackPop(t *testing.T) {
	stack := new(ArrayStack)
	testStackPop(t, stack)
}

func TestArrayStackOnGoroutine(t *testing.T) {
	stack := new(ArrayStack)
	testStackOnGoroutine(t, stack)
}

func TestLinkStackPush(t *testing.T) {
	stack := new(LinkStack)
	testStackPush(t, stack)
}

func TestLinkStackPop(t *testing.T) {
	stack := new(LinkStack)
	testStackPop(t, stack)
}

func TestLinkStackOnGoroutine(t *testing.T) {
	stack := new(LinkStack)
	testStackOnGoroutine(t, stack)
}

```

执行：
```shell
cd stack
go test -v
```

输出：
```shell
=== RUN   TestArrayStackPush
--- PASS: TestArrayStackPush (0.00s)
=== RUN   TestArrayStackPop         
--- PASS: TestArrayStackPop (0.01s) 
=== RUN   TestArrayStackOnGoroutine
--- PASS: TestArrayStackOnGoroutine (0.00s)
=== RUN   TestLinkStackPush
--- PASS: TestLinkStackPush (0.00s)
=== RUN   TestLinkStackPop
--- PASS: TestLinkStackPop (0.00s)
=== RUN   TestLinkStackOnGoroutine
--- PASS: TestLinkStackOnGoroutine (0.00s)
PASS
ok      github.com/leolu9527/algorithm/stack    0.208s
```

### 4.1 Benchmark

`benchmark_test.go`

```go
package stack

import (
	"fmt"
	"math/rand"
	"testing"
)

// Benchmark for ArrayStack
func BenchmarkArrayStackParallel(b *testing.B) {
	var stack = new(ArrayStack)
	b.RunParallel(func(pb *testing.PB) {
		s := RandStringBytes(16)
		for pb.Next() {
			x := rand.Intn(1)
			if x == 0 {
				stack.Push(s)
			} else {
				_, err := stack.Pop()
				if err != nil {
					fmt.Println(err)
				}
			}
		}
	})
}

// Benchmark for LinkStack
func BenchmarkLinkStackParallel(b *testing.B) {
	var stack = new(LinkStack)
	b.RunParallel(func(pb *testing.PB) {
		s := RandStringBytes(16)
		for pb.Next() {
			x := rand.Intn(1)
			if x == 0 {
				stack.Push(s)
			} else {
				_, err := stack.Pop()
				if err != nil {
					fmt.Println(err)
				}
			}
		}
	})
}

```

执行：
```shell
cd stack
go test -benchmem -bench . -run=none
```

输出：
```shell
goos: windows
goarch: amd64
pkg: github.com/leolu9527/algorithm/stack
cpu: Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
BenchmarkArrayStackParallel-8            8448782               138.2 ns/op            97 B/op          0 allocs/op
BenchmarkLinkStackParallel-8             8633092               129.8 ns/op            24 B/op          1 allocs/op
PASS
ok      github.com/leolu9527/algorithm/stack    2.841s
```

## 附录
代码：[https://github.com/leolu9527/go-algorithms/tree/main/stack](https://github.com/leolu9527/go-algorithms/tree/main/stack)