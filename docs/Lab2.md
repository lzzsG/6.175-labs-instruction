---
layout: page
title: 实验2：乘法器
permalink: /Lab2/
nav_order: 2
---

# 实验 2: 乘法器

> **实验 2 截止日期**： 2016年9月26日星期一，晚上11:59:59 EDT。
> 你的实验 2 交付物包括：
>
> - 在 `Multipliers.bsv` 和 `TestBench.bsv` 中对练习 1-9 的回答，
> - 在 `discussion.txt` 中对讨论问题 1-5 的回答。

## 引言

在本实验中，你将构建不同的乘法器实现，并使用提供的测试台模板的自定义实例进行测试。首先，你将使用重复加法实现乘法器。接下来，你将使用折叠架构实现一个布斯乘法器。最后，你将通过实现一个基-4 布斯乘法器来构建一个更快的乘法器。

这些模块的输出将通过测试台与 BSV 的 `*` 操作符进行比较以验证功能。

本实验的所有材料都在 git 仓库 `$GITROOT/lab2.git` 中。本实验中提出的所有讨论问题都应该在 `discussion.txt` 中回答。完成实验后，将你的更改提交到仓库并推送更改。

## 内置乘法

BSV 具有内置的乘法操作：`*`。它是有符号或无符号乘法，具体取决于操作数的类型。对于 `Bit#(n)` 和 `UInt#(n)`，`*` 操作符执行无符号乘法。对于 `Int#(n)`，它执行有符号乘法。就像 `+` 操作符一样，`*` 操作符假设输入和输出都是同一类型。如果你想从 n 位操作数得到一个 2n 位结果，你必须首先将操作数扩展为 2n 位值。

`Multipliers.bsv` 包含了对 `Bit#(n)` 输入进行有符号和无符号乘法的函数。两个函数都返回 `Bit#(TAdd#(n,n))` 输出。这些函数的代码如下所示：

> **注意**： `pack` 和 `unpack` 是内置函数，分别用于转换为和从 `Bit#(n)` 转换。

```bsv
function Bit#(TAdd#(n,n)) multiply_unsigned( Bit#(n) a, Bit#(n) b );
    UInt#(n) a_uint = unpack(a);
    UInt#(n) b_uint = unpack(b);
    UInt#(TAdd#(n,n)) product_uint = zeroExtend(a_uint) * zeroExtend(b_uint);
    return pack( product_uint );
endfunction

function Bit#(TAdd#(n,n)) multiply_signed( Bit#(n) a, Bit#(n) b );
    Int#(n) a_int = unpack(a);
    Int#(n) b_int = unpack(b);
    Int#(TAdd#(n,n)) product_int = signExtend(a_int) * signExtend(b_int);
    return pack( product_int );
endfunction
```

这些函数将作为你在本实验中的乘法器的功能基准进行比较。

## 测试台

本实验有两个参数化的测试台模板，可以很容易地用特定参数实例化，以测试两个乘法函数之间的对比，或测试一个乘法器模块与一个乘法器函数的对比。这些参数包括函数和模块接口。`mkTbMulFunction` 用相同的随机输入比较两个函数的输出，而 `mkTbMulModule` 比较测试模块（被测试设备或 DUT）和参考函数的输出。

以下代码展示了如何为特定的函数和/或模块实现测试台。

```bsv
(* synthesize *)
module mkTbDumb();
    function Bit#(16) test_function( Bit#(8) a, Bit#(8) b ) = multiply_unsigned( a, b );
    Empty tb <- mkTbMulFunction

(test_function, multiply_unsigned, True);
    return tb;
endmodule

(* synthesize *)
module mkTbFoldedMultiplier();
    Multiplier#(8) dut <- mkFoldedMultiplier();
    Empty tb <- mkTbMulModule(dut, multiply_signed, True);
    return tb;
endmodule
```

下面两行使用 `TestBenchTemplates.bsv` 中的测试台模板实例化特定的测试台。

```bsv
Empty tb <- mkTbMulFunction(test_function, multiply_unsigned, True);
Empty tb <- mkTbMulModule(dut, multiply_signed, True);
```

每个的第一个参数（`test_function` 和 `dut`）是要测试的函数或模块。第二个参数（`multiply_unsigned` 和 `multiply_signed`）是正确实现的参考函数。在这种情况下，参考函数是使用 BSV 的 `*` 操作符创建的。最后一个参数是一个布尔值，指示你是否希望输出详细信息。如果你只想让测试台打印 PASSED 或 FAILED，将最后一个参数设置为 `False`。

这些测试台（`mkTbDumb` 和 `mkTbFoldedMultiplier`）可以使用提供的 Makefile 轻松构建。要编译这些示例，你会为第一个写 `make Dumb.tb`，为第二个写 `make FoldedMultiplier.tb`。makefile 将产生可执行文件 `simDumb` 和 `simFoldedMultiplier`。要编译你自己的测试台 `mkTb<name>`，运行

```shell
make <name>.tb
./sim<name>
```

编译过程不会产生 `.tb` 文件，扩展名只是用来指示应使用哪个构建目标。

> **练习 1（2 分）**： 在 `TestBench.bsv` 中编写一个测试台 `mkTbSignedVsUnsigned`，测试 `multiply_signed` 是否产生与 `multiply_unsigned` 相同的输出。按上述描述编译此测试台并运行。 (即运行
>
> ```shell
> $ make SignedVsUnsigned.tb
> ```
>
> 然后
>
> ```
> $ ./simSignedVsUnsigned
> ```
>
> )

> **讨论问题 1（1 分）**： 在使用二的补码编码时，从硬件角度来看，无符号加法与有符号加法是相同的。根据测试台的证据，无符号乘法与有符号乘法是否相同？

> **讨论问题 2（2 分）**： 在 `mkTBDumb` 中排除以下行
>
> ```bsv
> function Bit#(16) test_function( Bit#(8) a, Bit#(8) b ) = multiply_unsigned( a, b );
> ```
>
> 并修改其余的模块，以便拥有
>
> ```bsv
> (* synthesize *)
> module mkTbDumb();
> Empty tb <- mkTbMulFunction(multiply_unsigned, multiply_unsigned, True);
> return tb;
> endmodule
> ```
>
> 将导致编译错误。原始代码是如何修复编译错误的？你也可以通过定义两个函数来修复错误，如下所示。
>
> ```bsv
> (* synthesize *)
> module mkTbDumb();
> function Bit#(16) test_function( Bit#(8) a, Bit#(8) b ) = multiply_unsigned( a, b );
> function Bit#(16) ref_function( Bit#(8) a, Bit#(8) b ) = multiply_unsigned( a, b );
> Empty tb <- mkTbMulFunction(test_function, ref_function, True);
> return tb;
> endmodule
> ```
>
> 为什么不需要两个函数定义？（即为什么 `mkTbMulFunction` 的第二个操作数可以有变量类型？）*提示：* 查看 `TestBenchTemplates.bsv` 中 `mkTbMulFunction` 的操作数类型。

## 通过重复加法实现乘法

### 作为组合函数

在 `Multipliers.bsv` 中，有一个用于计算乘法的函数框架代码，该函数通过重复加法来计算。由于这是一个函数，它必须代表一个组合电路。

> **练习 2（3 分）**： 填写 `multiply_by_adding` 的代码，使其能够使用重复加法在一个时钟周期内计算 a 和 b 的乘积。（你将在练习 3 中验证你的乘法器的正确性。）如果你需要一个加法器从两个 n 位操作数产生一个 (n+1) 位输出，请按照 `multiply_unsigned` 和 `multiply_signed` 的模型，先将操作数扩展为 (n+1) 位再相加。

> **练习 3（1 分）**： 在 `TestBench.bsv` 中填写测试台 `mkTbEx3` 来测试 `multiply_by_adding` 的功能。使用以下命令编译它：
>
> ```shell
> $ make Ex3.tb
> ```
>
> 并使用以下命令运行它：
>
> ```shell
> $ ./simEx3
> ```

> **讨论问题 3（1 分）**： 你实现的 `multiply_by_adding` 是有符号乘法器还是无符号乘法器？（注意：如果它既不符合 `multiply_signed` 也不符合 `multiply_unsigned`，那么它是错误的。）

### 作为顺序模块

使用重复加法乘以两个 32 位数需要三十一个 32 位加法器。这些加法器可能会根据你的目标和其余设计的限制占用大量面积。在讲座中，展示了重复加法乘法器的折叠版本，以减少乘法器所需的面积。折叠版本的乘法器使用顺序电路，通过每个时钟周期完成一个所需计算并将临时结果存储在寄存器中，来共享单个 32 位加法器。

在本实验中，我们将创建一个 n 位折叠乘法器。寄存器 `i` 将跟踪模块在计算结果中的进度。如果 `0 <= i < n`，则正在进行计算，规则 `mul_step` 应该在做工作并递增 `i`。有两种方法可以做到这一点。第一种方法是制作一个包含 if 语句的规则，如下所示：

```bsv
rule mul_step;
    if (i < fromInteger(valueOf(n))) begin
        // 做一些事情
    end
endrule
```

这个规则每个周期都运行，但只有当 `i < n` 时才做事情。第二种方法是制作一个带有 *保护* 的规则，如下所示：

```bsv
rule mul_step(i < fromInteger(valueOf(n)));
    // 做一些事情
endrule
```

这个规则不会每个周期都运行。相反，它只在其保护，`i < fromInteger(valueOf(i))`，为真时运行。虽然这在功能上没有区别，但在 BSV 语言的语义和编译器中有所不同。这种差异将在后续讲座中讨论，但在此之前，你应该在本实验的设计中使用保护。如果不这样做，你可能会遇到测试台因为运行超时而失败。

> **注意**： BSV 编译器防止多个规则在同一周期内触发，如果它们可能写入同一个寄存器（*有点类似...*）。BSV 编译器将规则 `mul_step` 视为每次触发时都写入 `i`。测试台中有一个规

则用于向乘法器模块提供输入，因为它调用了 `start` 方法，所以它也每次触发时都写入 `i`。BSV 编译器看到这些冲突的规则，并发出编译器警告，它将把一个规则视为比另一个更紧急，永远不会同时触发它们。它通常选择 `mul_step`，由于该规则每个周期都触发，它阻止了测试台规则向模块提供输入。

当 `i` 达到 n 时，结果已准备好读取，因此 `result_ready` 应返回 true。当调用动作值方法 `result` 时，`i` 的状态应增加 1 至 `n+1`。`i == n+1` 表明模块准备重新开始，因此 `start_ready` 应返回 true。当调用动作方法 `start` 时，模块中所有寄存器的状态（包括 `i`）应设置为正确的值，以便重新开始计算。

> **练习 4（4 分）**： 填写模块 `mkFoldedMultiplier` 的代码，以实现一个折叠的重复加法乘法器。
>
> 你能在不使用变量位移位移器的情况下实现它吗？在不使用动态位选择的情况下实现它吗？（换句话说，你能避免通过存储在寄存器中的值进行位移或位选择吗？）

> **练习 5（1 分）**： 填写测试台 `mkTbEx5` 以测试 `mkFoldedMultiplier` 的功能。如果你正确实现了 `mkFoldedMultiplier`，它们应产生相同的输出。运行它们，使用：
>
> ```shell
> $ make Ex5.tb
> $ ./simEx5
> ```

## 布斯乘法算法

重复加法算法适用于乘以无符号输入，但它无法乘以使用二进制补码编码的（负）数字。为了乘以有符号数字，你需要一个不同的乘法算法。

布斯乘法算法是一种适用于有符号二进制补码数的算法。该算法使用一种特殊的编码对其中一个操作数进行编码，使其能够用于有符号数字。这种编码有时被称为*布斯编码*。布斯编码的数字有时用 `+`、`-` 和 `0` 符号表示，例如：`0+-0b`。这种编码的数字类似于二进制数字，因为数字中的每个位置都代表同样的二的幂。*i* 位上的 `+` 表示 (+1) · 2^i，但 *i* 位上的 `-` 对应 (-1) · 2^i。

可以通过查看原始数字的当前位和前一位（较不重要的位）来逐位获得二进制数字的布斯编码。在编码最低有效位时，假定前一位为零。下表显示了转换到布斯编码的对应关系。

| 当前位 | 前一位 | 布斯编码 |
| :----- | :----- | :------- |
| 0      | 0      | 0        |
| 0      | 1      | +1       |
| 1      | 0      | -1       |
| 1      | 1      | 0        |

布斯乘法算法最好描述为使用乘数的布斯编码的重复加法算法。不是在添加 0 或添加被乘数之间切换，如重复加法中所做的那样，布斯算法根据乘数的布斯编码在添加 0、添加被乘数或减去被乘数之间切换。下面的例子显示了被乘数 *m* 通过将乘数转换为其布斯编码来乘以一个负数。

| -5 · m | = `1011b` · m                       |
| ------ | ----------------------------------- |
|        | = `-+0-b` · m                       |
|        | = (-m) · 2^3 + m · 2^2 + (-m) · 2^0 |
|        | = -8m + 4m - m                      |
|        | = -5m                               |

布斯乘法算法可以使用以下算法有效地在硬件中实现。这个算法假设一个 n 位被乘数 `m` 正在被一个 n 位乘数 `r` 乘。

```bsv
初始化：
    // 所有位宽 2n+1
    m_pos = {m, 0}
    m_neg = {(-m), 0}
    p = {0, r, 1'b0}

重复 n 次：
    let pr = p 的最低两位
    if ( pr == 2'b01 ): p = p + m_pos;
    if ( pr == 2'b10 ): p = p + m_neg;
    if ( pr == 2'b00 or pr == 2'b11 ): 无操作;

    算术右移 p 一位；

res = p 的最高 2n 位；
```

符号 `(-m)` 是 m 的二进制补码的逆。由于最负数在二进制补码中没有正数对应，这个算法不适用于 m = `10...0b`。因为这个限制，测试台已经修改，以避免在测试时使用最负数。

> **注意**： 这不是设计硬件的好方法。永远不要因为硬件无法通过而从测试台中删除测试。解决这个问题的一种方法是实现一个 (n+1)-位的布斯

乘法器来执行 n 位有符号乘法，方法是对输入进行符号扩展。如果你使用零扩展而不是符号扩展输入，你可以得到两个输入的 n 位无符号乘积。如果你添加一个额外的输入到乘法器中，允许你在符号扩展和零扩展输入之间切换，那么你就有了一个可以在有符号和无符号乘法之间切换的 32 位乘法器。这个功能对于有有符号和无符号乘法指令的处理器来说非常有用。

这个算法还使用了算术移位。这是为有符号数字设计的移位。当向右移位数字时，它将最高有效位的旧值移回到 MSB 位置以保持值的符号相同。在 BSV 中，当移动类型为 `Int#(n)` 的值时会进行算术移位。要对 `Bit#(n)` 进行算术移位，你可能需要编写类似于 `multiply_signed` 的函数。此函数将 `Bit#(n)` 转换为 `Int#(n)`，进行移位，然后再转换回来。

> **练习 6（4 分）**： 填写模块 `mkBooth` 的实现，实现一个折叠版本的布斯乘法算法：该模块使用参数化的输入大小 `n`；你的实现应该适用于所有 `n` >= 2。

> **练习 7（1 分）**： 填写测试台 `mkTbEx7a` 和 `mkTbEx7b` 来测试你的布斯乘法器选择的不同位宽。你可以用以下命令测试它们：
>
> ```shell
> $ make Ex7a.tb
> $ ./simEx7a
> ```
>
> 和
>
> ```shell
> $ make Ex7b.tb
> $ ./simEx7b
> ```

## 基-4 布斯乘法器

布斯乘法器的另一个优点是它可以通过一次执行原始布斯算法的两个步骤来有效地加速，这相当于每个周期完成两位部分和的加法。这种加速布斯算法的方法被称为基-4 布斯乘法器。

基-4 布斯乘法器在编码乘数时一次查看两个当前位。由于基-4 乘法器每次编码可以减少到不超过一个非零位的布斯编码，因此它能比原始的布斯乘法器运行得更快。例如，位 01 在之前（较不重要的）0位之后被转换为原始布斯编码的 `+-`。`+-` 表示 2^(i+1) - 2^i，等于 2^i，即 `0+`。下表显示了基-4 布斯编码的一个情况（你将在下一个讨论问题中填写其余的表格）。

| 当前位 | 前一位 | 原始布斯编码 | 基-4 布斯编码 |
| :----- | :----- | :----------- | :------------ |
| 00     | 0      |              |               |
| 00     | 1      |              |               |
| 01     | 0      | `+-`         | `0+`          |
| 01     | 1      |              |               |
| 10     | 0      |              |               |
| 10     | 1      |              |               |
| 11     | 0      |              |               |
| 11     | 1      |              |               |

> **讨论问题 4（1 分）**： 在 `discussion.txt` 中填写上表。基-4 布斯编码中不应有超过一个非零符号。

下面是一个基-4 布斯乘法器的伪代码：

```bsv
初始化：
    // 所有位宽 2n + 2
    m_pos = {msb(m), m, 0}
    m_neg = {msb(-m), (-m), 0}
    p = {0, r, 1'b0}

重复 n/2 次：
    let pr = p 的最低三位
    if ( pr == 3'b000 ): 无操作;
    if ( pr == 3'b001 ): p = p + m_pos;
    if ( pr == 3'b010 ): p = p + m_pos;
    if ( pr == 3'b011 ): p = p + (m_pos << 1);
    if ( pr == 3'b100 ): ...
        ... 根据表格填写剩余部分 ...

    算术右移 p 两位；

res = 去掉 p 的最高位和最低位；
```

> **练习 8（2 分）**： 填写模块 `mkBoothRadix4` 的实现，实现一个基-4 布斯乘法器。该模块使用参数化的输入大小 `n`；你的实现应该适用于所有*偶数* `n` >= 2。

> **练习 9（1 分）**： 填写测试台 `mkTbEx9a` 和 `mkTbEx9b` 为你的基-4 布斯乘法器测试不同的偶数位宽。你可以用以下命令测试它们：
>
> ```shell
> $ make Ex9a.tb
> $ ./simEx9a
> ```
>
> 和
>
> ```shell
> $ make Ex9b.tb
> $ ./simEx9b
> ```

> **讨论问题 5（1 分）**： 现在考虑将你的布斯乘法器进一步扩展到基-8 布斯乘法器。这就像在单个步骤中执行基-2 布斯乘法器的 3 个步骤。所有基-8 布斯编码是否可以像基-4 布斯乘法器那样只用一个非零符号来表示？你认为制作基-8 布斯乘法器还有意义吗？

> **讨论问题 6（可选）**： 你花了多久时间来完成这个实验？

当你完成所有练习并且代码工作正常时，提交你的更改到仓库，并将你的更改推送回源。

------

© 2016 [麻省理工学院](http://web.mit.edu/)。版权所有。
