# 排序算法

本文代码实现使用golang

## 0概述

### 0.1 分类

十种算法可以分为两类：

- 比较排序：通过比较元素的相对次序，由于时间复杂度不能突破$O(nlogn)$，也被叫做非线性时间比较排序
- 非比较排序：不通过比较来觉元素间的相对次序

![排序分类](https://dist.lyneee.com/blog/2021-09-03-排序分类.png)

### 0.2 算法复杂度

![时间复杂度](https://dist.lyneee.com/blog/2021-09-03-sort-time.png)

### 0.3 相关概念

- **稳定**：a本来在b前面，排序完成后，a和b的位置没有发生变化
- **不稳定**：a本来在b前面，a=b，排序之后，a可能会出现在b的后面
- **时间复杂度**
- **空间复杂度**

## 1冒泡排序

稳定排序，时间复杂度为 $O(N^{2})$，最好的情况下已经是排好序的，为$O(N)$


### 1.1 算法描述

- 比较相邻的元素。如果第一个比第二个大，就交换它们两个；
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
- 针对所有的元素重复以上的步骤，除了最后一个；
- 重复步骤1~3，直到排序完成。

### 1.2 演示

![冒泡排序](https://dist.lyneee.com/blog/2021-09-03-bubble-sort.gif)

### 1.3 代码实现

```go
// bubbleSort 
func bubbleSort(slice []int) []int {
	swap := true
	for swap {
		swap = false
		for i := 0; i < len(slice)-1; i++ {
			if slice[i] > slice[i+1] {
				slice[i+1], slice[i] = slice[i], slice[i+1]
				swap = true
			}
		}
	}
	return slice
}
```

## 2选择排序

选择排序(Selection-sort)是一种简单直观的排序算法。它的工作原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。 

### 2.1 算法描述

n个记录的直接选择排序可经过n-1趟直接选择排序得到有序结果。具体算法描述如下：

- 初始状态：无序区为R[1..n]，有序区为空；
- 第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R[1..i-1]和R(i..n）。该趟排序从当前无序区中-选出关键字最小的记录 R[k]，将它与无序区的第1个记录R交换，使R[1..i]和R[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
- n-1趟结束，数组有序化了。

### 2.2 演示

![选择排序](https://dist.lyneee.com/blog/2021-09-03-selection-sort.gif)

### 2.3 代码实现

```go
// selectionSort
func selectionSort(slice []int) []int {
	for i := 0; i < len(slice); i++ {
		min := i
		for j := i + 1; j < len(slice); j++ {
			if slice[j] < slice[min] {
				min = j
			}
		}
		slice[i], slice[min] = slice[min], slice[i]
	}
	return slice
}
```

### 2.4 算法分析

无论什么数据进去都是$O(N^{2})$的时间复杂度。

## 3插入排序

插入排序（Insertion-Sort）的算法描述是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

#### 3.1 算法描述

一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：

- 从第一个元素开始，该元素可以认为已经被排序；
- 取出下一个元素，在已经排序的元素序列中从后向前扫描；
- 如果该元素（已排序）大于新元素，将该元素移到下一位置；
- 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置；
- 将新元素插入到该位置后；
- 重复步骤2~5。

#### 3.2 演示

![插入排序](https://dist.lyneee.com/blog/2021-09-03-Insertion-Sort.gif)

#### 3.3 代码实现

```go
// insertionSort
func insertionSort(slice []int) []int {
	for currentIndex := 1; currentIndex < len(slice); currentIndex++ {
		iterator := currentIndex
		tmp := slice[currentIndex]
		for ; iterator > 0 && slice[iterator-1] > tmp; iterator-- {
			slice[iterator] = slice[iterator-1]
		}
		slice[iterator] = tmp
	}
	return slice
}
```

#### 3.4 算法分析

原地排序，时间复杂度取决于数组的状态，如果是已经有序的数组则时间复杂度为 $O(N)$，一般情况都是

$O(N^{2})$时间复杂度。

## 4希尔排序

1959年Shell发明，第一个突破$O(N^{2})$的排序算法，是简单插入排序的改进版，（分组插入排序）。它与插入排序的不同之处在于，它会优先比较距离较远的元素。希尔排序又叫**缩小增量排序**。

### 4.1 算法描述

- 通过使用插入排序对间隔d的序列进行排序。
- 通过不断较少d，最后d=1，使得整个数组是有序的。

### 4.2 演示

![shell-sort](https://dist.lyneee.com/blog/2021-09-03-shell-sort.gif)

### 4.3 代码实现

```go
// shellSort
func shellSort(slice []int) []int {
	for d := int(len(slice) / 2); d > 0; d /= 2 {
		for i := d; i < len(slice); i++ {
			for j := i; j >= d && slice[j-d] > slice[j]; j -= d {
				slice[j], slice[j-d] = slice[j-d], slice[j]
				fmt.Println(slice)
			}
		}
	}
	return slice
}
// INPUT:[8, 3, 4, 5, 6, 7, 8, 9]
// [6 3 4 5 8 7 8 9]  d == 4
// [4 3 6 5 8 7 8 9]  d == 2
// [3 4 6 5 8 7 8 9]  d == 1 
// [3 4 5 6 8 7 8 9]  d == 1
// [3 4 5 6 7 8 8 9]  d == 1
```

### 4.4 算法分析

时间复杂度$O(dM*N)$，M表示已经排序了的长度，d表示间隔， N表示若干倍乘递增序列的长度，为不稳定排序。

## 5归并排序

归并排序是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。

可参考一到经典题目，[剑指offer51](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

### 5.1 算法描述

- 把长度为n的输入序列分成两个长度为n/2的子序列；
- 对这两个子序列分别采用归并排序；
- 将两个排序好的子序列合并成一个最终的排序序列。

### 5.2 演示

![merge-sort](https://dist.lyneee.com/blog/2021-09-03-mergesort.gif)

### 5.3 代码实现

```go
// mergeSort
func mergeSort(nums []int, start, end int) []int {

	if start >= end {
		return
	}
	mid := start + (end-start)/2
	mergeSort(nums, start, mid)
	mergeSort(nums, mid+1, end)

	i, j := start, mid+1
	tmp := []int{}
	for i <= mid && j <= end {
		if nums[i] > nums[j] {
			tmp = append(tmp, nums[j])
			j++
		} else {
			tmp = append(tmp, nums[i])
			i++
		}
	}

	for ; i <= mid; i++ {
		tmp = append(tmp, nums[i])
	}

	for ; j <= end; j++ {
		tmp = append(tmp, nums[j])
	}
	for i := start; i <= end; i++ {
		nums[i] = tmp[i-start]
	}
}

```

### 5.4 算法分析

时间复杂度为$O(N \log N)$

## 6快速排序

快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。

### 6.1 算法描述

快速排序使用分治法来把一个串（list）分为两个子串（sub-lists）。具体算法描述如下：

- 从数列中挑出一个元素，称为 “基准”（pivot）；
- 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
- 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

### 6.2 演示

![quick-sort](https://dist.lyneee.com/blog/2021-09-03-quick-sort.gif)

### 6.3 代码实现

```go

func quickSort(slice []int, start, end int) {
	if start < end {
		i, j := start, end
		pivot := slice[start+(end-start)/2]
		for i <= j {
			for slice[i] < pivot {
				i++
			}
			for slice[j] > pivot {
				j--
			}
			if i <= j {
				slice[i], slice[j] = slice[j], slice[i]
				i++
				j--
			}
		}
		if start < j {
			quickSort(slice, start, j)
		}
		if end > i {
			quickSort(slice, i, end)
		}
	}
}
```

### 6.4 算法分析

时间复杂度为$n \log n$，原地排序,最坏时间复杂度为$O(n^{2})$

## 7堆排序

堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。

### 7.1 算法描述

- 将初始待排序关键字序列(R1,R2….Rn)构建成大顶堆，此堆为初始的无序区；
- 将堆顶元素R[1]与最后一个元素R[n]交换，此时得到新的无序区(R1,R2,……Rn-1)和新的有序区(Rn),且满足R[1,2…n-1]<=R[n]；
- 由于交换后新的堆顶R[1]可能违反堆的性质，因此需要对当前无序区(R1,R2,……Rn-1)调整为新堆，然后再次将R[1]与无序区最后一个元素交换，得到新的无序区(R1,R2….Rn-2)和新的有序区(Rn-1,Rn)。不断重复此过程直到有序区的元素个数为n-1，则整个排序过程完成。

### 7.2 演示

![heap-sort](https://dist.lyneee.com/blog/2021-09-03-heap-sort.gif)

### 
