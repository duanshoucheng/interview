现实中有很多的App卡顿是不会产生ANR的, 但是又是用户可以感知的, 给人感觉我们的App运行非常慢, 影响用户体验.
# 一、16ms原则
>Android系统每隔16ms会发出VSYNC信号重绘我们的界面(Activity).为什么是16ms, 因为Android设定的刷新率是60FPS(Frame Per Second), 也就是每秒60帧的刷新率, 约合16ms刷新一次. 

丢帧给用户的感觉就是卡顿, 而且如果运算过于复杂, 丢帧会更多, 导致界面常常处于停滞状态, 卡到爆
# 二、卡顿原因分析及处理
## 1 过于复杂的布局
界面性能取决于UI渲染性能. 我们可以理解为UI渲染的整个过程是由CPU和GPU两个部分协同完成的.

其中, CPU负责UI布局元素的Measure, Layout, Draw等相关运算执行. GPU负责栅格化(rasterization), 将UI元素绘制到屏幕上.

如果我们的UI布局层次太深,或是自定义控件的onDraw中有复杂运算, CPU的相关运算就可能大于16ms,导致卡顿.这个时候, 我们需要借助Hierarchy Viewer这个工具来帮我们分析布局了。 Hierarchy Viewer不仅可以以图形化树状结构的形式展示出UI层级, 还对每个节点给出了三个小圆点, 以指示该元素Measure, Layout, Draw的耗时及性能.
![打开层级图](http://note.youdao.com/yws/public/resource/9cd56f6819876ec80563ab61665a91d7/xmlnote/2AE8B0738E434C19B9D920CF6DD74577/8279)
 此时可以看到视图有多少层级，层级也多越有可能卡顿，所以尽量减少层级。
 ![profile node](http://note.youdao.com/yws/public/resource/9cd56f6819876ec80563ab61665a91d7/xmlnote/CB49A0095949461E9957EB8E7AC690F8/8281)
 此时点击profile node
 ![show result](http://note.youdao.com/yws/public/resource/9cd56f6819876ec80563ab61665a91d7/xmlnote/CB49A0095949461E9957EB8E7AC690F8/8281)
 就可以看到绘制每个view所需要的时间。
 ## 2 过度绘制(Overdraw)
 >通俗的说: 理想情况下, 每屏每帧上, 每个像素点应该只被绘制一次, 如果有多次绘制, 就是Overdraw, 过度绘制了.
 
 Android系统提供了可视化的方案来让我们很方便的查看overdraw的现象:
在"系统设置"-->"开发者选项"-->"调试GPU过度绘制"中开启调试:  
![image](http://note.youdao.com/yws/public/resource/9cd56f6819876ec80563ab61665a91d7/xmlnote/42A64B03FF674BC89AE1B9C56851DC36/8309)
- 原色: 没有overdraw
- 蓝色: 1次overdraw
- 绿色: 2次overdraw
- 粉色: 3次overdraw
- 红色: 4次及4次以上的overdraw  
 
一般来说, 蓝色是可接受的, 是性能优的

## 3 UI线程的复杂运算
关于运算阻塞导致的卡顿的分析,可以使用Traceview这个工具.具体Traceview的介绍,可以参考[App优化之提升你的App启动速度之理论基础](http://www.jianshu.com/p/98c1656a357a)和[App优化之提升你的App启动速度之实例挑战](http://www.jianshu.com/p/4f10c9a10ac9).

着重介绍下strictmode工具：  
**StrictMode**用来基于线程或VM设置一些策略, 一旦检测到策略违例, 控制台将输出一些警告，包含一个trace信息展示你的应用在何处出现问题.

通常用来检测主线程中的磁盘读写或网络访问等耗时操作.在Application或是Activity的onCreate中开启StrictMode:
```
public void onCreate() {
     if (BuildConfig.DEBUG) {
         // 针对线程的相关策略
         StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                 .detectDiskReads()
                 .detectDiskWrites()
                 .detectNetwork()   // or .detectAll() for all detectable problems
                 .penaltyLog()
                 .build());

         // 针对VM的相关策略
         StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                 .detectLeakedSqlLiteObjects()
                 .detectLeakedClosableObjects()
                 .penaltyLog()
                 .penaltyDeath()
                 .build());
     }
     super.onCreate();
 }
```
>如果你的线程出了问题, 控制台会有警告输出, 可以定位到代码.
相对简单, 在此就不多废话了.
解决UI线程的耗时操作方案, 可以参考ANR详解里面说到的那些线程模式.
## 4. 频繁的GC
上面说的都是处理上的, CPU, GPU相关的. 实际上内存原因也可能会造成应用不流畅, 卡顿的.为什么说频繁的GC会导致卡顿呢?
简而言之, 就是执行GC操作的时候，任何线程的任何操作都会需要暂停，等待GC操作完成之后，其他操作才能够继续运行, 故而如果程序频繁GC, 自然会导致界面卡顿.

导致频繁GC有两个原因:
- 内存抖动(Memory Churn),即大量的对象被创建又在短时间内马上被释放.
- 瞬间产生大量的对象会严重占用Young Generation的内存区域, 当达到阀值, 剩余空间不够的时候, 也会触发GC.

即使每次分配的对象需要占用很少的内存，但是他们叠加在一起会增加Heap的压力, 从而触发更多的GC.
一般来说瞬间大量产生对象一般是因为我们在代码的循环中new对象, 或是在onDraw中创建对象等. 所以说这些地方是我们尤其需要注意的...

关于内存的分析, 参考别的文件夹.

卡顿工具：BlockCannery，本文件夹下：BlockCannery.md

本文参考：
1. [Android App优化之消除卡顿](http://www.jianshu.com/p/1fb065c806e6)