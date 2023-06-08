# Lecture

## 数字逻辑与处理器基础小班辅导 20220605

---

刘雪枫 - 无 92

Copyright (C) Timothy Liu 2022-2023

许可证：[Creative Commons — 署名-相同方式共享 4.0 国际 — CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-Hans)

2022.06.05

---

## 引言

下半学期主要内容

+ MIPS 指令集
+ 汇编语言（MARS）
+ 处理器（MIPS）
  + 单周期
  + 多周期
  + 流水线
+ Cache

<font size="10">**强烈推荐：**</font>阅读[《计算机组成与设计：硬件/软件接口（原书第5版）》](https://item.jd.com/11729917.html)，讲解生动，深入浅出

## 试题（2021 期末模拟题）

### 一

#### 1

`(m + n - 1) t​`；Load-Use 冒险会阻塞流水线、分支和跳转指令会阻塞流水线

#### 2

Cache 与虚存：

+ 目的不同。Cache 是为了加快对物理内存的访问；虚拟存储器可以用于为进程分配内存，为进程提供独立的地址空间，控制进程对内存的访问权限，实现共享内存以及内存映射等
+ Cache 对软件是透明的，而虚拟内存是由操作系统管理的

如果使用 write through 策略则每次修改都需要将内容写到磁盘上，而磁盘读写时间开销很大。

#### 3

算术溢出、访存错误、未知指令、除零错误

> 给的往年解答里还加了各种冒险？

#### 4

参见 MAP 课（

以及课件：

xxx 会影响 xxx（CAI、指令数、……）

#### 5

`0x1234ABCD + 0x1FFFC = 0x1236ABC9`

`(0x1234ABCD + 0xFFFE0000) mod 0x100000000 = 0x1232ABCD`

> 答案给了两个可能的答案：
>
> `0x1232ABCD ~ 0x1236ABC9` 或 `0x1230ABD1 ~ 0x1238ABCD`
>
> 但是后者我没算出来 QAQ
>
> 后面那个问题不是很懂题问的是什么意思,大概是想让答出使用 `j`、`jr` 指令吧！
>
> 先使用 `bne` 指令判断是否相等，在后面插入 `j` 指令，如果相等则执行 `j` 指令，跳转到目标地址；否则 `bne` 指令将 `j` 指令跳过，继续向下执行。  
>
> ```asm
>        beq $t0, $t1, target
>        # ...
>     
> target:
>        # ...
> ```
>
> 变成：
>
> ```asm
>        bne $t0, $t1, continue
>        j target
> continue:
>        # ...
> 
> target:
>        # ...
> ```

#### 6

SRAM 比较快但容量小且贵，作为 Cache；DRAM 相对容量较大且便宜，作为主存。即 SRAM 作为 DRAM 的高速缓存。主存中经常访问的数据储存在 SRAM 中，其他暂时用不到的数据存储在容量较大的 DRAM 中。配合起来从外面看是个又大又快又便宜的存储系统。

#### 7

> 选自数逻课件：
>
> ||强迫性缺失|容量缺失|冲突缺失|命中时间|
> |:---:|:---:|:---:|:---:|:---:|
> |Cache 容量增加|-|下降|下降|上升|
> |Cache 行大小增加|下降|上升|上升|下降|
> |Cache 相联度增加|-|-|下降|上升|

Cache 行大小过大，则容量缺失和冲突缺失增加；过小则强迫性缺失增加，且命中时间增加，Cache 性能下降

### 二

#### 1

1. 块大小为 16B，最后 4 位是块内偏移。L1 Cache 每组大小 `16 * 8 = 128B`，共 `32 * 1024 / 128 = 256` 组，因此中间的 8 位是组号。前面的 20 位是 Tag。

   Tag：`0x4F4FA`；组号：`0x0A`。

   组号是 `0x0A = 10`，因此该组第一行的行号为 `8 * 10 = 80`。故行号为 `80 ~ 87`。  

2. `1 + 0.05 * 10 + 0.05 * 0.01 * 500 = 1.75` 个周期

3. 块大小为 4B，最后两位是块内偏移；直接映像，共 8 行，中间 3 位是行号

   访问顺序：`000000, 000011, 000100, 100000, 100001, 000101 `

   访问的 Cache 行号依次为：`000`、`000`、`001`、`000`、`000`、`001`；

   Tag 依次为：`0, 0, 0, 1, 1, 0`

   因此第 1、3、4 发生缺失，第 2、5、6 命中。命中率为 50%。

#### 2

1. `sra $s0, $s0, 1`，R 型指令

   > 一些同学的参考解答里是 `srl` 而非 `sra`

2. `0x100002`

   > 取 `while` 低 28 位并去掉最低的两位即可

3. `6` 次

   > 脑海里执行即可

4. 86ns

   > I1\~I2 执行 1 次、I3\~I8 执行 7 次、I9\~I10 执行 6 次、I11\~I13 执行 1 次、I14\~I15 执行 7 次、I16\~I17 执行 6 次

#### 3

1. 

   |       RegDst        |       ALUSrc        |      MemtoReg       |      RegWrite       |   Memwrite    |        branch         |
   | :-----------------: | :-----------------: | :-----------------: | :-----------------: | :-----------: | :-------------------: |
   |          x          |          1          |          x          |          0          |       1       |           0           |
   | `sw` 不需要写寄存器 | `sw` 是与立即数相加 | `sw` 不需要写寄存器 | `sw` 不需要写寄存器 | `sw` 需要内存 | 下一条指令地址是 PC+4 |

2. `B->F->L->O->P->F`、`300 + 50 + 150 + 300 + 10 + 50 = 860ns`

   > 可以看出，存储器的延时很长，可以粗略判断，`lw` 和 `sw` 之一在关键路径上。而 `lw` 比 `sw` 多了一个最后的写寄存器阶段，因此 `lw` 延时更长，为关键路径。

3. `1 / (860ns) = 1.16MHz`

4. 如下：

   `Branch = 0` 意味着分支指令失效

   |            | `add` | `sub` | `and` | `slt` | `or`  | `beq` | `lw`  | `sw`  |             `j`              |
   | :--------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--------------------------: |
   | Branch = 0 |   1   |   1   |   1   |   1   |   1   |   0   |   1   |   1   |              1               |
   |    备注    |       |       |       |       |       |       |       |       | `j` 由 `Jump` 信号控制，正常 |

   `ALUSrc = 1` 意味着 ALU 的第二个操作数一直是立即数。其中，`sw` 和 `lw` 的第二个操作数本来就是立即数，而 `j` 指令不需要 ALU，因此这三个指令可以正常执行

   |            | `add` | `sub` | `and` | `slt` | `or`  | `beq` | `lw`  | `sw`  |  `j`  |
   | :--------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
   | ALUSrc = 1 |   0   |   0   |   0   |   0   |   0   |   0   |   1   |   1   |   1   |
   |    备注    |       |       |       |       |       |       |       |       |       |

#### 4

1. 没有寄存器写操作。在对 `$t0` 进行读操作（但如果支持转发，则结果被来自 `I1` 指令计算出的 `$t0` 转发所代替）。

2. `I1 I2 nop I3 nop nop I4 I5 nop nop nop I6`

   > 注意写回寄存器堆的值需要至少隔两条指令才能被正确读取；`beq` 需要隔三条指令 <!-- 流水线 part2 P49 -->

3. I3、I4、I6

   > EX/MEM 转发相邻的指令，MEM/WB 转发相隔一条的指令。
   >
   > 转发失效导致 I3 错误；转发失效也会导致 I4 错误，但 I3 本来就错误了，即使转发成功也不会正确；I6 用到了 I3 的结果 `$t4` 也错误

4. 对调 I4 和 I5 即可

### 三

#### 1

1. `lwth`；

   > 延时：Instruction Memory -> Register File -> ALU -> 温度计 -> Register File
   >
   > `120 + 50 + 100 + 360 + 50 = 680ps`

2. >  `lwth ` 分支的状态转移图和 `lw` 指令完全相同，只是将 `MEM` 换成了温度计。只需要加一条和 `load` 分支几乎完全相同的 `lwth` 分支即可，区别只是 MEM 阶段变成了访问温度计。 
   >
   > `load` 和 `lwth` 需要 5 个状态，`store` 需要 4 个状态，`alu` 计算指令需要 4 个状态，分支指令需要 3 个状态。  
   >
   > 每个状态需要一个周期，因此平均 CPI 为：`0.25 * 5 + 0.1 * 4 + 0.05 * 5 + 0.5 * 4 + 0.1 * 3 = 4.2`
   >
   > 由于 CPU 频率是固定的，因此频率必须要满足最长的阶段所需时间，即 `lwth` 的取温度计阶段所需要的 `360ps`
   >
   > 因此平均单条指令耗时为 `360 * 4.2 = 1512ps`

3. 如下：

   1. `lw $t3, 0($zero)`
   2. 画图时，MEM 占两个周期，可表示为 MEM1 和 MEM2。ALU 运算的数据冒险仍然存在，可以转发解决；Load-Use 冒险仍然存在，需要 Stall 两个周期；分支跳转指令产生的控制冒险存在；MEM 需要两个周期，存在结构冒险（最后这一条往年解答中没有给出）

## 祝同学们考试顺利！

## 反馈二维码（求填！！！）

反馈问卷略。