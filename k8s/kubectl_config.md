## kubectl config
修改的为/.kube/config
### set namespace
```bash
# 设置默认命名空间为devops
kubectl config set-context --current --namespace=devops
#Context "default" modified.
```