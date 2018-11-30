---
layout:     post
title:      【译】微软将开源Edge的JavaScript引擎
subtitle:   ChakraCore引擎
date:       2018-11-25
author:     Lvsi
header-img: 
catalog: true
tags:
    - JavaScript
---

## 【译】微软将开源Edge的JavaScript引擎

> 原文 [《Microsoft Edge’s JavaScript engine to go open-source
》](https://blogs.windows.com/msedgedev/2015/12/05/open-source-chakra-core/)<br/>
> 译者：[Lvsi](https://github.com/Lvsi-China)

今天在佛罗里达的[JSConf US Last Call]大会上，我们宣布将会开源Chakra的核心组件，叫做ChakraCore，包含了微软Edge浏览器的JavaScript引擎中的所有重要组件。下个月将会把ChakraCore源代码以MIT许可证发布。

<img src="/img/posts/2018/11-25/1.jpg">

Chakra 提供了最一流的JavaScript执行引擎，广泛的覆盖了ES2015特性和可靠的性能，可靠性和可扩展性。

从基于云的各种服务到物联网及其他的各个重要的方面，我们希望ChakraCore都能得以使用。

我们比以往投资了更多去改善Chakra，并很高兴与我们的社区合作，去推动进一步的发展。 除了官方之外，一些组织已经表示有兴趣为ChakraCore做出贡献，其中我们期待在发展社区的时候与Intel，AMD和NodeSource的合作。

## Chakra: 现代JavaScript引擎

2008年，我们重新创建了一个代号为Chakra的新JavaScript引擎。 我们的原则是确保Chakra具有现代Web所需要的性能特征，并且可以轻松适应各种硬件配置文件等其他潜在的新兴场景。 简而言之，这意味着Chakra需要快速启动，快速运行并且提供出色的用户体验，同时充分利用底层硬件。 Chakra通过独特的多层管道实现了这些目标，该管道支持解释器，多层JIT编译器以及可以进行并发和部分集合的传统标记-清除垃圾收集器。

<img src="/img/posts/2018/11-25/2.png">

自Chakra成立以来，JavaScript已经从主要支持Web浏览器的语言扩大到一种开始支持商店应用，服务器端应用程序，基于云的服务，NoSQL数据库，游戏引擎，前端工具以及近年的物理网的技术。慢慢的，Chakra逐渐发展到适应多种环境，并经过优化，可以为所有不同环境提供出色的体验。 这意味着除了吞吐量之外，Chakra还必须支持本机互操作性，出色的可扩展性以及在受限的资源环境中通过限制资源消耗来执行代码的能力。Chakra的解释器在跨平台架构上的轻松移植技术方面发挥了关键作用。

随着今年早些时候的Windows 10发布，Chakra不仅被优化为更快地运行Web，而且在其他浏览器供应商的提供的JavaScript基准测试上的性能提高了一倍多。

## ChakraCore 内部结构

ChakraCore是一个完全成熟的，独立的JavaScript虚拟机，可以嵌入到需要脚本性的衍生产品和电源应用程序中，如NoSQL数据库，生产力软件和游戏引擎。 ChakraCore可用于通过Node.js和基于云的服务等平台扩展服务器上JavaScript的范围。它包含解析，解释，编译和执行JavaScript代码所需的所有内容，而不依赖于Microsoft Edge内部。

ChakraCore拥有Chakra在Microsoft Edge中支持的同一组功能，但有两个主要区别。首先，它没有将Chakra的私有绑定暴露给浏览器或通用Windows平台，这两者都将它限制在一个非常具体的用例中。其次，ChakraCore不再公开Chakra目前可用的基于COM的诊断API，而是支持一组新的现代诊断API，这些API将是平台无关的，并且可以标准化或在长期内跨不同实现进行互操作。随着我们在这些新的诊断API上取得进展，我们计划在Chakra中提供它们。

<img src="/img/posts/2018/11-25/3.png">

## ChakraCore的下一步是什么？

任何现代JavaScript引擎都必须提供优于浏览器场景的性能体验，包括从物联网应用程序的小型设备到基于云技术的高吞吐量，大规模并行服务器应用程序的所有软硬件。

ChakraCore已经为适合任何需要快速，可扩展和轻量级引擎的应用程序而设计。我们打算后续在Windows生态系统内外实现更多的功能。 虽然最初的1月发布的版本仅限Windows，但我们承诺将来会把ChakraCore引入到其他平台。 我们已邀请开发人员帮助我们实现这一目标，让我们知道他们希望ChakraCore支持其他的哪些平台，以帮助我们优先考虑未来的投资方向，甚至帮助我们将其移植到他们选择的平台上。

## 为ChakraCore贡献

从1月开始，我们将开放公共GitHub仓库为社区做贡献。并且，我们将提供有关如何有效的为项目做贡献的初步事项和指导的更多细节。 社区是任何开源项目的核心，因此我们期待社区克隆仓库，检查代码，构建代码，并提供从新功能到测试或bug修复的所有内容。 我们也欢迎提供有关如何针对你或你的业务的重要的特定场景以改进ChakraCore的建议。