# Docker

## 1. 镜像命令

```shell
1. docker images          # 显示所有镜像文件
2. docker rmi imageName/imageId  # 删除执行的镜像文件
```

## 2. 容器命令

```shell
1. docker ps      # 显示所有正在运行的容器
2. docker ps -a   # 显示所有的容器
3. docker stop/start/rm containerId   # 启动，停止，删除容器
4. docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)       # 查看所有容器的ip地址
5. docker exec -it fc7de5683f03(容器ID) /bin/bash   # 进入指定的容器ID
```

## 3. 应用部署

```shell
1. docker build -t imageName:version .           # 从dockerfile文件中构建镜像
2. docker run -dit -p 80:80 --name containName imageName # -d表示后台运行，-i表示交互式 -t表示开启终端 启动一个镜像，容器的名字为containName
3. docker exec -it containerId /bin/bash   # 进入一个正在运行的容器，并且退出之后，容器不会退出
```

```txt
# 基础镜像
FROM java:8

# 定义变量
ENV NAME=edu-eureka.jar DIR=eureka

# 复制上下文路径下的jar包到容器的指定路径中
COPY $NAME /usr/src/edu-service/$DIR/$NAME

# 指定工作目录
WORKDIR /usr/src/edu-service/$DIR

# 启动容器时执行的命令，会被命令行参数覆盖
ENTRYPOINT ["sh","-c","java -jar $NAME"]
```

## 4. 网络命令

```shell
1. docker network ls        # 查看docker中的所有网络
2. docker network create --subnet=172.18.0.0/16 my-network    # 创建my-network网络，子网范172.18.0.0/16
3. docker network rm networkID   # 删除指定的网络ID
```



# Docker-Compose.yml

## 1. 文件内容

```shell
version : '3'
services:
        eureka:
                image: eureka:v1.0
                container_name: eureka
                networks:
                        default:
                                ipv4_address: 172.20.0.10
                ports:
                        - "7001:7001"
                restart: always
        config-center:
                image: config-center:v1.0
                container_name: config-center
                hostname: config-center
                networks:               
                        default:
                                ipv4_address: 172.20.0.11
                depends_on:
                        - eureka
                ports:
                        - "3344:3344"
                restart: always
        authentication:
                image: authentication:v1.0
                container_name: authentication
                entrypoint: "bash /usr/local/bin/wait.sh config-center:3344 -- java -jar edu-authentication.jar "
                networks:       
                        default:
                                ipv4_address: 172.20.0.12
                depends_on:
                        - eureka
                        - config-center
                ports:
                        - "9401:9401"
                restart: always
networks:
        default:
                driver: bridge
                ipam:
                        config:
                                - subnet: 172.20.0.0/16

```

## 2. 命令详解

```shell
1. docker-compose up -d    # 后台启动compose中配置的镜像
```

## 3. 配置wait-for-it.sh实现容器的顺序启动

```shell
1. 脚本地址：https://github.com/vishnubob/wait-for-it
2. wait-for-it.sh  www.baidu.com:80 -- echo success # 当可以成功访问百度的80端口时，执行子命令，将-s参数时，当没有访问成功或访问超时的时候，不会执行后面的子命令
3. 将该脚本放到容器的目录下，之后在微服务启动时，先判断依赖的服务是否启动在去执行jar包
```



