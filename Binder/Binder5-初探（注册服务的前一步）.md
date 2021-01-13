> 资料来源：http://gityuan.com/2015/11/01/binder-driver/

~~~
kernel/drivers/ (不同Linux分支路径略有不同)
  - staging/android/binder.c
  - android/binder.c 
~~~

## 一、Binder驱动概述

### 1.1 概述

Binder驱动是Android专用的，但底层的驱动架构与Linux驱动一样。binder驱动在以misc设备进行注册，作为虚拟字符设备，没有直接操作硬件，只是对设备内存的处理。主要是驱动设备的初始化(binder_init)，打开 (binder_open)，映射(binder_mmap)，数据操作(binder_ioctl)。

![binder_driver](/Users/baoleiwei/project/Android-Framewok/img/binder_driver.png)

### 1.2 系统调用

用户态的程序调用Kernel层驱动是需要陷入内核态，进行系统调用(`syscall`)，比如打开Binder驱动方法的调用链为： open-> __open() -> binder_open()。 open()为用户空间的方法，__open()便是系统调用中相应的处理方法，通过查找，对应调用到内核binder驱动的binder_open()方法，至于其他的从用户态陷入内核态的流程也基本一致。

![](/Users/baoleiwei/project/Android-Framewok/img/binder_syscall.png)

简单说，当用户空间调用open()方法，最终会调用binder驱动的binder_open()方法；mmap()/ioctl()方法也是同理，在BInder系列的后续文章从用户态进入内核态，都依赖于系统调用过



### binder init

### binder open

### binder mmap

### binder_ioctl

binder_ioctl()函数负责在两个进程间收发IPC数据和IPC reply数据。



主要与用户交互的方法

binder_ioctl_write_read流程图如下:

![](/Users/baoleiwei/project/Android-Framewok/img/binder_write_read.png)

流程：

- 首先，把用户空间数据ubuf拷贝到内核空间bwr；
- 当bwr写缓存有数据，则执行binder_thread_write；当写失败则将bwr数据写回用户空间并退出；
- 当bwr读缓存有数据，则执行binder_thread_read；当读失败则再将bwr数据写回用户空间并退出；
- 最后，把内核数据bwr拷贝到用户空间ubuf。

这里涉及两个核心方法`binder_thread_write()`和`binder_thread_read()`方法，在Binder系列的后续文章[Binder Driver再探](http://gityuan.com/2015/11/02/binder-driver-2/)中详细介绍。



### 2.4 Command使用场景

ioctl命令常见命令的使用场景，其中BINDER_WRITE_READ最为频繁

- BINDER_WRITE_READ
  - Binder读写交互场景，IPC.talkWithDriver
- BINDER_SET_CONTEXT_MGR
  - servicemanager进程成为上下文管理者，binder_become_context_manager()
- BINDER_SET_MAX_THREADS
  - 初始化ProcessState对象，open_driver()
  - 主动调整参数，ProcessState.setThreadPoolMaxThreadCount()
- BINDER_VERSION
  - 初始化ProcessState对象，open_driver()

### 2.5 小节

- binder_init：初始化字符设备；
- binder_open：打开驱动设备，过程需要持有binder_main_lock同步锁；
- binder_mmap：申请内存空间，该过程需要持有binder_mmap_lock同步锁；
- binder_ioctl：执行相应的ioctl操作，该过程需要持有binder_main_lock同步锁；
  - 当处于binder_thread_read过程，read_buffer无数据则释放同步锁，并处于wait_event_freezable过程，等有数据到来则唤醒并尝试持有同步锁。

#### BWR核心数据图表

![](/Users/baoleiwei/project/Android-Framewok/img/binder_transaction_data.jpg)