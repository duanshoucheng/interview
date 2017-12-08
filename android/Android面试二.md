# 1、代理设计模式是什么  
 声明真实对象和代理对象的共同接口。代理对象角色内部含有对真实对象的引用，从而可以操作真实对象，同时代理对象提供与真实对象相同的接口以便在任何时刻都能代替真实对象。同时，代理对象可以在执行真实对象操作时，附加其他的操作，相当于对真实对象进行封装。代理角色所代表的真实对象，是我们最终要引用的对象。  
**装饰模式与代理模式的区别**：装饰器模式关注于在一个对象上动态的添加方法，然而代理模式关注于控制对对象的访问。换句话 说，用代理模式，代理类（proxy class）可以对它的客户隐藏一个对象的具体信息。因此，当使用代理模式的时候，我们常常在一个代理类中创建一个对象的实例。并且，当我们使用装饰器模 式的时候，我们通常的做法是将原始对象作为一个参数传给装饰者的构造器。
# 2、工厂设计模式是什么
 可以参考这篇[java设计--工厂模式](https://www.cnblogs.com/forlina/archive/2011/06/21/2086114.html)
# 3、远程调用（AIDL） 描述下  
 在安卓开发过程中我们可能见过这样的问题，就是在一个应用中调用另一个应用中的服务，并调用该服务中的方法。我们可能对调用服务并不陌生，可是要执行服务中的方法，却不能直接调用。因为两个服务和调用它的程序属于两个应用，在不同的项目中，根本访问不到。安卓在设计的时候也帮我们想到了这个问题，并设计了aidl
# 4、动态代理的关键点是什么  
 通过反射机制获取动态代理类的构造函数，其参数类型是调用处理器接口类型
# 5、vewPager加载一个大图片就是比较长的图片如何做
 [Android高效加载大图、多图解决方案，有效避免程序OOM](http://blog.csdn.net/guolin_blog/article/details/9316683)
# 6、TCP断开连接要几次握手  
需要四次。由于TCP连接是全双工的，因此每个方向都必须单独进行关闭。这个原则是当一方完成它的数据发送任务后就能发送一个FIN来终止这个方向的连接。收到一个 FIN只意味着这一方向上没有数据流动，一个TCP连接在收到一个FIN后仍能发送数据。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭。
 - （1）客户端A发送一个FIN，用来关闭客户A到服务器B的数据传送(报文段4)。
 - （2）服务器B收到这个FIN，它发回一个ACK，确认序号为收到的序号加1(报文段5)。和SYN一样，一个FIN将占用一个序号。
 - （3）服务器B关闭与客户端A的连接，发送一个FIN给客户端A(报文段6)。
 - （4）客户端A发回ACK报文确认，并将确认序号设置为收到序号加1(报文段7)。
# 7、Lrucache底层是如何实现？  
LinkedHashMap，双链表实现，当命中的时候， 把该map移到底部，删除顶部数据。
# 8、为什么不用软引用，而使用Lrucache  
 因为从 Android 2.3 (API Level 9)开始，垃圾回收器会更倾向于回收持有软引用或弱引用的对象，这让软引用和弱引用变得不再可靠。
# 9、Oauth2认证的4种模式
 - 授权码模式（authorization code）
 - 简化模式（implicit）
 - 密码模式（resource owner password credentials）
 - 客户端模式（client credentials）
 
参考：[理解OAuth2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
# 10、最后讲自己项目中的经验
比如说豆瓣客户端每次进入某个界面的时候都要看到最新的数据,这个刷新列表的操作 就放在onStart()的方法里面.
fillData() 这样保证每次用户看到的数据都是最新的.顺便说下为什么不能放在onResume(),如果有个对话框就会更新，显然这不是需要的需求。

多媒体播放, 播放来电话. onStop() 视频, 视频声音设置为0 , 记录视频播放的位置 mediaplayer.pause();
onStart() 根据保存的状态恢复现场. mediaplayer.start(); 

# 11、横竖屏切换时候Activity的生命周期。
 这个生命周期跟清单文件里的配置有关系
 - 不设置Activity的android:configChanges时，切屏会重新调用各个生命周期
默认首先销毁当前activity,然后重新加载
Onpause onstop ondestory oncreate onstart onresume 

- 设置Activity的android:configChanges="orientation|keyboardHidden"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

游戏开发中, 屏幕的朝向都是写死的.
# 12、如何退出Activity？如何安全退出已调用多个Activity的Application？
记录打开的Activity：
每打开一个Activity，就记录下来。在需要退出时，关闭每一个Activity即可。
```
List<Activity> lists ; 在application 全集的环境里面 
lists = new ArrayList<Activity>();
每一个activity在执行oncreate()方法的时候 lists.add(this);
Ondestory() 
lists.remove(this);
lists.add(activity);

for(Activity activity: lists)
{
	activity.finish();
}
```
# 13、什么是IntentService？有何优点？什么是HandlerThread。
普通的service ,默认运行在ui main 主线程
Sdk给我们提供的方便的,带有异步处理的service类,
异步处理的方法    OnHandleIntent()
OnHandleIntent() 处理耗时的操作

如果service运行期间调用了bindService，这时候再调用stopService的话，service是不会调用onDestroy方法的，service就stop不掉了，只能调用UnbindService, service就会被销毁


如果一个service通过startService 被start之后，多次调用startService 的话，service会多次调用onStart方法。多次调用stopService的话，service只会调用一次onDestroyed方法。


如果一个service通过bindService被start之后，多次调用bindService的话，service只会调用一次onBind方法。多次调用unbindService的话会抛出异常。

1．如果service正在调用onCreate,  onStartCommand或者onDestory方法，那么用于当前service的进程相当于前台进程以避免被killed。  
2．如果当前service已经被启动(start)，拥有它的进程则比那些用户可见的进程优先级低一些，但是比那些不可见的进程更重要，这就意味着service一般不会被killed.  
3．如果客户端已经连接到service (bindService),那么拥有Service的进程则拥有最高的优先级，可以认为service是可见的。  
4． 如果service可以使用startForeground(true)方法来将service设置为前台状态，那么系统就认为是对用户可见的，并不会在内存不足时killed。  
如果有其他的应用组件作为Service,Activity等运行在相同的进程中，那么将会增加该进程的重要性。
1. Service的特点可以让他在后台一直运行,可以在service里面创建线程去完成耗时的操作. 天气预报 widget TimerTask Timer 定期执行timertask  
 
2. Broadcast receiver捕获到一个事件之后,可以起一个service来完成一个耗时的操作. 
Broadcast receiver生命周期 和 响应时间很短 
3. 远程的service如果被启动起来,可以被多次bind, 但不会重新create.  索爱手机X10i的人脸识别的service可以被图库使用,可以被摄像机,照相机等程序使用.

# 14、在manifest和代码中如何注册和使用broadcast receiver 。
设置广播接收者的优先级,设置广播接受者的action名字 等…
详细见工程代码.
```
         <intent-filter android:priority="1000">
		   <action android:name="android.intent.action.NEW_OUTGOING_CALL"/>         
         </intent-filter>
        </receiver>
		<receiver android:name=".SmsReceiver">
			<intent-filter android:priority="1000">
				<action android:name="android.provider.Telephony.SMS_RECEIVED"/>
			</intent-filter>
		</receiver>
		<receiver android:name=".BootCompleteReceiver">
			<intent-filter >
				<action android:name="android.intent.action.BOOT_COMPLETED"	/>		
				</intent-filter>
		</receiver>
```
代码中注册,如果代码没有执行,就接受不到广播事件 
        registerReceiver(receiver, filter);

# 15、请介绍下ContentProvider是如何实现数据共享的。
把自己的数据通过uri的形式共享出去
android  系统下 不同程序 数据默认是不能共享访问
    需要去实现一个类去继承ContentProvider
```
public class PersonContentProvider extends ContentProvider{
    public boolean onCreate(){
        //..
    }
    query(Uri, String[], String, String[], String)
    insert(Uri, ContentValues)
    update(Uri, ContentValues, String, String[])
    delete(Uri, String, String[])
}
```
content:// 代表contentprovider
技巧: 1.看urlmarcher.
       2. 根据匹配码 查看增删改查的具体实现

 为什么要用ContentProvider？它和sql的实现上有什么差别？屏蔽数据存储的细节,对用户透明,用户只需要关心操作数据的uri就可以了,对应的参数 .不同app之间共享,操作数据,contentprovider还可以去增删改查本地文件. xml文件的读取,更改,网络数据读取更改 Sql也有增删改查的方法. 
# 16、请解释下在单线程模型中Message、Handler、Message Queue、Looper之间的关系。

Activity 里面默认会帮创建Looper


# 17、Framework工作方式及原理，Activity是如何生成一个view的，机制是什么。
反射 , 配置文件
可以讲下activity的源码,比如说每个activity里面都有window.callback和keyevent.callback,一些回调的接口或者函数吧. 框架把activity创建出来就会调用里面的这些回调方法,会调用activity生命周期相关的方法.
setContentView();
普通的情况:
Activity创建一个view是通过onDraw()画出来的,画这个view之前呢,还会调用onMeasure()方法来计算显示的大小.
 
Surfaceview 直接操作硬件  opengl .GLSurfaceView
图像要想被显示到界面上, 需要设备显卡, 显存.
写到显存.

每个dvm都是linux里面的一个进程.所以说这两个进程是一个进程.
# 18、Linux中跨进程通信的几种方式 。
- 管道( pipe )：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。
- 有名管道 (named pipe) ： 有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信。
- 信号量( semophore ) ： 信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
- 消息队列( message queue ) ： 消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。
- 信号 ( sinal ) ： 信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。
- 共享内存( shared memory ) ：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号两，配合使用，来实现进程间的同步和通信。
- 套接字( socket ) ： 套解口也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同及其间的进程通信。
谈对Android NDK的理解。
native develop kit   只是一个交叉编译的工具  .so 
	1.什么时候用ndk,   实时性要求高,游戏,图形渲染,   
opencv (人脸识别) , ffmpeg , rmvb  mp5 avi 高清解码. ffmpeg, opencore. 

# 19、为什么用ndk,ndk的优点 ,缺点, 
应用场景有两个：
1. 需要复杂的数学运算，消耗比较高的硬件资源的时候；
2. 跨平台编程的项目需要。比如cocos2d-x就是这样的例子

我们项目中那些地方用到了ndk, c opengl
在子线程里面可以通过postInvalide()方法;
```
  iv.invalidate();
  new Thread(){
      public void run(){
         iv.postInvalidate();
      }
  }.start();
```


# 20、简单描述下Android 数字签名。
Android 数字签名在Android系统中，所有安装到系统的应用程序都必有一个数字证书，此数字证书用于标识应用程序的作者和在应用程序之间建立信任关系. 
Android系统要求每一个安装进系统的应用程序都是经过数字证书签名的，数字证书的私钥则保存在程序开发者的手中。Android将数字证书用来标识应用程序的作者和在应用程序之间建立信任关系，不是用来决定最终用户可以安装哪些应用程序。
这个数字证书并不需要权威的数字证书签名机构认证(CA)，它只是用来让应用程序包自我认证的。
同一个开发者的多个程序尽可能使用同一个数字证书，这可以带来以下好处。
1. 有利于程序升级，当新版程序和旧版程序的数字证书相同时，Android系统才会认为这两个程序是同一个程序的不同版本。如果新版程序和旧版程序的数字证书不相同，则Android系统认为他们是不同的程序，并产生冲突，会要求新程序更改包名。

2. 有利于程序的模块化设计和开发。Android系统允许拥有同一个数字签名的程序运行在一个进程中，Android程序会将他们视为同一个程序。所以开发者可以将自己的程序分模块开发，而用户只需要在需要的时候下载适当的模块。  

在签名时，需要考虑数字证书的有效期：
1. 数字证书的有效期要包含程序的预计生命周期，一旦数字证书失效，持有改数字证书的程序将不能正常升级。
2. 如果多个程序使用同一个数字证书，则该数字证书的有效期要包含所有程序的预计生命周期。
3. Android Market强制要求所有应用程序数字证书的有效期要持续到2033年10月22日以后。 

Android数字证书包含以下几个要点：
1. 所有的应用程序都必须有数字证书，Android系统不会安装一个没有数字证书的应用程序
2. Android程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证
3. 如果要正式发布一个Android，必须使用一个合适的私钥生成的数字证书来给程序签名，而不能使用adt插件或者ant工具生成的调试证书来发布。
4. 数字证书都是有有效期的，Android只是在应用程序安装的时候才会检查证书的有效期。如果程序已经安装在系统中，即使证书过期也不会影响程序的正常功能。
# 21、udp连接和TCP的不同之处
 tcp/滑动窗口协议. 拥塞控制.
  udp 不关心数据是否达到,是否阻塞
 
画面优先. tcp 
流畅优先  udp

# 22、同步异步的理解,什么是同步,什么是异步,多次调用异步方法会出现什么问题.
进程同步：就是在发出一个功能调用时，在没有得到结果之前，该调用就不返回。**也就是必须一件一件事做,等前一件做完了才能做下一件事**  
异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。  
多次调用会出现卡顿，所以应该减少多次异步调用，或者在需要多次异步调用的地方传递不同的参数。