# OkHttp（一）
## 简介
1. 支持HTTP2/SPDY
2. socket自动选择最好路线，并支持自动重连
3. 拥有自动维护的socket连接池，减少握手次数
4. 拥有队列线程池，轻松写并发
5. 拥有Interceptors轻松处理请求与响应（比如透明GZIP压缩）基于Headers的缓存策略

## 使用
GET请求：

```
	OkHttpClient client = new OkHttpClient();
	Request request = new Request.Builder()
      		.url(url)
      		.build();
	Response response = client.newCall(request).execute();
	return response.body().string();
```
## 源码分析
来看看构造方法：

```
	public OkHttpClient() {
    	this(new Builder());
  	}
```
无参的构造方法里面会new一个Builder，然后初始化OkHttpClient。

之后用builder构建了Request对象，然后执行了OKHttpClient的newCall方法，那么咱们就看看这个newCall里面都做什么操作？

```
@Override 
public Call newCall(Request request) {
    return new RealCall(this, request, false /* for web socket */);
}
```
所以client.newCall(request).execute();实际上执行的是RealCall的execute方法，现在咱们再回来看下RealCall的execute的具体实现。

```
@Override 
public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    try {
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } finally {
      client.dispatcher().finished(this);
    }
  }
```
可以看到首先同步判断call是否执行过，可以看出每个Call对象只能使用一次原则。
然后调用了captureCallStackTrace()方法。然后进入了第一个核心类---Dispatcher的execute()方法。

那么我们来看看client的dispatcher()方法。

```
public Dispatcher dispatcher() {
    return dispatcher;
  }
```
可以看到，返回了Dispatcher对象。

那么我们是什么时候初始化的呢？回到构造方法。

```
 	public OkHttpClient() {
    	this(new Builder());
  	}

  	OkHttpClient(Builder builder) {
    	this.dispatcher = builder.dispatcher;
    	//其他代码暂时省略
	}
```
可以看到，我们一开始构造时就传入了Builder的builder.dispatcher。那么这个builder.dispatcher又是什么时候初始化的呢？继续看源码。

```
	public Builder() {
      	dispatcher = new Dispatcher();
		//其他代码暂时省略
	}
```
可以看到默认执行Builder()时就创建了一个Dispatcher。那么我们看下dispatcher里面的execute()是如何处理的。

```
	synchronized void executed(RealCall call) {
    	runningSyncCalls.add(call);
  	}
```
可以看到，runningSyncCalls执行了add方法，添加的参数是RealCall。runningSyncCalls是什么呢？

```
/** Ready async calls in the order they'll be run. */
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** Running synchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

```
可以看到runningSyncCalls是双向队列。另外，我们发现Dispatcher里面定义了三个双向队列，看下注释，我们大概能明白readyAsyncCalls是一个存放了等待执行任务Call的双向队列，runningAsyncCalls是一个存放异步请求任务Call的双向任务队列，runningSyncCalls是一个存放同步请求的双向队列。

回到之前，执行完client.dispatcher().executed(）方法，要执行getResponseWithInterceptorChain()方法。

```
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    //添加开发者应用层自定义的Interceptor
    interceptors.addAll(client.interceptors());
    //这个Interceptor是处理请求失败的重试，重定向    
    interceptors.add(retryAndFollowUpInterceptor);
    //这个Interceptor工作是添加一些请求的头部或其他信息
    //并对返回的Response做一些友好的处理（有一些信息你可能并不需要）
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    //这个Interceptor的职责是判断缓存是否存在，读取缓存，更新缓存等等
    interceptors.add(new CacheInterceptor(client.internalCache()));
    //这个Interceptor的职责是建立客户端和服务器的连接
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      //添加开发者自定义的网络层拦截器
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));
    //一个包裹这request的chain
    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    //把chain传递到第一个Interceptor手中
    return chain.proceed(originalRequest);
  }
```
可以看到，主要的操作就是new了一个ArrayList，然后就是不断的add拦截器，后面new了一个RealInterceptorChain对象，最后调用了chain.proceed()方法。

我们来看看RealInterceptorChain()构造方法。

```
public RealInterceptorChain(List<Interceptor> interceptors, StreamAllocation streamAllocation,
      HttpCodec httpCodec, Connection connection, int index, Request request) {
    this.interceptors = interceptors;
    this.connection = connection;
    this.streamAllocation = streamAllocation;
    this.httpCodec = httpCodec;
    this.index = index;
    this.request = request;
  }
```

就是一些赋值操作，有一点值得注意的是this.interceptors = interceptors;这里就保存了拦截链。

之后我们来看一下chain.proceed()方法。由于Interceptor是个接口，所以应该是具体实现类RealInterceptorChain的proceed实现。


```
public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      Connection connection) throws IOException {
		//省略其他代码   
	calls++;
  RealInterceptorChain next = new RealInterceptorChain(
        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);
	//省略其他代码   
   	return response;
}
```
然后看到在proceed方面里面又new了一个RealInterceptorChain类的next对象，温馨提示下，里面的streamAllocation, httpCodec, connection都是null，所以这个next对象和chain最大的区别就是index属性值不同chain是0.而next是1，然后取interceptors下标为1的对象的interceptor。由从上文可知，如果没有开发者自定义的Interceptor时，首先调用的RetryAndFollowUpInterceptor，如果有开发者自己定义的interceptor则调用开发者interceptor。

后面的流程都差不多，在每一个interceptor的intercept方法里面都会调用chain.proceed()从而调用下一个interceptor的intercept(next)方法，这样就可以实现遍历getResponseWithInterceptorChain里面interceptors的item，实现遍历循环。

之前我们看过getResponseWithInterceptorChain里面interceptors的最后一个item是CallServerInterceptor，最后一个Interceptor(即CallServerInterceptor)里面是直接返回了response 而不是进行继续递归。

CallServerInterceptor返回response后返回给上一个interceptor,一般是开发者自己定义的networkInterceptor，然后开发者自己的networkInterceptor把他的response返回给前一个interceptor，依次以此类推返回给第一个interceptor，这时候又回到了realCall里面的execute()里面了。

最后把response返回给get请求的返回值。至此整体GET请求的大体流程就已经结束了。当然return之前会执行finally中的client.dispatcher().finished(this);释放dispatcher。



