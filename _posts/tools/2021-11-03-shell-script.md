---
title: 【Shell笔记】（二）shell 脚本及工具
layout: post
categories: tools
tags: shell bash zsh
---

很多情况下我们需要执行一系列的操作并使用条件或循环这样的控制流，shell脚本就是实现这些操作的一种更加复杂度的工具。

大多数shell都有自己的一套脚本语言，包括变量、控制流和自己的语法。shell脚本与其他脚本语言不同之处在于，shell脚本针对shell所从事的相关工作进行来优化。因此，创建命令流程（pipelines）、将结果保存到文件、从标准输入中读取输入，这些都是shell脚本中的原生操作，这让它比通用的脚本语言更易用。



## 一、 变量

- 变量赋值命令：`=`，等号左右不能有空格
- 变量解析命令：`$变量名`，访问变量中存储的值

> 注意：等号两边有空格是不能正确工作的。以`foo = bar`为例，解释器会调用程序`foo` 并将 `=` 和 `bar`作为参数。 
>
> 总的来说，**在shell脚本中使用空格会起到分割参数的作用**，有时候可能会造成混淆，请务必多加检查。

- 定义字符串：`""`或`''`

  - `""`：定义的字符串会将变量值进行替换
  - `''`：定义的字符串为原义字符串，其中的变量不会被转义

  > 区别见 1.2

## 二、 流程控制

和其他大多数的编程语言一样，`bash`也支持`if`, `case`, `while` 和 `for` 这些控制流关键字。

> 和 Java、PHP 等语言不一样，sh 的流程控制不可为空，即若 else 分支没有语句执行，就不要写这个 else。

### 2.1 条件语句

- `if-fi`

  语法格式：

  ```shell
  if condition
  then
      command1 
      command2
      ...
      commandN 
  fi
  ```

  写成一行（适用于终端命令提示符）：

  ```zsh
  if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
  ```

- If-else-fi

  语法格式：

  ```shell
  if condition
  then
      command1 
      command2
      ...
      commandN
  else
      command
  fi	
  ```

- If-elif-else-fi

  ```shell
  if condition1
  then
      command1
  elif condition2 
  then 
      command2
  else
      commandN
  fi
  ```

例子：

```shell
# 例1
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi

# 例2——if 语句经常和 test 语句结合使用
num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi
```

## 2.2 循环语句

