## tcp/ip协议

### 网络分层

## ![OSI 参考模型与 TCP/IP 的关系](https://gitee.com/zhangmengless/images/raw/master/img/HMQGKWRwexdnVl6.png)

- 200 - 请求成功
- 301 - 资源（网页等）被永久转移到其它URL
- 302 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI
- 401 请求要求用户的身份认证
- 403 服务器理解请求客户端的请求，但是拒绝执行此请求
- 404 - 请求的资源（网页等）不存在
- 500 - 内部服务器错误

### message format(报文格式)

![TCP-Header](https://gitee.com/zhangmengless/images/raw/master/img/2SMhvgjCtLaNFzO.jpg)

sequence number 序列号 : 在建立连接时由计算机生成的随机数作为其初始值，通过 SYN 包传给接收端主机，每发送一次数据，就「累加」一次该「数据字节数」的大小。**用来解决网络包乱序问题。**

acknowledgement number 确认号: 指下一次「期望」收到的数据的序列号，发送端收到这个确认应答以后可以认为在这个序号以前的数据都已经被正常接收。**用来解决不丢包的问题。**	

ISN(Initial Sequence Number)初始化序号: *ISN = M + F (localhost, localport, remotehost, remoteport)*

 - M 是一个计时器，这个计时器每隔 4 毫秒加 1。
 - F 是一个 Hash 算法，根据源 IP、目的 IP、源端口、目的端口生成一个随机数值。要保证 Hash 算法不能被外部轻易推算得出，用 MD5 算法是一个比较好的选择。

#### 标志位

- SYN(*Syn*chronize Sequence Numbers)  同步序列编号 :该位为 `1` 时，表示希望建立连接，并在其「序列号」的字段进行序列号初始值的设定。

- ACK(**acknowledgement**)确认字符: 该位为 `1` 时，「确认应答」的字段变为有效，TCP 规定除了最初建立连接时的 `SYN` 包之外该位必须设置为 `1` 

- *FIN*(Finall)：该位为 `1` 时，表示今后不会再有数据发送，希望断开连接。当通信结束希望断开连接时，通信双方的主机之间就可以相互交换 `FIN` 位为 1 的 TCP 段。
- *RST*(Reset)：该位为 `1` 时，表示 TCP 连接中出现异常必须强制断开连接。()

![TCP 头格式](https://gitee.com/zhangmengless/images/raw/master/img/XdQw3Y6Z9Jjp1eh.png)

### 三次握手

![TCP 三次握手](https://gitee.com/zhangmengless/images/raw/master/img/RWpmKrAJlhwDz3U.png)

![image-20210405195600384](https://gitee.com/zhangmengless/images/raw/master/img/Xr6WV7G2Rz1Q3l8.png)

为什么使用三次握手而不是两次或者四次

「两次握手」：无法防止历史连接的建立，会造成双方资源的浪费，也无法可靠的同步双方序列号；
「四次握手」：三次握手就已经理论上最少可靠连接建立，所以不需要使用更多的通信次数



### 四次分手

![客户端主动关闭连接 —— TCP 四次挥手](https://gitee.com/zhangmengless/images/raw/master/img/JQnDhmpYVACtr86.png)

![image-20210405180701494](https://gitee.com/zhangmengless/images/raw/master/img/jNMRGHbuOd94lkY.png)

参考

1. [40 张图解 TCP 三次握手和四次挥手面试题]:https://leetcode-cn.com/circle/discuss/b4PW9S/

   



