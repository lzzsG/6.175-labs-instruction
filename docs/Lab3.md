---
layout: page
title: 实验3：快速傅里叶变换管道
permalink: /Lab3/
nav_order: 3
---

# 实验 3: 快速傅里叶变换管道

> **实验 3 截止日期**： 2016年10月5日星期三，晚上11:59:59 EDT。
>
> 实验 3 的交付内容包括：
>
> - 在 `Fifo.bsv` 和 `Fft.bsv` 中对练习 1-4 的回答，
> - 在 `discussion.txt` 中对讨论问题 1-2 的回答。

## 引言

在本实验中，你将构建不同版本的快速傅里叶变换（FFT）模块，从一个组合 FFT 模块开始。这个模块在之前版本的课程中名为“L0x”的讲座中详细描述，讲座标题为《FFT：复杂组合电路的一个示例》。你可以在以下链接找到该讲座的 [[pptx\]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L0x-FFT.pptx) 或 [[pdf\]](http://csg.csail.mit.edu/6.175/archive/2016/lectures/L0x-FFT.pdf)。

首先，你将实现一个折叠的三阶多周期 FFT 模块。这种实现通过阶段间共享硬件来减少所需的面积。接下来，你将实现一个使用寄存器连接各阶段的非弹性流水线 FFT。最后，你将通过使用 FIFO 连接各阶段实现一个弹性流水线 FFT。

## 守卫

发布的 FFT 讲座假设所有 FIFO 上都有守卫。`enq`、`deq` 和 `first` 上的守卫防止包含这些方法调用的规则在方法的守卫不满足时触发。因此，讲座中的代码在不检查 FIFO 是否 `notFull` 或 `notEmpty` 的情况下使用 `enq`、`deq` 和 `first`。

方法上的守卫语法如下所示：

```bsv
method Action myMethodName(Bit#(8) in) if (myGuardExpression);
    // 方法体
endmethod
```

`myGuardExpression` 是一个表达式，当且仅当调用 `myMethodName` 有效时才为 `True`。如果 `myMethodName` 将在下次触发时在规则中使用，那么规则将被阻止执行，直到 `myGuardExpression` 为 `True`。

> **练习 1（5 分）**： 作为热身，为包含在 `Fifo.bsv` 中的两元素无冲突 FIFO 的 `enq`、`deq` 和 `first` 方法添加守卫。

## 数据类型

提供了多种数据类型以帮助实现 FFT。提供的默认设置描述了一个与 64 个不同的 64 位复数输入向量一起工作的 FFT 实现。64 位复数数据的类型定义为 `ComplexData`。`FftPoints` 定义了复数的数量，`FftIdx` 定义了访问向量中一个点所需的数据类型，`NumStages` 定义了阶段数，`StageIdx` 定义了访问特定阶段的数据类型，`BflysPerStage` 定义了每个阶段中蝴蝶单元的数量。这些类型参数为你提供了方便，你可以在实现中自由使用这些类型。

应该注意的是，这个实验的目标不是理解 FFT 算法，而是在一个真实世界的应用中尝试不同的控制逻辑。`getTwiddle` 和 `permute` 函数为你的方便而提供，并包含在测试台中。然而，它们的实现并不严格遵循 FFT 算法，甚至可能会更改。有益的是，不要关注算法

，而是关注如何改变给定数据路径的控制逻辑，以增强其特性。

## 蝴蝶单元

模块 `mkBfly4` 实现了讲座中讨论的 4 路蝴蝶函数。这个模块应该完全按照你在代码中使用的次数来实例化。

```bsv
interface Bfly4;
    method Vector#(4,ComplexData) bfly4(Vector#(4,ComplexData) t, Vector#(4,ComplexData) x);
endinterface

module mkBfly4(Bfly4);
    method Vector#(4,ComplexData) bfly4(Vector#(4,ComplexData) t, Vector#(4,ComplexData) x);
        // 方法体
    endmethod
endmodule
```

## FFT 的不同实现

你将实现与以下 FFT 接口对应的模块：

```bsv
interface Fft;
    method Action enq(Vector#(FftPoints, ComplexData) in);
    method ActionValue#(Vector#(FftPoints, ComplexData)) deq();
endinterface
```

模块 `mkFftCombinational`、`mkFftFolded`、`mkFftInelasticPipeline` 和 `mkFftElasticPipeline` 都应该实现一个与组合模型功能相当的 64 点 FFT。模块 `mkFftCombinational` 已经提供给你。你的任务是实现其他三个模块，并使用提供的组合实现作为基准来验证它们的正确性。

每个模块都包含两个 FIFO，`inFifo` 和 `outFifo`，分别包含输入复数向量和输出复数向量，如下所示。

```bsv
module mkFftCombinational(Fft);
    Fifo#(2, Vector#(FftPoints, ComplexData)) inFifo <- mkCFFifo;
    Fifo#(2, Vector#(FftPoints, ComplexData)) outFifo <- mkCFFifo;
   ...
```

这些 FIFO 是课堂上展示的两元素无冲突 FIFO，在练习一中添加了守卫。

每个模块还包含一个或多个 `mkBfly4` 的 `Vector`，如下所示。

```bsv
Vector#(3, Vector#(16, Bfly4)) bfly <- replicateM(mkBfly4);
```

`doFft` 规则应该从 `inFifo` 中取出一个输入，执行 FFT 算法，最后将结果入队到 `outFifo`。这个规则通常需要其他函数和模块才能正确运作。弹性流水线实现将需要多个规则。

```bsv
   ...
    rule doFft;
        // 规则体
    endrule
   ...
```

`Fft` 接口提供了方法向 FFT 模块发送数据并从中接收数据。该接口只入队到 `inFifo` 并从 `outFifo` 出队。

```bsv
   ...
    method Action enq(Vector#(FftPoints, ComplexData) in);
        inFifo.enq(in);
    endmethod

    method ActionValue#(Vector#(FftPoints, ComplexData)) deq;
        outFifo.deq;
        return outFifo.first;
    endmethod
endmodule 
```

> **练习 2（5 分）**： 在 `mkFftFolded` 中，创建一个折叠的 FFT 实现，总共只使用 16 个蝴蝶单元。这个实现应该在恰好 3 个周期内完成整个 FFT 算法（从出队输入 FIFO 到入队输出 FIFO）。
>
> Makefile 可用于构建 `simFold` 来测试此实现。编译并运行使用
>
> ```shell
> $ make fold
> $ ./simFold
> ```

> **练习 3（5 分）**： 在 `mkFftInelasticPipeline` 中，创建一个非弹性流水线 FFT 实现。这个实现应该使用 48 个蝴蝶单元和 2 个大型寄存器，

每个寄存器携带 64 个复数。这个流水线单元的延迟也必须恰好是 3 个周期，尽管其吞吐量将是每个周期 1 个 FFT 操作。
>
>Makefile 可用于构建 `simInelastic` 来测试此实现。编译并运行使用
>
>```shell
>$ make inelastic
>$ ./simInelastic
>```

> **练习 4（10 分）**：
>
> 在 `mkFftElasticPipeline` 中，创建一个弹性流水线 FFT 实现。这个实现应该使用 48 个蝴蝶单元和两个大型 FIFO。FIFO 之间的阶段应该在它们自己的规则中，这些规则可以独立触发。这个流水线单元的延迟也必须恰好是 3 个周期，尽管其吞吐量将是每个周期 1 个 FFT 操作。
>
> Makefile 可用于构建 `simElastic` 来测试此实现。编译并运行使用
>
> ```shell
> $ make elastic
> $ ./simElastic
> ```

## 讨论问题

在实验室存储库提供的文本文件 `discussion.txt` 中写下你对这些问题的回答。

> **讨论问题 1 和 2**：
>
> 假设你被给予一个执行 10 阶段算法的黑盒模块。你不能查看它的内部实现，但你可以通过给它数据并查看模块的输出来测试这个模块。你被告知它是按照本实验中涵盖的结构之一实现的，但你不知道是哪一个。
>
> 1. 你如何判断模块的实现是折叠实现还是流水线实现？ **（3 分）**
> 2. 一旦你知道模块具有流水线结构，你如何判断它是非弹性的还是弹性的？ **（2 分）**

> **讨论问题 3（可选）**： 你花了多长时间来完成这个实验？

当你完成所有练习并且代码工作正常时，提交你的更改到仓库，并将你的更改推送回源。

## 奖励

作为额外的挑战，实现讲座中最后几张可选幻灯片中介绍的多态超折叠 FFT 模块。这个超折叠 FFT 模块在给定有限数量的蝴蝶单元（1、2、4、8 或 16 个蝴蝶单元）的情况下执行 FFT 操作。蝴蝶单元数量的参数由 `radix` 给出。由于 `radix` 是一个类型变量，我们必须在模块的接口中引入它，因此我们定义了一个名为 `SuperFoldedFft` 的新接口，如下所示：

```bsv
interface SuperFoldedFft#(radix);
    method Action enq(Vector#(64, ComplexData inVec));
    method ActionValue#(Vector#(64, ComplexData)) deq;
endinterface
```

我们还必须在模块 `mkFftSuperFolded` 中声明 *provisos*，以通知 Bluespec 编译器 `radix` 和 `FftPoints` 之间的算术约束（即 `radix` 是 `FftPoints`/4 的一个因数）。

我们最终使用 4 个蝴蝶单元实例化了一个超折叠流水线模块，该模块实现了正常的 `Fft` 接口。这个模块将用于测试。我们还向你展示了将 `SuperFoldedFft#(radix, n)` 接口转换为 `Fft

` 接口的函数。

Makefile 可用于构建 `simSfol` 来测试此实现。编译并运行使用

```shell
make sfol
./simSfol
```

为了做超折叠 FFT 模块，首先尝试编写一个只有 2 个蝴蝶单元的超折叠 FFT 模块，没有任何类型参数。然后尝试推广设计以使用任意数量的蝴蝶单元。

------

© 2016 [麻省理工学院](http://web.mit.edu/)。版权所有。
