---
layout:     post
title:      【译】不必要的 HTTP 头部信息
subtitle:   HTTP Headers
date:       2018-12-07
author:     Lvsi
header-img: 
catalog: true
tags:
    - HTTP
---

# 不必要的 HTTP 头部信息

> 原文 [《The headers we don't want》](https://www.fastly.com/blog/headers-we-dont-want)<br/>
> 译者：[Lvsi](https://github.com/Lvsi-China)

HTTP headers是控制缓存和浏览器处理Web内容的一种重要方式。但是许多不正确或者毫无意义的使用，会在加载页面的时刻增加额外开销，并且可能无法按预期工作。在关于header最佳实践的一系列文章的第一篇中，我们先看一些不必要的 header 。

大多数开发人员了解并依赖各种 HTTP headers 来使其内容有效。最常用的包括```Content-Type```和```Content-Length```，它们几乎都是通用的。但最近，```Content-Security-Policy``` 和 ```Strict-Transport-Security```等 header 已开始提高安全性，```Link rel = preload``` header 可提高性能。尽管不同的浏览器都广泛支持，但很少有网站会这么使用。

与此同时，有很多header非常受欢迎，但并不是新的，实际上并没有那么有用。我们可以使用[HTTP Archive](https://httparchive.org/)来证明这一点，该项目由Google运营并由Fastly赞助，每月使用WebPageTest加载500,000个网站，并在BigQuery中得出结果。

从HTTP的存档资料中来看，这里有30个最受欢迎的响应头（基于存档中使用不同响应头的域名个数），以及每个响应头的起到了多大的作用：

<img src="/img/posts/2018/12-07/1.png">

让我们看看不必要的header，看看为什么我们不需要它们，以及我们可以做些什么。

### Vanity (server, x-powered-by, via)

你可能对你选择的服务器软件感到非常自豪，但大多数人都不在乎。 最糟糕的是，这些header可能会泄露敏感数据，使你的网站更容易受到攻击。

```
Server: apache
X-Powered-By: PHP/5.1.1
Via: 1.1 varnish, 1.1 squid
```

[RFC7231](https://httpwg.org/specs/rfc7231.html#header.server)允许服务器在响应中包含```Server``` header，标识用于提供内容的软件。这通常是像“apache”或“nginx”这样的字符串。虽然它是允许的，但它不是强制性的，并且对开发人员或最终用户提供的价值非常小。不过，这是目前网络上第三个最受欢迎的HTTP响应头。 

<b></b>```X-Powered-By```是我们列表中最受欢迎的header，未在任何标准中定义，并且具有类似的用途，但通常指的是位于Web服务器后面的应用程序平台。常见值包括“ASP.net”，“PHP”和“Express”。同样，这并没有提供任何实际的好处，占用空间。

<b></b>更值得商榷的可能是Via，它需要（通过RFC7230）由任何代理添加到请求中，通过它传递以识别代理。这可以是代理的主机名，但更可能是通用标识符，如“vegur”，“varnish”或“squid”。在请求中删除（或不设置）此标头可能会导致代理转发循环。然而，有趣的是它也会在返回浏览器的过程中被复制到响应中，这里它只是信息性的，没有任何浏览器可以对它做任何事情，所以如果你愿意的话，摆脱它是相当安全的。

### 弃用的标准（P3P，Expires，X-Frame-Options，X-UA-Compatible）

另一类header是那些在浏览器中起作用但不是（或不再是）实现该效果的最佳方式的header。

```
P3P: cp="this is not a p3p policy"
Expires: Thu, 01 Dec 1994 16:00:00 GMT
X-Frame-Options: SAMEORIGIN
X-UA-Compatible: IE=edge
```

<b></b>```P3P```是一种好奇的动物。我不知道这是什么，更奇怪的是，最常见的价值之一是“这不是一个p3p政策”。嗯，是，还是不是？

这里的故事可以追溯到 [ attempt to standardise a machine readable privacy policy ](https://en.wikipedia.org/wiki/P3P#User_agent_support)。关于如何在浏览器中显示数据存在分歧，并且只有一个浏览器实现了它，就是 Internet Explorer。即使在IE中，P3P也没有对用户产生任何视觉效果;它只是允许访问iframe中第三方的cookie。有些网站甚至设置了一个不符合要求的P3P政策，就像上面提到的那样，即使这样做是在[不稳定的法律基础上](https://www.cylab.cmu.edu/_files/pdfs/tech_reports/CMUCyLab10014.pdf)。

毋庸置疑，读取第三方的cookie通常是一个坏主意，所以如果你不这样做，那么你就不需要设置P3P头了！

考虑到```Cache-Control```比```Expires```更受欢迎且已有20年之久，```Expires```几乎令人难以置信地受欢迎。如果```Cache-Control``` header 包含```max-age```指令，则将忽略同一响应上的任何Expires标头。但是有大量网站同时设置这两个网站，并且```Expires```header最常设置为 ```Thu, 01 Dec 1994 16:00:00 ```GMT，因为你希望不对内容进行缓存，并从中复制粘贴示例日期规范，肯定是这样的一种使用方式。

<img src="/img/posts/2018/12-07/2.png">

但是没有理由这样做。如果你有一个过去日期的 ```Expires``` header，请将其替换为：

```
Cache-Control: no-store, private
```

<b></b>```no-store ```是一个非常强大的指令，不将内容写入持久存储，因此根据你的使用情况，你可能实际上更喜欢无缓存以获得更好的性能，例如在使用后退/前进导航或恢复休眠标签时 。

审计的一些工具 你的网站会告诉你添加值为“SAMEORIGIN”的X-Frame-Options标头。 这告诉浏览器你拒绝被另一个站点框起来，并且通常可以很好地防止点击劫持。 但是，通过执行以下操作，可以实现相同的效果，具有更一致的支持和更强大的行为定义：

```
Content-Security-Policy: frame-ancestors 'self'
```

这有额外的好处是成为header（CSP）的一部分，无论如何由于其他原因（稍后将详细介绍）。 所以你现在可能没有X-Frame-Options。 

最后，回到他们的IE9时代，微软推出了“兼容性视图”，并且可能使用IE8或IE7引擎渲染页面，即使用户使用IE9进行浏览，如果浏览器认为该页面可能需要早期版本 好好工作。 这些启发式方法并不总是正确的，开发人员可以通过使用X-UA-Compatible标头或元标记来覆盖它们。 事实上，这越来越成为Bootstrap等框架的标准组成部分。 现在，这个header实现的很少 - 很少有人使用能够理解它的浏览器，如果你正在积极维护你的网站，你就不太可能使用会触发兼容性视图的技术。

### Debug Data（X-ASPNet-Version，X-Cache）

令人惊讶的是，一些常用的最流行的header没有任何标准。 从本质上讲，这意味着成千上万的网站似乎已经自发地同意以特定的方式使用特定的header。

```
X-Cache: HIT
X-Request-ID: 45a336c7-1bd5-4a06-9647-c5aab6d5facf
X-ASPNet-Version: 3.2.32
X-AMZN-RequestID: 0d6e39e2-4ecb-11e8-9c2d-fa7ae01bbebc
```

实际上，这些“未知”header并不是由网站开发人员单独和独立创建的。 它们通常是使用特定服务器框架，软件或特定供应商服务的人工制品（在此示例集中，最后一个标头是常见的AWS标头）。 

特别是，X-Cache实际上是由Fastly（其他CDN也这样做）添加的，以及其他与快速相关的标头，如X-Cache-Hits和X-Served-By。 启用调试后，我们会添加更多内容，例如Fastly-Debug-Path和Fastly-Debug-TTL。 

任何浏览器都无法识别这些header，删除它们对你的页面呈现方式没有任何影响。 但是，由于这些可能会为你（开发人员）提供有用的信息，因此你可能希望保留一种方法来启用它们。

### 对Pragma误解

我没想到会在2018年写一篇关于```Pragma```头的文章，但根据我们的HTTP档案数据，它仍然是第11大热门。 ```Pragma```不仅早在1997年就被弃用了，但它无论如何都不会被设计为响应头，按照规定，它只在作为请求的一部分时有意义。

```
Pragma: no-cache
```

然而，它作为响应头的使用是如此普遍，以至于一些浏览器也在这种情况下识别它。 目前，在响应上下文中，响应的```Pragma```被浏览器识别后并缓存，但浏览器不识别```Cache-Control```的可能性非常小。 如果你想确保不缓存某些东西，那么```Cache-Control：no-store，private```就是你所需要的。

### Non-Browser (X-Robots-Tag)

前30名headers中的一个header是 Non-Browser(非浏览器) hrader。 ```X-Robots-Tag```旨在供抓取工具使用，例如Google或Bing的机器人。由于它对浏览器没有意义，因此你可以选择仅在请求中 ```user-agent``` ( 用户代理 )是爬取程序时进行设置。同样，你可能会认为这使测试变得更难，或者可能会违反搜索引擎的服务条款。

### Bugs

最后，有些简单的Bug值得一提。在请求中，```Host```请求头是有意义的，但在响应中看到它则可能意味着你的服务器在某种程度上配置错误( 我确实很想知道是怎么配置导致了出错 )。然而，HTTP的存档资料中的68个域在其响应中返回了```Host```响应头。

### Removing headers at the edge

幸运的是，如果你的网站使用Fastly，使用[VCL](https://docs.fastly.com/vcl/)删除 headers 非常容易。有时你可能希望保留开发团队可用的真正有用的调试数据，但是对公共用户隐藏它，以便通过检测cookie或入站HTTP标头轻松完成：这里有交互操作，[请前往原网站进行操作](https://www.fastly.com/blog/headers-we-dont-want)

在本系列的下一篇文章中，我会讲解你应该设置的 headers 的最佳实践，以及如何在边缘启用它们。

> 有关HTTP协议的网站：[http://](https://httpwg.org/)
> 文中涉及到交互的那个网站：[fastlydemo](https://fiddle.fastlydemo.net/)
> HTTP Archive : [HTTP Archive](https://httparchive.org/)

