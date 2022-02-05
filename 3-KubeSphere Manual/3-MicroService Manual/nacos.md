# Nacos
## 1.配置
```
# 每个微服务准备 bootstrap.properties，配置 nacos地址信息。默认使用本地

# 每个微服务准备Dockerfile，启动命令，指定线上nacos配置等。

# 每个微服务制作自己镜像
```

## 2.文件 Dockerfile
```
# 容器默认以8080端口启动

# 时间为CST

# 环境变量 PARAMS 可以动态指定配置文件中任意的值

# nacos集群内地址为  his-nacos.his:8848 

# 微服务默认启动加载 nacos中   服务名-激活的环境.yml  文件，所以线上的配置可以全部写在nacos中
```

## 3.推送镜像
```
docker login --username=xx registry.cn-hangzhou.aliyuncs.com

# 本地镜像改名
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/lfy_ruoyi/镜像名:[镜像版本号]

docker push registry.cn-hangzhou.aliyuncs.com/lfy_ruoyi/镜像名:[镜像版本号]
```

## 4.部署规则
```
# 应用一启动会获取到 "应用名-激活的环境标识.yml"

# 每次部署应用的时候，需要提前修改nacos线上配置，确认好每个中间件的连接地址是否正确
```