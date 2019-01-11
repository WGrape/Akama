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
> 添加一个```fulfilledHandler```，```errorHandler```，```progressHandler```。并且调用```progressHandler```以完成promise。当 Promise 完成时调用```fulfilledHandler```。当Promise失败时调用```errorHandler```。```progressHandler```用于处理事件。所有参数都是可选的，忽略非函数值。这```progressHandler```不仅是一
> 个可选参数，而且进度事件纯粹是可选的。Promise实现者不需要调用```progressHandler```（```progressHandler```可以忽略），如果他们有进度事件需要报告，这个参数则不能为空以便
> 实现者可以调用它。
>
> 此函数在给定的```fulfilledHandler```或```errorHandler``` 回调完成时返回新的Promise。这允许链式操作Promise。回调处理程序返回的值是返
> 回的promise的 fulfillment 值。如果回调引发错误，则返回的promise将变成失败状态。

人们大多理解第一段。它归结为回调聚合。你可以使用```then```将回调附加到Promise，无论是成功还是错误（甚至是处理）。当promise转换状态，这比规范超出了很小的范围！你的回调将被调用。我想这非常有用。

人们似乎没有注意到的是第二段。这非常不好，因为它是最重要的一个！

## 什么是Promise的重点？

问题是，Promise不是关于回调聚合。这是一个简单的实用工具。Promise是关于更深层次的东西，即给同步函数和异步函数提供它们之间的直接对应。

这是什么意思？其实，同步函数有两个非常重要的方面：

- 返回值
- 抛出异常

这两者基本上都与组成有关。也就是说，你可以将一个函数的返回值直接输入另一个函数，并继续无限期地执行此操作。更重要的是，如果在任何时候该过程失败，组合链中的一个函数可以抛出一个异常，然后一步步的绕过链上的所有的函数，直到进入可以处理它的```catch```块中。

现在，在异步中，你无法再返回值：它们根本没有及时准备好。同样，你不能抛出异常，因为没有人可以捕获它们。所以我们进入所谓的“回调地狱”，其中返回值的会造成嵌套回调，错误的组合会手动将它们传递到链上，哦，顺便说一句，你最好不要抛出异常，否则你需要引入像[domains](https://nodejs.org/api/domain.html)一样疯狂的东西。

Promise的目的是让我们回到异步中来，使用功能组合和错误冒泡。这样做的话你的函数应该返回一个promise，它可以做两件事之一：

- 完成并返回新的Promise
- 抛出异常被拒绝

而且，如果你有一个正确实现了```then```的函数，遵循 Promises/A，那么完成和拒绝将像它们的```synchronous counterparts```一样构成。```fulfillment```在一个组合链上流动，但在任何时候被拒绝只能是由已准备好处理它的程序而中断。

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

相似的同步代码：

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

请特别注意错误如何从处理中的任何步骤流向我们的catch处理程序，而没有明确的手动冒泡代码。随着即将推出的ES6，代码不仅变得相似而且几乎完全相同。

## 第二段

所有这一切基本上都是由第二段启用的：

> 在给定的 ```fulfilledHandler```或```errorHandler``` 回调完成时，此函数应该返回一个已完成的新的Promise。这允许将Promise操
> 作链式在一起。回调处理程序返回的值是已完成的promise。如果回调抛出错误，则返回的promise将移至失败状态。

换言之，```then```不是一种将回调函数聚集到集合中的机制。它是一种将转换应用于Promise并从该转换中产生新Promise的机制 。

这解释了关键的第一句话：“这个函数应该返回一个新的Promise。”jQuery（1.8之前的）这样的库不是这样做的：它们只是改变了现有promise的状态。这意味着如果你向多个使用者(消费者-consumers)提供Promise，他们的状态可能会互相状态。太荒谬了，考虑同步并行：如果你给两个人一个函数的返回值，其中一个可以某种方式将它改成一个抛出的异常！确实，Promises/A 明确指出了这一点：

> 一旦Promise完成或失败，Promise的值绝不能改变，就像JavaScript，原语和对象标识中的值不能改变一样( 尽管对象本身可能总是可变的，但是它们本身不改变 )。

现在考虑最后两句话。他们告诉我们如何创建新的Promise。简而言之：

- 如果任一处理程序返回一个值，则使用该值实现新的promise。
- 如果任一处理程序抛出异常，则使用该异常拒绝新的promise。

这分为四种情况，具体取决于Promise的状态。在这里，我们给出了它们的同步并行，这样你就可以看出为什么四种情况都有重要意义：

1. Fulfilled ，Fulfilled 处理程序返回一个值：简单的功能转换
2. Fulfilled ，Fulfilled 处理程序抛出异常：获取数据，并抛出异常以响应它
3. Rejected  ，Rejected  处理程序返回一个值：一个```catch```子句获取错误并处理它
4. Rejected  ，Rejected  处理程序抛出异常：一个```catch```子句获取错误并再次抛出它(或一个新的)

如果不应用这些转换，你将失去```同步/异步```并行的所有功能，并且你所谓的“Promise”将成为简单的回调聚合器。这是jQuery目前“Promise”的问题：它们仅支持上面的场景1，省略了对场景2-4的完全支持。这也是Node.js 0.1的 
```EventEmitter-based``` Promise（甚至不是 ```thenable``` ）的问题。

此外，请注意，通过捕获异常并将其转换为拒绝，我们会处理有意和无意的异常，就像在同步代码中一样。也就是说，如果你在任何一个处理程序中写入```aFunctionThatDoesNotExist()```，你的Promise就会被拒绝，并且该错误会顺着Promise 链冒泡到最近的拒绝处理程序中，与你写的```throw new Error("bad data")```类似。看嘛，没有域！

## 所以呢？

也许你被我无情的逻辑和解释力所激起。更有可能的是，你会问自己为什么这个家伙对一些表现不佳的库要求如此苛刻。

这是问题所在：

> promise被定义为具有函数作为属性值的对象 ```then```

作为 ```Promises/A-consuming``` 的作者，我们想假设这个陈述是真的：具有```thenable```的对象实际上表现为 ```Promises/A``` Promise，它具有所该拥有的全部功能属性。

如果你可以做出这样的假设，那么你可以编[写非常广泛的库](https://github.com/domenic/chai-as-promised/)，这些库与他们所接受的Promise的实现完全无关！无论它们来自[Q](https://github.com/kriskowal/q)，[when.js](https://github.com/cujojs/when)，还是[WinJS](https://docs.microsoft.com/en-us/previous-versions/windows/apps/br211867(v=win.10))，你都可以使用```Promises/A```规范的简单组合规则来建立Promise行为。例如，这是一个与任何 Promises/A 实现都能一起使用的通用 [重试函数](https://gist.github.com/2936696)。

不幸的是，像jQuery这样的库破坏了规则。这需要[ugly hacks](https://github.com/domenic/chai-as-promised/blob/4bc1d6b217acde85c8af56dc3cd09f05bb752549/lib/chai-as-promised.js#L28-30)来检测是否有对象伪装成Promise存在，并且在自己的API文档中称为Promise，但不是真正的 ```Promises/A ``` Promise。如果你的API的用户尝试传递 jQuery Promise 时，你有两种很糟糕的选择：

- 你的源码内部出现奇怪和难以排查的错误而失败，
- 预先失败并阻止他们继续使用你的库。

## 未来之路

所以这就是为什么我要避免最终使用Ember解决```callback aggregator```问题。这就是我写这篇文章的原因。这就是为什么，在写完这篇文章的原始版本后的几个小时内，我编写了一个普遍的 [Promises/A合规套件](https://github.com/domenic/promise-tests)，我们可以在将来使用这些套件来响应到同样的页面。

自从该测试套件发布以来，在 ```promise interoperability``` 和理解方面取得了很大进展。发布了一个库[rsvp.js](https://github.com/tildeio/rsvp.js)，其明确目标是提供 ```Promises/A``` 的这些功能。但最令人兴奋的结果是 [Promises/A+](https://github.com/promises-aplus) 组织的形成。这是一个自由的组织，已经制定了 [Promises/A+ 规范](http://promises-aplus.github.io/promises-spec/)，并将原始的
```Promises/A``` 规范阐明为明确且经过了[充分测试(well-tested)](https://github.com/promises-aplus/promises-tests)。

当然，还有很多工作要做。值得注意的是，在目前编写的时候，最新的jQuery版本是1.9.1，并且它的promise实现在错误处理方面完全被破坏了。希望通过以上解释来设置阶段和Promises / A +规范和测试套件，这个问题可以在jQuery 2.0中得到纠正。

与此同时，这里有些符合 ```Promises/A+``` 的库，我会毫无保留地推荐给大家：

- 我参与开发的[Q](https://github.com/kriskowal/q)：一个功能齐全的Promise库，具有强大的API，NodeJS适配器，进度支持，以及对长堆栈跟踪的初步支持。
- [RSVP.js](https://github.com/tildeio/rsvp.js)：一个非常小巧轻便但仍完全符合规范的Promise库。
- [when.js](https://github.com/cujojs/when)：一个中间库，具有管理最终任务集合(managing collections of eventual tasks)，以及对进度和取消的支持。

如果你遇到像JQuery这样不标准的“Promise”，我建议使用上述库中其中的一个( 通常调用名称下的```when```属性 )尽快转换为真正的Promise。例如：

```
var promise = Q.when($.get("https://github.com/kriskowal/q"));
// aaaah, much better
```
