我们创建 的时候 都是 
resourtype 
- name
  image: 
但是 创建后

reourcetype
- image 
   name:

包括 volumeMounts: 

kubectl get pods redis-ds-sqchc -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.3.181/32
  creationTimestamp: "2020-08-20T02:58:35Z"
  generateName: redis-ds-
  labels:
    app: redis-ds
    controller-revision-hash: 96c8ff9c6
    pod-template-generation: "1"
  name: redis-ds-sqchc
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: DaemonSet
    name: redis-ds
    uid: 3f2b3b7b-fb48-45b9-84b4-b56a6188afdd
  resourceVersion: "435512223"
  selfLink: /api/v1/namespaces/default/pods/redis-ds-sqchc
  uid: ad8cae74-a289-4f22-9266-2fb24e870ab8
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchFields:
          - key: metadata.name
            operator: In
            values:
            - node-10-120-127-238
  containers:
  - image: redis
    imagePullPolicy: Always
    name: redis
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-cfnbd 
