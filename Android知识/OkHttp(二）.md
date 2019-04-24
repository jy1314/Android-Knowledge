# OkHttp（二）

## 异步请求的使用

```
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();

  Response response = client.newCall(request).enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {
        }
        @Override
        public void onResponse(Call call, Response response) throws IOException {
        }
    });
```
前面和同步一样new了一个OKHttp和Request。这块和同步一样就不说了，那么说说和同步不一样的地方，后面异步进入了newCall()的enqueue()方法。
由于之前分析过，newCall()里面是生成了一个RealCall对象，那么执行的其实是RealCall的enqueue()方法。

```
@Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
```
还是先同步判断是否被请求过了，之后和同步一样同步一样先调用了captureCallStackTrace();
然后调用client.dispatcher().enqueue(new AsyncCall(responseCallback));实际调用的是Dispatcher的enqueue()。

```
private int maxRequests = 64;
private int maxRequestsPerHost = 5;
synchronized void enqueue(AsyncCall call) {
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);//添加到执行队列，开始执行请求
      executorService().execute(call);//获得当前线程池，没有则创建一个
    } else {
      readyAsyncCalls.add(call);//添加到等待队列中
    }
  }
```
如果正在执行的请求小于设定值即64，并且请求同一个主机的request小于设定值5时，就先往正在运行的队列里面添加这个call，然后用线程池去执行这个call,否则就把他放到等待队列里面。
执行这个call的时候，自然会去走到这个call的run方法。那么我们来看看源码。

```
@Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }
```

```
final class AsyncCall extends NamedRunnable {
    private final Callback responseCallback;

    AsyncCall(Callback responseCallback) {
      super("OkHttp %s", redactedUrl());
      this.responseCallback = responseCallback;
    }

    String host() {
      return originalRequest.url().host();
    }

    Request request() {
      return originalRequest;
    }

    RealCall get() {
      return RealCall.this;
    }

    @Override protected void execute() {
      boolean signalledCallback = false;
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }
  }

```

上面看到NamedRunnable的构造方法设置了name在的run方法里面设定为当前线程的name,而NamedRunnable的run方法里面调用了它自己的抽象方法execute，由此可见NamedRunnable的作用就是设置了线程的name，然后回调子类的execute方法，那么我们来看下AsyncCall的execute方法。貌似好像又回到了之前同步的getResponseWithInterceptorChain()里面，根据返回的response来这只callback回调。