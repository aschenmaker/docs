# 	内存逃逸

!!! note "引入，一个简单思考"
		先抛出一个问题，在Go语言里传指针一定比值拷贝快吗？oh，这可能成为了Go的基本八股文之一了。

在C语言里，动态分配内存(new/malloc)需要我们手动释放资源。这样做的好处是，需要申请多少内存空间可以很好的掌握怎么分配。但是这有个缺点，如果忘记释放内存，则会导致内存泄漏。

在Go语言里，Go语言自带的垃圾回收(GC)能够帮助程序员忘却资源分配与回收这件事，把复杂的内存管理交给Go编译器。

## 堆和栈

- 堆（Heap）：一般来讲是人为手动进行管理，手动申请、分配、释放。一般所涉及的内存大小并不定，一般会存放较大的对象。另外其 **分配相对慢** ，涉及到的指令动作也相对多。堆分配内存首先需要去找到一块大小合适的内存块。之后要通过垃圾回收才能释放。

- 栈（Stack）：由编译器进行管理，自动申请、分配、释放。一般不会太大，我们常见的函数参数（不同平台允许存放的数量不同）， **局部变量** 等等都会存放在栈上。栈分配内存只需要两个CPU指令：“PUSH”和“RELEASE”分配和释放。


内存从栈escape到堆的过程，被叫做逃逸分析。

Go语言通过编译器，对堆栈进行分析，然后通过GC进行管理。

一个简单的例子，说明内存是分配在堆上还是栈上。

```go
// AllocateToStack: 作为局部变量放到了栈上
func AllocateToStack() {
	temp := make([]int, 0, 20)
	...
  
}
```

上面的例子，`Allocate`函数内部申请的临时变量，即使你是用make申请到的内存，如果发现在退出函数后没有用了，那么就把丢到栈上，毕竟栈上的内存分配比堆上快很多。

```go
// AllocateToHeap: 存在引用分配到了堆
func AllocateToHeap() []int{
	a := make([]int, 0, 20)
	return a
}
```

这上面这段代码，申请的代码和上面的一模一样，但是 **申请后作为返回值返回了** ，编译器会认为在退出函数之后还有其他地方在引用，当函数返回之后并不会将其内存归还。那么就申请到 **堆 **里。

过多的分配到栈上存在的问题：

- 垃圾回收（GC）的压力不断增大
- 申请、分配、回收内存的系统开销增大（相对于栈）
- 动态分配产生一定量的内存碎片



## 逃逸分析
这是我们需要进行逃逸分析

!!! note "逃逸分析"
		**逃逸分析**

    逃逸分析是一种确定指针动态范围的方法。简单来说就是分析在程序的哪些地方可以访问到该指针。
    
    简单来说，编译器会根据变量是否被外部引用来决定是否逃逸：
    
    ```c
    1. 如果函数外部没有引用，则优先放到栈中；如果该对象过大了，无法存放到栈上，则也有可能会被分配到堆上。
    2. 如果函数外部存在引用，则必定放到堆中；
    ```
    
    对此你可以理解为，逃逸分析是编译器用于决定变量分配到堆上还是栈上的一种行为。


    注意：go 在 **编译阶段确立逃逸** ，并不是在运行时。




### 内存逃逸

回答之前的问题，首先通过一个benchmark来说明问题。

这里我们有一个返回指针的函数，与一个返回拷贝值的函数。

```go
// memescape.go
func NewPersonWithPointer(age int, name, hobby string) *Person {
	person := new(Person)

	person.Age = age
	person.Name = name
	person.Hobby = hobby

	return person
}

func NewPersonWithValue(age int, name, hobby string) Person {
	return Person{
		Age:   age,
		Name:  name,
		Hobby: hobby,
	}
}

```

```go
// memescape_benchmark.go
func BenchmarkNewPersonWithPointer(b *testing.B) {
	var p *Person
	for i := 0; i < b.N; i++ {
		p = NewPersonWithPointer(1, "1", "1")
	}
	_ = fmt.Sprintf("%s", p.Name)
}

func BenchmarkNewPersonWithValue(b *testing.B) {
	var p Person
	for i := 0; i < b.N; i++ {
		NewPersonWithValue(1, "1", "1")
	}
	_ = fmt.Sprintf("%s", p.Name)
}

```

结果：

```shell
============
Benchmark 
============
goos: darwin
goarch: amd64
pkg: github.com/InsideOfTheIndustry/iotgtt/memescape
cpu: Intel(R) Core(TM) i5-7267U CPU @ 3.10GHz
BenchmarkNewPersonWithPointer-4   	  25721979	          46.33 ns/op	      48 B/op	       1 allocs/op
BenchmarkNewPersonWithValue-4   	1000000000	         0.3313 ns/op	       0 B/op	       0 allocs/op
PASS
ok      github.com/InsideOfTheIndustry/iotgtt/memescape 2.414s
```

可以看到通过指针传递，`1 allocs/op`，为什么呢。通过gcflags可以看到，`new(Person) escapes to heap`

```shell
go build -gcflags=-m 

# github.com/InsideOfTheIndustry/iotgtt/memescape/build
./main.go:18:15: new(Person) escapes to heap
```

这一段的代码可以：https://github.com/InsideOfTheIndustry/iotgtt/tree/master/memescape 得到。

使用`go build -gcflags '-m -l' main.go`可以看到调用了运行时函数，`runtime.newobject(SB)`

```apl
	0x0018 00024 (main.go:18)	FUNCDATA	$5, "".NewPersonWithPointer.arginfo1(SB)
	...
	0x002c 00044 (main.go:23)	MOVQ	DI, "".hobby+56(SP)
	0x0031 00049 (main.go:19)	LEAQ	type."".Person(SB), AX
	0x0038 00056 (main.go:19)	PCDATA	$1, $0
	0x0038 00056 (main.go:19)	CALL	runtime.newobject(SB)
	0x003d 00061 (main.go:21)	MOVQ	"".age+32(SP), CX
	0x0042 00066 (main.go:21)	MOVQ	CX, (AX)
	...
```

这是因为 `NewPersonWithPointer()` 返回的是指针对象，引用被返回到了方法之外了。因此编译器会把该对象分配到堆上，而不是栈上。否则方法结束之后，局部变量就被回收了，岂不是翻车。所以最终分配到堆上是理所当然的。

#### 额外的又一个问题

同样返回了`NewPersonWithValue()`为什么没有产生内存逃逸呢？

```go
func main() {
  // func NewPersonWithValue(age int, name, hobby string) Person 
	NewPersonWithValue(1, "12312", "123123")
}

func NewPersonWithValue(age int, name, hobby string) Person {
	return Person{
		Age:   age,
		Name:  name,
		Hobby: hobby,
	}
}
```

可以看到参数 name 和 hobby 产生了泄漏。

```shell
$ go build -gcflags '-m -l' main.go 
# command-line-arguments
./main.go:28:34: leaking param: name to result ~r3 level=0
./main.go:28:40: leaking param: hobby to result ~r3 level=0
```

结合代码可得知name 和 hobby传给 `NewPersonWithValue` 方法后，没有做任何引用之类的涉及变量的动作，直接就把这个变量返回出去了。因此这个变量实际上并没有逃逸，它的作用域还在 `main()` 之中，所以分配在栈上。

#### 如何让它产生内存逃逸

加以引用即可

```go
func NewPersonWithValue(age int, name, hobby string) *Person {

	return &Person{
		Age:   age,
		Name:  name,
		Hobby: hobby,
	}
}
```

```shell
$ go build -gcflags '-m -l' main.go 
# command-line-arguments
./main.go:28:34: leaking param: name
./main.go:28:40: leaking param: hobby
./main.go:30:9: &Person{...} escapes to heap
```



### 栈空间不足逃逸

当栈空间不足以存放当前对象时或无法判断当前切片长度时会将对象分配到堆中。

```go
package main

func main() {

	s := make([]int, 10000, 10000)

    for index, _ := range s {
        s[index] = index
    }
}
```

进行编译

```shell
$ go build -gcflags '-m -l' main.go            
# command-line-arguments
./main.go:5:11: make([]int, 10000, 10000) escapes to heap
```

可以看到`make([]int, 10000, 10000) escapes to heap`

### 动态类型逃逸

如果你使用interface类型作为函数参数，比如

```go
func Printf(format string, a ...interface{}) (n int, err error)
```

编译期间，无法确定具体的参数类型，interface对应的值会在运行时中通过反射得到，那么也会产生内存逃逸。如果使用了1.17的范型来作为参数的话，可能……

```go
package main

import "fmt"

func main() {
	fmt.Println("hello world")
}
```

可以看到 `"hello world" escapes to heap`

```shell
$ go build -gcflags '-m -l' main.go 
# command-line-arguments
./main.go:6:13: ... argument does not escape
./main.go:6:14: "hello world" escapes to heap
```



## **总结**

1、堆上动态分配内存比栈上静态分配内存，开销大很多。

2、变量分配在栈上需要能在编译期确定它的作用域，否则会分配到堆上。

3、Go编译器会在编译期对考察变量的作用域，并作一系列检查，如果它的作用域在运行期间对编译器一直是可知的，那么就会分配到栈上。简单来说，编译器会根据变量是否被外部引用来决定是否逃逸。

4、对于Go程序员来说，编译器的这些逃逸分析规则不需要掌握，我们只需通过go build -gcflags '-m’命令来观察变量逃逸情况就行了。

5、不要盲目使用变量的指针作为函数参数，虽然它会减少复制操作。但其实当参数为变量自身的时候，复制是在栈上完成的操作，开销远比变量逃逸后动态地在堆上分配内存少的多。

6、逃逸分析在编译阶段完成的。



### 参考文档

- [跟着煎鱼学go：我要在栈上。不，你应该在堆上](https://eddycjy.gitbook.io/golang/di-1-ke-za-tan/stack-heap)
- [Golang 内存分配之逃逸分析](https://zhuanlan.zhihu.com/p/113643434)
- [内存逃逸分析代码https://github.com/InsideOfTheIndustry/iotgtt/blob/master/memescape](https://github.com/InsideOfTheIndustry/iotgtt/blob/master/memescape)

