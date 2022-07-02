## socat
可进行终端和socket连接或代理
[参考文档,详细说明](https://zhuanlan.zhihu.com/p/347722248)
```bash
# 使用本地8080端口代理192.168.1.3:80
socat TCP-LISTEN:8080,fork,reuseaddr  TCP:192.168.1.3:80
```
