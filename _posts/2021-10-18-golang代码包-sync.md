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

### 3 互斥锁

互斥锁的使用注意事项：

#### 1.不要重复锁定互斥锁

虽然没有强制规定，但最好不要重复锁定，对一个已经被锁定的互斥锁进行锁定，是会立即阻塞当前的goroutine的，直到该互斥锁的unlock语句执行。

而这很容易出现一个问题：当前程序的主goroutine和我们启用的goroutine全都陷入阻塞状态，而一旦所有的都阻塞，Go就会抛出panic：`fatal error: all goroutines are asleep - deadlock!`.

需注意，由Go直接抛出的panic是无法恢复的，recover函数是无效的，程序必然崩溃。

而一个goroutine自己把自己锁住是违背常理的，这个是很难进行处理和维护的。

#### 2.不要忘记解锁互斥锁，必要时使用defer语句；

必须解锁互斥锁的根本原因也是避免互斥锁被重复锁定，从而出现问题。

为了避免这个问题，可以用defer语句来进行处理，在锁定之后，用`defer mu.unlock`，在执行完毕后解锁。

#### 3.不要对尚未锁定或已解锁的的互斥锁解锁；

需注意的是，如果试图解锁未被锁的互斥锁，会立刻出现panic，而且这个panic也无法恢复，因此，我们必须保证，`对于每一个锁定操作，都要有且只有一个对应的解锁操作。`

#### 4.不要在多个函数之间直接传递互斥锁。

这一条本身并不会产生问题，因为在传值时，会传该互斥锁的副本，但是这样会引发歧义，因此尽量不要这样做。

### 4 读写锁

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

