---
title: GitBook报错 与 node 版本管理工具“n”
layout: post
categories: tools
tags: gitbook error node
---

## 一、背景

最近在用 **Markdown** 写数据结构笔记，结果一篇笔记越写越长，博客检索起来太麻烦。想起来很久以前用 **GitBook** 做过笔记效果还不错，就准备在 Mac 上装一下，安装步骤如下：

```zsh
$ sudo npm install -g gitbook-cli
```

安装没出现什么问题，在初始化gitbook时报错：“TypeError [ERR_INVALID_ARG_TYPE]: The "data" argument must be of type string or an instance of Buffer, TypedArray, or DataView. Received an instance of Promise”，具体如下：

```zsh
$ gitbook init
warn: no summary file in this book 
info: create SUMMARY.md 

TypeError [ERR_INVALID_ARG_TYPE]: The "data" argument must be of type string or an instance of Buffer, TypedArray, or DataView. Received an instance of Promise
```

## 二、原因

百度了一下，因为 **node** 的版本过高（本机是16），导致与 gitbook 不兼容，降级到 10 就可以。搜索发现 Mac 下有 node 版本管理工具`nvm`和`n`，恰好发现本地装了 `n`，不知道是系统自带的还是啥时候装的……

> n 的开源地址：https://github.com/tj/n

## 三、node 版本管理工具 n 的简单使用

- 安装

  ```zsh
  $ sudo npm install -g n
  ```

- 查看帮助

  ```zsh
  $ n help
  ```

- 列出所有 node 版本

  ```zsh
  $ n ls
  ```

- 安装某个版本

  ```zsh
  $ n xx.xx.xx # xx 为版本号，可以只输入大版本号
  ```

- 安装最新版本

  ```zsh
  $ n lastest
  ```

- 安装稳定版

  ```zsh
  $ n stable
  ```

- 选取已安装版本

  ```zsh
  $ n # 然后上下键盘选择并回车确认
  ```

- 删除某个版本

  ```zsh
  $ n rm xx.xx.xx # xx 为版本号
  ```

- 指定版本来运行脚本

  ```zsh
  $  n use xx.xx.x a.js
  ```

## 四、解决

通过 `sudo n 10`及`sudo n`选择 node 版本为 10，完美解决问题。

## 五、参考资料

- [Mac上使用 nvm 管理不同版本的 node](https://www.jianshu.com/p/a6aca7bf4780)
- [Mac下通过n管理多个版本的node.js](https://www.jianshu.com/p/a52bb03cb279)



















