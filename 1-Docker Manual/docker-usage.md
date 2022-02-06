# Docker Image

## 镜像

### 1. 登录/登出docker hub(推送完后退出)
```
docker login       
docker logout
```

### 2. 拉取镜像(镜像名:版本名(标签)，无标签默认lastest)
```
docker pull nginx
docker pull 仓库名/镜像名:v1.0
```

### 3. 增
```
# 文件：Dockerfile
见Dockerfile，其中copy复制复制到容器根目录

# 打包成镜像 (-f Dockerfile的名字，Dockerfile和target同级)
docker build -t java-demo:v1.0 .

# 打包镜像成压缩包
docker save -o 压缩包名.tar 镜像名:v1.0

# 打标签 (把旧镜像的名字，改成仓库要求的新版名字)
docker tag 本地镜像名:v1.0 远程仓库名/远程镜像名:v1.0

# 推送
docker push 仓库名/镜像名:v1.0
```

### 4. 删
```
docker rmi 镜像名:版本号/镜像id
```

### 5. 改
```
# 从容器创建一个新镜像
docker commit -a "作者" -m "变化描述" 容器ID 镜像名:v1.0
```

### 6. 查
```
# 查看所有镜像
docker images  
docker search 镜像名

# 加载这个镜像
docker load -i 压缩包名.tar
```

## 容器

### 1. 启动/停止
```
# 启动 -d后台，--restart=always: 开机自启，最后可跟镜像启动运行命令
docker run --name=容器名 -d --restart=always -p 88:80 nginx

# 再次启动
docker start 容器id/名字

# 重启
docker restart 容器id/名字

# 停止容器
docker stop 容器id/名字

# 删除容器
docker kill -s KILL 容器名
```

### 2. 排错 (-f跟踪)
```
docker logs 容器名/id
```

## 3. 增
```
docker create --name 容器名 镜像名
```

### 3. 删
```
# 删除停止的容器 (-f 强制删除正在运行中的)
docker rm -f 容器id/名字
```

### 4. 查
```
# 查看正在运行的容器 (-a包含所有)
docker ps -a
```

### 5. 改
```
# 更新容器参数->应用开机自启
docker update 容器id/名字 --restart=always

# 改内容-方法1：进入容器内部修改容器内容
docker exec -it 容器id  /bin/bash
exit

# 改内容-方法2：挂载数据到外部修改 (:ro为只读权限[容器里不能改]，-v主机目录:容器目录)
docker run --name=容器名 -d --restart=always -p 主机端口:容器端口 \
            -v /data/conf/nginx.conf:/etc/nginx/nginx.conf:ro nginx

# 把容器指定位置的东西复制出来 
docker cp 容器ID:/etc/nginx/nginx.conf /data/conf/nginx.conf

# 把外面的内容复制到容器里面
docker cp  /data/conf/nginx.conf 容器ID:/etc/nginx/nginx.conf
```

## 部署中间件
```
# redis使用自定义配置文件(要写容器内配置文件位置)启动
docker run -v /data/redis/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data \
-d --name myredis \
-p 6379:6379 \
redis:latest redis-server /etc/redis/redis.conf
```
