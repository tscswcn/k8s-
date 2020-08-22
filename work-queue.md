队列的用处就是 1个 性能瓶颈的利器。
首先 队列 有3种， 并提供了3种队列， 分别如下：
1） Interface：FIFO队列接口，支持去重
2） DelayingInteface: 延迟队列接口
3） RateLimingInterface: 限速队列接口

其中 限速队列中 有个令牌桶算法 挺有意思。
![Image text](/k8s_working_queue.jpg)
