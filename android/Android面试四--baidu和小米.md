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
# 进程间通信
# 进程锁有哪些？只知道有个文件锁
  入门参考：[Java线程(一)：线程安全与不安全](http://blog.csdn.net/ghsau/article/details/7421217)  
  [Java对象锁和类锁全面解析（多线程synchronized关键字](ttp://www.importnew.com/20444.html)  
  需要特别说明：对于同一个类A，线程1争夺A对象实例的对象锁，线程2争夺类A的类锁，这两者不存在竞争关系。也就说对象锁和类锁互补干预内政
# handler源码，loop没数据的时候不阻塞吗，Handler的原理，线程Apost一个任务到线程B
# 桌面icon启动过程
# Activity的启动栈，怎么固定到一个栈（Manifest的属性）
# 软引用、弱引用，到底回收吗，如果还持有应用呢，就不回收了吗，岂不是和强引用一样了
# 事件分发机制
# Dragger2为什么不影响性能？变成了二进制？google后台服务器也实现了，怎么实现的注入
# broadcasts的源码，进程间通信，四大组件也是进程间通信吗
# 基本算法都会写
# postInvalidate()、invalidate()、requestLayout()的区别，什么时候调用onLayout()，都做了哪些操作
# draw()和onDraw()的区别，调用子的绘制的时候在哪里实现的
# 计算大图的大小？加载大图可以使用resize的方法
# greenDAO的事物是怎么样实现的
# 对称加密和非对称加密，（so文件破解的问题，很容易）
# reyclerview是怎么实现缓存的，多布局的呢
# 了解运营的数据
# 怎么自己实现hashmap和ArrayList,hashTable是怎样实现线程安全的。（比较少见）
 [java——HashMap的实现原理，自己实现简单的HashMap](https://www.cnblogs.com/xcr1234/p/6187663.html)
 [HashMap的容量与扩容](http://blog.csdn.net/gaopu12345/article/details/50831631);  
 [调试JDK源码-一步一步看HashMap怎么Hash和扩容](http://blog.csdn.net/unix21/article/details/50911387)  
 [调试JDK源码-Hashtable实现原理以及线程安全的原因](http://blog.csdn.net/unix21/article/details/50920708)  
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
