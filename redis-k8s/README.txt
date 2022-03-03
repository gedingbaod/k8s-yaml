

#创建cluster后执行此命令
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')



kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 172.17.1.5:6379 172.17.0.4:6379 172.17.0.135:6379 172.17.0.5:6379 172.17.0.136:6379 172.17.1.6:6379


#springclude配置
spring:
  redis:
    cluster:
      #设置key的生存时间，当key过期时，它会被自动删除；
      expire-seconds: 120
      #设置命令的执行时间，如果超过这个时间，则报错;
      command-timeout: 5000
      #设置redis集群的节点信息，其中namenode为域名解析，通过解析域名来获取相应的地址;
      nodes: redis-cluster-0.redis-headless.lapis-cmn:6379redis-cluster-1.redis-headless.lapis-cmn:6379redis-cluster-2.redis-headless.lapis-cmn:6379redis-cluster-3.redis-headless.lapis-cmn:6379redis-cluster-4.redis-headless.lapis-cmn:6379redis-cluster-5.redis-headless.lapis-cmn:6379
