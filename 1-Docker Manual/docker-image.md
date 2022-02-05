# Docker Image
## 镜像拉取、查看、移除
```
# 拉取镜像(镜像名:版本名(标签)，无标签默认lastest)
docker pull nginx
docker pull nginx:1.20.1

# 查看本地镜像
docker images  #查看所有镜像

# 移除镜像
docker rmi 镜像名:版本号/镜像id
```

## 容器启动、查看、删除、停止、重启、更新、排错
```
# 启动 -d后台，--restart=always: 开机自启，最后可跟镜像启动运行命令
docker run --name=容器名 -d --restart=always -p 88:80 nginx

# 查看正在运行的容器(-a包含所有)
docker ps -a

# 删除停止的容器(-f 强制删除正在运行中的)
docker rm -f 容器id/名字

# 停止容器
docker stop 容器id/名字

# 再次启动
docker start 容器id/名字

# 重启
docker restart 容器id/名字

# 更新容器参数->应用开机自启
docker update 容器id/名字 --restart=always

# 排错(-f跟踪)
docker logs 容器名/id
```

## 修改容器内容
```
# 进入容器内部修改容器内容
docker exec -it 容器id  /bin/bash

# 挂载数据到外部修改(:ro为只读权限[容器里不能改]，-v主机目录:容器目录)
docker run --name=容器名 -d --restart=always -p 主机端口:容器端口 \
            -v /data/conf/nginx.conf:/etc/nginx/nginx.conf:ro nginx

# 把容器指定位置的东西复制出来 
docker cp 容器ID:/etc/nginx/nginx.conf /data/conf/nginx.conf

# 把外面的内容复制到容器里面
docker cp  /data/conf/nginx.conf 容器ID:/etc/nginx/nginx.conf
```

## 镜像提交、压缩、加载
```
# 提交镜像
docker commit -a "作者" -m "变化描述" 容器ID 镜像名:v1.0

# 将镜像保存成压缩包
docker save -o 压缩包名.tar 镜像名:v1.0

# 加载这个镜像
docker load -i 压缩包名.tar
```

## 镜像改名、推送、下载
```
# 把旧镜像的名字，改成仓库要求的新版名字
docker tag 本地镜像名:v1.0 远程仓库名/远程镜像名:v1.0

# 登录/登出docker hub(推送完后退出)
docker login       
docker logout

# 推送
docker push 仓库名/镜像名:v1.0

# 下载
docker pull 仓库名/镜像名:v1.0
```

## 镜像打包
```
# 文件：Dockerfile
见Dockerfile，其中copy复制复制到容器根目录
# 打包(-f Dockerfile的名字，Dockerfile和target同级)
docker build -t java-demo:v1.0 .
```

## 部署中间件
```
#redis使用自定义配置文件(要写容器内配置文件位置)启动
docker run -v /data/redis/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data \
-d --name myredis \
-p 6379:6379 \
redis:latest redis-server /etc/redis/redis.conf
```
