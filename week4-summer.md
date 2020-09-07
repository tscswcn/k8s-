Kubernetes-源码研习社-(client-go)-总结回顾
第5章一共有7小节
其中内容涉及到kubeconfig 配置管理、informer机制、各种客户端对象、然后是Workqueue队列机制，最后一部分是client-gen 代码生产器

but 我对比 k8s 的 github
https://github.com/kubernetes/client-go

我看到有如下

What's included
The kubernetes package contains the clientset to access Kubernetes API.
The discovery package is used to discover APIs supported by a Kubernetes API server.
The dynamic package contains a dynamic client that can perform generic operations on arbitrary Kubernetes API objects.
The plugin/pkg/client/auth packages contain optional authentication plugins for obtaining credentials from external sources.
The transport package is used to set up auth and start a connection.
The tools/cache package is useful for writing controllers.

and Ihave　Seen　k8sclient fabric8客户端

what is 　k8sclient fabric8 回头 补充了解下
