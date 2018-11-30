---
layout:     post
title:      【译】对于静态站点，没有理由不使用CDN
subtitle:   托管你的网站到CDN
date:       2018-11-30
author:     Lvsi
header-img: 
catalog: true
tags:
    - CDN
---

## 【译】对于静态站点，没有理由不使用CDN

> 翻译文章来自 [《For Static Sites, There’s No Excuse Not to Use a CDN》](https://forestry.io/blog/for-static-sites-theres-no-excuse-not-to-use-a-cdn/#the-future-is-static)<br/>
> 译者：[Lvsi](https://github.com/Lvsi-China)

你是否充分利用了你的静态网站？如果你没有在CDN上托管你的网站，那你绝对没有充分利用。

CDN，或内容分发网络通常用于分发静态资源，如图像，视频和客户端代码（CSS和JavaScript）。由于静态网站完全由静态资源（包括HTML页面）组成，因此很有可能通过CDN去服务整个网站！

在本文中，我们将探讨为什么你应该使用CDN来托管你的静态站点，以及如何使用**Netlify**来实现它。 使用Netlify的CDN可以轻松部署和托管你的静态站点，你没有理由不使用！

### Ping 问题

评估网页性能时使用的一个指标是第一个字节的时间或TTFB( time to first byte )的时间。 这可以衡量客户端开始从服务器接收数据所需的时间。

造成 TTFB 增加的两个主要问题：

1. 必须等待数据在互联网的物理线路上的传输。 
2. 必须等待服务器在收到请求后处理请求，并在发送之前准备响应。

对于像WordPress这样的动态网站来说，问题＃2是一个经常出现的瓶颈。 为了准备要发送给客户端的响应，服务器有很多WordPress代码需要运行。但是，对于静态站点，这几乎是即时的，因为页面都是在部署站点时已经生成的。问题＃2对我们来说真的不是主要问题！

那么，对于问题#1，我们需要做些什么？

数据在网络中传输所需的时间称为延迟。 延迟主要是物理限制：信号需要穿过电缆; 电缆越长，信号传输的时间越长。理论上，我们可以以光速传输信号，但是实际上数据的传输速度要慢得多。

#### 测试延迟

你可能听说过网络延迟，比为提到ping时。 此名称来自可用于测量延迟的```ping```工具。在你的电脑上几乎不可能没有这个工具。

你可以自己使用ping工具来了解延迟的工作原理。打开终端并键入以下命令：

```html
ping ec2.us-east-1.amazonaws.com
```

此命令将向Amazon的```us-east-1```区域中的服务器发送请求，并等待响应。 ```us-east-1```地区位于美国北弗吉尼亚州。 当我从马里兰州的办公室运行此命令时，我得到以下响应：

```html
PING ec2.us-east-1.amazonaws.com (54.239.28.176) 56(84) bytes of data. 
64 bytes from 54.239.28.176: icmp_seq=1 ttl=232 time=50.3 ms
```
ping工具的报告显示：数据从我的计算机传输到亚马逊的服务器并返回需要50.3毫秒。

通过在命令中改变主机名，从```ec2.us-east-1.amazonaws.com```到```ec2.ap-southeast-2.amazonaws.com```，相反的我可以 ping 悉尼，澳大利亚地区。

```html
PING ec2.ap-southeast-2.amazonaws.com (54.240.195.243) 56(84) bytes of data. 
64 bytes from 54.240.195.243: icmp_seq=1 ttl=233 time=301 ms
```

与更远的服务器通信导致往返延迟增加了500％。 250毫秒可能看起来是很短暂的一个时间，但这个时间的延迟惩罚施加于用户对该服务器的每个请求中：CSS，JavaScript，图像等。即使少量的延迟也会对你的网站产生负面影响。 如果有一种简单的方法来消除这种延迟，你不想做吗？

> 如果你想要尝试ping其他区域，请查看[EC2区域](http://ec2-reachability.amazonaws.com/)的此图表。你可以直接ping IP地址，或使用遵循约定ec2的主机名 ```$ REGION.amazonaws.com```。

### 使用 CDN 来降低网络延迟

有许多技术依赖于实时点对点通信。 静态站点不是其中之一！ 解决此延迟问题的一种方法是将我们的网站复制到遍布全球的多个服务器上。 比如，我可以将我的站点的两个副本部署到AWS：一个副本在```us-east-1```区域，另一个副本早```ap-southeast-2```区域，并以某种方式将用户的请求分发到更接近他们的服务器。

这正是CDN可以为我们做的！CDN代表内容分发网络，其目的是将资源复制到分布在世界各地的```边缘节点```网络上。 当用户请求由CDN托管的资源时，请求就会被路由到最靠近他们的边缘节点上。

为了证明这一点，我征求了我的朋友Chris的帮助，他住在美国的另一边在加利福尼亚州圣地亚哥。 对于这个实验，我们都对两个主机运行```ping```命令。 第一个主机与上一个示例中的```ec2.us-east-1.amazonaws.com```相同，第二个主机是```forestry.io```。 Forestry 托管在AWS的CloudFront CDN上，所以我猜想它需要花更长的时间才能到达```us-east-1```服务器，但是我们两个人都要快速到达。结果证实了这一假设：

#### ping 响应时间

|  | DJ (Maryland)	| Chris (California) |
| --- | --- | --- |
| ec2.us-east-1.amazonaws.com	 | 50 ms | 80ms |
| forestry.io	 | 33ms | 15ms  |

现在，有几个影响ping的因素，但这里有一点：我们都收到了来自```forestry.io```的回复，比我们收到北弗吉尼亚州服务器的回复要快，尽管它位于美国的两端。

通过检查```forestry.io```的IP地址的PTR记录，我们可以更好地了解这里发生了什么。为此，请运行以下命令：

```html
dig -x $IPADDRESS
```

其中```$IPADDRESS```是ping命令报告的已解析IP。你的输出将如下所示：

```html
dig -x 216.137.41.157 

; <<>> DiG 9.10.3-P4-Ubuntu <<>> -x 216.137.41.157 
;; global options: +cmd 
;; Got answer: 
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25470 
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1 

;; OPT PSEUDOSECTION: 
; EDNS: version: 0, flags:; udp: 1280 
;; QUESTION SECTION: 
;157.41.137.216.in-addr.arpa.   IN      PTR 

;; ANSWER SECTION: 
157.41.137.216.in-addr.arpa. 68401 IN   PTR     server-216-137-41-157.ewr2.r.cloudfront.net. 

;; Query time: 38 msec 
;; SERVER: 127.0.1.1#53(127.0.1.1) 
;; WHEN: Thu Jun 07 12:50:18 EDT 2018 
;; MSG SIZE  rcvd: 113
```

在答案部分记下PTR记录。 此IP指向```server-216-137-41-157.ewr2.r.cloudfront.net```。 这是CloudFront的边缘节点之一，```ewr2```指示出该节点位于新泽西州纽瓦克市（Newark, New Jersey ）。

当Chris运行```ping```命令时，```forestry.io```解析为另一个IP地址。 在此IP地址上运行相同的命令，我们看到```server-52-85-83-253.lax1.r.cloudfront.net```。 我们再次遵循AWS的命名约定，推断出```lax```子域意味着该节点位于洛杉矶。

#### 要点
使用CDN可以让我们在世界各地托管我们网站的副本。当用户请求我们的站点时，CDN会将它们定向到最近的副本以最小化网络延迟。

### 在CDN上托管你的静态站点

如果前一部分让你觉得将静态站点放在CDN上很复杂，请不要担心。 Netlify让它变得简单！ 他们的服务负责部署和托管你的静态站点，而设置只需几个步骤。查看[Netlify文档](https://www.netlify.com/docs/welcome/#continuous-deployment)，了解有关使用Netlify进行部署和托管的更多信息，或访问其[注册页面](https://app.netlify.com/signup)，以便在几分钟内快速部署你的站点。

> Netlify不仅简化了设置：他们的部署过程会自动处理一些非常棘手的事情，如原子部署和即时缓存失效。查看其功[能页面](https://www.netlify.com/features/)以了解更多信息。

### 静态是未来

感谢由于静态网站，你的网站得以快速，高度可用于服务全球受众，从未如此简单。 选择#gostatic会让你获得出色的性能，使用CDN来托管你的网站将确保你的网站能够快速服务于所有人。 由于Netlify的简单性，没有理由不尝试它！

