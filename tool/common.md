## socat
可进行终端和socket连接或代理
[参考文档,详细说明](https://zhuanlan.zhihu.com/p/347722248)
```bash
# 使用本地8080端口代理192.168.1.3:80
socat TCP-LISTEN:8080,fork,reuseaddr  TCP:192.168.1.3:80
```
## proxychains
让终端的命令走代理(ping不支持)
### 安装
```
sudo apt-get install proxychains
```
### 使用
```
proxychains curl www.google.com
```
### 修改配置文件
使用自己的代理地址
```
# sudo vim /etc/proxychains.conf
# 代理地址
socks5  127.0.0.1 1080
```
