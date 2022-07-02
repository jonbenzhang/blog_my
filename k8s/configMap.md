## configMap
### 通过yaml创建
config_map1.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm1
data:
  # configMap中的key,value
  appLogLevel: info
  dir: /var/data
```
### 通过命令行创建
#### 通过命令中的key value直接创建
```yaml
# 创建 内容为{"a_key":"a_val","b_key":"b_val"}的configmap
kubectl create configmap cm4 --from-literal a_key=a_val --from-literal b_key=b_val
```
#### 通过文件创建
```yaml
# 创建 内容为{"aa.json":"<对应的值为aa.json文件中的内容>"}的configmap
kubectl create configmap cm2 --from-file ./aa.json
```
#### 通过文件夹创建
```yaml
# 创建 内容为{"<文件夹中的文件名>":"<对应的值为文件中的内容>"}的configmap
kubectl create configmap cm3 --from-file ./cm_dir
```