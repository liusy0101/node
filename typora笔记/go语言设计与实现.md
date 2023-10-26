<!-- vscode-markdown-toc -->
* 1. [数据结构](#)
	* 1.1. [数组](#-1)
	* 1.2. [切片](#-1)
* 2. [常用关键字](#-1)
	* 2.1. [for和range](#forrange)
		* 2.1.1. [循环永动机](#-1)
		* 2.1.2. [神奇的指针](#-1)
		* 2.1.3. [遍历清空数组](#-1)
		* 2.1.4. [随机遍历](#-1)
		* 2.1.5. [经典for循环](#for)
		* 2.1.6. [范围循环](#-1)
	* 2.2. [select](#select)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->


##  1. <a name=''></a>数据结构
###  1.1. <a name='-1'></a>数组
数组初始化：
```go
[10]int
[20]interface{}
```

数组初始化后大小边不会再变化，存储元素类型相同，但是大小不同的数组在go中也是不一样的。

```go
func NewArray(elem *Type, bound int64) *Type {
	if bound < 0 {
		Fatalf("NewArray: invalid bound %v", bound)
	}
	t := New(TARRAY)
	t.Extra = &Array{Elem: elem, Bound: bound}
	t.SetNotInHeap(elem.NotInHeap())
	return t
}
```
Elem和Bound是数组的两个属性。

如果数组中元素的个数小于或者等于 4 个，那么所有的变量会直接在栈上初始化，如果数组元素大于 4 个，变量就会在静态存储区初始化然后拷贝到栈上


###  1.2. <a name='-1'></a>切片
切片的长度是动态的，所以声明时只需要指定切片中的元素类型：

```go
[]int
[]interface{}
```

```go
func NewSlice(elem *Type) *Type {
	if t := elem.Cache.slice; t != nil {
		if t.Elem() != elem {
			Fatalf("elem mismatch")
		}
		return t
	}

	t := New(TSLICE)
	t.Extra = Slice{Elem: elem}
	elem.Cache.slice = t
	return t
}
```

可以看到，Slice在初始化的时候只会确定类型

```go
type SliceHeader struct {
	Data uintptr // 指向数据的指针
	Len  int     // 当前切片的长度
	Cap  int     // 当前切片的容量，即是Data数组的大小
}
```

初始化方式：
- 通过下标的方式获得数组或者切片的一部分；
  - 编译器会将 arr[0:3] 或者 slice[0:3] 等语句转换成 OpSliceMake 操作
  - 不会拷贝原数组或者原切片中的数据，它只会创建一个指向原数组的切片结构体，所以修改新切片的数据也会修改原切片
- 使用字面量初始化新的切片；
  - 根据切片中的元素数量对底层数组的大小进行推断并创建一个数组；
  - 将这些字面量元素存储到初始化的数组中；
  - 创建一个同样指向 [3]int 类型的数组指针；
  - 将静态存储区的数组 vstat 赋值给 vauto 指针所在的地址；
  - 通过 [:] 操作获取一个底层使用 vauto 的切片；
- 使用关键字 make 创建切片：
  - 进行参数检查，len是否有传入，cap是否>=len
  - 
    ```go
        func makeslice(et *_type, len, cap int) unsafe.Pointer {
            mem, overflow := math.MulUintptr(et.size, uintptr(cap))
            if overflow || mem > maxAlloc || len < 0 || len > cap {
                mem, overflow := math.MulUintptr(et.size, uintptr(len))
                if overflow || mem > maxAlloc || len < 0 {
                    panicmakeslicelen()
                }
                panicmakeslicecap()
            }

            return mallocgc(mem, et, true)
        }
    ```

```go
arr[0:3] or slice[0:3]
slice := []int{1, 2, 3}
slice := make([]int, 10)
```

扩容：

```go
func growslice(et *_type, old slice, cap int) slice {
	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
```
- 如果期望容量大于当前容量的两倍就会使用期望容量；
- 如果当前切片的长度小于 1024(新版本变成了256) 就会将容量翻倍；
- 如果当前切片的长度大于 1024 就会每次增加 25%（新版本 newcap += (newcap + 3*256) / 4） 的容量，直到新容量大于期望容量；


当数组中元素所占的字节大小为 1、8 或者 2 的倍数时，运行时会对齐内存，对其的是2的幂数，比如5个元素，字节是40，那么就会向上对齐为48，变成cap为6



##  2. <a name='-1'></a>常用关键字
###  2.1. <a name='forrange'></a>for和range

####  2.1.1. <a name='-1'></a>循环永动机
在遍历数组的同时修改数组的元素，能否一直循环打印遍历呢？
```go
func main() {
	arr := []int{1, 2, 3}
	for _, v := range arr {
		arr = append(arr, v)
	}
	fmt.Println(arr)
}

$ go run main.go
1 2 3 1 2 3
```

答案是不行的哈，因为在编译期将原切片或者数组赋值给一个新变量 ha，在赋值的过程中就发生了拷贝，而我们又通过 len 关键字预先获取了切片的长度，所以在循环中追加新的元素也不会改变循环执行的次数

```go
ha := a
hv1 := 0
hn := len(ha)
v1 := hv1
v2 := nil
for ; hv1 < hn; hv1++ {
    tmp := ha[hv1]
    v1, v2 = hv1, tmp
    ...
}
```

####  2.1.2. <a name='-1'></a>神奇的指针
在遍历一个数组时，如果获取 range 返回变量的地址并保存到另一个数组或者哈希时，保存的都是最后一个元素
```go
func main() {
	arr := []int{1, 2, 3}
	newArr := []*int{}
	for _, v := range arr {
		newArr = append(newArr, &v)
	}
	for _, v := range newArr {
		fmt.Println(*v)
	}
}

$ go run main.go
3 3 3
```

因为在遍历期，Go 语言会额外创建一个新的 v2 变量存储切片中的元素，循环中使用的这个变量 v2 会在每一次迭代被重新赋值而覆盖，赋值时也会触发拷贝。

不应该直接获取 range 返回的变量地址 &v2，而应该使用 &a[index] 这种形式。


####  2.1.3. <a name='-1'></a>遍历清空数组
一般使用方法，使用遍历依次将元素置为空值

```go
func main() {
	arr := []int{1, 2, 3}
	for i, _ := range arr {
		arr[i] = 0
	}
}
```

依次遍历切片和哈希看起来是非常耗费性能的，因为数组、切片和哈希占用的内存空间都是连续的，所以最快的方法是直接清空这片内存中的内容。

编译器会直接使用 runtime.memclrNoHeapPointers 清空切片中的数据


####  2.1.4. <a name='-1'></a>随机遍历
在遍历Map的时候，每次遍历的结果顺序都不一样，是因为go在遍历map的时候会引入随机数，随机选择一个bucket开始遍历。


####  2.1.5. <a name='for'></a>经典for循环
![](typora-user-images/2023-10-26-10-23-15.png)


####  2.1.6. <a name='-1'></a>范围循环
编译器会在编译期间将所有 for-range 循环变成经典循环。从编译器的视角来看，就是将 ORANGE 类型的节点转换成 OFOR 节点:

![](typora-user-images/2023-10-26-10-24-31.png)

节点类型的转换过程都发生在中间代码生成阶段，所有的 for-range 循环都会被转换成不包含复杂结构、只包含基本表达式的语句

##### 数组和切片
（1）清空数组
```go
// 原代码
for i := range a {
	a[i] = zero
}

// 优化后
if len(a) != 0 {
	hp = &a[0]
	hn = len(a)*sizeof(elem(a))
	memclrNoHeapPointers(hp, hn)
	i = len(a) - 1
}
```

go会直接使用memclrNoHeapPointers去清空一片连续的内存，然后更新索引

对于所有的 range 循环，Go 语言都会在编译期将原切片或者数组赋值给一个新变量 ha，在赋值的过程中就发生了拷贝，而我们又通过 len 关键字预先获取了切片的长度，所以在循环中追加新的元素也不会改变循环执行的次数

而遇到这种同时遍历索引和元素的 range 循环时，Go 语言会额外创建一个新的 v2 变量存储切片中的元素，循环中使用的这个变量 v2 会在每一次迭代被重新赋值而覆盖，赋值时也会触发拷贝。


##### 哈希表
```go
ha := a
hit := hiter(n.Type)
th := hit.Type
mapiterinit(typename(t), ha, &hit)
for ; hit.key != nil; mapiternext(&hit) {
    key := *hit.key
    val := *hit.val
}
```

```go
func mapiterinit(t *maptype, h *hmap, it *hiter) {
	it.t = t
	it.h = h
	it.B = h.B
	it.buckets = h.buckets

	r := uintptr(fastrand()) // 生成一个随机数，随机选定一个bucket开始遍历
	it.startBucket = r & bucketMask(h.B)
	it.offset = uint8(r >> h.B & (bucketCnt - 1))
	it.bucket = it.startBucket
	mapiternext(it)
}
```

哈希表遍历的顺序，首先会随机选出一个绿色的正常桶开始遍历，随后遍历所有黄色的溢出桶，最后依次按照索引顺序遍历哈希表中其他的桶，直到所有的桶都被遍历完成。

![](typora-user-images/2023-10-26-10-32-50.png)


##### 字符串
在遍历时会获取字符串中索引对应的字节并将字节转换成 rune。我们在遍历字符串时拿到的值都是 rune 类型的变量，for i, r := range s {} 的结构都会被转换成如下所示的形式：

```go
ha := s
for hv1 := 0; hv1 < len(ha); {
    hv1t := hv1
    hv2 := rune(ha[hv1])
    if hv2 < utf8.RuneSelf {
        hv1++
    } else {
        hv2, hv1 = decoderune(ha, hv1)
    }
    v1, v2 = hv1t, hv2
}
```

##### 通道
一个形如 for v := range ch {} 的语句最终会被转换成如下的格式：

```go
ha := a
hv1, hb := <-ha
for ; hb != false; hv1, hb = <-ha {
    v1 := hv1
    hv1 = nil
    ...
}
```
这个操作会调用 runtime.chanrecv2 并阻塞当前的协程，当 runtime.chanrecv2 返回时会根据布尔值 hb 判断当前的值是否存在：

- 如果不存在当前值，意味着当前的管道已经被关闭；
- 如果存在当前值，会为 v1 赋值并清除 hv1 变量中的数据，然后重新陷入阻塞等待新数据；


###  2.2. <a name='select'></a>select、channel
参考： https://docs.google.com/presentation/d/18_9LcMc8u93aITZ6DqeUfRvOcHQYj2gwxhskf0XPX2U/edit#slide=id.g5ea99f63e9_0_11

![](typora-user-images/2023-10-26-11-58-09.png)

#### select
现象：
- select 能在 Channel 上进行非阻塞的收发操作；
- select 在遇到多个 Channel 同时响应时，会随机执行一种情况；

（1）非阻塞的收发

在通常情况下，select 语句会阻塞当前 Goroutine 并等待多个 Channel 中的一个达到可以收发的状态。但是如果 select 控制结构中包含 default 语句，那么这个 select 语句在执行时会遇到以下两种情况：
- 当存在可以收发的 Channel 时，直接处理该 Channel 对应的 case；
- 当不存在可以收发的 Channel 时，执行 default 中的语句；


（2）随机执行

select遇到多个channel同时响应时，会随机执行一种情况。


**数据结构**

```go
type scase struct {
	c    *hchan         // chan
	elem unsafe.Pointer // data element
}
```

**实现原理：**

select 语句在编译期间会被转换成 OSELECT 节点。每个 OSELECT 节点都会持有一组 OCASE 节点，如果 OCASE 的执行条件是空，那就意味着这是一个 default 节点。

![](typora-user-images/2023-10-26-12-05-31.png)

有四种情况：
- 不存在任何case
- 只存在一个case
- 存在两个case，其中一个case和default
- 存在多个case


1. 不存在任何case

```go
func walkselectcases(cases *Nodes) []*Node {
	n := cases.Len()

	if n == 0 {
		return []*Node{mkcall("block", nil, nil)}
	}
	...
}

func block() {
	gopark(nil, nil, waitReasonSelectNoCases, traceEvGoStop, 1)
}
```

空的 select 语句会直接阻塞当前 Goroutine，导致 Goroutine 进入无法被唤醒的永久休眠状态。


2. 只存在一个case

编译器会将 select 改写成 if 条件语句

```go
// 改写前
select {
case v, ok <-ch: // case ch <- v
    ...    
}

// 改写后
if ch == nil {
    block()
}
v, ok := <-ch // case ch <- v
```

当 case 中的 Channel 是空指针时，会直接挂起当前 Goroutine 并陷入永久休眠。


3. 有两个case，其中一个是default

非阻塞的收发操作，会转换成if.else 操作

**发送：**

```go
select {
case ch <- i:
    ...
default:
    ...
}

if selectnbsend(ch, i) {
    ...
} else {
    ...
}


// 向 runtime.chansend 函数传入了非阻塞，所以在不存在接收方或者缓冲区空间不足时，当前 Goroutine 都不会阻塞而是会直接返回
func selectnbsend(c *hchan, elem unsafe.Pointer) (selected bool) {
	return chansend(c, elem, false, getcallerpc())
}
```

**接受：**

```go
// 改写前
select {
case v <- ch: // case v, ok <- ch:
    ......
default:
    ......
}

// 改写后，也是非阻塞的
if selectnbrecv(&v, ch) { // if selectnbrecv2(&v, &ok, ch) {
    ...
} else {
    ...
}


func selectnbrecv(elem unsafe.Pointer, c *hchan) (selected bool) {
	selected, _ = chanrecv(c, elem, false)
	return
}

func selectnbrecv2(elem unsafe.Pointer, received *bool, c *hchan) (selected bool) {
	selected, *received = chanrecv(c, elem, false)
	return
}
```

4. 多个case

- 将所有的 case 转换成包含 Channel 以及类型等信息的 runtime.scase 结构体；
- 调用运行时函数 runtime.selectgo 从多个准备就绪的 Channel 中选择一个可执行的 runtime.scase 结构体；
- 通过 for 循环生成一组 if 语句，在语句中判断自己是不是被选中的 case；

```go
selv := [3]scase{}
order := [6]uint16
for i, cas := range cases {
    c := scase{}
    c.kind = ...
    c.elem = ...
    c.c = ...
}
chosen, revcOK := selectgo(selv, order, 3)
if chosen == 0 {
    ...
    break
}
if chosen == 1 {
    ...
    break
}
if chosen == 2 {
    ...
    break
}
```

最重要的就是用于选择待执行 case 的运行时函数 runtime.selectgo:
- 执行一些必要的初始化操作并确定 case 的处理顺序；
- 在循环中根据 case 的类型做出不同的处理；


**初始化：**

runtime.selectgo 函数首先会进行执行必要的初始化操作并决定处理 case 的两个顺序 — 轮询顺序 pollOrder 和加锁顺序 lockOrder：

轮询顺序 pollOrder 和加锁顺序 lockOrder 分别是通过以下的方式确认的：

- 轮询顺序：通过 runtime.fastrandn 函数引入随机性；
- 加锁顺序：按照 Channel 的地址排序后确定加锁顺序；

**循环：**

- 查找是否已经存在准备就绪的 Channel，即可以执行收发操作；
- 将当前 Goroutine 加入 Channel 对应的收发队列上并等待其他 Goroutine 的唤醒；
- 当前 Goroutine 被唤醒之后找到满足条件的 Channel 并进行处理；

runtime.selectgo 函数会根据不同情况通过 goto 语句跳转到函数内部的不同标签执行相应的逻辑，其中包括：

- bufrecv：可以从缓冲区读取数据；
- bufsend：可以向缓冲区写入数据；
- recv：可以从休眠的发送方获取数据；
- send：可以向休眠的接收方发送数据；
- rclose：可以从关闭的 Channel 读取 EOF；
- sclose：向关闭的 Channel 发送数据；
- retc：结束调用并返回；

循环执行的第一个阶段，查找已经准备就绪的 Channel。循环会遍历所有的 case 并找到需要被唤起的 runtime.sudog 结构，在这个阶段，我们会根据 case 的四种类型分别处理：

- 当 case 不包含 Channel 时；
  - 这种 case 会被跳过；
- 当 case 会从 Channel 中接收数据时；
  - 如果当前 Channel 的 sendq 上有等待的 Goroutine，就会跳到 recv 标签并从缓冲区读取数据后将等待 Goroutine 中的数据放入到缓冲区中相同的位置；
  - 如果当前 Channel 的缓冲区不为空，就会跳到 bufrecv 标签处从缓冲区获取数据；
  - 如果当前 Channel 已经被关闭，就会跳到 rclose 做一些清除的收尾工作；
- 当 case 会向 Channel 发送数据时；
  - 如果当前 Channel 已经被关，闭就会直接跳到 sclose 标签，触发 panic 尝试中止程序；
  - 如果当前 Channel 的 recvq 上有等待的 Goroutine，就会跳到 send 标签向 Channel 发送数据；
  - 如果当前 Channel 的缓冲区存在空闲位置，就会将待发送的数据存入缓冲区；
- 当 select 语句中包含 default 时；
  - 表示前面的所有 case 都没有被执行，这里会解锁所有 Channel 并返回，意味着当前 select 结构中的收发都是非阻塞的；


![](typora-user-images/2023-10-26-12-29-38.png)

第一阶段的主要职责是查找所有 case 中是否有可以立刻被处理的 Channel。无论是在等待的 Goroutine 上还是缓冲区中，只要存在数据满足条件就会立刻处理，如果不能立刻找到活跃的 Channel 就会进入循环的下一阶段，按照需要将当前 Goroutine 加入到 Channel 的 sendq 或者 recvq 队列中：

```go
func selectgo(cas0 *scase, order0 *uint16, ncases int) (int, bool) {
	...
	gp = getg()
	nextp = &gp.waiting
	for _, casei := range lockorder {
		casi = int(casei)
		cas = &scases[casi]
		c = cas.c
		sg := acquireSudog()
		sg.g = gp
		sg.c = c

		if casi < nsends {
			c.sendq.enqueue(sg)
		} else {
			c.recvq.enqueue(sg)
		}
	}

	gopark(selparkcommit, nil, waitReasonSelect, traceEvGoBlockSelect, 1)
	...
}
```


除了将当前 Goroutine 对应的 runtime.sudog 结构体加入队列之外，这些结构体都会被串成链表附着在 Goroutine 上。在入队之后会调用 runtime.gopark 挂起当前 Goroutine 等待调度器的唤醒。

![](typora-user-images/2023-10-26-12-31-26.png)

等到 select 中的一些 Channel 准备就绪之后，当前 Goroutine 就会被调度器唤醒。这时会继续执行 runtime.selectgo 函数的第三部分，从 runtime.sudog 中读取数据：

```go
func selectgo(cas0 *scase, order0 *uint16, ncases int) (int, bool) {
	...
	sg = (*sudog)(gp.param)
	gp.param = nil

	casi = -1
	cas = nil
	sglist = gp.waiting
	for _, casei := range lockorder {
		k = &scases[casei]
		if sg == sglist {
			casi = int(casei)
			cas = k
		} else {
			c = k.c
			if int(casei) < nsends {
				c.sendq.dequeueSudoG(sglist)
			} else {
				c.recvq.dequeueSudoG(sglist)
			}
		}
		sgnext = sglist.waitlink
		sglist.waitlink = nil
		releaseSudog(sglist)
		sglist = sgnext
	}

	c = cas.c
	goto retc
	...
}
```

第三次遍历全部 case 时，我们会先获取当前 Goroutine 接收到的参数 sudog 结构，我们会依次对比所有 case 对应的 sudog 结构找到被唤醒的 case，获取该 case 对应的索引并返回。

由于当前的 select 结构找到了一个 case 执行，那么剩下 case 中没有被用到的 sudog 就会被忽略并且释放掉。为了不影响 Channel 的正常使用，我们还是需要将这些废弃的 sudog 从 Channel 中出队。

当我们在循环中发现缓冲区中有元素或者缓冲区未满时就会通过 goto 关键字跳转到 bufrecv 和 bufsend 两个代码段，这两段代码的执行过程都很简单，它们只是向 Channel 中发送数据或者从缓冲区中获取新数据：

```go
bufrecv:
	recvOK = true
	qp = chanbuf(c, c.recvx)
	if cas.elem != nil {
		typedmemmove(c.elemtype, cas.elem, qp)
	}
	typedmemclr(c.elemtype, qp)
	c.recvx++
	if c.recvx == c.dataqsiz {
		c.recvx = 0
	}
	c.qcount--
	selunlock(scases, lockorder)
	goto retc

bufsend:
	typedmemmove(c.elemtype, chanbuf(c, c.sendx), cas.elem)
	c.sendx++
	if c.sendx == c.dataqsiz {
		c.sendx = 0
	}
	c.qcount++
	selunlock(scases, lockorder)
	goto retc
Go
```

这里在缓冲区进行的操作和直接调用 runtime.chansend 和 runtime.chanrecv 差不多，上述两个过程在执行结束之后都会直接跳到 retc 字段。

两个直接收发 Channel 的情况会调用运行时函数 runtime.send 和 runtime.recv，这两个函数会与处于休眠状态的 Goroutine 打交道：

```go
recv:
	recv(c, sg, cas.elem, func() { selunlock(scases, lockorder) }, 2)
	recvOK = true
	goto retc

send:
	send(c, sg, cas.elem, func() { selunlock(scases, lockorder) }, 2)
	goto retc
Go
```

不过如果向关闭的 Channel 发送数据或者从关闭的 Channel 中接收数据，情况就稍微有一点复杂了：

从一个关闭 Channel 中接收数据会直接清除 Channel 中的相关内容；
向一个关闭的 Channel 发送数据就会直接 panic 造成程序崩溃：
```go
rclose:
	selunlock(scases, lockorder)
	recvOK = false
	if cas.elem != nil {
		typedmemclr(c.elemtype, cas.elem)
	}
	goto retc

sclose:
	selunlock(scases, lockorder)
	panic(plainError("send on closed channel"))
Go
```
总体来看，select 语句中的 Channel 收发操作和直接操作 Channel 没有太多出入，只是由于 select 多出了 default 关键字所以会支持非阻塞的收发。


**总结：**

首先在编译期间，Go 语言会对 select 语句进行优化，它会根据 select 中 case 的不同选择不同的优化路径：

- 空的 select 语句会被转换成调用 runtime.block 直接挂起当前 Goroutine；
- 如果 select 语句中只包含一个 case，编译器会将其转换成 if ch == nil { block }; n; 表达式；
  - 首先判断操作的 Channel 是不是空的；
  - 然后执行 case 结构中的内容；
- 如果 select 语句中只包含两个 case 并且其中一个是 default，那么会使用 runtime.selectnbrecv 和 runtime.selectnbsend 非阻塞地执行收发操作；
- 在默认情况下会通过 runtime.selectgo 获取执行 case 的索引，并通过多个 if 语句执行对应 case 中的代码；

在编译器已经对 select 语句进行优化之后，Go 语言会在运行时执行编译期间展开的 runtime.selectgo 函数，该函数会按照以下的流程执行：

- 随机生成一个遍历的轮询顺序 pollOrder 并根据 Channel 地址生成锁定顺序 lockOrder；
- 根据 pollOrder 遍历所有的 case 查看是否有可以立刻处理的 Channel；
  - 如果存在，直接获取 case 对应的索引并返回；
  - 如果不存在，创建 runtime.sudog 结构体，将当前 Goroutine 加入到所有相关 Channel 的收发队列，并调用 runtime.gopark 挂起当前 Goroutine 等待调度器的唤醒；
- 当调度器唤醒当前 Goroutine 时，会再次按照 lockOrder 遍历所有的 case，从中查找需要被处理的 runtime.sudog 对应的索引；

select 关键字是 Go 语言特有的控制结构，它的实现原理比较复杂，需要编译器和运行时函数的通力合作。