---
layout: page
title: 项目1：存储队列
permalink: /project1/
nav_order: 9
---

# 项目1: 存储队列

> 第一部分的项目没有明确的截止日期。s
>
> 然而，整个项目将在**12月14日，星期三下午3点EST**举行的项目展示时到期。

在最终项目的第一部分，我们将在实验7中设计的阻塞数据缓存（D$）中添加存储队列。

## 克隆项目代码

由于这是一个双人完成的项目，你需要首先联系我并提供你们小组成员的用户名。使用以下命令克隆你的Git仓库，其中`${PERSON1}`和`${PERSON2}`是你们的Athena用户名，并且`${PERSON1}`在字母顺序上排在`${PERSON2}`之前：

```
git clone /mit/6.175/groups/${PERSON1}_${PERSON2}/project-part-1.git project-part-1
```

## 改进阻塞缓存

只有对数据缓存实现存储队列才有意义，但我们希望保持指令缓存（I$）的设计与实验7中的相同。因此，我们需要分离数据缓存和指令缓存的设计。`src/includes/CacheTypes.bsv`包含了新的缓存接口，尽管它们看起来是相同的：

```bsv
interface ICache;
  method Action req(Addr a);
  method ActionValue#(MemResp) resp;
endinterface

interface DCache;
  method Action req(MemReq r);
  method ActionValue#(MemResp) resp;
endinterface
```

你将在`ICache.bsv`中实现你的I$，在`DCache.bsv`中实现你的D$。

### 实验7缓存设计的缺陷

在实验7中，缓存的`req`方法会检查标签数组，判断访问是缓存命中还是未命中，并执行处理两种情况所需的动作。然而，如果你查看实验7的编译输出，你会发现处理器的内存阶段规则与D$中替换缓存行、发送内存请求和接收内存响应的几条规则冲突。这些冲突是因为编译器无法准确判断当它们在你的处理器调用的`req`方法中被操作时，缓存的数据数组、标签数组和状态寄存器何时会被更新。

编译器还将内存阶段规则视为“更紧急”的，所以当内存阶段触发时，D$的规则不能在同一周期内触发。这种冲突不会影响缓存设计的正确性，但可能会损害性能。

### 解决规则冲突

为了消除这些冲突，我们在D$中添加了一个名为`reqQ`的单元素旁路FIFO。所有来自处理器的请求首先进入`reqQ`，在D$中得到处理后出队。更具体地说，`req`方法只是将传入的请求入队到`reqQ`中，我们将创建一个新规则，例如`doReq`，来完成原本在`req`方法中完成的工作（即从`reqQ`出队请求以便在没有其他请求的情况下进行处理）。

`doReq`规则的显式防护将使其与D$中的其他规则互斥，并消除这些冲突。由于`reqQ`是一个旁路FIFO，D$的命中延迟仍然是一个周期。

> **练习1（10分）：**将改进的D$（带旁路FIFO）集成到处理器中。以下是你需要做的简要概述：
>
> 1. 从实验7复制`Bht.bsv`到`src/includes/Bht.bsv`。
>
> 2. 在`src/

Proc.bsv`中完成处理器流水线。你可以用你在实验7中的`WithCache.bsv`中编写的代码来完成部分完成的代码。
>
>3. 在`src/includes/ICache.bsv`中实现I$。你可以直接使用实验7中的缓存设计。
>
>4. 在`src/includes/DCache.bsv`的`mkDCache`模块中实现改进的D$设计。
>
>5. 在`scemi/sim`文件夹下运行以下命令构建处理器：
>
> ```
> $ build -v cache
>   ```
>
> 这一次，你不应该看到与`mkProc`内部规则冲突相关的任何警告。
>
>6. 在`scemi/sim`文件夹下运行以下命令测试处理器：
>
> ```
> $ ./run_asm.sh cache
>   ```
>
> 和
>
> ```
> $ ./run_bmarks.sh cache
>   ```
>
> bluesim的标准输出将被重定向到`scemi/sim/logs`文件夹下的日志文件。对于新的汇编测试`cache_conflict.S`，IPC应该在0.9左右。如果你得到的IPC远低于0.9，那么你的代码中可能有错误。

> **讨论问题1（5分）：**即使每次循环迭代都有一个存储未命中，解释为什么汇编测试`cache_conflict.S`的IPC这么高。源代码位于`programs/assembly/src`。

## 添加存储队列

现在，我们将向D$添加存储队列。

### 存储队列模块接口

我们在`src/includes/StQ.bsv`中提供了一个参数化的*n*条目存储队列的实现。每个存储队列条目的类型就是`MemReq`类型，接口是：

```bsv
typedef MemReq StQEntry;
interface StQ#(numeric type n);
  method Action enq(StQEntry e);
  method Action deq;
  method ActionValue#(StQEntry) issue;
  method Maybe#(Data) search(Addr a);
  method Bool notEmpty;
  method Bool notFull;
  method Bool isIssued;
endinterface
```

存储队列与无冲突FIFO非常相似，但它具有一些独特的接口方法。

- `issue`方法：返回存储队列中最旧的条目（即`FIFO.first`），并在存储队列内设置一个状态位。后续对`issue`方法的调用将被阻塞，直到此状态位被清除。
- `deq`方法：从存储队列中移除最旧的条目，并清除由`issue`方法设置的状态位。
- `search(Addr a)`方法：返回存储队列中地址字段等于方法参数`a`的最年轻条目的数据字段。如果存储队列中没有写入地址`a`的条目，则该方法将返回`Invalid`。

你可以查看此模块的实现以更好地理解每个接口方法的行为。

### 插入到存储队列

设`stq`表示在D$内实例化的存储队列。如课堂上所述，来自处理器的存储请求应放入`stq`。由于我们在D$中引入了旁路FIFO`reqQ`，我们应该在从`reqQ`出队后将存储请求入队到`stq`。注意，存储请求不能在D$的`req`方法中直接入队到`stq`，因为这可能导致加载绕过较年轻存储的值。换句话说，所有来自处理器的请求仍然首先入队到`reqQ`。

还应该注意的是，将存储放入`stq`可以与几乎所有其他操作（如处理未命中）并行进行，因为存储队列的`

enq`方法被设计为与其他方法无冲突。

### 从存储队列发出

如果缓存当前没有处理任何请求，我们可以处理存储队列中最旧的条目或`reqQ.first`中的传入加载请求。来自处理器的加载请求应优先于存储队列。也就是说，如果`stq`有有效条目但`reqQ.first`有加载请求，那么我们处理加载请求。否则，我们调用`stq`的`issue`方法来获取最旧的存储以进行处理。

注意，当存储提交（即将数据写入缓存）时，才从存储队列中出队存储，而不是在处理开始时。这使我们能够实现一些稍后（但不在本节中）将实现的优化。`issue`和`dequeue`方法被设计为可以在同一规则中调用，以便我们在存储在缓存中命中时可以同时调用这两个方法。

还应该注意的是，当`reqQ.first`是存储请求时，不应阻塞从存储队列发出的存储。否则，缓存可能会死锁。

> **练习2（20分）：**在`src/includes/DCache.bsv`的`mkDCacheStQ`模块中实现带存储队列的阻塞D$。你应该使用`CacheTypes.bsv`中已定义的数值类型`StQSize`作为存储队列的大小。你可以通过在`scemi/sim`文件夹下运行以下命令来构建处理器：
>
> ```
> $ build -v stq
> ```
>
> 并通过运行以下命令来测试它：
>
> ```
> $ ./run_asm.sh stq
> ```
>
> 和
>
> ```
> $ ./run_bmarks.sh stq
> ```

为了避免由于编译器调度努力不足导致的冲突，我们建议将`doReq`规则分为两个规则：一个用于存储，另一个用于加载。

对于新的汇编测试`stq.S`，由于存储未命中的延迟几乎完全被存储队列隐藏，IPC应该在0.9以上。然而，你可能不会看到基准程序的任何性能改善。

## 在存储未命中下加载命中

尽管存储队列显著提高了汇编测试`stq.S`的性能，但它对基准程序没有任何影响。为了理解我们的缓存设计的局限性，让我们考虑一个情况：一个存储指令后跟一个加法指令，然后是一个加载指令。在这种情况下，存储将在缓存中开始处理，然后才发送加载请求到缓存。如果存储发生缓存未命中，即使加载可能在缓存中命中，加载也会被阻塞。也就是说，存储队列未能隐藏存储未命中的延迟。

为了在不过度复杂设计的情况下获得更好的性能，我们可以允许在*存储*未命中的同时发生加载命中。具体来说，假设`reqQ.first`是一个加载请求。如果缓存没有处理其他请求，我们当然可以处理`reqQ.first`。然而，如果存储请求正在等待尚未到达的来自内存的响应，我们可以尝试处理加载请求，检查它是否在存储队列或缓存中命中。如果加载在存储队列或缓存中命中，我们可以从`reqQ`中出队它，从存储队列转发数据或从缓存读取数据，并将加载的值返回给处理器。如果加载是未命中，我们不

采取进一步行动，只需将其保留在`reqQ`中。

注意，允许加载命中时没有结构冒险，因为待处理的存储未命中不访问缓存或其状态。我们还应注意，加载命中*不能*与*加载*未命中同时发生，因为我们不希望加载响应乱序到达。

为方便起见，我们在`CacheTypes.bsv`中定义的`WideMem`接口中添加了一个名为`respValid`的额外方法。当`WideMem`有响应可用时（即等于`WideMem`的`resp`方法的防护），此方法将返回`True`。

> **练习3（10分）：**在`src/includes/DCache.bsv`的`mkDCacheLHUSM`模块中实现允许在存储未命中下加载命中的带存储队列的阻塞D$。你可以通过在`scemi/sim`文件夹下运行以下命令来构建处理器：
>
> ```
> $ build -v lhusm
> ```
>
> 并通过运行以下命令来测试它：
>
> ```
> $ ./run_asm.sh lhusm
> ```
>
> 和
>
> ```
> $ ./run_bmarks.sh lhusm
> ```
>
> 你应该能看到一些基准程序性能的提升。

> **讨论问题2（5分）：**在未优化的汇编代码中，程序可能只是为了在下一条指令中读取而写入内存：
>
> ```bsv
> sw  x1, 0(x2)
> lw  x3, 0(x2)
> add x4, x3, x3
> ```
>
> 这经常发生在程序将其参数保存到栈上的子程序中。优化编译器（例如GCC）可以将寄存器的值保持在寄存器中以加快对这些数据的访问，而不是将寄存器的值写出到内存。这种优化编译器的行为如何影响你刚刚设计的内容？存储队列是否仍然重要？

> **讨论问题3（5分）：**与练习1和2中的缓存设计相比，你在每个基准的性能上看到了多少改进？

------

© 2016 [麻省理工学院](http://web.mit.edu/)。版权所有。
