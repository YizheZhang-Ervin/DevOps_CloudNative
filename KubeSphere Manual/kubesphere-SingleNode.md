# KubeSphere SingleNode Deployment

# 1. 开通服务器
```
hostnamectl set-hostname node1
```

# 2. 安装
## 2.1 准备kubeKey
```
export KKZONE=cn
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.1.1 sh -
chmod +x kk
```

## 2.2 使用kubeKey引导安装集群
```
# 可能需要下面命令
yum install -y conntrack
./kk create cluster --with-kubernetes v1.20.4 --with-kubesphere v3.1.1
```

# 3.安装后开启功能