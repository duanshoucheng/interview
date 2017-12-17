# 一、DDMS
打开姿势：Tools->Android->Android Device Monitor。  
DDMS全称Dalvik Debug Monitor Service。  
DDMS的作用它提供截屏，查看线程和堆的信息，logcat，进程，广播状态信息，模拟来电呼叫和短信，虚拟地理坐标等等。  
下面很多工具都在DDMS里可以找到。

# 二、Hierarchy（DDMS）
可以参考：[Android Studio上使用可视化调试工具Hierarchy Viewer](http://blog.csdn.net/cdw_wy/article/details/53240885)
。

# TraceView的使用（DDMS）
>主要分析traces.txt文件的，可惜Android Studio3.0编辑页面不显示该文件了。但DDMS还可以用.不但可以分析ANR，也可以分享卡顿问题。

生成traces.txt有两种方式：
- 通过DDMS(tools->android->Android Device Monitor)选择一个进程，然后按上面的“Start Method Profiling”按钮，等红色小点变成黑色以后就表示TraceView已经开始工作了。然后我就可以滑动一下列表（现在手机上的操作肯定会很卡，因为Android系统在检测Dalvik虚拟机中每个Java方法的调用，这是我猜测的）。操作最好不要超过5s，因为最好是进行小范围的性能测试。然后再按一下刚才按的按钮，等一会就会出现上面这幅图，然后就可以开始分析了。
- 第2种方式就是使用android.os.Debug.startMethodTracing();和android.os.Debug.stopMethodTracing();方法，当运行了这段代码的时候，就会有一个trace文件在/sdcard目录中生成，也可以调用startMethodTracing(String traceName) 设置trace文件的文件名，最后你可以使用adb pull /sdcard/test.trace /tmp 命令将trace文件复制到你的电脑中，然后用DDMS工具打开就会出现第一幅图了

>第一种方式相对来说是一种简单，但是测试的范围很宽泛，第二中方式相对来说精确一点，不过我个人喜欢使用第一种，因为简单，而且它是检测你的某一个操作。因为第二中更适合检测某一个方法的性能，其实也没有那种好，看使用的场景和喜好了。。。

看懂TraceView中的指标:

![image](http://note.youdao.com/yws/public/resource/9cd56f6819876ec80563ab61665a91d7/xmlnote/C2E19CA55CB14AEE8F72FCC92CD021EC/8425)
首先解除这些意思：[link](http://blog.jobbole.com/78995/)