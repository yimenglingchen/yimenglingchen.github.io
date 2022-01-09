---

layout: post

title:  golang代码包-waitgroup

tag: go语言

---

# golang代码包-waitgroup

---

## 1 前导

sync.WaitGroup类型（以下简称WaitGroup类型）是开箱即用的，也是并发安全的。

WaitGroup类型拥有三个指针方法：Add、Done和Wait。你可以想象该类型中有一个计数器，它的默认值是0。我们可以通过调用该类型值的Add方法来增加，或者减少这个计数器的值。

WaitGroup其实相当于把通道进行了分组处理，一组通道一并进行处理，通过Add增加，Done减少，Wait则是阻塞，直到这个变量归零。

这个值不能为零，如果为零的话会引发panic。因此，要避免对并发对这个值进行操作，即需要在同一个goroutine中操作。

我们最好用“先统一Add，再并发Done，最后Wait”这种标准方式，来使用WaitGroup值。

sync.Once类型则只会执行一次，即这个函数只执行一次操作。