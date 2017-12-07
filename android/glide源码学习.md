最新版本[Gilde 4.3.1](https://github.com/bumptech/glide)以及[使用说明](https://muyangmin.github.io/glide-docs-cn/)。

Glide.with(..)可以传入的参数：Activity，android.app.Fragment，android.support.v4.app.Fragment，FragmentActivity，Context。
在 Glide.with()内部会调用
```
  RequestManagerRetriever retriever = RequestManagerRetriever.get();
  return retriever.get(activity);
```
继续深入：
```
//Glide.java
    @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    public RequestManager get(Activity activity) { //以Activity为例
        if (Util.isOnBackgroundThread() || Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
            return get(activity.getApplicationContext());
        } else {
            assertNotDestroyed(activity);
            android.app.FragmentManager fm = activity.getFragmentManager();
            return fragmentGet(activity, fm);
        }
    }
```
get() 方法会根据传入的 context 对象和当前线程，创建不同的 RequestManager 实例，在fragmentGet()传入了activity、fm，其中fm这是一个可以感知传入的各个组件的生命周期的类
所以尽量不要在非 UI 线程使用 Glide 加载图片，尽量使用 Activity、Fragment 等带有生命周期的组件配合 Glide 使用。
 
深入fragmentGet():
```
//RequestManagerRetriever.java
@TargetApi(Build.VERSION_CODES.HONEYCOMB)
    RequestManager fragmentGet(Context context, final android.app.FragmentManager fm) {
        RequestManagerFragment current = (RequestManagerFragment) fm.findFragmentByTag(TAG);
        if (current == null) {
            current = pendingRequestManagerFragments.get(fm);
            if (current == null) {
                current = new RequestManagerFragment();
                pendingRequestManagerFragments.put(fm, current);
                fm.beginTransaction().add(current, TAG).commitAllowingStateLoss();
                handler.obtainMessage(ID_REMOVE_FRAGMENT_MANAGER, fm).sendToTarget();
            }
        }
        RequestManager requestManager = current.getRequestManager();
        if (requestManager == null) {
            requestManager = new RequestManager(context, current.getLifecycle());
            current.setRequestManager(requestManager);
        }
        return requestManager;

    }
```
在创建RequestManager的同时，也创建一个透明的 RequestManagerFragment（extends Fragment），可以看到通过添加一个透明的RequestManagerFragment，就可以感知到控件的生命周期了
在此回答关于Glide的一个问题了：&&为什么会内部拥有一个fragment&&，具体代码可以进入RequestManagerFragment之中，一目了然。
至此，Glide.with()过程分析完了，返回的结果为RequestManager
## RequestManager的工作原理
```
public DrawableTypeRequest<String> load(String string) {
        return (DrawableTypeRequest<String>) fromString().load(string);
    }
    public DrawableTypeRequest<String> fromString() {
        return loadGeneric(String.class);
    }
    private <T> DrawableTypeRequest<T> loadGeneric(Class<T> modelClass) {
        ModelLoader<T, InputStream> streamModelLoader = Glide.buildStreamModelLoader(modelClass, context);
        ModelLoader<T, ParcelFileDescriptor> fileDescriptorModelLoader =
                Glide.buildFileDescriptorModelLoader(modelClass, context);
        if (modelClass != null && streamModelLoader == null && fileDescriptorModelLoader == null) {
            throw new IllegalArgumentException("Unknown type " + modelClass + ". You must provide a Model of a type for"
                    + " which there is a registered ModelLoader, if you are using a custom model, you must first call"
                    + " Glide#register with a ModelLoaderFactory for your custom model class");
        }

        return optionsApplier.apply(
                new DrawableTypeRequest<T>(modelClass, streamModelLoader, fileDescriptorModelLoader, context,
                        glide, requestTracker, lifecycle, optionsApplier));
    }
    @Override
    public DrawableRequestBuilder<ModelType> load(ModelType model) {
        super.load(model);
        return this;
    }
``` 
以上代码创建了请求DrawableRequestBuilder<ModelType>，最关键的代码Glide().with().load().into()中的into，是真正执行请求的部分
## 请求处理
``` 
    // GenericRequestBuilder.java
    public Target<TranscodeType> into(ImageView view) {
        ...
        //可以暂时忽略
        return into(glide.buildImageViewTarget(view, transcodeClass)); //view需要一次封装转为顶层的target
    }
    
     public <Y extends Target<TranscodeType>> Y into(Y target) { //target资源加载目标
        ...
        Request previous = target.getRequest();

        if (previous != null) {  //首次可以忽略此部分
            previous.clear();
            requestTracker.removeRequest(previous);
            previous.recycle();
        }

        Request request = buildRequest(target); //创建并初始化请求的一些参数
        target.setRequest(request);
        lifecycle.addListener(target);
        requestTracker.runRequest(request);

        return target;
    }
```
重点代码：Request request = buildRequest(target);
```
  // GenericRequestBuilder.java
  buildRequest(target) -->buildRequestRecursive(target, null)-->obtainRequest(target, sizeMultiplier, priority, parentCoordinator) 
   
```
重点代码：requestTracker.runRequest(request)，RequesTracker如其名一样，可以跟踪取消、请求进度、请求结束、请求失败
```
  //开始跟踪请求
  public void runRequest(Request request) {
        requests.add(request);
        if (!isPaused) {
            request.begin();
        } else {
            pendingRequests.add(request);
        }
    }
```
至此 ，一个简单的图片加载请求就解析了一半。关于网络请求的未完待续。。。
