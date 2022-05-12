# context

context 源于 google，于 1.7 版本加入标准库，按照官方文档的说法，它是一个请求的全局上下文，携带了截止时间、手动取消等信号，并包含一个并发安全的 map 用于携带数据。
到底是个啥，也没看明白。下面列一些应用场景，以后慢慢悟吧

## 传值

```go
package main

import (
    "context"
    "fmt"
)

func func1(ctx context.Context) {
    ctx = context.WithValue(ctx, "k1", 12)
    func2(ctx)
    ctx = context.WithValue(ctx, "k1", 10)
    func2(ctx)
}
func func2(ctx context.Context) {
    fmt.Println(ctx.Value("k1"))
}

func main() {
    ctx := context.Background()
    func1(ctx)
}
```

执行结果：

```go
12
10
```

这个例子就是，感觉可以做一个全局变量的东西，或者是一个临时数据保存。这里数据输出不是非常严格，func2 里面并没有限定输出的类型。可以换成：

```go
func func2(ctx context.Context) {
    fmt.Println(ctx.Value("k1").(string))
    fmt.Println(ctx.Value("k1").(int))
}
```

这样，输出就限制死类型了。似乎没必要。

## 取消耗时操作，及时释放资源

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "sync"
    "time"
)

func func0(ctx context.Context, wg *sync.WaitGroup) error {
    defer wg.Done()
    respC := make(chan int)
    // 处理逻辑
    go func() {
        time.Sleep(time.Second * 5)
        respC <- 10
    }()
    // 取消机制
    select {
    case <-ctx.Done():
        fmt.Println("cancel")
        return errors.New("cancel")
    case r := <-respC:
        fmt.Println(r)
        return nil
    }
}

func main() {
    wg := new(sync.WaitGroup)
    ctx, cancel := context.WithCancel(context.Background())
    wg.Add(1)
    go func0(ctx, wg)
    time.Sleep(time.Second * 2)
    // 触发取消
    cancel()
    // 等待goroutine退出
    wg.Wait()
}

```

执行结果：

```shell
cancel
```

这个例子就是 func0 执行超时(这里的时间是 2 秒)，func0 执行实际需要 5 秒。然后，2 秒过了，func0 就执行退出(就是 select 死循环的逻辑)。
使用场景，就是个控制执行时间，有的程序需要自己写超时操作，就用这个。http 有专门的 timeout 参数，别的模块没有，就得这么搞。http 应该也是一样的方法。
这里面通道的使用和 context 没有关系，重点是`cancel()`这个函数的执行。`cancel()`函数执行，`ctx.Done()`这个通道被写入一个值，触发 case，然后执行对应逻辑。
就这个例子来讲，超时控制不用 context 似乎也行，我自己声明一个通道，往里面塞东西应该也能实现。**经过验证，确实能实现。**这特么就尴尬了，有啥用啊？
而且吧，不用通道，用 sync.WaitGroup 也能实现，就是代码多一点
