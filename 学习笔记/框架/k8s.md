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
minikube start --vm-driver=none --image-repository=registry.aliyuncs.com/google_containers
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

## 二、k8s学习

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

## 3. service机制



## 三、k8s命令

### 1. k8s的常用命令

```shell
# 运行一个指定的镜像，--expose会自动创建一个service，并暴露内部的9001端口，创建的pod为service1
1. kubectl run service1 --image=service1:v1.0 --expose --port=9001

# 运行yaml文件，并创建yaml文件中配置的组件
2. kubectl create -f service.yaml

# 更新yaml配置
3. kubectl apply -f *.yaml

# 查看组件中内容
4. kubectl get deploy | pods | services  

```

### 2. 使用yaml文件创建service，deployment，pod

```shell
// 1.使用service.yaml文件创建两个service

# 创建 service1
apiVersion: v1
kind: Service
metadata:
  name: service1-service
  labels:
    app: service1-service
spec:
  type: NodePort   # NodePort可以使集群外部用户访问
  ports:
    - port: 9001   # 暴露给集群内部访问的端口
      targetPort: 9001   # pod内部监听的端口
      nodePort：10001  # 暴露给集群外部访问的端口
  selector:
    app: service1-pod

---

# 创建 service2
apiVersion: v1
kind: Service
metadata:
  name: service2
  labels:
    app: service2-service
spec:
  ports:
    - port: 9002
      targetPort: 9002
  selector:
    app: service2-pod

---

# 创建deployment: service1-deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service1-deploy
  namespace: default
  labels:
    app: service1-deploy
    version: v1.0
spec:
  replicas: 3  # 创建3个pod副本
  selector:
    matchLabels: 
      app: service1-pod
  template:
    metadata: 
      labels:
        app: service1-pod
    spec:
      containers:
        - name: service1-pod
          image: service1:v1.0
          ports:
            - containerPort: 9001

---

# 创建deployment: service2-deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service2-deploy
  namespace: default
  labels:
    app: service2-deploy
    version: v1.0
spec:
  replicas: 3
  selector:
    matchLabels: 
      app: service2-pod
  template:
    metadata: 
      labels:
        app: service2-pod
    spec:
      containers:
        - name: service2-pod
          image: service2:v1.0
          ports:
            - containerPort: 9002

# 创建pod: service1
#apiVersion: v1
#kind: Pod
#metadata:
#  name: service1-pod
#  labels:
#    app: service1
#spec:
#  containers:
#    - name: service1-container
#      image: service1:v1.0
#      ports:
#        - containerPort: 9001

---

# 创建pod: service2
#apiVersion: v1
#kind: Pod
#metadata:
#  name: service2-pod
#  labels:
#    app: service2
#spec:
#  containers:
#    - name: service2-container
#      image: service2:v1.0
#      ports:
#        - containerPort: 9002
```

# istio

## 一、istio的安装

## 二、istio ingress-gateway

### 1. 创建gateway.yaml

```shell
# 创建网关，统一流量入口
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: service-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:          # 接收url中header的hosts为匹配所有
        - "*"  
```

### 2. 创建virtualService.yaml

```shell
# 该配置为gateway的路由规则
# 创建virtualService
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: service-vrsvc
spec:
  hosts:
    - "*"
  gateways:
    - service-gateway
  http:
    - match:
        - uri:
            prefix: /service1/hello
      route:
        - destination:
            port:
              number: 9001
            host: service1-service
    - match:
        - uri:
            prefix: /service2/hello
      route:
        - destination:
            port:
              number: 9002
            host: service2-service 
```

