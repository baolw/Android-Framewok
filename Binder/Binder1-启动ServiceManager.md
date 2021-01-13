资料来源：http://gityuan.com/2015/11/07/binder-start-sm/

~~~
framework/native/cmds/servicemanager/
  - service_manager.c
  - binder.c
  
kernel/drivers/ (不同Linux分支路径略有不同)
  - staging/android/binder.c
  - android/binder.c 
~~~

## 一. 概述

ServiceManager是Binder IPC通信过程中的守护进程，本身也是一个Binder服务，但并没有采用libbinder中的多线程模型来与Binder驱动通信，而是自行编写了binder.c直接和Binder驱动来通信，并且只有一个循环binder_loop来进行读取和处理事务，这样的好处是简单而高效。

ServiceManager本身工作相对简单，其功能：查询和注册服务。 对于Binder IPC通信过程中，其实更多的情形是BpBinder和BBinder之间的通信，比如ActivityManagerProxy和ActivityManagerService之间的通信等。

### 1.1 流程图

![](/Users/baoleiwei/project/Android-Framewok/img/create_servicemanager.jpg)

启动过程主要以下几个阶段：

1. 打开binder驱动：binder_open；
2. 注册成为binder服务的大管家：binder_become_context_manager；
3. 进入无限循环，处理client端发来的请求：binder_loop；

## 四. 总结

ServiceManger集中管理系统内的所有服务，通过权限控制进程是否有权注册服务,通过字符串名称来查找对应的Service; 由于ServiceManger进程建立跟所有向其注册服务的死亡通知, 那么当服务所在进程死亡后, 会只需告知ServiceManager. 每个Client通过查询ServiceManager可获取Server进程的情况，降低所有Client进程直接检测会导致负载过重。

**ServiceManager启动流程：**

1. 打开binder驱动，并调用mmap()方法分配128k的内存映射空间：binder_open();
2. 通知binder驱动使其成为守护进程：binder_become_context_manager()；
3. 验证selinux权限，判断进程是否有权注册或查看指定服务；
4. 进入循环状态，等待Client端的请求：binder_loop()。
5. 注册服务的过程，根据服务名称，但同一个服务已注册，重新注册前会先移除之前的注册信息；
6. 死亡通知: 当binder所在进程死亡后,会调用binder_release方法,然后调用binder_node_release.这个过程便会发出死亡通知的回调.

ServiceManager最核心的两个功能为查询和注册服务：

- 注册服务：记录服务名和handle信息，保存到svclist列表；
- 查询服务：根据服务名查询相应的的handle信息。