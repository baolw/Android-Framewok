资料来源：http://gityuan.com/2015/11/08/binder-get-sm/

~~~
framework/native/libs/binder/
  - ProcessState.cpp
  - BpBinder.cpp
  - Binder.cpp
  - IServiceManager.cpp

framework/native/include/binder/
  - IServiceManager.h
  - IInterface.h
~~~

## 一. 概述

获取Service Manager是通过`defaultServiceManager()`方法来完成，当进程[注册服务(addService)](http://gityuan.com/2015/11/14/binder-add-service/)或 [获取服务(getService)](http://gityuan.com/2015/11/15/binder-get-service/)的过程之前，都需要先调用defaultServiceManager()方法来获取`gDefaultServiceManager`对象。对于gDefaultServiceManager对象，如果存在则直接返回；如果不存在则创建该对象，创建过程包括调用open()打开binder驱动设备，利用mmap()映射内核的地址空间。



### 1.1 流程图

![](/Users/baoleiwei/project/Android-Framewok/img/get_servicemanager.jpg)

## 五. 总结

defaultServiceManager 等价于 new BpServiceManager(new BpBinder(0));

ProcessState::self()主要工作：

- 调用open()，打开/dev/binder驱动设备；
- 再利用mmap()，创建大小为`1M-8K`的内存地址空间；
- 设定当前进程最大的最大并发Binder线程个数为`16`。

BpServiceManager巧妙将通信层与业务层逻辑合为一体，

- 通过继承接口`IServiceManager`实现了接口中的业务逻辑函数；
- 通过成员变量`mRemote`= new BpBinder(0)进行Binder通信工作。
- BpBinder通过handler来指向所对应BBinder, 在整个Binder系统中`handle=0`代表ServiceManager所对应的BBinder。



### 个人总结



启动ServiceManager进程和获取BpServiceManager是前后流程

1. 启动ServiceManager进程是由init进程解析init.rc文件而创建的。

   service_manager.c

   ~~~
   int main(int argc, char **argv) {
       struct binder_state *bs;
       //打开binder驱动，申请128k字节大小的内存空间 【见小节2.2】
       bs = binder_open(128*1024);
       ...
   
       //成为上下文管理者 【见小节2.3】
       if (binder_become_context_manager(bs)) {
           return -1;
       }
   
       selinux_enabled = is_selinux_enabled(); //selinux权限是否使能
       sehandle = selinux_android_service_context_handle();
       selinux_status_open(true);
   
       if (selinux_enabled > 0) {
           if (sehandle == NULL) {  
               abort(); //无法获取sehandle
           }
           if (getcon(&service_manager_context) != 0) {
               abort(); //无法获取service_manager上下文
           }
       }
       ...
   
       //进入无限循环，处理client端发来的请求 【见小节2.4】
       binder_loop(bs, svcmgr_handler);
       return 0;
   }
   ~~~

   上面流程很清晰：

   * 先binder_open打开binder驱动
   * 然后使自己成为守护进程
   * 检查selinux权限
   * binder_loop进入循环，处理client发来的请求

   

2. 进程启动完成，当需要开始注册服务时，会调用

~~~
sp<IServiceManager> defaultServiceManager()
{
    if (gDefaultServiceManager != NULL) return gDefaultServiceManager;
    {
        AutoMutex _l(gDefaultServiceManagerLock); //加锁
        while (gDefaultServiceManager == NULL) {
             //这里创建defaultServiceManager的入口
            gDefaultServiceManager = interface_cast<IServiceManager>(
                ProcessState::self()->getContextObject(NULL));
            if (gDefaultServiceManager == NULL)
                sleep(1);
        }
    }
    return gDefaultServiceManager;
}
~~~

例如：media服务注册：

media入口函数是`main_mediaserver.cpp`中的`main()`

~~~
nt main(int argc __unused, char** argv)
{
    ...
    InitializeIcuOrDie();
    sp<ProcessState> proc(ProcessState::self());
    //获取BpServiceManager对象
    //注意这里
    sp<IServiceManager> sm = defaultServiceManager();
    AudioFlinger::instantiate();
    MediaPlayerService::instantiate();
    ResourceManagerService::instantiate();
    CameraService::instantiate();
    AudioPolicyService::instantiate();
    SoundTriggerHwService::instantiate();
    RadioService::instantiate();
    registerExtensions();
    ProcessState::self()->startThreadPool();
    IPCThreadState::self()->joinThreadPool();
 }
~~~

单例，有则直接返回。否则

* 创建ProcessState对象，打开binder驱动
* 创建BpBinder对象,作为与内核空间通信
* 获取BpServiceManager，持有BpBinder对象