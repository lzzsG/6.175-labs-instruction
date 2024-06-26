---
layout: page
title: 实验1：多路选择器和加法器
permalink: /Lab1/
nav_order: 1
---

# 实验 1: 多路选择器和加法器

> **实验 1 截止日期**： 2016年9月16日，星期五，晚上11:59:59 EDT。
>
> 实验 1 的交付物包括：
>
> - 在 `Multiplexer.bsv` 和 `Adders.bsv` 中对练习 1-5 的回答，
> - 在 `discussion.txt` 中对讨论问题的回答。

## 引言

在本实验中，你将从基本门原语构建多路选择器和加法器。首先，你将使用与门、或门和非门构建一个 1 位多路选择器。接下来，你将使用 for 循环编写一个多态性多路选择器。然后，你将转向加法器的工作，使用全加器构建一个 4 位加法器。最后，你将修改一个 8 位行波进位加法器，将其改为选择进位加法器。

本实验用作简单组合电路和 Bluespec SystemVerilog (BSV) 的引入。尽管 BSV 包含用于创建电路的高级功能，本实验将侧重于使用低级门来创建用于高级电路的块，例如加法器。这强调了 BSV 编译器生成的硬件。

### 多路选择器

多路选择器（简称 *muxes*）是在多个信号之间选择的块。多路选择器有多个数据输入 `inN`，一个选择输入 `sel`，和一个单一输出 `out`。`sel` 的值决定哪个输入显示在输出上。本实验中的多路选择器都是 2 路多路选择器。这意味着将有两个输入可以选择（`in0` 和 `in1`），并且 `sel` 是一个单一位。如果 `sel` 为 0，则 `out` = `in0`；如果 `sel` 为 1，则 `out` = `in1`。图 1a 显示了用于多路选择器的符号，图 1b 以图形方式显示了多路选择器的功能。

| ![多路选择器符号]({{ site.baseurl }}/docs/assets/mux.png) | ![多路选择器功能]({{ site.baseurl }}/docs/assets/mux-sel.png) |
| ----------------------------------- | --------------------------------------- |
| (a) 多路选择器符号                  | (b) 多路选择器功能                      |

图 1：1 位多路选择器的符号和功能

### 加法器

加法器是数字系统的基本构建块。有许多不同的加法器架构都可以计算相同的结果，但它们以不同的方式达到结果。不同的加法器架构在面积、速度和功耗方面也有所不同，并且没有一种架构在所有领域都超越其他加法器。因此，硬件设计师根据系统面积、速度和功耗约束选择加法器。

我们将探索的加法器架构是行波进位加法器和选择进位加法器。行波进位加法器是最简单的加法器架构。它由通过进位链连接的全加器块链组成。图 2b 中可以看到一个 4 位行波进位加法器。它非常小，但也非常慢，因为每个全加器必须等待前一个全加器完成后才能计算其位。

选择进位加法器为行波进位加法器添加了预测或推测以加快执行速度。它以与行波进位加法器相同的方式计算底部位，但计算顶部位的方式不同。它不等待来自下面位的进位信号计算，而是计算顶部位的两个可能结果：一个

假设下面的位没有进位，另一个假设有进位。一旦计算出进位位，多路选择器就会选择与进位位相对应的顶部位。图 3 中可以看到一个 8 位选择进位加法器。

| ![全加器]({{ site.baseurl }}/docs/assets/full-adder.png) | ![由全加器构建的 4 位行波进位加法器]({{ site.baseurl }}/docs/assets/ripple-carry.png) |
| ---------------------------------- | ------------------------------------------------------------ |
| (a) 全加器                         | (b) 由全加器构建的 4 位行波进位加法器                        |

| ![4 位加法器符号]({{ site.baseurl }}/docs/assets/full-adder-symbol.png) | ![8 位行波进位加法器]({{ site.baseurl }}/docs/assets/8-bit-ripple-carry.png) |
| ------------------------------------------------- | ------------------------------------------------------ |
| (c) 4 位加法器符号                                | (d) 8 位行波进位加法器                                 |

图 2：由全加器块构建的 4 位加法器和 8 位加法器

| ![8 位选择进位加法器]({{ site.baseurl }}/docs/assets/8-bit-carry-select.png) |
| ------------------------------------------------------ |

图 3：8 位选择进位加法器

### 测试台

已经编写了用于测试你的代码的测试台，测试台的链接包含在本实验的仓库中。文件 `TestBench.bsv` 包含多个可以使用提供的 `Makefile` 单独编译的测试台。`Makefile` 有每个模拟器可执行文件的目标，每个目标和可执行文件的使用在本手册中进行了解释。每个可执行文件在程序工作时打印出 `PASSED`，在程序遇到错误时打印出 `FAILED`。

以 `Simple` 结尾的测试台结构简化了，并且它们输出了来自单元测试期间单元的所有数据，以便你可以看到单元的工作情况。如果你有兴趣测试这些单元的自己的案例，你可以修改简单的测试台以输入你请求的值。普通测试台会为输入值生成随机数。

## 在 BSV 中构建多路选择器

构建我们的选择进位加法器的第一步是从门构建一个基本的多路选择器。让我们首先检查 `Multiplexer.bsv`。

```bsv
function Bit#(1) multiplexer1(Bit#(1) sel, Bit#(1) a, Bit#(1) b);
    return (sel == 0)? a: b;
endfunction
```

第一行开始定义一个名为 `multiplexer1` 的新函数。这个多路选择器函数接收几个参数，这些参数将用于定义多路选择器的行为。这个多路选择器操作单比特值，具体类型 `Bit#(1)`。稍后我们将学习如何实现多态函数，这些函数可以处理任何宽度的参数。

这个函数在其定义中使用了类似 C 的构造。如多路选择器这样简单的代码可以在高层次上定义，而不会带来实现上的惩罚。然而，由于硬件编译是一个复杂的多维问题，工具在它们可以执行的优化类型上是有限的。

`return` 语句构成了整个函数，它接收两个输入并使用 `sel` 选择其中之一。`endfunction` 关键字完成了我们多路选择器函数的定义。你应该能够编译该模块。

> **练习 1（4 分）**： 使用与门、或门和非门重新实现函数

 `multiplexer1` 在 `Multiplexer.bsv` 中。需要多少个门？（所需的函数分别称为 `and1`、`or1` 和 `not1`，在 `Multiplexers.bsv` 中提供。）

### 静态展开

现实世界系统中的许多多路选择器都大于 1 位宽。我们需要大于单比特的多路选择器，但手动实例化 32 个单比特多路选择器以形成一个 32 位多路选择器将是乏味的。幸运的是，BSV 提供了强大的静态展开构造，我们可以使用它来简化编写代码的过程。静态展开是指 BSV 编译器在编译时评估表达式的过程，使用结果来生成硬件。静态展开可以用几行代码表达极其灵活的设计。

在 BSV 中，我们可以使用方括号 (`[]`) 索引更宽 `Bit` 类型中的单个位，例如 `bitVector[1]` 选择 `bitVector` 中的第二个最低有效位（`bitVector[0]` 选择最低有效位，因为 BSV 的索引从 0 开始）。我们可以使用 for 循环来复制许多具有相同形式的代码行。例如，要聚合 `and1` 函数形成一个 5 位 `and` 函数，我们可以写：

```bsv
function Bit#(5) and5(Bit#(5) a, Bit#(5) b); Bit#(5) aggregate;
    for(Integer i = 0; i < 5; i = i + 1) begin
        aggregate[i] = and1(a[i], b[i]);
    end
    return aggregate;
endfunction
```

BSV 编译器在其静态展开阶段会用其完全展开的版本替换这个 for 循环。

```bsv
aggregate[0] = and1(a[0], b[0]);
aggregate[1] = and1(a[1], b[1]);
aggregate[2] = and1(a[2], b[2]);
aggregate[3] = and1(a[3], b[3]);
aggregate[4] = and1(a[4], b[4]);
```

> **练习 2（1 分）**： 使用 for 循环和 `multiplexer1` 完成函数 `multiplexer5` 在 `Multiplexer.bsv` 中的实现。
> 通过运行多路选择器测试台检查代码的正确性：
>
> ```shell
> $ make mux
> $ ./simMux
> ```
>
> 可以使用另一个测试台来查看单元的输出：
>
> ```shell
> $ make muxsimple
> $ ./simMuxSimple
> ```

### 多态性和高阶构造器

到目前为止，我们已经实现了两个版本的多路选择器函数，但可以想象需要一个 n 位多路选择器。如果我们不必完全重新实现多路选择器就能使用不同的宽度，那将是很好的。使用前一节中介绍的 for 循环，我们的多路选择器代码已经有些参数化，因为我们使用了常数大小和相同类型。我们可以通过使用 `typedef` 给多路选择器的大小起一个名字（`N`）来做得更好。我们的新多路选择器代码看起来像这样：

```bsv
typedef 5 N;
function Bit#(N) multiplexerN(Bit#(1) sel, Bit#(N) a, Bit#(N) b);
    // ...
    // 从 multiplexer5 中的代码，用 N（或 valueOf(N)）替换 5
    // ...
endfunction
```

typedef 使我们能够随意更改多路选择器的大小。`valueOf` 函数在我们的代码中引入了一个小细节：`N` 不是一个 `Integer` 而是一个 *数值类型*，必须在用于表达式之前转换为 `Integer`。尽管有所改进，我们的实现仍然

缺乏一些灵活性。所有多路选择器的实例必须具有相同的类型，我们仍然必须为每次想要新的多路选择器时产生新代码。然而在 BSV 中，我们可以进一步参数化模块以允许不同的实例具有实例特定的参数。这种模块是多态的，硬件的实现会根据编译时配置自动改变。多态性是 BSV 设计空间探索的本质。

真正的多态多路选择器可以从以下开始：

```bsv
// typedef 32 N; // 不需要
function Bit#(n) multiplexer n(Bit#(1) sel, Bit#(n) a, Bit#(n) b);
```

变量 `n` 代表多路选择器的宽度，替换了具体值 `N`（=32）。在 BSV 中，*类型变量*（`n`）以小写字母开头，而具体类型（`N`）以大写字母开头。

> **练习 3（2 分）**： 完成函数 `multiplexer_n` 的定义。通过将原始定义的 `multiplexer5` 只更改为：`return multiplexer_n(sel, a, b);` 来验证此函数的正确性。这种重新定义允许测试台在不修改的情况下测试你的新实现。

## 在 BSV 中构建加法器

现在我们将转向构建加法器。加法的基本单元是全加器，如图 2a 所示。这个单元将两个输入位和一个进位输入位相加，它产生一个和位和一个进位输出位。`Adders.bsv` 包含两个函数定义，描述了全加器的行为。`fa_add` 计算全加器的加法输出，`fa_carry` 计算进位输出。这些函数包含与第 2 讲中呈现的全加器相同的逻辑。

可以通过将 4 个全加器连接在一起制作一个操作 4 位数的加法器，如图 2b 所示。这种加法器架构被称为行波进位加法器，因为进位链的结构。为了生成这种加法器而不写出每个显式的全加器，可以使用类似于 `multiplexer5` 的 for 循环。

> **练习 4（2 分）**： 使用 for 循环正确连接所有使用 `fa_sum` 和 `fa_carry` 的代码来完成 `add4` 的代码。

通过连接 4 位加法器，可以构建更大的加法器，就像通过连接全加器构建 4 位加法器一样。`Adders.bsv` 包含两个使用 add4 和连接电路构建的加法器模块：`mkRCAdder` 和 `mkCSAdder`。注意，与此点到的其他加法器不同，这些加法器是作为模块而不是函数实现的。这是一个微妙但重要的区别。在 BSV 中，函数由编译器自动内联，而模块必须使用 '`<-`' 符号显式实例化。如果我们将 8 位加法器制作成一个函数，使用它的 BSV 代码的多个位置将实例化多个加法器。通过将其制作成一个模块，多个来源可以使用相同的 8 位加法器。

在模块 `mkRCAdder` 中包含了图 2d 所示的 8 位行波进位加法器的完整实现。可以通过运行以下命令进行测试：

```shell
make rca
./simRca
```

由于 `mkRCAdder` 是通过组合 `add4` 实例构建的，运行 `./simRCA` 也将测试 `add4`。可以使用另一个测试台来查看单元的输出：

```shell
$ make rcasimple
$ ./simRca

Simple
```

还有一个 `mkCSAdder` 模块，旨在实现图 3 所示的选择进位加法器，但其实现未包含。

> **练习 5（5 分）**： 在模块 `mkCSAdder` 中完成选择进位加法器的代码。使用图 3 作为所需硬件和连接的指南。可以通过运行以下命令测试此模块：
>
> ```shell
> $ make csa
> $ ./simCsa
> ```
>
> 可以使用另一个测试台来查看单元的输出：
>
> ```shell
> $ make csasimple
> $ ./simCsaSimple
> ```

## 讨论问题

> 在初始实验代码提供的文本文件 `discussion.txt` 中写下对这些问题的回答。
>
> 1. 你的一位多路选择器使用了多少个门？5 位多路选择器呢？写下 N 位多路选择器中门的数量的公式。**(2 分)**
> 2. 假设一个全加器需要 5 个门。8 位行波进位加法器需要多少个门？8 位选择进位加法器需要多少个门？**(2 分)**
> 3. 假设一个全加器需要 A 时间单位来计算其输出，一旦所有输入都有效，一个多路选择器需要 M 时间单位来计算其输出。用 A 和 M 表示，8 位行波进位加法器需要多长时间？8 位选择进位加法器需要多长时间？**(2 分)**
> 4. 可选：你花了多长时间来完成这个实验？

完成后，使用 git add 添加任何必要的文件，使用 `git commit -am "Final submission"` 提交更改，并用 `git push` 推送修改以进行评分。

------

© 2016 [麻省理工学院](http://web.mit.edu/)。版权所有。
