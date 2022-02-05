# Devops_CloudPlatform

## 云服务器
1. 部署云计算资源的方法
- 公有云&私有云&混合云  

2. 安全组：防火墙的端口设置（访问：入方向）  

3. 公有IP/私有IP  
- 同集群内用私有ip交互

4. 私有网络/专有网络VPC -> 交换机  
- 划分网段 -> 进一步划分子网络
- 提供隔离的网络

## Docker
1. 解决的问题(虚拟化技术->容器化技术)
- 统一标准
```
# 应用构建
docker build
# 分享
放到docker hub里docker pull
# 运行
docker run
```
- 资源隔离

2. 架构
```
Registry->
DockerHost(DockerDaemon + Images->Containers集群)
<-DockerClient(build/pull/run)
```

## K8S
1. 容器化部署
- hardware-OS-ContainerRuntime-Container(Bin/Lib+App)

2. 特性
- 服务发现和负载均衡
- 存储编排
- 自动部署和回滚
- 自动完成装箱计算
- 自我修复
- 密钥与配置管理

3. 架构
- K8S集群=N主节点(奇数个)+N工作节点
  - 每个节点都要装容器运行时环境
- 控制平面组件(总部-master节点)
  - cloud controller manager(外联部[可选]，连接cloud provider API)
  - controller manager(决策者)
  - etcd键值数据库(资料库)
  - api server(秘书部)
  - scheduler(调度者)
- Node组件(分厂)
  - kubelet(厂长)->探测应用情况+启停
  - kube-proxy(门卫)->所有应用间的网络访问
- 命令行kybectl
  - kubectl create deploy xxapp --image=镜像名

## Progression
```
(0) K8S 证书 -> CKA / CKS
Java Codes -> Github
Java 打包构建 -> Maven/Jenkins(CI/CD)
Java 测试 -> Junit
Java Jar -> Nexus
(1) Docker Image -> Docker Hub
(2 & 5) K8S Pod -> Helm / Harbor
(3 & 4) DevOps K8S流水线平台 -> KubeSphere
(6) 微服务网络 -> ServiceMesh
(7) 监控告警 -> Fluent Operator + Prometheus & Grafana
(8) 虚拟机管理 -> KubeVirt
```