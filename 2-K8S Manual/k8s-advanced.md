# K8S Advanced

## Cluster Deploy
```
# 方式
kubeadm / kops / Kubespray
KubeKey
```

## 集群备份恢复
- Velero对集群资源和持久卷辈分恢复/迁移
- Restic是开源版，备份前卷需要被注释，备份pv不支持hostpath卷类型
- velero-plugin-for-aws支持兼容S3
- Velero支持后端存储的CRD:
  - BackupStorageLocation(集群对象数据非PVC，兼容S3如Minio)
  - VolumeSnapshotLocation(PV快照，需要CSI存储机制)
- Minio开源对象存储服务，兼容S3
- 操作
  - 配置对象存储Minio
  - 集群AB上部署Velero
  - 部署验证服务(wordpress)
  - 备份KubeSphere集群A
  - 集群B还原KubeSphere
  - 验证服务

## 启停集群
- Master
  - 备份数据
  - 停止Master节点调度
  - 驱逐Master节点工作负载
  - 停止Master节点
  - 恢复Master节点
  - 允许Master节点调度
- Worker
  - 停止Worker节点调度
  - 驱逐Worker节点工作负载
  - 停止Docker、Kubelet等服务
  - 恢复Worker节点
  - 允许orker节点调度

## 启停类型、场景
- 集群启停
  - 紧急情况->防止异常
- 多集群(联邦集群)启停
  - 特殊情况->防止异常
- 操作
  - 备份数据
  - 停止所有结点
  - 恢复所有节点
  - 检查集群状态
  - 常见问题排查
    - etcd集群启动失败
    - 部分节点加入集群失败
    - 部分pod不断重启

## 应用调度类型与场景
- nodeSelector
  - 设置label相关策略将pod关联到对应label节点上
- nodeName定向调度
  - 指定nodeName调度器优先在指定node上运行pod
- nodeAffinity定向调度
  - 软亲和性、硬亲和性
    - 语言表现力
    - 调度器无法满足要求仍然调度该pod
- 操作
  - worker节点加label
  - nodeSeelctor调度应用
  - nodeName调度应用
  - 使用Affinity调度应用
  - 无法调度异常排查

## K8S Federation多集群
- 集群联邦：高可用，避免厂商锁定，故障隔离
- 逻辑组成
  - configuration
    - cluster configuration
    - type configuration
  - propagtion
- 新增托管资源
  - Federated Resource CRD
    - Templates/Placement/Overrides
- 集群连接(Host和Member)
  - 直接 / 代理

## Spring Cloud K8S
- 客户端服务发现
  - spring cloud k8s fabric
- 配置管理
  - spring cloud k8s fabric config
- 负载均衡
  - spring cloud k8s fabric loadbalance
- 选主
  - spring cloud k8s fabric leader
kubectl get services
kubectl get endpoint