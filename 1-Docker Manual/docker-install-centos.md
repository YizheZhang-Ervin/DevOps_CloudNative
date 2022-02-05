# Docker Centos Install
## 安装
```
# 卸载旧docker
sudo yum remove docker docker-client docker-client-latest \
                docker-common docker-latest docker-latest-logrotate \
                docker-logrotate docker-engine

# 配置yum源
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装dokcer
sudo yum install -y docker-ce docker-ce-cli containerd.io
##以下是在安装k8s的时候使用
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6

# 启动
systemctl enable docker --now

# 查看docker的进程
docker ps

# 配置加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```