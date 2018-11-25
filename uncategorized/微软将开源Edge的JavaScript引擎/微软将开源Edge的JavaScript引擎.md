## Microsoft Edge’s JavaScript engine to go open-source

> 翻译文章来自 [《Microsoft Edge’s JavaScript engine to go open-source
》](https://blogs.windows.com/msedgedev/2015/12/05/open-source-chakra-core/)


今天在佛罗里达的[JSConf US Last Call]大会上，我们宣布将会开源Chakra的核心组件，叫做ChakraCore，包含了微软Edge浏览器的JavaScript引擎中的所有重要组件。下个月将会把ChakraCore源代码以MIT许可证发布。

<img src="./images/1.jpg">

Chakra 提供了最一流的JavaScript执行引擎，广泛的覆盖了ES2015特性和可靠的性能，可靠性和可扩展性。

从基于云的各种服务到物联网及其他的各个重要的方面，我们希望ChakraCore都能得以使用。

我们比以往投入了更多精力去改善Chakra，并很高兴与我们的社区合作，去推动进一步的发展。 
除了官方之外，一些组织已经表示有兴趣为ChakraCore做出贡献，其中我们期待在发展社区的时候与Intel，AMD和NodeSource的合作。

## Chakra: 现代JavaScript引擎

2008年，我们重新创建了一个代号为Chakra的新JavaScript引擎。 我们的原则是确保Chakra具有现代Web所需要的性能特征，并且可以轻松适应各种硬件配置文件等其他潜在的新兴场景。 简而言之，这意味着Chakra需要快速启动，快速运行并且提供出色的用户体验，同时充分利用底层硬件。 Chakra通过独特的多层管道实现了这些目标，该管道支持解释器，多层JIT编译器以及可以进行并发和部分集合的传统标记-清除垃圾收集器。