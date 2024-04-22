---
layout: page
title: 实验7：带有 DRAM 和缓存的 RISC-V 处理器
permalink: /Lab7/
nav_order: 7
---


# 实验 7: 带有 DRAM 和缓存的 RISC-V 处理器

> **实验 7截止日期**： 2016年11月18日，美东时间晚上11:59:59。
>
> 你需要提交的实验 7内容包括：
>
> - 在 `WithoutCache.bsv` 和 `WithCache.bsv` 中完成练习 1、2 和 4 的答案
> - 在 `discussion.txt` 中完成讨论问题 1 到 3 的答案

## 简介

现在，你已经拥有了一个具有分支目标和方向预测器（BTB 和 BHT）的六级流水线 RISC-V 处理器。不幸的是，你的处理器只能运行能够适应 256 KB FPGA块 RAM 的程序。这对于我们运行的小型基准程序（如250项快速排序）来说是足够的，但大多数有趣的应用程序都远大于 256 KB。幸运的是，我们使用的 FPGA 板配备了 1 GB DDR3 DRAM，可供 FPGA 访问。这非常适合存储大型程序，但由于 DRAM 读取延迟相对较长，这可能会影响性能。

本实验将重点使用 DRAM 而非块 RAM 作为主程序和数据存储来存储更大的程序，并添加缓存以减少长延迟 DRAM 读取对性能的影响。

首先，你将编写一个转换模块，将 CPU 内存请求转换为 DRAM 请求。此模块大大扩展了你的程序存储空间，但由于几乎每个周期都要从 DRAM 读取，你的程序将运行得更慢。接下来，你将实现一个缓存来减少需要从 DRAM 读取的次数，从而提高处理器性能。最后，你将为 FPGA 合成你的设计，并运行需要 DRAM 和长时间运行的非常大的基准测试。

## 测试基础设施的变化

如果我们每次运行一个新测试就必须重新配置 FPGA，那么运行所有的组装测试将需要很长时间（重新配置 FPGA 大约需要一分钟）。由于我们没有更改硬件，我们将只配置一次 FPGA，然后每次想要运行新测试时进行*软重置*。软件测试台（位于 `scemi/Tb.cpp`）将启动软重置，将 `*.vmh` 文件写入您的 FPGA 的 DRAM，并启动新测试。在每个测试开始之前，软件测试台还会打印出基准测试的名称，以帮助调试。在软件仿真（无 FPGA）中，我们还将模拟将 `*.vmh` 文件写入 DRAM 的过程，因此仿真时间也会比以前更长。

以下是使用名为 `withoutcache` 的处理器来仿真 `simple.S` 和 `add.S` 组装测试的示例命令：

```
cd scemi/sim
./withoutcache_dut > log.txt &
./tb ../../programs/build/assembly/vmh/simple.riscv.vmh ../../programs/build/assembly/vmh/add.riscv.vmh 
```

这是样本输出：

```
---- ../../programs/build/assembly/vmh/simple.riscv.vmh ----
1196
103
通过

---- ../../programs/build/assembly/vmh/add.riscv.vmh ----
5635
427
通过

SceMi 服务线程完成！
```

我们还提供了两个脚本 `run_asm.sh` 和 `run_bmarks.sh` 来分别运行所有组装测试和基准测试。例如，我们可以使用以下命令测试处理器 `withoutcache`：

```
./run_asm.sh withoutcache
./run_bmarks.sh withoutcache
```

BSV 的标准输出将分别重定向到 `asm.log` 和 `bmarks.log`。

## DRAM 接口

你将在本课程中使用的 VC707 FPGA 板配备了 1 GB DDR3 DRAM。DDR3 内存具有 64 位宽的数据总线，但每次传输都会发送八个 64 位块，因此实际上它的作用就像一个 512 位宽的内存。DDR3 内存具有高吞吐量，但其读取延迟也比较高。

Sce-Mi 接口为我们生成了 DDR3 控制器，我们可以通过 `MemoryClient` 接口连接到它。本实验中为你提供的 typedef 使用了 BSV 的内存包中的类型（见 BSV 参考指南或 `$BLUESPECDIR/BSVSource/Misc/Memory.bsv` 的源代码）。以下是 `src/includes/MemTypes.bsv` 中与 DDR3 内存相关的一些 typedef：

```bsv
typedef 24 DDR3AddrSize;
typedef Bit#(DDR3AddrSize) DDR3Addr;
typedef 512 DDR3DataSize;
typedef Bit#(DDR3DataSize) DDR3Data;
typedef TDiv#(DDR3DataSize, 8) DDR3DataBytes;
typedef Bit#(DDR3DataBytes) DDR3ByteEn;
typedef TDiv#(DDR3DataSize, DataSize) DDR3DataWords;

// 下面的 typedef 等同于：
// typedef struct {
//     Bool        write;
//     Bit#(64)    byteen;
//     Bit#(24)    address;
//     Bit#(512)   data;
// } DDR3_Req deriving (Bits, Eq);
typedef MemoryRequest#(DDR3AddrSize, DDR3DataSize) DDR3_Req;

// 下面的 typedef 等同于：
// typedef struct {
//     Bit#(512)   data;
// } DDR3_Resp deriving (Bits, Eq);
typedef MemoryResponse#(DDR3DataSize) DDR3_Resp;

// 下面的 typedef 等同于：
// interface DDR3_Client;
//     interface Get#( DDR3_Req )  request;
//     interface Put#( DDR3_Resp ) response;
// endinterface;
typedef MemoryClient#(DDR3AddrSize, DDR3DataSize) DDR3_Client;
```

### `DDR3_Req`

对 DDR3 的读取和写入请求与对 `FPGAMemory` 的请求不同。最大的区别是字节启用信号 `byteen`。

- `write` —— 布尔值，指定此请求是写入请求还是读取请求。
- `byteen` —— 字节启用，指定将写入哪些 8 位字节。此字段对于读取请求无效。如果你想写入所有 16 字节（即 512 位），你将需要将此设置为全部为 1。你可以使用文字 `'1`（注意单引号）或 `maxBound` 来实现。
- `address` —— 读取或写入请求的地址。DDR3 内存以 512 位块为单位进行寻址，因此地址 0 指的是第一个 512 位块，地址 1 指的是第二个 512 位块。这与 RISC-V 处理器使用的字节寻址非常不同。
- `data` —— 用于写入请求的数据值。

### `DDR3_Resp`

DDR3 内存只对读取发送响应，就像 `FPGAMemory` 一样。内存响应类型是一种结构体——因此，你将不会直接接收到 `Bit#(512)` 值，而必须访问响应中的 `data` 字段以获取 `Bit#(512)` 值。

### `DDR3_Client`

`DDR3_Client` 接口由一个 `Get` 子接口和一个 `Put` 子接口组成。这个接口由处理器公开，Sce-Mi 基础设施将其连接到 DDR3 控制器。你无需担心构建此接口，因为示例代码中已为你完成。

### 示例代码

这里是一些展示如何构建 FIFOs 以及 DDR3 内存接口初始化接口的示例代码，

此示例代码提供在 `src/DDR3Example.bsv` 中。

```bsv
import GetPut::*;
import ClientServer::*;
import Memory::*;
import CacheTypes::*;
import WideMemInit::*;
import MemUtil::*;
import Vector::*;

// 其他包和类型定义

(* synthesize *)
module mkProc(Proc);
 Ehr#(2, Addr)  pcReg <- mkEhr(?);
 CsrFile         csrf <- mkCsrFile;
 
 // 其他处理器状态和组件
 
 // 接口 FIFO 到真实的 DDR3
 Fifo#(2, DDR3_Req)  ddr3ReqFifo  <- mkCFFifo;
 Fifo#(2, DDR3_Resp) ddr3RespFifo <- mkCFFifo;
 // 初始化 DDR3 的模块
 WideMemInitIfc       ddr3InitIfc <- mkWideMemInitDDR3( ddr3ReqFifo );
 Bool memReady = ddr3InitIfc.done;
 
 // 将 DDR3 包装成 WideMem 接口
 WideMem           wideMemWrapper <- mkWideMemFromDDR3( ddr3ReqFifo, ddr3RespFifo );
 // 将 WideMem 接口分割为两个（多路复用方式使用）
 // 这个分割器只在重置后生效（即 memReady && csrf.started）
 // 否则 guard 可能失败，我们将获取到垃圾 DDR3 响应
 Vector#(2, WideMem)     wideMems <- mkSplitWideMem( memReady && csrf.started, wideMemWrapper );
 // 指令缓存应使用 wideMems[1]
 // 数据缓存应使用 wideMems[0]
 
 // 在软重置期间，一些垃圾可能进入 ddr3RespFifo
 // 这条规则将排空所有此类垃圾
 rule drainMemResponses( !csrf.started );
  ddr3RespFifo.deq;
 endrule
 
 // 其他规则
 
 method ActionValue#(CpuToHostData) cpuToHost if(csrf.started);
  let ret <- csrf.cpuToHost;
  return ret;
 endmethod
 
 // 将 ddr3RespFifo empty 添加到 guard 中，确保垃圾已被排空
 method Action hostToCpu(Bit#(32) startpc) if ( !csrf.started && memReady && !ddr3RespFifo.notEmpty );
  csrf.start(0); // 只有 1 个核心，id = 0
  pcReg[0] <= startpc;
 endmethod
 
 // 为测试台提供 DDR3 初始化的接口
 interface WideMemInitIfc memInit = ddr3InitIfc;
 // 接口到真实 DDR3 控制器
 interface DDR3_Client ddr3client = toGPClient( ddr3ReqFifo, ddr3RespFifo );
endmodule
```

在上述示例代码中，`ddr3ReqFifo` 和 `ddr3RespFifo` 作为与真实 DDR3 DRAM 的接口。在仿真中，我们提供了一个名为 `mkSimMem` 的模块来模拟 DRAM，该模块在 `scemi/SceMiLayer.bsv` 中实例化。在 FPGA 合成中，DDR3 控制器在顶层模块 `mkBridge` 中实例化，位于 `$BLUESPECDIR/board_support/bluenoc/bridges/Bridge_VIRTEX7_VC707_DDR3.bsv`。还有一些胶合逻辑在 `scemi/SceMiLayer.bsv` 中。

在示例代码中，我们使用模块 `mkWideMemFromDDR3` 将 `DDR3_Req` 和 `DDR3_Resp` 类型转换为更友好的 `WideMem` 接口，该接口定义在 `src/includes/CacheTypes.bsv` 中。

### 共享 DRAM 接口

示例代码中仅暴露了单一的与 DRAM 的接口，但你有两个模块将使用它：指令缓存和数据缓存。如果它们都向 `ddr3ReqFifo` 发送请求，并且都从 `ddr3RespFifo` 获取响应，那么它们的响应可能会混淆。为了处理这个问题，你需要一个单独的 FIFO 来跟踪响应应当返回的顺序。每个加载请求都与一个入队到排序 FIFO 的操作配对，该操作指定谁应该获取响应。

为了简化这个过程，我们提供了模块 `mkSplitWideMem` 来将 DDR3 FIFOs 分割为两个 `WideMem` 接口。这个模块定义在 `src/includes/MemUtils.bsv` 中。为了防止 `mkSplitWideMem` 过早采取行动并显示出预期之外的行为，我们将其第一个参数设置为 `memReady && csrf.started`，以在处理器启动之前冻结它。这也可以避免与 DRAM 内容初始化发生调度冲突。

### 处理软重置问题

如前所述，你将在启动每个新测试前对处理器状态进行软重置。在软重置期间，由于某些跨时钟域问题，一些垃圾数据可能会入队到 `ddr3RespFifo` 中。为了处理这个问题，我们添加了 `drainMemResponses` 规则来排空垃圾数据，并在 `hostToCpu` 方法的保护条件中添加了检查 `drainMemResponses` 是否为空的条件。

> **建议**：在每个管道阶段的规则中添加 `csrf.started` 到 guard 中。这可以防止在处理器启动之前 DRAM 被访问。

## 从前一个实验室迁移代码

本实验室提供的代码非常相似，但存在一些差异需要注意。大多数差异都在提供的示例代码 `src/DDR3Example.bsv` 中显示。

### 修改的 Proc 接口

Proc 接口现在只有单一的内存初始化接口，以匹配统一的 DDR3 内存。此内存初始化接口的宽度已扩展到每次传输 512 位。这个新的初始化接口的类型是 `WideMemInitIfc`，在 `src/includes/WideMemInit.bsv` 中实现。

### 空文件

本实验室的两个处理器实现：`src/WithoutCache.bsv` 和 `src/WithCache.bsv` 最初是空的。你应该将代码从 `SixStageBHT.bsv` 或 `SixStageBonus.bsv` 复制过来作为这些处理器的起点。`src/includes/Bht.bsv` 也是空的，因此你还需要将前一个实验室的代码复制过来。

### 新文件

以下是在 `src/includes` 文件夹下提供的新文件概述：

| 文件名            | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `Cache.bsv`       | 一个空文件，你将在本实验室中实现缓存模块。                   |
| `CacheTypes.bsv`  | 关于缓存的类型和接口定义的集合。                             |
| `MemUtil.bsv`     | 关于 DDR3 和 `WideMem` 的有用模块和函数的集合。              |
| `SimMem.bsv`      | 在仿真中使用的 DDR3 内存。它有 10 个周期的流水线访问延迟，但额外的胶合逻辑可能会增加访问 DRAM 的总延迟。 |
| `WideMemInit.bsv` | DDR3 初始化模块。                                            |

`MemTypes.bsv` 也有一些变化。

## 使用 DRAM 而不使用缓存的处理器 `WithoutCache.bsv`

> **练习 1 (10 分)**：在 `Cache.bsv` 中实现一个名为 `mkTranslator` 的模块，它接受与 DDR3 内存相关的某些接口（例如 `WideMem`），并返回一个 `Cache` 接口（见 `CacheTypes.bsv`）。
>
> 该模块不应进行任何缓存，只需从 `MemReq` 到 DDR3 请求（如果使用 `WideMem` 接口，则为 `WideMemReq`）以及从 DDR3 响应（如果使用 `WideMem` 接口，则为 `CacheLine`）到 `MemResp` 的转换。这将需要一些内部存储来跟踪你从主存返回的缓存行中需要哪个字。将 `mkTranslator` 集成到文件 `WithoutCache.bsv` 中的六阶段管线中（即你不应再使用 `mkFPGAMemory`）。你可以通过在 `scemi/sim/` 目录下运行以下命令来构建这个处理器：
>
> ```
> $ build -v withoutcache
> ```
>
> 并通过运行以下命令来测试此处理器：
>
> ```
> $ ./run_asm.sh withoutcache
> ```
>
> 和
>
> ```
> $ ./run_bmarks.sh withoutcache
> ```
>
> 在 `scemi/sim/` 目录下。

> **讨论问题 1 (5 分)**：记录 `./run_bmarks.sh withoutcache` 的结果。你在每个基准测试中看到的 IPC 是多少？

## 使用带有缓存的 DRAM 的处理器 `WithCache.bsv`

通过使用模拟的 DRAM 运行基准测试，你应该已经注意到你的处理器速度大大减慢了。通过记住之前的 DRAM 加载到缓存中，你可以重新提升处理器的速度，正如课堂上所描述的那样。

> **练习 2 (20 分)**：实现一个名为 `mkCache` 的模块作为直接映射缓存，仅在替换缓存行时写回，并且仅在写缺失时分配。
>
> 该模块应接受一个 `WideMem` 接口（或类似的东西）并暴露一个 `Cache` 接口。使用 `CacheTypes.bsv` 中的 `typedefs` 来定义你的缓存大小和 `Cache` 接口。你可以使用寄存器向量或寄存器文件来实现缓存中的数组，但寄存器向量更容易指定初始值。将此缓存集成到 `WithoutCache.bsv` 中的相同管线，并将其保存在 `WithCache.bsv` 中。你可以通过在 `scemi/sim/` 目录下运行以下命令来构建此处理器：
>
> ```
> $ build -v withcache
> ```
>
> 并通过运行以下命令来测试此处理器：
>
> ```
> $ ./run_asm.sh withcache
> ```
>
> 和
>
> ```
> $ ./run_bmarks.sh withcache
> ```
>
> 在 `scemi/sim/` 目录下。

> **讨论问题 2 (5 分)**：记录 `./run_bmarks.sh withcache` 的结果。你在每个基准测试中看到的 IPC 是多少？

## 运行大型程序

通过添加对 DDR3 内存的支持，你的处理器现在可以运行比我们一直在使用的小基准测试更大的程序。不幸的是，这些大型程序需要更长的运行时间，在许多情况下，模拟完成需要太长时间。现在是尝试 FPGA 合成的好时机。通过在 FPGA 上实现你的处理器，由于设计在硬件而非软件中运行，你将能够更快地运行这些大型程序。

> **练习 3 (0 分，但你仍然应该做这个)**： 在为 FPGA 合成之前，让我们试试看一个在模拟中运行时间很长的程序。程序 `./run_mandelbrot.sh` 运行一个基准测试，使用 1 和 0 打印曼德博集合的方形图像。运行此基准测试以查看它在实时中的运行速度有多慢。请不要等待此基准测试完成，可以使用 Ctrl-C 提前终止。

### 为 FPGA 合成

你可以通过进入 `scemi/fpga_vc707` 文件夹并执行以下命令开始为 `WithCache.bsv` 进行 FPGA 合成：

```
vivado_setup build -v
```

这个命令将需要很长时间（大约一小时）并消耗大量计算资源。你可能想选择一个负载较轻的 vlsifarm 服务器。你可以使用 `w` 查看有多少人登录，并可以使用 `top` 或 `uptime` 查看正在使用的资源。

一旦完成，你可以通过运行 `./submit_bitfile` 命令将你的 FPGA 设计提交给共享的 FPGA 板进行测试，并可以使用 `./get_results` 检查结果。`get_results` 脚本将在你的结果准备好之前持续显示当前的 FPGA 状态。在 FPGA 上执行可能需要几分钟时间，如果其他学生也提交了作业，则可能需要更长时间。FPGA 上的 `*.vmh` 程序文件位于 `/mit/6.175/fpga-programs`。它包括在模拟中使用的所有程序，以及具有较大输入的基准程序（在 `large` 子目录中）。你还可以通过在 `programs/benchmarks` 文件夹中执行 `make -f Makefile.large` 生成大型基准的 `*.vmh` 文件。然而，这些 `*.vmh` 文件在软件中模拟将需要很长时间。

如果你想检查 FPGA 的状态，可以运行 `./fpga_status` 命令。

> **练习 4 (10 分)**：为 FPGA 合成 `WithCache.bsv` 并将你的设计发送到共享的 FPGA 执行。获取正常和大型基准的结果并将它们添加到 `discussion.txt`。

> **讨论问题 3 (10 分)**：曼德博程序在你的处理器中需要多少周期来执行？当前的 FPGA 设计有效时钟速度为 50 MHz。曼德博程序以秒为单位执行需要多长时间？估计你在硬件与模拟中看到的速度提升，通过估计在模拟中运行 `./run_mandelbrot.sh` 将需要多长时间（以墙钟时间为单位）。

> **讨论问题 4 (可选)**：完成这个实验室花了你多长时间？

完成后，请提交你的代码并执行 `git push`。

> **来自你友好的助教的提示**： 如果你在 FPGA 测试中遇到任何问题，请尽快通过电子邮件通知我。基础设施并不非常稳定，但及早通知我有关任何问题将使它们更快得到解决。

> **值得关注的内容：（添加于 11 月 17 日）** 让我们分

析一些 FPGA 合成的结果。
>
>查看 `scemi/fpga_vc707/xilinx/mkBridge/mkBridge.runs/synth_1/runme.log`，搜索“Report Instance Areas”。此报告显示了你的设计使用的单元数量的分解。`scemi_dut_dut_dutIfc_m_dut` 使用了多少个单元？总共有多少个单元？（查看 `top`）。
>
>看看 `scemi/fpga_vc707/xilinx/mkBridge/mkBridge.runs/impl_1/mkBridge_utilization_placed.rpt`。这包含了你的设计对 FPGA 资源的使用报告（这些资源被组织为“切片”，与单元不同）。在“1. 切片逻辑”下，你可以看到你的整个设计（包括存储器控制器和 Sce-Mi 接口）使用了多少切片。现在看看 `scemi/fpga_vc707/xilinx/mkBridge/mkBridge.runs/impl_1/mkBridge_timing_summary_routed.rpt`。这里有一些时序信息，最重要的是，你的 CPU 中最长的组合路径的延迟。在标记为“Max Delay Paths”的部分中查找“`scemi_dut_dut_dutIfc_m_dut/[signal]`” 的出现。"Slack" 是“所需时间”（本质上是时钟周期）与“到达时间”（你的信号传播通过设计的这部分所需时间）之间的差异。你在路径中看到了什么（看“Netlist Resource(s)”列）？为什么我们可能在最大延迟路径中看到 EHR（即关键路径）？（见顶部）

------

© 2016 [麻省理工学院](http://web.mit.edu/). 版权所有。
