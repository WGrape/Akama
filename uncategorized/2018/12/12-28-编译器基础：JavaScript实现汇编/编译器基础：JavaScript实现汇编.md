---
layout:     post
title:      【译】编译器基础：JavaScript实现汇编
subtitle:   编译器基础 & JavaScript实现汇编
date:       2018-12-28
author:     Lvsi
header-img: 
catalog: true
tags:
    - 汇编
    - 编译器
---

# 【译】编译器基础：JavaScript实现汇编

> 原文 [《Compiler basics: lisp to assembly》](http://notes.eatonphil.com/compiler-basics-lisp-to-assembly.html)<br/>
> 译者：[Lvsi](https://github.com/Lvsi-China)

在这篇文章中，我们将在Javascript(NodeJS)编写一个简单的编译器，而不需要任何第三方库。我们的目标是采用```(+ 1 (+ 2 3))```类似的输入程序生成并输出汇编程序，该程序执行这些操作并输出6后退出。最后的编译器可在[这里](https://github.com/eatonphil/ulisp)看到。

我们将涵盖：

* 解析
* 代码生成
* 汇编基础
* 系统调用

现在我们将省略：

* 可编程函数定义
* 非符号/非数字数据类型
* 超过3个的函数参数
* 很多安全
* 很多错误消息

## 解析

我们选择前面提到的[S-表达式](https://en.wikipedia.org/wiki/S-expression)语法，因为它很容易解析。此外，输入的语言非常有限，我们甚至不会将解析器分解为 lexing / parsing 阶段。

> 一旦需要支持字符串，注释，小数和其他更复杂的文字，就可以更容易地使用各个阶段。 
>
> 如果你对这些解析阶段感到好奇，你可能会对我写的关于[编写JSON解析器](http://notes.eatonphil.com/writing-a-simple-json-parser.html)的帖子感兴趣。 
>
> 或者，查看我的BSDScheme项目， 获取Scheme的全功能 [词法分析器](https://github.com/eatonphil/bsdscheme/blob/master/src/lex.d) 和 解析器。

解析器应该生成一个抽象语法树（AST），一个表示输入程序的数据结构。具体来说，在JavaScript中，我们用 ```(+ 1 (+ 2 3))``` 生成 ['+', 1, ['+', 2, 3]]。

解析有许多不同的方法，但对我来说最直观的是有一个接受程序（字符串）的函数并返回一个包含到目前为止解析的程序（AST）和程序，其余部分的元组（a字符串）尚未解析。

函数的骨架如下所示：

```js
module.exports.parse = function parse(program) {
  const tokens = [];

  ... logic to be added ...

  return [tokens, ''];
};
```

因此，最初调用parse的代码必须处理展开最外层元组才能到达AST。对于更有用的编译器，如果返回结果的第二个元素不是空字符串，我们可以通过失败检查整个程序是否实际被解析。

在函数中，我们将迭代每个字符并累积起来，直到我们命中空格，左括号或右括号：

```js
module.exports.parse = function parse(program) {
  const tokens = [];
  let currentToken = '';

  for (let i = 0; i < program.length; i++) {
    const char = program.charAt(i);

    switch (char) {
      case '(': // TODO
        break;
      case ')': // TODO
        break;
      case ' ':
        tokens.push(+currentToken || currentToken);
        currentToken = '';
        break;
      default:
        currentToken += char;
        break;
    }
  }

  return [tokens, ''];
};
```

递归部分始终是最具挑战性的。右括号是最简单的。我们必须push当前的token并返回所有token：

```js
module.exports.parse = function parse(program) {
  const tokens = [];
  let currentToken = '';

  for (let i = 0; i < program.length; i++) {
    const char = program.charAt(i);

    switch (char) {
      case '(': // TODO
        break;
      case ')':
        tokens.push(+currentToken || currentToken);
        return [tokens, program.substring(i + 1)];
      case ' ':
        tokens.push(+currentToken || currentToken);
        currentToken = '';
        break;
      default:
        currentToken += char;
        break;
    }
  }

  return [tokens, ''];
};
```

最后左边的括号应该递归，将解析后的标记添加到兄弟标记列表中，并强制循环从新的未解析点开始。

```js
module.exports.parse = function parse(program) {
  const tokens = [];
  let currentToken = '';

  for (let i = 0; i < program.length; i++) {
    const char = program.charAt(i);

    switch (char) {
      case '(': {
        const [parsed, rest] = parse(program.substring(i + 1));
        tokens.push(parsed);
        program = rest;
        i = 0;
        break;
      }
      case ')':
        tokens.push(+currentToken || currentToken);
        return [tokens, program.substring(i + 1)];
      case ' ':
        tokens.push(+currentToken || currentToken);
        currentToken = '';
        break;
      default:
        currentToken += char;
        break;
    }
  }

  return [tokens, ''];
};
```

假设这些都在parser.js中，我们在REPL中尝试：

```bash
$ node
> const { parse } = require('./parser');
undefined
> console.log(JSON.stringify(parse('(+ 3 (+ 1 2)')));
[[["+",3,["+",1,2]]],""]
```
很好！我们继续。

## 汇编 101
本质上，汇编是我们可以使用的最低级编程语言。它是人类可读的，1：1表示的，CPU可以解释的二进制指令。使用汇编程序完成从汇编到二进制的转换，反向步骤是用反汇编程序完成的。我们使用GCC编译，因为它处理macOS上汇编编程的一些[奇怪问题])(http://fabiensanglard.net/macosxassembly/index.php)。

汇编中的主要数据结构是寄存器（由CPU存储的临时变量）和程序堆栈。程序中的每个函数都可以访问相同的寄存器，但是每个函数都有各自独立的堆栈，因此它比寄存器更耐用。RAX，RDI，RDX，和 RSI是提供给我们使用的一些寄存器。

现在我们只需要知道一些指令来编译我们的程序（其余的汇编编程是惯例）：

* MOV: 将一个寄存器中的值存储到另一个寄存器中，或将一个值存储到寄存器中
* ADD: 将两个寄存器中的值的总存储在第一个寄存器中
* PUSH: 将寄存器中的值存储到堆栈中
* POP: 从堆栈中删除最顶层的值并存储在寄存器中
* CALL: 进入堆栈中的新部分并开始运行该函数
* RET: 进入函数调用堆栈，并在调用结束后，回到调用的下一条指令处。
* SYSCALL: 像CALL一样，但是此函数是由内核处理的

## 函数调用规则
汇编指令足够灵活，没有语言定义的方式来进行函数调用。因此，回答（至少）以下几个问题非常重要：

* 调用者传递的参数存储在哪里，以便被调用者可以访问它们？
* 被调用者的返回值存储在哪里，以便调用者可以访问它？
* 什么寄存器被谁存储？

在不深入细节的情况下，我们假设在x86_64 macOS和Linux系统上开发的答案如下：

* 参数存储（按顺序）在RDI，RSI和RDX寄存器
  * 我们不会支持传递三个以上的参数
* 返回值存储在RAX寄存器中
* RDI，RSI和RDX寄存器被调用者存储

## 代码生成
使用汇编基础知识和函数调用规则，已经足够我们从程序解析的AST中生成代码。

我们的编译代码的框架如下所示：

```js
function emit(depth, code) {
  const indent = new Array(depth + 1).map(() => '').join('  ');
  console.log(indent + code);
}

function compile_argument(arg, destination) {
  // If arg AST is a list, call compile_call on it

  // Else must be a literal number, store in destination register
}

function compile_call(fun, args, destination) {
  // Save param registers to the stack

  // Compile arguments and store in param registers

  // Call function

  // Restore param registers from the stack

  // Move result into destination if provided
}

function emit_prefix() {
  // Assembly prefix
}

function emit_postfix() {
  // Assembly postfix
}

module.exports.compile = function parse(ast) {
  emit_prefix();
  compile_call(ast[0], ast.slice(1));
  emit_postfix();
};
```

从我们在注释中的伪代码来看，它很容易填写。下面我们填写除前缀和后缀代码之外的所有内容。

```js
function compile_argument(arg, destination) {
  // If arg AST is a list, call compile_call on it
  if (Array.isArray(arg)) {
    compile_call(arg[0], arg.slice(1), destination);
    return;
  }

  // Else must be a literal number, store in destination register
  emit(1, `MOV ${destination}, ${arg}`);
}

const BUILTIN_FUNCTIONS = { '+': 'plus' };
const PARAM_REGISTERS = ['RDI', 'RSI', 'RDX'];

function compile_call(fun, args, destination) {
  // Save param registers to the stack
  args.forEach((_, i) => emit(1, `PUSH ${PARAM_REGISTERS[i]}`));

  // Compile arguments and store in param registers
  args.forEach((arg, i) => compile_argument(arg, PARAM_REGISTERS[i]));

  // Call function
  emit(1, `CALL ${BUILTIN_FUNCTIONS[fun] || fun}`);

  // Restore param registers from the stack
  args.forEach((_, i) => emit(1, `POP ${PARAM_REGISTERS[args.length - i - 1]}`));

  // Move result into destination if provided
  if (destination) {
    emit(1, `MOV ${destination}, RAX`);
  }

  emit(0, ''); // For nice formatting
}
```

在更好的编译器中，我们不会创建plus内置函数，只是会为汇编指令ADD发出代码。但是，创建一个plus函数可以使代码生成更简单，并且还允许我们查看函数调用的样子。

我们将在前缀代码中定义plus内置函数。

## 前缀
汇编程序由内存中的几个段(Section)组成，其中最重要的是文本段(Text Section)和文本段(Data Section)。文本段是只读的，其中存储了程序指令本身。指示CPU从此文本段中的某个位置开始解释，通过指令递增以执行每条指令，直到它到达一条告诉它跳转到另一个位置来执行指令的指令（例如，使用CALL，RET或JMP）。

.text在我们发出生成的代码之前，表示我们在前缀中发出的文本部分。

> 数据部分用于静态初始化值（例如全局变量）。我们现在没有任何需要，所以我们会忽略它。

另外，我们需要发出一个入口点（我们将使用_main），并添加一个notice（.global _main），以便该入口点的位置在外部可见。这很重要，因为我们使用GCC处理生成可执行文件的部分，它需要访问入口点。


到目前为止，我们的前缀如下所示：

```js
function emit_prefix() {
  emit(1, '.global _main\n');

  emit(1, '.text\n');

  // TODO: add built-in functions

  emit(0, '_main:');
}
```

我们前缀的最后一部分需要包含plus内置函数。为此，我们添加了我们商定的前两个参数寄存器（RDI和RSI），并将结果存储在RA中。

```js
function emit_prefix() {
  emit(1, '.global _main\n');

  emit(1, '.text\n');

  emit(0, 'plus:');
  emit(1, 'ADD RDI, RSI');
  emit(1, 'MOV RAX, RDI');
  emit(1, 'RET\n');

  emit(0, '_main:');
}
```

## 后缀
后缀的工作将很简单，使用RAX的值调用exit，因为RAX的值是程序调用的最后一个函数的结果。

exit是一个系统调用，所以我们将使用SYSCALL指令来调用它。macOS和Linux上的x86_64调用约定SYSCALL 以相同的方式定义参数CALL。但我们还需要告诉 SYSCALL要调用的系统调用。约定的是设置RAX为整数表示当前系统上的系统调用。在Linux上它是60，在macOS上它是0x2000001。

> 当我说“约定”时，我并不是说你真的要有作为程序员的选择。当操作系统和标准库选择它时，这是任意的。但是如果你想编写一个使用系统调用或比如调glibc的工作程序，你需要遵循这些约定。

后缀看起来像这样：

```js
const os = require('os');

const SYSCALL_MAP = os.platform() === 'darwin' ? {
    'exit': '0x2000001',
} : {
    'exit': 60,
};

function emit_postfix() {
  emit(1, 'MOV RDI, RAX'); // Set exit arg
  emit(1, `MOV RAX, ${SYSCALL_MAP['exit']}`); // Set syscall number
  emit(1, 'SYSCALL');
}
```

## 把它们合并在一起

最后我们可以编写Javascript的入口点，并针对示例程序运行我们的编译器。

入口点可能如下所示：
```js
const { parse } = require('./parser');
const { compile } = require('./compiler');

function main(args) {
  const script = args[2];
  const [ast] = parse(script);
  compile(ast[0]);
}

main(process.argv);
```

我们可以如下调用。

```assembly
$ node ulisp.js '(+ 3 (+ 2 1))'
  .global _main

  .text

plus:
  ADD RDI, RSI
  MOV RAX, RDI
  RET

_main:
  PUSH RDI
  PUSH RSI
  MOV RDI, 3
  PUSH RDI
  PUSH RSI
  MOV RDI, 2
  MOV RSI, 1
  CALL plus
  POP RSI
  POP RDI
  MOV RSI, RAX

  CALL plus
  POP RSI
  POP RDI

  MOV RDI, RAX
  MOV RAX, 0x2000001
  SYSCALL
```  
## 从输出中生成可执行文件

如果我们将前一个输出重定向到一个汇编文件并调用GCC，就可以生成一个可以运行的程序了。然后我们可以输出变量以查看前一个进程的退出代码(输出结果)。

```bash
$ node ulisp.js '(+ 3 (+ 2 1))' > program.S
$ gcc -mstackrealign -masm=intel -o program program.s
$ ./program
$ echo $?
6
```

这样我们得到了一个正常工作的编译器！编译器的全部源代码，请点击[这里](https://github.com/eatonphil/ulisp)。

## 进一步阅读
* [x86_64 调用规则](https://aaronbloomfield.github.io/pdr/book/x86-64bit-ccc-chapter.pdf)
* macOS 汇编编程
    * [在macOS上堆栈对齐](http://fabiensanglard.net/macosxassembly/index.php)
    * [在macOS上的Syscalls](https://filippo.io/making-system-calls-from-assembly-in-mac-os-x/)
* 目标驱动的代码生成
    * [Kent Dybvig的初始论文](https://www.cs.indiana.edu/~dyb/pubs/ddcg.pdf)
    * [V8中的一趟代码生成](http://cs.au.dk/~mis/dOvs/slides/46b-codegeneration-in-V8.pdf)

---

Find me on [Github](https://github.com/eatonphil).