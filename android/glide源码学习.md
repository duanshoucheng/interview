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
  buildRequest(target) -->buildRequestRecursive(target, null)-->obtainRequest(target, sizeMultiplier, priority, parentCoordinator) ，贴出具体代码：
   private Request obtainRequest(Target<TranscodeType> target, float sizeMultiplier, Priority priority,
            RequestCoordinator requestCoordinator) {
        return GenericRequest.obtain(  //返回GenericRequest，后面网络部分会重点涉及
                loadProvider,
                model,
                signature,
                context,
                priority,
                target,
                sizeMultiplier,
                placeholderDrawable,
                placeholderId,
                errorPlaceholder,
                errorId,
                requestListener,
                requestCoordinator,
                glide.getEngine(),
                transformation,
                transcodeClass,
                isCacheable,
                animationFactory,
                overrideWidth,
                overrideHeight,
                diskCacheStrategy);
    }
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
至此 ，一个简单的图片加载请求就解析了一半。小憩一会，接着下文关于网络请求的部分。
上文GenericRequest，这将是关于网络请求部分，在requestTracker.runRequest(request)-->request.begin(),上文GenericRequest.begin():
 ```
  //GenericRequest.java
  @Override
    public void begin() {
        ...
        if (Util.isValidDimensions(overrideWidth, overrideHeight)) { 如果Glide设置了override()
            onSizeReady(overrideWidth, overrideHeight);
        } else {
            target.getSize(this); //我们分析默认情况，让其自动计算，
        }

        if (!isComplete() && !isFailed() && canNotifyStatusChanged()) {
            target.onLoadStarted(getPlaceholderDrawable());
        }
        ...
    }
    
    public void onSizeReady(int width, int height) {
        ...
        loadStatus = engine.load(signature, width, height, dataFetcher, loadProvider, transformation, transcoder,
                priority, isMemoryCacheable, diskCacheStrategy, this);
        ...
    }
 ```
发现在target.getSize(this)内部路线中最后还是调用了onSizeReady(overrideWidth, overrideHeight)，所以继续engine.load()的源码：
```
  //Engine.java
  public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        ...
        EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
        if (cached != null) {
            cb.onResourceReady(cached);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Loaded resource from cache", startTime, key);
            }
            return null;
        }

        EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
        if (active != null) {
            cb.onResourceReady(active);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Loaded resource from active resources", startTime, key);
            }
            return null;
        }

        EngineJob current = jobs.get(key);
        if (current != null) {
            current.addCallback(cb);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Added to existing load", startTime, key);
            }
            return new LoadStatus(cb, current);
        }

        EngineJob engineJob = engineJobFactory.build(key, isMemoryCacheable);
        DecodeJob<T, Z, R> decodeJob = new DecodeJob<T, Z, R>(key, width, height, fetcher, loadProvider, transformation,
                transcoder, diskCacheProvider, diskCacheStrategy, priority);
        EngineRunnable runnable = new EngineRunnable(engineJob, decodeJob, priority);
        jobs.put(key, engineJob);
        engineJob.addCallback(cb);
        engineJob.start(runnable);
        ...
        return new LoadStatus(cb, engineJob);
    }
```
cb.onResourceReady(cached)和cb.onResourceReady(active);都是读取缓存的数据，这是一个回调，在GenericRequest.onResourceRead（）中可以看到target.onResourceReady(result, animation);target 在GenericRequestBuilder.into(View)中的into(glide.buildImageViewTarget(view, transcodeClass));创建：
```
public class ImageViewTargetFactory {

    @SuppressWarnings("unchecked")
    public <Z> Target<Z> buildTarget(ImageView view, Class<Z> clazz) {
        if (GlideDrawable.class.isAssignableFrom(clazz)) {
            return (Target<Z>) new GlideDrawableImageViewTarget(view);
        } else if (Bitmap.class.equals(clazz)) {
            return (Target<Z>) new BitmapImageViewTarget(view);
        } else if (Drawable.class.isAssignableFrom(clazz)) {
            return (Target<Z>) new DrawableImageViewTarget(view);
        } else {
            throw new IllegalArgumentException("Unhandled class: " + clazz
                    + ", try .as*(Class).transcode(ResourceTranscoder)");
        }
    }
}
```
内部可以看到：
```
  @Override
    protected void setResource(Drawable resource) {
       view.setImageDrawable(resource);
    }
```
至此就可以从缓存中加载数据加载了。
回到上面，如果缓存不存在，则开始读取网络数据，再看看源码：
```
        EngineJob engineJob = engineJobFactory.build(key, isMemoryCacheable);
        DecodeJob<T, Z, R> decodeJob = new DecodeJob<T, Z, R>(key, width, height, fetcher, loadProvider, transformation,
                transcoder, diskCacheProvider, diskCacheStrategy, priority);
        EngineRunnable runnable = new EngineRunnable(engineJob, decodeJob, priority);
        jobs.put(key, engineJob);
        engineJob.addCallback(cb);
        engineJob.start(runnable);
```
进入 engineJob.start(runnable):
```
  public void start(EngineRunnable engineRunnable) {
        this.engineRunnable = engineRunnable; //传递创建的线程
        future = diskCacheService.submit(engineRunnable);
    }
```
EngineRunnable肯定是开启线程，准备网络请求加载：
```
//EngineRunnable.java
@Override
    public void run() {
        if (isCancelled) {
            return;
        }

        Exception exception = null;
        Resource<?> resource = null;
        try {
            resource = decode();  //目光聚焦
        } catch (Exception e) {
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                Log.v(TAG, "Exception decoding", e);
            }
            exception = e;
        }

        if (isCancelled) {
            if (resource != null) {
                resource.recycle();
            }
            return;
        }

        if (resource == null) {
            onLoadFailed(exception);
        } else {
            onLoadComplete(resource);
        }
    }
    private Resource<?> decode() throws Exception {
        if (isDecodingFromCache()) {
            return decodeFromCache(); //以后再研究
        } else {
            return decodeFromSource(); //深入
       }
    }
    private Resource<?> decodeFromSource() throws Exception {
        return decodeJob.decodeFromSource();
    }
```
结构很清晰，直接进入decode()-->decodeFromSource(),然后转入DecodeJob类中：
```
    public Resource<Z> decodeFromSource() throws Exception {
        Resource<T> decoded = decodeSource();
        return transformEncodeAndTranscode(decoded);
    }
    private Resource<T> decodeSource() throws Exception {
        Resource<T> decoded = null;
        try {
            long startTime = LogTime.getLogTime();
            final A data = fetcher.loadData(priority); //
            ...
            if (isCancelled) {
                return null;
            }
            decoded = decodeFromSourceData(data); //先不关注
        } finally {
            fetcher.cleanup();
        }
        return decoded;
    }
```
聪明的你一定发现了什么，对，就是fetcher.loadData(priority);看来一切的秘密都在fetcher之中，在一路追踪fetcher的情况下，终于在GenericRequestBuilder中找到一点蛛丝马迹：private ChildLoadProvider<ModelType, DataType, ResourceType, TranscodeType> loadProvider;
