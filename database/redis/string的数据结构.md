

下为通过key查看对应的数据的结构的命令

```
OBJECT ENCODING    key 
```

list 现在使用quicklist

###### ziplist是由***一系列特殊编码的连续内存块组成的顺序存储结构\***，类似于数组

#### 1. String 类型

字符串对象的编码可以是int、raw或者embstr。



##### 底层实现

![img](https://gitee.com/zhangmengless/images/raw/master/img/1120165-20180527221351993-607035572.png)

- string数据类型的数据结构有 embstr 以及 int

如果一个字符串的内容可以转换为long，那么该字符串就会被转换成为long类型，对象的ptr就会指向该long，并且对象类型也用int类型表示。
普通的字符串有两种，embstr和raw。embstr应该是Redis 3.0新增的数据结构,在2.8中是没有的。如果字符串对象的长度小于39字节，就用embstr对象。否则用传统的raw对象。可以从下面这段代码看出：

```c
#define REDIS_ENCODING_EMBSTR_SIZE_LIMIT 39
robj *createStringObject(char *ptr, size_t len) {
    if (len <= REDIS_ENCODING_EMBSTR_SIZE_LIMIT)
        return createEmbeddedStringObject(ptr,len);
    else
        return createRawStringObject(ptr,len);
```

embstr的好处有如下几点：

- embstr的创建只需分配一次内存，而raw为两次（一次为sds分配对象，另一次为objet分配对象，embstr省去了第一次）。
- 相对地，释放内存的次数也由两次变为一次。
- embstr的objet和sds放在一起，更好地利用缓存带来的优势。

需要注意的是，redis并未提供任何修改embstr的方式，即embstr是只读的形式。对embstr的修改实际上是先转换为raw再进行修改。

SDS（simple dynamic string) 简单动态字符串,作为 Redis的默认字符串表示

```c
struct sdshdr{
     //记录buf数组中已使用字节的数量
     //等于 SDS 保存字符串的长度
     int len;
     //记录 buf 数组中未使用字节的数量
     int free;
     //字节数组，用于保存字符串
     char buf[];
}
```

用SDS保存字符串 “Redis”具体图示如下：

　　 ![img](https://gitee.com/zhangmengless/images/raw/master/img/1120165-20180528075607627-218845583.png)

　　　　　　　　　图片来源：《Redis设计与实现》

 　我们看上面对于 SDS 数据类型的定义：

　　1、len 保存了SDS保存字符串的长度

　　2、buf[] 数组用来保存字符串的每个元素

　　3、free j记录了 buf 数组中未使用的字节数量

　　上面的定义相对于 C 语言对于字符串的定义，多出了 len 属性以及 free 属性。为什么不使用C语言字符串实现，而是使用 SDS呢？这样实现有什么好处？

　　**①、常数复杂度获取字符串长度**

　　由于 len 属性的存在，我们获取 SDS 字符串的长度只需要读取 len 属性，时间复杂度为 O(1)。而对于 C 语言，获取字符串的长度通常是经过遍历计数来实现的，时间复杂度为 O(n)。通过 strlen key 命令可以获取 key 的字符串长度。

　　**②、杜绝缓冲区溢出**

　　我们知道在 C 语言中使用 strcat 函数来进行两个字符串的拼接，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。而对于 SDS 数据类型，在进行字符修改的时候，会首先根据记录的 len 属性检查内存空间是否满足需求，如果不满足，会进行相应的空间扩展，然后在进行修改操作，所以不会出现缓冲区溢出。

　　**③、减少修改字符串的内存重新分配次数**

　　C语言由于不记录字符串的长度，所以如果要修改字符串，必须要重新分配内存（先释放再申请），因为如果没有重新分配，字符串长度增大时会造成内存缓冲区溢出，字符串长度减小时会造成内存泄露。

　　而对于SDS，由于len属性和free属性的存在，对于修改字符串SDS实现了空间预分配和惰性空间释放两种策略：

　　1、空间预分配：对字符串进行空间扩展的时候，扩展的内存比实际需要的多，这样可以减少连续执行字符串增长操作所需的内存重分配次数。

　　2、惰性空间释放：对字符串进行缩短操作时，程序不立即使用内存重新分配来回收缩短后多余的字节，而是使用 free 属性将这些字节的数量记录下来，等待后续使用。（当然SDS也提供了相应的API，当我们有需要时，也可以手动释放这些未使用的空间。）

　　**④、二进制安全**

　　因为C字符串以空字符作为字符串结束的标识，而对于一些二进制文件（如图片等），内容可能包括空字符串，因此C字符串无法正确存取；而所有 SDS 的API 都是以处理二进制的方式来处理 buf 里面的元素，并且 SDS 不是以空字符串来判断是否结束，而是以 len 属性表示的长度来判断字符串是否结束。

　　**⑤、兼容部分 C 字符串函数**

　　虽然 SDS 是二进制安全的，但是一样遵从每个字符串都是以空字符串结尾的惯例，这样可以重用 C 语言库<string.h> 中的一部分函数。                                                                                                                                                                                                                                                                                                                                                 

　　**⑥、总结**

　　![img](https://gitee.com/zhangmengless/images/raw/master/img/1120165-20180527234349672-568401853.png)

　　一般来说，SDS 除了保存数据库中的字符串值以外，SDS 还可以作为缓冲区（buffer）：包括 AOF 模块中的AOF缓冲区以及客户端状态中的输入缓冲区。后面在介绍Redis的持久化时会进行介绍。

##### 实用场景

1.缓存： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。

2.计数器：redis是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。

3.session：常见方案spring session + redis实现session共享，