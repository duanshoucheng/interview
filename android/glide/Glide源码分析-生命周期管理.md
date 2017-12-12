Glide可以根据Activity/Fragment的生命周期来控制图片的加载。

和生命周期相关的类如下： 
- Lifecycle:用来监听Activity/Fragment的生命周期事件，包含addListener和removeListener两个方法。
- LifecycleListener:监听Fragment/Activity生命周期事件的接口。包含onStart、onStop、onDestroy三个方法，当Fragment/Activity生命周期放生变化时，会调用对应的方法。实现类是RequestManager、ConnectivityMonitor、Target。
- ActivityFragmentLifecycle:Lifecycle的实现类，用来跟踪和通知 Fragment/Activity监听器的生命周期变化。 
在addListener方法中，把LifecycleListener添加到lifecycleListeners中，最新的生命周期方法将被调用，如果Activity/Fragment是暂停状态,LifecycleListener#onStop()方法会被调用， 
onStart、onDestroy同理。ActivityFragmentLifecycle中还包含onStart、onStop、onDestroy三个方法，被调用的时候会遍历lifecycleListeners，调用LifecycleListener的对应的生命周期方法。
- SupportRequestManagerFragment:一个没有视图的Fragment，持有ActivityFragmentLifecycle实例，并持有RequestManager实例，RequestManager可以启动、停止、管理Glide的requests请求。

当调用with方法开始一个下载，并绑定Activity/Fragment的生命周期。 
绑定SupportRequestManagerFragment的方法是RequestManagerRetriever.getSupportRequestManagerFragment
```
//根据tag从FragmentManager里获取SupportRequestManagerFragment，
//如果不存在，从pendingSupportRequestManagerFragments里查找，
//如果依然为null,新建一个SupportRequestManagerFragment，
//存储在pendingSupportRequestManagerFragments里，并通过fm添加。
SupportRequestManagerFragment getSupportRequestManagerFragment(
      final FragmentManager fm, Fragment parentHint) {
    SupportRequestManagerFragment current =
        (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
      current = pendingSupportRequestManagerFragments.get(fm);
      if (current == null) {
        current = new SupportRequestManagerFragment();
        current.setParentFragmentHint(parentHint);
        pendingSupportRequestManagerFragments.put(fm, current);
        fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
        handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
      }
    }
    return current;
  }
```
当系统调用SupportRequestManagerFragment.onAttach方法时，调用registerFragmentWithRoot方法，将Fragment的子Fragment关联的SupportRequestManagerFragment 添加到childRequestManagerFragments里。当调用onDetach方法时，说明Fragment不再和Activity相关联，这时要将SupportRequestManagerFragment从childRequestManagerFragments里移除，并将rootRequestManagerFragment置为空。 
调用Fragment的onStart方法时，通过lifecycle.onStart();遍历lifecycleListeners，分别调用对应的onStart方法，例如，RequestManager的onStart方法的实现是 
resumeRequests(); 
targetTracker.onStart(); 
重新开启失败或暂停的请求任务。

获取到SupportRequestManagerFragment之后，通过getRequestManager方法获取关联的RequestManager，如果为null，新建一个，并设置到SupportRequestManagerFragment。
```
RequestManagerRetriever#supportFragmentGet方法
//在RequestManager构造方法中，会通过addListener方法，将自身添加到SupportRequestManagerFragment的lifecycle中。
 Glide glide = Glide.get(context);
 requestManager = new RequestManager(glide, current.getLifecycle(),    current.getRequestManagerTreeNode());
 current.setRequestManager(requestManager);
```
通过以上几步，就可以通过Activity/Fragment的生命周期管理Request了。

**网络变化后，如何重新开启下载任务？**
```
RequestManager(
      Glide glide,
      Lifecycle lifecycle,
      RequestManagerTreeNode treeNode,
      RequestTracker requestTracker,
      ConnectivityMonitorFactory factory) {
    this.glide = glide;
    this.lifecycle = lifecycle;
    this.treeNode = treeNode;
    this.requestTracker = requestTracker;

    final Context context = glide.getGlideContext().getBaseContext();

......
    connectivityMonitor =
        factory.build(context, new RequestManagerConnectivityListener(requestTracker));
    lifecycle.addListener(connectivityMonitor);

    ......
  }
```
GlideBuilder中生成DefaultConnectivityMonitorFactory的实例。 
在RequestManager构造方法中，通过DefaultConnectivityMonitorFactory的build方法创建ConnectivityMonitor的实现类，DefaultConnectivityMonitor(如果有网络权限)。然后再通过lifecycle的add方法注册。当调用DefaultConnectivityMonitor的onStart方法时，注册广播，监听网络变化。当调用onStop()时，取消注册广播。当收到广播变化通知时，调用 
listener.onConnectivityChanged(isConnected);（listener是RequestManagerConnectivityListener(requestTracker)的实例)，如果有网络，调用requestTracker.restartRequests()的方法，重新开启请求。

