面试个人总结：
###### 1、开发中遇到的问题，是怎样解决的：
1）启动页遇见的问题。启动页会返回一组JSON数组，数组包含
2）首页
##### 2、有没有碰到卡顿，是怎样解决的：
1）首页布局嵌套太深，使用相对布局减少部分线性布局。


# 第一面：电话面试
是否熟悉framework层，如果熟悉，那就对framework做个简介。
  3：是否熟悉多线程，如果熟悉，介绍下线程。
对象锁和类锁是否会互相影响，会举例子让你判断锁的使用是否恰当，并说出原因。
  5：是否熟悉Lopper架构，如果熟悉说下其原理，如果你自己实现，你会怎么实现。这里主要考察阻塞消息队列原理，和其变形。
  6：自定义控件原理，及消息分发流程。
7：binder工作原理。
  8：ActivityThread，Ams，Wms的工作原理。
  9：如果工作中需要修改framework，你如何寻找切入点。

# 技术面：
主要聊的是framework、binder、并发容器、线程并发和对象锁，再就是ndk使用的是否熟练，是否熟练hook技术等；还有你遇到过什么技术难点，是如何解决的。还有你读过什么开源工程，有什么感想，你是否考虑过做一个开源工程。最后就是设计一个多线程并发处理大数据量，然后刷新UI的架构。

安卓view绘制机制和加载过程，请详细说下整个流程
1.ViewRootImpl会调用performTraversals(),其内部会调用performMeasure()、performLayout、performDraw()。


2.performMeasure()会调用最外层的ViewGroup的measure()-->onMeasure(),ViewGroup的onMeasure()是抽象方法，但其提供了measureChildren()，这之中会遍历子View然后循环调用measureChild()这之中会用getChildMeasureSpec()+父View的MeasureSpec+子View的LayoutParam一起获取本View的MeasureSpec，然后调用子View的measure()到View的onMeasure()-->setMeasureDimension(getDefaultSize(),getDefaultSize()),getDefaultSize()默认返回measureSpec的测量数值，所以继承View进行自定义的wrap_content需要重写。


3.performLayout()会调用最外层的ViewGroup的layout(l,t,r,b),本View在其中使用setFrame()设置本View的四个顶点位置。在onLayout(抽象方法)中确定子View的位置，如LinearLayout会遍历子View，循环调用setChildFrame()-->子View.layout()。


4.performDraw()会调用最外层ViewGroup的draw():其中会先后调用background.draw()(绘制背景)、onDraw()(绘制自己)、dispatchDraw()(绘制子View)、onDrawScrollBars()(绘制装饰)。


5.MeasureSpec由2位SpecMode(UNSPECIFIED、EXACTLY(对应精确值和match_parent)、AT_MOST(对应warp_content))和30位SpecSize组成一个int,DecorView的MeasureSpec由窗口大小和其LayoutParams决定，其他View由父View的MeasureSpec和本View的LayoutParams决定。ViewGroup中有getChildMeasureSpec()来获取子View的MeasureSpec。


6.三种方式获取measure()后的宽高：

  ● 1.Activity#onWindowFocusChange()中调用获取 


  ● 2.view.post(Runnable)将获取的代码投递到消息队列的尾部。


  ● 3.ViewTreeObservable.
activty的加载过程 请详细介绍下:
1.ctivity中最终到startActivityForResult()（mMainThread.getApplicationThread()传入了一个ApplicationThread检查APT）
->Instrumentation#execStartActivity()和checkStartActivityResult()(这是在启动了Activity之后判断Activity是否启动成功，例如没有在AM中注册那么就会报错)
->ActivityManagerNative.getDefault().startActivity()（类似AIDL，实现了IAM，实际是由远端的AMS实现startActivity()）
->ActivityStackSupervisor#startActivityMayWait()
->ActivityStack#resumeTopActivityInnerLocked
->ActivityStackSupervisor#realStartActivityLocked()（在这里调用APT的scheduleLaunchActivity,也是AIDL，不过是在远端调起了本进程Application线程）
->ApplicationThread#scheduleLaunchActivity()（这是本进程的一个线程，用于作为Service端来接受AMS client端的调起）
->ActivityThread#handleLaunchActivity()（接收内部类H的消息，ApplicationThread线程发送LAUNCH_ACTIVITY消息给H）
->最终在ActivityThread#performLaunchActivity()中实现Activity的启动完成了以下几件事：


2.从传入的ActivityClientRecord中获取待启动的Activity的组件信息


3.创建类加载器，使用Instrumentation#newActivity()加载Activity对象


4.调用LoadedApk.makeApplication方法尝试创建Application，由于单例所以不会重复创建。


5.创建Context的实现类ContextImpl对象，并通过Activity#attach()完成数据初始化和Context建立联系，因为Activity是Context的桥接类，
最后就是创建和关联window，让Window接收的事件传给Activity，在Window的创建过程中会调用ViewRootImpl的performTraversals()初始化View。


6.Instrumentation#callActivityOnCreate()->Activity#performCreate()->Activity#onCreate().onCreate()中会通过Activity#setContentView()调用PhoneWindow的setContentView()
更新界面。


参考：http://www.jianshu.com/p/cf5092fa2694

# 12.2号阿里蚂蚁财富招聘：
以下内容为阿里官网各个部门招聘要求：
### 1 精通Android性能调优，熟悉 Android 源码 
### 2 具有一定的架构设计能力，能够很好的进行模块设计
### 3 具有安全性，性能（电量、流量、渲染速度）方面的开发经验
### 4 java 虚拟机调用方法的过程
### 5 问到了渲染引擎，问到了Linux内核和驱动。。。。
### 6 自定义view的过程。  
  自定义控件的实现有三种方式，分别是：组合控件、自绘控件和继承控件。还可以自定义自己的属性。有时候需要测量，测量的模式共有三种：
  - EXACTLY 精确模式，两种情况
   控件的layout_width，layout_height
    1. 指定数值时，例如layout_width="100dp"
    2. 指定为match_parent
  - AT_MOST 最大模式
   控件的layout_width，layout_height指定为wrap_content
    >控件大小一般会随着子空间的或内容的变化而变化，此时要求控件的尺寸只要不超过父类控件允许的最大尺寸即可
  - UNSPECIFIED 未指明模式
   不指定控件大小，View想多大就多大。  

    View类默认的onMeasure()方法只支持EXACTLY模式。自定义View时，需要重写onMeasure()方法后，才可以支持其他的模式。假如想要自定义的控件支持wrap_content，就要在onMeasure()中告诉系统自定义控件wrap_content时的大小。
### 7 数据库、greenDAO源码
### 8 什么是架构
架构，又名软件架构，是有关软件整体结构与组件的抽象描述，用于指导大型软件系统各个方面的设计。MVP应该不错的框架。无论从架构还是代码上看，分层都是三层：视图层(PresentationLayer)、控制层(DomainLayer)、数据流层(Data Layer)。层级之间通过添加接口层作为分隔实现解耦。可以使用的架构[Android-cleanArchitecture](https://github.com/android10/Android-CleanArchitecture) 简单来说，优点有以下        
1. 层次分明，各层级之间都不管对方如何实现，只关注结果；        
2. 在视图层(Presentation Layer)使用MVP架构，使原本臃肿的Activity(或Fragment)变得简单，其处理方法都交给了Presenter。        
3. 易于做测试，只要基于每个模块单独做好单元测试就能确保整体的稳定性。        
4. 易于快速迭代，基于代码的低耦合，只需在业务逻辑上增加接口，然后在相应的层级分别实现即可，丝毫不影响其他功能。        ....等等

### 9 简历上的内容
### 10 熟悉Android调试工具和方法
### 11 熟悉网络编程、多线程、熟悉TCP、HTTP/HTTPS等协议； 
### 12 精通UIKit Framework以及内存模型; 
 

自测结果：
1. ~~Handler机制~~ ；
2. ~~事件分发机制~~；
3. OkHttp，看过一遍；
4. EventBus看过一遍；
5. 内存泄露问题；
6. ReactNative集成；
7. GreenDAO实现[3.0详解](https://www.cnblogs.com/whoislcj/p/5651396.html)
 优点：
- 性能高，号称Android最快的关系型数据库
- 内存占用小
- 库文件比较小，小于100K，编译时间低，而且可以避免65K方法限制
- 支持数据库加密  greendao支持SQLCipher进行数据库加密 有关SQLCipher可以参考这篇博客Android数据存储之Sqlite采用SQLCipher数据库加密实战
- 简洁易用的API

GreenDao  3.0最大的变化就是采用注解的方式通过编译方式生成Java数据对象和DAO对象  
DaoMaster --(create)-->DaoSession--(create&manage)-->xxDao--(load&saves)-->xxEntity.除了XXEntity需要手动写，其他都是自动生成的。
8. 新闻列表的已读状态；  
 保存新闻列表已读状态：Map<String, Long> cache = new HashMap<>(); 
9. Rxjava/RxAndroid;
10. MVC /MVP
11. dagger2
 Dagger2中没有再使用反射机制，而是使用派遣方法，自动生成如同自己手写的代码，好处是：第一，谷歌声称提高了13%的效率；第二，代码的调试变得更简单，缺点是缺乏灵活性。
12. Kotlin
13. NodeJS
14. Lambdas
15. AS分析性能，找出APK内存泄露问题
16. Volley
17. Glide?picasso
18. Retrofit2  
 Retrofit就是充当了一个适配器（Adapter）的角色：将一个Java接口翻译成一个Http请求，然后用OkHttp去发送这个请求*。  
Retrofit2的动态代理理解：
ZhuanLanApi api = retrofit.create(ZhuanLanApi.class);。插播一个动态代理的定义：**Java动态代理就是给了程序员一种可能：当你要调用某个Class的方法前或后，插入你想要执行的代码**。传递进去的是一个接口，使用动态代理生成了一个实现类。  
在调用的时候：Call<ZhuanLanAuthor> call = api.getAuthor("qinchao");
会被动态代理拦截，然后调用Proxy.newProxyInstance方法中的InvocationHandler对象，它的invoke方法有三个参：
- Object proxy: 代理对象，不关心这个
- Method method：调用的方法，就是getAuthor方法
- Object... args：方法的参数，就是"qinchao"  
而Retrofit关心的就是method和它的参数args，接下去Retrofit就会用Java反射获取到getAuthor方法的注解信息，配合args参数，创建一个ServiceMethod对象。ServiceMethod就像是一个中央处理器，传入Retrofit对象和Method对象，调用各个接口和解析器，最终生成一个Request，包含api 的域名、path、http请求方法、请求头、是否有body、是否是multipart等等。最后返回一个Call对象，Retrofit2中Call接口的默认实现是OkHttpCall，它默认使用OkHttp3作为底层http请求client
[Retrofit2 源码解析](http://www.jianshu.com/p/c1a3a881a144)  
Retrofit.create里的动态代理有ServiceMethod，ServiceMethod解析请求的接口方法的注解（包括方法注解和参数注解），然后通过ServiceMethod.toRequest()方法把解析后的值都传递给RequestBuilder。正如其名RequestBuilder，最后需要通过build()方法创建Request。
19. ButterKnife
20. EventBus
21. Gson
22. groovy
23. jni
 

友盟性能分析、网络安全
onstart onpause
How to use onStart()? For example, you should unregister listeners for GPS, sensors, etc in onStop() and register again in onStart(). If you register it in onCreate() and unregister in onDestroy(), then GPS service will work always and it will drain battery.

https://stackoverflow.com/questions/21302220/what-does-onstart-really-do-android
