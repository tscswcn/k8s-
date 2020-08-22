队列的用处就是 1个 性能瓶颈的利器。
首先 队列 有3种， 并提供了3种队列， 分别如下：
1） Interface：FIFO队列接口，支持去重
2） DelayingInteface: 延迟队列接口
3） RateLimingInterface: 限速队列接口

![Image text](/k8s_working_queue.jpg)
其中 限速队列中 有个令牌桶算法 挺有意思。

限速算法有： 令牌桶算法，排队指数算法， 计算器算法，还有1个混合模式
 官网有个例子 https://kubernetes.io/zh/docs/tasks/job/fine-parallel-processing-work-queue/
 

// 代码源自client-go/util/workqueue/queue.go
// 这是一个interface类型，说明有其他的各种各样的实现
type Interface interface {
    Add(item interface{})                   // 向队列中添加一个元素，interface{}类型，说明可以添加任何类型的元素
    Len() int                               // 队列长度，就是元素的个数
    Get() (item interface{}, shutdown bool) // 从队列中获取一个元素，双返回值，这个和chan的<-很像，第二个返回值告知队列是否已经关闭了
    Done(item interface{})                  // 告知队列该元素已经处理完了
    ShutDown()                              // 关闭队列
    ShuttingDown() bool                     // 查询队列是否正在关闭
}


// 代码源于client-go/util/workqueue/queue.go
type Type struct {
    queue []t              // 元素数组
    dirty set              // dirty的元素集合
    processing set         // 正在处理的元素集合
    cond *sync.Cond        // 与pthread_cond_t相同，条件同步
    shuttingDown bool      // 关闭标记
    metrics queueMetrics   // 这个metrics和prometheus的metrics概念相同，此处不做过多说明，知道功能就行
}
// 以下的这些类型定义也是够了，对于C/C++程序猿来说不能忍~
type empty struct{}        // 空类型，因为sizeof(struct{})=0
type t interface{}         // 元素类型是泛型
type set map[t]empty       // 用map实现的set，所有的value是空数据就行了
 
 // 代码源自client-go/util/workqueue/delaying_queue.go
// 这部分就是演示队列的核心代码
func (q *delayingType) waitingLoop() {
    defer utilruntime.HandleCrash()
    // 这个变量后面会用到，当没有元素需要延时添加的时候利用这个变量实现长时间等待
    never := make(<-chan time.Time)
    // 构造我们上面提到的有序队列了，并且初始化
    waitingForQueue := &waitForPriorityQueue{}
    heap.Init(waitingForQueue)
    // 这个map是用来避免对象重复添加的，如果重复添加就只更新时间
    waitingEntryByData := map[t]*waitFor{}
    // 开始无限循环
    for {
        // 队列关闭了，就可以返回了
        if q.Interface.ShuttingDown() {
            return
        }
        // 获取当前时间
        now := q.clock.Now()
        // 有序队列中是否有元素，有人肯定会问还没向有序队列里添加呢判断啥啊？后面会有添加哈
        for waitingForQueue.Len() > 0 {
            // Peek函数我们前面注释了，获取第一个元素，注意：不会从队列中取出哦
            entry := waitingForQueue.Peek().(*waitFor)
            // 元素指定添加的时间过了么？如果没有过那就跳出循环
            if entry.readyAt.After(now) {
                break
            }
            // 既然时间已经过了，那就把它从有序队列拿出来放入通用队列中，这里面需要注意几点：
            // 1.heap.Pop()弹出的是第一个元素，waitingForQueue.Pop()弹出的是最后一个元素
            // 2.从有序队列把元素弹出，同时要把元素从上面提到的map删除，因为不用再判断重复添加了
            // 3.此处是唯一一个地方把元素从有序队列移到通用队列，后面主要是等待时间到过程
            entry = heap.Pop(waitingForQueue).(*waitFor)
            q.Add(entry.data)
            delete(waitingEntryByData, entry.data)
        }
 
        // 如果有序队列中没有元素，那就不用等一段时间了，也就是永久等下去
        // 如果有序队列中有元素，那就用第一个元素指定的时间减去当前时间作为等待时间，逻辑挺简单
        // 有序队列是用时间排序的，后面的元素需要等待的时间更长，所以先处理排序靠前面的元素
        nextReadyAt := never
        if waitingForQueue.Len() > 0 {
            entry := waitingForQueue.Peek().(*waitFor)
            nextReadyAt = q.clock.After(entry.readyAt.Sub(now))
        }
        // 进入各种等待
        select {
        // 有退出信号么？
        case <-q.stopCh:
            return
        // 定时器，没过一段时间没有任何数据，那就再执行一次大循环，从理论上讲这个没用，但是这个具备容错能力，避免BUG死等
        case <-q.heartbeat.C():
        // 这个就是有序队列里面需要等待时间信号了，时间到就会有信号
        case <-nextReadyAt:
        // 这里是从chan中获取元素的，AddAfter()放入chan中的元素
        case waitEntry := <-q.waitingForAddCh:
            // 如果时间已经过了就直接放入通用队列，没过就插入到有序队列
            if waitEntry.readyAt.After(q.clock.Now()) {
                insert(waitingForQueue, waitingEntryByData, waitEntry)
            } else {
                q.Add(waitEntry.data)
            }
            // 下面的代码看似有点多，目的就是把chan中的元素一口气全部取干净，注意用了default意味着chan中没有数据就会立刻停止
            drained := false
            for !drained {
                select {
                case waitEntry := <-q.waitingForAddCh:
                    if waitEntry.readyAt.After(q.clock.Now()) {
                        insert(waitingForQueue, waitingEntryByData, waitEntry)
                    } else {
                        q.Add(waitEntry.data)
                    }
                default:
                    drained = true
                }
            }
        }
    }
}
// 下面的代码是把元素插入有序队列的实现
func insert(q *waitForPriorityQueue, knownEntries map[t]*waitFor, entry *waitFor) {
    // 看看元素是不是被添加过？如果添加过看谁的时间靠后就用谁的时间
    existing, exists := knownEntries[entry.data]
    if exists {
        if existing.readyAt.After(entry.readyAt) {
            existing.readyAt = entry.readyAt
            heap.Fix(q, existing.index)
        }
 
        return
    }
    // 把元素放入有序队列中，并记录在map里面,这个map就是上面那个用于判断对象是否重复添加的map
    // 注意，这里面调用的是heap.Push，不是waitForPriorityQueue.Push
    heap.Push(q, entry)
    knownEntries[entry.data] = entry
}
 
 参考链接 
 深入浅出kubernetes之client-go的workqueue
 https://www.dazhuanlan.com/2020/02/03/5e37290eb5ae8/
 https://blog.csdn.net/weixin_42663840/article/details/81482553
 
