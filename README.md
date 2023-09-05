-![image](https://github.com/wujie1234/apisix-k8s/assets/63633025/36f6483f-e024-483f-a23e-9081bdc5b50f)
Apache APISIX 是一个基于 OpenResty 和 Etcd 实现的动态、实时、高性能的 API 网关，目前已经是 Apache 顶级项目。提供了丰富的流量管理功能，如负载均衡、动态路由、动态 upstream、A/B 测试、金丝雀发布、限速、熔断、防御恶意攻击、认证、监控指标、服务可观测性、服务治理等。可以使用 APISIX 来处理传统的南北流量以及服务之间的东西向流量。


APISIX 基于 Nginx 和 etcd，与传统 API 网关相比，APISIX 具有动态路由和热加载插件功能，避免了配置之后的 reload 操作，同时 APISIX 支持 HTTP(S)、HTTP2、Dubbo、QUIC、MQTT、TCP/UDP 等更多的协议。而且还内置了 Dashboard，提供强大而灵活的界面。同样也提供了丰富的插件支持功能，而且还可以让用户自定义插件。


安装
这里咱们使用helm安装
➜ helm repo add apisix https://charts.apiseven.com
➜ helm repo update


由于 APISIX 的 Chart 包中包含 dashboard 和 ingress 控制器的依赖，我们只需要在 values 中启用即可安装 ingress 控制器了：
➜ helm fetch apisix/apisix
➜ tar -xvf apisix-0.7.2.tgz
➜ mkdir -p apisix/ci


由于apisix需要默认的sc进行绑定这里咱们使用nfs的helm包进行绑定
➜ helm repo add nfs-subdir-external-provisioner https:kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
➜ helm upgrade --install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=192.168.31.31 \  
--set nfs.path=/var/lib/k8s/data \
--set image.repository=cnych/nfs-subdir-external-provisioner \
--set storageClass.defaultClass=true -n kube-system![image](https://github.com/wujie1234/apisix-k8s/assets/63633025/36f6483f-e024-483f-a23e-9081bdc5b50f)
Apache APISIX 是一个基于 OpenResty 和 Etcd 实现的动态、实时、高性能的 API 网关，目前已经是 Apache 顶级项目。提供了丰富的流量管理功能，如负载均衡、动态路由、动态 upstream、A/B 测试、金丝雀发布、限速、熔断、防御恶意攻击、认证、监控指标、服务可观测性、服务治理等。可以使用 APISIX 来处理传统的南北流量以及服务之间的东西向流量。


APISIX 基于 Nginx 和 etcd，与传统 API 网关相比，APISIX 具有动态路由和热加载插件功能，避免了配置之后的 reload 操作，同时 APISIX 支持 HTTP(S)、HTTP2、Dubbo、QUIC、MQTT、TCP/UDP 等更多的协议。而且还内置了 Dashboard，提供强大而灵活的界面。同样也提供了丰富的插件支持功能，而且还可以让用户自定义插件。


安装
这里咱们使用helm安装
➜ helm repo add apisix https://charts.apiseven.com
➜ helm repo update


由于 APISIX 的 Chart 包中包含 dashboard 和 ingress 控制器的依赖，我们只需要在 values 中启用即可安装 ingress 控制器了：
➜ helm fetch apisix/apisix
➜ tar -xvf apisix-0.7.2.tgz
➜ mkdir -p apisix/ci


由于apisix需要默认的sc进行绑定这里咱们使用nfs的helm包进行绑定
➜ helm repo add nfs-subdir-external-provisioner https:kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
➜ helm upgrade --install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=192.168.31.31 \  ---nfs节点ip
--set nfs.path=/var/lib/k8s/data \  
--set image.repository=cnych/nfs-subdir-external-provisioner \
--set storageClass.defaultClass=true -n kube-system
安装完后会创建出对应的sc
<img width="989" alt="image" src="https://github.com/wujie1234/apisix-k8s/assets/63633025/4b1f7341-c13f-47c6-8cf2-48d5d6d2be08">

在 apisix/ci 目录中新建一个用于安装的 values 文件，内容如下所示：
# ci/prod.yaml
apisix:
  enabled: true

  nodeSelector: # 固定在node2节点上
    kubernetes.io/hostname: node2

gateway:
  type: NodePort
  externalTrafficPolicy: Cluster
  http:
    enabled: true
    servicePort: 80
    containerPort: 9080
  tls:
    enabled: true # 启用 tls
    servicePort: 443
    containerPort: 9443

etcd:
  enabled: true # 会自动创建3个节点的etcd集群
  replicaCount: 1 # 多副本需要修改下模板，这里暂时运行一个etcd pod

dashboard:
  enabled: true

ingress-controller:
  enabled: true
  config:
    apisix:
      serviceName: apisix-admin
      serviceNamespace: apisix # 指定命名空间，如果不是 ingress-apisix 需要重新指定

