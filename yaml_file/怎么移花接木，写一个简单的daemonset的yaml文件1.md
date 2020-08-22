大家都知道yaml文件比较复杂，一不小心就容易出错，今天我教大家一个简单的办法写个daemonset的yaml文件
1，首先按照要求创建1个deployment的yaml文件，这个非常简单
如，要求创建1个以redis 的image为基础，创建名为redis-ds的 daemonset。
       首先我们用 

kubectl create deployment redis-ds --image=redis  --dry-run -o yaml > redis-ds.yaml

得到如下文件



`apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: redis-ds
  name: redis-ds
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-ds
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: redis-ds
    spec:
      containers:`

​          `- image: redis
​         name: redis
​         resources: {}
status: {}`





去掉不需要的，并简单修改
  replicas: 1 主要删掉这行

  strategy: {} 这行 也删掉


    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
    
      labels:
        app: redis-ds
      name: redis-ds
    spec:
    
      selector:
        matchLabels:
          app: redis-ds
    
      template:
        metadata: 
     labels:
        app: redis-ds
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
修改后 去 kubectl apply -f redis-ds.yaml 就可以DaemonSet 

kubectl apply -f redis-ds.yaml
daemonset.apps/redis-ds created

```
[root@node-×××××× ~]# kubectl get daemonset
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
redis-ds          4         4         0       4          0          <none>          20s
wavefront-1596521047-collector          7         7         5       7          5          <none>          15d
........」
- - - - - - - - - - - - - - -」
—————————
```


