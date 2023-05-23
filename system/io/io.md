#### 网络io模型有哪些?

io模型一共有5种,

同步阻塞IO、同步非阻塞IO、IO多路复用、信号驱动IO和异步IO。

其中信号驱动IO并不常用，我们只要介绍其他四种。

#### 什么是阻塞和非阻塞?

​		阻塞和非阻塞：阻塞IO，指的是需要内核IO操作彻底完成后，才返回用户空间执行用户的操作。

非阻塞IO指的是用户空间的程序不需要等待内核IO操作彻底完成，可以立即返回用户空间执行用户的操作，即处于非阻塞的状态，于此同时内核会立即返回给用户一个状态值。

​		换句话说就是，阻塞就是用户空间（调用线程）死等内核IO，这期间什么都不干。非阻塞是用户空间（调用线程）拿到内核返回的状态就直接返回自己的空间，IO操作可以干就干，不可以干就去干别的事情。

#### 什么是同步和异步?

同步和异步： 同步和异步指的是用户空间和内核空间IO发起的方式。同步IO只是用户空间的线程是主动发起IO的一方，内核空间是被动接受方。异步IO则是反过来，系统内核是主动发起IO请求的一方，用户空间的线程是被动接受方。

#### 同步阻塞IO**（blocking IO**）

在同步阻塞IO模型中，程序用IO调用开始，直到系统调用返回，在这段时间内，进程是阻塞的。直到返回成功后，应用程序开始处理用户空间的缓存区数据。

主要分为两个阶段：

等待数据就绪。网络IO就是等待远端数据陆续到达；磁盘IO就是等到磁盘数据从磁盘读取到内核缓冲区。
数据复制。用户空间的程序没有权限直接读取内核缓冲区的数据（操作系统处于安全的考虑），因此内核与需要把内核缓冲区的数据复制一份到进程缓冲区。
同步阻塞IO模型如下图所示：

![在这里插入图片描述](https://gitee.com/zhangmengless/images/raw/master/img/20200809183300557.PNG)


阻塞IO的特点是：在内核进行IO执行的两个阶段，用户线程都被阻塞了。

#### 同步非阻塞IO(None Blocking IO)

将Socket设置为NONBLOCK，当前连接就变成了非阻塞IO。使用非阻塞模式的IO读写，叫做同步非阻塞IO（None Blocking IO），简称NIO模型（需要注意，这里的NIO和Java里面的NIO不是一个东西！）。

在同步非阻塞IO模型中，会出现下面几种情况：

在内核缓冲区没有数据的情况下，系统调用会立即返回，返回一个调用失败的信息。这样请求就不会阻塞。
用户线程需要不断的发起IO系统调用，测试内核数据是否准备好。
内核数据准备好以后，用户线程发起系统抵用，用户线程阻塞。内核开始将数据从内核缓冲区复制到用户缓冲区，然后内核返回结果。直到这时，用户线程读到了数据，才会结束阻塞状态，重新运行起来。
同步非阻塞IO模型如下图所示：

![在这里插入图片描述](https://gitee.com/zhangmengless/images/raw/master/img/20200809183322810.PNG)

同步非阻塞IO特点：程序需要不断的进行IO系统调用，轮询数据是否准备好，如果没有准备好，就继续轮询。

#### IO多路复用模型（IO Multiplexing）

在IO多路复用模型中，引入了一种新的系统调用select/epoll，查询IO的就绪状态。通过该系统调用可以监视多个文件描述符，一旦某个描述符就绪（一般是内核缓存区可读/可写），内核能够将就绪的状态返回给应用程序。随后，应用程序根据就绪的状态，进行相应的IO系统调用。

在IO多路复用模型中通过select/epoll系统调用，单个应用程序的线程，可以不断轮询成百上千的socket连接，当某个或者某些socket网络连接有IO就绪的状态，就返回对应的可以执行的读写操作。

IO多路复用模型如下图所示：

![在这里插入图片描述](https://gitee.com/zhangmengless/images/raw/master/img/20200809183332632.PNG)


IO多路复用模型的特点：IO多路复用模型涉及两种系统调用，一种是就绪查询（select/epoll），一种是IO操作。

和NIO模型相似，多路复用IO也需要轮询。负责就绪状态查询系统调用的线程，需要不断的进行select/epoll轮询，查找出达到IO操作就绪的socket连接。

![image-20210408170845603](https://gitee.com/zhangmengless/images/raw/master/img/image-20210408170845603.png)

##### Reactor

https://blog.csdn.net/bingxuesiyang/article/details/89888664

https://www.jianshu.com/p/4d88d4e3b6d4?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation

基于IO多路复用,增加回调函数

1. 同步的等待多个事件源到达（采用select()实现）
2. 将事件多路分解以及分配相应的事件服务进行处理，这个分派采用server集中处理（dispatch）
3. 分解的事件以及对应的事件服务应用从分派服务中分离出去（handler）

###### 1、单Reactor单线程模型

这是最基本的单Reactor单线程模型。其中Reactor线程，负责多路分离套接字，有新连接到来触发connect 事件之后，交由Acceptor进行处理，有IO读写事件之后交给hanlder 处理。

Acceptor主要任务就是构建handler ，在获取到和client相关的SocketChannel之后 ，绑定到相应的hanlder上，对应的SocketChannel有读写事件之后，基于racotor 分发,hanlder就可以处理了（所有的IO事件都绑定到selector上，有Reactor分发）。

该模型 适用于处理器链中业务处理组件能快速完成的场景。不过，这种单线程模型不能充分利用多核资源，所以实际使用的不多。

###### 2、单Reactor多线程模型

相对于第一种单线程的模式来说，在处理业务逻辑，也就是获取到IO的读写事件之后，交由线程池来处理，这样可以减小主reactor的性能开销，从而更专注的做事件分发工作了，从而提升整个应用的吞吐。

###### 3、多Reactor多线程模型

1. mainReactor负责监听server socket，用来处理新连接的建立，将建立的socketChannel指定注册给subReactor。
2. subReactor维护自己的selector, 基于mainReactor 注册的socketChannel多路分离IO读写事件，读写网 络数据，对业务处理的功能，另其扔给worker线程池来完成。



![img](https://gitee.com/zhangmengless/images/raw/master/img/aHR0cHM6Ly9pbWFnZXMyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvNzk2MjE2LzIwMTgwOS83OTYyMTYtMjAxODA5MTAxNjMyMTgwOTEtNTMyNTEyNzcwLnBuZw)

##### Proactor

![img](https://gitee.com/zhangmengless/images/raw/master/img/aHR0cHM6Ly9pbWFnZXMyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvNzk2MjE2LzIwMTgwOS83OTYyMTYtMjAxODA5MTAxNjQ5MzE0MTktMTQ5OTcxODIzNC5wbmc)

#### 异步IO模型（Asynchronous IO）

异步IO模型（Asynchronous IO）简称AIO，其基本流程时：用户线程通过系统调用，向内核注册某个IO操作。内核在整个IO操作（包括数据准备、数据复制）完成后，通知用户程序，执行后续的业务操作。

在异步IO模型中，整个内核的数据处理过程中，包括内核将数据从网络物理设备（网卡）读取到内核缓存区、将内核缓冲区的数据复制到用户缓冲区，用户程序都不需要阻塞。

异步IO模型如下图所示：

![在这里插入图片描述](https://gitee.com/zhangmengless/images/raw/master/img/20200809183346595.PNG)

异步IO模型的特点：在内核等待数据和复制数据的两个阶段，用户线程都不是阻塞的。当内核的IO操作（等待数据和复制数据）全部完成后，内核会通知应用程序读数据。

#### 四种IO模型的优缺点?

###### 同步阻塞IO**（blocking IO**）

优点：程序开发简单；在阻塞等待数据期间，用户线程挂起，不占用CPU资源。

缺点：需要为每一个连接的IO配备一个线程，在高并发应用场景下，需要大量的线程来维护大量的网络连接，内存、线程切换开销会十分巨大。

###### 同步非阻塞IO(None Blocking IO)

优点:每次发起的IO系统调用，在内核等待数据过程中可以立即返回。用户线程不会阻塞，实时性较好。

缺点：需要不断的进行系统调用轮询，测试数据是否准备好，将占用大量的CPU时间，效率低下。

###### IO多路复用模型（IO Multiplexing）

优点：使用一个查询就绪状态的线程就可以同时同时轮询成千上万个连接。系统不必要创建和维护大量的线程，大大减小了系统的开销。

缺点：select/epoll调用都是阻塞式的，属于同步IO。都需要在读写事件就绪后，由系统调用本身负责进行读写，也就是这个读写过程时阻塞的。

###### 异步IO（Asynchronous IO）

优点：在内核等待数据和复制数据的两个阶段，用户线程都不是阻塞的。

缺点：需要操作系统底层内核提供支持。



设计模式的两大主题是系统复用与系统扩展

单一职责原则，开闭原则，里氏代换原则，依赖倒转原则，接口隔离原则，迪米特法则、

1. 创建型模式：用于描述“怎样创建对象”，它的主要特点是“将**对象的创建与使用分离**”。GoF 中提供了**单例**、原型、工厂方法、抽象工厂、建造者等 5 种创建型模式。
2. 结构型模式：用于描述如何将类或对象按某种布局组成更大的结构，GoF 中提供了代理、适配器、桥接、**装饰**、外观、享元、组合等 7 种结构型模式。
3. 行为型模式：用于描述类或对象之间怎样相互协作共同完成单个对象都无法单独完成的任务，以及怎样分配职责。GoF 中提供了模板方法、策略、命令、职责链、状态、观察者、中介者、**迭代器**、访问者、备忘录、**解释器**等 11 种行为型模式

- **单一职责原则(Single Responsibility Principle, SRP)：**

　　**一个类只负责一个功能领域中的相应职责，或者可以定义为：就一个类而言，应该只有一个引起它变化的原因。**

**开闭原则**(Open-Closed Principle, OCP)：

　　**一个软件实体应当对扩展开放，对修改关闭。即软件实体应尽量在不修改原有代码的情况下进行扩展。**

**里氏代换原则**(Liskov Substitution Principle, LSP)：

　　**所有引用基类（父类）的地方必须能透明地使用其子类的对象。**

　　在程序中尽量使用基类类型来对对象进行定义，而在运行时再确定其子类类型，用子类对象来替换父类对象

**依赖倒转原则**(Dependency Inversion Principle, DIP)：抽象不应该依赖于细节，细节应当依赖于抽象。换言之，要针对接口编程，而不是针对实现编程。

**接口隔离原则**(Interface Segregation Principle, ISP)：据接口隔离原则，当一个接口太大时，我们需要将它分割成一些更细小的接口

**迪米特法则**:  一个软件实体应当尽可能少地与其他实体发生相互作用。