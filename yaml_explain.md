apiVersion: v1  
kind: Pod 
metadata:  
  name: web04-pod  
  labels:  
    k8s-app: apache  
spec: 
  nodeSelector: 
    disk: ssd  
  containers:  
  - name: web04-pod  
    image: web:apache 
  - name: nginx
    image: nginx
    command: ['sh']   
    env:  
    - name: TOPSECRET
      valueFrom:
        secretKeyRef:
          name: super-secret
          key: Credential 
    volumeMounts:  
    - name: volume  
      mountPath: /data  
  initContainers:
    - name: init-lumpy-koala
      image: busybox
      command: ['sh','-c','touch /workdir/calm.txt']
      volumeMounts:
        - name: workdir
          mountPath: /workdir
  volumes:  
  - name: volume  
    #emptyDir: {}  
    hostPath:  
      path: /opt
    secret:
      secretName: super-secret	 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxapp
  labels:
    app: nginxapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.11.9-alpine
        ports:
        - containerPort: 80

apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-config
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /srv/app-config	  
	  
