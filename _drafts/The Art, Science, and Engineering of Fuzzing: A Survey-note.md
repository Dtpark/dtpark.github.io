---
layout: post
title: Fuzzing： The Art, Science, and Engineering of Fuzzing: A Survey 笔记
categories: paper
tags: fuzzing

---

## 〇、简介

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

## 一、摘要

- 漏洞挖掘技术中，fuzzing（fuzz testing，模糊测试）因概念简单、部署容易，且能发掘出漏洞，故仍然非常流行
- fuzzing 指的是使用生成的输入重复运行程序的过程
- 改善 fuzzing 工作的激增使得人们很难对 fuzzing 有一个全面和连贯的看法
- 本文提出了一个统一、通用的 fuzzing 模型
- 本文对当前 fuzzing 文献进行了分类

## 二、结论

- 本文目标是提炼出对现代 fuzzing 文献全面而连贯的看法
- 提出了一个通用的现代 fuzzer（模糊器）来解释当前各种类型的 fuzzing
- 使用图 1 和 表 1 表示了 fuzzer 的丰富分类
- 作者通过探讨设计决策来探索提出的 fuzzer 模型的每个阶段
- 作者希望为未来的工作带来更多一致性，特别是 fuzzing 术语和算法方面

## 三、引言

- fuzzing 自 上世纪 90 年代被提出以来就广泛应用于软件测试
- fuzzing 的宏观概念（同 0x02 中第 2 点）
- 实践中攻防双方都广泛应用
- 社区非常有活力，论文、博客很多
- 各个 fuzzer 中很多概念不统一，使得 fuzzing 的知识很难传播发展
- 作者试图统一该领域的概念等

## 四、系统、分类及测试程序

- “fuzz”一词最初由 Miller 等人于1990年发明，指的是一个“生成目标程序消耗的随机字符流”的程序
- 自那以后，fuzzing”出现在各种背景下，包括动态符号执行、基于语法的测试用例生成、权限测试、行为测试、复杂性测试、内核测试、表示依赖性测试、函数检测、鲁棒性评估、利用开发、GUI测试、签名生成、渗透测试、嵌入式设备和神经网络测试
- 为了将大量模糊文献的知识系统化，让我们首先提出一个从现代用途中提取的模糊术语

### 4.1 定义

- **PUT**：Program Under Test【待测程序】

- **fuzzing**：Fuzzing is the execution of the PUT using input(s) sampled from an input space (the “fuzz input space”) that protrudes the expected input space of the PUT. 【测试待测程序】
- **fuzz testing**： Fuzz testing is the use of fuzzing to test if a PUT violates a correctness policy【利用 fuzzing 获取异常输入】
- **fuzzer**：A fuzzer is a program that performs fuzz testing on a PUT【执行 fuzzing 的程序】
- *Fuzz Campaign*：A fuzz campaign is a specific execution of a fuzzer on a PUT with a specific correctness policy【Fuzz 运动。在具有特定正确性策略的被测程序上执行模糊测试】

- *Bug Oracle*：A bug oracle is a program, perhaps as part of a fuzzer, that determines whether a given execution of the PUT violates a specific correct nesspolicy.【Bug 预言。确定被测程序的给定执行是否违反特定的正确策略】

- *Fuzz Configuration*：A fuzz configuration of a fuzz algorithm comprises the parameter value(s) that control(s) the fuzz  algorithm.【Fuzz 配置。控制 fuzzing 算法的参数值】

- **seed（种子）**：种子是待测程序的(通常结构良好的)输入，用于通过修改它来生成测试用例

### 4.2 论文选择范围

2008 年 1 月 - 2019 年 2 月 发布于 4 大安全顶会和 3 大软件工程顶会的论文。

- ACM Conference on Computer and Communications Security (CCS)
- IEEE Symposium on Security and Privacy (S&P)
- Networkand Distributed System Security Symposium (NDSS)
- USENIX Security Symposium (USEC)

---

- ACM International Symposium on the Founda-tions of Software Engineering (FSE)
- IEEE/ACM Inter-national Conference on Automated Software Engineering(ASE)
- International Conference on Software Engi-neering (ICSE)

### 4.3 模糊测试算法

算法 1 用于模糊测试的通用算法

![image-20220224172503131](https://s2.loli.net/2022/02/24/7yhmLMDWzOCb62q.png)

- $$ C $$：fuzz 配置

- $$ t_{limit}$$：超时时间
- $$B$$：一组有限的输出（bug）

- $$PREPROCESS(C)\rightarrow{C}$$：预处理。用户向 PREPROCESS 提供一组模糊配置作为输入，并且它返回潜在修改的一组模糊配置。如，插桩等。
- $$SCHEDULE(C, t_{elapsed}, t_{limit})\rightarrow{conf}$$：调度？将 fuzz 配置 C、当前时间、超时时间作为输入，选择一种策略（conf）作为当前 fuzz 迭代的策略。
- $$INPUTGEN(conf)\rightarrow{tcs}$$：用 fuzz 策略（conf）生成测试用例(test cases, tcs)。一些 fuzzer 用种子生成测试用例，还有一些用 *模型* 或 *语法* 生成。
- $$INPUTEVAL(conf, tcs, O_{bug})\rightarrow{B}^{'}, execinfos$$：输入评价？用测试用例执行被测程序，输出错误集 $$B^{'}$$和执行信息 $$execinfos$$ 。
- $$CONFUPDATE(C,conf,execinfos)\rightarrow{C}$$：或许会更新 fuzz 配置 C。
- $$CONTINUE(C)\rightarrow{\{True, False\}}$$：判断是否进行新的 fuzz。在白盒测试中判断是否有新路径被发现。

### 4.4 Fuzzer 分类

fuzzer 根据每次 fuzz 观察到的语意粒度将 fuzzer 分为下述三组。

- 黑盒——许多传统的 fuzzer 属于此类，一些现代的 fuzzer 如 funfuzz、peach 也属于此类；
- 白盒——符号执行、污点分析结合的 fuzzer 属于此类；
- 灰盒——可以获取部分待测程序的信息或通过运行时获取部分信息，从而提高测试速度等。

将出现在大型会议/GitHub 上超过 100 star 的 fuzzer 整理为下述关系图。

![img](https://s2.loli.net/2022/02/24/sZmqLBWzeGrbo6V.png)

### 4.5 fuzzer 谱系与概述

![img](https://s2.loli.net/2022/02/24/XEURJexnLKcPwCq.png)

> 1. fuzzer 分类
> 2. fuzzer 是否开源
> 3. 是否需要被测程序源码
> 4. 是否支持内存 fuzzing
> 5. fuzzer 是否可以推断模型？？
> 6. fuzzer 在 预处理 阶段中执行动态分析还是静态分析
> 7. 是否支持处理多个种子并进行调度
> 8. fuzzer 是否执行输入变异以生成测试用例。◑ 表示根据执行反馈指导输入变异
> 9. fuzzer 是否基于模型生成测试用例
> 10. 是否执行符号执行来生成测试用例
> 11. 是否利用污点分析来生成测试用例
> 12. 是否使用堆栈散列执行崩溃分类
> 13. 是否使用代码覆盖率执行崩溃分类
> 14. 是否在 CONFUPDATE 期间进化种子池。如向池中添加新种子
> 15. fuzzer 是否以在线方式学习输入模型
> 16. 哪些 fuzzer 从种子池中移除种子

## 五、预处理

- 检测待测程序
- 剔除潜在冗余配置（如，种子选择）
- 修建种子
- 生成驱动程序

### 5.1 检测

- 通过检测获取有价值的信息
- 分为静态检测和动态检测
  - 静态检测：程序运行前、开销比动态检测少、如果依赖其他库，要对库进行单独检测、可以通过二进制重写工具检测；
  - 动态检测：程序运行时、开销大、易于检测动态链接库；
- 举例 AFL 两种检测方式都支持

#### 5.1.1 执行反馈？？？

灰盒模糊器通常将执行反馈作为输入来演化测试用例。 LibFuzzer、AFL 及其后代通过检测被测程序中的每个分支指令来计算分支覆盖率。然而，它们将分支覆盖信息存储在一个紧凑的位向量中，这可能由于路径冲突而变得不准确。 CollAFL 最近通过引入一个新的路径敏感哈希函数来解决这个缺点。 同时，Syzkaller 使用节点覆盖率作为他们的执行反馈，而 honggfuzz  允许用户选择使用哪个执行反馈。

#### 5.1.2 线程调度

条件竞争错误可能很难触发，因为它们依赖于非确定性行为，这种行为可能很少发生。然而，检测（插桩？）也可用于通过显式控制线程的调度方式来触发不同的非确定性程序行为。 现有工作表明，即使是随机调度线程也可以有效地发现竞争条件错误 。

#### 5.1.3 内存 fuzzing

在测试大型程序时，有时希望只对待测程序的一部分进行模糊测试，而不为每次模糊迭代重新生成一个进程，以最大限度地减少执行开销。 例如，复杂的（例如，GUI）应用程序在接受输入之前通常需要几秒钟的处理时间。 对此类程序进行模糊测试的一种方法是在 GUI 初始化后获取被测程序的快照。 为了对新的测试用例进行模糊测试，可以在将新的测试用例直接写入内存并执行之前恢复内存快照。 同样的直觉适用于涉及客户端和服务器之间大量交互的模糊网络应用程序。 这种技术被称为内存模糊测试。

AFL 还使用 fork server 来避免一些进程启动成本。 尽管它与内存模糊测试具有相同的动机，但 fork server 涉及为每次模糊迭代分叉一个新进程。

一些模糊器对函数执行内存模糊测试，而不在每次迭代后恢复被测程序的状态。 我们将这种技术称为内存 API 模糊测试。 例如，AFL 有一个称为持久模式的选项，它在循环中重复执行内存 API 模糊测试，而无需重新启动进程。 在这种情况下，AFL 忽略了在同一执行中多次调用函数的潜在副作用。 

尽管高效的内存 API 模糊测试会受到不合理的模糊测试结果的影响：内存模糊测试发现的错误（或崩溃）可能无法重现，因为（1）为目标函数构造有效的调用上下文并不总是可行的， (2) 可能存在跨多个函数调用未捕获的副作用。 请注意，内存 API 模糊测试的可靠性主要取决于良好的入口点函数，而找到这样的函数可能是一项具有挑战性的任务。

### 5.2 种子选择

回想一下第 2 节，模糊器接收一组模糊配置来控制模糊算法的行为。 然而，一些模糊配置的参数，例如基于变异的模糊器的种子，可以有很大甚至无限的域。 例如，假设分析师对接受 MP3 文件作为输入的 MP3 播放器进行模糊测试。 由于有效 MP3 文件的数量不限，我们应该使用哪个种子进行模糊测试？ 这个减小初始种子池大小的问题被称为种子选择问题。

- minset

- 代码覆盖率每增加 1%，发现的错误百分比会增加 0.92%。

Fuzzers 在实践中使用了各种不同的覆盖率指标。 例如，AFL 的 minset 基于分支覆盖率，每个分支上都有一个对数计数器。 这背后的基本原理是，仅当分支数量的数量级不同时，它们才被视为不同。 Hon-ggfuzz 根据执行指令、执行分支和唯一基本块的数量计算覆盖率。 该指标允许模糊器向 minset 添加更长的执行，这可以帮助发现拒绝服务漏洞或性能问题。

### 5.3 种子修剪

较小的种子可能会消耗更少的内存并带来更高的吞吐量。 因此，一些 fuzzer 试图在对种子进行模糊测试之前减小种子的大小，这称为种子修剪。 种子修剪可以发生在 PREPROCESS 中的主要模糊循环之前或作为 CONFUPDATE 的一部分。 使用种子修剪的一个值得注意的 fuzzer 是 AFL，只要修改后的种子达到相同的覆盖率，它就使用其代码覆盖检测来迭代删除种子的一部分。 同时，Rebert 等人。 报道称，他们的 size minset 算法通过对较小的种子给予更高的优先级来选择种子，与随机种子选择相比，导致的独特错误更少。 对于模糊 Linux 系统调用处理程序的特定情况，Moon-Shine [173] 扩展了 Syzkaller [226] 以减少种子的大小，同时保留使用静态分析检测到的调用之间的依赖关系。

### 5.4 准备驱动程序

当难以直接对被测程序进行模糊测试时，为模糊测试准备驱动程序是有意义的。 这个过程在实践中主要是手动的，尽管这只在 fuzz 活动开始时完成一次。 例如，当我们的目标是一个库时，我们需要准备一个调用库中函数的驱动程序。 类似地，内核模糊器可以模糊用户态应用程序以测试内核。 IoT-Fuzzer 通过让驱动程序与相应的智能手机应用程序通信来瞄准物联网设备。

## 六、调度

- 调度即选择下次 fuzz 迭代的策略
- 本文只讨论灰盒和黑盒的调度；白盒因符号执行器有特有的复杂设置，本文不表

### 6.1 Fuzz 配置调度（FCS）问题

调度的目标是分析有关配置的当前可用信息，并选择一个更有可能导致最有利结果的信息，例如，找到最多数量的独特错误，或最大化生成的输入集所获得的覆盖率 . 从根本上说，每个调度算法都面临着相同的探索与利用冲突——时间可以花在收集每个配置的更准确信息以告知未来决策（探索），或者对当前被认为会导致更有利结果的配置进行模糊处理 （开发）。 吴等人将这种内在冲突称为模糊配置调度 (FCS) 问题。

在我们的模型模糊器（算法 1）中，函数 SCHEDULE 基于（i）当前模糊配置集 C、（ii）当前时间 t_elapsed 和（iii）总时间预算 t_limit 选择下一个配置。 然后将此配置用于下一次模糊迭代。 请注意，SCHEDULE 仅与决策有关。 该决定所依据的信息由 PREPROCESS 和 CONFUPDATE 获取，它们用这些知识扩充了 C。

### 6.2 黑盒 FCS 算法

在黑盒设置中，FCS 算法可以使用的唯一信息是配置的模糊结果——发现的崩溃和错误的数量以及到目前为止花费的时间。 Householder 和 Foote 是第一个研究如何在 CERT BFF 黑盒突变模糊器 [52] 中利用这些信息的人。在他们的工作中，他们将黑盒突变模糊测试建模为一系列伯努利试验。通过偏爱具有更高观察到的成功概率（#unique crash/#runs）的配置，他们证明了 BFF 在固定时间内发现的唯一崩溃数量有所增加。
Woo 等人进一步改进了这一结果。 [235] 在多个方面。首先，他们将模型改进为权重未知的加权优惠券收集器问题（WCCP/UW），该模型学习了每次试验成功概率的衰减上限。其次，他们将多臂老虎机（MAB）算法应用于模糊测试，这是面对探索与利用冲突时的常见应对策略 [37]。第三，他们将配置的成功概率归一化为已经花费的时间，在那里他们更喜欢更快的配置。第四，他们重新定义了模糊迭代，从运行固定数量的模糊运行到固定的时间，进一步降低了较慢配置的优先级。

### 6.3 灰盒 FCS 算法

在灰盒设置中，FCS 算法可以选择使用关于每个配置的更丰富的信息集，例如，在对配置进行模糊测试时获得的覆盖率。 AFL [241] 是这一类别的先驱，它基于进化算法（EA）。 直观地说，EA 维护着一组配置，每个配置都有一些“适合度”值。 EA 选择适合的配置并将它们应用于遗传转化，例如突变和重组以产生后代，这些后代可能在以后成为新的配置。假设是这些产生的配置更有可能是适合的。

要在 EA 的上下文中理解 FCS，我们需要定义 (i) 什么使配置适合，(ii) 如何选择配置，以及 (iii) 如何使用所选配置。 作为一种高级近似，在行使控制流边缘的配置中，AFL 认为包含最快和最小输入的配置是合适的（AFL 用语中的“最喜欢的”）。 AFL 维护一个配置队列，它从中选择下一个合适的配置，基本上就好像队列是循环的一样。 选择后，AFL 会将更多运行分配给最快且具有更高分支覆盖率的配置。 从 FCS 的角度来看，注意对快速配置的偏好与黑盒设置相同。

B€ohme 等人的 AFLFast。 [40] 在所有三个方面都对 AFL 进行了改进。首先，它修改配置适应度设置和选择以优先探索新的稀有路径。此外，AFLFast 对选定的配置进行模糊测试，其次数由功率调度确定。它的 FAST 功率计划从一个小的“能量”值开始，以确保在配置之间进行初始探索，并以指数方式增加到极限以快速确保充分利用。最后，它还通过执行相同路径的生成输入的数量对能量进行归一化，从而促进对不太频繁的模糊配置的探索。AFLFast 的创新已部分集成到 AFL 中，即 FidgetyAFL [5]。 Zalewski 发现 AFLFast 最大的改进是快速遍历所有新添加的种子。因此，AFL 现在在每个种子上花费的时间更少。在相关工作中，AFLGo [39] 通过修改其优先级属性来扩展 AFLFast，以针对特定的程序位置。 Hawkeye [56] 通过在种子调度和输入生成中利用静态分析，进一步改进了定向模糊测试。 FairFuzz [141] 通过对每对种子和稀有分支使用突变掩码来指导运动来锻炼稀有分支。 QTEP [228] 使用静态分析来推断二进制文件的哪个部分更“错误”，并优先考虑覆盖它们的配置。

## 七、输入生成

由于测试用例的内容直接控制是否触发错误，因此用于输入生成的技术自然是模糊器中最有影响力的设计决策之一。传统上，模糊器分为基于生成或基于突变的模糊器 [212]。基于生成的模糊器根据给定模型生成测试用例，该模型描述了被测程序预期的输入。在本文中，我们将此类模糊器称为基于模型的模糊器。另一方面，基于突变的模糊器通过改变给定的种子输入来产生测试用例。基于突变的模糊器通常被认为是无模型的，因为种子只是示例输入，即使数量很大，它们也不能完全描述被测程序的预期输入空间。在本节中，我们基于底层测试用例生成（INPUTGEN）机制解释和分类模糊器使用的各种输入生成技术。因为，该函数的唯一输入是配置 conf，如算法 1 所示，它基于 PREPROCESS 或 CONFUPDATE 收集的信息。

### 7.1 基于模型（基于生成）的 Fuzzer

基于模型的模糊器基于给定模型生成测试用例，该模型描述了被测程序可能接受的输入或执行，例如精确表征输入格式的语法或不太精确的约束，例如标识文件类型的魔术值。

#### 7.1.1 预定义模型

一些模糊器使用可由用户配置的模型。例如，Peach [79]、PROTOS [124] 和 Dharma [4] 采用用户提供的规范。 Autodaf e [224]、Sulley [19]、SPIKE [16]、SPIKEfile [211] 和 LibFuzzer [9]、[12] 公开了允许分析师创建自己的输入模型的 API。 Tavor [250] 还接受以扩展巴科斯-瑙尔形式 (EBNF) 编写的输入规范，并生成符合相应语法的测试用例。类似地，PROTOS [124]、SNOOZE [32]、KiF [15] 和 T-Fuzz [118] 等网络协议模糊器也从用户那里获取协议规范。内核 API 模糊器 [119]、[163]、[168]、[226]、[231] 以系统调用模板的形式定义输入模型。这些模板通常指定系统调用期望作为输入的参数的数量和类型。在内核模糊测试中使用模型的想法起源于 Koopman 等人的开创性工作 [132]，他们将操作系统的鲁棒性与一组有限的手动选择的系统调用测试用例进行了比较。 Nautilus [25] 使用基于语法的输入生成进行通用模糊测试，并使用其语法进行种子修剪（参见第 3.3 节）。

其他基于模型的模糊器针对特定的语言或语法，并且该语言的模型内置于模糊器本身。例如，cross_fuzz [242] 和 DOM-fuzz [194] 生成随机文档对象模型（DOM）对象。同样，jsfunfuzz [194] 根据自己的语法模型生成随机但语法正确的 JavaScript 代码。 QuickFuzz [97] 在生成测试用例时利用现有的描述文件格式的 Haskell 库。一些网络协议模糊器，如 Franken-certs [44]、TLS-Attacker [203]、tlsfuzzer [128] 和 llfuz-zer [207] 设计有特定网络协议的模型，如 TLS 和 NFC。杜威等人。 [72]，[73]提出了一种生成测试用例的方法，该方法不仅在语法上正确，而且通过利用约束逻辑编程在语义上也多样化。 LangFuzz [109] 通过解析一组作为输入的种子来生成代码片段。然后，它随机组合片段并将种子与片段进行变异以生成测试用例。由于它提供了语法，它总是产生语法正确的代码。虽然 LangFuzz 应用于 JavaScript 和 PHP，但 BlendFuzz [239] 以 XML 和正则表达式解析器为目标，并基于与 LangFuzz 类似的想法。

#### 7.1.2 推断模型

推断模型而不是依赖于预定义的或用户提供的模型最近越来越受到关注。 尽管有大量关于自动输入格式和协议逆向工程主题的已发表研究[31]、[48]、[66]、[69]、[146]，但只有少数模糊器利用了这些技术。 与检测（第 3.1 节）类似，模型推断可以发生在 PREPROCESS 或 CONFUPDATE 中。

预处理中的模型推理。一些模糊器将模型推断为预处理步骤。 TestMiner [70] 搜索被测程序中可用的数据，例如文字，以预测合适的输入。给定一组种子和一个语法，Skyfire [227] 使用数据驱动的方法来推断概率上下文敏感语法，然后使用它来生成一组新的种子。与以前的作品不同，它专注于生成语义有效的输入。 IMF [103] 通过分析系统 API 日志来学习内核 API 模型，并生成使用推断模型调用一系列 API 调用的 C 代码。 Code Alchemist [104] 将 JavaScript 代码分解为“代码块”并计算组装约束，这些约束描述了何时可以将不同的块组装或合并在一起以产生语义上有效的测试用例。这些约束是使用静态和动态分析计算的。 Neural [65] 和 Learn&Fuzz [94] 使用基于神经网络的机器学习技术从给定的一组测试文件中学习模型，然后从推断的模型中生成测试用例。刘等人。 [147] 提出了一种特定于文本输入的类似方法。

CONFUPDATE 中的模型推理。其他模糊器可能会在每次模糊迭代后更新他们的模型。 PULSAR [88]从程序生成的一组捕获的网络数据包中自动推断网络协议模型。然后使用学习到的网络协议对程序进行模糊测试。PULSAR 在内部构建状态机并映射与状态相关的消息令牌。此信息稍后用于生成覆盖状态机中更多状态的测试用例。 Doup e 等人。 [76] 提出了一种通过观察 I/O 行为来推断 Web 服务状态机的方法。然后使用推断的模型来扫描网络漏洞。 Ruiter 等人的工作。 [187] 类似，但它以 TLS 为目标，并基于 LearnLib [181] 实现。GLADE [33] 从一组 I/O 样本中合成上下文无关文法，并使用推断文法对 PUT 进行模糊测试。 go-fuzz [225] 是一个灰盒模糊器，它为添加到其种子池中的每个种子建立一个模型。该模型用于从该种子生成新的输入。为了帮助限制符号执行，Shenet al. [201]使用神经网络来解决困难的分支条件。

#### 7.1.3 编码模型

模糊测试通常用于测试解析特定文件格式的解码器程序。 很多文件格式都有对应的编码器程序，可以认为是文件格式的隐式模型。 MutaGen [127] 利用编码器程序中包含的隐式模型来生成新的测试用例。 与大多数基于变异的模糊器不同，后者会变异现有的测试用例（参见第 5.2 节）以生成测试用例，MutaGen 会变异编码器程序。 具体来说，为了生成一个新的测试用例，MutaGen 计算编码器程序的动态程序片段并运行它。 其基本思想是程序切片会稍微改变编码器程序的行为，从而生成稍微畸形的测试用例。

### 7.2 无模型（基于突变）的 fuzzer



## 八、输入评价

## 九、配置更新

## 十、相关工作
