# Golang 内存对齐

和其他语言一样，Golang中为了提高内存访问效率也需要内存对齐。

可以参看`runtime2.go`中`interface`底层实现中itab代码中可以看到：

```go
type itab struct {
	inter *interfacetype 
	_type *_type
	hash  uint32 // copy of _type.hash. Used for type switches.
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}
```

可以注意到使用一个4长度的byte数组，这里的作用主要是为了进行内存对齐。能够方便CPU一次性读取到后后面的fun的数据，`fun`是可变长度的，可能会存多个值，因此将这一块进行对齐落在一个新的区间上。

## 为什么要内存对齐

简单的说，CPU不会一个一个字节的去读取和写入内存。相反CPU读取内存是一块一块的进行读的，块的大小可以是2，4，6，8字节等大小。块的大小被称为 **内存访问粒度 **。32位的系统访问粒度是4字节，64位的系统访问是8字节。

当访问的数据长度为n字节且该数据地址与n字节对齐，那就能够一次性的定位到数据，不需要多次读取、处理对其运算等额外操作。如果访问未对齐的内存，CPU 需要做两次内存访问。

![align-1](https://dist.lyneee.com/blog/2021-09-10-align-1.png)

如上图中，一条数据存在于 1-4 上，这时候是不对齐的。

1. CPU **首次**读取未对齐地址的第一个内存块，读取 0-3 字节。并移除不需要的字节 0
2. CPU **再次**读取未对齐地址的第二个内存块，读取 4-7 字节。并移除不需要的字节 5、6、7 字节
3. 合并 1-4 字节的数据
4. 合并后放入寄存器

如果将这条数据放在0-3上那么，只需要读取一次，就可以获得到全部数据。

可以看到这是一种典型的 **空间换时间** 方法。

## 进行对齐

go官方文档中，对具体的大小进行了描述

```go
// https://golang.org/ref/spec#Size_and_alignment_guarantees
type                                 size in bytes
byte, uint8, int8,bool                1
uint16, int16                         2
uint32, int32, float32                4
uint64, int64, float64, complex64     8
complex128                           16
```

不同的那些对齐保证如下。

```go
type                      alignment guarantee
------                    ------
bool, uint8, int8         1
uint16, int16             2
uint32, int32             4
float32, complex64        4
int64, float64						8
arrays                    depend on element types 取决于元素个数
structs                   depend on field types 取决于结构体的字段类型
other types               size of a native word 取决于机器的字长
```

### 实例分析1

现在有如下代码：

```golang
type T1 struct {
    a [2]int8
    b int64
    c int16
}
type T2 struct {
    a [2]int8
    c int16
    b int64
}
fmt.Printf("arrange fields to reduce size:\n"+
    "T1 align: %d, size: %d\n"+
    "T2 align: %d, size: %d\n",
    unsafe.Alignof(T1{}), unsafe.Sizeof(T1{}),
    unsafe.Alignof(T2{}), unsafe.Sizeof(T2{}))
/*
output:
arrange fields to reduce size:
T1 align: 8, size: 24
T2 align: 8, size: 16
*/
```

T1 和T2 两个结构体字段最大都是 int64，

T1中，size = 24

1. a大小是2字节，填充6分字节进行对齐（第一部分八个字节）

2. b大小是8字节，直接对齐

3. c大小是2字节，填充六个字节使它对齐

T2中，size = 16

1. a和c大小为一共4，填充4字节对齐
2. b本身自己是对齐的

所以，合理的字段排序，能够使字段更加紧密，减少填充



### 实例分析2

下面有一个新的结构体：

```go
type Part1 struct {
    a bool // 1byte
    b int32 // 4byte
    c int8 // 1byte
    d int64 // 8byte
    e byte // 1byte
}

type Part2 struct {
    e byte // 1byte
    c int8 // 1byte
    a bool // 1byte
    b int32 // 4byte
    d int64 // 8byte
}

func main() {
    part1 := Part1{}
    part2 := Part2{}

    fmt.Printf("part1 size: %d, align: %d\n", unsafe.Sizeof(part1), unsafe.Alignof(part1))
    fmt.Printf("part2 size: %d, align: %d\n", unsafe.Sizeof(part2), unsafe.Alignof(part2))
}

/*
output:
part1 size: 32, align: 8
part2 size: 16, align: 8
*/
```

和上面一样进行分析：

- part1: a _ _ _ | b b b b || c _ _ _ | _ _ _ _ || d d d d | d d d d || e _ _ _ | _ _ _ _ *长度为4个字*
- part2: e c a _ | b b b b || d d d d | d d d d  *长度为2个字*



## 总结

1. 进行内存对齐，能够提高CPU对内存访问的效率，仅需要一次就能够读到对应所需要的值。
2. go语言中不同字段所要求的对齐保证的长度是不一样的
3. 合理的结构题字段布局，能够有效的节省内存空间。
