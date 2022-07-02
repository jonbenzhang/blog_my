## plugin
plugin need start with `kubectl-`,copy command file to one $PATH
### example1
文件 kubectl-hello
```bash
#!/bin/sh
echo "hello kubectl"
```
```bash
chmod +x ./kubectl-hello
```
run `kubectl hello`
```bash
output:
hello kubectl
```

### View the list of plug-ins

执行命令`kubectl plugin list`
```text
The following compatible plugins are available:
/usr/local/bin/kubectl-hello
```