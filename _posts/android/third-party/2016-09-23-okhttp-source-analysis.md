---
layout: single
title: OkHttp源码解析
excerpt: "okhttp 源码解析"
author_profile: false
sidebar:
  nav: "android"
categories: android third-party
---
# OkHttp源码解析  
> 基于 OkHttp 3.4.1  

OkHttp可以高效的处理网络请求, 节省流量:  
- 支持HTTP/2协议, 允许指向同一主机的多个请求使用同一个socket  
- 如果HTTP/2不可用, 将会使用**连接池**来降低请求延迟  
- 支持GZIP压缩, 节省流量  
- 支持Response缓存  

OkHttp可以自动处理一些问题, 比如: 发生常见的连接问题时可以自动恢复; 服务器配置多个IP地址时, 如果第一个IP连接失败, 会自动尝试连接下一个IP．  

OkHttp的request/response采用流式的builder设计．支持同步和异步两种请求．  

## 1.OkHttp的基本使用　　
{% highlight java linenos %}
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
    .url("https://api.github.com/repos/square/okhttp/contributors")
    .build();

Response response = client.newCall(request).execute();
{% endhighlight %}
OkHttp的使用分为三个步骤：  
第一步，创建OkHttpClient对象.  
第二步，创建Request请求．  
第三步，发出请求，获取Response.  
下面我们来一步一步的分析这三个步骤后面的具体逻辑．．．

## 2.创建OkHttpClient对象　　
`new OkHttpClient();`做了那些工作呢？我们来看一下它的源码：
{% highlight java linenos %}
public OkHttpClient() {
  this(new Builder());
}

private OkHttpClient(Builder builder) {
  this.dispatcher = builder.dispatcher;
  this.proxy = builder.proxy;
  this.retryOnConnectionFailure = builder.retryOnConnectionFailure;
  this.connectTimeout = builder.connectTimeout;
  this.readTimeout = builder.readTimeout;
  this.writeTimeout = builder.writeTimeout;
  ...//还有Ｎ多个成员变量
}
{% endhighlight %}
可以看到，`new OkHttpClient();`中调用了`this(new Builder());`  
`OkHttpClient(Builder);`中将builder中的成员变量传递给OkHttpClient中同名的成员变量．  
这个Builder又是什么鬼呢？这里其实用到了**Builder设计模式**．  
Builder设计模式又是什么呢？下面我们边看源码边分析．　　
{% highlight java linenos %}
public class OkHttpClient｛
  final Dispatcher dispatcher;
  final Proxy proxy;
  final List<Interceptor> interceptors;
  final List<Interceptor> networkInterceptors;
  final boolean retryOnConnectionFailure;
  final int connectTimeout;
  final int readTimeout;
  final int writeTimeout;
  ...//还有Ｎ多个成员变量

  public OkHttpClient() {
    this(new Builder());
  }

  private OkHttpClient(Builder builder) {
    this.dispatcher = builder.dispatcher;
    this.proxy = builder.proxy;
    this.retryOnConnectionFailure = builder.retryOnConnectionFailure;
    this.connectTimeout = builder.connectTimeout;
    this.readTimeout = builder.readTimeout;
    this.writeTimeout = builder.writeTimeout;
    ...//还有Ｎ多个成员变量
  }

  public static final class Builder {
    Dispatcher dispatcher;
    Proxy proxy;
    final List<Interceptor> interceptors = new ArrayList<>();
    final List<Interceptor> networkInterceptors = new ArrayList<>();
    boolean retryOnConnectionFailure;
    int connectTimeout;
    int readTimeout;
    int writeTimeout;
    ...//还有Ｎ多个成员变量

    public Builder() {
      dispatcher = new Dispatcher();
      protocols = DEFAULT_PROTOCOLS;
      retryOnConnectionFailure = true;
      connectTimeout = 10_000;
      readTimeout = 10_000;
      writeTimeout = 10_000;
      ...//还有Ｎ多个成员变量
    }

    public Builder dispatcher(Dispatcher dispatcher) {
      if (dispatcher == null) throw new IllegalArgumentException("dispatcher == null");
      this.dispatcher = dispatcher;
      return this;
    }
    public Builder proxy(Proxy proxy) {
      this.proxy = proxy;
      return this;
    }

    //这个方法很重要，后面会讲到
    public Builder addInterceptor(Interceptor interceptor) {
      interceptors.add(interceptor);
      return this;
    }

    //这个方法也很重要，后面会讲到
    public Builder addNetworkInterceptor(Interceptor interceptor) {
      networkInterceptors.add(interceptor);
      return this;
    }

    public Builder retryOnConnectionFailure(boolean retryOnConnectionFailure) {
      this.retryOnConnectionFailure = retryOnConnectionFailure;
      return this;
    }

    public Builder connectTimeout(long timeout, TimeUnit unit) {
      if (timeout < 0) throw new IllegalArgumentException("timeout < 0");
      if (unit == null) throw new NullPointerException("unit == null");
      long millis = unit.toMillis(timeout);
      if (millis > Integer.MAX_VALUE) throw new IllegalArgumentException("Timeout too large.");
      if (millis == 0 && timeout > 0) throw new IllegalArgumentException("Timeout too small.");
      connectTimeout = (int) millis;
      return this;
    }

    public Builder readTimeout(long timeout, TimeUnit unit) {
      if (timeout < 0) throw new IllegalArgumentException("timeout < 0");
      if (unit == null) throw new NullPointerException("unit == null");
      long millis = unit.toMillis(timeout);
      if (millis > Integer.MAX_VALUE) throw new IllegalArgumentException("Timeout too large.");
      if (millis == 0 && timeout > 0) throw new IllegalArgumentException("Timeout too small.");
      readTimeout = (int) millis;
      return this;
    }

    public Builder writeTimeout(long timeout, TimeUnit unit) {
      if (timeout < 0) throw new IllegalArgumentException("timeout < 0");
      if (unit == null) throw new NullPointerException("unit == null");
      long millis = unit.toMillis(timeout);
      if (millis > Integer.MAX_VALUE) throw new IllegalArgumentException("Timeout too large.");
      if (millis == 0 && timeout > 0) throw new IllegalArgumentException("Timeout too small.");
      writeTimeout = (int) millis;
      return this;
    }

    public OkHttpClient build() {
      return new OkHttpClient(this);
    }
｝
{% endhighlight %}
由上面这段代码可以看出，Builder是OkHttpClient的一个静态内部类．OkHttpClient中有的成员变量，在Builder中也都存在．  
通过`new OkHttpClient();`方式创建对象使用的是默认Builder构造的，从外部看不到builder的影子．我们还可以像下面这样构造OkHttpClient对象：
{% highlight java linenos %}
OkHttpClient client = new OkHttpClient.Builder()
                          .addInterceptor(new MyInterceptor())
                          .addNetWorkInterceptor(new MyNetWorkInterceptor())
                          .connectTimeout(20_000)
                          .readTimeout(20_000)
                          .writeTimeout(20_000)
                          .build();
{% endhighlight %}
首先，第一行代码中，`new OkHttpClient.Builder()`创建了一个Builder对象．  
最后，在第五行代码，`build()`才是真正创建OkHttpClient对象，由上一段代码可以看出，`build()`内部是通过`new OkHttpClient(Builder);`创建对象的，这一步与直接通过`new OkHttpClient();`创建对象是一样的．  
不同之处在第２～４行，通过这种链式调用，可以给builder对象动态设置成员变量的值，这样最后build()出来的OkHttpClient对象也就采用了builder对象动态设置的的成员变量．  

**Builder设计模式**的优点：Builder设计模式有一个最重要的优点就是，当一个对象有较多的可选参数时，依然可以通过一条链式调用优雅的创建对象．其次，保证了对象构建过程的连续性和使用过程中的稳定性．换句话说，也就是不论在`new OkHttpClient.Builder()`和`build()`之间动态设置了多少成员变量，如果不调用`build()`就无法创建OkHttpClient对象，这样就保证了构建过程的连续性；一旦通过`build()`创建出OkHttpClient对象，就无法更改OkHttpClient对象的成员变量，这就保证了OkHttpClient对象使用过程中的稳定性．如果想更改OkHttpClient对象的成员变量的话，就只能重新build()一个对象了．  

## ３.创建Request对象  
下面分析Request对象的创建过程:  
{% highlight java linenos %}
Request request = new Request.Builder()
    .url("https://api.github.com/repos/square/okhttp/contributors")
    .build();
{% endhighlight %}
通过上一步中对OkHttpClient创建过程的分析，很容易看出Request也是使用Builder模式构建的．在此就不做过多的分析了．  

## 4.发出请求，获取Response  
前面两步，创建OkHttpClient对象和Request对象都只是准备工作．第三步，发起请求，获取Response对象是我们要分析的重点．  
{% highlight java linenos %}
Response response = client.newCall(request).execute();
{% endhighlight %}
我们从`client.newCall(request)`开始看起：  
{% highlight java linenos %}
public Call newCall(Request request) {
  return new RealCall(this, request);
}
{% endhighlight %}
可以看到，`newCall(request)`调用的是`RealCall(Builder,Reqeust)`，继续看一下RealCall的源码
{% highlight java linenos %}
protected RealCall(OkHttpClient client, Request originalRequest) {
  this.client = client;
  this.originalRequest = originalRequest;
  this.retryAndFollowUpInterceptor = new RetryAndFollowUpInterceptor(client);
}
{% endhighlight %}
可以看到RealCall中就是接收一个OkHttpClient对象和一个Request对象,并且new了一个RetryAndFollowUpInterceptor（看到retry就大概可以知道这个一个和重连相关的interceptor,稍后会用到)。

OK，到此为止，OkHttpClient已经建立，Request已经建立，RealCall也已经建立。所有的准备工作已经完成，接下来进入发起请求，获取数据阶段:  
`client.newCall(request).execute();`其中的`newCall(request)`返回的是RealCall对象,因此`execute()`执行的是RealCall中`的execute()`  
{% highlight java linenos %}
public Response execute() throws IOException {
  ...
  try {
    client.dispatcher().executed(this);
    Response result = getResponseWithInterceptorChain();
    if (result == null) throw new IOException("Canceled");
    return result;
  } finally {
    client.dispatcher().finished(this);
  }
}
{% endhighlight %}
`execute()`的关键代码就在第4,5行,我们先来分析第4行:  
第4行中,`client.dispatcher()`获取一个Dispatcher对象(这个Dispatcher对象就是在创建OkHttpClient对象时,在OkHttpClient.Builder中默认创建的),然后执行`executed(this)`将RealCall添加到Dispatcher中专门存储同步请求的RealCall的队列中.关于Dispatcher的具体分析点击<a href="#dispatcher"><font color="blue">这里</font></a>  

第5行中getResponseWithInterceptorChain():  
{% highlight java linenos %}
private Response getResponseWithInterceptorChain() throws IOException {
  // Build a full stack of interceptors.
  List<Interceptor> interceptors = new ArrayList<>();
  interceptors.addAll(client.interceptors());
  interceptors.add(retryAndFollowUpInterceptor);
  interceptors.add(new BridgeInterceptor(client.cookieJar()));
  interceptors.add(new CacheInterceptor(client.internalCache()));
  interceptors.add(new ConnectInterceptor(client));
  if (!retryAndFollowUpInterceptor.isForWebSocket()) {
    interceptors.addAll(client.networkInterceptors());
  }
  interceptors.add(new CallServerInterceptor(
      retryAndFollowUpInterceptor.isForWebSocket()));

  Interceptor.Chain chain = new RealInterceptorChain(
      interceptors, null, null, null, 0, originalRequest);
  return chain.proceed(originalRequest);
}
{% endhighlight %}
第4行,先将自定义的interceptor添加到interceptors中  
接下来的5~8行,将系统定义的interceptor添加到interceptors中  
第9~11行,将自定义的networkInterceptor添加到interceptors中  
第12,13行,添加的CallServerInterceptor是真正访问服务器,获取数据的interceptor  
第15,16行,将interceptors和originalRequest添加到RealInterceptorChain中.  
最后,第17行,`chain.proceed(originalRequest)`将所有有的interceptor执行一遍,最终获取服务器的数据:  
(为了方便阅读,我将getResponseWithInterceptorChain代码和RealInterceptorChain代码放在一起)  
{% highlight java linenos %}
final Class RealCall{
  private Response getResponseWithInterceptorChain() throws IOException {
    ...
    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }
}

public final class RealInterceptorChain{
  private final int index;

  public RealInterceptorChain(List<Interceptor> interceptors, StreamAllocation streamAllocation,
      HttpCodec httpCodec, Connection connection, int index, Request request) {
    this.interceptors = interceptors;
    this.connection = connection;
    this.streamAllocation = streamAllocation;
    this.httpCodec = httpCodec;
    this.index = index;
    this.request = request;
  }

  public Response proceed(Request request) throws IOException {
      return proceed(request, streamAllocation, httpCodec, connection);
  }

  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      Connection connection) throws IOException {
    ...
    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(
        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);
    return response;
  }
}

{% endhighlight %}
1. 在`RealCall`的`getResponseWithInterceptorChain()`中new了一个RealInterceptorChain对象(注意这里传递的第5个参数,也就是index为0)  
2. 在RealCall执行`chain.proceed(originalRequest)`,最终调用`RealInterceptorChain`的`proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,Connection connection)`方法  
3. 在proceed方法中, 又new了一个RealInterceptorChain对象:命名为next.注意,此时这个next对象中的index=1(因为new对象传递的index参数为index+1)
4. interceptors.get(index)获取第一个interceptor(第一次调用proceed时,index=0;第一次调用proceed,index=1;以此类推...)
5. interceptor.intercept(next),执行interceptor的intercept方法. 写过拦截器的同学应该知道,拦截器必须实现intercept方法,并且intercept方法中必需执行chain.proceed(request)来获取一个Response对象.下面我们再将自定义Interceptor和proceed代码放在一起来看:  

{% highlight java linenos %}
public class XxxInterceptor implements Interceptor{
  public Response intercept(Chain chain){
    Request request = chain.request();
    ...//do something
    return chain.proceed(request);
  }
}

public final class RealInterceptorChain{
  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      Connection connection) throws IOException {
    ...
    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(
        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);
    return response;
  }
}
{% endhighlight %}
第17行`Response response = interceptor.intercept(next);`调用XxxInterceptor的intercept方法  
第5行`chain.proceed(request)`又调用RealInterceptorChain的proceed方法  
RealInterceptorChain的proceed方法中每次新建RealInterceptorChain对象index是+1的. 因此第16行`Interceptor interceptor = interceptors.get(index);`是按照interceptors.get(0),interceptors.get(1),interceptors.get(2)...这样的顺序取的interceptor. 这就保证了每一个拦截器可以按照添加的顺序依次执行.  

有的同学可能会问,Interceptor和RealInterceptorChain这样一直相互调用, 那么什么时候停止呢?  
还记得在`getResponseWithInterceptorChain()`中添加的那一坨拦截器么? 最后一个拦截器`CallServerInterceptor`中是没有执行`chain.proceed(Request)`方法的,因此这条调用连自然就可以停止.由此可见,拦截器的位置很重要.  

Interceptor这种链式调用模式称之为<a herf="https://zh.wikipedia.org/wiki/%E8%B4%A3%E4%BB%BB%E9%93%BE%E6%A8%A1%E5%BC%8F"><font color="blue">责任链模式</font></a>: 它包含了一些命令对象和一系列的处理对象。每一个处理对象决定它能处理哪些命令对象，它也知道如何将它不能处理的命令对象传递给该链中的下一个处理对象。该模式还描述了往该处理链的末尾添加新的处理对象的方法。

下面附上一张OkHttp的执行过程流程图, 图片来自<a href="http://blog.piasy.com/"><font color="blue">Piasy</font></a>的博客:  

### 5.参考文章  
OkHttp官方文档: http://square.github.io/okhttp/

拆轮子系列--拆OkHttp: http://blog.piasy.com/2016/07/11/Understand-OkHttp/


## Tips  

### <span id="dispatcher">Dispatcher的部分代码</span>  
{% highlight java linenos %}
public final class Dispatcher {

  /** Executes calls. Created lazily. */
  private ExecutorService executorService;

  /** Ready async calls in the order they'll be run. */
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** Running synchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

  public Dispatcher(ExecutorService executorService) {
    this.executorService = executorService;
  }

  public Dispatcher() {
  }

  public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }

  synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }

  synchronized void enqueue(AsyncCall call) {
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);
      executorService().execute(call);
    } else {
      readyAsyncCalls.add(call);
    }
  }

}
{% endhighlight %}
可以看到Dispatcher中有三个Deque:  
- readyAsyncCalls: 存放准备异步执行的AsyncCall  
- runningAsyncCalls: 存放正在异步执行的AsyncCall,包括被取消,但还没有完成的请求.(这里的没有完成指的是取消没有完成, 还是请求已经发出去,但是和服务器的通讯还没有完成??? 有待分析)  
- runningSyncCalls: 存放正在同步执行的RealCall,包括正在被取消的RealCall  
可见同步和异步请求的Call都被存放在Dispatcher的Deque中.同步请求的RealCall通过第30~32行的`executed(RealCall)`添加; 异步请求通过第34~41行的enqueue添加.  
