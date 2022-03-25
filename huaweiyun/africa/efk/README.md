



### 五、创建集群证书
#### 1、生成证书文件
##### 运行容器生成证书

```bash
docker run --name elastic-charts-certs -i -w /app elasticsearch:7.7.1 /bin/sh -c  \
  "elasticsearch-certutil ca --out /app/elastic-stack-ca.p12 --pass '' \
    && elasticsearch-certutil cert --name security-master \
    --dns security-master --ca /app/elastic-stack-ca.p12 --pass '' \
    --ca-pass '' --out /app/elastic-certificates.p12"
```

##### 从容器中将生成的证书拷贝出来
```bash 
docker cp elastic-charts-certs:/app/elastic-certificates.p12 ./ 
```

##### 删除容器

```bash
docker rm -f elastic-charts-certs
```

##### 将 pcks12 中的信息分离出来，写入文件

```bash
openssl pkcs12 -nodes -passin pass:'' -in elastic-certificates.p12 -out elastic-certificate.pem
```

#### 2、添加证书到 Kubernetes

##### 添加证书到K8S
```bash
kubectl create secret -n lapis-es generic elastic-certificates --from-file=elastic-certificates.p12
```

##### 设置集群用户名密码，用户名不建议修改
```bash 
kubectl create secret -n lapis-es generic elastic-credentials --from-literal=username=elastic --from-literal=password=Admin123
```

#### 六、配置应用参数

通过 Helm 安装 ElasticSearch、Kibana 需要事先创建一个带有配置参数的 values.yaml 文件。然后再执行 Helm install 安装命令时，指定使用此文件。

#### 1、创建 ElasticSearch Master 安装的配置文件

```bash
vi es-master-values.yaml 
```

```bash
# ============设置集群名称============
## 设置集群名称
clusterName: "elastic"
## 设置节点名称
nodeGroup: "master"
## 设置角色
roles:
  - master

# ============镜像配置============
## 指定镜像与镜像版本
image: "docker.elastic.co/elasticsearch/elasticsearch"
imageTag: "7.7.1"
## 副本数
replicas: 3

# ============资源配置============
## JVM 配置参数
esJavaOpts: "-Xmx1g -Xms1g"
## 部署资源配置(生成环境一定要设置大些)
resources:
  requests:
    cpu: "2000m"
    memory: "2Gi"
  limits:
    cpu: "2000m"
    memory: "2Gi"

## 数据持久卷配置
persistence:
  enabled: true

## 存储数据大小配置
volumeClaimTemplate:
  storageClassName: local-path-delete
  accessModes: [ "ReadWriteOnce" ]
  resources:
    requests:
      storage: 5Gi

# ============安全配置============
## 设置协议，可配置为 http、https
protocol: http

## 不要创建验证文件
createCert: false

## 不用自动生成用户密码
secret:
  enabled: false
  
## 证书挂载配置，这里我们挂入上面创建的证书
secretMounts:
  - name: elastic-certificates
    secretName: elastic-certificates
    path: /usr/share/elasticsearch/config/certs

## 允许您在/usr/share/elasticsearch/config/中添加任何自定义配置文件,例如 elasticsearch.yml
## ElasticSearch 7.x 默认安装了 x-pack 插件，部分功能免费，这里我们配置下
## 下面注掉的部分为配置 https 证书，配置此部分还需要配置 helm 参数 protocol 值改为 https
esConfig:
  elasticsearch.yml: |
    xpack.security.enabled: true
    xpack.security.transport.ssl.enabled: true
    xpack.security.transport.ssl.verification_mode: certificate
    xpack.security.transport.ssl.keystore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    xpack.security.transport.ssl.truststore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    # xpack.security.http.ssl.enabled: true
    # xpack.security.http.ssl.truststore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    # xpack.security.http.ssl.keystore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12

## 环境变量配置，这里引入上面设置的用户名、密码 secret 文件
extraEnvs:
  - name: ELASTIC_USERNAME
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: username
  - name: ELASTIC_PASSWORD
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: password

# ============调度配置============
## 设置调度策略
## - hard：只有当有足够的节点时 Pod 才会被调度，并且它们永远不会出现在同一个节点上
## - soft：尽最大努力调度
antiAffinity: "hard"

## 容忍配置（一般 kubernetes master 或其它设置污点的节点，只有指定容忍才能进行调度，如果测试环境只有三个节点，则可以开启在 master 节点安装应用）
#tolerations: 
#  - operator: "Exists"  ##容忍全部污点
```



#### 2、创建 ElasticSearch Data 安装的配置文件

```bash
vi es-data-values.yaml
```

```bash
# ============设置集群名称============
## 设置集群名称
clusterName: "elastic"
## 设置节点名称
nodeGroup: "data"
## 设置角色
roles:
  - data
  - data_content
  - data_hot
  - data_warm
  - data_cold
  - data_frozen
  - ingest

# ============镜像配置============
## 指定镜像与镜像版本
image: "docker.elastic.co/elasticsearch/elasticsearch"
imageTag: "7.7.1"
## 副本数
replicas: 3

# ============资源配置============
## JVM 配置参数
esJavaOpts: "-Xmx1g -Xms1g"
## 部署资源配置(生成环境一定要设置大些)
resources:
  requests:
    cpu: "1000m"
    memory: "2Gi"
  limits:
    cpu: "1000m"
    memory: "2Gi"
    
## 数据持久卷配置
persistence:
  enabled: true
  
## 存储数据大小配置
volumeClaimTemplate:
  storageClassName: local-path-delete
  accessModes: [ "ReadWriteOnce" ]
  resources:
    requests:
      storage: 100Gi

# ============安全配置============
## 设置协议，可配置为 http、https
protocol: http

## 证书挂载配置，这里我们挂入上面创建的证书
secretMounts:
  - name: elastic-certificates
    secretName: elastic-certificates
    path: /usr/share/elasticsearch/config/certs
    
## 允许您在/usr/share/elasticsearch/config/中添加任何自定义配置文件,例如 elasticsearch.yml
## ElasticSearch 7.x 默认安装了 x-pack 插件，部分功能免费，这里我们配置下
## 下面注掉的部分为配置 https 证书，配置此部分还需要配置 helm 参数 protocol 值改为 https
esConfig:
  elasticsearch.yml: |
    xpack.security.enabled: true
    xpack.security.transport.ssl.enabled: true
    xpack.security.transport.ssl.verification_mode: certificate
    xpack.security.transport.ssl.keystore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    xpack.security.transport.ssl.truststore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    # xpack.security.http.ssl.enabled: true
    # xpack.security.http.ssl.truststore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    # xpack.security.http.ssl.keystore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    
## 环境变量配置，这里引入上面设置的用户名、密码 secret 文件
extraEnvs:
  - name: ELASTIC_USERNAME
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: username
  - name: ELASTIC_PASSWORD
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: password

# ============调度配置============
## 设置调度策略
## - hard：只有当有足够的节点时 Pod 才会被调度，并且它们永远不会出现在同一个节点上
## - soft：尽最大努力调度
antiAffinity: "hard"

## 容忍配置（一般 kubernetes master 或其它设置污点的节点，只有指定容忍才能进行调度，如果测试环境只有三个节点，则可以开启在 master 节点安装应用）
#tolerations: 
#  - operator: "Exists"  ##容忍全部污点
```

#### 3、创建 ElasticSearch Client 安装的配置文件

```bash
vi es-client-values.yaml 
```

```bash
# ============设置集群名称============
## 设置集群名称
clusterName: "elastic"
## 设置节点名称
nodeGroup: "client"
## 设置角色
roles: []

# ============镜像配置============
## 指定镜像与镜像版本
image: "docker.elastic.co/elasticsearch/elasticsearch"
imageTag: "7.7.1"
## 副本数
replicas: 3

# ============资源配置============
## JVM 配置参数
esJavaOpts: "-Xmx1g -Xms1g"

## 部署资源配置(生成环境一定要设置大些)
resources:
  requests:
    cpu: "1000m"
    memory: "2Gi"
  limits:
    cpu: "1000m"
    memory: "2Gi"
    
## 数据持久卷配置
persistence:
  enabled: false

# ============安全配置============
## 设置协议，可配置为 http、https
protocol: http

## 证书挂载配置，这里我们挂入上面创建的证书
secretMounts:
  - name: elastic-certificates
    secretName: elastic-certificates
    path: /usr/share/elasticsearch/config/certs
    
## 允许您在/usr/share/elasticsearch/config/中添加任何自定义配置文件,例如 elasticsearch.yml
## ElasticSearch 7.x 默认安装了 x-pack 插件，部分功能免费，这里我们配置下
## 下面注掉的部分为配置 https 证书，配置此部分还需要配置 helm 参数 protocol 值改为 https
esConfig:
  elasticsearch.yml: |
    xpack.security.enabled: true
    xpack.security.transport.ssl.enabled: true
    xpack.security.transport.ssl.verification_mode: certificate
    xpack.security.transport.ssl.keystore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    xpack.security.transport.ssl.truststore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    # xpack.security.http.ssl.enabled: true
    # xpack.security.http.ssl.truststore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    # xpack.security.http.ssl.keystore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    
## 环境变量配置，这里引入上面设置的用户名、密码 secret 文件
extraEnvs:
  - name: ELASTIC_USERNAME
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: username
  - name: ELASTIC_PASSWORD
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: password

# ============Service 配置============
service:
  type: NodePort
  nodePort: "30200"
```

#### 4、创建 Kibana 安装的配置文件

```bash 
vi kibana-values.yaml 
```

```bash
# ============镜像配置============
## 指定镜像与镜像版本
image: "docker.elastic.co/kibana/kibana"
imageTag: "7.7.1"

## 配置 ElasticSearch 地址
elasticsearchHosts: "http://elasticsearch-client:9200"

# ============环境变量配置============
## 环境变量配置，这里引入上面设置的用户名、密码 secret 文件
extraEnvs:
  - name: 'ELASTICSEARCH_USERNAME'
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: username
  - name: 'ELASTICSEARCH_PASSWORD'
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: password

# ============资源配置============
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
  limits:
    cpu: "500m"
    memory: "1Gi"

# ============配置 Kibana 参数============ 
## kibana 配置中添加语言配置，设置 kibana 为中文
kibanaConfig:
  kibana.yml: |
        i18n.locale: "zh-CN"

# ============Service 配置============
service:
  type: NodePort
  nodePort: "30601"
```

### 七、Helm 安装 ElasticSearch、Kibana

#### 1、Helm 增加 Elastic 仓库

```bash
helm repo add elastic https://helm.elastic.co
```

#### 2、Helm 安装 ElasticSearch

```bash
## 安装 ElasticSearch Master 节点
helm install elastic-master -f es-master-values.yaml --namespace lapis-es --version 7.7.1 ../helm-charts/elasticsearch

## 安装 ElasticSearch Data 节点
helm install elasticsearch-data -f es-data-values.yaml --namespace lapis-es --version 7.7.1 ../helm-charts/elasticsearch

## 安装 ElasticSearch Client 节点
helm install elasticsearch-client -f es-client-values.yaml --namespace lapis-es --version 7.7.1 ../helm-charts/elasticsearch
```

