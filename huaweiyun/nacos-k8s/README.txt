

#springcloud集群配置

spring:
  cloud:
    nacos:
      discovery:
        #nacos环境
        namespace: lapis-pre
        # 服务注册地址
        server-addr: nacos-headless.lapis-cmn:8848