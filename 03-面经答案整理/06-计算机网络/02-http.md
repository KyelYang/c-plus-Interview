# 面试问题
- 网页解析的过程与实现方法
- Web请求过程
- 在浏览器中输入URL后执行的全部过程（如www.baidu.com）
- 输入网址后的执行过程，详细讲解每一层
- 输入www.baidu.com,会发生什么事情。在这个过程中，会请求什么东西
- http/1.0和http/1.1的区别
- http的请求方法有哪些？get和post的区别。
- http的状态码
- 讲讲http的状态码，403,201什么意思
- http有状态吗？什么情况下用http连接不用tcp
- HTTP如何实现状态化，cookie被禁用了怎么办
- http和https的区别，由http升级为https需要做哪些操作
- https的具体实现，怎么确保安全性
- http中浏览器一个URL的流程，这个过程中浏览器做了什么，URL包括哪三个部分？
- HTTP和HTTPS的区别（一般都会问感觉）
- HTTP了解吗，HTTP1，HTTP1.1，HTTP2有啥区别，HTTP header都有啥内容啊，HTTP方法有哪些啊，GET/POST区别是啥，长链接的发送顺序和接受顺序要一致嘛，可能被阻塞嘛？
- HTTPS了解吗，握手流程是怎么样的，加密是对称还是非对称，协议相比HTTP有什么好处呢？HTTP和HTTPS的端口分别是啥？
- 如果用户输入的是http://xxx.com的网址，但是已经升级到了HTTPS，会发生什么呢？你会怎么处理呢？
- 你刚刚提到mmap，怎么实现的了解吗？是只能映射用户空间和内核空间吗？
- 详细讲解HTTPS的原理，客户端为什么信任第三方证书
- HTTP长连接和短连接

## 中间人攻击
- [网络攻击介绍](https://cshihong.github.io/2019/05/13/%E7%BD%91%E7%BB%9C%E6%94%BB%E5%87%BB%E4%BB%8B%E7%BB%8D/)
- [什么是 TLS 中间人攻击？如何防范这类攻击？](https://www.zhihu.com/question/20744215)

## 单向验证与双向验证
- [HTTPS实战之单向验证和双向验证](https://mp.weixin.qq.com/s/UiGEzXoCn3F66NRz_T9crA)

## 数字证书的验证原理
- [数字证书的原理是什么？](https://www.zhihu.com/question/24294477)
- [数字证书应用综合揭秘（包括证书生成、加密、解密、签名、验签）](https://www.cnblogs.com/leslies2/p/7442956.html)

## http保障账号密码安全性的认证方法
- [HTTP协议下保证密码不被获取更健壮方式](https://www.cnblogs.com/intsmaze/p/6009648.html)
- [Web认证那些事儿（一）从用户体验感受认证流程](https://liaodanqi.me/2018/10/16/web-authentication-intro/)
- [Web认证那些事儿（二）用户名密码认证](https://liaodanqi.me/2018/10/18/username-password-authentication/)
- [Web认证那些事儿（三）Token-based认证](https://liaodanqi.me/2018/10/23/token-authentication/)

## MD5算法算不算加密算法
- [MD5算法算不算加密算法，加了盐的MD5算不算加密算法？](https://www.zhihu.com/question/68735830)
- [MD5、对称加密、非对称加密的比较区别](https://blog.csdn.net/u012438830/article/details/89045609)

## http和https概述
- [十分钟搞懂HTTP和HTTPS协议？](https://zhuanlan.zhihu.com/p/72616216)
## http和https的区别，由http升级为https需要做哪些操作
### http和https的区别
#### HTTPS是HTTP协议的安全版本，HTTP协议的数据传输是明文的，是不安全的，HTTPS使用了SSL/TLS协议进行了加密处理
http是HTTP协议运行在TCP之上。所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份。  

https是HTTP运行在SSL/TLS之上，SSL/TLS运行在TCP之上。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。此外客户端可以验证服务器端的身份，如果配置了客户端验证，服务器方也可以验证客户端的身份。

#### http和https使用连接方式不同
https建立连接的过程需要多次握手，导致页面的加载时间延长近50%。  

SSL涉及到的安全算法会消耗 CPU 资源，对服务器资源消耗较大。  

#### 默认端口不一样，http是80，https是443


## https是对称加密还是非对称加密
HTTPS 在内容传输的加密上使用的是对称加密，非对称加密只作用在证书验证阶段。  

在证书验证阶段，客户端和服务端会相互传输随机码，该随机码用于生成会话密钥，该会话密钥就是本次临时通讯的对称加密密匙。  

- [Https的故事](https://www.hrwhisper.me/introduction-to-https/)

## 对称加密与非对称加密的区别
### 对称加密
在对称加密算法中，加密和解密使用的是同一把钥匙，即：使用相同的密匙对同一密码进行加密和解密。  
### 非对称加密
非对称加密自然是使用不同的密钥进行加密和解密啦。  

非对称加密有两个钥匙，及公钥（Public Key）和私钥（Private Key）。公钥和私钥是成对的存在，如果对原文使用公钥加密，则只能使用对应的私钥才能解密；因为加密和解密使用的不是同一把密钥，所以这种算法称之为非对称加密算法。  

- [浅谈对称加密与非对称加密](https://zhuanlan.zhihu.com/p/49494990)


- [HTTPS用的是对称加密还是非对称加密？](https://zhuanlan.zhihu.com/p/96494976)

- [数字签名是什么？](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)

- [RSA的公钥和私钥到底哪个才是用来加密和哪个用来解密？](https://www.zhihu.com/question/25912483)

## 12306刚上线的时候，有证书，为什么还提示证书不可信呢？
- [为什么在12306买火车票要装根证书？](https://www.williamlong.info/archives/3461.html)
- [12306购票网站终于启用可信的HTTPS数字证书](https://cloud.tencent.com/developer/news/26635)

## 在浏览器中输入URL后执行的全部过程（如www.baidu.com）
- [github超详细过程](https://github.com/skyline75489/what-happens-when-zh_CN/blob/master/README.rst#%E5%9B%9E%E8%BD%A6%E9%94%AE%E6%8C%89%E4%B8%8B)
- [在浏览器输入 URL 回车之后发生了什么（超详细版）](https://zhuanlan.zhihu.com/p/80551769)
- [在浏览器地址栏输入一个URL后回车，背后会进行哪些技术步骤？](https://www.zhihu.com/question/34873227/answer/518086565)
- [浏览器输入 URL 后发生了什么？](https://zhuanlan.zhihu.com/p/43369093)

## HTTP1，HTTP1.1，HTTP2有啥区别
- [HTTP 2.0 和 HTTP 1.1 相比有哪些优势呢？](https://www.zhihu.com/question/306768582/answer/595200654)
- [HTTP/2 相比 1.0 有哪些重大改进？](https://www.zhihu.com/question/34074946/answer/108588042)
- [HTTP1.1和HTTP2.0的区别，HTTP2.0核心优势有哪些](https://zhuanlan.zhihu.com/p/53603744)

## http的请求方法有哪些？get和post的区别
### get:获取资源
get方法用来请求访问已被URI识别的资源。指定的资源经服务器端解析后返回响应内容。也就是说，如果请求的资源是文本，那就保持原样返回；如果是像 CGI（Common Gateway Interface，通用网关接口）那样的程序，则返回经过执行后的输出结果。  

### post:传输实体主体
post方法用来传输实体的主体。  

虽然用get方法也可以传输实体的主体，但一般不用get方法进行传输，而是用post方法。虽说post的功能与get很相似，但post的主要目的并不是获取响应的主体内容。  
### 知乎答案
#### GET用于获取指定的资源（幂等）
“读取“一个资源。  

比如Get到一个html文件。反复读取不应该对访问的数据有副作用。比如”GET一下，用户就下单了，返回订单已受理“，这是不可接受的。没有副作用被称为“幂等“（Idempotent)。  

因为GET因为是读取，就可以对GET请求的数据做缓存。这个缓存可以做到浏览器本身上（彻底避免浏览器发请求），也可以做到代理上（如nginx），或者做到server端（用Etag，至少可以减少带宽消耗）

#### POST用于创建/更新某个资源 （非幂等）
在页面里\<form> 标签会定义一个表单。  
  
 点击其中的submit元素会发出一个POST请求让服务器做一件事。这件事往往是有副作用的，不幂等的。不幂等也就意味着不能随意多次执行。因此也就不能缓存。  
 
 比如通过POST下一个单，服务器创建了新的订单，然后返回订单成功的界面。这个页面不能被缓存。试想一下，如果POST请求被浏览器缓存了，那么下单请求就可以不向服务器发请求，而直接返回本地缓存的“下单成功界面”，却又没有真的在服务器下单。那是一件多么滑稽的事情。  
 
 因为POST可能有副作用，所以浏览器实现为不能把POST请求保存为书签。想想，如果点一下书签就下一个单，是不是很恐怖？。此外如果尝试重新执行POST请求，浏览器也会弹一个框提示下这个刷新可能会有副作用，询问要不要继续。

- [GET 和 POST 到底有什么区别？](https://www.zhihu.com/question/28586791)
- [http中的get和post的区别是什么呢?](https://www.zhihu.com/question/326687967)
- [都2019年了，还问GET和POST的区别](https://zhuanlan.zhihu.com/p/57361216)

### HTTP/1.0 和 HTTP/1.1 支持的方法

| 方法 | 说明 | 支持的HTTP协议版本 |
| --- | --- | --- |
| GET  | 获取资源 | 1.0、1.1 |  
| POST | 传输实体主体 | 1.0、1.1 |  
| PUT  | 传输文件 | 1.0、1.1 |  
| HEAD | 获得报文首部 | 1.0、1.1 |  
| DELETE | 删除文件 | 1.0、1.1 |  
| OPTIONS | 询问支持的方法 | 1.1 |  
| TRACE | 追踪路径 | 1.1 |  
| CONNECT | 要求用隧道协议连接代理 | 1.1 |  
| LINK | 建立和资源之间的联系 | 1.0 |  
| UNLINK | 断开连接关系 | 1.1 |  

在这里列举的众多方法中，LINK 和 UNLINK 已被 HTTP/1.1 废弃，不再支持。  

## Cookie和Session的区别
- [90%程序员面试时都没有完全答对Cookie和Session的区别](https://zhuanlan.zhihu.com/p/66754258)
- [彻底理解cookie，session，token](https://zhuanlan.zhihu.com/p/63061864)
- [COOKIE和SESSION有什么区别？](https://www.zhihu.com/question/19786827)

## 讲讲http的状态码，403,201什么意思
HTTP 状态码负责表示客户端 HTTP 请求的返回结果、标记服务器端的处理是否正常、通知出现的错误等工作。  

### 201 Created
201 Created表示请求已经成功，且请求创建了新的服务器资源。新资源的地址或是请求的URL，或是Location响应头的值。  

常用作POST方法的状态码。  

### 403 Forbidden
该状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述，这样就能让用户看到了。  

未获得文件系统的访问授权，访问权限出现某些问题（从未授权的发送源 IP 地址试图访问）等列举的情况都可能是发生 403 的原因。  

### http的状态码
- [HTTP 响应代码](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)
- [HTTP状态码](https://tool.lu/httpcode/)

## HTTP 缓存之no-cache和no-store的区别
no-cache 和 no-store 都是 HTTP 协议头 Cache-Control 的值。区别是：  
### no-store
彻底禁用缓冲，所有内容都不会被缓存到缓存或临时文件中。  
### no-cache
在浏览器使用缓存前，会往返对比 ETag，如果 ETag 没变，返回 304，则使用缓存。  

除了 no-cache 和 no-store，Cache-Control 头的取值还有：  
### public

所有内容都将被缓存（客户端和代理服务器都可缓存）  

### private

内容只缓存到私有缓存中（仅客户端可以缓存，代理服务器不可缓存）

### max-age=xxx

缓存的内容将在 xxx 秒后失效，这个选项只在 HTTP1.1 可用，并如果和 Last-Modified 一起使用时，优先级较高。  

## http和tcp的关系
HTTP是应用层协议。TCP是传输层协议。HTTP是建立在TCP协议之上。  
- [TCP/IP、Http、Socket的区别?](https://www.zhihu.com/question/39541968)
- [TCP/IP 和 HTTP 的区别和联系是什么？](https://www.zhihu.com/question/38648948)


## 常见的数字签名算法
- [数字签名算法介绍和区别](https://zhuanlan.zhihu.com/p/33195438)
- [签名算法](https://www.liaoxuefeng.com/wiki/1252599548343744/1304227943022626)

## Cookie的secure和httpOnly属性的含义
- [Cookie的secure和httpOnly属性的含义](https://blog.csdn.net/wang252949/article/details/79557963)
