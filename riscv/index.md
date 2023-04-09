---
date: 2023-04-05
title: RISC-V 特权架构学习
# linkTitle: 
featured: true
authors:
  - yang
series:
  - RISC-V 特权架构学习
categories:
  - rcore_study
tags:
  - RISC-V
nav_weight: 2
nav_icon:
  vendor: bs
  name: book
  color: deepskyblue
---

> 为了不让学习的战线拉太长，RISC-V 的部分就粗略地看了一下特权模式相关的部分，先进入操作系统的学习了。（主要是怕前面学了 Rust 马上又要忘了。。。）

## 2023-04-05 RISC-V特权指令级架构 特权级别

### 特权体系结构

#### 为什么需要分层
- 需要一些管理、保护共享资源和隔离实现细节的方法，于是将软件栈进行了清晰的分层。

|软件层级|与谁通信|通过什么通信|
|:-:|:-:|:-:|
|应用程序|AEE (**A**pplication **E**xecution **E**nvironment 应用程序执行环境)|ABI (**A**pplication **B**inary **I**nterface 应用程序二进制接口)|
|操作系统|SEE (**S**upervisor **E**xecution **E**nvironment 监管者执行环境)|SBI (**S**upervisor **B**inary **I**nterface 监管者二进制接口)|
|超级监管程序|HEE (**H**ypervisor **E**xecution **E**nvironment 超级监管者执行环境)|HBI (**H**ypervisor **B**inary **I**nterface 超级监管者二进制接口)|

> 以上名词翻译仅供参考。

- ABI 应用程序二进制接口，包含 user-level ISA 和一组用于与 AEE 互动的 ABI 调用 (ABI calls)。传统的操作系统中 AEE 是由 OS 提供的。
- SBI Supervisor 二进制接口，与 Supervisor 执行环境交互（SEE）。SBI 包括 user-level ISA 和 supervisor-level ISA，以及一组 SBI 功能调用（SBI function calls）。SEE 既可以是低端硬件平台上的简单 boot loader 和 BIOS-style IO 系统，也可以是高端服务器上由 hypervisor 提供的虚拟机，亦或者是架构模拟环境中主机操作系统上的轻型转换层（thin translation layer）。
- HBI Hypervisor 二进制接口，Hypervisor 上可以支持多个多程序OS。其中每个 OS 通过 SBI 与 hypervisor 通信，SEE 由 hypervisor 提供。hypervisor 通过 hypervisor 二进制接口（HBI）与 hypervisor 执行环境（HEE）通信，以对 hypervisor 隔离硬件平台的细节。

> RISC-V ISA的硬件实现通常还需要特权ISA之外的额外功能，以用于支持各种执行环境（AEE，SEE或HEE）。

#### 特权级别
RISC-V 将
- RISC-V 硬件线程 hart（**har**dware **t**hread）由一或多个 CSRs（**C**ontrol and **s**tatus **r**egisters）编码决定了其运行的权级模式。目前已经定义了三个权级，如下表所示：

|级别|编码|名称|缩写|
|:-:|:-:|:-:|:-:|
|0|00|用户(**U**ser)/应用(Application)权级|U|
|1|01|监管者(**S**upervisor)|S|
|2|10|保留||
|3|11|机器权级(**M**achine)|M|

> 在具体的硬件实现中，至少需要实现 Machine 模式，其它两种模式根据硬件功能进行组合。
> |配置|实现模式|安全保障|内存保护|其它|
> |:-:|:-:|:-:|:-:|:-:|
> |无保护的嵌入式硬件|M|需要所有程序可信|无|
> |带保护的嵌入式硬件|M + U|需要应用程序可信|硬件内存保护|
> |可运行类 Unix 操作系统的硬件|M + S + U|需要操作系统可信|虚拟内存 + 读写运行保护|
> |可运行云端操作系统的硬件|M + [V](S + U)|需要超级监管程序可信|2 级虚拟内存 + 读写运行保护|

- machine-level（M-mode）权级最高，也是 RISC-V 硬件必须实现的一个权级。在 M-mode 下运行的代码对硬件有完全的控制权。M-mode 可用于管理 RISC-V 上的安全执行环境（secure execution environment）。user-mode（U-mode）和supervisor-mode（S-mode）则分别用于传统应用程序以及操作系统。

- 每个权级都各有一组特权 ISA 的核心扩展，以及相关可选扩展和变体。例如，由 machine-mode 支持的一个关于内存保护的可选标准扩展。同样，也可通过扩展 supervisor-mode 来支持 type-2 hypervisor。
  - 共有的指令
    - ECALL  产生当前模式的 ecall 异常
    - EBREAK  产生断点异常
    - FENCE[.I]  同步更新到内存
	- \<x>RET  从指定模式的 trap 中返回
      - MRET  M-mode 是必须实现的
      - SRET  实现了 S-mode 才提供此指令
	  - URET  实现了 U-mode 才提供此指令
  - S-mode (+ M-mode)
    - SFENCE.VMA  同步更新到隐式获取的内存
  - M-mode
    - WFI  暂停当前硬件线程直到有中断需要处理

- CSRs 有自己的地址空间。
	- 每一个 hart 都有自己独立的 4K CSRs 集（每种模式占 1K）。
	- CSRs 需要通过专门的操作才能访问。
	- CSRs 对模式敏感，只能在恰当的或更高特权模式才能获取，在更低特权模式下访问将会产生 trap。

- hart 通常在 U-mode 下运行应用程序代码，直到遇到某些 trap（如 supervisor call 或者 timer interrupt）而迫使其切换至 trap handler（trap handler 通常在更高权级运行）。然后 hart 将运行 trap handler，并最终在造成 trap 的原指令 上/后 以 U-mode 恢复执行（resume execution）。
会提高权级的 trap 被称之为 vertical trap（垂直陷阱），而保持相同权级的 trap 则称之为 horizontal trap（水平陷阱）。RISC-V 提供了能灵活路由到各个不同权级层的 trap。

> 也可以用 vertical trap来实现 horizontal trap，只要将控制返回给特权模式更低的 horizontal trap handler。

#### 调试模式
- 在特权级的具体实现中还可以包含一个 debug mode，以支持片外调试/测试。debug mode（D-mode）可被看作为一个额外的特权模式，它的权限甚至比 M-mode 还多。分离出的 debug specification 中描述了debug mode 下 RISC-V hart 的操作。debug mode 保留了一些 CSR 地址，这些地址只能在 D mode 下访问，此外也可以在平台上保留一些物理地址空间。


## 2023-04-06 RISC-V特权指令级架构 机器模式、用户模式、监管者模式

### 机器模式
在 M 模式下运行的 hart 对内存，I/O 和一些对于启动和配置系统来说必要的底层功能有着完全的使用权。M 模式是所有标准 RISC-V 处理器都必须实现的权限模式。

#### 异常
- RISC-V 将异常分为两类： 同步异常、中断
- 同步异常：指令执行期间产生，如访问了无效的存储器地址或执行了具有无效操作码的指令时。
- 中断：与指令流异步的外部事件。
- M 模式运行期间可能发生的同步例外：
  1. 访问错误异常：当物理内存的地址不支持访问类型时发生（例如尝试写入 ROM）。
  2. 断点异常：在执行 ebreak 指令，或者地址或数据与调试触发器匹配时发生。
  3. 环境调用异常：在执行 ecall 指令时发生。
  4. 非法指令异常：在译码阶段发现无效操作码时发生。
  5. 非对齐地址异常：在有效地址不能被访问大小整除时发生，例如地址为 0x12 的 amoadd.w。
- 有三种标准的中断：软件中断、时钟中断和外部中断。

> 中断时 mcause 的最高有效位置 1，同步异常时置 0。并且 mcause 低有效位表示了中断/异常的具体原因。只有在实现了监管者模式时才能处理监管者模式中断和页面错误异常。


#### 机器模式下的异常处理
- 控制状态寄存器（CSR）
  - mtvec (Machine Trap Vector): 保存发生异常时处理器需要跳转到的地址
  - mepc (Machine Exception PC): 指向发生异常的指令
  - mcause (Machine Exception Cause): 指示发生异常的种类
  - mie (Machine Interrupt Enable): 指示处理器目前能处理和必须忽略的中断
  - mip (Machine Interrupt Pending): 指示目前正准备处理的中断
  - mtval (Machine Trap Value): 保存了陷入（trap）的附加信息：地址例外中出错
的地址、发生非法指令例外的指令本身，对于其他异常，它的值为 0
  - mscratch (Machine Scratch): 暂存一个字大小的数据
  - mstatus (Machine Status): 保存全局中断使能以及许多其它状态

- 处理器在 M 模式下运行时，只有在全局中断使能位 mstatus.MIE 置 1 时才会产生中断
- 默认情况下，发生所有异常（不论在什么权限模式下）的时候，控制权都会被移交到 M 模式的异常处理程序。

### 用户模式
由于 M 模式下程序代码可以自由地访问硬件平台，而我们有时候并不能信任所有的应用程序代码。因此，RISC-V 提供了保护系统免受不可信的代码危害的机制，并且为不受信任的进程提供隔离保护。这种限制机制是通过加入额外的权限模式：用户模式（U 模式）来实现的。在 U 模式下将无法执行 M 模式的特权指令（如 mret）和访问特权控制状态寄存器（如 mstatus），并且在尝试访问 M 模式指令或访问 CSR 时会产生非法指令异常。

- 实现了 M 和 U 模式的处理器具有 PMP（物理内存保护 Physical Memory Protection）功能，允许 M 模式指定 U 模式可以访问的内存地址，实现对 U 模式运行代码的内存访问限制。

> PMP 只能支持固定数量的内存区域保护，且这些区域必须在物理存储中连续。

### 监管者模式
S 模式的权限比 U 模式高，但是比 M 模式低。S 模式下运行的软件与 U 模式一样不能使用 M 模式的 CSR 和指令，并且受到 PMP 的限制。

#### 监管者模式下的异常处理
RISC-V 提供了一种异常委托机制。通过该机制可以选择性地将中断和同步异常交给 S 模式处理，而完全绕过 M 模式。
- medeleg（Machine Exception Delegation，机器异常委托）CSR 控制将哪些同步异常委托给 S 模式。
- mideleg（Machine Interrupt Delegation，机器中断委托）CSR 控制将哪些中断委托给 S 模式。
- S 模式有和 M 模式一样的系列异常处理 CSR：sepc, stvec, scause, scratch, stval, sstatus, sie 和 sip。它们的功能和 M 模式的 CSR 相同。

> 异常只在本权限模式处理或向更高权限模式委派，不会移交给权限更低的模式。

#### 基于页面的虚拟内存
S 模式提供了一种传统的虚拟内存系统，它将内存划分为固定大小的页来进行地址转换和对内存内容的保护。

- RISC-V 的分页方案以 SvX 的模式命名,其中 X 是以位为单位的虚拟地址的长度。
- RV32 的分页方案 Sv32 支持 4GiB 的虚址空间，这些空间被划分为 2^10 个 4 MiB 大小的巨页。每个巨页被进一步划分为 2^10 个 4 KiB 大小的基页（分页的基本单位）。因此，Sv32 的页表是基数为 2^10 的两级树结构。
- RV64 支持多种分页方案，最受欢迎的一种是 Sv39。Sv39 使用和 Sv32 相同的 4 KiB 大的基页，不过页表的高基数树变成了三层。页表项的大小变成 8 个字节，可以容纳 512 GiB 的物理地址空间。Sv39 将地址空间划分为 2^9 个 1 GiB 大小的吉页。每个吉页被进一步划分为 2^9 个巨页。在 Sv39 中这些巨页大小为 2 MiB，比 Sv32 中略小。每个巨页再进一步分为 2^9 个 4 KiB 大小的基页。
- S 模式的控制状态寄存器 satp（Supervisor Address Translation and Protection，监管者地址转换和保护）控制了分页系统。通常 M 模式的程序在第一次进入 S 模式之前会把零写入 satp 以禁用分页，然后 S 模式的程序在初始化页表以后会再次进行 satp 寄存器的写操作。
- S 模式添加了 sfence.vma 指令，它通知处理器，软件可能已经修改了页表，于是处理器可以相应地刷新地址转换缓存（用于优化页表访问性能的缓存）。

