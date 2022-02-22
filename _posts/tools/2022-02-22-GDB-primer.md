---
title: GDB 的基本使用
layout: post
categories: tools
tags: 调试 GDB
---

## 0x01、基本概念

GDB（GUN debugger），是一种调试和执行程序的程序。

## 0x02、基本使用

- 准备工作

  要使用gdb调试，需要使用-g选项编译用C或C++编写的目标程序。请注意，您不应使用-01和-02等优化选项。

  ```bash
  cc -g -o filename filename.c # Linux 下 cc 指向 gcc
  # 或者
  gcc -g filename.c
  ```

  

- 启动（两种方式）

  ```bash
  # 方式 1：直接输入文件名
  gdb <filename>
  
  # 方式2：进入 gdb 后再输入待调试文件名
  gdb
  GNU gdb (Ubuntu 9.2-0ubuntu1~20.04.1) 9.2
  Copyright (C) 2020 Free Software Foundation, Inc.
  License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
  This is free software: you are free to change and redistribute it.
  There is NO WARRANTY, to the extent permitted by law.
  Type "show copying" and "show warranty" for details.
  This GDB was configured as "x86_64-linux-gnu".
  Type "show configuration" for configuration details.
  For bug reporting instructions, please see:
  <http://www.gnu.org/software/gdb/bugs/>.
  Find the GDB manual and other documentation resources online at:
      <http://www.gnu.org/software/gdb/documentation/>.
  
  For help, type "help".
  Type "apropos word" to search for commands related to "word".
  (gdb) file filename # 此处 filename 即为待调试文件名
  ```

- 执行控制

  | 命令                        | 简写     | 作用                                                         |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | run [args]                  | r [args] | 开始执行程序。如果设置了断点，当达到断点时，执行将停止；否则，程序将运行至完成。gdb在终止时打印一条消息，说明程序的状态 |
  | continue                    | c        | 从停止处开始继续执行程序                                     |
  | step [N]                    | s [N]    | 单步调试。若有函数调用，此命令会进入函数内部。参数 N 表示执行 N 次 s |
  | next [N]                    | n [N]    | 单步跟踪。若有函数调用，不进入函数内部而直接调用。参数 N 表示执行 N 次 n |
  | until                       | -        | 在循环体内时，直接运行程序到退出循环体                       |
  | until [line]                | -        | 运行至某行，不仅仅用来跳出循环                               |
  | call \<function name(args)> | -        | 调用程序中可见的函数并传递参数 args                          |
  | finish                      | -        | 执行到当前函数到 return                                      |

- 断点（break point）管理

  | 命令       | 简写 | 作用                                        |
  | ---------- | ---- | ------------------------------------------- |
  | break \<line>            | b \<line>          | 在指定行号设置中断（执行到目标行停止执行，即不执行目标行）   |
  | break \<filename: line>  | b \<filename:line> | 同上                                                         |
  | break \<function name>   | b \<function name> | 在制定函数出设置中断（调用函数时停止执行）                   |
  | delete [n] | -    | 删除所有断点。若有参数 n，则删除第 n 号断点 |
  | clear \<line> | - | 删除指定行前的断点 |
  | clear \<function name> | - | 删除指定函数前的断点 |
  | info b | - | 显示当前程序的断点情况 |

- 查看程序源码

  | 命令                  | 简写               | 作用                                                         |
  | --------------------- | ------------------ | ------------------------------------------------------------ |
  | list                  | l                  | 列出程序的源码。默认显示 10 行，再按一次则继续显示下边的 10 行 |
  | list -                | l -                | 打印上次打印前的 10 行                                       |
  | list \<line>          | l \<line>          | 显示当前文件以行号为中心的前后 10 行代码                     |
  | list \<function name> | l \<function name> | 显示指定函数的源码                                           |

- 打印

  | 命令                  | 简写            | 作用                                                         |
  | --------------------- | --------------- | ------------------------------------------------------------ |
  | print \<expression>   | p \<expression> | 打印表达式的值。表达式可以为 变量、指针、数组、表达式、函数等 |
  | display \<expression> | -               | 每次程序执行后打印表达式的值。单步调试时很有用，如跟踪某变量的变化 |
  | delete display        | -               | 取消 display 设置的表达式                                    |
  | watch \<expression>   | -               | 设置一个监视点。一旦被监视的表达式值改变，强行终止正在被调试的程序 |

- 查看运行时信息

  | 命令            | 简写 | 作用                                     |
  | --------------- | ---- | ---------------------------------------- |
  | where/bt        | -    | 查看当前运行的堆栈列表                   |
  | frame \<number> | -    | 选择并打印一个帧栈                       |
  | up              | -    | 选择并打印一个调用此（函数）的帧栈       |
  | down            | -    | 选择并打印一个此函数调用的帧栈           |
  | info program    | -    | 查看程序是否在运行、进程号、被暂停原因等 |

- 其他命令

  | 命令                    | 简写 | 作用                                                 |
  | ----------------------- | ---- | ---------------------------------------------------- |
  | quite                   | q    | 退出 GDB                                             |
  | help [category/command] | -    | 查询手册                                             |
  | kill                    | k    | 停止正在运行的程序。通常用于从头开始准备重新启动程序 |
  | return                  | -    | 略                                                   |

## 0x03 参考资料

- https://people.cs.pitt.edu/~mosse/gdb-note.html
- https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/gdb.html