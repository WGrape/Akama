---
layout:     post
title:      【译】Immer在React社区的崛起
subtitle:   React
date:       2018-12-20
author:     WGrape
header-img: 
catalog: true
tags:
    - React
---

# 【译】Immer在React社区的崛起

> 原文 [《The Rise of Immer in React》](https://www.netlify.com/blog/2018/09/12/the-rise-of-immer-in-react/)<br/>
> 译者：[WGrape](https://github.com/WGrape)

```不变性```正在发生着变化。至少，我们在React中进行```不变性```的方式正在发生着变化。（我们都懂这个反语）

<b></b><img src="./images/1.png" />

## 历史
JavaScript中不可变性的需求并不明显。从本质上说，不可变性的主要优点是[较少担心的并发](https://www.infoq.com/articles/dhanji-prasanna-concurrency)，但由于JavaScript是单线程的，因此并没有太大的好处。

React的不变性历史可以追溯到2013年12月在[David Notlen首次引入Om时](https://swannodette.github.io/2013/12/17/the-future-of-javascript-mvcs)。Om是使ClojureScript用户能够使用的React wrapper(包装器)，但Om的奇怪之处在于它比React更快。

**一个东西的wrapper怎么能比这个东西更快？**David在[这里](https://www.youtube.com/watch?v=DMtwq3QtddY)说了很多，但主要原因是不可改变的数据。React的大部分工作都是[Reconciliation(协调)](https://reactjs.org/docs/reconciliation.html)，如果可以简单地比较对象、数组和[memoize函数](https://github.com/reactjs/react-basic#memoization)，你可以跳过很多Reconciliation。React Fiber的数据结构有效地在黑盒下(under the hood)执行了大量记忆操作，以避免重复工作。

在2015年，多产的Lee Byron用[Immutable.js](https://www.youtube.com/watch?v=I7IdS-PbEgI)（[这里](https://vimeo.com/144790954)有ReactConf讨论的其他版本，还有[Q&A](https://blog.adroll.com/news/lee-byron-immutable))使其成为[JavaScript的专用不可变性库](https://facebook.github.io/immutable-js/)，并将这一进步带入到React主流。特别要注意，他的观点是可变对象完成时间和值，以及低层次结构共享的好处。

鉴于其与Flux的原理相似性，Immutable.js 很快被 [Redux社区采用](https://redux.js.org/recipes/usingimmutablejs)（以及[其他67个替代品](https://github.com/markerikson/redux-ecosystem-links/blob/master/immutable-data.md#immutable-update-utilities)），我们[也在Netlify中采用了它](https://www.netlifycms.org/docs/architecture/)！不变性得以解决！对吧？

## Immer的用户，文化，社区
早在2018年，Michel Weststrate开放Immer源码。我就不在这里重复了，推荐你们读他的[介绍博客](https://hackernoon.com/introducing-immer-immutability-the-easy-way-9d73d8f71cb3)和
[项目自述](https://github.com/mweststrate/immer)，我还推荐他的[React Finland](https://www.youtube.com/watch?v=-gJbS7YjcSo)演讲( [幻灯片](https://immer.surge.sh/) )作为后续行动。

大家对Immer的反应，让公司很开心：

> 尽管存在缺点，但是Immer不仅满足这两个要求，而且重量轻、简单，并且通常具有性能。到目前为止，开发人员都喜欢使用Immer；它是非侵入性的，并且易于使用，学习曲线很少。- 
[Workday Engineering](https://medium.com/workday-engineering/workday-prism-analytics-the-search-for-a-strongly-typed-immutable-state-a09f6768b2b5)

和教师：

> 我目前更喜欢Immer。- [Cory House](https://medium.freecodecamp.org/handling-state-in-react-four-immutable-approaches-to-consider-d1f5c00249d5)

和开源维护者：

> 我会选择Immer - [Mark Erikson](https://twitter.com/acemarke/status/999436116280262656)

> react-copy-write在内部使用 @mweststrate 的Immer。它允许你改变对象的草稿以处理不可变更新。因为它使用结构共享，所以react-copy-write非常适合在需要时重新渲染 - [Brandon Dail](https://twitter.com/aweary/status/984828941595652096)

和React团队：

> “如果你喜欢MobX，我强烈推荐跟随@mweststrate关于Immer的工作。而MobX与我们使用React的设想相去甚远。Immer是正确的。” - [Sebastian Markbåge](https://twitter.com/sebmarkbage/status/1032684851063705600)

## React和Immer的“为什么”
因此，Immer显然是很奇妙的，值得花一些时间思考为什么Immer的理念与React的原理最适合，以理解相关的原理。

当谈到理解React的原理时( 与现在的React不同 )，我经常提到两个文档：官方文档上的[设计原则](https://reactjs.org/docs/design-principles.html)和[React基础](https://github.com/reactjs/react-basic)。我将通过伪代码描述React中的核心理念，重点介绍与Immer相关的三个概念：

### 时间可变性

[React的数据模型是不可变的](https://github.com/reactjs/react-basic#state)，具有状态更新程序功能。使用简单的Click Counter应用程序，我们故意不在React中执行此操作：

```js
// MobX?
clickHandler = () => this.state.count++
```

虽然你可以在内部重写React来支持它( 或者[在用户空间上用MobX](https://dev.to/swyx/introduction-to-mobx-4-for-reactredux-developers-3k07)）。相反，我们写：

```js
// React
clickHandler = () => this.setState(state => ({
    count: state.count + 1
}))
```
这与Immer有很大的相似之处：

```js
// Immer
const nextState = produce(currentState, draft => {
    draft.count = draft.count + 1
})
```

### 互通性

[Immutable.js最大的问题](https://redux.js.org/recipes/usingimmutablejs#what-are-the-issues-with-using-immutable-js)是互操作性的难度。尽管Netlify上的Immutable.js用户已经超过2年，但我们每次尝试结构Immutable.js时，我们都不会只使用JavaScript的```Map```：

```js
// Immutable.js
const map1 = Immutable.Map({ foo: 1, bar: 2 })
const { foo, bar } = map1
console.log(foo) // undefined
console.log(bar) // undefined
```
这使得Immutable.js不是完整的抽象，因为我们必须不断思考我们正在操作的变量是否包含在Immutable.js中。

使用Immer，你的对象和数组实际上是JavaScript对象和数组，因此你可以执行通常所做的一切：

```js
// Immer
const map1 = { foo: 1, bar: 2 }
const map2 = produce(map1, draft => {
    draft.foo += 10
})
const { foo, bar } = map2
console.log(foo) // 11
console.log(map1.bar === bar) // true
```
这也是React成功的原因 - [React对互操作性的关注](https://reactjs.org/docs/design-principles.html#interoperability)允许逐步采用（而不是必须一次性转换所有内容）以及与在JavaScript生态系统中处理的数据结构是原生JS的其他库一起工作的能力。这与Brendan Eich 在[2010年左右推出代理](https://www.youtube.com/watch?v=sClk6aB_CPk)时的目标相同。Immer是一个可以不引人注意地扩展语言的精彩用例！

### 调试

Immer的高级[修补程序](https://github.com/mweststrate/immer#patches)功能允许进行细粒度的调试和跟踪，甚至可能在其上构建开发人员工具。

这与[React对可调试性的关注](https://reactjs.org/docs/design-principles.html#debugging)非常相似，其中包括允许将错误的UI更新跟踪到源，并在React的这些保证之上构建像[React DevTools](https://www.netlify.com/blog/2018/08/29/using-the-react-devtools-profiler-to-diagnose-react-app-performance-issues/)这样的大型devtools 。

而且Patches还允许在非正式下[实现类似Redux的撤销/重做](https://redux.js.org/recipes/implementingundohistory)。请查看代码的链接资源，因为完整示例太长，无法列在此处。

## 保持生命力
在JavaScript生态系统中，没落是一种常见的现象，
**识别具有持久生命力的技术**的**能力**是可持续增长的关键。
[我们的CEO Mathias Biilmann为此写了三个教训：](https://medium.com/netlify/leveling-up-why-developers-need-to-be-able-to-identify-technologies-with-staying-power-and-how-to-9aa74878fc08)：

* 了解你的历史
* 人，文化和社区都很重要
* 始终了解“为什么”

我向你坦白的是，当你阅读这篇文章时，我一直在慢慢地通过这个心理框架。我认为这是一种评估技术的好方法，也解释了为什么一些开源项目在某些社区得到了更多的采用。

Immer的迅速崛起是值得注意的，但它并非没有理由。从历史中学习，并且已经在React社区中获得了很多的追随者，如果其基本理念不正确，这一切都不可能实现。

Dan Abramov最近还指出了这些技术的演变周期以及人们如何成功打破范式：

> 成功秘诀：采取易于调试的方式，并减少编写的烦恼。感谢@mweststrate！- 丹阿布拉莫夫

我觉得这是一个深刻的见解，如果现有技术还没有建立核心开发人员体验不可变性的好处，那么Immer是不可能的，使剩下的问题成为漏洞的API。因此，Immer专注于保持相同的好处，同时以与React成功相同的方式改进API。

今天逐步尝试Immer的最佳方法是减少[Redux](https://github.com/mweststrate/immer#reducer-example)或[React setState](https://github.com/mweststrate/immer#reactsetstate-example)样板。在将来，寻找更多Immer驱动的库，如备受期待的[redux-starter-kit](https://github.com/markerikson/redux-starter-kit)项目以及非Redux状态管理解决方案，[react-copy-write](https://github.com/aweary/react-copy-write)，[immer-wieder](https://github.com/drcmda/immer-wieder#readme)和[bey](https://github.com/jamiebuilds/bey)，用于构建快速免费的样板应用！
