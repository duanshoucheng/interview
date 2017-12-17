# 一、DDMS
打开姿势：Tools->Android->Android Device Monitor。  
DDMS全称Dalvik Debug Monitor Service。  
DDMS的作用它提供截屏，查看线程和堆的信息，logcat，进程，广播状态信息，模拟来电呼叫和短信，虚拟地理坐标等等。  
下面很多工具都在DDMS里可以找到。

# 二、Hierarchy（DDMS）
可以参考：[Android Studio上使用可视化调试工具Hierarchy Viewer](http://blog.csdn.net/cdw_wy/article/details/53240885)
。

# 三、TraceView的使用（DDMS）
>主要分析traces.txt文件的，可惜Android Studio3.0编辑页面不显示该文件了。但DDMS还可以用.不但可以分析ANR，也可以分享卡顿问题。

生成traces.txt有两种方式：
- 通过DDMS(tools->android->Android Device Monitor)选择一个进程，然后按上面的“Start Method Profiling”按钮，等红色小点变成黑色以后就表示TraceView已经开始工作了。然后我就可以滑动一下列表（现在手机上的操作肯定会很卡，因为Android系统在检测Dalvik虚拟机中每个Java方法的调用，这是我猜测的）。操作最好不要超过5s，因为最好是进行小范围的性能测试。然后再按一下刚才按的按钮，等一会就会出现上面这幅图，然后就可以开始分析了。
- 第2种方式就是使用android.os.Debug.startMethodTracing();和android.os.Debug.stopMethodTracing();方法，当运行了这段代码的时候，就会有一个trace文件在/sdcard目录中生成，也可以调用startMethodTracing(String traceName) 设置trace文件的文件名，最后你可以使用adb pull /sdcard/test.trace /tmp 命令将trace文件复制到你的电脑中，然后用DDMS工具打开就会出现第一幅图了

>第一种方式相对来说是一种简单，但是测试的范围很宽泛，第二中方式相对来说精确一点，不过我个人喜欢使用第一种，因为简单，而且它是检测你的某一个操作。因为第二中更适合检测某一个方法的性能，其实也没有那种好，看使用的场景和喜好了。。。

看懂TraceView中的指标:

![image](http://note.youdao.com/yws/public/resource/9cd56f6819876ec80563ab61665a91d7/xmlnote/C2E19CA55CB14AEE8F72FCC92CD021EC/8425)
首先解除这些意思：[link](http://blog.jobbole.com/78995/)
# 四、内存泄露工具：LeakCannary
JAVA是垃圾回收语言的一种，开发者无需特意管理内存分配。但是JAVA中还是存在着许多内存泄露的可能性，如果不好好处理内存泄露，会导致APP内存单元无法释放被浪费掉，最终导致内存全部占据堆栈(heap)挤爆进而程序崩溃  
[官网:LeakCanary 中文使用说明](https://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)  
使用步骤：  
### 1、在 build.gradle 中加入引用，不同的编译使用不同的引用：
```
 dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3'
 }
 ```
在 Application 中：
```
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    LeakCanary.install(this);
  }
}
```
这样，就万事俱备了！ 在 debug build 中，如果检测到某个 activity 有内存泄露，LeakCanary 就是自动地显示一个通知。

### 2、监控Activity
```
public class ExampleApplication extends Application {

  public static RefWatcher getRefWatcher(Context context) {
    ExampleApplication application = (ExampleApplication) context.getApplicationContext();
    return application.refWatcher;
  }

  private RefWatcher refWatcher;

  @Override public void onCreate() {
    super.onCreate();
    refWatcher = LeakCanary.install(this);
  }
}
```
### 3、监控Fragment
```
public abstract class BaseFragment extends Fragment {

  @Override public void onDestroy() {
    super.onDestroy();
    RefWatcher refWatcher = ExampleApplication.getRefWatcher(getActivity());
    refWatcher.watch(this);
  }
}
```

### 4、最新版本
在6.0以上的版本中会弹出异常，log里也会打印出空指针。
在最新版本已经修复了这个问题。
```
java.lang.NullPointerException: Attempt to invoke virtual method 'boolean java.lang.String.equals(java.lang.Object)' on a null object reference at com.squareup.leakcanary.HeapAnalyzer.findLeakingReference(HeapAnalyzer.java:160)
```
新更新的SDK 1.5版本，已经完美解决、android6.0以上运行时权限，Notification无法弹出的问题，需要重新配置   
build.gradle:
```
dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
   testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
 }
```
application:
```
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
      // This process is dedicated to LeakCanary for heap analysis.
      // You should not init your app in this process.
      return;
    }
    LeakCanary.install(this);
    // Normal app init code...
  }
}
```
# 五、查找卡顿工具：BlockCannary
BlockCannary参数的解读：
- cpuCore：手机cpu个数
- processName：应用包名。
- freeMemory: 手机剩余内存,单位KB。
- timecost: 该Message(事件)执行时间，单位 ms。
- threadtimecost: 该Message(事件)执行线程时间（线程实际运行时间，不包含别的线程占用cpu时间），单位 ms。
- cpubusy: true表示cpu负载过重，false表示cpu负载不重。cpu负载过重导致该Message(事件) 超时，错误不在本事件处理上。

用法如下:  
第一步配置：
```
//compile 'com.github.markzhai:blockcanary-android:1.5.0'
debugCompile'com.github.markzhai:blockcanary-android:1.5.0'
releaseCompile 'com.github.markzhai:blockcanary-no-op:1.5.0'
```
第二步：
在Application的onCreate（）方法里加入：
```
BlockCanary.install(this, new AppBlockCanaryContext()).start();
```
第三步：
监视应用程序的标签和图标可以通过在xhdpi drawable目录和strings.xml中放置一个可以绘制的块金丝雀图来配置：
```
/**
 * 实现各种上下文，包括应用标示符，用户 uid，网络类型，卡慢判断阙值，Log 保存位置等
 */

public class AppBlockCanaryContext  extends BlockCanaryContext {

    /**
     * Implement in your project.
     *
     * @return Qualifier which can specify this installation, like version + flavor.
     */
    public String provideQualifier() {
        return "unknown";
    }

    /**
     * Implement in your project.
     *
     * @return user id
     */
    public String provideUid() {
        return "uid";
    }
  ......
}
```
第四步：运行Demo即可
