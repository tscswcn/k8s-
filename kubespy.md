利用 kubespy 方便 诊断 容器网络 问题
去经常有些有些容易没有bash 或者 sh， 使得 我们没有办法attach 到容器进行诊断
这个时候我们可以利用
kubespy

 curl -so kubectl-spy https://raw.githubusercontent.com/huazhihao/kubespy/master/kubespy
 sudo install kubectl-spy /usr/local/bin/

然后 可以
kubectl spy POD [-c CONTAINER] [--spy-image SPY_IMAGE]

可以利用 这样1个 image netshoot，有很多现成的 工具
