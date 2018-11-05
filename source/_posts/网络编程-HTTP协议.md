---
title: 网络编程 - HTTP协议
date: 2018-11-05 14:54:08
tags: 网络
---
HTTP协议，全称超文本传输协议(HyperText Transfer Protocol)，是目前互联网上应用最为广泛的一种网络协议，位于应用层。
##  HTTP基础
### HTTP协议用于客户端和服务端之间的通信
两台计算机之间使用HTTP协议通信时，必有一端是客户端，另外一端是服务器端。其中请求访问资源的一端为客户端，提供资源响应的一端称为服务器端。有时候两台计算机的角色可能会互换，但是仅从一条通信路线来说，客户端和服务器端的角色是确定的。
### 通过请求和响应的交换达成通信
HTTP协议规定，请求从客户端发出，最后服务器端响应该请求并返回。
### HTTP是不保存状态的协议
HTTP是一种无状态协议。协议自身不对请求和响应之间的通信状态进行保存。也就是说在 HTTP 这个级别，协议对于发送过的请求或响应都不做持久化处理。这是为了更快地处理大量事务，确保协议的可伸缩性，而特意把 HTTP 协议设计成如此简单的。可是随着 Web 的不断发展，我们的很多业务都需要对通信状态进行保存。于是我们引入了 Cookie 技术。有了 Cookie 再用 HTTP 协议通信，就可以管理状态了。
### 请求URI定位资源
HTTP协议使用 URI 定位互联网上的资源。正是因为 URI 的特定功能，在互联网上任意位置的资源都能访问到。
### 告知服务器意图的 HTTP 方法
|方法名|说明|描述|支持版本|
--|--|--|--|
|GET|获取资源|GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回响应内容|1.0 1.1|
|POST|传输实体主体|POST 方法用来传输实体的主体。虽然 GET 也可以传输实体的主体，但一般不用 GET 而用 POST，POST 的主要目的并不是获取响应的主体内容|1.0 1.1|
|PUT|传输文件|PUT方法用来传输文件，要求再请求报文的主体中包含文件内容，然后保存到请求URI指定的位置|1.0 1.1|
| HEAD |获取报文首部|HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确认 URI 的有效性及资源更新的日期时间等等|1.0 1.1|
| DELETE |删除文件|与 PUT 相反的方法，DELETE 方法按请求 URI 删除指定资源|1.0 1.1|
| OPTIONS |询问支持的方法|OPTIONS 用来查询针对请求 URI 指定的资源支持的方法|1.1|
| TRACE |追踪路径|TRACE 方法是让 Web 服务器将之前的请求通信返回给客户端的方法，TRACE 方法不常用，并且容易引发 XST ( Cross-Site-Tracing ，跨站追踪)攻击，所以通常更不会用到了|1.1|
| CONNECT |要求用隧道协议连接代理|CONNECT 方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信，主要使用 SSL （ Secure Sockets Layers ，安全套接层）和 TLS （ Transport Layer Security ，传输层安全）协议把通信内容加密后经网络隧道传输|1.1|
| PATCH |更新部分文件内容|当资源存在的时候，PATCH 用于资源的部分内容的更新，例如更新某一个字段。具体比如说只更新用户信息的电话号码字段，而 PUT 用于更新某个资源较完整的内容，比如说用户要重填完整表单更新所有信息，后台处理更新时可能只是保留内部记录 ID 不变。  当资源不存在的时候，PATCH 是修改原来的内容，也可能会产生一个新的版本。比如当资源不存在的时候，PATCH 可能会去创建一个新的资源，这个意义上像是 saveOrUpdate 操作。而 PUT 只对已有资源进行更新操作，所以是 update 操作|1.1|
### 持久链接节省通信量
HTTP协议的初始版本中，每进行一个 HTTP 通信都要断开一次 TCP 连接。比如使用浏览器浏览一个包含多张图片的 HTML 页面时，在发送请求访问 HTML 页面资源的同时，也会请求该 HTML 页面里包含的其他资源。因此，每次的请求都会造成无谓的 TCP 连接建立和断开，增加通信量的开销。
#### 持久链接
为了解决上述 TCP 连接的问题，HTTP/1.1 和部分 HTTP/1.0 想出了持久连接的方法。其特点是，只要任意一端没有明确提出断开连接，则保持 TCP 连接状态。旨在建立一次 TCP 连接后进行多次请求和响应的交互。在 HTTP/1.1 中，所有的连接默认都是持久连接。它的好处在于减少了TCP连接的重复建立和断开所造成的额外开销，减轻了服务器端的负载。
#### 管线化(pipelining)
持久连接使得多数请求以管线化方式发送成为可能。以前发送请求后需等待并接收到响应，才能发送下一个请求。管线化技术出现后，不用等待亦可发送下一个请求。这样就能做到同时并行发送多个请求，而不需要一个接一个地等待响应了。比如，当请求一个包含多张图片的 HTML 页面时，与挨个连接相比，用持久连接可以让请求更快结束。而管线化技术要比持久连接速度更快。请求数越多，时间差就越明显。
### 使用Cookie的状态管理
Cookie 技术通过在请求和响应报文中写入 Cookie 信息来控制客户端的状态。Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie 的首部字段信息，通知客户端保存Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入 Cookie 值后发送出去。服务器端发现客户端发送过来的 Cookie 后，会去检查究竟是从哪一个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前的状态信息。

## HTTP工作流程
###  TCP/IP通信传输流
HTTP协议是站在在TCP/IP协议肩膀上的，从HTTP往下看，是TCP协议保证了运输的可靠性，是IP协议保证了数据可以达到目标地址，是以太网协议在局域网内传递信息。所以说HTTP工作流程，先谈TCP/IP通信传输流。
![TCP:IP通信传输流.png](https://upload-images.jianshu.io/upload_images/2855070-57fc5e699c7be51f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
利用 TCP/IP 协议族进行网络通信时，会通过分层顺序与对方进行通信。发送端从应用层往下走，接收端则从链路层往上走。
- 首先作为发送端的客户端在应用层（HTTP 协议）发出一个想看某个 Web 页面的 HTTP 请求。
- 为了传输方便，在传输层（TCP 协议）把从应用层处收到的数据（HTTP 请求报文）进行分割，并在各个报文上打上标记序号及端口号后转发给网络层。
- 在网络层（IP 协议），增加作为通信目的地的 MAC 地址后转发给链路层。这样一来，发往网络的通信请求就准备齐全了。
- 接收端的服务器在链路层接收到数据，按序往上层发送，一直到应用层。当传输到应用层，才能算真正接收到由客户端发送过来的 HTTP请求。
### HTTP请求流程
![HTTP请求流程.png](https://upload-images.jianshu.io/upload_images/2855070-f5023e4b1746cd1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 发送端在层与层之间传输数据时，每经过一层时必会被打上该层所属的头部信息。
- 接收端在层与层之间传输数据时，每经过一层时会把对应的头部消去。

具体介绍如下：
1. 地址解析
比如我们用百度搜索**swift**
```
http://www.baidu.com/baidu?wd=swift
```
协议名：http。这里指要发出的是什么协议。
主机名：www.baidu.com。通过DNS解析，我们可以把主机名解析成服务器的IP地址。
请求文件名：baidu。当我们访问到服务器后，就可以通过文件名请求指定的文件。
请求参数：wd=swift。即使同一个网页，可能针对不同的用户，服务器要返回给客户端的信息也是不一样的 。而服务器就是通过URL中“?”后面携带的参数不同来响应不同的用户或者同一个用户的不同请求的。
2. 封装HTTP 请求
这一步把上面写的 URL 以及本机的一些信息封装成一个 HTTP 请求数据包
3.  封装 TCP 包
第三步就是封装 TCP 包 , 建立 TCP 连接 , 也就是常说的"三次握手"。
4. 客户端发送请求命令
在连接建立之后 , 客户端发送 HTTP 请求到服务端与请求相关的信息都会包含在请求头和请求体中发送给服务器端 。
5. 服务器端响应
服务器端在收到请求之后 , 根据客户端的请求发送给客户端相应的信息 , 相关的响应信息都会放在响应头和响应体中。
6. 关闭连接
服务器端在发送完响应之后 , 就会关闭连接 , 如果过客户端的请求的头部信息中有 Connection-alive , 那么客户端在响应完这个请求之后不会关闭连接 , 知道客户端的所有请求都响应完毕 , 才会关闭连接 , 这样大大节省了带宽和 IO 资源。

## HTTP协议报文结构
### 报文
用于 HTTP 协议交互的信息被称为 HTTP 报文。请求端（客户端）的 HTTP 报文叫做请求报文；响应端（服务器端）的叫做响应报文。HTTP 报文本身是由多行（用 CR+LF 作换行符）数据构成的字符串文本。
### 报文结构
HTTP 报文大致可分为报文首部和报文主体两部分。两者由最初出现的空行（CR+LF）来划分。通常，并不一定有报文主体。
报文结构如下：

![HTTP报文结构.png](https://upload-images.jianshu.io/upload_images/2855070-8e774f70956a66cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 请求报文及响应报文的结构
![请求报文及响应报文的结构.png](https://upload-images.jianshu.io/upload_images/2855070-b6b3c129a6e8e93a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面是请求报文，下面是响应报文。
![请求报文和响应报文实例.png](https://upload-images.jianshu.io/upload_images/2855070-91c78683682246a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面是请求报文实例，下面是响应报文实例。
请求报文和响应报文的首部内容由以下数据组成。
- 请求行：包含用户请求的方法，请求URI和HTTP版本。
- 状态行：包含表明响应结果的状态码，原因短语和HTTP版本。
- 首部字段：包含表示请求和响应的各种条件和属性的各类首部。一般有四种首部，分别是：通用首部、请求首部、响应首部和实体首部。
- 其他：可能包含HTTP的RFC里未定义的首部(Cookie等)。

举个例子：
我们用**Chrome**浏览器打开百度，然后对当前网页进行检查，右键选择**检查**。然后刷新当前页面，选择`Netword`选项，就可以看到当前页面网络活动情况。查看`www.baidu.com`:
![例子.png](https://upload-images.jianshu.io/upload_images/2855070-2079c62c209e84ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们来看一下，这里有什么：
```
General：
Request URL: https://www.baidu.com/
Request Method: GET
Status Code: 200 OK
Remote Address: 180.97.33.108:443
Referrer Policy: no-referrer-when-downgrade
````
Response Headers：
```
HTTP/1.1 200 OK
Bdpagetype: 1
Bdqid: 0xa9952ef000020d8a
Cache-Control: private
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html
Cxy_all: baidu+9cadcdf18d354de5e0816c739f51f361
Date: Tue, 30 Oct 2018 08:50:53 GMT
Expires: Tue, 30 Oct 2018 08:50:27 GMT
Server: BWS/1.1
Set-Cookie: delPer=0; path=/; domain=.baidu.com
Set-Cookie: BDSVRTM=0; path=/
Set-Cookie: BD_HOME=0; path=/
Set-Cookie: H_PS_PSSID=26524_1420_21093_27400_26350; path=/; domain=.baidu.com
Strict-Transport-Security: max-age=172800
Vary: Accept-Encoding
X-Ua-Compatible: IE=Edge,chrome=1
Transfer-Encoding: chunked
```
Request Headers:
```
GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,zh-TW;q=0.8
Cookie: BAIDUID=0A51A4710C9C0407204028C7D18379A0:FG=1; BIDUPSID=0A51A4710C9C0407204028C7D18379A0; PSTM=1529891328; BD_UPN=123253; MCITY=-315%3A; ispeed_lsm=3; delPer=0; BD_HOME=0; H_PS_PSSID=26524_1420_21093_27400_26350; BD_CK_SAM=1; PSINO=3
```
## HTTP 报文首部字段具体分析
HTTP首部字段是构成HTTP报文的要素之一。在客户端与服务端之间以HTTP协议进行通信的过程中，无论是请求还是响应都会使用首部字段，它能起到传递额外重要信息的作用。比如给浏览器和服务器提供报文主体大小、所使用的语言、认证信息等内容。
### HTTP首部字段结构
HTTP首部字段是由首部字段名和字段值构成，中间用冒号“：”分开。比如上面我们看到的：
```
Host: www.baidu.com
Connection: keep-alive
Pragma: no-cache
```
### 4种HTTP首部字段类型
HTTP首部字段根据实际用途可以分为以下4种：
- 通用首部字段：请求报文和响应报文两方都会使用的 首部。
- 请求首部字段：从客户端向服务器端发送请求报文时使用的首部。补充了请求的附加内容、客户端信息、响应内容相关优先级等信息。
- 响应首部字段：从服务器端向客户端返回响应报文时使用的首部。补充了响应的附加内容，也会要求客户端附加额外的内容信息。
- 实体首部字段：针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更新时间等与实体有关的信息。

### 通用首部字段
|首部字段名|说明|
|--|--|
| Cache-Control |控制缓存的行为，用于随报文传送缓存的指示|
| Connection |允许客户端和服务器指定与请求/响应连接有关的选项|
| Date |提供日期和时间标志，说明报文是什么时间创建的
|
| Pragma |报文指令，另一种随报文传送指示的方式，但并不专用于缓存|
| MIME-Version |给出了发送端使用的 MIME 版本|
| Trailer |如果报文采用了分块传输编码（chunked transfer encoding）方式，就可以用这个首部列出位于报文拖挂（trailer）部分的首部集合|
| Transfer- Encoding |告知接收端为了保证报文的可靠传输，对报文采用了什么编码方式|
| Update |给出了发送端可能想要 “升级” 使用的新版本或协议|
| Via |显示了报文经过的中间节点（代理、网关）|
| Warning |错误通知|
####  Cache-Control
通过指定首部字段Cache-Control的指令，就能操作缓存的工作机制。
指令的参数是可选的，多个指令之间通过“，”分隔。
**缓存请求指令**
|指令|参数|说明|
|--|--|--|
| no-cache |无|强制向资源服务器再次验证|
| no-store |无|不缓存请求或响应的任何内容|
| max-age = [秒] |必需|响应的最大Age值|
| max-stale ( = [秒]) |可忽略|接收已过期的响应|
| min-fresh = [秒] |必需|期望在指定时间内的响应仍有效|
| no-transform |无|代理不可更改媒体类型|
| only-if-cached |无|从缓存获取资源|
| cache-extension |-|新指令标记（token）|

**缓存响应指令**
|指令|参数|说明|
|--|--|--|
| public |无|可向任意方提供响应的缓存|
| private |可省略|仅向特定用户返回响应|
| no-cache |可省略|缓存前必须先确认其有效性|
| no-store |无|不缓存请求或响应的任何内容|
| no-transform |无|代理不可更改媒体类型|
| must-revalidate |无|可缓存但必须再向源服务器进行确认|
| proxy-revalidate |无|要求中间缓存服务器对缓存的响应有效性再进行确认|
| max-age = [秒] |必需|响应的最大Age值|
| s-max-age = [秒] |必须|公共缓存服务器响应的最大Age值|
| cache-extension |-|新指令标记（token）|

### 请求首部字段

|首部字段名|说明|
|--|--|
|Accept|用户可处理的媒体类型|
|Accept- Charset|优先的字符集|
|Accept- Encoding|优先的编码内容|
|Accept- Language|优先的语言|
| TE |传输编码的优先级|
| Expect |期待服务器的特定行为|
|If-Match|比较实体标记(ETAG)|
|If-Modified-Since|比较资源的更新时间|
|If-None-Match|比较实体标记(与If-Match相反)|
|If-Range|资源未更新时发送实体Btye的范围请求|
|If-Unmodified-Since|比较资源的更新时间(与If-Modified-Since相反)|
| Range |实体的字节范围请求|
| Authorization |WEB认证信息|
| Cookie |客户端用它向服务器传送一个令牌|
| Cookie2 |用来说明请求端支持的 cookie 版本|
|Max-Forward|在通往源端服务器的路径上，将请求转发给其他代理或网关的最大次数|
|Proxy-Authorization    |代理服务器要求客户端的认证信息|
### 响应首部字段
|首部字段名|说明|
|--|--|
| Age |推算资源创建经过时间|
| Public |服务器为其资源支持的请求方法列表|
|Retry-After    |对再次发起请求的时机要求|
|Title|对 HTML 文档来说，就是 HTML 文档 的源端给出的标题|
|Warning|比原因短语中更详细一些的警告报文|
|Accept-Ranges|服务器可接受的范围类型|
| Vary |代理服务器的安装信息|
|Proxy-Authenticate|代理服务器对客户端的认证信息|
|Set-Cookie|在客户端设置，以便服务器对客户端进行标识|
|WWW-Authenticate    |服务器对客户端的认证信息|
### 5. 实体首部字段
|首部字段名|说明|
|--|--|
| Allow |资源可支持的HTTP方法|
| Location |告知客户端实体实际上位于何处|
|Content-Base16    |解析主体中的相对 URL 时使用的基础 URL|
|Content-Encoding|实体主体适用的编码方式|
|Content-Language|实体主体的自然语言|
|Content-Length|实体主体的大小(单位：字节)|
|Content-Location|替代对应资源的URI|
|Content-MD5|实体主体的报文摘要|
|Content-Range|实体主体的位置范围|
| ETag |与此实体相关的实体标记|
| Expires |实体主体的过期时间|
|Last-Modified|资源的最后修改日期时间|

### 其他首部字段
HTTP 首部字段是可以自行扩展的。所以在 Web 服务器和浏览器的应用上，会出现各种非标准的首部字段。以下是最为常用的首部字段：
#### X-Frame-Options
X-Frame-Options 属于 HTTP 响应首部，用于控制网站内容在其他 Web 网站的 Frame 标签内的显示问题。其主要目的是为了防止点击劫持（clickjacking）攻击。首部字段 X-Frame-Options 有以下两个可指定的字段值：
- DENY：拒绝
- SAMEORIGIN：仅同源域名下的页面（Top-level-browsing-context）匹配时许可

####  X-XSS-Protection
X-XSS-Protection 属于 HTTP 响应首部，它是针对跨站脚本攻击（XSS）的一种对策，用于控制浏览器 XSS 防护机制的开关。首部字段 X-XSS-Protection 可指定的字段值如下:
- 0：将 XSS 过滤设置成无效状态
- 1：将 XSS 过滤设置成有效状态

####  DNT
DNT 属于 HTTP 请求首部，其中 DNT 是 Do Not Track 的简称，意为拒绝个人信息被收集，是表示拒绝被精准广告追踪的一种方法。首部字段 DNT 可指定的字段值如下：
- 0：同意被追踪 
- 1：拒绝被追踪 

####  P3P
P3P 属于 HTTP 响应首部，通过利用 P3P（The Platform for Privacy Preferences，在线隐私偏好平台）技术，可以让 Web 网站上的个人隐私变成一种仅供程序可理解的形式，以达到保护用户隐私的目的。要进行 P3P 的设定，需按以下操作步骤进行：
1. 创建 P3P 隐私
2. 创建 P3P 隐私对照文件后，保存命名在 /w3c/p3p.xml
3. 从 P3P 隐私中新建 Compact policies 后，输出到 HTTP 响应中

## HTTP响应状态吗
状态码的职责是当客户端向服务器端发送请求时，描述返回的请求结果。借助状态码，用户可以知道服务器端是正常处理了请求，还是出现了错误。
### 状态码类别
||类别|原因|
|--|--|--|
|1XX|nformational（信息性状态码）|接收的请求正在处理|
|2XX|Success（成功状态码）|请求正常处理完毕|
|3XX|Redirection（重定向状态码）|需要进行附加操作以完成请求|
|4XX|Client Error（客户端错误状态码）|服务器无法处理请求|
|5XX|Server Error（服务器错误状态码|服务器处理请求出错|
### 状态具体描述
在 RFC2616 中定义了 40 种 HTTP 状态码，webDAV ( Web-based Distributed Authoring and Versioning，基于万维网的分布式创作和版本控制)在 RFC4918 和 RFC5842 中，定义了一些特殊的状态码，在 RFC2518、RFC2817、RFC2295、RFC2774、RFC6585 中还额外定义了一些附加的 HTTP 状态码。总共有 60+ 种。具体链接可以见 [HTTP状态码 (wikipedia)](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)。虽然状态种类繁多，但实际上经常使用的大概有14种，我们简单介绍一下这14种。
|消息|描述|
|--|--|
|200 OK|请求成功（其后是对GET和POST请求的应答文档）|
|204 No Content|没有新文档。浏览器应该继续显示原来的文档|
|206 Partial Content|客户发送了一个带有Range头的GET请求，服务器完成了它|
|301 Moved Permanently|所请求的页面已经转移至新的url|
|302 Found|所请求的页面已经临时转移至新的url|
|303 See Other|所请求的页面可在别的url下被找到|
|304 Not Modified|未按预期修改文档|
|307 Temporary Redirect|被请求的页面已经临时移至新的url|
|400 Bad Request|服务器未能理解请求|
|401 Unauthorized|被请求的页面需要用户名和密码|
|403 Forbidden|对被请求页面的访问被禁止|
|404 Not Found|服务器无法找到被请求的页面|
|500 Internal Server Error|请求未完成，服务器遇到不可预知的情况|
|503 Service Unavailable|请求未完成，服务器临时过载或当机|

