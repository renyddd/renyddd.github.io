---
title: "Golang Pkg Heap"
date: 2021-06-20T11:26:50+08:00
draft: false
tags: ["golang"]
categories: ["golang"]
series: ["golang系列"]
---

## heap 实现
官方文档地址：[https://golang.org/src/container/heap/heap.go](https://golang.org/src/container/heap/heap.go)

堆就是一棵完全二叉树，并且你也可以用数组的来存储完全二叉树；同时堆也常用来实现优先队列。
> 对于数组中的任意位置 i 上的元素，其左儿子在位置 2i 上，右儿子在左儿子后的 (2i + 1) 中，它的父节点则在 (i/2) 上。

如下 Interface 接口中的 Pop 和 Push 方法，都是声明了为**堆元素**的容器尾部的增删操作方法；也就是用于从自定义的元素集合中真正完成从尾部的增删的方法。

因此想要在客户端代码处完成**从堆中**添加或移除元素时，请使用带有包名的方法：heap.Pop, heap.Push
```go
type Interface interface {
	sort.Interface
	// 实现向尾添加
	Push(x interface{})
	// 实现从尾删除
	Pop() interface{}
}

func Init(h Interface) {
	// 堆化
	n := h.Len() // 为 sort 中接口而实现
	// 从非叶子层开始
	for i := n/2 - 1; i >= 0; i-- {
		// 下滤
		down(h, i, n)
	}
}

// down 这里的下滤描述的是从 i0 节点开始的不断向下 percolate 的过程
func down(h Interface, i0, n int) bool {
	i := i0
	for {
		// 从左孩子开始比较
		j1 := 2*i + 1
		// 注意此处的等于号，否则会因此数组越界 panic
		if j1 >= n || j1 < 0 { // 当整型溢出时 j1 < 0
			break
		}
		j := j1 // 左孩子
		if j2 := j1 + 1; j2 < n && h.Less(j2, j1) {
			// 当其右孩子比左孩子更小时
			j = j2 // 右孩子
		}
		// 以上操作目的是从当前节点的左右孩子中选取一个最小值，给 j

		if !h.Less(j, i) {
			// 注意这是小顶堆，需要将较小的叶子节点交换上来
			break
		}
		// 用左右孩子中的最小值来和跟节点交换
		h.Swap(i, j)
		// 这里的赋值终会导致循环判断条件不符合
		// j 由 j1,j2 而选择出来的左节点 or 右节点，赋值给 i
		// 进行 for 内的下一次 percolate
		// 因为以 Init 中的调用为例，i0 参数由数组中值开始递减为 0
		// 若在初始化的切片中堆顶元素为最大值，且堆顶元素的操作只会在 i0 为 0 时涉及到
		// 交换完最大的堆顶元素后，则必然会导致堆的性质被破坏，因此需要继续下滤
		i = j
	}
	return i > i0
}

// Push 将元素 x 加入到堆中，时间复杂度 O(log n)
// 始终将待插入元素追加至尾部，即二叉堆的最后一个叶子节点，以保证完全二叉树的性质不会被破坏
func Push(h Interface, x interface{}) {
	h.Push(x)
	// 这里默认你已经成功插入了元素在末尾（append）
	// 并且长度信息也已经更新
	// 传递给上滤的参数就是末尾元素，也就是 x
	up(h, h.Len()-1)
}

// up 将末尾元素不断与其父节点进行比较，若更小则交换
func up(h Interface, j int) {
	for {
		i := (j - 1) / 2 // parent
		if i == j || !h.Less(j, i) {
			break
		}
		h.Swap(i, j)
		j = i
	}
}

// Pop 移除并且返回最小值元素
// 时间复杂度是 O(log n)
// 注意，因保证堆的结构性质 故在删除首元素时需要先交换首位元素
// 因此 Interface 中的 Pop 方法交给用户实现的也是删除尾元素
func Pop(h Interface) interface{} {
	n := h.Len() - 1
	// 首先将堆顶元素与末尾元素进行交换
	// 再从堆顶开始下滤，以保持（最小堆的）堆序性质
	h.Swap(0, n)
	down(h, 0, n)
	return h.Pop()
}

// Remove

// Fix
```

## heap 用例

以 [LeetCode: 23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)题目为例，来展现该包的使用方式，首先看看题目：给你一个链表数组，且各链表元素已按升序排序，现需要你将所有链表合并到一个升序链表中，并且返回。

对链表数组使用顺序合并或二分合并的方法在这里略过不提，仅关注使用堆，来存储全部元素，并且一次弹出最小值来构建新链表的解法。

首先是对接口的实现部分代码，如下 Pop 与 Push 的代码均是直接复制了官方示例；需要注意，这两方法均通过指针声明了 intHeap 对象的方法，关于 go Slice 的实现可以参考我的这片[博文。](https://blog.csdn.net/weixin_43391310/article/details/113825072)

```go
import (
	"container/heap"
)

type intHeap []int

func (h intHeap) Len() int           { return len(h) }
func (h intHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h intHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *intHeap) Push(x interface{}) {
	// Push and Pop use pointer receivers because they modify the slice's length,
	// not just its contents.
	*h = append(*h, x.(int))
}

func (h *intHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}
```

有了自定义 intHeap 类型对 heap.Interface 的实现之后，我们就可以通过该包所提供的函数来使用堆的性质了。

```go
func mergeKLists(lists []*ListNode) *ListNode {
	h := &intHeap{}

	for _, l := range lists {
		for p := l; p != nil; p = p.Next {
			// heap.Push(h, p.Val)
			// 之前插入元素时，无需始终保持堆序性质，使用直接向尾部添加的方式即可
			h.Push(p.Val)
		}
	}

	heap.Init(h)

	tail := &ListNode{Val: -1}
	dummyHead := tail

	for h.Len() > 0 {
		node := heap.Pop(h)
		if val, ok := node.(int); ok {
			tail.Next = &ListNode{Val: val}
			tail = tail.Next
		}
	}

	return dummyHead.Next
}
```

