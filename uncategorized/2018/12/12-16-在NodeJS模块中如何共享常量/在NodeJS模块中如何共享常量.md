---
layout:     post
title:      【译】在NodeJS模块中如何共享常量
subtitle:   NodeJS模块 & 共享常量
date:       2018-12-16
author:     Lvsi
header-img: 
catalog: true
tags:
    - NodeJS
---

# 【译】在NodeJS模块中如何共享常

> 原文 [《How do you share constants in NodeJS modules?》](https://stackoverflow.com/questions/8595509/how-do-you-share-constants-in-nodejs-modules)<br/>
> 译者：[Lvsi](https://github.com/Lvsi-China)

## 问题介绍

目前我是这样做的：

foo.js

```
const FOO = 5;

module.exports = {
    FOO: FOO
};
```

在```bar.js```中使用

<b></b>
```
var foo = require('foo');
foo.FOO; // 5
```

在exports对象中这样声明常量感觉很别扭，请问还有更好的方法吗？

## 回答

### 解答1

在我看来，使用 ```Object.freeze``` 允许 DRYer 和更具说明性的风格。我的首选模式是：

./lib/constants.js
```
module.exports = Object.freeze({
    MY_CONSTANT: 'some value',
    ANOTHER_CONSTANT: 'another value'
});
```

./lib/some-module.js
```
var constants = require('./constants');

console.log(constants.MY_CONSTANT); // 'some value'

constants.MY_CONSTANT = 'some other value';

console.log(constants.MY_CONSTANT); // 'some value'
```

