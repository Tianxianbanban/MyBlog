## HTTPClient和HttpURLConnection的一些缺点



+ HttpClient 是 Apache 提供的库，提供了高效的、最新的支持 HTTP 协议的工具包，封装了众多的 http 请求、响应等方法，但是太重量级了，API 数量过多；HttpURLConnection 是 SUN 公司的类库，是轻量级的 HTTP 框架，提供的方法都是一些我们进行 http 操作常用的，因而如果你想进行实现一些比较高级的功能比如代理、会话或者 Cookie 方面的，就需要自己写了。

+ HttpURLConnection 直接支持 GZIP 压缩，从 2.3 版本开始自动在每个发出的请求的请求头中加入 Accept-Encoding:gzip；HttpClient 也支持，但是需要自己写。 

+ HttpURLConnection 支持系统级连接池，即打开的连接不会直接关闭，在一段时 间内所有应用程序可以共用；HttpClient 也能做到，但至少不如直接系统级支持好。    

+ 从 4.0 版本开始，HttpURLConnection 在系统层面也开始支持缓存机制，加快了 重复请求的次数； 