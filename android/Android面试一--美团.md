一面二面的问题汇总：
### 1、java的包安全问题
public，protect，private，默认。这四种的区别
### 2、Intent的数据大小
当然是越小越好啊，并没有固定，当然大到一定程度会出现异常
### 3、注解问题
用过哪些注解，ButterKnife、Retrofit、dagger2等都有用过。ButterKnife和Retrofit注解的区别：
ButterKnife注解是在编译阶段生成类，在build/generated/source/apt目录之下可以看到自己的包名，在包下可以看到使用ButterKnife的类名_ViewBinding.java，该类继承Unbinder，
在构造函数中可以看到熟悉的findViewById等代码。  
Retrofit使用的动态代理的方式实现的
### 4、Retrofit的流程是怎样的，说说源码，优势在哪里，有没有二次封装
建议okhttp、glide、Retrofit等等源码都看一遍，这些被问到的可能非常大。
```
Retrofit restAdapter = new Retrofit.Builder()
        .baseUrl(ConstantURL.MOB_BASE)
        .addConverterFactory(new NullOnEmptyConverterFactory())
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
        .client(client)
        .build();
instance = restAdapter.create(NetworkServiceAPI.class);
```
以上面的为例来说，Retrofit首先使用了builder模式，构建者模式创建了okhttp3.Call.Factory、Executor、List<CallAdapter.Factory>、List<Converter.Factory>等，
create里使用的是动态代理模式，动态代理通过反射获取函数内容，传递到ServiceMethod，由其分析组成请求地址，然后传给OKhttp，就可以了。  
这里说明下是怎么传给okhttp的，先贴源码：
```
public <T> T create(final Class<T> service) {
    ```
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, Object... args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod serviceMethod = loadServiceMethod(method);
            OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
  }
```
我们在使用的时候，一般会设置 ...client(client).build(),这里可以设置网络请求模块，可以看到我用到的是Okhttp，其实如果不设置，默认也是okhttp：
```
//Retrofit.java内部类Builder.build()方法内
public Retrofit build() {
      ```
      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }
      ```
    }
```
### 5、service的启动方式，Service提高级别
Service的两种启动方式比较简单，还有多次startService()会怎样，当然是不走onCreate()了，只走onStart().提高优先级我说成了保活了，在我说了几种保活机制之后面试官提醒我回答不对，

### 6、IntentService和Service的区别，IntentService的原理，多次启动IntentService会怎样
### 7、网络优化
### 8、性能优化
### 9、打包问题，如何多渠道打包，还有gradle其他的一些问题。
### 10、手写一个单例，注意懒汉式、饿汉式、线程安全
### 11、手写一个线程池，Synchronize和volatile的区别
### 12、其他问题，比如为什么跳槽，自己职业规划。
### 13、Fragment和Activity的通信
这个可以互相调用，但是Fragment和Fragment之间通信需要通过Activity，但是不建议这样，可以使用EventBus。  

问题没有排名，记多少写多少。一面二面用了两个小时，还有一些问题记不住了，网络优化和性能优化探讨的比较多，开源框架还有RxJava，设计模式MVC和MVP的区别
