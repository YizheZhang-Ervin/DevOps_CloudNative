# K8S Kubeadm 安装&初始化 - ubuntu

# 1. linux配置&kubeadm等的安装
## 1.1 基础环境
```
# 主机名
hostnamectl set-hostname xxxx

# 主节点地址
echo "192.168.137.225 k8s-master" >> /etc/hosts

# 关防火墙
sudo ufw disable
sudo ufw status

# 允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

# 关闭swap
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 改rp_filter
sudo vi /etc/sysctl.d/10-network-security.conf
改net.ipv4.conf.default.rp_filter=1
改net.ipv4.conf.all.rp_filter=1
sudo sysctl --system

# 设置docker镜像仓库
 /etc/docker/daemon.json中添加
{
  "registry-mirrors": ["https://9cpn8tt6.mirror.aliyuncs.com"]
}
systemctl daemon-reload
systemctl restart docker
```

## 1.2 安装kubelet、kubeadm、kubectl
```
# 安装k8s
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF 
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet 
```

# 2. 使用kubeadm引导集群
## 2.1 下载各个机器需要的k8s镜像
```
# 方法1
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

# 方法2
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
for  i  in  `kubeadm config images list`;  do
    imageName=${i#k8s.gcr.io/}
    docker pull registry.aliyuncs.com/google_containers/$imageName
    docker tag registry.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.aliyuncs.com/google_containers/$imageName
done
EOF

chmod +x ./images.sh && ./images.sh
```

## 2.2 初始化主节点 & 设置config
```
sudo kubeadm init --pod-network-cidr 172.16.0.0/16 \
    --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 2.3 安装网络组件
```
wget https://docs.projectcalico.org/manifests/calico.yaml
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

## 2.5 加入node节点(待验证)
```
# 加入节点(kubeadm init时产生的编号，在主节点执行)
kubeadm join cluster-endpoint:6443 --token xx.xx \
	--discovery-token-ca-cert-hash sha256:xxx

# 新令牌(令牌过期，则在主节点执行生成新的)
kubeadm token create --print-join-command
```

## 2.6 单节点集群(待验证)
```
kubectl get node -o yaml | grep taint -A 5
kubectl taint nodes --all node-role.kubernetes.io/master-
```

# 3.卸载
```
kubeadm reset -f
modprobe -r ipip
lsmod
rm -rf ~/.kube/
rm -rf /etc/kubernetes/
rm -rf /etc/systemd/system/kubelet.service.d
rm -rf /etc/systemd/system/kubelet.service
rm -rf /usr/bin/kube*
rm -rf /etc/cni
rm -rf /opt/cni
rm -rf /var/lib/etcd
rm -rf /var/etcd
sudo apt-get --purge remove kubeadm
sudo apt-get --purge remove kubectl
sudo apt-get --purge remove kubelet
```