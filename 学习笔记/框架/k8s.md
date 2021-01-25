# Kubernetes

## 一、Kubernetes安装(单机版，Centos7)

- 注：采用minikube安装k8s
- 对于需要翻墙下载的镜像，可以直接chorme浏览器下载好之后导入centos中

### 1. 安装kubectl

```shell
# 1. 先用浏览器访问：https://storage.googleapis.com/kubernetes-release/release/stable.txt得到一个kubectl的版本号
例如：v1.20.2

# 2. 直接在浏览器中下载：https://storage.googleapis.com/kubernetes-release/release/v1.20.2/bin/linux/amd64/kubectl

# 3. 复制kubectl到bin目录，并赋予可执行权限：cp kubectl /usr/local/bin/       &&      chmod +x /usr/local/bin/kubectl
```

### 2. 安装minikube

```shell
# 1. 从浏览器下载：https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# 2. install minikube-linux-amd64 /usr/local/bin/minikube  安装minikube到bin目录

# 3. minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com  这一步骤比较慢
```

### 3. 安装minikube dashboard

```shell
# 1. 直接运行 minikube dashboard 会给出一个url链接，该URL只能在集群内部访问

# 2. 添加dashboard可以外部访问，增加一个proxy地址
 kubectl proxy --port=9001 --address='192.168.4.123' --accept-hosts='^.*' &    # 该命令中port为外部访问的端口，可以随意指定一个没有占用的端口，address为centos虚拟机的IP地址
```

### 4. 测试minikube是否安装成功

```shell
# 1. kubectl create deployment tomcat-deployment --image=tomcat:8.0  创建一个名为tomcat-deployment的deployment，并下载tomcat:8.0并运行

# 2. kubectl expose deployment tomcat-deployment --type=NodePort --port=8080  随机暴露给外部一个访问端口，该端口将会映射到内部的8080端口，该命令也是创建了一个service供外部访问

# 3. http://192.168.4.123:port  可以看到tomcat的页面
```

## 二、istio的安装







## 三、k8s学习

## 1. pod

- pod分为自主性pod和控制器管理的pod

- Pod是一个或一个以上的 容器（例如Docker容器）组成的，且具有共享存储/网络/UTS/PID的能力，以及运行容器的规范。并且在Kubernetes中，Pod是最小的可被调度的原子单位。

  通俗来讲，Pod就是一组容器的集合，在Pod里面的容器共享网络/存储，所以它们可以通过Localhost进行内部的通信。虽然网络存储都是共享的，但是CPU和Memory就不是。多容器之间可以有属于自己的Cgroup，也就是说我们可以单独的对Pod中的容器做资源（MEM/CPU）使用的限制。

  Pod就像是我们的一个”专有主机”，上面除了运行我们的主应用程序之外，还可以运行一个与该应用紧密相关的进程。如日志收集工具、Git文件拉取器、配置文件更新重启器等。因为在Kubernetes中，一个Pod里的所有container都只会被分配到同一台主机上运行。

## 2. 管理Pod的控制器

- RC(RepliactionController)
- RS(ReplicaSet)
- Deployment(常用)
  - 支持滚动更新，回滚

## 四、k8s命令

```shell
# 运行一个指定的镜像，--expose会自动创建一个service，并暴露内部的9001端口，创建的pod为service1
1. kubectl run service1 --image=service1:v1.0 --expose --port=9001

```

