---
layout:     post
title:      【译】HTTP简史
subtitle:   HTTP简史
date:       2019-01-07
author:     Lvsi-China
header-img: 
catalog: true
tags:
    - HTTP
---

# HTTP简史

> 原文 [Brief History of HTTP](https://hpbn.co/brief-history-of-http/)<br/>
> 译者：[Lvsi-China](https://github.com/Lvsi-China)

超文本传输​​协议（HTTP）是Internet中最普遍和广泛采用的应用程序协议之一：它是客户端和服务器之间的通用语言，支持现代Web。从简单的单一关键字和文档路径开始，它已成为不仅适用于浏览器，而且适用于几乎所有连接互联网的软件和硬件应用程序的协议。

在本章中，我们将简要介绍HTTP协议的演变。对不同HTTP语义的完整讨论超出了本书的范围，但是要理解HTTP的关键设计变化，以及每个变化背后的动机，将为我们讨论HTTP性能提供必要的背景知识，特别是在即将到来的HTTP/2许多改进的背景之下。

## HTTP0.9 单线协议

Tim Berners-Lee 最初的HTTP提案在设计时考虑到了简单性，以帮助他采用他的另一个新生想法：万维网。该策略似乎有效：有抱负的协议设计者，请注意。

1991年，Berners-Lee概述了新协议的动机并列出了几个高级设计目标：文件传输功能，请求索引搜索超文本存档的能力，格式协商以及将客户端引用到另一个服务器的能力。为了证明该理论的实际应用，我们构建了一个简单的原型，它实现了所提议功能的一小部分：

- 客户端请求是单个ASCII字符串。
- 客户请求由回车（CRLF）终止。
- 服务器响应是ASCII字符流。
- 服务器响应是一种超文本标记语言（HTML）。
- 文档传输完成后终止连接。

然而，即便这听起来比实际复杂得多。但实际这些规则启用的是一个非常简单的服务器都支持的Telnet-friendly 协议。
```
$> telnet google.com 80
 
已连接到74.125.xxx.xxx 

GET / about / 

(hypertext response)
(connection closed)
```
请求由单行：GET方法和所请求文档的路径组成。响应是单个超文本文档 - 没有标题或任何其他元数据，只有HTML。它真的不能变得更简单。此外，由于先前的交互是预期协议的子集，因此它非正式地获取HTTP 0.9标签。其余的，正如他们所说，是历史。

从1991年这些不起眼的开始，HTTP开始了自己的生命，并在未来几年迅速发展。让我们快速回顾一下HTTP 0.9的功能：

* 客户端 - 服务器，请求 - 响应协议。
* ASCII协议，通过TCP / IP链路运行。
* 旨在传输超文本文档（HTML）。
* 每次请求后，服务器和客户端之间的连接都将关闭。

> 流行的Web服务器，如Apache和Nginx，仍然支持HTTP 0.9协议 - 部分原因，因为它没有多少！如果您感到好奇，请打开Telnet会话并尝试通过HTTP 0.9访问google.com或您自己喜欢的网站，并检查此早期协议的行为和限制。

## HTTP/1.0：快速增长和信息化RFC
1991年至1995年期间是HTML规范的快速共同进化，一种被称为“网络浏览器”的新型软件，以及面向消费者的公共互联网基础设施的出现和快速增长。

```
#### 完美风暴：20世纪90年代初的互联网热潮
在Tim Berner-Lee最初的浏览器原型的基础上，国家超级计算应用中心（NCSA）的一个团队决定实施他们自己的版本。有了这个，第一个流行的浏览器诞生了：NCSA Mosaic。1994年10月，NCSA团队的一名程序员Marc Andreessen与Jim Clark合作创建了Mosaic Communications。该公司后来改名为Netscape，并于1994年12月发布了Netscape Navigator 1.0。到目前为止，已经很清楚了万维网必将引起更多的不仅仅是一个学术好奇心。

实际上，同年第一次万维网会议在瑞士日内瓦举办，这导致创建了万维网联盟（W3C），以帮助指导HTML的发展。同样，在IETF内部建立了并行HTTP工作组（HTTP-WG），专注于改进HTTP协议。这两个群体继续有助于网络的发展。

最后，为了创造完美的风暴，CompuServe，AOL和Prodigy在1994-1995相同的时间框架内开始向公众提供拨号上网服务。凭借这股迅速采用的浪潮，Netscape在1995年8月9日以非常成功的IPO创造了历史 - 互联网热潮已经到来，每个人都想要它的一部分！
```

新兴网络的所需功能及其在公共网站上的用例的不断增加的列表很快暴露了HTTP 0.9的许多基本限制：我们需要的协议不仅可以提供超文本文档，还可以提供有关请求的更丰富的元数据。响应，启用内容协商等。反过来，新兴的Web开发人员社区通过临时流程生成大量实验性HTTP服务器和客户端实现来做出响应：实现，部署，并查看其他人是否采用它。

从这段快速实验开始，一系列最佳实践和常见模式开始出现，1996年5月，HTTP工作组（HTTP-WG）发布了RFC 1945，它记录了许多HTTP/1.0实现的“常见用法”。在野外发现。请注意，这只是一个信息RFC：HTTP/1.0，因为我们知道它不是正式规范或Internet标准！

话虽如此，示例HTTP/1.0请求看起来应该非常熟悉：

```
$> telnet website.org 80

Connected to xxx.xxx.xxx.xxx

GET /rfc/rfc1945.txt HTTP/1.0 
User-Agent: CERN-LineMode/2.15 libwww/2.17b3
Accept: */*

HTTP/1.0 200 OK 
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 01 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 1 May 1996 12:45:26 GMT
Server: Apache 0.84

(plain-text response)
(connection closed)
```
1. 具有HTTP版本号的请求行，后跟请求标头
2. 响应状态，后跟响应标头

前面的交换不是HTTP/1.0功能的详尽列表，但它确实说明了一些关键的协议更改：

* 请求可能包含多个换行符分隔的标题字段。
* 响应对象以响应状态行为前缀。
* 响应对象有自己的一组换行符分隔的标题字段。
* 响应对象不限于超文本。
* 每次请求后，服务器和客户端之间的连接都将关闭。

请求和响应头都保存为ASCII编码，但响应对象本身可以是任何类型：HTML文件，纯文本文件，图像或任何其他内容类型。因此，HTTP的“超文本传输​​”部分在引入后不久就变成了用词不当。实际上，HTTP已经迅速发展成为超媒体传输，但原始名称仍然存在。

除了媒体类型协商之外，RFC还记录了许多其他常用功能：内容编码，字符集支持，多部分类型，授权，缓存，代理行为，日期格式等。

> 今天，Web上的几乎所有服务器都可以并且仍将使用HTTP/1.0。除此之外，到现在为止，你应该知道更好！每个请求需要新的TCP连接会对HTTP/1.0造成严重的性能损失; 看三态握手，然后是慢启动。

## HTTP/1.1 Internet标准
将HTTP转变为官方IETF互联网标准的工作与围绕HTTP/1.0的文档工作并行进行，并发生在大约四年的时间内：1995年至1999年。事实上，第一个正式的HTTP/1.1标准定义于RFC 2068，在HTTP/1.0发布大约六个月后于1997年1月正式发布。然后，两年半之后，即1999年6月，标准中包含了许多改进和更新，并作为RFC 2616发布。

HTTP/1.1标准解决了早期版本中发现的许多协议歧义，并引入了许多关键性能优化：keepalive连接，分块编码传输，字节范围请求，附加缓存机制，传输编码和请求流水线。

有了这些功能，我们现在可以检查由任何现代HTTP浏览器和客户端执行的典型HTTP/1.1会话：

```bash
$> telnet website.org 80
Connected to xxx.xxx.xxx.xxx

GET /index.html HTTP/1.1 
Host: website.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_4)... (snip)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate,sdch
Accept-Language: en-US,en;q=0.8
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.3
Cookie: __qca=P0-800083390... (snip)

HTTP/1.1 200 OK 
Server: nginx/1.0.11
Connection: keep-alive
Content-Type: text/html; charset=utf-8
Via: HTTP/1.1 GWA
Date: Wed, 25 Jul 2012 20:23:35 GMT
Expires: Wed, 25 Jul 2012 20:23:35 GMT
Cache-Control: max-age=0, no-cache
Transfer-Encoding: chunked

100 
<!doctype html>
(snip)

100
(snip)

0 

GET /favicon.ico HTTP/1.1 
Host: www.website.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_4)... (snip)
Accept: */*
Referer: http://website.org/
Connection: close 
Accept-Encoding: gzip,deflate,sdch
Accept-Language: en-US,en;q=0.8
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.3
Cookie: __qca=P0-800083390... (snip)

HTTP/1.1 200 OK 
Server: nginx/1.0.11
Content-Type: image/x-icon
Content-Length: 3638
Connection: close
Last-Modified: Thu, 19 Jul 2012 17:51:44 GMT
Cache-Control: max-age=315360000
Accept-Ranges: bytes
Via: HTTP/1.1 GWA
Date: Sat, 21 Jul 2012 21:35:22 GMT
Expires: Thu, 31 Dec 2037 23:55:55 GMT
Etag: W/PSA-GAu26oXbDi

(icon data)
(connection closed)
```
1. 请求HTML文件，包含编码，字符集和cookie元数据
2. 原始HTML请求的分块响应
3. 块中的八位字节数表示为ASCII十六进制数（256字节）
4. 分块流响应结束
5. 在同一TCP连接上请求创建的图标文件
6. 通知服务器不会重用连接
7. 图标响应，然后关闭连接

哎呀，那里有很多事情发生！第一个也是最明显的区别是我们有两个对象请求，一个用于HTML页面，另一个用于图像，两者都通过单个连接传递。这是连接keepalive的实际应用，它允许我们重用现有的TCP连接，以便对同一主机发出多个请求，并提供更快的最终用户体验; 请参阅优化TCP。

要终止持久连接，请注意第二个客户端请求close通过Connection标头向服务器发送显式令牌 。类似地，一旦传输响应，服务器就可以通知客户端关闭当前TCP连接的意图。从技术上讲，任何一方都可以在没有此类信号的情况下终止TCP连接，但客户端和服务器应尽可能提供它以在双方上实现更好的连接重用策略。

> HTTP/1.1默认情况下将HTTP协议的语义更改为使用连接keepalive。这意味着，除非另有说明（通过Connection: close标题），否则 服务器应默认保持连接处于打开状态。
>
> 但是，同样的功能也被反向移植到HTTP/1.0并通过Connection: Keep-Alive标头启用。因此，如果您使用HTTP/1.1，从技术上讲，您不需要 Connection: Keep-Alive标头，但许多客户仍然选择提供它。

此外，HTTP/1.1协议添加了内容，编码，字符集，甚至语言协商，传输编码，缓存指令，客户端cookie，以及可以在每个请求上协商的十几种其他功能。

我们不打算详述每个HTTP/1.1功能的语义。这是一本专门的书的主题，已经写了很多很棒的书。相反，前面的示例可以很好地说明HTTP的快速进展和演变，以及每个客户端 - 服务器交换的错综复杂的舞蹈。那里有很多事情发生！

> 有关HTTP协议所有内部工作原理的详细参考，请查看David Gourley和Brian Totty撰写的O'Reilly的HTTP：The Definitive Guide。

## HTTP/2 提高运输性能
自发布以来，RFC 2616已经成为互联网空前增长的基础：数十亿台各种形状和大小的设备，从台式电脑到我们口袋里的小型网络设备，每天都会通过HTTP来传送新闻，视频，以及数以百万计的其他网络应用程序我们都依赖于我们的生活。

最初用于检索超文本的简单单行协议最终演变为通用的超媒体传输，现在十年后可用于为您能想象的任何用例提供支持。无处不在的服务器可以说协议以及客户端使用它的广泛可用性意味着现在许多应用程序都是专门在HTTP之上设计和部署的。

需要一个协议来控制你的咖啡壶？RFC 2324已经涵盖了超文本咖啡壶控制协议（HTCPCP / 1.0） - 原本是IETF的愚人节笑话，并且在我们新的超连接世界中越来越多的笑话。

> 超文本传输​​协议（HTTP）是用于分布式协作超媒体信息系统的应用程序级协议。它是一种通用的无状态协议，可以通过扩展其请求方法，错误代码和标头，用于超出其用于超文本的许多任务，例如名称服务器和分布式对象管理系统。HTTP的一个特性是数据表示的输入和协商，允许系统独立于正在传输的数据而构建。
>
> RFC 2616: HTTP/1.1, June 1999

HTTP协议的简单性使其最初的采用和快速增长成为可能。事实上，现在发现使用HTTP作为主要控制和数据协议的嵌入式设备（传感器，执行器和咖啡壶）并不罕见。但在其自身成功的重压下，随着我们越来越多地继续将我们的日常互动转移到网络 - 社交，电子邮件，新闻和视频，以及越来越多的个人和工作空间 - 它也开始显示出压力的迹象。用户和Web开发人员现在都要求HTTP/1.1提供近乎实时的响应和协议性能，如果不进行一些修改，它就无法满足。

为了应对这些新挑战，HTTP必须继续发展，因此HTTPbis工作组在2012年初宣布了一项针对HTTP/2的新计划：

> 新的实现经验和对保留HTTP语义的协议感兴趣，而没有HTTP/1.x消息框架和语法的遗留问题，这些问题已被确定为妨碍性能并鼓励滥用底层传输。
>
> 工作组将在有序的双向流中生成HTTP当前语义的新表达式的规范。与HTTP/1.x一样，主要目标传输是TCP，但应该可以使用其他传输。
>
> HTTP/2 charter, January 2012

HTTP/2的主要重点是提高传输性能并实现更低的延迟和更高的吞吐量。主要的版本增量听起来是一个很大的步骤，就性能而言，它将是一个重要的步骤，但重要的是要注意，没有任何高级协议语义受到影响：所有HTTP头，值和用例是相同的。

任何现有的网站或应用程序都可以并且将通过HTTP/2传送而无需修改：您无需修改​​应用程序标记以利用HTTP/2。HTTP服务器必须使用HTTP/2，但这应该是大多数用户的透明升级。如果工作组实现其目标，唯一的区别应该是我们的应用程序以更低的延迟和更好的网络链接利用率交付！

话虽如此，让我们不要超越自己。在我们开始使用新的HTTP/2协议功能之前，值得退一步并检查我们现有的HTTP/1.1部署和性能最佳实践。HTTP/2工作组正在新规范上取得快速进展，但即使最终标准已经完成并准备就绪，我们仍然必须在可预见的未来支持旧的HTTP/1.1客户端 - 实际上，十年或更长时间。