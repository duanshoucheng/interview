Android设备不像PC那样有着足够大的内存，而且单个App占用的内存实际上是比较小的。所以避免创建不必要的对象对于Android开发尤为重要。
>Android系统每隔16ms发出VSYNC信号，对UI进行渲染，如果每次渲染都成功，就能够达到流畅的画面所需要的60fps，为了能够实现60fps，这意味着程序的大多数操作都必须在16ms内完成，时间超出16ms越多，丢的帧就越多。  

>假设我们更新屏幕的背景图片，需要24ms来做这次运算。当系统在第一个16ms时刷新界面，然而我们的运算还没有结束，无法绘出图片。当系统隔16ms再发一次VSYNC信息重绘界面时，用户才会看到更新后的图片。也就是说用户是32ms后看到了这次刷新(注意，并不是24ms)，这就是丢帧。

丢帧只是用户能感知到的表面现象，严重的会引起程序卡顿甚至ANR，深层次的原因是代码中有比较耗时的操作阻塞到了主线程，也就是性能问题。

执行GC操作的时候，任何线程的任何操作都会需要暂停，等待GC操作完成之后，其他操作才能够继续运行（所以垃圾回收运行的次数越少，对性能的影响就越少）。通常来说，单个的GC并不会占用太多时间，但是大量不停的GC操作则会显著占用帧间隔时间(16ms)。如果在帧间隔时间里面做了过多的GC操作，那么自然其他类似计算，渲染等操作的可用时间就变得少了。
导致GC频繁执行的两个原因：
1. Memory Churn内存抖动，内存抖动是因为大量的对象被创建又在短时间内马上被释放（一般是循环内创建对象）。
2. 瞬间产生大量的对象会严重占用Young Generation的内存区域，当达到阀值，剩余空间不够的时候，也会触发GC。

# 1. 过度绘制
过度绘制（Overdraw）是指屏幕上同一个像素点在同一帧时间内被绘制多次。在层级复杂的UI结构中，如果不可见的UI也被绘制，会导致某些像素区被绘制多次，浪费大量的CPU以及GPU资源。
打开android手机开发者选项，显示布局边界看绘制信息：
- 蓝色代表一次绘制，
- 淡绿代表两次绘制，
- 淡红代表三次绘制，
- 深红代表四及以上次绘制。

我们的目的是尽量减少红色的绘制信息，方法有两种。
- 简化页面UI结构，复杂的UI布局会导致大量View重叠，出现过度绘制的可能性比较大，要避免布局嵌套过多，例如一般情况下，优先使用LinearLayout布局。
- 复用背景色，例如如果父布局和子View背景色是相同的，只需要父布局设置背景色即可，子View不用设置。 

# 2. 布局优化
上面提到简化布局可以减少过度绘制的问题，布局优化可以从下面几方面入手。

- 布局的选择，能满足需求的情况下优先选择LinearLayout，因为RelativeLayout在measure会比LinearLayout多绘制一次， 
即使LinearLayout使用了weight后，性能依然会比RelativeLayout好
- 边距的设置，RelativeLayout的measure过程中，如果出现子View和布局本身高度不同时候，还会触发measure过程，解决方法很简单，使用padding代替marigin
- 复用布局，include
- 延迟加载，ViewStub
- 合并布局层级，merge

用SurfaceView或TextureView代替普通View  
SurfaceView或TextureView可以通过将绘图操作移动到另一个单独线程上提高性能。
普通View的绘制过程都是在主线程(UI线程)中完成，如果某些绘图操作影响性能就不好优化了，这时我们可以考虑使用SurfaceView和TextureView，他们的绘图操作发生在UI线程之外的另一个线程上。
因为SurfaceView在常规视图系统之外，所以无法像常规试图一样移动、缩放或旋转一个SurfaceView。TextureView是Android4.0引入的，除了与SurfaceView一样在单独线程绘制外，还可以像常规视图一样被改变。
# 3. I/O操作（SharedPreferences）
## 1. 读操作
读操作一般不会阻塞到主线程，如果读取的数据比较大而且需要有大量的处理操作，直接开子线程，在子线程处理。
## 2. 写操作
写操作比读操作复杂一些，向SharedPreferences写入数据时候，有commit和apply两个方法。

- commit方法，直接将数据同步写入磁盘
apply方法，先将数据写入内存，再异步写入磁盘
- commit和apply方法区别在于同步写入和异步写入，以及是否需要返回值。在不需要返回值的情况下，使用apply方法可以极大的提高性能。

在使用lint静态扫描代码时候，会建议使用apply去替代commit，说明官方也是支持使用apply方法的。 

使用apply方法时候，会向QueuedWork队列中添加一个等待写入操作完成的线程，只有当写入操作完成后，才会从QueuedWork将等待线程线程移除掉。

而主线程中，Service的启动和stop以及Activity的onPause()和onStop()生命周期都会等待其他异步线程完成，才会继续执行。

例如停止Service的时候代码如下（ActivityThread类中）：
```
private void handleStopService(IBinder token) {
    ···

    QueuedWork.waitToFinish(); // 等待其他异步线程完成

    ···
}
```
可以看到，如果使用apply方法写入大量复杂数据，确实有可能会阻塞到主线程，甚至可能导致ANR。

优化方法：

复杂数据 
即时性较弱（距离下次使用时间较长），新建子线程使用commit方法，防止阻塞到主线程
即时性较强，可以考虑直接放在内存中
简单数据，使用apply方法

# 4 序列化
大部分情况下，与后台交互使用的数据是gson格式。相信很多是直接使用google官方的Gson类来序列化和反序列化的。

我在项目中就遇见了有使用Gson反序列化复杂数据时候，造成的卡顿现象。
使用流配合Gson序列化，性能提高25%。
```
/ 反序列化
public List<Message> readJsonStream(InputStream in) throws IOException {
    JsonReader reader = new JsonReader(new InputStreamReader(in, "UTF-8"));
    List<Message> messages = new ArrayList<Message>();
    reader.beginArray();
    while (reader.hasNext()) {
        Message message = gson.fromJson(reader, Message.class);
        messages.add(message);
    }
    reader.endArray();
    reader.close();
    return messages;
}

// 序列化
public void writeJsonStream(OutputStream out, List<Message> messages) throws IOException {
    JsonWriter writer = new JsonWriter(new OutputStreamWriter(out, "UTF-8"));
    writer.setIndent("  ");
    writer.beginArray();
    for (Message message : messages) {
        gson.toJson(message, Message.class, writer);
    }
    writer.endArray();
    writer.close();
}
```
# 5 反射
大部分框架或者SDK为了解耦，或多或少的会使用到反射，反射本身会性能就会比直接调用差很多。

有时候会大批量使用反射去实例化对象，所以不要在那些会被反射实例化的类的构造函数中做太多的事情。

建议在获取实例化对象后，再使用对象直接调用初始化方法。
# 6 异常

在代码中有时候可能会发生异常，所以一般使用try/catch来做保护，避免程序崩溃。

一旦有异常发生，系统会耗费资源去处理，本身对内存和CPU就会有消耗。

所以不能因为有了try/catch而不顾代码质量，要尽量保证没有大量的异常出现，我在项目中见过大量空指针异常出现，这种异常我们可以在代码层面就直接避免。

# 7 频繁GC

频繁的GC也会对性能造成影响，严重的会导致卡顿或者ANR。

频繁GC原因有两个

内存抖动，大量的对象被创建又在短时间内马上被释放
瞬间产生大量的对象会严重占用Young Generation的内存区域，当达到阀值，剩余空间不够的时候，也会触发GC。即使每次分配的对象占用了很少的内存，但是他们叠加在一起会增加 Heap的压力，从而触发更多其他类型的GC。
在项目中避免短时间内突然创建大量的对象。

# 8 设备&应用基本信息

获取设备&应用基本信息，比如说包名、IMEI信息等等，是比较耗时的操作，可以进行一些优化。

固定信息获取采用缓存策略，例如版本号、版本名称、手机mac地址、运营商信息等等，一次获取，缓存到内存，下次直接使用
不部分不固定信息，例如网络情况（2G/3G/4G/WIFI），可以采用监听网络变更方式，来即时更新网络情况
其他不固定信息，比如基站信息，只能每次获取

# 9 使用SparesArray/ArrayMap替代HashMap
SparseArray是android里为这样的Hashmap而专门写的类,目的是提高效率，其核心是折半查找函数。主要是因为SparseArray不需要对key和value进行auto-boxing（将原始类型封装为对象类型，比如把int类型封装成Integer类型），结构比HashMap简单（SparseArray内部主要使用两个一维数组来保存数据，一个用来存key，一个用来存value）不需要额外的数据结构（主要是针对HashMap中的HashMapEntry而言的）。  
ArrayMap是一个<key,value>映射的数据结构，它设计上更多的是考虑内存的优化，内部是使用两个数组进行数据存储，一个数组记录key的hash值，另外一个数组记录Value值，它和SparseArray一样，也会对key使用二分法进行从小到大排序，在添加、删除、查找数据的时候都是先使用二分查找法得到相应的index，然后通过index来进行添加、查找、删除等操作，所以，应用场景和SparseArray的一样，如果在数据量比较大的情况下，那么它的性能将退化至少50%。
# 10 使用单例
给一种很极客的方式：
```
public static class SingleInstance {   
    private SingleInstance() {   
    }   
   
   public static SingleInstance getInstance() {   
            return SingleInstanceHolder.sInstance;   
   }   
   
  private static class SingleInstanceHolder {   
            private static SingleInstance sInstance = new SingleInstance();   
  }   
}   
```

# 11 避免隐私装箱
比如：
```
Integer sum = 0;   
  for (int i = 1000; i < 5000; i++) {   
        sum += i;   
}  
```
其内部变化如下:
```
int result = sum.intValue() + i;   
Integer sum = new Integer(result);
```
# 12 谨慎选用容器
Java和Android提供了很多编辑的容器集合来组织对象。比如ArrayList(默认是10，加载因子是0.5，一次扩充之后是原理容量*0.5+1),ContentValues,HashMap（默认是16，加载因子是0.75，扩充2倍），HashSet（默认初始容量为16，加载因子为0.75，扩充一倍）等。
# 13 用好LaunchMode 
因为默认是standard，可以考虑singletask,
# 14 注意字符串拼接
考虑使用StringBuilder
# 15 提前检查，减少不必要的异常
异常对于程序来说，在平常不过了，然后其实异常的代码很高的，因为它需要收集现场数据stacktrace。但是还是有一些避免异常抛出的措施的，那就是做一些提前检查。
比如，我们想要打印一个文件的每一行字符串，没做检查的代码如下，是存在FileNotFoundException抛出可能的。
```
private void printFileByLine(String filePath) {   
   try {   
      FileInputStream inputStream = new FileInputStream("textfile.txt");   
      BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));   
      String strLine;   
      //Read File Line By Line   
      while ((strLine = br.readLine()) != null) {   
          // Print the content on the console   
          System.out.println(strLine);   
      }   
        br.close();   
    } catch (FileNotFoundException e) {   
       e.printStackTrace();   
    } catch (IOException e) {   
        e.printStackTrace();   
   }   
}   
```
如果我们进行文件是否存在的检查，抛出FileNotFoundException的概率会减少很多，
```
private void printFileByLine(String filePath) {   
   if (!new File(filePath).exists()) {   
            return;   
   }   
    try {   
       FileInputStream inputStream = new FileInputStream("anonymous.txt");   
       BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));   
       String strLine;   
      //Read File Line By Line   
       while ((strLine = br.readLine()) != null) {   
            // Print the content on the console   
           System.out.println(strLine);   
       }   
         br.close();   
    } catch (FileNotFoundException e) {   
        e.printStackTrace();   
    } catch (IOException e) {   
        e.printStackTrace();   
   }   
}   
```

# 16 使用注解替代枚举
枚举是我们经常使用的一种用作值限定的手段，使用枚举比单纯的常量约定要靠谱。然后枚举的实质还是创建对象。好在Android提供了相关的注解，使得值限定在编译时进行，进而减少了运行时的压力。相关的注解为IntDef和StringDef。
# 17 选用对象池。
位于android.support.v4.util.Pool包下,经常会在ThreadPool 和 MessagePool看到。
在Android中有很多池的概念，如线程池，连接池。包括我们很长用的Handler.Message就是使用了池的技术。
比如，我们想要使用Handler发送消息，可以使用Message msg = new Message()，也可以使用Message msg = handler.obtainMessage()。使用池并不会每一次都创建新的对象，而是优先从池中取对象。
使用对象池需要需要注意几点
- 将对象放回池中，注意初始化对象的数据，防止存在脏数据
- 合理控制池的增长，避免过大，导致很多对象处于闲置状态
 
# 18 谨慎初始化Application
Android应用可以支持开启多个进程。 通常的做法是这样
```
<service   
   android:name=".NetworkService"   
   android:process=":network"/>
```
通常我们在Application的onCreate方法中会做很多初始化操作,但是每个进程启动都需要执行到这个onCreate方法,为了避免不必要的初始化,建议按照进程(通过判断当前进程名)对应初始化.
```
public class MyApplication extends Application {   
   private static final String LOGTAG = "MyApplication";   
   
     @Override   
     public void onCreate() {   
        String currentProcessName = getCurrentProcessName();   
        Log.i(LOGTAG, "onCreate currentProcessName=" + currentProcessName);   
           super.onCreate();   
           if (getPackageName().equals(currentProcessName)) {   
                //init for default process   
           } else if (currentProcessName.endsWith(":network")) {   
                //init for netowrk process   
           }   
     }   
   
    private String getCurrentProcessName() {   
        String currentProcessName = "";   
        int pid = android.os.Process.myPid();   
        ActivityManager manager = (ActivityManager) this.getSystemService(Context.ACTIVITY_SERVICE);   
        for (ActivityManager.RunningAppProcessInfo processInfo : manager.getRunningAppProcesses()) {   
            if (processInfo.pid == pid) {   
                    currentProcessName = processInfo.processName;   
                    break;   
            }   
       }   
      return currentProcessName;   
    }   
}   
```
# 19 尽量为所有分辨率创建资源
[减少不必要的硬件缩放，这会降低UI的绘制速度，可借助Android asset studio]( http://android-ui-utils.googlecode.com/hg/asset-studio/dist/index.html/)
# 20 SurfaceView
可以在主线程之外的线程中向屏幕绘图上。这样可以避免画图任务繁重的时候造成主线程阻塞，从而提高了程序的反应速度。在游戏开发中多用到SurfaceView，游戏中的背景、人物、动画等等尽量在画布canvas中画出。  
# 21 foreach和一般for的使用
数组使用一般for效率高，链表使用foreach效率高。foreach相当于实现了iterator.
# 22 tyr的优化
有些地方空指针添加一个判空就好了，没必要抛出一个空指针异常。try里面的代码块能少则少。
# 23 使用实类比接口好
HashMap mMap = new HashMap();比Map mMap = new HashMap()好，嵌入式系统调用一个接口的引用会比调用实体类的引用多花费一倍的时间。
# 24 将成员变量缓存到本地
访问成员变量比访问本地变量慢的多，比如：
```
for (int i = 0; i < this.mCount; i++){
    dumpItem(this.mItems[i]);
}
```
最好修改如下：
```
int count = this.mCount;
Item[] items = this.mItems;
for (int i = 0; i<count;i++){
    dumpItem(items[i]);
}
```
另一个相似的原则：永远不要在for的第二个条件调用任何方法。
# 25 避免使用枚举
会牺牲执行的速度和并大幅度增加文件体积
# 总结
1. 关于性能优化方面知识，官方也出过视频A[ndroid Performance Patterns](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)
2. [java/android 设计模式学习笔记（5）---对象池模式](http://blog.csdn.net/self_study/article/details/51477002)
3. [【Android优化】最强ListView优化方案](http://blog.csdn.net/s003603u/article/details/47261393)
