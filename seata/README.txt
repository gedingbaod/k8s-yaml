

#pull and push to huawei docker server
docker pull seataio/seata-server:1.4.2

docker tag seataio/seata-server:1.4.2 swr.cn-east-3.myhuaweicloud.com/lapis/seata-server:1.4.2
 
docker push swr.cn-east-3.myhuaweicloud.com/lapis/seata-server:1.4.2



# create configmap
kubectl create configmap  cm-seata-registery --from-file=./registry.conf -n lapis-prod

kubectl create configmap  cm-seata-file --from-file=./file.conf -n lapis-prod
