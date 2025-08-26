---
layout: post
title: 微读Go并发编程：goroutine与channel的高效协作
categories: [语言特性]
tags: [goroutine, channel, 并发编程, CSP模型, 同步]
---

### 1. goroutine：轻量级线程的魅力

goroutine是Go语言并发编程的基石，它比传统线程更轻量，启动成本极低，让并发编程变得简单直观。

```go
// 启动goroutine只需要go关键字
func main() {
    // 顺序执行
    processTask("任务1")
    processTask("任务2")
    
    // 并发执行
    go processTask("任务1") // 在新goroutine中执行
    go processTask("任务2") // 在另一个goroutine中执行
    
    time.Sleep(time.Second) // 等待goroutine完成
}

func processTask(name string) {
    fmt.Printf("处理%s\n", name)
    time.Sleep(500 * time.Millisecond)
    fmt.Printf("%s完成\n", name)
}
```

**核心优势：**
- 内存占用小：初始栈只有2KB，可根据需要增长
- 创建成本低：创建百万个goroutine也不是问题
- 调度高效：Go运行时负责调度，无需操作系统切换上下文

---

### 2. channel：goroutine间的通信桥梁

channel实现了CSP（Communicating Sequential Processes）模型，让goroutine通过通信来共享内存，而不是通过共享内存来通信。

```go
// 创建channel
ch := make(chan string)        // 无缓冲channel
bufferedCh := make(chan int, 5) // 有缓冲channel，容量为5

// 基本使用
func worker(ch chan string) {
    msg := <-ch        // 从channel接收
    fmt.Println(msg)
}

func main() {
    ch := make(chan string)
    
    go worker(ch)      // 启动worker goroutine
    ch <- "Hello World" // 发送数据到channel
}
```

**通信方式：**
- `ch <- value`：发送数据
- `value := <-ch`：接收数据
- `<-ch`：接收但忽略值（常用于同步）

---

### 3. 无缓冲channel：同步通信

无缓冲channel提供同步通信，发送方会阻塞直到接收方准备好接收。

```go
func syncExample() {
    done := make(chan bool) // 无缓冲channel
    
    go func() {
        fmt.Println("goroutine开始工作...")
        time.Sleep(2 * time.Second)
        fmt.Println("goroutine工作完成")
        done <- true // 发送完成信号
    }()
    
    fmt.Println("主程序等待...")
    <-done // 阻塞等待goroutine完成
    fmt.Println("程序退出")
}
```

**适用场景：**
- 任务同步：等待goroutine完成
- 流量控制：控制并发数量
- 事件通知：状态变更通知

---

### 4. 有缓冲channel：异步通信

有缓冲channel允许异步通信，在缓冲区未满时发送不会阻塞。

```go
func bufferExample() {
    jobs := make(chan string, 3) // 缓冲区大小为3
    
    // 发送任务（不会阻塞，因为有缓冲）
    jobs <- "任务1"
    jobs <- "任务2"
    jobs <- "任务3"
    close(jobs) // 关闭channel，表示不再发送
    
    // 处理所有任务
    for job := range jobs {
        fmt.Printf("处理: %s\n", job)
    }
}
```

**缓冲区作用：**
- 解耦生产者和消费者的处理速度
- 减少goroutine阻塞，提高吞吐量
- 实现简单的任务队列

---

### 5. select语句：多路复用

select让goroutine可以同时等待多个channel操作，实现非阻塞通信。

```go
func selectExample() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    // 启动两个数据源
    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "来自ch1的数据"
    }()
    
    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "来自ch2的数据"
    }()
    
    // select等待任一channel准备就绪
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("收到:", msg1)
        case msg2 := <-ch2:
            fmt.Println("收到:", msg2)
        case <-time.After(3 * time.Second):
            fmt.Println("超时")
            return
        }
    }
}
```

**select特性：**
- 随机选择：多个case同时准备时随机选择
- default分支：提供非阻塞操作
- timeout：结合time.After实现超时控制

---

### 6. 实战模式：工作池

工作池是Go并发编程的经典模式，用于控制并发数量并提高任务处理效率。

```go
func workerPool() {
    const numWorkers = 3
    jobs := make(chan int, 5)
    results := make(chan int, 5)
    
    // 启动worker pool
    for w := 1; w <= numWorkers; w++ {
        go func(id int, jobs <-chan int, results chan<- int) {
            for job := range jobs {
                fmt.Printf("Worker %d 处理任务 %d\n", id, job)
                time.Sleep(time.Second)
                results <- job * 2
            }
        }(w, jobs, results)
    }
    
    // 发送任务
    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)
    
    // 收集结果
    for a := 1; a <= 5; a++ {
        result := <-results
        fmt.Printf("结果: %d\n", result)
    }
}
```

**优势：**
- 控制并发数：避免创建过多goroutine
- 资源复用：worker可以处理多个任务
- 任务解耦：任务生产和消费分离

---

### 7. 最佳实践与注意事项

**避免goroutine泄漏：**
```go
func leakPrevention(ctx context.Context) {
    ch := make(chan int)
    
    go func() {
        defer fmt.Println("goroutine退出") // 确保能看到退出日志
        for {
            select {
            case <-ctx.Done(): // 监听取消信号
                return // 正确退出，避免泄漏
            case data := <-ch:
                processData(data)
            }
        }
    }()
}
```

**Channel最佳实践：**
- 发送方关闭channel，接收方检查关闭状态
- 使用context传递取消信号
- 合理设置缓冲区大小
- 避免在接收方关闭channel

goroutine和channel的组合让Go在并发编程领域独树一帜，掌握它们是编写高性能Go程序的关键。