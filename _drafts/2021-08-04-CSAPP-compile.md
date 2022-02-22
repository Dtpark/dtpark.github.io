---
title: CSAPP 笔记（一）——
layout: post
categories: program
tags: csapp c 
---

## 一、源程序

```c
// hello.c
#include <stdio.h>

int main() {
	printf("hello, world\n");
	return 0;
}
```

```bash
linux> gcc -o hello hello.c
```

![c编译链接流程](https://i.loli.net/2021/10/19/om6OvfP3dY9gUIA.png)

- **预处理阶段。**预处理器 (CPP)根据以字符`#`开头的命令,修改原始的 C 程序。比如 **hello.c** 中第 1 行的 `#include < stdio.h>` 命令告诉预处理器读取系统头文件 **stdio.h** 的内容，并把它直接插入程序文本中。结果就得到了另一个 C 程序,通常是以 **.i** 作为文件扩展名。

- **编译阶段。**编译器(ccl)将文本文件 **hello.i** 翻译成文本文件 **hello.s**，它包含一个汇编语言程序。该程序包含函数 `main` 的定义,如下所示:

  ```assembly
  main:
  	subq	$8, %rsp
  	movl	$.LCO, %edi
  	call	puts
  	movl	$0, %eax
  	addq	$8, %rsp
  ```

  定义中 2 7 行的每条语句都以一种文本格式描述了一条低级机器语言指令。汇编语言是非常有用的,因为它为不同高级语言的不同编译器提供了通用的输出语言。例如,C 编译器和 Fortran 编译器产生的输出文件用的都是一样的汇编语言。

- **汇编阶段。**接下来,汇编器(as)将 **hello.s** 翻译成机器语言指令，把这些指令打包成一种叫做**可重定位目标程序 (relocatable object program)**的格式,并将结果保存在目标文件 **hello.o** 中。**hello.o** 文件是一个二进制文件，它包含的 17 个字节是函数 `main` 的指令编码。如果我们在文本编辑器中打开 **hello.o** 文件，将看到一堆乱码。
- **链接阶段。**请注意 hello 程序调用了printf 函数,它是每个 C 编译器都提供的标准 C 库中的一个函数。printf 函数存在于一个名为 printf.o 的单独的预编译好了的目标文件中,而这个文件必须以某种方式合并到我们的 hello.o 程序中。链接器(Id)就负责处理这种合并。结果就得到 hello 文件,它是一个可执行目标文件(或者简称为可执行文件), 可以被加载到内存中,由系统执行。

了解编译系统工作的好处

- 优化程序性能
- 理解链接时出现的错误
- 避免安全漏洞

处理器读并解释储存在内存中的指令

```bash
linux> ./hello
hello, world
linux> 
```

shell 是一个命令行解释器,它输出一个提示符,等待输人一个命令行,然后执行这个命令。如果该命令行的第一个单词不是一个内置的 shell 命令,那么 shell 就会假设这是一个可执行文件的名字,它将加载并运行这个文件。所以在此例中,shell 将加载并运行hello 程序,然后等待程序终止。hello 程序在屏幕上输出它的消息,然后终止。shell 随后输出一个提示符,等待下一个输人的命令行。



