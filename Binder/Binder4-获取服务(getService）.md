> 资料来源：http://gityuan.com/2015/11/15/binder-get-service/

## 一、 获取服务

在Native层的服务注册，我们选择以media为例来展开讲解，先来看看media的类关系图。

#### 1.1 类图

![](/Users/baoleiwei/project/Android-Framewok/img/get_media_player_service.png)

图解：

- 蓝色: 代表获取MediaPlayerService服务相关的类；
- 绿色: 代表Binder架构中与Binder驱动通信过程中的最为核心的两个类；
- 紫色: 代表[注册服务](http://gityuan.com/2015/11/14/binder-add-service/)和获取服务的公共接口/父类；



## 二. 总结

请求服务(getService)过程，就是向servicemanager进程查询指定服务，当执行binder_transaction()时，会区分请求服务所属进程情况。

1. 当请求服务的进程与服务属于不同进程，则为请求服务所在进程创建binder_ref对象，指向服务进程中的binder_node;
   - 最终readStrongBinder()，返回的是BpBinder对象；
2. 当请求服务的进程与服务属于同一进程，则不再创建新对象，只是引用计数加1，并且修改type为BINDER_TYPE_BINDER或BINDER_TYPE_WEAK_BINDER。
   - 最终readStrongBinder()，返回的是BBinder对象的真实子类；



[binder_write_read结构体](http://gityuan.com/2015/11/01/binder-driver/#binderwriteread)用来与Binder设备交换数据的结构, 通过ioctl与mDriverFD通信，是真正与Binder驱动进行数据读写交互的过程。 先向service manager进程发送查询服务的请求(BR_TRANSACTION)，见[Binder系列3—启动ServiceManager](http://gityuan.com/2015/11/07/binder-start-sm/)。当service manager进程收到该命令后，会执行do_find_service() 查询服务所对应的handle，然后再binder_send_reply()应答 发起者，发送BC_REPLY协议，然后调用binder_transaction()，再向服务请求者的Todo队列 插入事务。



以上同个进程和不同进程跟AIDL原理是一样的，用IBinder queryLocalInterface查看本地是否存在，即当前进程存在，则直接发挥这个处理对象。如果是不同进程，则返回的是一个代理对象，而这里的代理对象就是BpBinder，包装了handler。