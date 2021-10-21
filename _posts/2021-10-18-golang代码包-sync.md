---

layout: post

title:  golang代码包-sync

tag: go语言

---

# golang代码包-sync

---

## 1 前导

### 1 竞态条件

竞态条件：一旦数据被多个线程共享，那么就很有可能产生争用和冲突的情况。这往往会破坏共享数据的一致性。

简单地说就是，多个线程共同操作一个数据，数据未及时更新而导致数据错乱。

共享数据的一致性代表着某种约定，即：多个线程对共享数据的操作总是可以达到它们各自预期的效果。

同步：同步的用途有两个，一个是避免多个线程在同一时刻操作同一个数据块，另一个是协调多个线程，以避免它们在同一时刻执行同一个代码块。

可以粗略理解为，需要一个操作的时候，线程需要申请权限，在使用后归还，下次再使用时需要重新申请。

### 2 临界区

多个并发运行的线程对这个共享资源的访问是完全串行的。只要一个代码片段需要实现对共享资源的串行化访问，就可以被视为一个临界区（critical section）。

而为了施加这个保护，需要使用实现了某种同步机制的工具，也称同步工具。

Go中比较常用的工具为互斥量（mutex），sync中的sync.Mutex就是其对应的类型，该类型的值就被称为互斥量。

一个互斥锁可以被用来保护一个临界区或者一组相关临界区。我们可以通过它来保证，在同一时刻只有一个 goroutine 处于该临界区之内。

为了兑现这个保证，每当有 goroutine 想进入临界区时，都需要先对它进行锁定，并且，每个 goroutine 离开临界区时，都要及时地对它进行解锁。

代码示例：

```go
mu.Lock()
_, err := writer.Write([]byte(data))
if err != nil {
 log.Printf("error: %s [%d]", err, id)
}
mu.Unlock()
```

## 2 互斥锁

互斥锁的使用注意事项：

### 1.不要重复锁定互斥锁

虽然没有强制规定，但最好不要重复锁定，对一个已经被锁定的互斥锁进行锁定，是会立即阻塞当前的goroutine的，直到该互斥锁的unlock语句执行。

而这很容易出现一个问题：当前程序的主goroutine和我们启用的goroutine全都陷入阻塞状态，而一旦所有的都阻塞，Go就会抛出panic：`fatal error: all goroutines are asleep - deadlock!`.

需注意，由Go直接抛出的panic是无法恢复的，recover函数是无效的，程序必然崩溃。

而一个goroutine自己把自己锁住是违背常理的，这个是很难进行处理和维护的。

### 2.不要忘记解锁互斥锁，必要时使用defer语句；

必须解锁互斥锁的根本原因也是避免互斥锁被重复锁定，从而出现问题。

为了避免这个问题，可以用defer语句来进行处理，在锁定之后，用`defer mu.unlock`，在执行完毕后解锁。

### 3.不要对尚未锁定或已解锁的的互斥锁解锁；

需注意的是，如果试图解锁未被锁的互斥锁，会立刻出现panic，而且这个panic也无法恢复，因此，我们必须保证，`对于每一个锁定操作，都要有且只有一个对应的解锁操作。`

### 4.不要在多个函数之间直接传递互斥锁。

这一条本身并不会产生问题，因为在传值时，会传该互斥锁的副本，但是这样会引发歧义，因此尽量不要这样做。


代码示例：

```go
package main

import (
	"bytes"
	"errors"
	"fmt"
	"io"
	"log"
	"sync"
	"time"
)

// singleHandler 代表单次处理函数的类型。
type singleHandler func() (data string, n int, err error)

// handlerConfig 代表处理流程配置的类型。
type handlerConfig struct {
	handler   singleHandler // 单次处理函数。
	goNum     int           // 需要启用的goroutine的数量。
	number    int           // 单个goroutine中的处理次数。
	interval  time.Duration // 单个goroutine中的处理间隔时间。
	counter   int           // 数据量计数器，以字节为单位。
	counterMu sync.Mutex    // 数据量计数器专用的互斥锁。

}

// count 会增加计数器的值，并会返回增加后的计数。
func (hc *handlerConfig) count(increment int) int {
	hc.counterMu.Lock()
	defer hc.counterMu.Unlock()
	hc.counter += increment
	return hc.counter
}

func main() {
	// mu 代表以下流程要使用的互斥锁。
	// 在下面的函数中直接使用即可，不要传递。
	var mu sync.Mutex

	// genWriter 代表的是用于生成写入函数的函数。
	genWriter := func(writer io.Writer) singleHandler {
		return func() (data string, n int, err error) {
			// 准备数据。
			data = fmt.Sprintf("%s\t",
				time.Now().Format(time.StampNano))
			// 写入数据。
			mu.Lock()
			defer mu.Unlock()
			n, err = writer.Write([]byte(data))
			return
		}
	}

	// genReader 代表的是用于生成读取函数的函数。
	genReader := func(reader io.Reader) singleHandler {
		return func() (data string, n int, err error) {
			buffer, ok := reader.(*bytes.Buffer)
			if !ok {
				err = errors.New("unsupported reader")
				return
			}
			// 读取数据。
			mu.Lock()
			defer mu.Unlock()
			data, err = buffer.ReadString('\t')
			n = len(data)
			return
		}
	}

	// buffer 代表缓冲区。
	var buffer bytes.Buffer

	// 数据写入配置。
	writingConfig := handlerConfig{
		handler:  genWriter(&buffer),
		goNum:    5,
		number:   4,
		interval: time.Millisecond * 100,
	}
	// 数据读取配置。
	readingConfig := handlerConfig{
		handler:  genReader(&buffer),
		goNum:    10,
		number:   2,
		interval: time.Millisecond * 100,
	}

	// sign 代表信号的通道。
	sign := make(chan struct{}, writingConfig.goNum+readingConfig.goNum)

	// 启用多个goroutine对缓冲区进行多次数据写入。
	for i := 1; i <= writingConfig.goNum; i++ {
		go func(i int) {
			defer func() {
				sign <- struct{}{}
			}()
			for j := 1; j <= writingConfig.number; j++ {
				time.Sleep(writingConfig.interval)
				data, n, err := writingConfig.handler()
				if err != nil {
					log.Printf("writer [%d-%d]: error: %s",
						i, j, err)
					continue
				}
				total := writingConfig.count(n)
				log.Printf("writer [%d-%d]: %s (total: %d)",
					i, j, data, total)
			}
		}(i)
	}

	// 启用多个goroutine对缓冲区进行多次数据读取。
	for i := 1; i <= readingConfig.goNum; i++ {
		go func(i int) {
			defer func() {
				sign <- struct{}{}
			}()
			for j := 1; j <= readingConfig.number; j++ {
				time.Sleep(readingConfig.interval)
				var data string
				var n int
				var err error
				for {
					data, n, err = readingConfig.handler()
					if err == nil || err != io.EOF {
						break
					}
					// 如果读比写快（读时会发生EOF错误），那就等一会儿再读。
					time.Sleep(readingConfig.interval)
				}
				if err != nil {
					log.Printf("reader [%d-%d]: error: %s",
						i, j, err)
					continue
				}
				total := readingConfig.count(n)
				log.Printf("reader [%d-%d]: %s (total: %d)",
					i, j, data, total)
			}
		}(i)
	}

	// signNumber 代表需要接收的信号的数量。
	signNumber := writingConfig.goNum + readingConfig.goNum
	// 等待上面启用的所有goroutine的运行全部结束。
	for j := 0; j < signNumber; j++ {
		<-sign
	}
}
```

## 3 读写锁

读写锁是读/写互斥锁的简称，在Go语言中，读写锁由`sync.RWMutex`类型的值代表。

一个读写锁中实际上包含了两个锁，即：读锁和写锁。sync.RWMutex类型中的Lock方法和Unlock方法分别用于对写锁进行锁定和解锁，而它的RLock方法和RUnlock方法则分别用于对读锁进行锁定和解锁。

另外，对于同一个读写锁来说有如下规则：

在写锁已被锁定的情况下再试图锁定写锁，会阻塞当前的 goroutine。

在写锁已被锁定的情况下试图锁定读锁，也会阻塞当前的 goroutine。

在读锁已被锁定的情况下试图锁定写锁，同样会阻塞当前的 goroutine。

在读锁已被锁定的情况下再试图锁定读锁，并不会阻塞当前的 goroutine。

换一个角度来说，对于某个受到读写锁保护的共享资源，多个写操作不能同时进行，写操作和读操作也不能同时进行，但多个读操作却可以同时进行。

除此之外，读写锁对写操作之间的互斥，其实是通过它内含的一个互斥锁实现的。因此，也可以说，Go 语言的读写锁是互斥锁的一种扩展。

最后，需要强调的是，与互斥锁类似，解锁“读写锁中未被锁定的写锁”，会立即引发 panic，对于其中的读锁也是如此，并且同样是不可恢复的。

## 4 条件变量

### 1 条件变量与互斥锁

条件变量是基于互斥锁的，它必须有互斥锁的支撑才能发挥作用。

条件变量并不是被用来保护临界区和共享资源的，它是用于协调想要访问共享资源的那些线程的。当共享资源的状态发生变化时，它可以被用来通知被互斥锁阻塞的线程。

条件变量在共享资源的状态产生变化的时候，起到了通知作用。

条件变量提供的方法有三个：等待通知（wait）、单发通知（signal）和广播通知（broadcast）。

条件变量的创建示例：

```go
var mailbox uint8
var lock sync.RWMutex
sendCond := sync.NewCond(&lock)
recvCond := sync.NewCond(lock.RLocker())
```

定义一个变量，然后定义互斥锁，根据状态来确定是否发送和收取。当互斥锁的状态发生改变的时候，修改条件变量的值。sendCond和recvCond都是条件变量，用于记录值。

在执行的时候，通过等待和通知，就可以实现两个goroutine之间的通讯了。

代码示例：

```go
lock.Lock()
for mailbox == 1 {
 sendCond.Wait()
}
mailbox = 1
lock.Unlock()
recvCond.Signal()
```

另一个goroutine也类似，只是对mailbox的状态有所区分。

完整的代码示例：

```go
package main

import (
	"log"
	"sync"
	"time"
)

func main() {
	// mailbox 代表信箱。
	// 0代表信箱是空的，1代表信箱是满的。
	var mailbox uint8
	// lock 代表信箱上的锁。
	var lock sync.RWMutex
	// sendCond 代表专用于发信的条件变量。
	sendCond := sync.NewCond(&lock)
	// recvCond 代表专用于收信的条件变量。
	recvCond := sync.NewCond(lock.RLocker())

	// sign 用于传递演示完成的信号。
	sign := make(chan struct{}, 3)
	max := 5
	go func(max int) { // 用于发信。
		defer func() {
			sign <- struct{}{}
		}()
		for i := 1; i <= max; i++ {
			time.Sleep(time.Millisecond * 500)
			lock.Lock()
			for mailbox == 1 {
				sendCond.Wait()
			}
			log.Printf("sender [%d]: the mailbox is empty.", i)
			mailbox = 1
			log.Printf("sender [%d]: the letter has been sent.", i)
			lock.Unlock()
			recvCond.Signal()
		}
	}(max)
	go func(max int) { // 用于收信。
		defer func() {
			sign <- struct{}{}
		}()
		for j := 1; j <= max; j++ {
			time.Sleep(time.Millisecond * 500)
			lock.RLock()
			for mailbox == 0 {
				recvCond.Wait()
			}
			log.Printf("receiver [%d]: the mailbox is full.", j)
			mailbox = 0
			log.Printf("receiver [%d]: the letter has been received.", j)
			lock.RUnlock()
			sendCond.Signal()
		}
	}(max)

	<-sign
	<-sign
}

```

### 2 Wait的用法

条件变量的Wait方法主要做了四件事：

1把调用它的 goroutine（也就是当前的 goroutine）加入到当前条件变量的通知队列中。

2.解锁当前的条件变量基于的那个互斥锁。

3.让当前的 goroutine 处于等待状态，等到通知到来时再决定是否唤醒它。此时，这个 goroutine 就会阻塞在调用这个Wait方法的那行代码上。

4.如果通知到来并且决定唤醒这个 goroutine，那么就在唤醒它之后重新锁定当前条件变量基于的互斥锁。自此之后，当前的 goroutine 就会继续执行后面的代码了。

因此，在调用Wait方法之前，必须将互斥锁加锁，否则解锁一个没有被锁住的互斥锁，会引发无法panic，出现系统异常。而for就是为了重复校验判断，进行多次检查。

比如下面这种情况：

`1.有多个 goroutine 在等待共享资源的同一种状态。比如，它们都在等mailbox变量的值不为0的时候再把它的值变为0，这就相当于有多个人在等着我向信箱里放置情报。虽然等待的 goroutine 有多个，但每次成功的 goroutine 却只可能有一个。别忘了，条件变量的Wait方法会在当前的 goroutine 醒来后先重新锁定那个互斥锁。在成功的 goroutine 最终解锁互斥锁之后，其他的 goroutine 会先后进入临界区，但它们会发现共享资源的状态依然不是它们想要的。这个时候，for循环就很有必要了。`

`2.共享资源可能有的状态不是两个，而是更多。比如，mailbox变量的可能值不只有0和1，还有2、3、4。这种情况下，由于状态在每次改变后的结果只可能有一个，所以，在设计合理的前提下，单一的结果一定不可能满足所有 goroutine 的条件。那些未被满足的 goroutine 显然还需要继续等待和检查。`

`3.有一种可能，共享资源的状态只有两个，并且每种状态都只有一个 goroutine 在关注，就像我们在主问题当中实现的那个例子那样。不过，即使是这样，使用for语句仍然是有必要的。原因是，在一些多 CPU 核心的计算机系统中，即使没有收到条件变量的通知，调用其Wait方法的 goroutine 也是有可能被唤醒的。这是由计算机硬件层面决定的，即使是操作系统（比如 Linux）本身提供的条件变量也会如此。`

因此，我们最好采用for来重复检查。

### 3 Signal方法和Broadcast方法

条件变量的Signal方法和Broadcast方法并不需要在互斥锁的保护下执行。恰恰相反，我们最好在解锁条件变量基于的那个互斥锁之后，再去调用它的这两个方法。这更有利于程序的运行效率。

条件变量的Signal方法和Broadcast方法都是被用来发送通知的，不同的是，前者的通知只会唤醒一个因此而等待的 goroutine，而后者的通知却会唤醒所有为此等待的 goroutine。

最后，请注意，条件变量的通知具有即时性。也就是说，如果发送通知的时候没有 goroutine 为此等待，那么该通知就会被直接丢弃。在这之后才开始等待的 goroutine 只可能被后面的通知唤醒。