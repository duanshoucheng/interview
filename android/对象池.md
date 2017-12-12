 #  1、为什么需要对象池
对象池类的成员应该都是静态的。用户也不应该能访问池子里装着的对象的构造函数，以防用户绕开对象池创建实例。
Java对象的生命周期大致包括三个阶段：对象的创建，对象的使用，对象的清除。
Java对象是通过构造函数来创建的，在这一过程中，该构造函数链中的所有构造函数也都会被自动调用。另外，默认情况下，调用类的构造函数时，Java会把变量初始化成确定的值：所有的对象被设置成null，整数变量（byte、short、int、long）设置成0，float和double变量设置成0.0，逻辑值设置成false。

java 虽然可以自己回收内存，但同时它也带来了较大的性能开销。这种开销包括两方面，首先是对象管理开销，GC为了能够正确释放对象，它必须监控每一个对象的运行状态，包括对象的申请、引用、被引用、赋值等。其次，在GC开始回收“垃圾”对象时，系统会暂停应用程序的执行，而独自占用CPU。

因此，如果要改善应用程序的性能，一方面应尽量减少创建新对象的次数；同时，还应尽量减少对象的创建、对象的清除的时间，而这些均可以通过对象池技术来实现。
 
 #  2、对象池技术的基本原理
对象池技术基本原理的核心有两点：缓存和共享，即对于那些被频繁使用的对象，在使用完后，不立即将它们释放，而是将它们缓存起来，以供后续的应用程序重复使用，从而减少创建对象和释放对象的次数，进而改善应用程序的性能。
并非所有对象都适合拿来池化――因为维护对象池也要造成一定开销。对生成时开销不大的对象进行池化，反而可能会出现“维护对象池的开销”大于“生成新对象的开销”，从而使性能降低的情况。但是对于生成时开销可观的对象，池化技术就是提高性能的有效策略了。
```
public class ObjectPool {       
    private int numObjects = 10; // 对象池的大小       
    private int maxObjects = 50; // 对象池最大的大小       
    private Vector objects = null; //存放对象池中对象的向量( PooledObject类型)         
    
    public ObjectPool() {              
    }       
      
    /*** 创建一个对象池***/       
    public synchronized void createPool(){       
        // 确保对象池没有创建。如果创建了，保存对象的向量 objects 不会为空       
        if (objects != null) {       
            return; // 如果己经创建，则返回       
        }       
      
        // 创建保存对象的向量 , 初始时有 0 个元素       
        objects = new Vector();       
      
        // 根据 numObjects 中设置的值，循环创建指定数目的对象       
        for (int x = 0; x < numObjects; x++) {       
           if ((objects.size() == 0)&&this.objects.size() <this.maxObjects) {    
              Object obj = new Obj();       
              objects.addElement(new PooledObject(obj));                 
　　　　　　}  
　　　　}  
    }       
      
    public synchronized Object getObject(){       
        // 确保对象池己被创建       
        if (objects == null) {       
            return null; // 对象池还没创建，则返回 null       
        }       
      
        Object conn = getFreeObject(); // 获得一个可用的对象       
      
        // 如果目前没有可以使用的对象，即所有的对象都在使用中       
        while (conn == null) {       
            wait(250);       
            conn = getFreeObject(); // 重新再试，直到获得可用的对象，如果       
            // getFreeObject() 返回的为 null，则表明创建一批对象后也不可获得可用对象       
        }       
      
        return conn;// 返回获得的可用的对象       
    }       
      
    /**    
     * 本函数从对象池对象 objects 中返回一个可用的的对象，如果    
     * 当前没有可用的对象，则创建几个对象，并放入对象池中。    
     * 如果创建后，所有的对象都在使用中，则返回 null    
     */      
    private Object getFreeObject(){       
      
        // 从对象池中获得一个可用的对象       
        Object obj = findFreeObject();       
      
        if (obj == null) {       
            createObjects(incrementalObjects);     //如果目前对象池中没有可用的对象，创建一些对象       
  
            // 重新从池中查找是否有可用对象       
            obj = findFreeObject();       
                     
           // 如果创建对象后仍获得不到可用的对象，则返回 null       
            if (obj == null) {       
                return null;       
            }       
        }       
      
        return obj;       
    }       
      
    /**    
     * 查找对象池中所有的对象，查找一个可用的对象，    
     * 如果没有可用的对象，返回 null    
     */      
    private Object findFreeObject(){       
      
        Object obj = null;       
        PooledObject pObj = null;       
      
        // 获得对象池向量中所有的对象       
        Enumeration enumerate = objects.elements();       
      
        // 遍历所有的对象，看是否有可用的对象       
        while (enumerate.hasMoreElements()) {       
            pObj = (PooledObject) enumerate.nextElement();       
                      
           // 如果此对象不忙，则获得它的对象并把它设为忙       
            if (!pObj.isBusy()) {       
                obj = pObj.getObject();       
                pObj.setBusy(true);       
           }  
     
        return obj;// 返回找到到的可用对象       
    }       
      
      
    /**    
     * 此函数返回一个对象到对象池中，并把此对象置为空闲。    
     * 所有使用对象池获得的对象均应在不使用此对象时返回它。    
     */      
      
    public void returnObject(Object obj) {       
      
        // 确保对象池存在，如果对象没有创建（不存在），直接返回       
        if (objects == null) {       
            return;       
        }       
      
        PooledObject pObj = null;       
      
        Enumeration enumerate = objects.elements();       
      
        // 遍历对象池中的所有对象，找到这个要返回的对象对象       
        while (enumerate.hasMoreElements()) {       
            pObj = (PooledObject) enumerate.nextElement();       
  
            // 先找到对象池中的要返回的对象对象       
            if (obj == pObj.getObject()) {       
                // 找到了 , 设置此对象为空闲状态       
                pObj.setBusy(false);       
                break;       
            }       
        }       
    }       
      
      
    /**    
     * 关闭对象池中所有的对象，并清空对象池。    
     */      
    public synchronized void closeObjectPool() {       
      
        // 确保对象池存在，如果不存在，返回       
        if (objects == null) {       
            return;       
        }       
      
        PooledObject pObj = null;       
      
        Enumeration enumerate = objects.elements();       
      
        while (enumerate.hasMoreElements()) {       
      
            pObj = (PooledObject) enumerate.nextElement();       
      
            // 如果忙，等 5 秒       
            if (pObj.isBusy()) {       
                wait(5000); // 等 5 秒       
            }       
      
            // 从对象池向量中删除它       
            objects.removeElement(pObj);       
        }       
      
        // 置对象池为空       
        objects = null;       
    }       
      
      
    /**    
     * 使程序等待给定的毫秒数    
     */      
    private void wait(int mSeconds) {       
        try {       
            Thread.sleep(mSeconds);       
        }  
       catch (InterruptedException e) {       
        }       
    }       
      
     
    /**    
     * 内部使用的用于保存对象池中对象的类。    
     * 此类中有两个成员，一个是对象，另一个是指示此对象是否正在使用的标志 。 
     */      
    class PooledObject {       
      
        Object objection = null;// 对象       
        boolean busy = false; // 此对象是否正在使用的标志，默认没有正在使用       
      
        // 构造函数，根据一个 Object 构告一个 PooledObject 对象       
        public PooledObject(Object objection) {       
      
            this.objection = objection;       
      
        }       
      
        // 返回此对象中的对象       
        public Object getObject() {       
            return objection;       
        }       
      
        // 设置此对象的，对象       
        public void setObject(Object objection) {       
            this.objection = objection;       
      
        }       
      
        // 获得对象对象是否忙       
        public boolean isBusy() {       
            return busy;       
        }       
      
        // 设置对象的对象正在忙       
        public void setBusy(boolean busy) {       
            this.busy = busy;       
        }       
    }       
}      
   
   
测试类：  
代码如下：  
      
public class ObjectPoolTest {       
    public static void main(String[] args) throws Exception {       
        ObjectPool objPool = new ObjectPool();  
      
        objPool.createPool();       
        Object obj = objPool.getObject();       
        returnObject(obj);  
        objPool.closeObjectPool();       
    }       
}  
```

# 3、android的对象池---RecycledViewPool
 以给RecyclerView设置一个ViewHolder的对象池，这个池称为RecycledViewPool，这个对象池可以节省你创建ViewHolder的开销，更能避免GC。即便你不给它设置，它也会自己创建一个。
如果多个RecylerView间共用一个RecycledViewPool是不是能让你的UI更加的“顺滑”？  
RecycledViewPool使用起来也是非常的简单：先从某个RecyclerView对象中获得它创建的RecycledViewPool对象，或者是自己实现一个RecycledViewPool对象，然后设置个接下来创建的每一个RecyclerView即可。
需要注意的是，如果你使用的LayoutManager是LinearLayoutManager或其子类（如GridLayoutManager），需要手动开启这个特性：
layout.setRecycleChildrenOnDetach(true)
代码如下：
```
RecyclerView view1 = new RecyclerView(context);  
LinearLayoutManager layout = new LinearLayoutManager(context);  
layout.setRecycleChildrenOnDetach(true);  
view1.setLayoutManager(layout);  
RecycledViewPool pool = view1.getRecycledViewPool();  
//...  
  
RecyclerView view2 = new RecyclerView(context);  
//... (set layout manager)  
view2.setRecycledViewPool(pool);  
  
//...  
  
RecyclerView view3 = new RecyclerView(context);  
//...(set layout manager)  
view3.setRecycledViewPool(pool); 
```
>ViewPager中使用原理同上，只是你得通过ViewPagerAdapter传递个下一个RecyclerView。

# 4、android自带对象池android.support.v4.util.Pools
>别在循环里分配内存 (创建新对象)  
>尽量别在 View 的 onDraw 函数里分配内存(创建新对象)  
>实在无法避免在这些场景里分配内存时(创建新对象)，考虑使用对象池 (Object Pool)

源码很简单，主要有Pool接口、SimplePool、SynchronizedPool组成，官方也给出了demo，直接上我的使用封装：
```
import android.support.v4.util.Pools;
import com.sing.client.live_audio.entity.BaseChatMsgEntity;

/**
 * Created by mayi on 17/4/8.
 *
 * @Autor CaiWF
 * @Email 401885064@qq.com
 * @TODO UIGeter对象池
 */

public class UIGeterPoolModule {
    private Pools.SynchronizedPool<BaseChatMsgEntity> pool;
    private static UIGeterPoolModule uiGeterPoolModule;

    private UIGeterPoolModule() {
        pool = new Pools.SynchronizedPool<BaseChatMsgEntity>(55);
    }

    public static synchronized UIGeterPoolModule getInstance() {
        if (uiGeterPoolModule == null) {
            uiGeterPoolModule = new UIGeterPoolModule();
        }
        return uiGeterPoolModule;
    }

    public Pools.SynchronizedPool<BaseChatMsgEntity> getPool() {
        return pool;
    }

    //对象池中获取对象
    public static BaseChatMsgEntity getUIGetObject() {
        try {
            BaseChatMsgEntity baseChatMsgEntity = getInstance().getPool().acquire();
            return baseChatMsgEntity == null ? new BaseChatMsgEntity() : baseChatMsgEntity;
        } catch (Exception e) {
//            e.printStackTrace();
            return new BaseChatMsgEntity();
        }
    }

    //返回对象
    public static void returnObject(BaseChatMsgEntity uiGeter) {
        try {
            getInstance().getPool().release(uiGeter);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```
# 5、Android开发Message源码分析【享元模式|对象池】
 享元模式是对象池的一种实现，尽可能减少内存的使用，使用缓存来共享可用的对象，避免创建过多的对象。Android中Message使用的设计模式就是享元模式，获取Message通过obtain方法从对象池获取，Message使用结束通过recycle将Message归还给对象池，达到循环利用对象，避免重复创建的目的  
 Message中有关于对象池的属性如下：
 ```
 Message next; 
private static final Object sPoolSync = new Object(); 
private static Message sPool; 
private static int sPoolSize = 0; 
private static final int MAX_POOL_SIZE = 50
 ```
 采用链表的形式达到对象池的功能，next为指向对象池下一个对象的指针，sPool始终指向链表队首，sPoolSize表示当前链表数据量，MAX_POOL_SIZE表示对象池容量，sPoolSync专门用来给对象池上锁，达到互斥访问的目的。 
 使用obtain从对象池获取Message对象
```
 /**
     * Return a new Message instance from the global pool. Allows us to
     * avoid allocating new objects in many cases.
     */
    public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
```
对象池属于临界区，访问需要加锁，这里并没有使用synchronized修饰obtain方法，是为了减小锁的粒度。 
可见obtain方法首先判断对象池是否为空，空的话直接通过构造方法创建一个新的Message对象返回。不空的话从链表头部取出一个Message对象，链表头指针指向下一个对象，将该Message对象next指针域置空，标志位置空，对象池对象数量大小减一，返回这个Message对象

 使用recycle方法回收Message对象:
 ```
 /**
     * Return a Message instance to the global pool.
     * <p>
     * You MUST NOT touch the Message after calling this function because it has
     * effectively been freed.  It is an error to recycle a message that is currently
     * enqueued or that is in the process of being delivered to a Handler.
     * </p>
     */
    public void recycle() {
        if (isInUse()) {
            if (gCheckRecycle) {
                throw new IllegalStateException("This message cannot be recycled because it "
                        + "is still in use.");
            }
            return;
        }
        recycleUnchecked();
    }
    
     /**
     * Recycles a Message that may be in-use.
     * Used internally by the MessageQueue and Looper when disposing of queued Messages.
     */
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
 ```
 回收方法是把Message对象的所有标志位清空，数据清空，然后将此回收的Message对象加入到链表的头部，这样一个废弃的Message对象就被回收了。
 
 
# 参考
 1. [java对象池](http://blog.csdn.net/shimiso/article/details/9814917)；
 2. [ Android RecyclerView之RecycledViewPool、SortedListAdapter](http://blog.csdn.net/jdsjlzx/article/details/51121090)
 
