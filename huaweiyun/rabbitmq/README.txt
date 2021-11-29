# prevision sc, default sas
kubectl apply -f csi-disk-sas.yaml
kubectl apply -f csi-disk-ssd.yaml

# prevision image
docker pull rabbitmq:3.8.21-management
docker tag rabbitmq:3.8.21-management swr.cn-east-3.myhuaweicloud.com/lapis/rabbitmq:3.8.21-management
docker push swr.cn-east-3.myhuaweicloud.com/lapis/rabbitmq:3.8.21-management

# install RabbitMQ Operator
kubectl apply -f "https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml"

# check resource
kubectl get all -n rabbitmq-system
kubectl get customresourcedefinitions.apiextensions.k8s.io

# apply k8s yaml
kubectl apply -f rabbitmq-cluster.yaml

# add ingress domain cmn-ssl
rabbitmq.laison.top

# get user password
username="$(kubectl -n lapis-cmn get secret rabbitmq-cluster-security-default-user -o jsonpath='{.data.username}' | base64 --decode)"
default_user_iKUN4RWdsEpjX5EPxGn
password="$(kubectl -n lapis-cmn get secret rabbitmq-cluster-security-default-user -o jsonpath='{.data.password}' | base64 --decode)"
ZVmnAn7ZwTntvZHfFExl3v-fPneyAI8k

# login http

# add vhost

# add policy

# test queue

# set mem mod
rabbit@rabbitmq-cluster-security-server-0.rabbitmq-cluster-security-nodes.lapis-cmn
k -n lapis-cmn exec -it rabbitmq-cluster-security-server-0 -- rabbitmqctl stop_app
k -n lapis-cmn exec -it rabbitmq-cluster-security-server-0 -- rabbitmqctl change_cluster_node_type ram
k -n lapis-cmn exec -it rabbitmq-cluster-security-server-0 -- rabbitmqctl start_app

# use
https://rabbitmq.laison.top
管理用户：admin Admin123
消息收发用户： lapis lapis
容器连接：--uri amqp://lapis:lapis@rabbitmq-cluster-security
内部端口：5672
外部端口：124.71.203.43:32672



