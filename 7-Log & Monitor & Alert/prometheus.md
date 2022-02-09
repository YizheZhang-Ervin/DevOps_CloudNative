# Prometheus

## PromQL
- 概念
  - 瞬时向量
    - http_requests_total
  - 区间向量
    - rate/irate
  - 标量
    - 简单Counter/Gauge
    - 复杂Histogram/Summary
  - 字符串

## 告警处理架构
- prometheus->alertManager(基本路由)->Mail/Slack/WebHook(子路由)
- 配置告警规则

## Prometheus Operator架构
- Prometheus
- Prometheus Server
- AlertManager
- ServiceMonitor
- Operator

## Kubesphere 监控
- 基于Prometheus生态
- 多租户隔离
- 多维度监控
- 全面丰富指标
- 灵活多样展现方式

### 集群状态监控
- 物理资源监控
  - 集群资源
  - 节点资源
- k8s核心组件监控
  - API Server
  - Etcd
  - Scheduler
- KubeSphere核心组件监控

### 应用资源监控
- 管理员
  - 集群层级
    - 项目与应用资源统计
    - 用量排行
- 普通用户
  - 企业空间层级
  - 项目层级
  - 工作负载层级
    - 容器组层级
    - 容器层级

### 告警功能
- 兼容Prometheus规则
- 多租户支持
- 内置平台告警策略
- 规则配置方式

#### 集群告警
- 内置平台告警策略
  - 物理资源(cpu/内存/存储)
  - 核心组件(k8s/etcd)
- 规则模板配置策略
  - 节点(cpu/内存/磁盘/网络/容器组利用率)
- 自定义规则配置策略

#### 应用告警
- 规则模板配置策略(cpu/内存/网络/副本不可用)
  - 部署
  - 有状态副本集
  - 守护进程集
- 自定义规则配置策略

### 自定义监控
- 数据模型Grafana
- 转换Grafana link->Dashboard content0>SDK->apis->dashboard CR