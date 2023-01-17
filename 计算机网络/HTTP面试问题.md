## HTTP面试问题

### HTTP状态码？

HTTP响应的状态码指示了特定的HTTP请求是不是已经成功完成了，响应分为**5类**：**信息响应**（100-199）、**成功响应**（200-299）表明请求被正常处理了、**重定向**（300-399）表明浏览器需要执行某些特殊的处理以正确处理请求、**客户端错误**（400-499）表明客户端是发生错误的原因所在、**服务器错误**（500-599）表明服务器本身发生错误。

100 Continue 表明迄今为止的所有内容都是可行的，客户端应该继续请求，如果已经完成，则忽略它。

**200 ok** 正常处理，表示从客户端发来的请求在服务器端被正常处理了。

204 No Content 请求资源成功但没有资源返回，一般在只需要从客户端往服务器发送信息，而对客户端不需要发送新信息内容的情况下使用。

206 Partial Content 表示客户端进行了范围请求，而服务器成功执行了这部分的 GET 请求，响应报文中包含由 **Content-Range** 指定范围的实体内容。

**301 Moved Permanently** 被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个 URI 之一。

302 Found  请求的资源现在临时从不同的 URI 响应请求。由于这样的重定向是临时的，客户端应当继续向原有地址发送以后的请求。

**304 Not Modified** 表示客户端发送附带条件的请求时，服务器端允许请求访问资源，但未满足条件的情况。与缓存相关。

**400 Bad Request**  语义有误，当前请求无法被服务器理解。除非进行修改，否则客户端不应该重复提交这个请求；请求参数有误。

403 Forbidden 表明对请求资源的访问被服务器拒绝了。

**404 Not Found** 服务器没有所请求的资源或者拒绝访问。

500 Internal Server Error 服务器遇到了不知道如何处理的情况。

**503 Service Unavailable** 服务器没有准备好处理请求， 常见原因是服务器因维护或重载而停机。

[MDN]( https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status )

[具有代表性的 HTTP 状态码](https://juejin.im/post/5a276865f265da432c23b8d2#comment)



### 说一下https？  

**语言不简练**

首先HTTPS就是建立在安全层（SSL/TLS）之上的HTTP，通过**加密**把HTTP消息变得安全。

因为HTTP之所以不安全的原因，其中一个是因为**网络结构**的特点，信息传输的过程中会经过许多的结点，不免会被窃取或者篡改或冒充；而且HTTP是**明文传输**，消息内容不做加工直接传输。所以HTTP的不安全性就需要进行解决。但是产生一个新的协议代价是很高的，所以HTTPS不是一个新的协议，他的做法就是使用了一个叫做**SSL/TSL**的安全层，以此提供**数据加密**的支持，让HTTP消息运行在这个安全层的上面，在性能和安全性之间达到一个平衡。

HTTPS和HTTP相比：HTTP 是明文传输，而HTTPS 通过 SSL\TLS 进行了**加密**（在没有使用这个安全层的时候，应用程序的数据是通过TCP套接字与运输层进行交互的；使用了SSL/TLS之后通过这个安全层也加强了TCP的服务，HTTP和SSL子层之间还有SSL套接字，作用是跟TCP套接字相似的。发送方，SSL从SSL套接字接收应用层的数据，对数据进行加密，然后把加密的数据送往TCP套接字；接收方SSL会从TCP套接字读取数据，解密后通过SSL套接字把数据交给应用层）；**身份验证**SSL/TLS提供的安全服务有服务器鉴别、客户鉴别、加密SSL会话（对客户和服务器之间发送的所有报文进行加密，并检测是否篡改） ； TCP的HTTPS端口号是443，而不是平时HTTP端口号的80； HTTPS 需要到 CA 申请**证书**，一般免费证书很少，需要交费 ； HTTPS 的连接很简单，是无状态的；HTTPS 协议是由 SSL+HTTP 协议构建的可进行加密传输、身份认证的网络协议，比 HTTP 协议安全。

 HTTPS 最主要的用处是以下两点：

- 建立一个信息安全通道，来保证数据传输的安全
- 确认网站的真实性，防止钓鱼网站



### 说一下http、https的过程？

+ HTTPS

  + 握手阶段[阮一峰SSL/TLS协议运行机制的概述]( http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html )
  + https://www.cnblogs.com/snowater/p/7804889.html
  
+ HTTPS握手的详细过程、为什么要用非对称密钥？
  
  **非对称加密只用在了证书验证的阶段，一直到客户端将秘密数使用服务端公钥加密后传输到服务端私钥解密**。
  
  使用非对称加密是因为它的一对公钥私钥的使用中，私钥加密公钥解，公钥加密私钥解的特点。利用这个特点，可以对数字签名进行进行**鉴别**，来保证它的**完整性**以及使发送方**不可否认**的凭证。因为其他人如果对签名篡改了，公钥解密会得出不可读的明文，就可以知道被篡改了；第三方公证机构也可以利用公钥去证实确实是发送方发送的签名，保证了发送方不能否认。
  
  之后由秘密数生成对话密钥用于会话当中的对称加密，则是因为**对称加密比非对称加密速度更快**。



### https的过程中是否会有安全隐患？

[https降级攻击]( https://juejin.im/post/58501c8f2f301e00573e7197 )旧的协议版本会存在安全隐患，攻击者仍然可以在使用旧版本协议的时候针对漏洞进行攻击。



### HTTP短连接和长连接？

HTTP的短连接和长连接实际上是TCP的短连接长连接



### POST和GET的区别？

+ 语义上，get**获取资源**并且可以把参数放在url当中，而post是**新增资源**request body传递参数。
+ get请求url有参数限制，post的requestbody没有参数限制。
+ get请求的安全性较低。
+ 幂等性上来说，get是幂等的，而post是非幂等的。
+ [关于restful的幂等性](http://blog.720ui.com/2016/restful_idempotent/)
+ [关于两种方法区别的思考](https://www.jianshu.com/p/fd67b576365d)



### 用的哪一个HTTP版本?

项目里面用的是HTTP/1.1的版本



### 列举HTTP Header中的字段

HTTP消息头可以让客户端和服务器通过请求和响应**传递附加的数据**。

首先**根据不同上下文**，可以将消息头分为四类：

+ [General headers](https://developer.mozilla.org/en-US/docs/Glossary/General_header): 同时适用于请求和响应消息，但与最终消息主体中传输的数据无关的消息头。 通用首部可以是响应头部或者请求头部。但是不可以是实体头部。比如：Date、Cache-Control、Connection

  Date 请求发送的日期和时间

  Cache-Control 控制缓存时间

  Connection 设置Keep-Alive可以减少TCP建立和销毁的成本，但是占用端口时间长，高并发时需要考虑；而设置成Close：每次连接将使用新的TCP连接。

+ [Request headers](https://developer.mozilla.org/en-US/docs/Glossary/Request_header): 包含更多有关要获取的资源或客户端本身信息的消息头。 它可在 HTTP 请求中使用，并且和请求主体无关 。比如Accept、Cookie。

  Accept 用户代理期望的MIME 类型列表

  Cookie 携带Cookie信息

+ [Response headers](https://developer.mozilla.org/en-US/docs/Glossary/Response_header): 包含有关响应的补充信息，如其位置或服务器本身（名称和版本等）的消息头。 被用于http响应中并且和响应消息主体无关的那一类。比如：Age、Location、Server

  Age 从原始服务器到代理缓存形成的估算时间（以秒计，非负）

  Location 重定向标识新的资源的位置

+ [Entity headers](https://developer.mozilla.org/en-US/docs/Glossary/Entity_header): 包含有关实体主体的更多信息。 用来描述消息体内容。比如：Content-length、Content-Language、Content-Encoding

  Content-length 代表请求体的大小，或者响应体内容的大小。

  Content-Language 响应体的语言

  Content-Encoding 压缩文本数据能减少带宽并加快显示速度

  Accept-Encoding 用户代理支持的压缩方法

 消息头也可以**根据代理对其的处理方式**分为：

+   **端到端消息头** ， 必须被传输到最终的消息接收者，也即，请求的服务器或响应的客户端。中间的代理服务器必须转发未经修改的端到端消息头，并且必须缓存它们。 
+  **逐跳消息头** ， 这类消息头仅对单次传输连接有意义，不能通过代理或缓存进行重新转发。这些消息头包括 [`Connection`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Connection), [`Keep-Alive`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Keep-Alive), [`Proxy-Authenticate`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Proxy-Authenticate), [`Proxy-Authorization`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Proxy-Authorization), [`TE`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/TE), [`Trailer`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Trailer), [`Transfer-Encoding`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Transfer-Encoding) 及 `Upgrade`。注意，只能使用 [`Connection`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Connection) 来设置逐跳一般头。 

其他：

Accept-Charset 列出用户代理支持的字符集

Accept-Language 列出用户代理期望的页面语言

Authorization 包含用服务器验证用户代理的凭证

Content-Security-Policy 指示用户代理在一个页面上可以加载使用的资源。

Content-Type 指示服务器文档的MIME类型。帮助用户代理去处理接收到的数据。

Vary 列出了用作web服务器选择特定内容的条件的标头。此服务器对于高效和正确缓存发送的资源很重要。

等等

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)

[Http--Header](https://www.jianshu.com/p/59d36b01608d)



### 网络安全方面的协议是否了解

（HTTPS SSL的加密机制）



### HTTP 方法

表明要对给定资源执行的操作 

+ GET 请求一个指定资源
+ POST 将实体提交到指定的资源，会导致服务器的状态变化
+ PUT **更新资源**（PUT和POST的区别在于， PUT方法是幂等的：调用一次与连续调用多次是等价的，就是没有副作用，而连续调用多次POST方法可能会有副作用，比如将一个订单重复提交多次。 ）
+ DELETE 删除指定的资源。
+ OPTIONS 用于描述目标资源的通信选项。
+ PATCH 用于对资源应用**部分更新**。
+ TRACE 沿着到目标资源的路径执行一个消息环回测试。
+ HEAD 请求一个与**GET**请求的响应相同的响应，但**没有响应体**。
+ CONNECT建立一个到由目标资源标识的服务器的隧道。



### **HTTP2和HTTP1**

[MDN Doc](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP)

`HTTP/0.9 单行协议`

  + 请求由单行指令构成，以唯一可用的GET方法开头，后接目标资源路径。响应也极简单，只含响应文档本身。

`HTTP/1.0 构建可扩展性`

  + **协议版本**信息会随每个请求发送
  + **状态码**会在响应头。
  + 引入了**HTTP头**的概念，无论对于请求还是响应，就允许传输元数据，使协议变得更灵活，更具有扩展性
  + 在新HTTP头的帮助下，具备传输除了纯文本HTML以外的**其他类型文档**的能力（比如content_type头）

`HTTP/1.1 标准化协议`

  + 使用了TCP的**持续连接**，持续连接有两种工作方式，**非流水线**方式（空闲状态多，浪费服务器资源）和**流水线**方式（空闲时间少，提高了效率）。
  + 支持**响应分块**
  + 引入额外的**缓存控制**机制
  + 引入**内容协商**机制，包括语言、编码、类型等，并且允许客户端和服务器之间约定以最合适的内容进行交换。
  + **host头**可以使不同域名配置在同一个IP地址的服务器上。

`HTTP/2`

  + **二进制格式解析**：HTTP/1.x解析基于文本，考虑到文本多样性以及健壮性考虑，HTTP2.0的协议解析采用二进制格式，压缩请求响应的体积。

  + 多路复用：就是连接共享， 允许同时通过单一的 HTTP/2 连接发起**多重的请求-响应消息**，合并多个请求或者响应为一个。

  + **header压缩**： HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。 

  + **服务端推送**： 客户端所需要的资源伴随着响应一起发送到客户端，省去了客户端重复请求的步骤。 

    [阮一峰 关于服务器推送]( http://www.ruanyifeng.com/blog/2018/03/http2_server_push.html )
    
    [http2 简介](https://juejin.im/post/5aaccf8f51882555784dbabc#heading-4)
    
    [HTTP2 详解](https://juejin.im/post/5b88a4f56fb9a01a0b31a67e#heading-62)

`HTTP/3`

传输层部分使用 QUIC，而不是TCP。QUIC 通过 UDP 运行多个流，并为每个流独立实现数据包丢失检测和重传，因此如果发生错误，只有该数据包中包含数据的流才会被阻止。



### **HTTP的长轮询**

[长连接、短连接、长轮询、短轮询](https://www.jianshu.com/p/00daa2d84266)



### **浏览器输入URL回车后**

[浏览器输入URL回车后](https://juejin.im/post/59e2e928f265da430c10d996#comment)

[在浏览器中输入一个URL会发生什么？](https://blog.csdn.net/jochebed666/article/details/88377253)

如果是http访问：

1. **DNS域名解析**，查看本机DNS高速缓存（浏览器中的缓存，或者操作系统中的hosts文件？）是否有当前域名到IP地址的映射，如果没有就成为DNS的一个客户，将待解析的域名放在DNS请求报文中，以UDP用户数据报的形式发送给本地域名服务器，然后本地域名服务器如果能够查找到结果，就将IP地址放在回应报文中返回。而如果本地域名服务器无法回应这个请求，它就会向其他域名服务器发出查询请求，根域名服务器、顶级域名服务器、权限域名服务器，直到能够回答出这个域名对应的IP地址。

   应用层获得了IP地址就可以进行通信了。**封装好HTTP请求报文**。

2. 应用层获得目的IP地址以后，加上目的IP地址对应的默认使用80端口或者使用其他端口号，组成套接字Socket，给数据添加TCP首部，运输层建立**TCP连接**。

   [nginx基础概念(100%)之keepalive](https://blog.csdn.net/wangpengqi/article/details/17245349)内容正确性有待证实，只做参考。

3. 网络层会添加IP首部，包括了从上层获得的源IP地址和目的IP地址等信息。

   然后根据**ARP地址解析协议**由IP地址得出MAC地址，如果要通信的目的主机和源主机是同一片局域网的，那么可以从本机的ARP高速缓存中就可以得出这个硬件地址，或者通过ARP进程在本局域网上广播发送一个包含自己IP地址和硬件地址的ARP请求分组，得到目的主机的ARP响应，然后源主机和目的主机都更新ARP高速缓存，添加对方的IP地址到硬件地址的映射信息；如果不在同一片局域网，就把IP数据报通过和源主机连接在同一局域网的路由器来转发，这个转发的过程要先把路由器的IP地址解析为硬件地址，然后路由器收到IP数据报以后从转发表中找到下一跳路由器（路由器连接了不同的网络，细节就是路由选择协议），由这个路由器继续转发。

   在这里如果是从专用网的主机发送出去，或者发往专用网上的主机，还会涉及**NAT地址转换**。

4. 数据链路层添加首部尾部。

5. 物理层将分组传递到物理传输媒介。



### **HTTP缓存**

[HTTP----HTTP缓存机制](https://juejin.im/post/5a1d4e546fb9a0450f21af23)

