### create namespace
#### by file
create namespace learn
```bash
# ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: learn
```
kubectl apply -f ./ns.yaml
#### by command
create namespace learn2
```bash
kubectl create namespace learn2
```