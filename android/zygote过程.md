
Android系统是基于Linux内核的，而在Linux系统中，所有的进程都是init进程的子孙进程，也就是说，所有的进程都是直接或者间接地由init进程fork出来的。Zygote进程也不例外，它是在系统启动的过程，由init进程创建的
 
启动zygote的脚本文件为/system/bin/app_process（脚本位置），要执行的zygote的文件为app_process，对应的源码在frameworks/base/cmds/app_process目录下其入口函数main在文件app_main.cpp中，接下来我们就从这个main函数入手来分析zygote的内部逻辑。
 
 main函数主要就是创建了runtime实例，并且解析参数，然后调用runtime的start函数，接着我们分析AppRuntime的start函数，start函数主要做了以下几件事情：
- 调用startVm函数创建虚拟机；
- 调用startReg函数注册Android Natvie函数；
- 让虚拟机去执行com.android.internal.os.ZygoteInit的main函数。
 
接下来我们分析下com.android.internal.os.ZygoteInit的main函数， 创建虚拟机以及注册native函数的过程后续再分析。
com.android.internal.os.ZygoteInit的main函数主要做了四件事情：
- 调用registerZygoteSocket()创建一个套接字，用于监听ams发过来的fork请求；
- 调用preload()预加载classes 和resources；
- 调用startSystemServer()创建system server进程，ams wms pms等常见service都在该进程里面；
- 调用runSelectLoopMode()进入循环监听模式，监听外来请求。





 
简单总结一下：
1. 系统启动时init进程会创建Zygote进程，Zygote进程负责后续Android应用程序框架层的其它进程的创建和启动工作。
2. Zygote进程会首先创建一个SystemServer进程，SystemServer进程负责启动系统的关键服务，如包管理服务PackageManagerService和应用程序组件管理服务ActivityManagerService。
3. 当我们需要启动一个Android应用程序时，ActivityManagerService会通过Socket进程间通信机制，通知Zygote进程为这个应用程序创建一个新的进程。



[Android Zygote进程启动过程](http://blog.csdn.net/ericming200409/article/details/45566153)
http://blog.csdn.net/Luoshengyang/article/details/6768304
