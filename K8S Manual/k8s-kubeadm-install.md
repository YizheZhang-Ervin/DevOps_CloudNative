# K8S Kubeadm 创建集群
- 基本要求：设置防火墙放行规则/设置不同hostname/内网互信/永久关闭

# 1. 安装kubeadm
## 1.1 基础环境
```
# (1)各个机器设置自己的域名
hostnamectl set-hostname xxxx

# (2)将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# (3)关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

# (4)允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

## 1.2 安装kubelet、kubeadm、kubectl
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

# 2. 使用kubeadm引导集群
## 2.1下载各个机器需要的k8s镜像
```
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF
   
chmod +x ./images.sh && ./images.sh
```

## 2.2初始化主节点
```
# 所有机器添加master域名映射，以下需要修改为自己的
echo "172.31.0.4  cluster-endpoint" >> /etc/hosts

# 主节点初始化(所有网络范围不重叠)
kubeadm init \
--apiserver-advertise-address=172.31.0.4 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16
# 查看集群所有节点
kubectl get nodes

# 根据配置文件，给集群创建资源
kubectl apply -f xxxx.yaml

# 查看集群部署了哪些应用？
docker ps   ===   kubectl get pods -A
# 运行中的应用在docker里面叫容器，在k8s里面叫Pod
kubectl get pods -A
```

## 2.3 设置config & 安装网络组件
```
# 设置.kube/config

# 安装网络组件
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

## 2.4 加入node节点
```
# 加入节点
kubeadm join cluster-endpoint:6443 --token x5g4uy.wpjjdbgra92s25pp \
	--discovery-token-ca-cert-hash sha256:6255797916eaee52bf9dda9429db616fcd828436708345a308f4b917d3457a22

# 新令牌
kubeadm token create --print-join-command
```

## 2.5 验证集群
```
# 验证集群节点状态
kubectl get nodes
```

## 2.6 部署dashboard
```
# (1)可视化界面
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

# (2)设置访问端口
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
kubectl get svc -A |grep kubernetes-dashboard
# 找到端口，在安全组放行
# 访问： https://集群任意IP:端口

# (3)创建访问账号
#创建访问账号，准备一个yaml文件； vi dash.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

kubectl apply -f dash.yaml

# (4)令牌访问
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```