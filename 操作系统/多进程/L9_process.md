实验环境搭建网址

https://gitee.com/cn-guoziyang/oslab

多进程

多进程的资源

代码 栈 PCB 现场 映射表

PCB(Process Control Block)

![image-20210517101205397](https://gitee.com/zhangmengless/images/raw/master/img/image-20210517101205397.png)

多线程
TCB 切换不用进行映射表和PCB的切换

计算机启动步骤

1.通过寄存器CS:IP读取第一条命令,这个由ROMBIOS开始

2.然后由ROMBIOS 跳转到，零道零扇区读入内存中(这就是一般所谓的bootsect),继续进行执行

#### bootsect.s

https://blog.csdn.net/longintchar/article/details/106912096?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162132371616780274130837%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162132371616780274130837&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-5-106912096.nonecase&utm_term=setup&spm=1018.2226.3001.4450

1. bootsect 把自己搬运到 0x90000，并跳转
2. 设置栈指针
3. 加载 setup 模块到 0x90200
4. 获得磁盘驱动器参数（主要是每磁道的扇区数）
5. 打印 “Loading system …”
6. 加载 system 到 0x10000
7. 调用 kill_motor
8. 确认根文件系统设备号
9. 跳转到 setup 去执行





## 操作系统的启动流程

### 1.BIOS

​	开机cs:ip会指向BIOS中的ROM的内存地址,通过BIOS中的ROM的程序开始执行

#### 1.内存中建立中断向量表和中断服务程序。

#### 2. 加载bootsect

​		int 0x19中断向量所指向的中断服务程序，即启动加载服务程序，将软驱0号磁头对应盘面的0磁道1扇区的内容复制至内存0x07C00

### 2.bootsect(引导程序)



