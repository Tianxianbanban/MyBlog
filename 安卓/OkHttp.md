## OkHttp梳理



### **OkHttp的使用**

+ 添加依赖

+ 基本代码

  ```java
  class Util{
      public static void sendRequest(String url, Callback callback) {
          RequestBody requestBody = new FormBody.Builder()
                  .add("key","content")
                  .build();
          Request request = new Request.Builder()
                  .url(url)
                  .post(requestBody)
                  .build();
          OkHttpClient client = new OkHttpClient();
          client.newCall(request).enqueue(callback);//异步请求的方式
          //同步请求方式则是调用execute方法，并且有Response类型的返回值。
      }
  }
  ```



### OkHttp请求的整体流程

OkHttp请求过程中最少只需要接触OkHttpClient、Request、Call、 Response，但是内部会进行大量的逻辑处理。所有网络请求的逻辑大部分集中在**拦截器**中，但是在进入拦截器之前还需要依靠**分发器**来调配请求任务。

- 分发器：内部维护队列与线程池，完成请求调配。
- 拦截器：五大默认拦截器完成整个请求过程。

整个网络请求过程大致过程：

1. 通过建造者模式构建OKHttpClient与 Request。
2. OKHttpClient通过newCall发起一个新的请求。
3. 通过**分发器**维护请求队列与线程池，完成请求调配。
4. 通过五大**默认拦截器**完成请求重试，缓存处理，建立连接等一系列操作。
5. 得到网络请求结果。



### OKHttp分发器工作

分发器的主要作用是**维护请求队列与线程池**。（比如我们有100个异步请求，肯定不能把它们同时请求，而是应该把它们排队分个类，分为正在**请求中**的列表和**正在等待**的列表，请求完成后，可从等待中的列表中取出等待的请求，从而完成所有的请求。）

#### 同步请求

因为同步请求不需要线程池，也不存在任何限制。所以分发器仅做一下记录。后续按照加入队列的顺序同步请求即可。

```kotlin
//Dispatcher.kt
@Synchronized internal fun executed(call: RealCall) {
    runningSyncCalls.add(call)
  }
```

#### 异步请求

Dispatcher将AsyncCall加到了readyAsyncCalls的队尾，并检测是否存在**跟当前请求域名相同的请求**，如果存在则复用，然后调用promoteAndExecute方法处理请求。

```kotlin
//Dispatcher.kt
internal fun enqueue(call: AsyncCall) {
    synchronized(this) {
      readyAsyncCalls.add(call)

      // Mutate the AsyncCall so that it shares the AtomicInteger of an existing running call to
      // the same host.
      if (!call.call.forWebSocket) {
        val existingCall = findExistingCallWithHost(call.host) //检测
        if (existingCall != null) call.reuseCallsPerHostFrom(existingCall)
      }
    }
    promoteAndExecute() //处理请求
  }
```

遍历等待队列中的请求，当正在执行的任务未超过最大限制64，同时同一Host的请求不超过5个，则会添加到正在执行队列，同时提交给线程池。

```kotlin
//Dispatcher.kt
private fun promoteAndExecute(): Boolean {
    this.assertThreadDoesntHoldLock()

    val executableCalls = mutableListOf<AsyncCall>()
    val isRunning: Boolean
    synchronized(this) {
      val i = readyAsyncCalls.iterator()
      while (i.hasNext()) {
        val asyncCall = i.next()

        // maxRequests = 64 maxRequestsPerHost = 5
        if (runningAsyncCalls.size >= this.maxRequests) break // Max capacity. 
        if (asyncCall.callsPerHost.get() >= this.maxRequestsPerHost) continue // Host max capacity. 

        i.remove()
        asyncCall.callsPerHost.incrementAndGet()
        executableCalls.add(asyncCall) //
        runningAsyncCalls.add(asyncCall)
      }
      isRunning = runningCallsCount() > 0
    }

    for (i in 0 until executableCalls.size) {
      val asyncCall = executableCalls[i]
      asyncCall.executeOn(executorService) //
    }

    return isRunning
  }
```



### OKHttp拦截器工作（还可以继续补充）

经过上面分发器的任务分发，下面就要利用拦截器开始一系列配置了。

==（没看出这里谁调用的？）==

```kotlin
//RealCall.kt
override fun execute(): Response {
    check(executed.compareAndSet(false, true)) { "Already Executed" }

    timeout.enter()
    callStart()
    try {
      client.dispatcher.executed(this) //分发器分发
      return getResponseWithInterceptorChain() //责任链的构建与处理其实就是在这个方法里面。
    } finally {
      client.dispatcher.finished(this)
    }
  }
```

如下构建了一个OkHttp拦截器的责任链（就是用来处理相关事务责任的一条执行链，上有多个节点，每个节点都有机会（条件匹配）处理请求事务，如果某个节点处理完了就可以根据实际业务需求传递给下一个节点继续处理或者返回处理完毕）。网络请求就是这样经过责任链一级一级的递推下去，最终会执行到最后一个拦截器`CallServerInterceptor#intercept`方法，此方法会将网络响应的结果封装成一个Response对象并return。之后沿着责任链一级一级的回溯，最终就回到`getResponseWithInterceptorChain`方法的返回

```kotlin
//RealCall.kt
@Throws(IOException::class)
  internal fun getResponseWithInterceptorChain(): Response {
    // Build a full stack of interceptors.
    val interceptors = mutableListOf<Interceptor>() 
    interceptors += client.interceptors //应用拦截器
    interceptors += RetryAndFollowUpInterceptor(client) //重试重定向拦截器
    interceptors += BridgeInterceptor(client.cookieJar) //应用层和网络层的桥接拦截器
    interceptors += CacheInterceptor(client.cache) //缓存拦截器
    interceptors += ConnectInterceptor //连接拦截器
    if (!forWebSocket) {
      interceptors += client.networkInterceptors //网络拦截器，用户自定义拦截器，通常用于监控网络层的数据传输
    }
    interceptors += CallServerInterceptor(forWebSocket) //请求拦截器

    val chain = RealInterceptorChain(
        call = this,
        interceptors = interceptors,
        index = 0,
        exchange = null,
        request = originalRequest,
        connectTimeoutMillis = client.connectTimeoutMillis,
        readTimeoutMillis = client.readTimeoutMillis,
        writeTimeoutMillis = client.writeTimeoutMillis
    )

    var calledNoMoreExchanges = false
    try {
      val response = chain.proceed(originalRequest)
      if (isCanceled()) {
        response.closeQuietly()
        throw IOException("Canceled")
      }
      return response
    } catch (e: IOException) {
      calledNoMoreExchanges = true
      throw noMoreExchanges(e) as Throwable
    } finally {
      if (!calledNoMoreExchanges) {
        noMoreExchanges(null)
      }
    }
  }
```

#### ==五个默认拦截器==和两种自定义拦截器

+ 应用拦截器（用户自定义拦截器）

  拿到的是原始请求，可以添加一些自定义的header、通用参数、参数加密、网关接入等。

+ **RetryAndFollowUpInterceptor**（重试重定向拦截器）

  当请求内部抛出异常的时候，判定是否需要重试；当响应结果是3xx重定向的时候，决定是否构建一个新的请求并发送请求。

+ **BridgeInterceptor**（连接桥拦截器）

  应用层和网络层的桥接拦截器，把服务器返回的响应转换为用户友好的响应，是应用程序代码和网络代码的桥梁。

  1. 负责把用户构造的请求转换为发送到服务器的请求 ，转换的过程就是添加一些服务端需要的header信息，比如Host、Content-Length、Content-Type、User-Agent、Keep-alive（是实现连接复用的必要步骤）等；
  2. 主要工作是为请求添加cookie，保存响应结果的cookie；
  3. 设置gzip压缩如果响应使用gzip压缩过，则还需要进行解压。

+ **CacheInterceptor**（缓存拦截器）

  负责读取缓存以及更新缓存。他首先通过将**请求的url进行了md5加密后的转为了16进制的字符串作为key，取出在磁盘上用lru算法存储的缓存，接着会通过这个缓存获取相应的缓存策略**，即通过上次缓存的response提取他response header中的Cache-Control字段，判断这个请求是否为服务器下发的静态资源，如果是的话，就不去访问网络，而是返回缓存的response，还有一种情况也会不去访问网络，就是他缓存到现在的一个时间加上最小请求网络的时间小于他由服务器获取时的时间加上这个请求能够被客户端过期接受的时间，这样他也就不会去访问网络而是去返回缓存的response，这样就能很大程度上**减轻服务器的压力**。

+ **ConnectInterceptor**（连接拦截器）

  ==内部维护一个连接池，负责连接复用、创建连接（三报文握手）、释放连接以及创建连接上的Socket流。==

  负责和服务器建立连接，这里才是真正的请求网络。同时负责了Dns解析和Socket连接（包括tls连接）。开始执行的时候他先会先去找出一个健康的链接，**这个健康的标准是socket未关闭，输入输出流未关闭，若遇到不健康的连接则会将他从连接池中移除，并关闭连接。在寻找连接的时候，他会先根据目标地址尝试从连接池中获取这个链接，若没有匹配的链接，则新建立一个与目标地址连接的socket**。并将它的输入输出流用okio封装起来，在连接确立好之后用来后续发送请求与接收响应。

+ 网络拦截器（用户自定义拦截器）

  通常用于监控网络层的数据传输。

+ **CallServerInterceptor**（请求服务拦截器）

  真正的发起网络请求。执行流操作(写出请求体、获得响应数据) ，负责向服务器发送请求数据，从服务器读取响应数据，进行HTTP请求报文的封装与响应报文的解析。它是最后一个拦截器了，前面的拦截器已经完成了socket连接和tls连接，那么这一步就是传输HTTP的头部和body数据了。以及读取 response header，先构造一个 Response 对象，如果有 response body，就在 response header 的基础上加上 body 构造一个新的 Response 对象。

##### 五个默认拦截器逻辑

+ RetryAndFollowUpInterceptor（重试重定向拦截器）

+ BridgeInterceptor（连接桥拦截器）

+ CacheInterceptor（缓存拦截器）

+ ConnectInterceptor（连接拦截器）

  获取到了一个Exchange的类，能够引用到ExchangeCodec，包含两个子类一个是Http1ExchangeCodec，一个是Http2Exchangecodec，分别对应的是Http1协议和Http2协议。

  Exchange类里面包含了ExchangeCodec对象，而这个对象里面又包含了一个RealConnection对象，RealConnection的属性成员有socket、handlShake、protocol等，可见它应该是一个Socket连接的包装类，而ExchangeCode对象是对RealConnection操作（writeRequestHeader、readResposneHeader）的封装。

  socket连接其实已经建立了，可以通过realChain拿到socket做一些事情了，这也就是为什么称之为network Interceptor的原因。

+ CallServerInterceptor（请求服务拦截器）

##### 应用拦截器和网络拦截器的区别

从整个责任链路来看，应用拦截器是最先执行的拦截器，也就是用户自己设置request属性后的原始请求，而网络拦截器位于ConnectInterceptor和CallServerInterceptor之间，此时网络链路已经准备好，只等待发送请求数据。它们主要有以下区别：

1. 拦截器触发时机：应用拦截器在RetryAndFollowUpInterceptor和CacheInterceptor之前，所以一旦发生错误重试或者网络重定向，网络拦截器可能执行多次，但是应用拦截器永远只会触发一次。另外如果在CacheInterceptor中命中了缓存就不需要走网络请求了，因此会存在短路网络拦截器的情况。
2. 拦截器应用场景：应用拦截器因为只会调用一次，**通常用于统计客户端的网络请求发起情况**；而网络拦截器一次调用代表了一定会发起一次网络通信，因此通常可用于**统计网络链路上传输的数据**。

### OKHttp如何复用TCP连接？

ConnectInterceptor的主要工作就是负责建立TCP连接，建立TCP连接需要经历**三报文握手、四报文挥手**等操作，如果每个HTTP请求都要新建一个TCP消耗资源比较多。而Http1.1已经支持keep-alive，即多个Http请求复用一个TCP连接，OKHttp也做了相应的优化，下面我们来看下OKHttp是怎么复用TCP连接的。

ConnectInterceptor中查找连接的代码会最终会调用到ExchangeFinder.findConnection方法，具体如下：

```kotlin
//ExchangeFinder.kt
//为承载新的数据流 寻找 连接。寻找顺序是 已分配的连接、连接池、新建连接
@Throws(IOException::class)
  private fun findConnection(
    connectTimeout: Int,
    readTimeout: Int,
    writeTimeout: Int,
    pingIntervalMillis: Int,
    connectionRetryEnabled: Boolean
  ): RealConnection {
    if (call.isCanceled()) throw IOException("Canceled")

    // 1.尝试使用 已给数据流分配的连接.（例如重定向请求时，可以复用上次请求的连接）
    // Attempt to reuse the connection from the call.
    val callConnection = call.connection // This may be mutated by releaseConnectionNoEvents()!
    if (callConnection != null) {
      var toClose: Socket? = null
      synchronized(callConnection) {
        if (callConnection.noNewExchanges || !sameHostAndPort(callConnection.route().address.url)) {
          toClose = call.releaseConnectionNoEvents()
        }
      }

      // If the call's connection wasn't released, reuse it. We don't call connectionAcquired() here
      // because we already acquired it.
      if (call.connection != null) {
        check(toClose == null)
        return callConnection
      }

      // The call's connection was released.
      toClose?.closeQuietly()
      eventListener.connectionReleased(call, callConnection)
    }

    // We need a new connection. Give it fresh stats.
    refusedStreamCount = 0
    connectionShutdownCount = 0
    otherFailureCount = 0

     // 2. 没有已分配的可用连接，就尝试从连接池获取。（连接池稍后详细讲解）
    // Attempt to get a connection from the pool.
    if (connectionPool.callAcquirePooledConnection(address, call, null, false)) {
      val result = call.connection!!
      eventListener.connectionAcquired(call, result)
      return result
    }

    // Nothing in the pool. Figure out what route we'll try next.
    val routes: List<Route>?
    val route: Route
    if (nextRouteToTry != null) {
      // Use a route from a preceding coalesced connection.
      routes = null
      route = nextRouteToTry!!
      nextRouteToTry = null
    } else if (routeSelection != null && routeSelection!!.hasNext()) {
      // Use a route from an existing route selection.
      routes = null
      route = routeSelection!!.next()
    } else {
      // Compute a new route selection. This is a blocking operation!
      var localRouteSelector = routeSelector
      if (localRouteSelector == null) {
        localRouteSelector = RouteSelector(address, call.client.routeDatabase, call, eventListener)
        this.routeSelector = localRouteSelector
      }
      val localRouteSelection = localRouteSelector.next()
      routeSelection = localRouteSelection
      routes = localRouteSelection.routes

      if (call.isCanceled()) throw IOException("Canceled")

      // 现在我们有了一组 IP 地址，再次尝试从池中获取连接。 由于连接合并，我们有更好的匹配机会。
      // Now that we have a set of IP addresses, make another attempt at getting a connection from
      // the pool. We have a better chance of matching thanks to connection coalescing.
      if (connectionPool.callAcquirePooledConnection(address, call, routes, false)) {
        val result = call.connection!!
        eventListener.connectionAcquired(call, result)
        return result
      }

      route = localRouteSelection.next()
    }

    // Connect. Tell the call about the connecting call so async cancels work.
    val newConnection = RealConnection(connectionPool, route)
    call.connectionToCancel = newConnection
    try {
      // 4.第二次没成功，就把新建的连接，进行TCP + TLS 握手，与服务端建立连接. 是阻塞操作
      newConnection.connect(
          connectTimeout,
          readTimeout,
          writeTimeout,
          pingIntervalMillis,
          connectionRetryEnabled,
          call,
          eventListener
      )
    } finally {
      call.connectionToCancel = null
    }
    call.client.routeDatabase.connected(newConnection.route())

    // 5. 最后一次尝试从连接池获取，注意最后一个参数为true，即要求 多路复用（http2.0）意思是，如果本次是http2.0，那么为了保证 多路复用性，（因为上面的握手操作不是线程安全）会再次确认连接池中此时是否已有同样连接
    // If we raced another call connecting to this host, coalesce the connections. This makes for 3
    // different lookups in the connection pool!
    if (connectionPool.callAcquirePooledConnection(address, call, routes, true)) {
      val result = call.connection!!
      nextRouteToTry = route
      // 如果获取到，就关闭我们创建里的连接，返回获取的连接
      newConnection.socket().closeQuietly()
      eventListener.connectionAcquired(call, result)
      return result
    }

    synchronized(newConnection) {
       //最后一次尝试也没有的话，就把刚刚新建的连接存入连接池
      connectionPool.put(newConnection)
      call.acquireConnectionNoEvents(newConnection)
    }

    eventListener.connectionAcquired(call, newConnection)
    return newConnection
  }
```

连接拦截器使用了5种方法查找连接: (代码注释和以下流程可能还需要校正！)

1. 首先会尝试使用已给请求分配的连接。（已分配连接的情况例如重定向时的再次请求，说明上次已经有了连接）
2. 若没有已分配的可用连接，就尝试从连接池中匹配获取。因为此时没有路由信息，所以匹配条件：address一致——host、port、代理等一致，且匹配的连接可以接受新的请求。
3. 若从连接池没有获取到，则传入routes再次尝试获取，这主要是针对Http2.0的一个操作，Http2.0可以复用square.com与square.ca的连接。
4. 若第二次也没有获取到，就创建RealConnection实例，进行TCP + TLS握手，与服务端建立连接。
5. 此时为了确保Http2.0连接的多路复用性，会第三次从连接池匹配。因为新建立的连接的握手过程是非线程安全的，所以此时可能连接池新存入了相同的连接。
6. 第三次若匹配到，就使用已有连接，释放刚刚新建的连接；若未匹配到，则把新连接存入连接池并返回。

### OKHttp空闲连接如何清除？

上面说到我们会建立一个TCP连接池，但如果没有任务了，空闲的连接也应该及时清除，OKHttp是如何做到的呢？

```Kotlin
//RealConnectionPool.kt
fun cleanup(now: Long): Long {
    var inUseConnectionCount = 0 //正在使用的连接数
    var idleConnectionCount = 0 //空闲连接数
    var longestIdleConnection: RealConnection? = null //空闲时间最长的连接
    var longestIdleDurationNs = Long.MIN_VALUE //最长的空闲时间

  //遍历连接：找到待清理的连接, 找到下一次要清理的时间（还未到最大空闲时间）
    // Find either a connection to evict, or the time that the next eviction is due.
    for (connection in connections) {
      synchronized(connection) {
        //若连接正在使用，continue，正在使用连接数+1
        // If the connection is in use, keep searching.
        if (pruneAndGetAllocationCount(connection, now) > 0) {
          inUseConnectionCount++
        } else {
          //空闲连接数+1
          idleConnectionCount++

          // 赋值最长的空闲时间和对应连接
          // If the connection is ready to be evicted, we're done.
          val idleDurationNs = now - connection.idleAtNs
          if (idleDurationNs > longestIdleDurationNs) {
            longestIdleDurationNs = idleDurationNs
            longestIdleConnection = connection
          } else {
            Unit
          }
        }
      }
    }

    when {
      //若最长的空闲时间大于5分钟 或 空闲数 大于5，就移除并关闭这个连接
      longestIdleDurationNs >= this.keepAliveDurationNs
          || idleConnectionCount > this.maxIdleConnections -> {
        // We've chosen a connection to evict. Confirm it's still okay to be evict, then close it.
        val connection = longestIdleConnection!!
        synchronized(connection) {
          if (connection.calls.isNotEmpty()) return 0L // No longer idle.
          if (connection.idleAtNs + longestIdleDurationNs != now) return 0L // No longer oldest.
          connection.noNewExchanges = true
          connections.remove(longestIdleConnection)
        }

        connection.socket().closeQuietly()
        if (connections.isEmpty()) cleanupQueue.cancelAll()

        // Clean up again immediately.
        return 0L
      }

      // else，就返回 还剩多久到达5分钟，然后wait这个时间再来清理
      idleConnectionCount > 0 -> {
        // A connection will be ready to evict soon.
        return keepAliveDurationNs - longestIdleDurationNs
      }

      //连接没有空闲的，就5分钟后再尝试清理.
      inUseConnectionCount > 0 -> {
        // All connections are in use. It'll be at least the keep alive duration 'til we run
        // again.
        return keepAliveDurationNs
      }

      else -> {
        // 没有连接，不清理
        // No connections, idle or in use.
        return -1
      }
    }
  }
```

思路还是很清晰的：

1. 在将连接加入连接池时就会启动定时任务。
2. 有空闲连接的话，如果最长的空闲时间大于5分钟或空闲数大于5，就移除关闭这个最长空闲连接；如果空闲数不大于5且最长的空闲时间不大于5分钟，就返回到5分钟的剩余时间，然后等待这个时间再来清理。
3. 没有空闲连接就等5分钟后再尝试清理。
4. 没有连接不清理。



### OKHttp有哪些优点？

1. 使用简单，在设计时使用了外观模式，将整个系统的复杂性给隐藏起来，将子系统接口通过一个客户端OkHttpClient统一暴露出来。
2. 扩展性强，可以通过**自定义应用拦截器与网络拦截器**，完成用户各种自定义的需求。
3. 功能强大，支持Spdy、Http1.X、Http2、以及WebSocket等**多种协议**。
4. 通过连接池**复用底层TCP(Socket)**，减少请求延时
5. 无缝的支持GZIP减少数据流量。
6. 支持数据缓存，减少重复的网络请求。
7. 支持请求失败自动重试主机的其他ip，自动重定向。



### OKHttp框架中用到了哪些设计模式？

1. 构建者模式：OkHttpClient与Request的构建都用到了构建者模式。
2. 外观模式：OkHttp使用了外观模式,将整个系统的复杂性给隐藏起来，将子系统接口通过一个客户端OkHttpClient统一暴露出来。很多框架都会提供最基本的使用接口，而框架的重构与维护等也只用面向这个接口，而使用框架的时候就只需要面向这些简单的接口，比如client.newCall(request).enqueue(callback)。
3. 责任链模式：OKHttp的核心就是责任链模式，通过5个默认拦截器构成的责任链完成请求的配置。一个Interceptor对应一个功能，所有Interceptor连接成一个Interceptor.Chain，典型的责任链模式实现。
4. 享元模式：享元模式的核心即池中复用，OKHttp复用TCP连接时用到了连接池，同时在异步请求中也用到了线程池。
5. 工厂方法



---

**OkHttp的原理**

+ OkHttp子系统层级结构
  + 网络配置层
  + 重定向层
  + Header拼接层
  + HTTP缓存层
  + 网络连接层
  + 数据响应层

+ 五个默认拦截器

  + RetryAndFollowUpInterceptor（重试重定向拦截器）

  + BridgeInterceptor（连接桥拦截器）

  + CacheInterceptor（缓存拦截器）

  + ConnectInterceptor（连接拦截器）

  + CallServerInterceptor（请求服务拦截器）

+ 连接机制

  + 创建连接
  + 连接池



### **相关文章**

[OPPO互联网基础技术团队-OkHttp源码深度解析](https://juejin.im/post/5e7b27a0e51d4567eb6bc8a9)

[郭孝星-Android开源框架源码鉴赏：Okhttp](https://juejin.im/post/5a704ed05188255a8817f4c9#heading-5 )

[Andriod 网络框架 OkHttp 源码解析](https://juejin.im/post/5bc89fbc5188255c713cb8a5#heading-10)

[[知识点]OkHttp原理8连问](https://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650267613&idx=1&sn=a4f1ae77700723e548cf9209bbab61b5&chksm=88632eb2bf14a7a44d752ef76ebdb55297c9478ca5bd485d13167f0ee985c5e7b172fd984996&mpshare=1&scene=23&srcid=11105nTuD6fAB9LBoN6F17j2&sharer_sharetime=1668016634729&sharer_shareid=2f8ce22845d2e6b396d5027db882cc52%23rd)

[Okhttp缓存源码分析以及自定义缓存实现](https://mp.weixin.qq.com/s/Y_TFsLwVBQw_PVEt7PF-NA)



**OkHttp的应用**

+ Get请求
+ Post请求
+ 上传文件、下载文件、断点续传
  + [Android Okhttp 断点续传面试解析](https://juejin.im/post/5ceab960e51d4510926a7acf#comment)

+ 上传Multipart文件
+ 设置超时时间
+ 缓存和自定义缓存
+ 取消请求
+ 等等
