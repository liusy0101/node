[toc]





# Go并发编程



互斥锁Mutex、读写锁RWMutex、并发编排WaitGroup、条件变量Cond、Channel等同步原语

适用场景：

- 共享资源：并发读写共享资源，会出现数据竞争（data race）的问题，所以需要Mutex、RWMutex
- 任务编排：需要goroutine按照一定的规律执行，而goroutine之间有相互等待或者依赖的顺序关系，需要使用WaitGroup或者Channel来实现
- 消息传递：信息交流以及不同的goroutine之间的线程安全的数据交流，常常使用Channel来实现



go语言的并发标准库是  sync



### 一、常用关键字

## （1）for、range





## （2）select





## （3）defer

defer会在当前函数返回前执行传入的函数，相当于java中的final代码块

被用于关闭文件描述符、关闭数据库链接以及解锁资源



### 1、作用域

![image-20220118222331655](typora-user-images/image-20220118222331655.png)

![image-20220118222358421](typora-user-images/image-20220118222358421.png)

defer传入的函数不是在退出代码块的作用域时执行的，只会在当前函数和方法返回之前被调用。











## （4）panic、recover

### 1、panic

panic能改变程序的控制流，调用panic后会立刻停止执行当前函数的剩余代码，并在当前Goroutine中递归执行调用方的defer





### 2、recover

recover可以中止panic造成的程序崩溃，是一个只能在defer中发挥作用的函数，在其它作用域中调用不会发挥作用







- `panic` 只会触发当前 Goroutine 的 `defer`；
- `recover` 只有在 `defer` 中调用才会生效；
- `panic` 允许在 `defer` 中嵌套多次调用；

![image-20220118223329042](typora-user-images/image-20220118223329042.png)





## （5）make、new



- make是初始化内置的数据结构，例如切片、哈希表和channel
- new是根据传入的类型分配一片内存空间并返回指向这片内存空间的指针









## 二、Mutex

**不是可重入锁**

Mutex是Locker的实现类

```go
type Locker interface {
	Lock()
	Unlock()
}
```



互斥锁Mutex就提供了两个方法Lock和UnLock

例如：开启十个goroutine去对一个变量进行加1操作



```go
func test1()  {
	var count = 0

	var wg sync.WaitGroup

	wg.Add(10)

	var mutex sync.Mutex

	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100000; j++ {
				mutex.Lock()
				count++
				mutex.Unlock()
			}
		}()
	}

	wg.Wait()

	fmt.Println(count)
}
```



使用Go race detector可以帮忙发现并发问题

使用方法：go run -race xxx.go，可在运行期发现问题

运行： go tool compile -race -S xxx.go， 可在编译器发现问题





Mutex一般会嵌入到其它struct中使用

```go
type Counter2 struct {
	mu sync.Mutex

	count uint64
}
```



也可以采用嵌入字段的方式：

```go
type Counter struct {
	sync.Mutex
	Count uint64
}
```

此时就可以在struct上直接调用Lock/Unlock方法

```go
func test2()  {
	var counter Counter
	var wg sync.WaitGroup

	wg.Add(10)

	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 10000; j++ {
				counter.Lock()
				counter.Count++
				counter.Unlock()
			}
		}()
	}

	wg.Wait()

	fmt.Println(counter.Count)

}
```







也可以通过struct中方法进行封装，用方法进行调用，而不是直接使用字段

```go
type Counter2 struct {
	mu sync.Mutex

	count uint64
}

func (c *Counter2) Incr() {
	c.mu.Lock()
	c.count++
	c.mu.Unlock()
}

func (c *Counter2) GetCount() uint64  {
	c.mu.Lock()

	defer c.mu.Unlock()

	return c.count
}
```





**可重入锁方案**

- 通过hacker的方式获取到goroutine id，记录下获取锁的goroutine id，可以实现Locker接口
- 调用Lock/Unlock方法时，由goroutine提供一个token，用来标识它自己，而不是通过hacker的方式获取到goroutine id，但是，这样就不满足Locker接口



（1）goroutine id

获取goroutine id，方式有两种，分别是简单方式和hacker方式



简单方式，就是通过runtime.Stack方法获取栈帧信息，栈帧里面包含goroutine id，

```go
func GoID() int {
	var buf [64]byte
	n := runtime.Stack(buf[:], false)
	//获取id字符串
	idField := strings.Fields(strings.TrimPrefix(string(buf[:n]),"goroutine "))[0]
	id, err := strconv.Atoi(idField)

	if err != nil {
		panic(fmt.Sprintf("cannot get goroutine id"))
	}

	return id
}
```



（2）hacker方式

首先，我们获取运行时的 g 指针，反解出对应的 g 的结构。每个运行的 goroutine 结构的 g 指针保存在当前 goroutine 的一个叫做 TLS 对象中。

第一步：我们先获取到 TLS 对象；

第二步：再从 TLS 中获取 goroutine 结构的 g 指针；

第三步：再从 g 指针中取出 goroutine id。





实现可重入锁：

![image-20220118204353136](typora-user-images/image-20220118204353136.png)





**token**

![image-20220118204447298](typora-user-images/image-20220118204447298.png)







**使用Mutex实现一个线程安全的队列：**

利用Mutex来保证线程安全

队列可通过Slice来实现

![image-20220118205328190](typora-user-images/image-20220118205328190.png)









## 三、RWMutex

标准库中的RWMutex是一个reader/writer互斥锁。

在某一时刻只能由任意数量的reader持有，或者是只被单个的writer持有

方法：

- Lock/Unlock：写操作时调用的方法，如果锁已经被reader或者writer持有，那么Lock方法会一直阻塞，知道能获取到锁；Unlock则是配对的释放锁的方法
- RLock/RUnlock：读操作时调用的方法。如果锁已经被writer持有的话，RLock方法会一直阻塞，知道能获取到锁，否则就直接返回，而RUnlock是reader释放锁的方法
- RLocker：这个方法的作用是为读操作返回一个Locker接口的对象，它的Lock方法发会调用RWMutex的RLock方法 ，它的Unlock方法会调用RWMutex的RUnlock方法。



RWMutex的零值是未加锁的状态，所以，当使用的时候，无论是声明变量还是嵌入到其它struct中，都不必显式地初始化。



如果遇到可明确区分reader和writer goroutine地场景，且有大量地并发读、少量地并发写，并有强烈的性能需求，就可考虑使用RWMutex替换Mutex



RWMutex是基于Mutex实现的



**读写锁的设计实现分成三类：**

- Read-preferring：读优先可提供很高的并发量，但在竞争激烈的情况下可能会导致写饥饿，原因是如果有大量读，这种设计会导致只有所有的读都释放了锁之后，写才有可能获取到锁
- Write-preferring：写优先，如果已有一个writer在等待请求锁的话，会阻止新的reader获取到锁，优先保障writer。如果有一些reader已经请求了锁的话，新的writer也会等待已有的reader都释放锁之后才能获取。
- 不指定优先级



Go标准库中的RWMutex设计是Write-preferring方案，一个正在阻塞的Lock调用会排除新的reader请求到锁。



