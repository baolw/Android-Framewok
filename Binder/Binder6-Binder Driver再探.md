> 资料来源 http://gityuan.com/2015/11/02/binder-driver-2/

## 一、Binder通信简述

上一篇文章[Binder Driver初探](http://gityuan.com/2015/11/01/binder-driver/)介绍了Binder驱动的`init`、`open`、`mmap`、`ioctl`这4个核心方法，并说明与Binder相关的常见结构体。

Client进程通过RPC(Remote Procedure Call Protocol)与Server通信，可以简单地划分为三层，驱动层、IPC层、业务层。`demo()`便是Client端和Server共同协商好的统一方法；handle、RPC数据、代码、协议这4项组成了IPC层的数据，通过IPC层进行数据传输；而真正在Client和Server两端建立通信的基础设施便是Binder Driver。

IPC代表IPCThreadState



![](/Users/baoleiwei/project/Android-Framewok/img/binder_ipc.jpg)

## 二、Binder通信协议

### 2.1 通信模型

先列举一次完整的Binder通信过程：

![](/Users/baoleiwei/project/Android-Framewok/img/binder_transaction_ipc.jpg)

Binder协议包含在IPC数据中，分为两类:

1. `BINDER_COMMAND_PROTOCOL`：binder请求码，以”BC_“开头，简称BC码，用于从IPC层传递到Binder Driver层；
2. `BINDER_RETURN_PROTOCOL` ：binder响应码，以”BR_“开头，简称BR码，用于从Binder Driver层传递到IPC层；

Binder IPC通信至少是两个进程的交互：

- client进程执行binder_thread_write，根据BC_XXX命令，生成相应的binder_work；
- server进程执行binder_thread_read，根据binder_work.type类型，生成BR_XXX，发送到用户空间处理。

#### 2.1.1 通信过程

![binder_protocol](/Users/baoleiwei/project/Android-Framewok/img/binder_protocol.jpg)



* 根据流程图，也就是从客户端线程发起请求，是BC，对应的binder_thread_write写入
* 然后根据查找到的服务，生成BR码，调用binder_thread_read，去走server进程

#### 2.2.2 BC_PROTOCOL

binder请求码，是用`enum binder_driver_command_protocol`来定义的，是用于应用程序向binder驱动设备发送请求消息，应用程序包含Client端和Server端，以BC_开头，总17条；(-代表目前不支持的请求码)

### 2.3 binder_thread_read

响应处理过程是通过`binder_thread_read()`方法，该方法根据不同的`binder_work->type`以及不同状态，生成相应的响应码。



#### 3.3 协议转换图

![protocol_transaction](/Users/baoleiwei/project/Android-Framewok/img/protocol_transaction.jpg)

#### 3.4 数据转换图

![](/Users/baoleiwei/project/Android-Framewok/img/binder_dataflow.jpg)

图(左)说明：

- AMP.startService: 将数据封装到Parcel类型；
- IPC.writeTransactionData：将数据封装到binder_transaction_data结构体；
- IPC.talkWithDriver：将数据进一步封装到binder_write_read结构体；
  - 再通过ioctl()写入命令BINDER_WRITE_READ和binder_write_read结构体到驱动层
- binder_transaction: 将发起端数据拷贝到接收端进程的buffer结构体；

图(右)说明：

- binder_thread_read：根据binder_transaction结构体和binder_buffer结构体数据生成新的binder_transaction_data结构体，写入bwr的write_buffer，传递到用户空间。
- IPC.executeCommand: 解析binder_transaction_data数据，找到目标BBinder并调用其transact()方法;
- AMN.onTransact： 解析Parcel数据，然后调用目标服务的目标方法；
- AMS.startService： 层层封装和拆分后，执行真正的业务逻辑。

## 四、Binder内存机制

在上一篇文章从代码角度阐释了[binder_mmap()](http://gityuan.com/2015/11/01/binder-driver/#bindermmap)，这也是Binder进程间通信效率高的核心机制所在，如下图：以下是针对数据接收方的，参考最下面的进程间数据通信流程图，

![](/Users/baoleiwei/project/Android-Framewok/img/binder_physical_memory.jpg)

虚拟进程地址空间(vm_area_struct)和虚拟内核地址空间(vm_struct)都映射到同一块物理内存空间。当Client端与Server端发送数据时，Client（作为数据发送端）先从自己的进程空间把IPC通信数据`copy_from_user`拷贝到内核空间，而Server端（作为数据接收端）与内核共享数据，不再需要拷贝数据，而是通过内存地址空间的偏移量，即可获悉内存地址，整个过程只发生一次内存拷贝。一般地做法，需要Client端进程空间拷贝到内核空间，再由内核空间拷贝到Server进程空间，会发生两次拷贝。

对于进程和内核虚拟地址映射到同一个物理内存的操作是发生在数据接收端，而数据发送端还是需要将用户态的数据复制到内核态。到此，可能有读者会好奇，为何不直接让发送端和接收端直接映射到同一个物理空间，那样就连一次复制的操作都不需要了，0次复制操作那就与Linux标准内核的共享内存的IPC机制没有区别了，对于共享内存虽然效率高，但是对于多进程的同步问题比较复杂，而管道/消息队列等IPC需要复制2两次，效率较低。这里就不先展开讨论Linux现有的各种IPC机制跟Binder的详细对比，总之Android选择Binder的基于速度和安全性的考虑。

下面这图是从Binder在进程间数据通信的流程图，从图中更能明了Binder的内存转移关系。

![](/Users/baoleiwei/project/Android-Framewok/img/binder_memory_map.jpg)