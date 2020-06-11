---
title: OkHttp源码分析
date: 2020-06-11 19:48:00
tags:
---
### 前言
OkHttp源码分析网上已经有很多相关文章了，本人写的质量还不如其他的文章，但是希望通过文章的编写加深自己对 OkHttp 源码的印象

首先我们看下 OkHttp 最基本的使用，在出追踪它内部的逻辑。
```java
OkHttpClient okHttpClient = new OkHttpClient();
Request request = new Request.Builder().url("https://www.baidu.com").build();
//同步请求
okHttpClient.newCall(request).execute();
//异步请求
okHttpClient.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {

    }
});
```
一个请求开始都需要调用 OkHttp 的 newCall 方法
```java
@Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
}
```
OkHttp 的 newCall 方法实际有调用的是 RealCall 静态方法 newRealCall 来实例化 Call 对象
```java
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    call.transmitter = new Transmitter(client, call);
    return call;
}
```
RealCall 的 execute 方法比较简单。首先判断又没执行过，有执行过直接抛出异常 (这里表示一个 Call 只能执行一次请求，如果还要请求一次应该调用 Call.clone 来产生一个新对象执行)，再调用 dispatcher 的相关方法，最后的返回结果是调用的 getResponseWithInterceptorChain 产生的。
```java
@Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    transmitter.timeoutEnter();
    transmitter.callStart();
    try {
      client.dispatcher().executed(this);
      return getResponseWithInterceptorChain();
    } finally {
      client.dispatcher().finished(this);
    }
}
```
dispatcher 调度器，它内部有三个队列 readyAsyncCalls、runningAsyncCalls、runningSyncCalls分别存储着待运行的异步请求、运行中的异步请求、运行中的同步请求。execute 方法执行的时候会先调用 dispatcher 的 execute 发自己加到 dispatcher 的 runningSyncCalls 队列中。结束后调用 dispatcher 的 finished 移除队列。

```java
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
}
```

getResponseWithInterceptorChain 最后将我们之前设置的过滤器和系统默认实现的过滤器添加到一个容器里，然后作为参数构造出一个 RealInterceptorChain 并调用它的 proceed 方法。下面介绍系统过滤器的大致功能。

- retryAndFollowUpInterceptor——失败和重定向过滤器
- BridgeInterceptor——封装request和response过滤器
- CacheInterceptor——缓存相关的过滤器，负责读取缓存直接返回、更新缓存
- ConnectInterceptor——负责和服务器建立连接，连接池等
- networkInterceptors——配置 OkHttpClient 时设置的 networkInterceptors
- CallServerInterceptor——负责向服务器发送请求数据、从服务器读取响应数据(实际网络请求)



```java
public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      RealConnection connection) throws IOException {
    if (index >= interceptors.size()) throw new AssertionError();

    calls++;

    // If we already have a stream, confirm that the incoming request will use it.
    if (this.httpCodec != null && !this.connection.supportsUrl(request.url())) {
      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
          + " must retain the same host and port");
    }

    // If we already have a stream, confirm that this is the only call to chain.proceed().
    if (this.httpCodec != null && calls > 1) {
      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
          + " must call proceed() exactly once");
    }

    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
        connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
        writeTimeout);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);

    // Confirm that the next interceptor made its required call to chain.proceed().
    if (httpCodec != null && index + 1 < interceptors.size() && next.calls != 1) {
      throw new IllegalStateException("network interceptor " + interceptor
          + " must call proceed() exactly once");
    }

    // Confirm that the intercepted response isn't null.
    if (response == null) {
      throw new NullPointerException("interceptor " + interceptor + " returned null");
    }

    if (response.body() == null) {
      throw new IllegalStateException(
          "interceptor " + interceptor + " returned a response with no body");
    }

    return response;
}
```
再来看看 RealInterceptorChain 对象的 proceed 方法。它内部调用了 interceptors[index] 的 intercept 方法，并将 index + 1 构造出一个新的 RealInterceptorChain 对象作为参数传进去。而一般的 Interceptor 的 intercept 方法内部又会调用传进来参数 Chain 的 proceed 方法，根据前面说的 RealInterceptorChain 的 proceed 就相当于调用了 interceptors[index + 1] 的 intercept 方法。这样就形成了一个链式(递归)调用的现象，典型的责任链模式。