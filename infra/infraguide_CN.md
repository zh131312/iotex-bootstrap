## IoTeX 代表(delegate)基础设施指南

1. 基于商业云负载均衡器运行您的节点。

   像amazon AWS[^1], google GCE[^2] or Cloudflare[^3] 等云提供商会提供良好的保护, 防止大量的包括 SYN floods, IP fragment floods, port exhaustion在内的 L3, L4 DDoS 攻击, 我们建议使用这些专业解决方案来帮助您保护您的节点

2. 将Envoy Proxy作为边缘代理放在节点前面。

   [Envoy](https://www.envoyproxy.io/) 与 [ratelimit](https://github.com/lyft/ratelimit) 对HTTP / HTTP2和所有网络流量的连接限制提供请求速率限制。将您的IoTeX节点置于Envoy代理后面，为您提供针对攻击者的另一层保护。此外，由于IoTeX节点API使用GRPC，如果您想成为服务节点并为浏览器单页面应用程序提供API请求, 您将需要Envoy的 [grpc-web](https://github.com/grpc/grpc-web) 过滤器以启用此功能。

   在我们的测试网络集群中，我们在HTTP / HTTP2请求上为每个IP速率上限为每秒20个请求

   ```yaml
   domain: iotex-api
   descriptors:
     - key: remote_address
       rate_limit:
         unit: second
         requests_per_unit: 20
   ---
   domain: iotex-stats
   descriptors:
     - key: remote_address
       rate_limit:
         unit: second
         requests_per_unit: 20
   ```

   并且最大连接数设置为50

   ```yaml
   circuit_breakers:
     thresholds:
       max_connections: 50
       max_pending_requests: 500
       max_requests: 100
       max_retries: 3
   ```

   

   以下是完整配置: [envoy config](https://gist.github.com/yutongp/c61292bf5c9c6e3058df96989365cb0c)

3. 使用Kubernetes进行部署和运行状况监控。

   [Kubernetes](https://kubernetes.io)(k8s) 是一个非常强大的用于管理您的容器化服务工具。在Kubernetes中部署和升级在docker容器中运行的服务非常方便。

   k8s 还将执行运行状况检查并自动重启下降节点以最大化正常运行时间。

   我们在测试环境中非常依赖k8s。根据我们自己的经验，我们可以确定k8s肯定会降低您的运营成本。


   这些是我们在deployment.yaml中使用的探测器配置：
   
   ```yaml
   livenessProbe:
     httpGet:
       path: "/liveness"
       port: 8080
     initialDelaySeconds: 15
     timeoutSeconds: 2
     periodSeconds: 15
     failureThreshold: 5
   readinessProbe:
     httpGet:
       path: "/readiness"
       port: 8080
     initialDelaySeconds: 30
     timeoutSeconds: 2
     periodSeconds: 15
     failureThreshold: 5
   ```

   我们目前为testnet运行的代理机器人设置与下图非常相似:![infra](infra.png?raw=true)
   
   ## 高可用性

 由于代码错误，计算资源不足，主机故障，网络故障等原因，一个IoTeX节点可能处于中断状态，或者您必须关闭节点以执行某些维护工作。运行多个IoTeX节点是保证授权的高可用性（零停机时间）的方法。我们提供了方便地运行这些节点的功能。
 
 假设您将以一个注册身份运行3个节点，1个将主动运行并参与共识工作，2个将待机并且仅监听块。
 
 所有节点可以使用同样的 `producerPrivKey`. 对于所有的节点，添加下述设置到 `config.yaml`:

 ```yaml
...
network:
  ...
  masterKey: producer_private_key-replica_id
  ...
...
```

 对于主动节点，天下如下命令到 `config.yaml`:

 ```yaml
...
system:
  ...
  active: true
  ...
...
```

 对于待机节点，添加以下命令到 `config.yaml`: 

 ```yaml
...
system:
  ...
  active: false
  ...
...
```

 除此之外, 从节点的docker container导出 `9009` 端口. 一旦主动节点关闭，使用`http://ip-to-one-node:9009/ha?activate=true` 将待机节点变为主动模式。同样，您可以使用`http://ip-to-one-node:9009/ha?activate=false`将活动节点转为待机模式。 `http://ip-to-one-node:9009/ha`可以告诉你节点的模式。

 如果您有相当多的节点，并希望从您的节点中删除繁琐的手动操作，或者只是想尝试高可用性集群的设置，请在[此处](https://github.com/zjshen14/iotex-leader-election)查看领导者选举解决方案。
 
## 让它变得更好
   运行分布式基础设施仍在起步阶段，我们还有很多东西需要学习和改进。去中心化模式可以帮我们从不同角度看待问题，希望大家集思广益，与我们共同建立一个更开放的高性能网络。


[^1]: https://aws.amazon.com/answers/networking/aws-ddos-attack-mitigation/
[^2]: https://cloud.google.com/files/GCPDDoSprotection-04122016.pdf
[^3]: https://www.cloudflare.com/ddos/
