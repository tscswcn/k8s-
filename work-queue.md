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
 
 
 参考链接 
 深入浅出kubernetes之client-go的workqueue
 https://www.dazhuanlan.com/2020/02/03/5e37290eb5ae8/
 https://blog.csdn.net/weixin_42663840/article/details/81482553
 
