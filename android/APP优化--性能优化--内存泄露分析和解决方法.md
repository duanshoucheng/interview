虽然Android有自动管理内存的机制，但是对内存的不恰当使用仍然容易引起严重的性能问题。在同一帧里面创建过多的对象是件需要特别引起注意的事情。
执行GC操作的时候，任何线程的任何操作都会需要暂停，等待GC操作完成之后，其他操作才能够继续运行（所以垃圾回收运行的次数越少，对性能的影响就越少）。通常来说，单个的GC并不会占用太多时间，但是大量不停的GC操作则会显著占用帧间隔时间(16ms)。如果在帧间隔时间里面做了过多的GC操作，那么自然其他类似计算，渲染等操作的可用时间就变得少了。  

导致GC频繁执行的两个原因：
1. Memory Churn内存抖动，内存抖动是因为大量的对象被创建又在短时间内马上被释放（一般是循环内创建对象）。
2. 瞬间产生大量的对象会严重占用Young Generation的内存区域，当达到阀值，剩余空间不够的时候，也会触发GC。  

例子一：
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

例子二：
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
解决方法可以将new Singleton(context)改为new Singleton(context.getApplicationContext())即可，这样便和传入的Activity没关系了。

例子三：
在Activity生命周期结束的时候将一些自定义监听器的Activity引用置空。 

例子四：
自定义View中的onDraw方法也需要引起注意，每次屏幕发生绘制以及动画执行过程中，onDraw方法都会被调用到，避免在onDraw方法里面执行复杂的操作，避免创建对象。对于那些无法避免需要创建对象而产生内存抖动的情况，我们可以考虑对象池模型，通过对象池来解决频繁创建与销毁的问题，但是这里需要注意结束使用之后，需要手动释放对象池中的对象。
使用对象池技术有很多好处，它可以避免内存抖动，提升性能，但是在使用的时候有一些内容是需要特别注意的。通常情况下，初始化的对象池里面都是空白的，当使用某个对象的时候先去对象池查询是否存在，如果不存在则创建这个对象然后加入对象池，但是我们也可以在程序刚启动的时候就事先为对象池填充一些即将要使用到的数据，这样可以在需要使用到这些对象的时候提供更快的首次加载速度，这种行为就叫做预分配。使用对象池也有不好的一面，程序员需要手动管理这些对象的分配与释放，所以我们需要慎重地使用这项技术，避免发生对象的内存泄漏。为了确保所有的对象能够正确被释放，我们需要保证加入对象池的对象和其他外部对象没有互相引用的关系。



内存抖动是指在短时间内有大量的对象被创建或者被回收的现象。
频繁内存抖动会导致垃圾回收频繁运行。
内存泄露是指某一段内存在程序里功能上已经不需要了，但是垃圾回收机制回收内存时检测那段内存还是被需要的，不能被回收，这种在程序中在没有使用的但是又不能被回收的内存就是被泄露的内存。垃圾回收机制中，判断一段内存是否是垃圾，是否可回收的条件，这个条件是通过检查这段内存是否存在引用和被引用关系，不存在这关系时，就认为可回收，若还存在引用或被引用关系，就认为不可回收，现在就可以知道导致内存泄漏的原因是程序员没有将不用的内存去掉引用关系。
内存泄漏会导致一些内存没法被正常利用，话句话就是可以使用内存变少了，这样轻则增加垃圾回收机制运行频率，重则内存溢出（当系统需要分配一段内存，但是现有内存在垃圾回收运行后任然不足时，就会内存溢出）；为避免内存泄漏，在写程序时已经确定不需要的引用型变量，就置空；虽然即使内存没泄露，也有可能出现内存溢出，这时的内存溢出就是有别的问题导致的。


# Handler引起的内存泄露
1. 在程序启动的时候就在主线程中创建了一个Looper 对象，它内部维护着一个消息队列，并且一条一条的对消息进行处理。 
2. 当我们发送消息的时候，在Message的target里面存放着发送该消息的handler对象，即消息里面包含了一个Handler实例的引用，并且就是通过这个handler实例来回调handleMessage方法进行处理。 
如果对上面两点不明白的话，可以看看Looper与Handler解析 
3. **在Java中，非静态的内部类和匿名内部类都会隐式地持有其外部类的引用。静态的内部类不会持有外部类的引用**。   
 
理解了上面的三点，应该就差不多清楚了为什么handler会出现内存泄漏：  
Handler通过发送Message与其他线程交互，Message发出之后是存储在目标线程的MessageQueue中的，而有时候Message 也不是马上就被处理的，可能会驻留比较久的时间。在Message类中存在一个成员变量 target，它强引用了handler实例，如果Message在Queue中一直存在，就会导致handler实例无法被回收，如果handler对 应的类是非静态内部类或者匿名类 ，它持有外部Activity的引用，则当使用finish销毁Activity的时候，会出现外部类实例Activity并不会被回收，这就造成了外部类实例的泄露。 
现在应该清楚了吧，所以从中我们可以知道一点：如果内部类的存在的时间比Activity的生命周期长的时候，应该避免在Activity中使用非静态的内部类，因为内部类持有外部类的引用，如果持有的时间要长于外部类实例，当外部类结束销毁的时候，它并不能被回收掉，因为它还被内部类持有，这样就会导致内存泄漏。  
**如何解决这个问题？**
也很简单，上面说到要避免使用非静态的内部类，那我们就使用静态的内部类，或者把内部类单独写个文件，让它成为一个单独的类。另外，我们可以在里面增加一个成员变量来弱引用外部类实例，就可以调用外部类的方法。 
第一种改进方法：使用静态内部类
```
public class MainActivity extends Activity {
    //定义一个静态的内部类
    static private class MyHandler extends Handler{
        //定义一个外部类的弱引用
        private WeakReference<MainActivity> mActivity;

        public MyHandler(MainActivity activity){
            mActivity = new WeakReference<MainActivity>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = mActivity.get();
            super.handleMessage(msg);
        }
    }
    //Runnable也要定义为静态的，因为它如果定义为匿名内部类的话同样也持有外部类的引用
    private static final Runnable sRunnable = new Runnable() {
        @Override
        public void run() { /* ... */ }
    };


    private final MyHandler mHandler = new MyHandler(this);

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        new Thread(sRunnable).start();

    }
}
```
第二种改进方法：单独定义一个类
```
public class MyHandler extends Handler {
    private WeakReference<MainActivity> mActivity;

    public MyHandler(MainActivity activity){
        mActivity = new WeakReference<MainActivity>(activity);
    }

    @Override
    public void handleMessage(Message msg) {
        MainActivity activity = mActivity.get();
        //进行一些操作
        super.handleMessage(msg);
    }
}
```
```
public class MainActivity extends Activity {

    private final MyHandler mHandler = new MyHandler(this);

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mHandler.sendEmptyMessage(0);

    }
}
```
# Thread 内存泄露
线程也是造成内存泄露的一个重要的源头。线程产生内存泄露的主要原因在于线程生命周期的不可控。

看一下下面是否存在问题
```
public class ThreadActivity extends Activity {
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            new MyThread().start();
        }

        private class MyThread extends Thread {
            @Override
            public void run() {
                super.run();
                dosomthing();
            }
        }
        private void dosomthing(){

        }
    }
```
我们思考一个问题：假设MyThread的run函数是一个很费时的操作，当我们开启该线程后，将设备的横屏变为了竖屏， 
一般情况下当屏幕转换时会重新创建Activity，按照我们的想法，老的Activity应该会被销毁才对，然而事实上并非如此。 
由于我们的线程是Activity的内部类，所以MyThread中保存了Activity的一个引用，当MyThread的run函数没有结束时， 
MyThread是不会被销毁的，因此它所引用的老的Activity也不会被销毁，因此就出现了内存泄露的问题。

这种线程导致的内存泄露问题应该如何解决呢？

- 将线程的内部类，改为静态内部类。
- 在线程内部采用弱引用保存Context引用。

代码如下：
```
public class ThreadAvoidActivity extends Activity {
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
                new MyThread(this).start();
            }

            private void dosomthing() {

            }

            private static class MyThread extends Thread {
                WeakReference<ThreadAvoidActivity> mThreadActivityRef;

                public MyThread(ThreadAvoidActivity activity) {
                    mThreadActivityRef = new WeakReference<ThreadAvoidActivity>(
                            activity);
                }

                @Override
                public void run() {
                    super.run();
                    if (mThreadActivityRef == null)
                        return;
                    if (mThreadActivityRef.get() != null)
                        mThreadActivityRef.get().dosomthing();
                    // dosomthing
                }
            }
        }
```
上面的两个步骤其实是切换两个对象的双向强引用链接 
静态内部类：切断Activity 对于 MyThread的强引用。 
弱引用： 切断MyThread对于Activity 的强引用。

其他tips：
1. **尽可能使用静态内部类而不是非静态内部类**。每一个非静态内部类实例都会持有一个外部类的引用，若该引用是 Activity 的引用，那么该 Activity 在被销毁时将无法被回收。如果你的静态内部类需要一个相关 Activity 的引用以确保功能能够正常运行，那么你得确保你在对象中使用的是一个 Activity 的弱引用，否则你的 Activity 将会发生意外的内存泄漏。
2. **不要总想着 Java 的垃圾回收机制会帮你解决所有内存回收问题**。Java 线程会一直存活，直到他们都被显式关闭，抑或是其进程被 Android 系统杀死。所以，为你的后台线程实现销毁逻辑是你在使用线程时必须时刻铭记的细节，此外，你在设计销毁逻辑时要根据 Activity 的生命周期去设计，避免出现 Bug。
3. **考虑你是否真的需要使用线程**。Android 应用的框架层为我们提供了很多便于开发者执行后台操作的类。例如：我们可以使用 Loader 代替在 Activity 的生命周期中用线程通过注入执行短暂的异步后台查询操作，考虑用 Service 将结构通知给 UI 的 BroadcastReceiver。


内存泄漏产生的原因在Android中大致分为以下几种：
### 1.static变量引起的内存泄漏 
因为static变量的生命周期是在类加载时开始 类卸载时结束，也就是说static变量是在程序进程死亡时才释放，如果在static变量中 引用了Activity 那么 这个Activity由于被引用，便会随static变量的生命周期一样，一直无法被释放，造成内存泄漏。  
解决办法：   
在Activity被静态变量引用时，使用 getApplicationContext 因为Application生命周期从程序开始到结束，和static变量的一样。
### 2.线程造成的内存泄漏 
类似于上述例子中的情况，线程执行时间很长，及时Activity跳出还会执行，因为线程或者Runnable是Acticvity内部类，因此握有Activity的实例(因为创建内部类必须依靠外部类)，因此造成Activity无法释放。   
AsyncTask 有线程池，问题更严重  
解决办法：   
- 1.合理安排线程执行的时间，控制线程在Activity结束前结束。 
- 2.将内部类改为静态内部类，并使用弱引用WeakReference来保存Activity实例 因为弱引用 只要GC发现了 就会回收它 ，因此可尽快回收
- 3.BitMap占用过多内存   
bitmap的解析需要占用内存，但是内存只提供8M的空间给BitMap，如果图片过多，并且没有及时 recycle bitmap 那么就会造成内存溢出。  
解决办法：   
及时recycle 压缩图片之后加载图片  
### 4.资源未被及时关闭造成的内存泄漏 
比如一些Cursor 没有及时close 会保存有Activity的引用，导致内存泄漏  
解决办法：   
在onDestory方法中及时 close即可
### 5.Handler的使用造成的内存泄漏 
由于在Handler的使用中，handler会发送message对象到 MessageQueue中 然后 Looper会轮询MessageQueue 然后取出Message执行，但是如果一个Message长时间没被取出执行，那么由于 Message中有 Handler的引用，而 Handler 一般来说也是内部类对象，Message引用 Handler ，Handler引用 Activity 这样 使得 Activity无法回收。  
解决办法：   
依旧使用 静态内部类+弱引用的方式 可解决  
### 6.带参数的单例
 
如果我们在在调用Singleton的getInstance()方法时传入了Activity。那么当instance没有释放时，这个Activity会一直存在。因此造成内存泄露。
解决方法：  
可以将new Singleton(context)改为new Singleton(context.getApplicationContext())即可，这样便和传入的Activity没关系了。
