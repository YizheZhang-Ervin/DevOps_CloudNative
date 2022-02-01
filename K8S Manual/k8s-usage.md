# K8S Usage

# 1.资源创建方式
- 命令行/YAML

# 2.Namespace
```
kubectl create ns hello
kubectl delete ns hello
```

# 3.Pod
```
kubectl run mynginx --image=nginx

# 查看default名称空间的Pod
kubectl get pod 
# 描述
kubectl describe pod 你自己的Pod名字
# 删除
kubectl delete pod Pod名字
# 查看Pod的运行日志
kubectl logs Pod名字

# 每个Pod - k8s都会分配一个ip
kubectl get pod -owide
# 使用Pod的ip+pod里面运行容器的端口
curl 192.168.169.136

# 集群中的任意一个机器以及任意的应用都能通过Pod分配的ip来访问这个Pod

```

# 4.Deployment
```
# 清除所有Pod，比较下面两个命令有何不同效果？
kubectl run mynginx --image=nginx
kubectl create deployment mytomcat --image=tomcat:8.5.68
```

## 4.1多副本
```
kubectl create deployment my-dep --image=nginx --replicas=3
```

## 4.2扩缩容
```
kubectl scale --replicas=5 deployment/my-dep
# 修改 replicas
kubectl edit deployment my-dep
```

## 4.3自愈&故障转移
- 停机
- 删除Pod
- 容器崩溃

## 4.4滚动更新
```
kubectl set image deployment/my-dep nginx=nginx:1.16.1 --record
kubectl rollout status deployment/my-dep
```

## 4.5版本回退
```
#历史记录
kubectl rollout history deployment/my-dep

#查看某个历史详情
kubectl rollout history deployment/my-dep --revision=2

#回滚(回到上次)
kubectl rollout undo deployment/my-dep

#回滚(回到指定版本)
kubectl rollout undo deployment/my-dep --to-revision=2
```

# 5.Service
```
#暴露Deploy
kubectl expose deployment my-dep --port=8000 --target-port=80

#使用标签检索Pod
kubectl get pod -l app=my-dep
```

## 5.1 ClusterIP
```
# 等同于没有--type的
kubectl expose deployment my-dep --port=8000 --target-port=80 --type=ClusterIP
```

## 5.2 NodePort
```
kubectl expose deployment my-dep --port=8000 --target-port=80 --type=NodePort
```

# 6.Ingress
## 6.1 安装
```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml

#修改镜像
vi deploy.yaml
#将image的值改为如下值：
registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ingress-nginx-controller:v0.46.0

# 检查安装的结果
kubectl get pod,svc -n ingress-nginx

# 最后别忘记把svc暴露的端口要放行
```

## 6.2 使用
```
# 测试环境
# 域名方问
# 路径重写
# 流量限制

```

# 7.存储抽象
## 7.1环境准备
```
# (1)所有节点 - 所有机器安装
yum install -y nfs-utils

# (2)主节点 - nfs主节点
echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports
mkdir -p /nfs/data
systemctl enable rpcbind --now
systemctl enable nfs-server --now
exportfs -r  # 配置生效

# (3)从节点
showmount -e 172.31.0.4
#执行以下命令挂载 nfs 服务器上的共享目录到本机路径 /root/nfsmount
mkdir -p /nfs/data
mount -t nfs 172.31.0.4:/nfs/data /nfs/data
# 写入一个测试文件
echo "hello nfs server" > /nfs/data/test.txt

# (4)原生方式数据挂载
```

## 7.2 PV&PVC
```
# 创建pv池 - nfs主节点
mkdir -p /nfs/data/01
mkdir -p /nfs/data/02
mkdir -p /nfs/data/03

# PVC创建与绑定
```

## 7.3 ConfigMap - redis示例
```
# 配置文件创建为配置集 - 创建配置，redis保存到k8s的etcd；
kubectl create cm redis-conf --from-file=redis.conf

# 创建pod

# 检查默认配置
kubectl exec -it redis -- redis-cli
127.0.0.1:6379> CONFIG GET appendonly
127.0.0.1:6379> CONFIG GET requirepass

# 修改ConfigMap

# 检查配置是否更新
kubectl exec -it redis -- redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
127.0.0.1:6379> CONFIG GET maxmemory-policy

```

## 7.4 Secret
```
# 命令格式
kubectl create secret docker-registry regcred \
  --docker-server=<你的镜像仓库服务器> \
  --docker-username=<你的用户名> \
  --docker-password=<你的密码> \
  --docker-email=<你的邮箱地址>
```