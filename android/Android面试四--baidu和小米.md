12.14
# HTTPS的三次握手，怎么把秘钥传递过去的
非对称加密算法：RSA，DSA/DSS;对称加密算法：AES，RC4，3DES;HASH算法：MD5，SHA1，SHA256 .记住RSA、AES、DES是对称还是非对称，以及HASH这几种算法  
HTTPS解决的是信息传输中数据被篡改、窃取。HTTPS通过对称和非对称实现安全的，因为非对称效率比较低，所以是通过非对称传递对称加密的秘钥，使用非对称加密来实现数据传输的。
[Https简介](https://github.com/duanshoucheng/interview/blob/master/java/http%E5%92%8Chttps%E7%AE%80%E4%BB%8B.md)

# 静态类有什么作用
 如果一个类要被声明为static的，只有一种情况，就是静态内部类。  
 静态内部类实际上与普通类（即类名必须与文件名一样的顶级类）一样，只是静态内部类在某一类的内部定义了而已，既然是类，要想使用就必须实例化。   
 概念上与静态变量、静态方法是不一样的，不要被“静态”两个字迷惑了（不要以为凡是静态的东西就不需要实例化就可以直接使用，静态内部类是有区别）， 而且只有静态内部类，而没有静态类（顶级类）的概念。
 [静态类的特点及用途](http://blog.csdn.net/chpdirector84/article/details/5406978)，主要功能：
 - 它们仅包含静态成员。
 - 它们不能被实例化。
 - 它们是密封的。
 - 它们不能包含实例构造函数（C# 编程指南）。
 - 静态类不能使用abstract或sealed修饰符。
 - 静态类默认继承自System.Object根类，不能显式指定任何其他基类。
 - 静态类不能指定任何接口实现。
 - 静态类的成员不能有protected或protected internal访问保护修饰符。
# Lambda有什么优点
- 费神去命名一个函数
- 享受了一些函数式编程的优势
 
请 继续补充
# 并发包
# 线程池，如果超出最大线程数怎么办
# 线程安全，生产者消费者，两个线程同时读数据，valite和sync, leisuo对象锁
# 性能分析：Hprof+MAT工具，debug怎么生成这个文件，问的比较多，关于性能分析，重点（profile文件分分析，怎么上传自动生成，没有问）
# MVP和MVC的区别，最好知道写MVVM
# 单链表翻转，查找环的位置。快排、插入、堆、二叉树
# animotor,animotion
# ReactNative，有什么优势，为什么用这个
# NodeJS 
# 进程间通信（问的也比较多），broadcasts的源码，进程间通信，四大组件也是进程间通信吗
这题主要是问android的进程间通信 

 broadcasts也可以跨进程广播。[Android四大组件：BroadcastReceiver史上最全面解析](http://www.jianshu.com/p/ca3d87a4cdf3) 
  原理描述：
- 广播接收者 通过 Binder机制在 AMS 注册
- 广播发送者 通过 Binder 机制向 AMS 发送广播
- AMS 根据 广播发送者 要求，在已注册列表中，寻找合适的广播接收者
- AMS将广播发送到合适的广播接收者相应的消息循环队列中；
- 广播接收者通过 消息循环 拿到此广播，并回调 onReceive()
 
 
附其他Linux下进程间通信有（可以了解下，知道当然加分）：
 - 管道( pipe )：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。
 - 有名管道 (named pipe) ： 有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信。
 - 信号量( semophore ) ： 信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
 - 消息队列( message queue ) ： 消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。
 - 信号 ( sinal ) ： 信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。
 - 共享内存( shared memory ) ：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号两，配合使用，来实现进程间的同步和通信。
 - 套接字( socket ) ： 套解口也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同及其间的进程通信。
# 进程锁有哪些？只知道有个文件锁
 线程锁 有Synchronize和volatile和lock。
 类的对象实例可以有很多个，不同对象实例的对象锁是互不干扰的，所以只有多个线程访问一个对象时才有意义，否则给一个对象家线程锁是没意义的。但是每个类只有一个类锁。但是有一点必须注意的是，其实类锁只是一个概念上的东西，并不是真实存在的，它只是用来帮助我们理解锁定实例方法和静态方法的区别的。
  入门参考：[Java线程(一)：线程安全与不安全](http://blog.csdn.net/ghsau/article/details/7421217)    
  [Java对象锁和类锁全面解析（多线程synchronized关键字](http://www.importnew.com/20444.html)  
  需要特别说明：对于同一个类A，线程1争夺A对象实例的对象锁，线程2争夺类A的类锁，这两者不存在竞争关系。也就说对象锁和类锁互补干预内政
  [Java并发编程：Lock](https://www.cnblogs.com/dolphin0520/p/3923167.html)  
  文件锁是进程锁。  
  当线程操作某个对象时，执行顺序如下： (1) 从主存复制变量到当前工作内存 (read and load) (2) 执行代码，改变共享变量值 (use and assign) (3) 用工作内存数据刷新主存相关内容 (store and write)

一个线程执行临界区代码过程如下： 1 获得同步锁 2 清空工作内存 3 从主存拷贝变量副本到工作内存 4 对这些变量计算 5 将变量从工作内存写回到主存 6 释放锁 可见，synchronized既保证了多线程的并发有序性，又保证了多线程的内存可见性。

volatile是java提供的一种同步手段，只不过它是轻量级的同步，为什么这么说，因为volatile只能保证多线程的内存可见性，不能保证多线程的执行有序性。而最彻底的同步要保证有序性和可见性，例如synchronized。任何被volatile修饰的变量，都不拷贝副本到工作内存，任何修改都及时写在主存。因此对于Valatile修饰的变量的修改，所有线程马上就能看到，但是volatile不能保证对变量的修改是有序的。所以简单来说，volatile适合这种场景：一个变量被多个线程共享，线程直接给这个变量赋值。这是一种很简单的同步场景，这时候使用volatile的开销将会非常小。
# handler源码以及原理（不常问，可以参考线程Apost是如何发一个任务到线程B），loop没数据的时候不阻塞吗
[源码分析Android Handler是如何实现线程间通信的](https://www.cnblogs.com/angrycode/p/6576905.html)

Looper.loop()无限循环中有个Message msg = queue.next();然后进入queue.next()查看源码：
```
 Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
```
nativePollOnce(ptr, nextPollTimeoutMillis)是native方法，它在内部调用了epoll_wait()，这个方法实现流程大概是有message就返回，没有的话就一直等待并且监听管道的事件；当Java层有新的message（或者barrier）被添加至messageQueue时，会触发native方法nativeWake，这个方法的里面大概是往管道中写入一个“w”，这样就会唤醒epoll_wait，继而使nativePollOnce方法返回，继续执行以后的部分。  
所以如果队列中没有消息，线程就会进入空闲等待状态，等待下一个消息的到来。 
# 桌面icon启动过程
# Activity的启动栈，怎么固定到一个栈（Manifest的属性）
# 软引用、弱引用，到底回收吗，如果还持有应用呢，就不回收了吗，岂不是和强引用一样了。
如果一个对象只具有软引用，那么垃圾回收器在内存充足的时候不会回收它，而在内存不足时会回收这些对象。软引用对象被回收后，Java 虚拟机会把这个软引用加入到与之关联的引用队列中。  
如果一个对象只具有弱引用，那么垃圾回收器在扫描到该对象时，无论内存充足与否，都会回收该对象的内存。与软引用相同，弱引用对象被回收后，Java 虚拟机会把这个弱引用加入到与之关联的引用队列中。
弱引用解决内存泄露问题。用 WeakHashMap 来作为缓存的容器可以有效解决这一问题。WeakHashMap 和 HashMap 几乎一样，唯一的区别就是它的键使用弱引用。当 WeakHashMap 的键标记为过期时，这个键对应的条目就会自动被移除。这就避免了内存泄漏问题。
# 事件分发机制（必问问题）
  郭神有个事件分发机制图可以帮助记忆
 [ Android事件分发机制完全解析，带你从源码的角度彻底理解(上)](http://blog.csdn.net/guolin_blog/article/details/9097463/),当然还有下。  
  [Android ViewGroup事件分发机制](http://blog.csdn.net/lmj623565791/article/details/39102591)
# Dragger2为什么不影响性能？变成了二进制？google后台服务器也实现了，怎么实现的注入
# 基本算法都会写
# postInvalidate()、invalidate()、requestLayout()的区别，什么时候调用onLayout()，都做了哪些操作
# draw()和onDraw()的区别，调用子的绘制的时候在哪里实现的
# 计算大图的大小？加载大图可以使用resize的方法
# greenDAO的事物是怎么样实现的
# 对称加密和非对称加密，（so文件破解的问题，很容易）
# reyclerview是怎么实现缓存的，多布局的呢
# 了解运营的数据
DAU (Daily Active User) 即日活跃用户数量，统计的是一日之内，登录或使用过某个应用的用户数（去除重复登录的用户 ）。
WAU (Weekly Active User) 即周活跃用户数量，是指在一周之内登录或使用该应用的用户数量。
MAU（Monthly Active Users）即月活跃用户数，指的是在一个月中至少使用过一次该应用的独立用户数量。
UV（Unique Vister）独立访客。
UED：用户体验设计，比较装逼的岗位
PV（page view）页面浏览的总次数。
首发：分为独家首发和联合首发，独家首发是指App新版本第一个选择的分发渠道，期间只在指定的市场进行新版本发布，其 他渠道的发布时间至少须晚于首发市场24小时。联合首发是指在多个应用市场同步进行新版本发布。
# 怎么自己实现hashmap和ArrayList,hashTable是怎样实现线程安全的。（比较少见）
 [java——HashMap的实现原理，自己实现简单的HashMap](https://www.cnblogs.com/xcr1234/p/6187663.html)
 [HashMap的容量与扩容](http://blog.csdn.net/gaopu12345/article/details/50831631);  
 [调试JDK源码-一步一步看HashMap怎么Hash和扩容](http://blog.csdn.net/unix21/article/details/50911387)  
 [调试JDK源码-Hashtable实现原理以及线程安全的原因](http://blog.csdn.net/unix21/article/details/50920708)  
 [Java线程(八)：锁对象Lock-同步问题更完美的处理方式](http://blog.csdn.net/ghsau/article/details/7461369/)
 hashTable其实就是因为这个put方法是synchronized的所以可以保证其线程安全。
# 二叉树的问题看面试官，有的面试官很鄙视这样的问题，有的面试官会觉得很有面
# 内存泄露有哪些
# AAR怎么实现自己混淆，而不用用户自己再做keep等相关的问题
# OKhttp怎么实现缓存的
# Glide怎么实现缓存(更早之前有个面试官问过怎么实现生命周期的问题)
# 不使用属性动画，怎么实现该功能：补间动画+重新布局
# 为什么离职（重要，可以说公司项目趋于稳定，最近比较空闲。或者说公司没什么起色）
# 觉得自己比较熟悉的（可以稍微重视些）
# 项目中遇到的困难的（重要，最好提前准备）
# JS和java相互调用的问题，WebView怎么实现的缓存，如果前端没有做缓存，客户端是否能自己实现（应该可以，说了大致的方案） 
# 热更新，怎么给别人的APP打log，怎么在别人的应用上添加自己的应用，不能使用反编译
