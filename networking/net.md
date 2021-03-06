### 1. TCP <div id="tcp"/>

#### 1.1 TCP 与 UDP 的区别？

TCP，全称是传输控制协议，是一种面向连接的、可靠的、基于字节流的传输层通信协议。

UDP，全称用户数据报协议，在发送端，应用层将数据传递给传输层的 UDP 协议，UDP 只会给数据增加一个 UDP头部 标识这是 UDP 协议，然后就传递给网络层了；在接收端，网络层将数据传递给传输层，UDP 只去除 IP 报文头就传递给应用层，不会任何拼接操作；UDP 不止支持一对一的传输方式，同样支持一对多，多对多，多对一的方式，也就是说 UDP 提供了单播，多播，广播的功能。



##### 1.1.1 TCP 可靠传输

TCP 的可靠性是建立在：

1. 数据校验
2. 序列号/ 确认应答
3. 超时重传，间隔加倍
4. 滑动窗口控制。发一个包就停止等待 ACK 没有效率，TCP 使用滑动窗口来一次发送多个数据包。
5. 拥塞控制



##### 1.1.2 TCP 流量控制

TCP 头里有一个字段叫 Window，用于接收方通知发送方自己还有多少缓冲区可以接收数据。发送方根据接收方的处理能力来发送数据，不会导致接收方处理不过来，这就是流量控制。

滑动窗口会一次发送 N 个数据包，此时出现超时等事件时：

1. 将窗口回退到上一次确认的位置，重新发送 N 个（窗口内仅第一个设置计时器）：Go-Back-N协议
2. 仅选择需要重传的数据报进行重传（窗口内每个数据报都有计时器）： Selective Repeat

TCP 使用 Go-Back-N 的变种。



##### 1.1.3 TCP 拥塞控制

拥塞控制是为了防止网络拥塞。

1. 慢启动

当建立连接时，窗口大小 cwnd 的值设置为1（不一定，只是一个较小值），然后逐渐翻倍，到达门限值 threshold 时，进入拥塞避免状态。

2. 快恢复

threshold 为当前 cwnd 的一半，cwnd = 新的threshold + 3， 将丢失的报文进行重传，如果重传成功，进入拥塞避免状态，如果仍然丢失，进入慢启动状态

3. 拥塞避免

此状态，cwnd 开始线性增长，逐次加1


* 当 cwnd 追上 threshold，进入拥塞避免状态
* 当出现超时重传事件，进入慢启动状态，cwnd = 1， threshold = cwnd/2
* 当出现三次重复确认，代表网络并不拥塞，但是发生了丢包，进入快恢复状态




#### 1.2 TCP 的握手? 

1. 客户端（SYN_SENT） ----- Seq = 客户端初始序号，SYN = 1 ----> 服务端（LISTEN）
2. 服务端 (SYN_RCVD) ----- Seq = 服务端初始序号，ACK = 客户端初始序号 + 1，SYN = 1 ----> 客户端 (ESTABLISHED)
3. 客户端 (ESTABLISHED) ----- Seq = 客户端初始序号 + 1，ACK = 服务端初始序号 + 1，传输的数据 ----> 服务端 (ESTABLISHED)




#### 1.3 TCP 的挥手？TIME_WAIT 的意义？

1. 客户端（FIN_WAIT-1） ----- Seq = C，FIN = 1 ----> 服务端（CLOSE_WAIT）

2. 服务端 (LAST_ACK) ----- Seq = S，Ack = C + 1，ACK = 1 ----> 客户端 (FIN_WAIT-2)

3. 服务端 (LAST_ACK) ----- Seq = U，Ack = C + 1，ACK = 1， FIN = 1 ----> 客户端 (TIME_WAIT)

4. 客户端 (TIME_WAIT....CLOSED) ----- Seq = C + 1，Ack = U + 1，ACK = 1 ----> 服务端 (CLOSED)

TIME_WAIT 的意义：

1. 假定最后一个 ACK 丢失，则需要重发
2. 如果立即关闭，在次端口上建立了新的 Socket，上个关闭的连接丢失的FIN到达了，就会错误关闭新的连接



#### 1.4 DDOS 攻击

DDOS：分布式拒绝访问攻击

如：

1. SYN 洪泛攻击，向服务器发送大量建立连接的请求，服务器预先分配内存等资源，使服务器瘫痪。可以不分配资源，而是存一个 SYN_Cookie，等到客户端发来合法的 ACK 时再建立全连接。



### 2. 详解 HTTP <div id="http" />

#### 2.1 URL 的格式详解

HTTP 的 URL 将从因特网获取信息的五个基本元素包括在一个简单的地址中：

1. 传送协议。HTTP / HTTPS
2. 层级URL标记符号(为[//],固定不变)
3. 服务器。（通常为域名，有时为IP地址）
4. 端口号。（以数字方式表示，若为默认值可省略）
5. 路径。（以“/”字符区别路径中的每一个目录名称）
6. 查询。（GET模式的窗体参数，以“?”字符为起点，每个参数以“&”隔开，再以“=”分开参数名称与资料，通常以UTF8的URL编码，避开字符冲突的问题）
9. 片段（哈希）。以“#”字符为起点，可用于定位页面的位置

以https://zh.wikipedia.org:443/w/index.php?title=Special, 其中：

1. https，是协议；
2. zh.wikipedia.org，是服务器；
3. 443，是服务器上的网络端口号；
4. /w/index.php，是路径；
5. ?title=Special，是查询

**‘#’：**

\# 代表网页中的一个位置。其右面的字符，就是该位置的标识符。比如

http://www.example.com/index.html#print



1. 是用来指导浏览器动作的，对服务器端完全无用。所以，HTTP请求中不包括#

2. 改变#不触发网页重载

   单单改变#后的部分，浏览器只会滚动到相应位置，不会重新加载网页。

   比如，从

   http://www.example.com/index.html#location1

   改成

   http://www.example.com/index.html#location2

   浏览器不会重新向服务器请求index.html。

3. 改变#会改变浏览器的访问历史




#### 2.2 请求和响应格式

##### 请求报文格式

    1. 请求行；请求方法，URL，协议版本：GET /index.html HTTP/1.2
    2. 首部行，如：
        * 指明对象所在主机，Host：www.baidu.com 
        * 是否长连接：Connection: close
        * 浏览器类型：USer-agent: Chrome
    3. 空行
    4. 实体行（GET方法，实体行为空）



GET 方法会在URL中添加参数，？param1=val1&param2=val2

##### 响应报文格式

    1. 状态行；协议版本，状态码，状态信息，协议版本：HTTP/1.2 200 OK
        * 200: 响应成功
        * 301: 永久转移
        * 400: 无效请求
        * 404
        * 500
    2. 首部行，如：
        * 日期，Date：Tue 18
        * 服务器信息：Server: Apache/2.2.3
        * 最后修改事件：Last-Modified: Tue 18...
        * 发送对象的字节数：Content-Length
        * 发送对象的类型：text/html
    3. 空行
    4. 实体行



#### 2.4 会话技术：Session 和 Cookie

HTTP 是无状态的，要记住一些用户信息，需要 Session 或 Cookie。



##### 客户端会话技术 —— Cookie

Cookie技术有四个组件：

1. HTTP 响应报文中的 Cookie字段
2. HTTP 请求报文中的 Cookie字段
3. 用户端存储的 Cookie，由浏览器进行管理
4. 位于 Web后端的数据库，存储 Cookie

用户请求后，后端返回一个 cookie，之后的用户请求都带上这个 cookie。
Cookie具有有效期，且切换了浏览器就失去作用。



##### Session

用户使用浏览器访问服务器的时候，服务把用户的信息，以某种形式记录在服务器，这就是 Session。保存这个 SessionID 的方式就可以是Cookie。
由于关闭浏览器不会导致session被删除，迫使服务器为seesion设置了一个失效时间（周期）。Session周期指的是不活动的时间，如果我们设置Session是10s，在10s内，没有访问session，session中属性失效，如果在9s的时候，你访问了session，则会重新计时。

Session可以存储对象，Cookie只能存储字符串，因此可以解决很多Cookie解决不了的问题



### 3. DNS <div id="dns"/>

    DNS 域名系统。为的是通过域名查询对应主机的IP 地址。运行于 UDP 协议，53端口。

域名解析的过程：

1. 请求主机向 Local DNS Server 发送请求（递归）
2. Local DNS 查询缓存，缓存未命中，则由 Local DNS 迭代地逐级从根域名服务器往下进行查询
3. Local DNS 获取到查询结果，返回给请求主机

    如何配置 Local DNS，一般是 DHCP，也可以手动配置。当主机连入网络，网络服务提供商会提供一个主机的IP，这个主机具有一台或多台其本地 DNS 服务器的 IP 地址。

    DHCP：用一台或一组DHCP服务器来管理网络参数的分配，这种方案具有容错性。即使在一个仅拥有少量机器的网络中，DHCP仍然是有用的，因为一台机器可以几乎不造成任何影响地被增加到本地网络中。

可实现：

1. 主机别名（多个域名）
2. 邮件服务器别名
3. 负载均衡



### 4. HTTPS 的过程简介？证书如何确保安全？<div id="https"/>

1. 对称加密，双方使用同一个密钥进行加密解密，第三方窃取到密钥就可以获取通信内容。
2. 非对称加密，采用公钥和私钥两个密钥进行加密，用一个密钥加密的只能使用另一个密钥解密，但是运算速度远远慢于对称加密。

    HTTPS的大致流程，是先从服务器获取公钥，本地生成对称加密密钥，经公钥加密传递给服务器，服务器获取到对称密钥，此后使用对称加密进行通信。即，协商对称加密的过程。
    
    然而，这样容易遭到中间人攻击：
    
1. A ---- 公钥 pubA ---- 中间人 C，拦截下 pubA：A 的公钥是 pubC ----> B，以为 A的密钥就是pubC
2. B ---- 用 pubC 加密数据 ---- 中间人 C，拦截下数据，用 privC解密，再用 pubA 加密传递给 A ----> A

    因此，为了确保公钥未经篡改，使用证书进行认证。

    经过权威的认证中心（CA），提供一个证书，证书里面记录了网站的公钥和其他相关信息，这就是数字证书。为了确保这是有效的真实的证书，我们又需要在证书中加入 CA 的签名，即数字签名。
    
    相应的，判断数字证书是否正确的过程如下，首先通过 CA 的公钥解密证书中的数字签名得到证书摘要1，同时用相同的 Hash 算法计算证书中的公钥和其他信息得出证书摘要2，比较证书1和2，即可判断证书是否正确。
    
    但是 CA 的公钥也会遭到篡改，怎么办？

    1. 我们必须相信CA（有失信的 CA 被浏览器拉黑）
    2. CA本身也有证书证明自己身份
    3. CA是分级结构，顶级 CA 证书是自签名的，通常会内置于操作系统和浏览器



### 5. WebSocket <div id="websocket"/>

    WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。
    
    由于HTTP 协议是一种无状态的、无连接的、单向的应用层协议。它采用了请求/响应模型。通信请求只能由客户端发起，服务端对请求做出应答处理。这种通信模型有一个弊端：HTTP 协议无法实现服务器主动向客户端发起消息。这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。
    
    WebSocket 就是这样发明的。WebSocket 连接允许客户端和服务器之间进行全双工通信，以便任一方都可以通过建立的连接将数据推送到另一端。WebSocket 只需要建立一次连接，就可以一直保持连接状态。这相比于轮询方式的不停建立连接显然效率要大大提高。
    
    特点：
    
    1. 建立在TCP协议之上，服务器端容易实现。
    2. 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
    3. 性能开销小，通信高效。
    4. 可发送文本或二进制数据。
    5. 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。



#### 握手

1. 客户端握手请求

HTTP请求必须是1.1或更高，并且该方法必须是GET：

```python
GET /chat HTTP/1.1
Host: http://example.com:8000
Upgrade: websocket # 告诉服务器,升级到websocket协议
Connection: Upgrade # 协议升级
Sec-WebSocket-Protocol: chat, superchat # 用户自定义的字符串 在同一个url下 不同服务的所需要的协议 比如聊天 chat 也可以其他的自定义
Sec-WebSocket-Version: 13 # 服务器所使用的协议版本
Sec-WebSocket-key: XXXX # base64加密的字符串 浏览器自动生成
```

2. 服务端接受并响应握手请求
```python
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo= # 对Sec-WebSocket-key 的加密 同意握手建立链接, 客户端收到 Sec-WebSocket-Accept后, 将本地的Sec-WebSocket-key 编码做一个对比来验证
```

Sec-WebSocket-key 主要是为了确保服务端避免接受非 websocket 的连接



### 6. 从输入URL到完成响应的一系列过程 <div id="after-url"/>

1. 输入 URL，构建 HTTP 协议报文（这样的话，一般是GET 请求）
2. 将 HTTP 报文传给 TCP，设定端口号（浏览器这边可以是操作系统指定，服务器那边一般是 80（HTTP）或 443（HTTPS））
3. 将 TCP 报文传给 IP，需要服务器的 IP 地址，因此需要进行域名解析 Hosts -> Local DNS -> ...
4. 域名解析完成，发送 HTTP 报文，HTTP 请求到达 服务器
5. 服务器处理请求，返回结果

请求到达服务器以后，以 Nginx 为例：
    
   Nginx 以 Daemon（守护进程）形式运行在后台。（Daemon（守护进程）是运行在后台的一种特殊进程。它独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。常见的守护进程包括系统日志进程syslogd、 web服务器httpd、邮件服务器sendmail和数据库服务器mysqld等。）

   Nginx 采用多进程 + 异步非阻塞IO事件模型来处理各种连接请求。多进程模型包括一个 Master 进程，多个 Worker 进程，一般 Worker 进程个数是根据服务器 CPU 核数来决定的。master 进程负责管理 Nginx 本身和其他 worker 进程。

   首先，master进程一开始就会根据我们的配置，来建立需要 Listen 的网络 socket_fd，然后 **fork** 出多个 Worker 进程。其次，


**从 Nginx 的内部来看，一个 HTTP Request 的处理过程涉及到以下几个阶段**：

1. 初始化 HTTP Request（读取来自客户端的数据，生成 HTTP Request 对象，该对象含有该请求所有的信息）。
2. 处理请求头。
3. 处理请求体。
4. 如果有的话，调用与此请求（URL 或者 Location）关联的 handler。
    1. 如果自身可以处理，如静态文件，则进行处理
    2. 如果需要代理转发，则 创建新的 HTTP 请求到转发的服务器，将获取到的结果返回
    



### 7. Nginx 如何处理请求 <div id="nginx"/>

#### 7. 1 Nginx 如何实现高性能低消耗的呢？

   网络事件处理机制Nginx 

   1. 采用异步非阻塞的方式处理请求，可以同时处理上万的请求Nginx 
   2. 支持 select/epoll 等流行事件处理机制，根据系统环境自动选择Nginx 
   3. 采用独立于系统的事件处理机制，能够高效处理请求

   资源分配技术Nginx

   1. 采用分阶段资源分配技术，使得它的CPU和内存消耗非常低

   多核处理优化

   1. Nginx 默认采用多进程启动模式
   2. Nginx 包含Master 进程 和 Worker 进程
   3. 能够充分利用 SMP 对称多处理的优势，减少Worker进程磁盘I/O的阻塞
   4. Nginx 支持Worker进程和CPU内核 一一对应绑定，避免进程上下文的切换致使cache失效



#### 7. 2 进程模型

Linux 系统中，Nginx默认以守护进程daemon方式启动，默认采用多进程方式。Nginx包括两种类型的进程：- Master 进程，数量只有一个，管理 Nginx本身和 Worker 进程- Worker 进程，数量一般和 CPU 核数相等，Nginx 的所有请求处理，均是在 Worker 进程中完成。



##### Master 进程工作机制

   在Nginx启动时，Master进程创建，主要负责初始化 Nginx 和相关模块、fork Worker 进程、接收处理外界信号等工作。

   Nginx的初始化过程：解析配置文件，这是 Nginx 初始化最重要的一个环节调用各个配置指令回调函数，完成各个模块的配置，建立需要 Listen 的网络 socket_fd，最后 fork 创建 Worker 子进程和 Cache 子进程。

   这样，也可以实现**热部署**，配置文件修改后，由 Master 通知各 Woker 进程。



##### Worker 进程工作机制

   Worker进程负责所有请求的处理工作。

   1. 根据进程的特性，新建立的 Worker 进程，会和 Master 进程一样，具有相同的设置。因此，其也会去监听相同 IP 端口的套接字 socket_fd。
   
   2. **有多个 Worker 进程都在监听同样设置的 socket_fd，意味着当有一个请求进来的时候，所有的 worker 都会感知到。这样就会产生所谓的“惊群现象”。 **
   
   3. 为了保证只会有一个进程成功注册到 listen fd 的读事件，**Nginx 中实现了一个 “accept_mutex” 类似互斥锁，只有获取到这个锁的进程，才可以去注册读事件。其他进程全部 Accept 失败.**
   
   4. 最后，监听成功的 Worker 进程，读取请求，解析处理，响应数据返回给客户端，断开连接，结束。因此，一个HTTP请求，完全由Worker进程处理，而且只在一个Worker中处理。

   进程模型的处理方式带来的一些好处就是：

   1. 进程之间是独立的，也就是一个 Worker 进程出现异常退出，其他 worker 进程是不会受到影响的。
   2. 独立进程也会避免一些不需要的锁操作，这样子会提高处理效率，并且开发调试也更容易。
   3. **避免的多线程的上下文切换导致的性能损失**。

   Worker 进程会竞争监听客户端的连接请求：这种方式可能会带来一个问题，就是可能所有的请求都被一个 Worker 进程给竞争获取了，导致其他进程都比较空闲，而某一个进程会处于忙碌的状态，这种状态可能还会导致无法及时响应连接而丢弃 Discard 掉本有能力处理的请求。这种不公平的现象，是需要避免的，尤其是在高可靠web服务器环境下。针对这种现象，Nginx采用了一个是否打开accept_mutex选项的值，ngx_accept_disabled 标识控制一个 Worker进程是否需要去竞争获取 accept_mutex 选项，进而获取 Accept 事件.



##### 网络事件处理机制

   Nginx 采用的是异步非阻塞事件处理机制。

   首先，请求过来，要建立连接，然后再接收数据，接收数据后，再发送数据。

   具体到系统底层，就是读写事件，而当读写事件没有准备好时，必然不可操作，如果不用非阻塞的方式来调用，那就得阻塞调用了，事件没有准备好，那就只能停止等待，线程被挂起。

   非阻塞就是，事件没有准备好，马上返回 EAGAIN，告诉我们，事件还没准备好。但是如果持续的来询问事件是否准备好，也会是一笔不小的开销。

   所以，才会有了异步非阻塞的事件处理机制，具体到系统调用就是像 select/poll/epoll/kqueue 这样的系统调用。它们提供了一种机制，让我们可以同时监控多个事件，调用他们是阻塞的，但可以设置超时时间，在超时时间之内，如果有事件准备好了，就返回。这种机制正好解决了我们上面的两个问题，拿epoll为例(在后面的例子中，我们多以 epoll 为例子，以代表这一类函数)，当事件没准备好时，放到 epoll 里面，事件准备好了，我们就去读写，当读写返回 EAGAIN 时，我们将它再次加入到 epoll 里面。

   这样，只要有事件准备好了，我们就去处理它，只有当所有事件都没准备好时，才在 epoll 里面等着。这样，我们就可以并发处理大量的并发请求了。

   当然，这里的并发请求，是指未处理完的请求，线程只有一个，所以同时能处理的请求当然只有一个了，只是在请求间进行不断地切换而已，切换也是因为异步事件未准备好，而主动让出的。

   这里的切换是没有任何代价，你可以理解为循环处理多个准备好的事件，事实上就是这样的。与多线程相比，这种事件处理方式是有很大的优势的，不需要创建线程，每个请求占用的内存也很少，没有上下文切换，事件处理非常的轻量级。并发数再多也不会导致无谓的资源浪费（上下文切换）。更多的并发数，只是会占用更多的内存而已。 我之前有对连接数进行过测试，在24G内存的机器上，处理的并发请求数达到过200万。现在的网络服务器基本都采用这种方式，这也是 Nginx 性能高效的主要原因。

   我们之前说过，推荐设置 Worker 的个数为 CPU 的核数，在这里就很容易理解了，更多的 Worker 数，只会导致进程来竞争 CPU 资源了，从而带来不必要的上下文切换。而且，Nginx 为了更好的利用多核特性，提供了 CPU 亲缘性的绑定选项，我们可以将某一个进程绑定在某一个核上，这样就不会因为进程的切换带来 Cache 的失效。像这种小的优化在 Nginx 中非常常见，同时也说明了 Nginx 作者的苦心孤诣。比如，Nginx 在做4个字节的字符串比较时，会将 4个字符转换成一个 int 型，再作比较，以减少 CPU 的指令数等等。



##### 负载均衡

1. 轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
```python
upstream backserver { 
    server 192.168.0.14; 
    server 192.168.0.15; 
} 
```

2. 指定权重

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
```python
upstream backserver { 
    server 192.168.0.14 weight=8; 
    server 192.168.0.15 weight=10; 
} 
```



##### 缓存机制

例如，可以将响应静态文件缓存到一起，下一次相同的请求就可以直接返回数据。



### 8. TODO

* 浏览器请求一个域名，经历了什么。 

* DNS是基于什么协议，有没有基于HTTP的DNS协议。

* 什么是网关，网关的作用，限流的算法：固定/滑动窗口，消息队列，令牌桶。消息队列怎么限流，消费方如何返回给消息生产方。【同步的话就阻塞，等待返回；异步的话，可以从另一个topic里面取？（不确定）】 

* HTTP的方法，GET和POST的区别，HTTP状态码。502和504有什么区别。 

* HTTP如何保存用户状态。cookie 

* HTTPS 对称加密和非对称加密 

* 为什么TCP建立连接的时候少一次握手？ 

* 有没有用过HTTP3.0

* HTTP的指令有哪些POST 和 PUT的区别 幂等性没有答出来

* HTTP 2.0 具体怎么实现 为什么要二进制分帧 有哪些好处 共享连接能同时发送多个文件吗 

* HTTP 3.0是什么

* STL框架了解吗

* HTTP的keep-alive机制（传输数据什么的，当时太紧张忘了面试官说了什么了） 

* TCP的粘包拆包有了解过吗 

* cookie和seesion的区别（sessionID的存放点）

* tcp报文结构 

* HTTP状态码，面试官提问了几个 200 404 500 

* 滑动窗口

    - 如果一直没收到确认报文会怎样
    - 如果收到重复报文怎么办
    