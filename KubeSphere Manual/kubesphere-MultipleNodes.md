# KubeSphere MultipleNodes Deployment

# 1.使用kubekey创建集群
## 1.1 下载kubekey
```
export KKZONE=cn
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.1.1 sh -
chmod +x kk
```

## 1.2 创建集群配置文件
```
./kk create config --with-kubernetes v1.20.4 --with-kubesphere v3.1.1
```

## 1.3 创建集群
```
yum install -y conntrack

configs/multiple_node.yaml
./kk create cluster -f multiple_node.yaml
```

## 1.4 查看进度
```
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```