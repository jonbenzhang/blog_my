# IP and PORT
## 一 IP
### PodIP
​        Pod中的Container启动时，会分配一个Pod网段中的IP地址（`network_cidr`），但是这个IP地址是浮动的（ephemeral），**Pod重启，就可能换一个新的IP**，所以，通常情况下，Pod之间不会用Pod的IP地址直接通信。K8S中使用Service来访问Pod。

​         **一个Pod中容器相互通信，只需要用localhost即可**。如果想用别的Pod的服务，就要知道别的Pod对应的Serice的DNS名称，解析到ClusterIP。 ClusterIP会被每个Node上的Kube Proxy修改iptables和ipvs等路由信息，这样保证发往ClusterIP的IP包，实际上被发往在这个Node上的Pod的IP地址和端口。注意：这里的ClusterIP是内部IP地址，不是Node公网地址，K8S外部不能访问。

### ClusterIP

 虚拟IP地址。**外部网络无法ping通，只有kubernetes集群内部访问使用.**一般为service使用。

1. Cluster IP仅仅作用于Kubernetes Service这个对象，并由Kubernetes管理和分配P地址
2. Cluster IP无法被ping，他没有一个“实体网络对象”来响应
3. Cluster IP只能结合Service Port组成一个具体的通信端口，单独的Cluster IP不具备通信的基础，并且他们属于Kubernetes集群这样一个封闭的空间。
4. 在不同Service下的pod节点在集群间相互访问可以通过Cluster IP

**Node IP**

Node节点的IP地址，即物理网卡的IP地址。

## 二 Port

### NodePort

NodePort是在ClusterIP之上建造的Service。
如果想让外网访问Pod的业务，最简单的办法就是通过Node的公网地址，生成一个NodePort类型的Service. 本质就是把ClusterIP Service暴露到集群中所有Node的高端端口（default 30000-32767）。 所有的Node都会监这个端口，即使这个Node中没有这个service的Pod，**如果设置了.spec.externalTrafficPolicy属性是local，则只有运行Pod的Node会监听**，这样就避免了多跳一次。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort            // 配置NodePort，外部流量可访问k8s中的服务
  ports:
  - port: 30080             // ClusterPort,服务访问端口，集群内部访问的端口
    targetPort: 80          // TargetPort,pod控制器中定义的端口（应用访问的端口）
    nodePort: 30001         // NodePort，外部客户端访问的端口
  selector:
    name: nginx-pod
```



### Port(ClusterPort)

port是暴露在cluster ip上的端口，:port提供了集群内部客户端访问service的入口，即`clusterIP:port`。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  ports:
  # 33306 为ClusterPort
  - port: 33306
  	#3306 为targetPort是pod上的端口
    targetPort: 3306
  selector:
    name: mysql-pod
```

### TargetPort

targetPort是pod上的端口，从port/nodePort上来的数据，经过kube-proxy流入到后端pod的targetPort上，最后进入容器。

与制作容器时暴露的端口一致（使用DockerFile中的EXPOSE），例如官方的nginx（参考[DockerFile](https://github.com/nginxinc/docker-nginx)）暴露80端口。 对应的service.yaml如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort            // 配置NodePort，外部流量可访问k8s中的服务
  ports:
  - port: 30080             // ClusterPort,服务访问端口，集群内部访问的端口
    targetPort: 80          // TargetPort,pod控制器中定义的端口（应用访问的端口）
    nodePort: 30001         // NodePort，外部客户端访问的端口
  selector:
    name: nginx-pod
```

### ContainerPort

containerPort是在pod控制器中定义的、pod中的容器需要暴露的端口.**ContainerPort和Pod Port 是相同的。**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      ports:
        - containerPort: 80  # 此处定义暴露的端口
          name: http
```

