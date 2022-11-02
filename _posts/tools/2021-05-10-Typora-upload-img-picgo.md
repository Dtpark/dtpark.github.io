---
layout: post
title: Typora图片上传配置——picgo篇
categories: tools
tags: tools
---

## 一、背景

众所周知，markdown作为一个轻量级标记语言在文档书写方面非常好用，可以说是程序员必备。但是其也存在一定的缺点，比如在文档中插入本地图片后，图片在本地能够正常显示，但将文档分享给其他人图片就不能显示。针对这个问题，各家markdown书写工具都提供本地图片上传功能，只需要经过简单的配置即可将图片上传到图床，这样再将文档分享出去后，图片链接变为公网链接，其他人就能正常访问。

笔者常用的markdown书写工具为**Typora**，图片上传工具为**picgo app**，近日将Typora升级后上传图片一直失败，错误信息如下：

```bash
[PicGo ERROR]: Error: API v1 is deprecated, please refer to https://doc.sm.ms/ for v2 API documentation.
```
查阅文档：

```bash
It is caused by PicGo’s support issue of its default image hosting service: sm.ms, please refer PicGo/PicGo-Core#30, or use other image service other than the default one
```

得知图片托管服务器有问题。

排查无果尝试对picgo进行升级，升级后图片可以正常上传。

正当以为完事的时候，发现在Typora粘贴剪贴板图片后，每次都会提示**前序上传任务未完成，请稍后再试**，以为上传失败了，其实在提示这个信息后图片都会接着上传。这就导致了文档里的图片路径不会被自动替换为图床链接，每次都要手动替换，体验极差。

为了实现图片路径由本地到图床的无感替换，试着将Typora的图片上传工具由**picgo app** 替换为**picgo core(command line)**。

## 二、软件版本

Typora：beta 0.10.8 (5313)

picgo-core: 1.4.19

操作系统：MacOS 10.15

## 三、配置步骤

> 以下步骤大部分来源于官方手册:
>
> https://support.typora.io/Upload-Image/
>
> https://picgo.github.io/PicGo-Core-Doc/zh/guide/config.html

1. 安装picgo-core

```bash
npm install picgo -g
# 或者
yarn global add picgo
```

2. 配置picgo配置文件（用于设置默认图床等）

在命令行输入`picgo set uploader`，通过键盘方向键和回车选择图床。笔者选择的是smms

```bash
picgo set uploader
? Choose a(n) uploader (Use arrow keys)
❯ smms 
tcyun 
github 
qiniu 
imgur 
aliyun 
upyun 
```

继续回车会要求输入相应图床的配置，如smms要求输入api token
    
    ```bash
    picgo set uploader
    ? Choose a(n) uploader smms
    ? api token (此处为sm.ms图传登录后后台获取的token字符串) 
    ```

配置完成后会出现：
    
    ```bash
    [PicGo SUCCESS]: Configure config successfully!
    ```

配置成功。

3. 获取picgo-core 和 npm的位置，供后续使用

在命令行输入`which picgo`获得picgo路径

```bash
which picgo
/usr/local/bin/picgo
```

在命令行输入 `which node`获取node路径

```bash
which node          
/usr/local/bin/node
```

4. 设置Typora

打开Typora-偏好设置…-图像，将上传服务改为`Custom Command`,命令改为`node 路径+空格+picgo路径+u`，测试上传成功

![image-20210508174243879](https://i.loli.net/2021/05/08/zIHPMh7dxs5aqDQ.png)

> 【注意】
>
> 不加node的路径会报错
>
> 不加u也会报错