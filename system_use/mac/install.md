# 使用mac os安装,配置环境

## 编程环境

### sdkman

#### install

安装官网:  https://sdkman.io/install

#### 查看支持的java版本号

可以选用结尾为-open,为open-jdk的版本，结尾为-oracle 为oracle 的jdk

```shell
sdk list java
```

#### 版本号对应

版本对应:   https://sdkman.io/jdks



#### 安装java sdk

```shell
sdk install java <version> # 使用tab键,会自动提示可使用的java版本
```

### 切换java使用的版本

```shell
sdk use java <version> # 使用tab键,会自动提示可使用的java版本
```
