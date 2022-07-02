## pod
nginx-pod.yaml
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
        - containerPort: 80
          name: http
          #　expose container port 80 to node port 
          hostPort: 30006
          protocol: TCP
```