---
layout: post
title: Fuzzing： Docker 中 执行 'echo core > /proc/sys/kernel/core_pattern' 报错
categories: develop
tags: fuzzing AFL Docker
---


## 一、背景

前文中，课题组要做AFL的web端可视化展示，需要用Docker封装AFL为WEB API。在改进脚本重新部署后，调用AFL进行fuzzing时，AFL启动失败。因为在执行`afl-fuzz`前，如果系统配置为将核心转储文件（core）通知发送到外部程序，将导致将崩溃信息发送到Fuzzer之间的延迟增大，进而可能将崩溃被误报为超时。所以得临时修改`core_pattern`文件，如下所示：

```bash
echo core > /proc/sys/kernel/core_pattern
```

但是在Docker容器中执行上述命令时报错：

```bash
bash: /proc/sys/kernel/core_pattern: Read-only file system
```

> 大意：core_pattern 为系统只读文件，不能修改

## 二、疑惑

Docker执行时内部是以root用户执行各种命令，为啥更改不了只读权限的文件呢？带着这个疑问百度了一遍发现也没查出个所以然，就查了一个机翻的帖子也模棱两可的。

既然百度不行，那就Google吧。查了两分钟查到个海峡对岸同胞的博客，找到了解决方案。

## 三、解决方案

直接上代码：

```bash
# 构建容器时加入 --privileged 参数
docker run -idt -p xx:xx --privileged afl-api:0.0.3
```

如上述命令，在构建容器时多加一个 `--privileged`参数即可解决问题。

## 四、原因

> 大约在0.6版，privileged被引入docker。
> 使用该参数，container内的root拥有真正的root权限。
> 否则，container内的root只是外部的一个普通用户权限。
> privileged启动的容器，可以看到很多host上的设备，并且可以执行mount。
> 甚至允许你在docker容器中启动docker容器
>
> ——引用自：https://blog.csdn.net/wylfengyujiancheng/article/details/90576040

即加了`--privileged`参数的容器才能真正执行root权限。

## 五、参考文献
- https://ephrain.net/docker-在-container-裡設定-core-dump-的檔案名稱格式/
- https://blog.csdn.net/wylfengyujiancheng/article/details/90576040

