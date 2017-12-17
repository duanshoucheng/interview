# 蓝厂，OPPO
1. 事件分发流程
2. View的渲染机制  
 Android系统每隔16ms就重新绘制一次Activity，也就是说，我们的应用必须在16ms内完成屏幕刷新的全部逻辑操作，即每一帧只能停留16ms。Android系统每隔16ms发出VSYNC信号，触发对UI进行渲染，VSync是Vertical Synchronization(垂直同步)的缩写，是一种在PC上很早就广泛使用的技术，可以简单的把它认为是一种定时中断。16毫秒的时间主要被两件事情所占用，第一件：将UI对象转换为一系列多边形和纹理；第二件：CPU传递处理数据到GPU。[参考1](http://www.jianshu.com/p/9ac245657127), [参考二2](http://www.cnblogs.com/ldq2016/p/6668148.html)
3. 动画的原理，底层如何给上层信号
4. 编译打包的过程
5. Android有多个资源文件夹，应用在不同分辨率下是如何查找对应文件夹下的资源的，描述整个过程
6. ANR的原理（回答主线程5秒阻塞是不行的，要读源码）

面试官是做framework的，所以问的东西偏framework，最后他说“虽然你是做应用的，但是不能浮于表面，要深入研究”，我觉得他说的很有道理。 
# 度娘 (电话面)
1. Bitmap 使用时候注意什么？
 在Android移动应用开发中，对Bitmap的不小心处理，很容易引起程序内存空间耗尽而导致的程序崩溃问题。
- 高效地加载大图片。  
BitmapFactory类提供了一些加载图片的方法：decodeByteArray(), decodeFile(), decodeResource(), 等等。
　　为了避免占用较大内存，经常使用BitmapFactory.Options 类，设置inJustDecodeBounds属性为true。 
- 不要在主线程处理图片。  
为了保证使用的资源能被回收，建议使用WeakReference, 以应用内存内存紧张时，回收部分资源，保证程序进程不被杀死。
2. Oom 是否可以try catch？  
 在try语句中声明了很大的对象，导致OOM，并且可以确认OOM是由try语句中的对象声明导致的，那么在catch语句中，可以释放掉这些对象，解决OOM的问题，继续执行剩余语句。如果OOM的原因不是try语句中的对象（比如内存泄漏），那么在catch语句中会继续抛出OOM
3. 内存泄露如何产生？  
 当一个对象已经不需要再使用本该被回收时，另外一个正在使用的对象持有它的引用从而导致它不能被回收，这导致本该被回收的对象不能被回收而停留在堆内存中，这就产生了内存泄漏。内存泄漏是造成应用程序OOM的主要原因之一。我们知道Android系统为每个应用程序分配的内存是有限的，而当一个应用中产生的内存泄漏比较多时，这就难免会导致应用所需要的内存超过系统分配的内存限额，这就造成了内存溢出从而导致应用Crash。通常我们可以借助MAT、LeakCanary等工具来检测应用程序是否存在内存泄漏.LeakCanary则是由Square开源的一款轻量级的第三方内存泄漏检测工具，当检测到程序中产生内存泄漏时，它将以最直观的方式告诉我们哪里产生了内存泄漏和导致谁泄漏了而不能被回收。
常见的内存泄漏及解决方法:
- 单例造成的内存泄漏:由于单例的静态特性使得其生命周期和应用的生命周期一样长，如果一个对象已经不再需要使用了，而单例对象还持有该对象的引用，就会使得该对象不能被正常回收，从而导致了内存泄漏。
- 非静态内部类创建静态实例造成的内存泄漏
```
public class MainActivity extends AppCompatActivity {

    private static TestResource mResource = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if(mResource == null){
            mResource = new TestResource();
        }
        //...
    }

    class TestResource {
    //...
    }
}
```
这样在Activity内部创建了一个非静态内部类的单例，每次启动Activity时都会使用该单例的数据。虽然这样避免了资源的重复创建，但是这种写法却会造成内存泄漏。因为非静态内部类默认会持有外部类的引用，而该非静态内部类又创建了一个静态的实例，该实例的生命周期和应用的一样长，这就导致了该静态实例一直会持有该Activity的引用，从而导致Activity的内存资源不能被正常回收。
解决方法：将该内部类设为静态内部类或将该内部类抽取出来封装成一个单例，如果需要使用Context，就使用Application的Context。
- Handler造成的内存泄漏
```
public class MainActivity extends AppCompatActivity {

    private final Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            // ...
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        new Thread(new Runnable() {
            @Override
            public void run() {
                // ...
                handler.sendEmptyMessage(0x123);
            }
        });
    }
}
```
>1、从Android的角度
当Android应用程序启动时，该应用程序的主线程会自动创建一个Looper对象和与之关联的MessageQueue。当主线程中实例化一个Handler对象后，它就会自动与主线程Looper的MessageQueue关联起来。所有发送到MessageQueue的Messag都会持有Handler的引用，所以Looper会据此回调Handle的handleMessage()方法来处理消息。只要MessageQueue中有未处理的Message，Looper就会不断的从中取出并交给Handler处理。另外，主线程的Looper对象会伴随该应用程序的整个生命周期。  
>2、 Java角度
在Java中，非静态内部类和匿名类内部类都会潜在持有它们所属的外部类的引用，但是静态内部类却不会。

对上述的示例进行分析，当MainActivity结束时，未处理的消息持有handler的引用，而handler又持有它所属的外部类也就是MainActivity的引用。这条引用关系会一直保持直到消息得到处理，这样阻止了MainActivity被垃圾回收器回收，从而造成了内存泄漏。
解决方法：将Handler类独立出来或者使用静态内部类，这样便可以避免内存泄漏。
- 线程造成的内存泄漏
```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        new Thread(new MyRunnable()).start();
        new MyAsyncTask(this).execute();
    }

    class MyAsyncTask extends AsyncTask<Void, Void, Void> {

        // ...

        public MyAsyncTask(Context context) {
            // ...
        }

        @Override
        protected Void doInBackground(Void... params) {
            // ...
            return null;
        }

        @Override
        protected void onPostExecute(Void aVoid) {
            // ...
        }
    }

    class MyRunnable implements Runnable {
        @Override
        public void run() {
            // ...
        }
    }
}
```
AsyncTask和Runnable都使用了匿名内部类，那么它们将持有其所在Activity的隐式引用。如果任务在Activity销毁之前还未完成，那么将导致Activity的内存资源无法被回收，从而造成内存泄漏。
解决方法：将AsyncTask和Runnable类独立出来或者使用静态内部类，这样便可以避免内存泄漏。
- 资源未关闭造成的内存泄漏
对于使用了BraodcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap(2.3以后的bitmap应该是不需要手动recycle了，内存已经在java层了。)等资源，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，从而造成内存泄漏。
- 使用ListView时造成的内存泄漏
 构造Adapter时，没有使用缓存的convertView。
解决方法：在构造Adapter时，使用缓存的convertView。
- 集合容器中的内存泄露
 我们通常把一些对象的引用加入到了集合容器（比如ArrayList）中，当我们不需要该对象时，并没有把它的引用从集合中清理掉，这样这个集合就会越来越大。如果这个集合是static的话，那情况就更严重了。
解决方法：在退出程序之前，将集合里的东西clear，然后置为null，再退出程序。
- WebView造成的泄露
 当我们不要使用WebView对象时，应该调用它的destory()函数来销毁它，并释放其占用的内存，否则其长期占用的内存也不能被回收，从而造成内存泄露。
解决方法：为WebView另外开启一个进程，通过AIDL与主线程进行通信，WebView所在的进程可以根据业务的需要选择合适的时机进行销毁，从而达到内存的完整释放。

4. 适配器模式，装饰者模式，外观模式的异同？  
适配器模式，一个适配允许通常因为接口不兼容而不能在一起工作的类工作在一起，做法是将类自己的接口包裹在一个已存在的类中。  
装饰器模式，原有的不能满足现有的需求，对原有的进行增强。  
代理模式，同一个类而去调用另一个类的方法，不对这个方法进行直接操作。
5. ANR 如何产生？  
 默认情况下，在android中Activity的最长执行时间是5秒，BroadcastReceiver的最长执行时间则是10秒。主线程执行了耗时操作就会参数ANR。网络或数据库操作，或者高耗时的计算如改变位图尺寸。运行在主线程里的任何方法都尽可能少做事情。特别是，Activity应该在它的关键生命周期方法（如onCreate()和onResume()）里尽可能少的去做创建操作。[参考](http://blog.csdn.net/lv18092081172/article/details/50564526)
6. StringBuffer 与stringBuilder 的区别？
StringBuffer是线程安全的，StringBuilder是线程不安全的，StringBuilder性能更高，比SBuffer更快。
7. 如何保证线程安全？  
在单线程中不会出现线程安全问题，而在多线程编程中，有可能会出现同时访问同一个资源的情况，这种资源（这个资源被称为：临界资源（也有称为共享资源））可以是各种类型的的资源：一个变量、一个对象、一个文件、一个数据库表等，而当多个线程同时访问同一个资源的时候，就会存在一个问题：
　　由于每个线程执行的过程是不可控的，所以很可能导致最终的结果与实际上的愿望相违背或者直接导致程序出错。基本上所有的并发模式在解决线程安全问题上，都采用“序列化访问临界资源”的方案，即在同一时刻，只能有一个线程访问临界资源，也称同步互斥访问。
　通常来说，是在访问临界资源的代码前面加上一个锁，当访问完临界资源后释放锁，让其他线程继续访问。   
在Java中，提供了两种方式来实现同步互斥访问：synchronized和Lock。
8. java四中引用
 强引用、软引用、弱引用、虚引用。Java中提供这四种引用类型主要有两个目的：
第一是可以让程序员通过代码的方式决定某些对象的生命周期；第二是有利于JVM进行垃圾回收。
9. Jni 的使用  
JNI是Java Native Interface的缩写，它提供了若干的API实现了Java和其他语言的通信
10. 多进程场景遇见过么？
11. 关于handler，在任何地方new handler 
 都是什么线程下
12. sqlite升级，增加字段的语句
- 改变表名 - ALTER TABLE 旧表名 RENAME TO 新表名 
- 增加一列 - ALTER TABLE 表名 ADD COLUMN 列名 数据类型 
13. bitmap recycler 相关
14. 强引用置为null，会不会被回收？  
 强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。不会立即被回收的，垃圾回收器会在不定时的把垃圾回收，而不会立马就回收。
15. glide 使用什么缓存？
Glide 内存缓存如何控制大小？
使用内存和磁盘缓存两种方式。Glide支持图片的二级缓存(并不是三级缓存，因为从网络加载并不属于缓存)，即内存缓存和磁盘缓存。
开发者可以通过构建一个自定义的GlideModule来配置Glide磁盘缓存的位置和大小。[参考](http://www.jianshu.com/p/325bd2f56ca7)
15. 如何保证多线程读写文件的安全？

# 某海外直播公司
一面:
1. 线程和进程的区别？
2. 为什么要有线程，而不是仅仅用进程？
3. 算法判断单链表成环与否？
4. 如何实现线程同步？  
 同步方法：synchronized关键字修饰的方法。同步代码块
参考：[link](http://blog.csdn.net/wenwen091100304/article/details/48318699)
5. hashmap数据结构？
6. arraylist 与 linkedlist 异同？
7. object类的equal 和hashcode 方法重写，为什么？
8. hashmap如何put数据（从hashmap源码角度讲解）？
9. 简述IPC？
fragment之间传递数据的方式？
简述tcp四次挥手?
threadlocal原理
内存泄漏的可能原因？
用IDE如何分析内存泄漏？
OOM的可能原因？
线程死锁的4个条件？
差值器&估值器

二面：
简述消息机制相关
进程间通信方式？
Binder相关？
触摸事件的分发？
简述Activity启动全部过程？
okhttp源码？
RxJava简介及其源码解读？
性能优化如何分析systrace？
广播的分类？
点击事件被拦截，但是相传到下面的view，如何操作？
Glide源码？
ActicityThread相关？
volatile的原理
synchronize的原理
lock原理

三面： 
三道算法题，要求在一个小时之内做完。
翻转一个单项链表 1->2->3->4->5->null =====> 5->4->3->2->1->null
string to integer
合并多个单有序链表（假设都是递增的）

四面： 

总监面，问了一些java 同步相关的。
# 由鹅厂与其他公司合资创立的公司 

一场笔试加一场面试后挂了，面试官T4级别……。 

笔试：
Activity生命周期简述
.常见内存泄漏情景及避免内存泄漏的措施
Actvity启动模式简述
简绘观察者设计模式UML图
算法，求公共子序列（或者是子串，记不清了）
Java四种引用
自定义view重写哪几个方法？
http 的session&cookie的区别
简述工作线程更新UI的方法

面试：
应用最多占多少内存
滑动卡顿如何解决（不同原因及对应处理方式）
自定义view实战
多线程，多进程 相关
Java四种引用的使用
# 某ding 
一面就挂。
XX项目你负责什么
Sqlite 怎么增加一个字段
XX项目中是怎么创建数据库的
Sqlite 怎么删除一个字段
有什么你觉得自己做得好的地方
为什么用Retrofit（一个开源库）
Retrofit与之前的网络库有什么优势
XX项目中你们自己定义的线程池来管理任务，不使用框架，那么，后来新的项目怎么设计的
你认为Rxjava的线程池与你们自己实现任务管理框架有什么区别？
内存泄漏的常见场景
怎么发现&分析内存泄漏
# 头条
1. 处理有序数组为什么比无序数组更快 参考StackOverflow
2. 热修复与插件化相关
3. Integer类是不是线程安全的，为什么
4. 不使用同步锁如何实现线程安全
面试头条的时候在线编程：从上到下从左到右输出二叉树
针对concurrent包下面的一些类的问题

# 爱奇艺（已参加一面）
1. ListView和RecyclerView优化，通过复用view实现的，是怎样复用的？  
  final RecycleBin mRecycler = new RecycleBin();//
没有被使用，下次可以复用的，不用再次创建的数据集合  
RecycleBin能够使ListView实现成百上千条数据都不会OOM最重要的一个原因
2. ListView和RecyclerView一次初始化几个item
    应该是一屏数据，不固定
3. ViewPager动画怎么
4. 事件分发机制
5. SparseArray和ArrayMap相比HashMap性能好在哪里，为什么性能更好
6. Looper、Handler、Queue、Message是怎样工作的
7. 事件分发机制，onTouchEvent在哪里（onMove）
8. 插件热修复  
 AndFix正在引入到手机国搜，全称Android hot-fix，是alibaba的Android热修复框架，支持Android 2.3到6.0的版本，支持arm与X86系统架构，支持Dalvik和ART Runtime。AndFix的原理就是方法的替换，把有bug的方法替换成补丁文件中的方法。资源文件不能替换。
9. ButterKnife源码
10. eventbus3相比之前性能提升在哪里，或者说是怎么样提升的？  
新增订阅者索引代替反射以提升性能及新加@Subscribe注解等。从EventBus3开始，我们需要在处理订阅事件的方法上添加@Subscribe注解，方法名可以任意取。我们可以使用threadMode指定分发的线程。  
ThreadMode有四种，和以前的版本差不多：
```
//和post为同一线程，这也是默认的
@Subscribe(threadMode = ThreadMode.POSTING)
public void onMessage(MessageEvent event) {
    log(event.message);
}

//回调在Android主线程
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessage(MessageEvent event) {
  textField.setText(event.message);
}

//回调在后台线程
@Subscribe(threadMode = ThreadMode.BACKGROUND)
public void onMessage(MessageEvent event){
    saveToDisk(event.message);
}

//回调在单独的线程
@Subscribe(threadMode = ThreadMode.ASYNC)
public void onMessage(MessageEvent event){
    backend.send(event.message);
}
```
使用索引不是必须的，但为了在Android上有更好的性能推荐使用。当EventBus不能使用索引的时候，它会在运行时自动回退到反射模式。因此，它还会正常工作，只是有点小慢。  
参考：  
[EventBus 3.0 新特性](http://www.jianshu.com/p/655864b5c13e);  
[EventBus3.0索引的使用](http://blog.csdn.net/aiyh0202/article/details/52923573)；

12. ANR  
 5s内无法响应用户输入事件(例如键盘输入,触摸屏幕等).造成以上两种情况的首要原因就是在主线程(UI线程)里面做了太多的阻塞耗时操作, 例如文件读写, 数据库读写, 网络查询等等.
BroadcastReceiver在10s内无法结束。下面是分析方法：
    1. 看Log
    2. 出现ANR时，系统会产生一个traces.txt的文件放在/data/anr/下，可以通过adb命令将其导出到本地：
$adb pull data/anr/traces.txt.
    3. 第三方分析工具：内存泄露工具LeaksCanary（square出品）。它可以发现内存泄露的问题
13. 滑动不流畅怎么办，卡顿的问题分析  
    1. 查看层级
    2. Fragment常驻内存的设计，造成内存消耗偏多，频繁的GC可能造成界面卡顿
    3. 由于Banner的图片太大，造成图片占用内存偏多，频繁的GC可能造成界面卡顿  
    **解决办法**：  
    - 考虑是否放弃Fragment常驻内存的方案，不使用hide()和show()对Fragment进行控制，改用replace()等方案
    - 减小Banner图片大小或者是分辨率    
    android studio的性能分析工具有：systrace和traceview。
    通过Systrace的数据，可以大体上的发现是否存在性能问题。如果要知道具体情况，就需要用到traceview工具，具体到函数级别，查看代码执行时间，分析哪些是耗时操作，可以查看线程的执行情况，某个方法执行时间、调用次数、在总体中占用的时间占比。可惜在android studio3.0给废掉了，不知道为什么。  
另外使用HierarchyViewer可以查看层级 
14. 静态使用context可能会内存泄露  
    ```
    public class Singleton {
        private static Singleton ourInstance ;
        private Context mContext;
        public static Singleton getInstance(Context context) {
            if (ourInstance == null) {
                synchronized (Singleton.class) {
                    if (ourInstance == null) {
                        ourInstance = new Singleton(context);
                    }
                }
            }
            return ourInstance;
        }
    
        private Singleton(Context context) {
            this.mContext = context;
        }
    }
    ```
    如果我们在在调用Singleton的getInstance()方法时传入了Activity。那么当instance没有释放时，这个Activity会一直存在。因此造成内存泄露。
解决方法可以将new Singleton(context)改为new Singleton(context.getApplicationContext())即可，这样便和传入的Activity没关系了。根据爱奇艺面试官，如果静态函数里没有被再次调用，或者静态类持有，那么也不会参数内存泄露。  
还有一种内存泄露方式：
```
private static Leak mLeak;
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate();
    setContentView(R.layout.activity_main);
    mLeak = new Leak();
}
class Leak{
    
}
```
mLeak是存储在静态区的静态变量，而Leak是内部类，其持有外部类Activity的引用，这样导致Activity需要被销毁时，由于被mLeak所持有，所以系统不会对其进行GC，这样就造成了内存泄露

15. MVP、MVVM
16. RxJava  
 响应式编程，用一个字来概括就是流(Stream)。Stream 就是一个按时间排序的 Events 序列,它可以放射三种不同的 Events：(某种类型的)Value、Error 或者一个” Completed” Signal。通过分别为 Value、Error、”Completed”定义事件处理函数，我们将会异步地捕获这些 Events。基于观察者模式，事件流将从上往下，从订阅源传递到观察者。  
至于使用Rx框架的优点，它可以避免回调嵌套，更优雅地切换线程实现异步处理数据。配合一些操作符，可以让处理事件流的代码更加简洁，逻辑更加清晰。
3. 先后面试阿里、阿里优酷、搜狗、爱奇艺、蔚来汽车简历没过。说下感受把，没有通过并不是说技术不行，感觉有一下几个原因：1）来自对国企的鄙视，甚至直接说我们是加班很厉害的；2）市场饱和，对人才并不是非常想招进来。3）自身简历需要二次优化