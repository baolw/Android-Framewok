> 资料来源：http://gityuan.com/2015/11/14/binder-add-service/

## 一.概述

#### 1.1 media服务注册

media入口函数是`main_mediaserver.cpp`中的`main()`方法，代码如下：

```
int main(int argc __unused, char** argv)
{
    ...
    InitializeIcuOrDie();
    //获得ProcessState实例对象【见小节2.1】
    sp<ProcessState> proc(ProcessState::self());
    //获取BpServiceManager对象
    sp<IServiceManager> sm = defaultServiceManager();
    AudioFlinger::instantiate();
    //注册多媒体服务  【见小节3.1】
    MediaPlayerService::instantiate();
    ResourceManagerService::instantiate();
    CameraService::instantiate();
    AudioPolicyService::instantiate();
    SoundTriggerHwService::instantiate();
    RadioService::instantiate();
    registerExtensions();
    //启动Binder线程池
    ProcessState::self()->startThreadPool();
    //当前线程加入到线程池
    IPCThreadState::self()->joinThreadPool();
 }
```

过程说明:

1. [获取ServiceManager](http://gityuan.com/2015/11/08/binder-get-sm/#defaultservicemanager): 讲解了defaultServiceManager()返回的是BpServiceManager对象， 用于跟servicemanager进程通信;
2. [理解Binder线程池的管理](http://gityuan.com/2016/10/29/binder-thread-pool/), 讲解了startThreadPool和joinThreadPool过程.

本文的重点就是讲解Native层服务注册的过程.

#### 1.2 类图

在Native层的服务以media服务为例，来说一说服务注册过程，先来看看media的整个的类关系图。

![](/Users/baoleiwei/project/Android-Framewok/img/add_media_player_service.png)

图解：

- 蓝色代表的是注册MediaPlayerService服务所涉及的类
- 绿色代表的是Binder架构中与Binder驱动通信过程中的最为核心的两个类；
- 紫色代表的是注册服务和[获取服务](http://gityuan.com/2015/11/15/binder-get-service/)的公共接口/父类；

#### 1.3 时序图

先通过一幅图来说说，media服务启动过程是如何向servicemanager注册服务的。

![](/Users/baoleiwei/project/Android-Framewok/img/addService.jpg)

#### 个人总结

* Binder_loop循环中，有请求到来，执行binder_parse
* BR_TRANSACTION，执行func，func指向svcmgr_handler
* 内部调用do_add_service注册指定服务
* 服务已注册，释放，否则注册服务，并保存在svclist
* 向binder驱动通信，向client端发送reply

## 六. 总结

服务注册过程(addService)核心功能：在服务所在进程创建binder_node，在servicemanager进程创建binder_ref。 其中binder_ref的desc再同一个进程内是唯一的：

- 每个进程binder_proc所记录的binder_ref的handle值是从1开始递增的；
- 所有进程binder_proc所记录的handle=0的binder_ref都指向service manager；
- 同一个服务的binder_node在不同进程的binder_ref的handle值可以不同；

Media服务注册的过程涉及到MediaPlayerService(作为Client进程)和Service Manager(作为Service进程)，通信流程图如下所示：

![](/Users/baoleiwei/project/Android-Framewok/img/media_player_service_ipc.png)

过程分析：

1. MediaPlayerService进程调用`ioctl()`向Binder驱动发送IPC数据，该过程可以理解成一个事务`binder_transaction`(记为`T1`)，执行当前操作的线程binder_thread(记为`thread1`)，则T1->from_parent=NULL，T1->from =`thread1`，thread1->transaction_stack=T1。其中IPC数据内容包含：
   - Binder协议为BC_TRANSACTION；
   - Handle等于0；
   - RPC代码为ADD_SERVICE；
   - RPC数据为”media.player”。
2. Binder驱动收到该Binder请求，生成`BR_TRANSACTION`命令，选择目标处理该请求的线程，即ServiceManager的binder线程(记为`thread2`)，则 T1->to_parent = NULL，T1->to_thread = `thread2`。并将整个binder_transaction数据(记为`T2`)插入到目标线程的todo队列；
3. Service Manager的线程`thread2`收到`T2`后，调用服务注册函数将服务”media.player”注册到服务目录中。当服务注册完成后，生成IPC应答数据(`BC_REPLY`)，T2->form_parent = T1，T2->from = thread2, thread2->transaction_stack = T2。
4. Binder驱动收到该Binder应答请求，生成`BR_REPLY`命令，T2->to_parent = T1，T2->to_thread = thread1, thread1->transaction_stack = T2。 在MediaPlayerService收到该命令后，知道服务注册完成便可以正常使用。



整个过程中，BC_TRANSACTION和BR_TRANSACTION过程是一个完整的事务过程；BC_REPLY和BR_REPLY是一个完整的事务过程。 到此，其他进行便可以获取该服务，使用服务提供的方法，下一篇文章将会讲述[如何获取服务](http://gityuan.com/2015/11/15/binder-get-service/)。



通俗的讲，个人总结：

* binder_loop中，当MediaPlaerService作为客户端，执行binder_thread_write_read,此时是write，将用户空间的命令BC_TRANSACTION写入，
* 然后由binder走BR_TRANSACTION去ServiceManager进程将相应的服务注册到服务目录中。并生成BC_REPLY给binder，binder又将BR_REPLY返给MediaPlaerService告知服务注册成功可以使用。

