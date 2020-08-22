1,yaml file 不支持_
 2, 不支持  none-persistent-redis  第而个字符 大写， 如 这里写成none-Persistent-redis  会报错
 apiVersion: v1
kind : Pod
metadata:
  name: none-persistent-redis
spec:
