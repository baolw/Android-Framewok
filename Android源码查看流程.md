# Android源码查看流程 

## 一、 创建ubuntu系统

1. windows上创建双系统（未尝试）
2. 利用win10系统的子系统创建ubuntu（这里介绍第二种）

https://docs.microsoft.com/zh-cn/windows/wsl/install-win10

具体步骤参考这个微软官方网页

**必须完整按照上面的步骤**

### 步骤一、启用适用于Linux的Windows子系统

以管理员身份打开 PowerShell 并运行：

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

非命令模式是在底下工具栏搜索中搜索**启用或关闭Windows功能**

![](E:\AndroidFramework\img\启用或关闭windows功能.png)

y以上适用于Linux的Windows子系统项目选中。

**以上步骤完成需要重启**



### 步骤2、更新到WSL 2

因为wsl2有更好的性能，避免后面编译AOSP失败，最好做个升级

有要求：

- 对于 x64 系统：**版本 1903** 或更高版本，采用 **内部版本 18362** 或更高版本。
- 对于 ARM64 系统：**版本 2004** 或更高版本，采用 **内部版本 19041** 或更高版本。
- 低于 18362 的版本不支持 WSL 2。 使用 [Windows Update 助手](https://www.microsoft.com/software-download/windows10)更新 Windows 版本。

如果版本过低，可能先需要手动下载升级



### 步骤3、启用虚拟机功能

以管理员身份打开 PowerShell 并运行：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

这里完成再一次重启



### 步骤4、下载Linux内核更新包

下载最新包：

1. [适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

2. 运行上一步中下载的更新包。 （双击以运行 - 系统将提示你提供提升的权限，选择“是”以批准此安装。）

### 步骤5 将WSL 2设置为默认版本

PowerShell复制

```powershell
wsl --set-default-version 2
```

查看已安装的WSL版本

~~~
wsl --list --verbose
~~~

### 步骤6、安装所选的Linux分发

打开win10上的应用商店，或者在搜索栏搜索**商店**

或者点击在线网站 [Microsoft Store](https://aka.ms/wslstore)

以下是不同版本的ubuntu

1. 单击以下链接会打开每个分发版的 Microsoft Store 页面：

   - [Ubuntu 16.04 LTS](https://www.microsoft.com/store/apps/9pjn388hp8c9)
   - [Ubuntu 18.04 LTS](https://www.microsoft.com/store/apps/9N9TNGVNDL3Q)
   - [Ubuntu 20.04 LTS](https://www.microsoft.com/store/apps/9n6svws3rx71)
   - [openSUSE Leap 15.1](https://www.microsoft.com/store/apps/9NJFZK00FGKV)
   - [SUSE Linux Enterprise Server 12 SP5](https://www.microsoft.com/store/apps/9MZ3D1TRP8T1)
   - [SUSE Linux Enterprise Server 15 SP1](https://www.microsoft.com/store/apps/9PN498VPMF3Z)
   - [Kali Linux](https://www.microsoft.com/store/apps/9PKR34TNCV07)
   - [Debian GNU/Linux](https://www.microsoft.com/store/apps/9MSVKQC78PK6)
   - [Fedora Remix for WSL](https://www.microsoft.com/store/apps/9n6gdm4k2hnc)
   - [Pengwin](https://www.microsoft.com/store/apps/9NV1GV1PXZ6P)
   - [Pengwin Enterprise](https://www.microsoft.com/store/apps/9N8LP0X93VCP)
   - [Alpine WSL](https://www.microsoft.com/store/apps/9p804crf0395)

2. 选择获取即可下载

3. 下载后命令行中会按顺序提示

   * 用户名
   * 密码
   * 确认密码

   

以上确认完成后即可下载创建好win10下的wsl ubuntu



## 二、下载AOSP源码

Android Source官方下载方法https://source.android.google.cn/setup/develop

由于众所周知的墙的原因，而且AOSP项目极大

所以不推荐直接下载



一般会使用国内镜像

参考https://www.jianshu.com/p/3922ec229077

关于目录位置选择

https://www.vectoros.club/post/fe9083b4.html



当下载了清华大学的镜像时，下载的是初始化包，tar，步骤如下：

0. 第一步，看最后 一定要先将下载解压的目录先设置成大小写区分。参考后面编译章节出现大小写区分错误的介绍

1. 电脑搜索winrar，邮件使用管理者打开(不用管理者权限会导致有些文件解压失败)

2. winrar内打开下载的aosp-latest.tar

3. 打开wsl ubuntu。首先可能没安装python（因为如果没装后面repo sync会说找不到python）

4. wsl中安装`python，sudo apt install python3`

5. 因为安装的是python3,此时输入python3才是python编辑模式，先执行`whereis python3`,执行`sudo ln -s /usr/bin/python3 /usr/bin/python`更改名称

6. 然后利用cd命令进入目标aosp文件夹，`cd /mnt/d/aosp/`，进入到有.repo的位置下

7. 在开始同步之前，因为google为了验证身份，所以直接同步可能会出现错误：

   error: .repo/manifests/: contains uncommitted changes

8. 进入`cd .repo/manifests `进入manifests目录:

   ~~~
   git config --global user.email "baoleiwei@gmail.com"
   git config --global user.name "baoleiwei"
   
   git stash
   git clean -d -f
   ~~~

9. 如果以上步骤执行了，利用git status发现老是清除不了，执行:

   `git config core.filemode false`
   
10. 解压出来后先到`.repo/manifests`下git status看下有没有未commit的文件，执行上述先清除掉，避免后续步骤出错

### 重要

如果init出现这个错误

`Traceback (most recent call last): File`

如果出现manifests文件夹下的文件git出问题

执行

~~~
rm -rf .repo/manifest*
~~~

将相关文件删除，然后重新初始化就可以重新下载，就正常了

0. 如果使用初始化包的方式，此时想同步指定版本的系统，跟普通init一致

   `$ repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-5.0.2_r1`，-b后面的版本标记在以下链接中：

   https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds

1. 这一步骤之后，因为当时的初始化包都已经在.repo中，此时执行`repo sync -l`即可只检出本地代码出来，不需要网络同步，速度快很多。如果此时执行`repo sync`会导致要下载很多内容。

2. 以上步骤结束之后，在设置的源码目录下就检出了整个aosp的源码了。即可进入编译环节

3. 如果需要更新最新代码，执行`repo sync`、

* 解决错误：

`RPC failed; curl 56 GnuTLS recv error (-54): Error in the pull function.`

出现这个是因为配置缓存太小导致，运行如下：

~~~
git config --global http.postBuffer 1048576000
~~~

* 在编译之前，需要执行repo init，而repo作为一个工具，执行init时会去网络更新自己，但是配置的是外网的，翻墙也没用。因此需要将该更新链接设置到国内的镜像

1. 解决办法：

   ~~~
   repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest --repo-url=https://gerrit-googlesource.lug.ustc.edu.cn/git-repo
   ~~~

   在init后面加入repo的更新链接

2. 更改配置：https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/

### .重要，防止每次都更新好久

```
 repo sync -l 仅checkout代码(这种方式比较快,不会fetch最新的远程仓库,也就是如果我下载的是20190101.tar,则最新的修改就到这天,之后至今天的修改,不同步.)
# 如果不加 -l 选项, 则更新本地仓库为最新上百G了.
```

* 以上repo sync是在master分支上，master分支上根目录是没有build等其他文件的。因此之前就导致下一步编译时说找不到build下的文件

~~~
repo init -b android-8.1.0_r35
或者
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-5.0.2_r1

然后执行
repo sync
~~~

因此需要切换到具体的版本分支才可以，

https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds

版本选择一个init就可以出现额外的目录

做完以上则可以开始编译源码



###自动下载脚本

~~~
#!/bin/bash
repo sync  -j 8  
while [ $? = 1 ]; do  
        echo “======sync failed, re-sync again======”  
        sleep 3  
        repo sync  -j 8 
done 
~~~

如果是同步个别项目，将上面的repo sync替换成

~~~
repo sync platform/development platform/frameworks/base platform/packages/apps/Calculator -j 8

~~~

然后可以使用windows Terminal，下拉打开安装的ubuntu运行，在aosp目录下

~~~
chmod a+x down.sh
./down.sh
~~~

以上运行可能报错：

参考[这里][https://blog.csdn.net/youzhouliu/article/details/79051516]，在aosp目录下，执行

~~~
sed -i "s/\r//" a.sh
~~~

然后再执行

~~~
./down.sh
~~~

即可开启repo sync。



```
sudo apt-get remove git
sudo apt-get update
sudo apt-get install git
```

```
# Linux
export GIT_TRACE_PACKET=1
export GIT_TRACE=1
export GIT_CURL_VERBOSE=1

#Windows
set GIT_TRACE_PACKET=1
set GIT_TRACE=1
set GIT_CURL_VERBOSE=1

git config --global http.postBuffer 500M
git config --global http.maxRequestBuffer 100M
git config --global core.compression 0
```



```[
sudo apt install openssl
```

[也许这个有效][https://devopscube.com/gnutls-handshake-failed-aws-codecommit/]





### 错误合集

1. 如果Repo sync的时候出错，error: RPC failed; curl 56 GnuTLS recv error (-9): A TLS packet with unexpected length was received.
    解决方法是：
    `apt install gnutls-bin`

   或者参考[这里][https://gitee.com/Fastdriod/aosp-mirror-creator]

   或者更新

   ```
   sudo apt install openssl
   ```



###切换分支

先重新init到新的分支，本地有代码改动执行这个

~~~
repo forall -c git reset --hard
~~~

然后再重新init一次，并且repo sync



## 编译源码

1.安装jdk

```
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

2.Ubuntu 18.04需要安装如下

```
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig 
```

3. 开始编译执行

   ~~~
   source build/envsetup.sh
   lunch 
   选择一个版本
   
   m
   ~~~

   # 针对源码工作目录，开启大小写敏感（重要）

   - Windows系统中文件和文件夹默认是不区分大小写的，但是编译Android源码，需要区分大小的系统。

   - 为了给WSL提供更好的支持，微软从Windows10 18917更新开始，为NTFS文件系统新增了一个`SetCaseSensitiveInfo`标志。可以有选择的根据所需的文件夹启用此flag，启用之后，NTFS文件系统就会针对**该文件夹及其子文件**视为区分大小写。

   - 此功能不仅可以在WSL中起作用，也可以在Windows下起作用。

   - ## 开启方式

     - 以管理员身份打开命令提示符或者Powershell

     - 执行以下命令开启

       ```
       fsutil file SetCaseSensitiveInfo G:\source\aosp enable
       fsutil file SetCaseSensitiveInfo G:\source\CCACHE enable  这个可能不需要
       ```

     - 执行以下命令关闭

       ```
       fsutil file SetCaseSensitiveInfo G:\source\aosp disable
       fsutil file SetCaseSensitiveInfo G:\source\CCACHE disable  这个可能不需要
       ```

     需要注意的是，这个操作不会对此目录中已有的文件生效，只有新写入的文件才会继承这个属性。所以对于目录中已有的文件，需要把文件剪切到其它目录，然后再复制回来。（同盘符下的剪切不是写入，所以后面的操作是复制。

## 参考链接：

> 里面包含了编译，也可以跳过编译直接看源码的步骤，并且罗列了很多参考网页
>
> https://www.vectoros.club/post/fe9083b4.html 后续参考这个来做
>
> https://blog.csdn.net/bondw/article/details/109188576，也可以是这个
>
> https://www.jianshu.com/p/3922ec229077
>
> 错误借鉴：https://blog.csdn.net/forgot2015/article/details/54345020     
>
> 中科大镜像：https://lug.ustc.edu.cn/wiki/mirrors/help/aosp/

