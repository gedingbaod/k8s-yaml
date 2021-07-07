0.安装Helm
wget https://mirrors.huaweicloud.com/helm/v3.6.2/helm-v3.6.2-linux-amd64.tar.gz

# 解压 Helm
tar -zxvf helm-v3.6.2-linux-amd64.tar.gz

# 复制客户端执行文件到 bin 目录下，方便在系统下能执行 helm 命令
cp linux-amd64/helm /usr/local/bin/

# 检查
helm version
helm env

# 添加仓库
helm repo add microsoft https://microsoft.github.io/charts/repo
helm repo add elastic https://helm.elastic.co
helm dep up skywalking
helm repo update

# 把k8s环境变量KUBECONFIG进行配置
export KUBECONFIG=/root/.kube/config #可以写到/etc/profile里


1.部署skywalking的oap


git clone https://github.com/apache/skywalking-kubernetes

cd skywalking-kubernetes/chart
helm repo add elastic https://helm.elastic.co
helm repo update
export SKYWALKING_RELEASE_NAME=skywalking  # change the release name according to your scenario
export SKYWALKING_RELEASE_NAMESPACE=lapis-cmn  # change the namespace according to your scenario


docker pull apache/skywalking-oap-server:8.6.0-es7
docker pull apache/skywalking-ui:8.6.0
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.5.1

必须在此目录下执行
cd skywalking-kubernetes/chart/


helm install "${SKYWALKING_RELEASE_NAME}" skywalking -n "${SKYWALKING_RELEASE_NAMESPACE}" \
  --set oap.image.tag=8.6.0-es7 \
  --set oap.storageType=elasticsearch7 \
  --set ui.image.tag=8.6.0 \
  --set elasticsearch.imageTag=7.5.1

删除命令
helm delete skywalking -n "${SKYWALKING_RELEASE_NAMESPACE}" 

创建了skywalking的ingress


----------------------------------------------------


2.制作SIDECAR


下载SkyWalking官方发行包，并解压到指定目录
wget https://archive.apache.org/dist/skywalking/8.6.0/apache-skywalking-apm-es7-8.6.0.tar.gz
tar -zxvf apache-skywalking-apm-es7-8.6.0.tar.gz
apache-skywalking-apm-bin-es7


构建skywalking-agentsidecar镜像并push至hub私有镜像仓库

创建 Dockerfile
FROM busybox:1.28.4
ENV LANG=C.UTF-8
RUN set -eux && mkdir -p /usr/skywalking/agent
add ./agent /usr/skywalking/agent
WORKDIR /

docker build .  -t <harbor.laisontech.com/lapis>/skywalking-agent-sidecar:8.6.0

docker push <harbor.laisontech.com/lapis>/skywalking-agent-sidecar:8.6.0

----------------------------------------------------
在部署yaml文件中增加下面两段
      initContainers:
        - image: harbor.laisontech.com/lapis/skywalking-agent-sidecar:8.6.0
          name: sw-agent-sidecar
          imagePullPolicy: IfNotPresent
          command: [ "sh" ]
          args:
            [
                "-c",
                "mkdir -p /skywalking/agent && cp -r /usr/skywalking/agent/* /skywalking/agent",
            ]
          volumeMounts:
            - mountPath: /skywalking/agent
              name: sw-agent



          env:
            #这里通过JAVA_TOOL_OPTIONS，而不是JAVA_OPTS可以实现不通过将agent命令加入到java应用jvm参数而实现agent的集成
            - name: JAVA_TOOL_OPTIONS
              value: -javaagent:/usr/skywalking/agent/skywalking-agent.jar
            - name: SW_AGENT_NAME
              value: lapis-module-job-app
            - name: SW_AGENT_COLLECTOR_BACKEND_SERVICES
              value: skywalking-oap.lapis-cmn.svc.cluster.local
          volumeMounts:
            - mountPath: /usr/skywalking/agent
              name: sw-agent
      volumes:
        - name: sw-agent
          emptyDir: {}