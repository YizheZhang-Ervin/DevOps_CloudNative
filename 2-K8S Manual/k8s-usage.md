# K8S Usage

# 1.资源创建方式
- 命令行/YAML

# 2.Namespace
```
# 隔离资源，不隔离网络
kubectl get ns

# 创/删(会连带删资源)
kubectl create ns 命名空间名
kubectl delete ns 命名空间名
```

# 3.Pod
```
# 创建(用命令/用yaml)
kubectl run pod名字 --image=镜像名
kubectl apply -f xxx.yaml

# 查看default名称空间的Pod
kubectl get pod

# 查看所有ns的pod
kubectl get pod -A

# 描述
kubectl describe pod Pod名字

# 删除(用命令/用yaml)
kubectl delete pod Pod名字
kubectl delete -f xx.yaml

# 查看Pod的运行日志
kubectl logs Pod名字

# 查看详细信息 - 每个Podk8s都会分配一个ip
kubectl get pod -owide
# 使用Pod的ip+pod里面运行容器的端口
curl 192.168.169.136

# 集群中的任意一个机器以及任意的应用都能通过Pod分配的ip来访问这个Pod

# pod内部多个image互相访问
用127.0.0.1加端口号

# 进控制台
kubectl exec -it Pod名字 -- /bin/bash

```

# 4.Deployment
```
# 普通方式运行pod
kubectl run mynginx --image=nginx
```

## 4.1控制pod多副本
```
kubectl create deployment 部署名 --image=镜像名 --replicas=3
```

## 4.2扩缩容
```
kubectl scale --replicas=5 deployment/部署名
# 修改 replicas
kubectl edit deployment 部署名
```

## 4.3自愈&故障转移(保证副本数量)
```
# 打印pod状态
kubectl get pod w
```

- 停机
- 删除Pod
- 容器崩溃

## 4.4滚动更新(不停机维护)
```
# 记录版本的更新
kubectl set image deployment/部署名 nginx=nginx:1.16.1 --record
kubectl rollout status deployment/部署名
```

## 4.5版本回退
```
# 历史记录
kubectl rollout history deployment/部署名

# 查看某个历史详情
kubectl rollout history deployment/部署名 --revision=2

# 回滚(回到上次)
kubectl rollout undo deployment/部署名

# 回滚(回到指定版本)
kubectl rollout undo deployment/部署名 --to-revision=2

# 查看部署的镜像
kubectl get deploy/部署名 -oyaml | grep image
```

- 工作负载
  - Deployment无状态
    - 微服务，多副本
  - StatefulSet有状态副本集
    - redis，稳定存储网络
  - DaemonSet守护进程集
    - 日志收集组件，各机器一份
  - Job/CronJob任务及定时任务
    - 垃圾清理组件，指定时间运行

# 5.Service
- pod服务发现与负载均衡
- 将一组pod公开为网络服务的抽象方法
```
# 暴露Deploy(pod端口是targetport，暴露端口是port)
kubectl expose deployment 部署名 --port=8000 --target-port=80

# 使用标签检索Pod
kubectl get pod -l app=部署名

# 集群内访问
curl 部署名.命名空间名.svc:端口
curl ip:端口
```

## 5.1 ClusterIP(默认:集群内)
```
# 等同于没有--type的
kubectl expose deployment 部署名 --port=8000 --target-port=80 --type=ClusterIP
```

## 5.2 NodePort(公网访问)
```
# 查看服务(可见每台机器的暴露端口)
kubectl get svc

# NodePort在30000-32767之间，公网:NodePort
kubectl expose deployment 部署名 --port=8000 --target-port=80 --type=NodePort
```

# 6.Ingress
- service的统一网关入口(service层网络)
## 6.1 安装
```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml

# 修改镜像
vi deploy.yaml

# yaml中修改image的值
registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ingress-nginx-controller:v0.46.0

# 使用yaml
kubectl apply -f deploy.yaml

# 检查安装的结果
kubectl get pod,svc -n ingress-nginx

# 把svc暴露的端口放行
kubectl get svc -A
```

## 6.2 使用
```
# 测试环境
test_yaml/test.yaml
kubectl apply -f test.yaml

# 多个规则可同时生效
# 规则1:域名方问
test_yaml/visit.yaml
kubectl apply -f visit.yaml
kubectl get ingress

# 规则2:路径重写
test_yaml/path.yaml
kubectl apply -f path.yaml
kubectl get ingress

# 规则3:流量限制
test_yaml/volume.yaml
kubectl apply -f volume.yaml
kubectl get ingress
```

# 7.存储抽象
- 存储层：Glusterfs/NFS/CephFS
- NFS Server变NFS Client跟着变
## 7.1环境准备
```
# (1)所有节点 - 所有机器安装
yum install -y nfs-utils

# (2)nfs主节点
# 暴露文件夹
echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports
mkdir -p /nfs/data

# 启动
systemctl enable rpcbind --now
systemctl enable nfs-server --now

# 配置生效
exportfs -r

# (3)从节点
# 私有网络ip地址
ip a
showmount -e 172.31.0.4

# 执行以下命令挂载 nfs 服务器上的共享目录到本机路径 /root/nfsmount
mkdir -p /nfs/data
mount -t nfs 172.31.0.4:/nfs/data /nfs/data

# 主/从节点写入测试数据
echo "hello nfs server" > /nfs/data/test.txt

# (4)原生方式数据挂载
test_yaml/store.yaml
```

## 7.2 PV&PVC
- 持久卷：应用要持久化的数据存到指定位置
- 持久卷申明：申明需要使用的持久卷规格
- PV池：静态供应

```
# 创建pv池 - nfs主节点
mkdir -p /nfs/data/01
mkdir -p /nfs/data/02
mkdir -p /nfs/data/03
test_yaml/create_pv.yaml
kubectl apply -f create_pv.yaml
kubectl get persistentvolume

# PVC创建
test_yaml/create_pvc.yaml
kubectl apply -f create_pvc.yaml
kubectl get pv
kubectl get pvc

# 创pod绑pvc
test_yaml/bind_pvc.yaml
kubectl apply -f bind_pvc.yaml
kubectl get pod,pv,pvc
```

## 7.3 ConfigMap - redis示例
- 配置文件挂载
```
# 配置文件创建为配置集 - 创建配置，redis保存到k8s的etcd
kubectl create cm redis-conf --from-file=redis.conf
或用test_yaml/redis_configmap.yaml

# 创建pod
test_yaml/redis_pod.yaml
kubectl apply -f redis_pod.yaml
kubectl get cm

# 修改ConfigMap
kubectl edit cm redis-conf

# 检查配置是否更新(文件已改，但不生效，因为redis没有热更新能力)
kubectl exec -it redis -- redis-cli
127.0.0.1:6379> CONFIG GET appendonly
```

## 7.4 Secret
- 存密码/OAuth令牌/SSH密钥

```
# 创建、查看secret
kubectl create secret docker-registry xx \
  --docker-server=<你的镜像仓库服务器> \
  --docker-username=<你的用户名> \
  --docker-password=<你的密码> \
  --docker-email=<你的邮箱地址>

kubectl get secret xx -oyaml

# 使用
test_yaml/redis_secret.yaml
kubectl apply -f redis_secret.yaml
```