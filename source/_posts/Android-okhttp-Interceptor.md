---
title: Android开发-OkHttp拦截器
date: 2021-08-19 18:38:56
tags: Android
categories: Android
---

### 1. RetryAndFollowUpInterceptor (重定向拦截器)

客户端向服务器发送一个请求，获取对应的资源，服务器收到请求后，发现请求的这个资源实际放在另一个位置，于是服务器在返回的响应头的Location字段中写入那个请求资源的正确的URL，并设置reponse的状态码为30x 。

1. 创建一个 StreamAllocation。 
2. 调用下一个chain 。 
3. 判断是否重定向。重定向会对request做一些修改，返回新的request，否则会返回null。
4. followUp == null 无重定向，释放资源，直接返回response。
5. 比较重定向前后的 host、port、scheme是否一致，一致的话复用，否则重新创建 StreamAllocation。
6. 通过 while (true) ，重复步骤 2

### 2. BridgeInterceptor (桥接拦截器)

负责把用户构造的请求转换为发送到服务器的请求、把服务器返回的响应转换为用户友好的响应。

1. 如果这个请求有请求体，就添加 Content-Type, Content-Length等。
2. 如果这个请求没有 Host，就通过 url 来获取 Host 值添加到 Header中。 
3. 如果这个请求没有接收的数据类型Accept-Encoding，且没指定接收的数据范围，就添加默认接受格式为 gzip。
4. 去 CookieJar 中根据 url 查询 Cookie 添加到 Header。
5. 如果当前没有，就添加 User-Agent 信息。 发起请求后： 
6. 解析响应 Header 中的 Cookie 
7. 如果想要数据的格式是 gzip，就创建 GzipSource 进行解压，同时移除 Content-Encoding 和 Content-Length

### 3. CacheInterceptor (缓存拦截器)

1. 根据Request和之前缓存的Response得到CacheStrategy 
2. 根据CacheStrategy决定是请求网络还是直接返回缓存 
3. 如果 step 2中决定请求网络，则在这一步将返回的网络响应和本地缓存对比，对本地缓存进行改增删操作

### 4. ConnectInterceptor (连接拦截器)

okhttp内部维护了可以重复使用的 Socket 连接池，减少握手次数，加快请求响应。

1. 连接 RealConnection 是对 Socket 的封装。
2. OkHttp 的连接复用主要是通过 StreamAllocation 来实现的，每个连接上持有一个。
3. StreamAllocation 引用的列表，以此来标识当前连接是否空闲。它里面维护了List<Reference>的引用。List中StreamAllocation的数量也就是socket被引用的计数，如果计数为0的话，说明此连接没有被使用就是空闲的，需要通过下文的算法实现回收；如果计数不为0，则表示上层代码仍然引用，就不需要关闭连接。
4. 判断连接是否可以重用，除了比较连接当前的 host，也可以比较路由信息。 
5. 连接池在添加新连接时会运行清理任务，默认最多空闲连接为 5，最长空闲时间为 5 分钟。

### 5. CallServerInterceptor(读写拦截器)

负责实现网络 IO，所有拦截器都要依赖它才能拿到响应数据。

1. CallServerInterceptor 首先会将请求中的 header 写给服务器。
2. 如果有请求体的话，会再将 body 发送给服务器。 
3. 最后通过httpCodec.finishRequest() 结束 http 请求的发送。 
4. 然后从连接中读取服务器的响应，并构造 Response。
5. 如果请求的 header或服务器响应的 header 中，Connection 的值为 close，就关闭连接。
6. 最后返回 Response。
