---
layout: post
title: Fuzzing： The Art, Science, and Engineering of Fuzzing: A Survey 笔记
categories: paper
tags: fuzzing
---

## 0x01 简介

- 出处（期刊）：
  - [IEEE Transactions on Software Engineering](https://ieeexplore.ieee.org/xpl/RecentIssue.jsp?punumber=32) ( Volume: 47, [Issue: 11](https://ieeexplore.ieee.org/xpl/tocresult.jsp?isnumber=9611545), Nov. 1 2021)
  - CCF A
- 作者：
  - Valentin J.M. Man`es
  - HyungSeok Han
  - Sang Kil Cha
  - Manuel Egele
  - Edward J. Schwartz
  - Maverick Woo
- 关键词：
  - software security
  - automated software testing
  - fuzzing

## 0x02 摘要

- 漏洞挖掘技术中，fuzzing（fuzz testing，模糊测试）因概念简单、部署容易，且能发掘出漏洞，故仍然非常流行
- fuzzing 指的是使用生成的输入重复运行程序的过程
- 改善 fuzzing 工作的激增使得人们很难对 fuzzing 有一个全面和连贯的看法
- 本文提出了一个统一、通用的 fuzzing 模型
- 本文对当前 fuzzing 文献进行了分类

## 0x03 结论

- 本文目标是提炼出对现代 fuzzing 文献全面而连贯的看法
- 提出了一个通用的现代 fuzzer（模糊器）来解释当前各种类型的 fuzzing
- 使用图 1 和 表 1 表示了 fuzzer 的丰富分类
- 作者通过探讨设计决策来探索提出的 fuzzer 模型的每个阶段
- 作者希望为未来的工作带来更多一致性，特别是 fuzzing 术语和算法方面

## 0x04 引言

- fuzzing 自 上世纪 90 年代被提出以来就广泛应用于软件测试
- fuzzing 的宏观概念（同 0x02 中第 2 点）
- 实践中攻防双方都广泛应用
- 社区非常有活力，论文、博客很多
- 各个 fuzzer 中很多概念不统一，使得 fuzzing 的知识很难传播发展
- 