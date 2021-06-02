https://blog.csdn.net/weixin_34375233/article/details/92229312
https://blog.csdn.net/hjnth/article/details/107561820


创建ConfigMap
kubectl create configmap  cm-nginx-default-config --from-file=./default.conf -n lapis-dev
kubectl create configmap  cm-nginx-config --from-file=./nginx.conf -n lapis-dev

创建容器
kubectl apply -f lapis-log.yaml
创建NodePort
kubectl apply -f lapis-log-svc.yaml