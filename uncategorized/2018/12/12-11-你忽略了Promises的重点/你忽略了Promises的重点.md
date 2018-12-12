---
layout:     post
title:      【译】你忽略了Promises的重点
subtitle:   关于 Promises
date:       2018-12-11
author:     Lvsi
header-img: 
catalog: true
tags:
    - Promises
---

# 【译】你忽略了Promises的重点

> 原文 [《You're Missing the Point of Promises》](https://blog.domenic.me/youre-missing-the-point-of-promises/) [Gist备用链接](https://gist.github.com/domenic/3889970) <br/>
> 译者：[Lvsi](https://github.com/Lvsi-China)

这篇文章最初是在[gist](https://gist.github.com/domenic/3889970)上出现的。从那时起，Promises/A+ 的发展似乎使其对 Promises/A 规范的重视有点过时了。

与互联网上的一些错误陈述相反，这里解释的 ```jQuery Promises``` 的问题在最近的版本中没有修复。 从2.1 beta 1开始，他们已经在此列出了所有相同的问题，并且根据一个jQuery核心团队成员所说，[他们将以](https://esdiscuss.org/topic/a-challenge-problem-for-promise-designers-was-re-futures#content-43)向后兼容的名义[永远保留](https://esdiscuss.org/topic/a-challenge-problem-for-promise-designers-was-re-futures#content-43)。

**Promise**是一种软件抽象，使得异步操作更加容易。在最基本的定义中，你的代码将从 ```continuation-passing ``` 风格：

```
getTweetsFor("domenic", function (err, results) {
  // the rest of your code goes here.
});
```

转变为这种：你的函数返回一个值，称为promise，表示该操作的最终结果。


```
var promiseForTweets = getTweetsFor("domenic");
```

这很强大，因为你现在可以将这些promises视为第一类对象，传递它们，聚合它们等等，而不是把与其他回调绑定在一起的虚拟回调插入以执行相同操作。

[最后]((https://www.slideshare.net/domenicdenicola/callbacks-promises-and-coroutines-oh-my-the-evolution-of-asynchronicity-in-javascript))我谈到了我认为的Promises很酷的地方,与此篇文章无关。相反，一个不好的的趋势是我在最近的JavaScript库中看到了增加了Promise支持：他们完全忽略了Promises的重点。

## Thenables和CommonJS Promises/A.

当有人在JavaScript中说 Promise 时，通常他们的意思，或者至少认为他们是说[ CommonJS Promises/ A ](http://wiki.commonjs.org/wiki/Promises/A)。这是我见过的最小的规范之一。它的内容完全是关于指定单个函数的行为，```then```：

> promise被定义为具有函数 ```then``` 作为属性值的对象：
>
> ```then(fulfilledHandler, errorHandler, progressHandler)```
>
> 添加一个```fulfilledHandler```，```errorHandler```，```progressHandler```。并且调用```progressHandler```以完成promise。当 Promise 完成时调用```fulfilledHandler```。当Promise失败时调用```errorHandler```。```progressHandler```用于处理事件。所有参数都是可选的，忽略非函数值。这 progressHandler不仅是一
> 个可选参数，而且进度事件纯粹是可选的。Promise实现者不需要调用progressHandler（progressHandler可以忽略），这个参数存在，以便
> 实现者可以调用它，如果他们有进度事件要报告。

> 此函数应返回在给定fulfilledHandler或errorHandler 回调完成时满足的新Promise。这允许将Promise操作链接在一起。回调处理程序返回的值是返
> 回的promise的履行值。如果回调引发错误，则返回的promise将移至失败状态。

人们大多理解第一段。它归结为回调聚合。你可以使用then将回调附加到Promise，无论是成功还是错误（甚至是进度）。当promise转换状态 - 这超出了这个非常小的规范的范围！ - 你的回调将被调用。我想这非常有用。

人们似乎没有注意到的是第二段。这是一个耻辱，因为它是最重要的一个。

## 什么是Promise的重点？

问题是，Promise不是关于回调聚合。这是一个简单的实用程序。Promise是关于更深层次的东西，即提供同步函数和异步函数之间的直接对应。

这是什么意思？那么，同步函数有两个非常重要的方面：

- 他们返回值
- 他们抛出异常

这两者基本上都与成分有关。也就是说，你可以将一个函数的返回值直接输入另一个函数，并继续无限期地执行此操作。更重要的是，如果在任何时候该进程失败，组合链中的一个函数可以抛出一个异常，然后绕过所有进一步的组合层，直到它进入可以用它处理它的人的手中catch。

现在，在异步世界中，你无法再返回值：它们根本没有及时准备好。同样，你不能抛出异常，因为没有人可以捕获它们。所以我们进入所谓的“回调地狱”，其中返回值的组成涉及嵌套回调，错误的组合涉及手动将它们传递到链上，哦，顺便说一句，你最好不要抛出异常，否则你我需要引入像域一样疯狂的东西。

Promise的目的是让我们回到异步世界中的功能组合和错误冒泡。他们这样说是你的函数应该返回一个promise，它可以做两件事之一：

- 完成并返回新的Promise
- 抛出异常被拒绝

而且，如果你有一个正确实现的then功能，遵循 Promises/A，那么完成和拒绝将像他们的同步对应物一样构成，```fulfillment```在一个组合链上流动，但在任何时候被拒绝只是由某人已准备好处理它而中断。

换句话说，以下异步代码：

```javascript
getTweetsFor("domenic") // promise-returning function
  .then(function (tweets) {
    var shortUrls = parseTweetsForUrls(tweets);
    var mostRecentShortUrl = shortUrls[0];
    return expandUrlUsingTwitterApi(mostRecentShortUrl); // promise-returning function
  })
  .then(httpGet) // promise-returning function
  .then(
    function (responseBody) {
      console.log("Most recent link text:", responseBody);
    },
    function (error) {
      console.error("Error with the twitterverse:", error);
    }
  );
```

并行同步代码：

```javascript
try {
  var tweets = getTweetsFor("domenic"); // blocking
  var shortUrls = parseTweetsForUrls(tweets);
  var mostRecentShortUrl = shortUrls[0];
  var responseBody = httpGet(expandUrlUsingTwitterApi(mostRecentShortUrl)); // blocking x 2
  console.log("Most recent link text:", responseBody);
} catch (error) {
  console.error("Error with the twitterverse: ", error);
}
```

请特别注意错误如何从处理中的任何步骤流向我们的catch处理程序，而没有明确的手动冒泡代码。随着即将推出的ECMAScript 6版本的JavaScript，代码不仅变得平行而且几乎完全相同。

## 第二段

所有这一切基本上都是由第二段启用的：

> 在给定的 ```fulfilledHandler```或```errorHandler``` 回调完成时，此函数应该返回一个已完成的新的Promise。这允许将Promise操
> 作链式在一起。回调处理程序返回的值是已完成的promise。如果回调抛出错误，则返回的promise将移至失败状态。

换言之，```then```不是一种将回调函数聚集到集合中的机制。它是一种将转换应用于Promise并从该转换中产生新Promise的机制 。

这解释了关键的第一句话：“这个函数应该返回一个新的Promise。”jQuery（1.8之前的）这样的库不会这样做：它们只是改变了现有promise的状态。这意味着如果你向多个消费者提供Promise，他们可能会干扰其状态。为了实现这是多么荒谬，考虑同步并行：如果你给两个人一个函数的返回值，其中一个可以某种方式将它改成一个抛出的异常！确实，Promises / A明确指出了这一点：

一旦Promise完成或失败，Promise的值绝不能改变，就像JavaScript，原语和对象标识中的值不能改变一样（尽管对象本身可能总是可变的，即使它们的身份不是）。

现在考虑最后两句话。他们告诉我们如何创造新的Promise。简而言之：

如果任一处理程序返回一个值，则使用该值实现新的promise。
如果任一处理程序抛出异常，则使用该异常拒绝新的promise。
这分为四种情况，具体取决于Promise的状态。在这里，我们给出了它们的同步并行，这样你就可以看出为什么对所有四个语义都有重要意义：

履行，履行处理程序返回一个值：简单的功能转换
完成后，履行处理程序抛出异常：获取数据，并抛出异常以响应它
拒绝，拒绝处理程序返回一个值：一个catch子句得到错误并处理它
拒绝，拒绝处理程序抛出异常：一个catch子句得到错误并重新抛出它（或一个新的）
如果不应用这些转换，你将失去同步/异步并行的所有功能，并且你所谓的“Promise”将成为简单的回调聚合器。这是jQuery当前“Promise”的问题：它们仅支持上面的场景1，省略了对场景2-4的完全支持。这也是Node.js 0.1的 EventEmitter“Promise”（甚至then不能）的问题。

此外，请注意，通过捕获异常并将其转换为拒绝，我们会处理有意和无意的异常，就像在同步代码中一样。也就是说，如果你aFunctionThatDoesNotExist()在任何一个处理程序中写入，你的Promise就会被拒绝，并且该错误会将链条冒泡到最近的拒绝处理程序，就像你写的一样throw new Error("bad data")。看马，没有域名！

## 所以呢？

也许你被我无情的逻辑和解释力所吸引。更有可能的是，你问自己为什么这个家伙对一些表现不佳的图书馆如此苛刻。

这是问题所在：

> promise被定义为具有函数作为属性值的对象 ```then```

作为 ```Promises/A-consuming``` 的作者，我们想假设这个陈述是真的：“可以”的东西实际上表现为Promises / A的Promise，具有所有的权力。

如果你可以做出这样的假设，那么你可以编[写非常广泛的库](https://github.com/domenic/chai-as-promised/)，这些库与他们所接受的Promise的实现完全无关！无论它们来自[Q](https://github.com/kriskowal/q)，[when.js](https://github.com/cujojs/when)，还是[WinJS](https://docs.microsoft.com/en-us/previous-versions/windows/apps/br211867(v=win.10))，你都可以使用```Promises/A```规范的简单组合规则来建立Promise行为。例如，这是一个与任何 Promises/A 实现都能一起使用的通用 [重试函数](https://gist.github.com/2936696)。

不幸的是，像jQuery这样的库打破了它。这需要丑陋的黑客来检测伪装成Promise的对象的存在，并且在自己的API文档Promise中称自己，但不是真正的Promises/A Promise。如果你的API的消费者开始尝试通过jQuery Promise，你有两种选择：当你的组合技术失败时，以神秘和难以破译的方式失败，或者预先失败并阻止他们完全使用你的库。这很糟糕。

## 未来之路

所以这就是为什么我要避免最终使用Ember解决```callback aggregator```问题。这就是我写这篇文章的原因。这就是为什么，在写完这篇文章的原始版本后的几个小时内，我编写了一个普遍的 [Promises/A合规套件](https://github.com/domenic/promise-tests)，我们可以在将来使用这些套件来响应到同样的页面。

自该测试套件发布以来，在 ```promise interoperability``` 和理解方面取得了很大进展。发布了一个库[rsvp.js](https://github.com/tildeio/rsvp.js)，其明确目标是提供 Promises/A 的这些功能。其他人则 效仿。但最令人兴奋的结果是 [Promises/A+](https://github.com/promises-aplus) 组织的形成。这是一个松散的实施者组织，他们制定了 [Promises/A+ 规范](http://promises-aplus.github.io/promises-spec/)，并将原始 Promises/A 规范阐明为明确且经过充分测试的内容。

当然，还有很多工作要做。值得注意的是，在目前编写的时候，最新的jQuery版本是1.9.1，并且它的promise实现在错误处理方面完全被破坏了。希望通过以上解释来设置阶段和Promises / A +规范和测试套件，这个问题可以在jQuery 2.0中得到纠正。

与此同时，这里是符合Promises/A +的库，因此我可以毫无保留地推荐：

- 问：Kris Kowal和我自己：一个功能齐全的promise库，具有强大的API表面，Node.js适配器，进度支持以及对长堆栈跟踪的初步支持。
- Yehuda Katz的RSVP.js：一个非常小巧轻便但仍完全符合规范的Promise库。
- 由Brian Cavalier 撰写的when.js：一个中间库，包含用于管理最终任务集合的实用程序，以及对进度和取消的支持。
如果你遇到像jQuery这样的来源的残缺“Promise”，我建议使用上述库中的一个同化实用程序（通常在名称下when）尽快转换为真正的Promise。例如：

```
var promise = Q.when($.get("https://github.com/kriskowal/q"));
// aaaah, much better
```
