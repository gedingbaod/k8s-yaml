

#创建cluster后执行此命令
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')


kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '

kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 172.17.1.15:6379 172.17.0.13:6379 172.17.0.146:6379 172.17.0.14:6379 172.17.1.17:6379 172.17.0.145:6379 
# 查看集群状态
kubectl exec -it redis-cluster-0 -- redis-cli cluster info
# 查看集群节点
kubectl exec -it redis-cluster-0 -- redis-cli cluster nodes

#springclude配置
spring:
  redis:
    cluster:
      #设置key的生存时间，当key过期时，它会被自动删除；
      expire-seconds: 120
      #设置命令的执行时间，如果超过这个时间，则报错;
      command-timeout: 5000
      #设置redis集群的节点信息，其中namenode为域名解析，通过解析域名来获取相应的地址;
      nodes: redis-cluster-0.redis-headless.lapis-cmn:6379,redis-cluster-1.redis-headless.lapis-cmn:6379,redis-cluster-2.redis-headless.lapis-cmn:6379,redis-cluster-3.redis-headless.lapis-cmn:6379,redis-cluster-4.redis-headless.lapis-cmn:6379,redis-cluster-5.redis-headless.lapis-cmn:6379



#手工处理集群重启问题

kubectl exec -it redis-cluster-0 -- redis-cli flushall &&
kubectl exec -it redis-cluster-1 -- redis-cli flushall &&
kubectl exec -it redis-cluster-2 -- redis-cli flushall &&
kubectl exec -it redis-cluster-3 -- redis-cli flushall &&
kubectl exec -it redis-cluster-4 -- redis-cli flushall &&
kubectl exec -it redis-cluster-5 -- redis-cli flushall

kubectl exec -it redis-cluster-0 -- redis-cli cluster reset &&
kubectl exec -it redis-cluster-1 -- redis-cli cluster reset &&
kubectl exec -it redis-cluster-2 -- redis-cli cluster reset &&
kubectl exec -it redis-cluster-3 -- redis-cli cluster reset &&
kubectl exec -it redis-cluster-4 -- redis-cli cluster reset &&
kubectl exec -it redis-cluster-5 -- redis-cli cluster reset

kubectl exec -it redis-cluster-0 -- rm -fr dump.rdb nodes.conf appendonly.aof &&
kubectl exec -it redis-cluster-1 -- rm -fr dump.rdb nodes.conf appendonly.aof &&
kubectl exec -it redis-cluster-2 -- rm -fr dump.rdb nodes.conf appendonly.aof &&
kubectl exec -it redis-cluster-3 -- rm -fr dump.rdb nodes.conf appendonly.aof &&
kubectl exec -it redis-cluster-4 -- rm -fr dump.rdb nodes.conf appendonly.aof &&
kubectl exec -it redis-cluster-5 -- rm -fr dump.rdb nodes.conf appendonly.aof

kubectl exec -it redis-cluster-0 -- ls &&
kubectl exec -it redis-cluster-1 -- ls &&
kubectl exec -it redis-cluster-2 -- ls &&
kubectl exec -it redis-cluster-3 -- ls &&
kubectl exec -it redis-cluster-4 -- ls &&
kubectl exec -it redis-cluster-5 -- ls


kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '

kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 172.17.1.15:6379 172.17.0.13:6379 172.17.0.146:6379 172.17.0.14:6379 172.17.1.17:6379 172.17.0.145:6379 
