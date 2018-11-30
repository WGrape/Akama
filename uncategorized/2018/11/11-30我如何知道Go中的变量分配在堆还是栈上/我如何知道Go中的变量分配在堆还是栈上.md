---
layout:     post
title:      【译】我如何知道Go中的变量分配在堆还是栈上
subtitle:   Go & 堆 & 栈
date:       2018-11-30
author:     Lvsi
header-img: 
catalog: true
tags:
    - Go堆栈
---

## 我如何知道Go中的变量分配在堆还是栈上

> 翻译文章来自 [《How do I know whether a variable is allocated on the heap or the stack?》](https://golang.org/doc/faq#stack_or_heap)<br/>
> 补充翻译：[原文翻译](http://docscn.studygolang.com/doc/faq#%E5%A0%86%E6%88%96%E6%A0%88)<br/>
> 译者：[Lvsi](https://github.com/Lvsi-China)

从正确的观点上分析，你不必知道这个。对于Go中的所有变量而言，只要有对它的引用，它就是存在的。选择的存储位置和语言的语义是无关的。

存储位置对于编写高效的程序来说并无影响，如果可能，Go编译器将在该函数的堆栈帧中分配局部变量。但是，如果编译器在函数返回后无法证明变量不再被引用，则编译器必须在垃圾回收的堆上分配变量以避免悬空指针错误。此外，如果局部变量非常大，将它存储在堆而不是堆栈上可能更有意义。

在当前的编译器中，若一个变量的地址已被占用，则该变量是堆上分配的候选变量。但是，当这样的变量在函数返回后不再被引用时，它们就会存储在栈上，一个基本的逃逸分析就能识别出此情况。
