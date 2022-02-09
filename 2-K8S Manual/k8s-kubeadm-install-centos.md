# K8S Kubeadm 安装&初始化 - centos
- 基本要求：设置防火墙放行规则/设置不同hostname/内网互信/永久关闭

# 1. linux配置&kubeadm等的安装
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
# 设置repo
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

# 下载安装
sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes

# 启动
sudo systemctl enable --now kubelet
```

# 2. 使用kubeadm引导集群
## 2.1下载各个机器需要的k8s镜像
```
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.23.3
kube-proxy:v1.23.3
kube-controller-manager:v1.23.3
kube-scheduler:v1.23.3
pause:3.6
etcd:3.5.1-0
coredns:1.8.6
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
EOF
   
chmod +x ./images.sh && ./images.sh
```

## 2.2 初始化主节点 & 设置config
```
# 所有机器添加master域名映射(告诉每个机器master位置)
echo "172.31.0.4  cluster-endpoint" >> /etc/hosts

# 主节点初始化(service-cidr和pod-network-cidr所有网络范围不重叠)
kubeadm init \
--apiserver-advertise-address=172.31.0.4 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16

# 查看集群所有节点
kubectl get nodes

# 查看集群部署了哪些应用(运行中的应用在docker里面叫容器，在k8s里面叫Pod)
docker ps
watch -n 1 kubectl get pod -A

# 设置.kube/config
mkdir -p $HOME/.kube
sudo cp /etc/kubenetes/admin.conf $HOME/.kube/config
sudo chown ${id -u}:${id -g} $HOME/.kube/config
```

## 2.3 安装网络组件
```
# 安装网络组件
curl https://docs.projectcalico.org/manifests/calico.yaml -O

# 根据配置文件，给集群创建资源
kubectl apply -f calico.yaml
```

## 2.4 查看集群信息
```
# 查看节点
kubectl get nodes

# 查看pods
kubectl get pods -A

# 查看错误日志
journalctl -u -f
```

## 2.5 加入node节点
```
# 加入节点(kubeadm init时产生的编号，在主节点执行)
kubeadm join cluster-endpoint:6443 --token x5g4uy.wpjjdbgra92s25pp \
	--discovery-token-ca-cert-hash sha256:6255797916eaee52bf9dda9429db616fcd828436708345a308f4b917d3457a22

# 新令牌(令牌过期，则在主节点执行生成新的)
kubeadm token create --print-join-command
```

# 3.安装补全命令
```
yum -y install bash-completion  #安装补全命令的包
kubectl completion bash
source /usr/share/bash-completion/bash_completion
kubectl completion bash >/etc/profile.d/kubectl.sh
source /etc/profile.d/kubectl.sh
cat  >>  /root/.bashrc <<EOF
source /etc/profile.d/kubectl.sh
EOF
```