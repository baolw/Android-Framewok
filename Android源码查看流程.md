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

