原作者：http://gityuan.com/

# 一、概述

Android系统中，四大组件所涉及的多进程间的通信底层都是依赖于Binder IPC机制。例如A进程的Activity跟B进程的Service通信，就是依赖于Binder IPC。也有其他进程间通讯方式，比如Zygote通信便是采用socket。



# 二、Binder

## 2.1 IPC原理

![IPC机制](/Users/baoleiwei/project/Android-Framewok/img/binder_interprocess_communication.png)

用户空间的进程内存地址不是共享的，内核空间的内存地址是共享的。Client进程向Server进程通信，就是利用这点，采用ioctl等方法跟内核空间的驱动进行交互。

## 2.2Binder原理

![Binder机制](/Users/baoleiwei/project/Android-Framewok/img/IPC-Binder.jpg)

Client，Server，ServiceManager以及binder驱动

。ServiceManager负责管理系统中的各种服务，此处是指Native层的ServiceManager，不是Java层的，是Android进程间通讯机制Binder的守护进程。

先了解

> 启动Service Manager
>
> 获取Service Manager

图中三大步骤

1. **[注册服务(addService)](http://gityuan.com/2015/11/14/binder-add-service/)**：Server进程要先注册Service到ServiceManager。该过程：Server是客户端，ServiceManager是服务端。
2. **[获取服务(getService)](http://gityuan.com/2015/11/15/binder-get-service/)**：Client进程使用某个Service前，须先向ServiceManager中获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。
3. **使用服务**：Client根据得到的Service信息建立与Service所在的Server进程通信的通路，然后就可以直接与Service交互。该过程：client是客户端，server是服务端。

图中3个端之间交互都是虚线表示，是因为他们之间不是直接交互的，而是通过Binder驱动交互的。

## 2.3 C/S模式

![](/Users/baoleiwei/project/Android-Framewok/img/Ibinder_classes.jpg)

BpBinder（客户端）和BBinder（服务端）都是Android中Binder通讯相关的代表，它们都从IBinder类中派生而来，关系图如上

* client端：BpBinder.transact()发送事务请求；
* server端：BBinder.onTransact()接收相应事务。



- [Binder系列1—Binder Driver初探](http://gityuan.com/2015/11/01/binder-driver/)
- [Binder系列2—Binder Driver再探](http://gityuan.com/2015/11/02/binder-driver-2/)
- [Binder系列3—启动Service Manager](http://gityuan.com/2015/11/07/binder-start-sm/)
- [Binder系列4—获取Service Manager](http://gityuan.com/2015/11/08/binder-get-sm/)
- [Binder系列5—注册服务(addService)](http://gityuan.com/2015/11/14/binder-add-service/)
- [Binder系列6—获取服务(getService)](http://gityuan.com/2015/11/15/binder-get-service/)
- [Binder系列7—framework层分析](http://gityuan.com/2015/11/21/binder-framework/)
- [Binder系列8—如何使用Binder](http://gityuan.com/2015/11/22/binder-use/)
- [Binder系列9—如何使用AIDL](http://gityuan.com/2015/11/23/binder-aidl/)
- [Binder系列10—总结](http://gityuan.com/2015/11/28/binder-summary/)



首先阅读[Binder系列5—注册服务(addService)](http://gityuan.com/2015/11/14/binder-add-service/)和[Binder系列6—获取服务(getService)](http://gityuan.com/2015/11/15/binder-get-service/)，这两个过程都需要于ServiceManager打交道，那么这两个过程在开始之前都需要[Binder系列4—获取Service Manager](http://gityuan.com/2015/11/08/binder-get-sm/)，既然要获取Service Manager，那么就需要先[Binder系列3—启动Service Manager](http://gityuan.com/2015/11/07/binder-start-sm/)。在看Binder服务的注册和获取这两个过程中，不断追溯下去，最终调用到底层Binder底层驱动，这时需要了解[Binder系列1—Binder Driver初探](http://gityuan.com/2015/11/01/binder-driver/)和[Binder系列2—Binder Driver再探](http://gityuan.com/2015/11/02/binder-driver-2/)。



看完Binder系列1~系列6，那么对Binder的整个流程会有一个比较清晰的认知，这还只是停留在Native层(C/C++)。接下来，可以看看上层[Binder系列7—framework层分析](http://gityuan.com/2015/11/21/binder-framework/)的Binder架构情况，Java层 Binder架构的核心逻辑都是交由Native架构来完成，更多的是对Binder的一个封装过程，只有真正理解了Native层Binder架构，才能算掌握的Binder。

前面的这些都是讲述Binder整个流程以及原理，再接下来你可能想要自己写一套Binder的C/S架构服务。如果你是**系统工程师**可能会比较关心Native层和framework层分别该如何实现自己的自定义的Binder通信服务，见[Binder系列8—如何使用Binder](http://gityuan.com/2015/11/22/binder-use/)；如果你是**应用开发工程师**则应该更关心App是如何使用Binder的，那么可以查看文章[Binder系列9—如何使用AIDL](http://gityuan.com/2015/11/23/binder-aidl/)。

最后是对Binder的一个简单总结[Binder系列10—总结](http://gityuan.com/2015/11/28/binder-summary/)。