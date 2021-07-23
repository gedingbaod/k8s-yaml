https://www.cnblogs.com/Sunzz/p/13716477.html

git clone https://gitee.com/SunHarvey/helloworld_java.git

1. 获取GitLab Runner的注册信息
登录你的GitLab。
在顶部导航栏中，选择Projects > Your projects。
在Your projects页签下，选择相应的Project。
在左侧导航栏中，选择Settings > CI / CD。
单击Runners右侧的Expand。

复制信息
gitlabUrl: http://xx.xx.xx.xx/
runnerRegistrationToken: "xxxxxx"

到values.yaml，同时修改namespace


helm package .
helm install --namespace gitlab  gitlab-runner *.tgz
在k8s中可以看到gitlab-runner已经启动

2.在某个项目的导航栏中
在左侧导航栏中，选择Settings > CI / CD。
单击Variables右侧的Expand。添加GitLab Runner可用的环境变量。
注意不要选择保护模式
REGISTRY_USERNAME：镜像仓库用户名。
REGISTRY_PASSWORD：镜像仓库密码。
kube_config：KubeConfig的编码字符串。echo $(cat ~/.kube/config | base64) | tr -d " "


编写.gitlab-ci.yml
image: docker:stable
stages:
  - package
  - docker_build
  - deploy_k8s
variables:
  KUBECONFIG: /etc/deploy/config
  MAVEN_OPTS: "-Dmaven.repo.local=/opt/cache/.m2/repository"
mvn_build_job:
  image: harbor.laisontech.com/lapis/maven:3.6.2-jdk-8
  stage: package
  tags:
    - k8s-runner
  script:
    - mvn clean install
    - cp target/gitlab-k8s-example.jar /opt/cache/
docker_build_job:
  image: harbor.laisontech.com/lapis/docker:20.10.2
  stage: docker_build
  tags:
    - k8s-runner
  script:
    - echo $REGISTRY_USERNAME
    - echo $REGISTRY_PASSWORD
    - echo $kube_config
    - docker login -u gitlab -p Admin123 harbor.laisontech.com
    - pwd
    - ls -l /opt/cache/gitlab-k8s-example.jar
    - ls -l
    - mkdir target
    - ls -l target
    - cp /opt/cache/gitlab-k8s-example.jar target/gitlab-k8s-example.jar
    - ls -l target/gitlab-k8s-example.jar
    - echo "get CI_PIPELINE_ID value"
    - echo "$CI_PIPELINE_ID"
    - docker build -t harbor.laisontech.com/lapis/gitlab-k8s-example:$CI_PIPELINE_ID .
    - docker push harbor.laisontech.com/lapis/gitlab-k8s-example:$CI_PIPELINE_ID
    - docker rmi harbor.laisontech.com/lapis/gitlab-k8s-example:$CI_PIPELINE_ID

deploy_k8s_job:
  #image: registry.cn-hangzhou.aliyuncs.com/haoshuwei24/kubectl:1.16.6
  #image: registry.cn-hangzhou.aliyuncs.com/sanchar/kubectl:v1.20.1
  image: harbor.laisontech.com/lapis/kubectl:v1.20.1
  stage: deploy_k8s
  tags:
    - k8s-runner
  script:
    - cat /etc/hosts
    - echo "192.168.2.202 master02" >> /etc/hosts
    - cat /etc/hosts
    - mkdir -p /etc/deploy
    - echo $env:kube_config
    - echo "$env:kube_config"
    - echo $kube_config |base64 -d > $KUBECONFIG
    - sed -i "s/IMAGE_TAG/$CI_PIPELINE_ID/g" deployment.yaml
    - cat deployment.yaml
    - kubectl delete -f deployment.yaml
    - kubectl apply -f deployment.yaml

编写Dockerfile
FROM harbor.laisontech.com/lapis/jdk8:v4
MAINTAINER SunHarvey
ENV USER centos
ENV APP_HOME /home/$USER/apps/
ADD target/gitlab-k8s-example.jar $APP_HOME
WORKDIR $APP_HOME
ENTRYPOINT ["java","-Dfile.encoding=UTF-8", "-jar", "gitlab-k8s-example.jar"]

编写deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab-k8s-example
  namespace: devops
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gitlab-k8s-example
  template:
    metadata:
      labels:
        app: gitlab-k8s-example
    spec:
      imagePullSecrets:
        - name: myregistry
      containers:
      - name: gitlab-k8s-example
        image: harbor.laisontech.com/lapis/gitlab-k8s-example:IMAGE_TAG
        imagePullPolicy: Always
        ports:
        - containerPort: 8085
---
apiVersion: v1
kind: Service
metadata:
  name: gitlab-k8s-example-svc
  namespace: devops
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8085
  selector:
    app: gitlab-k8s-example
  type: LoadBalancer
---
apiVersion: v1
kind: Service
metadata:
  name: gitlab-k8s-example-np
  namespace: devops
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8085
  selector:
    app: gitlab-k8s-example
  type: NodePort



使用Pipeline部署项目