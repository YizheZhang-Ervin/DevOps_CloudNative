# Helm

## 定义
- K8S的包管理器
- 管理Charts
- Helm Charts封装K8S应用程序的一系列YAML文件

## 应用仓库Harbor
- 管理和分发Helm Charts
```
# 安装
helm repo add harbor https://helm.goharbor.io
helm fetch harbor/harbor
改values.yaml
helm install xxharbor harbor -n harbor
helm list -n harbor

# 使用
helm create xxchart
# chart.yaml,values.yaml,templates/,charts/
helm template xxchart
helm install xx xxchart/ -n default
helm package xxchart
```

## 应用生命周砌
- 开发中
- 待发布
- 已审核通过
- 审核未通过
- 已上架
- 已下架

## 应用仓库管理
- 添加应用仓库
- 删除应用仓库
- 手动更新应用仓库
- 应用仓库自动同步远程仓库