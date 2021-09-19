# 堆-go实现

堆(heap)是一种重要的数据结构，是实现优先队列的首先数据结构。堆有很多种实现方式，二项式堆、斐波拉契堆，以及二叉堆。

## 1.堆heap

实现一个堆有两个要求

- 堆是一个 **完全二叉树** ；
- 堆中每一个节点的值都必须大于等于（或者小于等于）某一个子树中的所有值

根据堆的性质，作为根的值是最大或者最小分为大根堆和小根堆。也就是字节点

![heap](https://dist.lyneee.com/blog/2021-09-17-heap.png)

我们将这种结构，映射到数组中可以有，

```go
// 大根堆
[50, 45, 40, 20, 25, 35, 30, 10, 15]
[ 0,  1,  2,  3,  4,  5,  6,  7,  8]
// 小根堆
[10, 20, 15, 25, 50, 30, 40, 35, 45]
[ 0,  1,  2,  3,  4,  5,  6,  7,  8]
```

所以可以得到一个定义：

大根堆:  `arr[i] >= arr[2i+1] && arr[i]  >= arr[2i+2]`

小根堆:  `arr[i] <= arr[2i+1] && arr[i] <= arr[2i+2]`

### 1.1插入元素

一个大根堆，大致的过程可以描述为：

1. 将一个元素插入到队尾，
2. 如果子节点的值，比父节点大则进行交换

这是一个从堆底向上冒泡的过程：

```go
type Heap struct {
	data   []int // 数据
	length int   // 堆的长度
	count  int   // 堆已经存入的数量
}

func HeapConstructor(capacity int) *Heap {
	data := make([]int, 0, capacity)
	return &Heap{
		data:   data,
		length: capacity,
		count:  0,
	}
}

func (h *Heap) Insert(i int) {
	h.data = append(h.data, i)
	h.count++
	h.Up()
}

func (h *Heap) Up() {
	i := h.count - 1
	for i/2 >= 0 && h.data[i] > h.data[i/2] {
		// 如果当前节点（子节点）比父节点大，那么则交换两个节点
		h.Swap(i, i/2)
		i /= 2
	}
}
```

### 1.3删除堆顶元素

对于一个大根堆，删除堆顶元素的的过程：

1. 移除堆顶元素
2. 为了保持完全二叉树的性质，将最后一个元素和堆顶元素进行交换
3. 移除堆尾元素
4. 从顶向下进行堆化

```go
// PopMax: 弹出最大元素
func (h *Heap) PopMax() int {
	max := h.data[0]
	h.Swap(0, h.count-1)
	h.count--
	h.data = h.data[:h.count]
	h.Down(0, h.count)
	return max
}

// Down: 从根元素向子元素进行堆化
func (h *Heap) Down(i0, count int) {
	i := i0

	for {
		j1 := i*2 + 1
		if j1 >= count || j1 < 0 {
			break
		}
		j := j1
		// 本次实现的是大根堆，所以找到，两个叶子节点中，较大的那个节点
		if j2 := j1 + 1; j2 < count && h.data[j1] < h.data[j2] {
			j = j2
		}
		// 如果此时的根节点，比字节点都要大，那么则不用进行交换
		if h.data[j] < h.data[i] {
			break
		}
		// 进行交换
		h.Swap(i, j)
		// 对交换后的节点继续，进行比较和交换，直到不需要交换为止
		i = j
	}
}
```

### 1.3 构建堆

如何构建一个堆呢，这是一个大根堆。

```go
// Init: 进行堆化
func (h *Heap) Init(nums ...int) {
	h.data = append(h.data, nums...)
	h.count += len(nums)
	// 因为大于count/2 - 1 的都是叶子结点，所以不用进行堆化
	for i := h.count/2 - 1; i >= 0; i-- {
		h.Down(i, h.count)
	}
}


// Down: 从根元素向子元素进行堆化
func (h *Heap) Down(i0, count int) {
	i := i0

	for {
		j1 := i*2 + 1
		if j1 >= count || j1 < 0 {
			break
		}
		j := j1
		// 本次实现的是大根堆，所以找到，两个叶子节点中，较大的那个节点
		if j2 := j1 + 1; j2 < count && h.data[j1] < h.data[j2] {
			j = j2
		}
		// 如果此时的根节点，比字节点都要大，那么则不用进行交换
		if h.data[j] < h.data[i] {
			break
		}
		// 进行交换
		h.Swap(i, j)
		// 对交换后的节点继续，进行比较和交换，直到不需要交换为止
		i = j
	}
}
```



## 2. 使用golang 提供的 heap进行实现

主要是实现提供的五个接口，调用heap.Init来进行构建堆

## 3.堆排序和快排

### 3.1堆排序数据访问没有快排友好

- 快排的数据是顺序访问的
- 堆排序的数据是跳跃访问的

这一点导致在堆排序的时候，CPU在进行读入数据的时候，如果跳跃距离过大，需要进行多次读取。另外，跳跃读不利于CPU cache。

### 3.2堆排序的交换次数会比快速排序的数据交换次数更多

## 4.应用

### 4.1topK问题

比如，数据中前K个最大元素，维护一个堆，和堆顶元素进行比较，如果大或者小则弹出堆顶，压入新的元素

### 4.2求数据流中的中位数

-> https://leetcode-cn.com/problems/find-median-from-data-stream/

### 4.3优先队列的实现





## 5最后贴上代码

<https://gist.github.com/aschenmaker/e81419898e4f12cc475c83eab68e2a21>

<script src="https://gist.github.com/aschenmaker/e81419898e4f12cc475c83eab68e2a21.js"></script>

