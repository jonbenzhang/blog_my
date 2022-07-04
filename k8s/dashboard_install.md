#### dashboard

```
使用kubectl proxy命令就可以使API server监听在本地的8001端口上：

$ kubectl proxy --port=8009
Starting to serve on 127.0.0.1:8009
如果想通过其它主机访问就需要指定监听的地址：

$ kubectl proxy --address=0.0.0.0  --port=8009 
Starting to serve on [::]:8009
此时通过curl访问会出现未认证的提示：

$ curl -X GET -L http://k8s-master:8009/
<h3>Unauthorized</h3>
设置API server接收所有主机的请求：

$ kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$' --port=8009
Starting to serve on [::]:8009

$kubectl  proxy --address=0.0.0.0 --disable-filter=true
--disable-filter=true 允许非localhost的访问
```

#### 创建token(dashboard登录使用)

第一为官方地址,第二个很全

https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

https://blog.csdn.net/weixin_52270081/article/details/121426166

#### 在外访问dashboard

https://blog.csdn.net/weixin_52270081/article/details/121426166

选用方式2暴露nodePort

```bash
kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
```

```yaml
apiVersion: v1
kind: Service
...
...
  ports:
  # 添加nodePort端口
  - nodePort: 30169
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort	#修改这一行即可，原为type: ClusterIP
status:
  loadBalancer: {}

```
重新查看命令空间kubernetes-dashboard中的kubernetes-dashboard服务的端口地址。

```bash
kubectl -n kubernetes-dashboard get service kubernetes-dashboard
```

这时在外部机器浏览器输入IP加`30169`，（注意必须是 https）即可访问：

```
https://192.168.152.100:30169/
```



#### 跳过登录(没啥用)

删除`--auto-generate-certificates`后网页无法访问

~~首先，我们需要修改 Kubernetes 仪表板的部署以删除参数`--auto-generate-certificates`并添加以下额外参数：~~

- ~~`--enable-skip-login`~~
- ~~`--disable-settings-authorizer`~~
- ~~`--enable-insecure-login`~~
- ~~`--insecure-bind-address=0.0.0.0`~~

~~在此更改之后，Kubernetes 仪表板服务器现在在`9090`HTTP 端口上启动。然后我们需要修改`livenessProbe`为`HTTP`用作方案和`9090`端口。端口`9090`也需要添加为`containerPort`.~~
