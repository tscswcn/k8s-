第一次读源码，比较吃力。
1，首先基础的是 RESTClient，RESTclient 通过 kubeconfig  跟K8s API Server交互。
在RESTclient 之上 是3种DiscoveryClient,DynamicClient,ClientSet.
Kubeconfig配置管理就是 咱们常用的kubeconfig文件，其中 主要 内容 是 集群信息，users，contexts。
RESTClient 发送请求的过程是用 go 语言标准库 net/http 进行的封装，由 do-> request 函数实现。 
ClientSet使用的时候需要知道Resource所在的Group和对应的Version。
通过kubectl命令，通过ClientSet列出 运行中素有的Pod 对象。

DynamecClient包括 自定义 CRD 资源， 提供的操作 包括 Create,update,Delete,Get,List,Watch,Patch等方法
对于DiscoveryClient，值得一提的是它可以 把相关的资源信息存贮在本地，默认位置在 ~/.kube/cache 和 ~/.kube/http-cache，默认10分钟跟 APIServer同步一次。

2， Informer机制 
这个是k8s的重要机制。 Informer机制中有Reflector,DeltaFIFO队列，Index.
Indexer是带索引功能的本地存储。
WorkQueue分为FIFO队列，延迟队列、限速队列。
限速队列中 有个令牌桶算法 比较新鲜，（我没有听说过）

3，书中还 说道5种 代码生成器。
先写这么多吧

  
看各位大佬的作业，我有很多需要学习，加油
