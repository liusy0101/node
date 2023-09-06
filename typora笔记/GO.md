# GO

## 数组与切片
slice的底层数据是数组，slice是对数组的封装， 描述一个数组的片段。
数组是定长的，长度定义好之后，不能再更改。在 Go 中，数组是不常见的，因为其长度是类型的一部分，限制了它的表达能力，比如 [3]int 和 [4]int 就是不同的类型。

而切片则非常灵活，它可以动态地扩容。切片的类型和长度无关。

数组就是一片连续的内存， slice 实际上是一个结构体，包含三个字段：长度、容量、底层数组。

```go
// runtime/slice.go
type slice struct {
	array unsafe.Pointer // 元素指针
	len   int // 长度 
	cap   int // 容量
}
```
![](typora-user-images/2023-09-06-16-09-41.png)

注意，底层数组是可以被多个 slice 同时指向的，因此对一个 slice 的元素进行操作是有可能影响到其他 slice 的。

【引申1】 [3]int 和 [4]int 是同一个类型吗？

不是。因为数组的长度是类型的一部分，这是与 slice 不同的一点。

【引申2】 下面的代码输出是什么？
```go
package main

import "fmt"

func main() {
	slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    // s1是slice第二到第五，cap为8， s1=[2,3,4]
	s1 := slice[2:5]
    // s2是从s1上搞下来的，容量到索引7，所以cap为5， s2=[4,5,6,7]
	s2 := s1[2:6:7]

    // 追加100，由于地层的数组是同一个，所以slice索引8都改为了100
	s2 = append(s2, 100)
    // 扩容，会重新分配内存，所以改的只是s2的引用，不会改到slice下的数据
	s2 = append(s2, 200)

	s1[2] = 20

	fmt.Println(s1)
	fmt.Println(s2)
	fmt.Println(slice)
}
```
结果：
[2 3 20]
[4 5 6 7 100 200]
[0 1 2 3 20 5 6 7 100 9]


### 扩容机制
使用 append 可以向 slice 追加元素，实际上是往底层数组添加元素。但是底层数组的长度是固定的，如果索引 len-1 所指向的元素已经是底层数组的最后一个元素，就没法再添加了。

这时，slice 会迁移到新的内存位置，新底层数组的长度也会增加，这样就可以放置新增的元素。同时，为了应对未来可能再次发生的 append 操作，新的底层数组的长度，也就是新 slice 的容量是留了一定的 buffer 的。否则，每次添加元素的时候，都会发生迁移，成本太高。

新 slice 预留的 buffer 大小是有一定规律的。在golang1.18版本更新之前网上大多数的文章都是这样描述slice的扩容策略的：

｜当原 slice 容量小于 1024 的时候，新 slice 容量变成原来的 2 倍；原 slice 容量超过 1024，新 slice 容量变成原来的1.25倍。

在1.18版本更新之后，slice的扩容策略变为了：

｜当原slice容量(oldcap)小于256的时候，新slice(newcap)容量为原来的2倍；原slice容量超过256，新slice容量newcap = oldcap+(oldcap+3*256)/4

确定容量之后，会进行容量对其

golang版本1.9.5

```go
// go 1.9.5 src/runtime/slice.go:82
func growslice(et *_type, old slice, cap int) slice {
    // ……
    newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			for newcap < cap {
				newcap += newcap / 4
			}
		}
	}
	// ……
	
	capmem = roundupsize(uintptr(newcap) * ptrSize)
	newcap = int(capmem / ptrSize)
}
```

golang版本1.18
```go
// go 1.18 src/runtime/slice.go:178
func growslice(et *_type, old slice, cap int) slice {
    // ……
    newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		const threshold = 256
		if old.cap < threshold {
			newcap = doublecap
		} else {
			for 0 < newcap && newcap < cap {
                // Transition from growing 2x for small slices
				// to growing 1.25x for large slices. This formula
				// gives a smooth-ish transition between the two.
				newcap += (newcap + 3*threshold) / 4
			}
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
	// ……
    
	capmem = roundupsize(uintptr(newcap) * ptrSize)
	newcap = int(capmem / ptrSize)
}
```

后半部分还对 newcap 作了一个内存对齐，这个和内存分配策略相关。进行内存对齐之后，新 slice 的容量是要 大于等于 按照前半部分生成的newcap。

之后，向 Go 内存管理器申请内存，将老 slice 中的数据复制过去，并且将 append 的元素添加到新的底层数组中。

最后，向 growslice 函数调用者返回一个新的 slice，这个 slice 的长度并没有变化，而容量却增大了。

```go
package main

import "fmt"

func main() {
	s := []int{1,2}
    // cap为2，增加三个元素之后，新的cap为5，此时double都无法解决，所以就用新的cap，然后再进行容量对其，所以后续最新的cap是6
	s = append(s,4,5,6)
	fmt.Printf("len=%d, cap=%d",len(s),cap(s))
}
```
运行结果是：
len=5, cap=6


### 切片作为函数参数
当 slice 作为函数参数时，就是一个普通的结构体。其实很好理解：若直接传 slice，在调用者看来，实参 slice 并不会被函数中的操作改变；若传的是 slice 的指针，在调用者看来，是会被改变原 slice 的。

值得注意的是，不管传的是 slice 还是 slice 指针，如果改变了 slice 底层数组的数据，会反应到实参 slice 的底层数据。为什么能改变底层数组的数据？很好理解：底层数据在 slice 结构体里是一个指针，尽管 slice 结构体自身不会被改变，也就是说底层数据地址不会被改变。 但是通过指向底层数据的指针，可以改变切片的底层数据，没有问题。

通过 slice 的 array 字段就可以拿到数组的地址。在代码里，是直接通过类似 s[i]=10 这种操作改变 slice 底层数组元素值。

另外，值得注意的是，Go 语言的函数参数传递，只有值传递，没有引用传递。

```go
package main

func main() {
	s := []int{1, 1, 1}
	f(s)
	fmt.Println(s)
}

func f(s []int) {
	// i只是一个副本，不能改变s中元素的值
	/*for _, i := range s {
		i++
	}
	*/

	for i := range s {
		s[i] += 1
	}
}
```

运行一下，程序输出：
[2 2 2]

果真改变了原始 slice 的底层数据。这里传递的是一个 slice 的副本，在 f 函数中，s 只是 main 函数中 s 的一个拷贝。在f 函数内部，对 s 的作用并不会改变外层 main 函数的 s。

要想真的改变外层 slice，只有将返回的新的 slice 赋值到原始 slice，或者向函数传递一个指向 slice 的指针

```go
package main

import "fmt"

func myAppend(s []int) []int {
	// 这里 s 虽然改变了，但并不会影响外层函数的 s
    fmt.Printf("%p", s) // 打印地层数组的地址
	s = append(s, 100)
    fmt.Printf("%p", s) // 打印地层数组的地址
	return s
}

func myAppendPtr(s *[]int) {
	// 会改变外层 s 本身
	*s = append(*s, 100)
	return
}

func main() {
    // s的cap是3，当myappend增加新元素时候，需要扩容
	s := []int{1, 1, 1}
    fmt.Printf("%p", s) // 打印地层数组的地址
	newS := myAppend(s)

	fmt.Println(s)
	fmt.Println(newS)

	s = newS

	myAppendPtr(&s)
	fmt.Println(s)
}
```
运行结果：
[1 1 1]
[1 1 1 100]
[1 1 1 100 100]

myAppend 函数里，虽然改变了 s，但它只是一个值传递，并不会影响外层的 s，因此第一行打印出来的结果仍然是 [1 1 1]。

而 newS 是一个新的 slice，它是基于 s 得到的。因此它打印的是追加了一个 100 之后的结果： [1 1 1 100]。

最后，将 newS 赋值给了 s，s 这时才真正变成了一个新的slice。之后，再给 myAppendPtr 函数传入一个 s 指针，这回它真的被改变了：[1 1 1 100 100]。