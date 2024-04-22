---
layout: post
title: Home
nav_order: 0
description: ""
permalink: /
---

# MIT-6.175 简介/目录

## 什么是 6.175?

6.175 通过实现不同版本的带缓存、分支预测和虚拟内存的流水线机器，教授计算机架构的基本原理。强调编写和评估可以模拟和合成到真实硬件或在 FPGA 上运行的架构描述。使用和设计测试台。本课程适合想要将计算机科学技术应用于复杂硬件设计的学生。

课题包括组合电路（包括加法器和乘法器）、多周期和流水线功能单元、RISC 指令集架构 (ISA)、非流水线和多周期处理器架构、2 至 10 阶段顺序流水线架构、带缓存和层次内存系统的处理器、TLB 和页面错误、I/O 中断。

### 讲师

- [Arvind](http://csg.csail.mit.edu/Users/arvind)
- Quan Nguyen  (助教)

### 讲座

周一周三周五 下午 3:00, [34-302](http://whereis.mit.edu/map-jpg?mapterms=34-302)。

---

### 实验课程目录

- [实验 0: 入门](./Lab0)
- [实验 1: 多路选择器和加法器](./Lab1)
- [实验 2: 乘法器](./Lab2)
- [实验 3: 快速傅里叶变换管道](./Lab3)
- [实验 4: N 元 FIFOs](./Lab4)
- [实验 5: RISC-V 引介 - 多周期与两阶段流水线](./Lab5)
- [实验 6: 具有六阶段流水线和分支预测的 RISC-V 处理器](./Lab6)
- [实验 7: 带有 DRAM 和缓存的 RISC-V 处理器](./Lab7)
- [实验 8: 具有异常处理的 RISC-V 处理器](./Lab8)

### 项目

- [项目1: 存储队列](./project1)
- [项目2: 缓存一致性](./project2)

------

## 日程安排

| 周数 | 日期           | 描述                                                         | 下载链接                                                     |
| :--- | :------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1    | 周三, 9月7日   | 讲座 1: 介绍                                                 | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L01-CCAwoPictures.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L01-CCAwoPictures.pdf) |
|      | 周五, 9月9日   | 讲座 2: 组合电路 **[实验 0](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab0-getting-started.html) 发布, [实验 1](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab1-multiplexers-adders.html) 发布** | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L02-CombinationalCkts.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L02-CombinationalCktsPrint.pdf) |
| 2    | 周一, 9月12日  | 讲座 3: 组合电路 2                                           | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L03-CombinationalCkts-2.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L03-CombinationalCkts-2.pdf) |
|      | 周三, 9月14    | 讲座 4: 时序电路                                             | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L04-SequentialCircuits.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L04-SequentialCircuits.pdf) |
|      | 周五, 9月16    | 讲座 5: 时序电路 2 **实验 1 截止, [实验 2](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab2-multipliers.html) 发布** | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L05-Folded-Combinational-Circuits-revised.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L05-Folded-Combinational-Circuits-revised.pdf) |
| 3    | 周一, 9月19日  | 讲座 6: 组合电路的流水线化                                   | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L06-CombinationalPipelines.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L06-CombinationalPipelines.pdf) |
|      | 周三, 9月21日  | 讲座 7: 基本良好的 BSV 程序 短暂历史寄存器                   | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L07-EHRs-Multirule-Systems-and-Concurrency.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L07-EHRs-Multirule-Systems-and-Concurrency.pdf) |
|      | 周五, 9月23日  | **无课: 学生假日 (秋季招聘会)** **[实验 3](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab3-fft.html) 发布** |                                                              |
| 4    | 周一, 9月26日  | 讲座 8: 多规则系统与规则的并发执行 **实验 2 截止**           | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L08-Multirule-Systems-and-Concurrency.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L08-Multirule-Systems-and-Concurrency.pdf) |
|      | 周三, 9月28日  | 讲座 9: 保护条件                                             | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L09-Guards.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L09-Guards.pdf) |
|      | 周五, 9月30日  | 辅导课 1: Bluespec                                           | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T01-BSV.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T01-BSV.pdf) |
| 5    | 周一, 10月3    | 讲座 10: 非流水线处理器 **[实验 4](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab4-fifo.html) 发布** | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L10-NonPipelinedProcessors.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L10-NonPipelinedProcessors.pdf) |
|      | 周三, 10月5日  | 讲座 11: 非流水线和流水线处理器 **实验 3 截止**              | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L11-Nonpipelined-Processors-2.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L11-Nonpipelined-Processors-2.pdf) |
|      | 周五, 10月7日  | 辅导课 2: 高级 Bluespec                                      | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T02-Advanced-BSV.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T02-Advanced-BSV.pdf) |
| 6    | 周一, 10月10日 | **无课: 原住民日 / 哥伦布日**                                |                                                              |
|      | 周二, 10月11日 | **[实验 5](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab5-riscv-intro.html) 发布** |                                                              |
|      | 周三, 10月12日 | 讲座 12: 控制冒险 **实验 4 截止**                            | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L12-Control-Hazards.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L12-Control-Hazards.pdf) |
|      | 周五, 10月14   | 辅导课 3: RISC-V 处理器 RISC-V 和调试                        | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T03-RISCV-debug.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T03-RISCV-debug.pdf) |
| 7    | 周一, 10月17日 | 讲座 13: 数据冒险                                            | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L13-Data-Hazards.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L13-Data-Hazards.pdf) |
|      | 周三, 10月19日 | 讲座 14: 多阶段流水线 **[实验 6](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab6-riscv-pipeline.html) 发布** | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L14-MultistagePipelines.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L14-MultistagePipelines.pdf) |
|      | 周五, 10月21日 | 辅导课 4: 调试时代和记分板 实验 5 截止                       | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T04-Epochs.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T04-Epochs.pdf) |
| 8    | 周一, 10月24日 | 讲座 15: 分支预测 **实验 5 截止**                            | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L15-BranchPrediction.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L15-BranchPrediction.pdf) |
|      | 周三, 10月26日 | 讲座 16: 分支预测 2                                          | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L16-BranchPrediction-2.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L16-BranchPrediction-2.pdf) |
|      | 周五, 10月28日 | 辅导课 5: 时代和分支预测器 时代、调试和缓存                  | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T05-Caches-Exceptions.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T05-Caches-Exceptions.pdf) |
| 9    | 周一, 10月31日 | 讲座 17: 缓存                                                | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L17-Caches-1.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L17-Caches-1.pdf) |
|      | 周三, 11月2日  | 讲座 18: 缓存 2 **[实验 7](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab7-riscv-caches.html) 发布** | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L18-Caches-2.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L18-Caches-2.pdf) |
|      | 周五, 11月4日  | 辅导课 6: 缓存和异常 实验 6 截止                             | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T06-Caches-Exceptions.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T06-Caches-Exceptions.pdf) |
| 10   | 周一, 11月7日  | 讲座 19: 异常 **实验 6 截止**                                | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L19-Exceptions.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L19-Exceptions.pdf) |
|      | 周三, 11月9日  | 讲座 20: 虚拟内存                                            | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L20-VirtualMemory.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L20-VirtualMemory.pdf) |
|      | 周五, 11月11日 | **无课: 退伍军人节**                                         |                                                              |
| 11   | 周一, 11月14日 | 讲座 21: 虚拟内存和异常 **实验 8 发布**                      | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L21-VirtualMemory-2.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L21-VirtualMemory-2.pdf) |
|      | 周三, 11月16日 | 讲座 22: 缓存一致性 实验 7 截止                              | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L22-CacheCoherence.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L22-CacheCoherence.pdf) |
|      | 周四, 11月17日 | **[实验 8](http://csg.csail.mit.edu/6.175/archive/2016/labs/lab8-riscv-exceptions.html) 发布** |                                                              |
|      | 周五, 11月18日 | 辅导课 7: 项目概述 **实验 7 截止, [项目第一部分](http://csg.csail.mit.edu/6.175/archive/2016/labs/project-part-1.html) 发布** | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T07-Project-Overview.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T07-Project-Overview.pdf) |
| 12   | 周一, 11月21日 | 讲座 23: 顺序一致性                                          | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L23-Sequential-Consistency.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L23-Sequential-Consistency.pdf) |
|      | 周三, 11月23日 | 辅导课 8: 项目第二部分: 一致性 **取消: (提前) 感恩节** 实验 8 截止 |                                                              |
|      | 周五, 11月25日 | **无课: 感恩节** **实验 8 截止**                             |                                                              |
| 13   | 周一, 11月28日 | **无课: 从事项目** 项目第二部分发布                          |                                                              |
|      | 周三, 11月30日 | **无课: 从事项目**                                           |                                                              |
|      | 周四, 12月1日  | **[项目第二部分](http://csg.csail.mit.edu/6.175/archive/2016/labs/project-part-2.html) 发布** |                                                              |
|      | 周五, 12月2日  | 无课: 从事项目 辅导课 8: 项目第二部分: 一致性                | [[pptx]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T08-Project-Coherence.pptx) [[pdf]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/T08-Project-Coherence.pdf) |
| 14   | 周一, 12月5日  | **无课: 从事项目**                                           |                                                              |
|      | 周三, 12月7日  | **无课: 从事项目**                                           |                                                              |
|      | 周五, 12月9日  | **无课: 从事项目**                                           |                                                              |
| 15   | 周一, 12月12日 | **无课: 从事项目**                                           |                                                              |
|      | 周三, 12月14日 | **课程最后一天** **项目展示**                                |                                                              |

---

© 2016 [麻省理工学院](http://web.mit.edu/) 版权所有。
