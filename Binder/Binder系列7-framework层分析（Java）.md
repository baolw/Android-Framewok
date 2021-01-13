> 资料来源:http://gityuan.com/2015/11/21/binder-framework/

~~~
framework/base/core/java/android/os/
  - IInterface.java
  - IServiceManager.java
  - ServiceManager.java
  - ServiceManagerNative.java(内含ServiceManagerProxy类)

framework/base/core/java/android/os/
  - IBinder.java
  - Binder.java(内含BinderProxy类)
  - Parcel.java

framework/base/core/java/com/android/internal/os/
  - BinderInternal.java

framework/base/core/jni/
  - AndroidRuntime.cpp
  - android_os_Parcel.cpp
  - android_util_Binder.cpp
~~~

## 一、概述

### 1.1 Binder架构

binder在framework层，采用JNI技术来调用native(C/C++)层的binder架构，从而为上层应用程序提供服务。 看过binder系列之前的文章，我们知道native层中，binder是C/S架构，分为Bn端(Server)和Bp端(Client)。对于java层在命名与架构上非常相近，同样实现了一套IPC通信架构。

framework Binder架构图：查看[大图](http://gityuan.com/images/binder/java_binder/java_binder.jpg)

![](/Users/baoleiwei/project/Android-Framewok/img/java_binder.jpg)

**图解：**

- 图中红色代表整个framework层 binder架构相关组件；
  - Binder类代表Server端，BinderProxy类代码Client端；
- 图中蓝色代表Native层Binder架构相关组件；
- 上层framework层的Binder逻辑是建立在Native层架构基础之上的，核心逻辑都是交予Native层方法来处理。
- framework层的ServiceManager类与Native层的功能并不完全对应，framework层的ServiceManager类的实现最终是通过BinderProxy传递给Native层来完成的，后面会详细说明。

### 1.2 Binder类图

![](/Users/baoleiwei/project/Android-Framewok/img/class_ServiceManager.jpg)

图解：(图中浅蓝色都是Interface，其余都是Class)

1. **ServiceManager：**通过getIServiceManager方法获取的是ServiceManagerProxy对象； ServiceManager的addService, getService实际工作都交由ServiceManagerProxy的相应方法来处理；
2. **ServiceManagerProxy：**其成员变量mRemote指向BinderProxy对象，ServiceManagerProxy的addService, getService方法最终是交由mRemote来完成。
3. **ServiceManagerNative**：其方法asInterface()返回的是ServiceManagerProxy对象，ServiceManager便是借助ServiceManagerNative类来找到ServiceManagerProxy；
4. **Binder：**其成员变量mObject和方法execTransact()用于native方法
5. **BinderInternal：**内部有一个GcWatcher类，用于处理和调试与Binder相关的垃圾回收。
6. **IBinder：**接口中常量FLAG_ONEWAY：客户端利用binder跟服务端通信是阻塞式的，但如果设置了FLAG_ONEWAY，这成为非阻塞的调用方式，客户端能立即返回，服务端采用回调方式来通知客户端完成情况。另外IBinder接口有一个内部接口DeathDecipient(死亡通告)。

### 1.3 Binder类分层

整个Binder从kernel至，native，JNI，Framework层所涉及的全部类

点击查看[大图](http://gityuan.com/images/binder/java_binder_framework.jpg)

![](/Users/baoleiwei/project/Android-Framewok/img/java_binder_framework.jpg)

## 总结

* zygote初始化时执行3个类的注册

  1. 注册Binder
  2. 注册BinderInternal
  3. 注册BinderProxy

  主要就是联连通了jni和java之间的调用

* 注册服务，SM.addService

  1. getIServiceManager（）返回ServiceManagerProxy对象，持有mRemote，指向BinderProxy，BinderProxy交给Native是BpBinder，可执行binder驱动
  2. addService()，主要就是通过Java层的BinderProxy执行transact交给Native层的BpBinder::transact完成。
  3. 核心流程如下：

  ~~~
  public void addService(String name, IBinder service, boolean allowIsolated) throws RemoteException {
      ...
      Parcel data = Parcel.obtain(); //此处还需要将java层的Parcel转为Native层的Parcel
      data->writeStrongBinder(new JavaBBinder(env, obj));
      BpBinder::transact(ADD_SERVICE_TRANSACTION, *data, reply, 0); //与Binder驱动交互
      ...
  }
  ~~~

* 获取服务 SM.getService

  1. 同上获取ServiceManagerProxy对象。
  2. getService(),transact一个GET_SERVICE_TRANSACTION,交给BinderProxy，再调用native调用BpBinder，交给IPC，经过处理，最终创建了指向Binder服务端的BpBinder代理对象，即转换为Java层的BinderProxy对象。