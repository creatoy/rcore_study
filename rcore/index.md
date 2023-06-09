---
date: 2023-04-07
title: rCore OS 实验
# linkTitle: 
# noindex: false
featured: true
authors:
  - yang
series:
  - rCore OS
categories:
  - rcore_study
tags:
  - 操作系统
nav_weight: 3
nav_icon:
  vendor: bs
  name: book
  color: deepskyblue
---

[rCore 操作系统教程书](http://rcore-os.cn/rCore-Tutorial-Book-v3/index.html)

[实验指导书](https://learningos.github.io/rCore-Tutorial-Guide-2023S/index.html)

[操作系统 slides](https://www.yuque.com/xyong-9fuoz/qczol5/glemuu?)

[我的实验仓库](https://github.com/LearningOS/2023s-rcore-creatoy)


## 2023-04-07 搭建实验环境

### 实验环境配置

> 实验环境搭建建议跟着教程中的来，自己搭建别的版本可能会出一些不太好解决的问题。比如我这里用了 7.2 的 QEMU 配合新版本的 RustSBI 后，运行实验程序的时候表现就有一些差异，为了降低查错成本最终还是老实换回了 7.0。（不过编译 QEMU 7.0 也出现了个小问题，将在下面补充）

### QEMU 7.2 不兼容实验使用的 RustSBI 版本

前面的工具安装都没有什么问题，最后试运行的时候由于使用的 RustSBI 和最新版本 QEMU（使用的是 7.2，教程里是 7.0）不兼容，启动后会卡死无输出。这个问题在 rCore 教程主仓库中有人反馈过：[使用rustsbi-qemu教育版规避rustsbi某些版本与qemu不兼容导致卡死的问题](https://github.com/rcore-os/rCore-Tutorial-v3/issues/110)

这个问题可以更新 rustsbi 解决。方法如下：

1. 克隆 rustsbi-qemu 仓库
```sh
git clone https://github.com/rustsbi/rustsbi-qemu.git
```

> 实验时 clone 下来的 repo 具体对应的 commit 是 [a4f0bbe44d9f2f1069a9e5becd09f291e542852c](https://github.com/rustsbi/rustsbi-qemu/tree/a4f0bbe44d9f2f1069a9e5becd09f291e542852c)

2. 修改 rustsbi-qemu/xtask/src/main.rs 文件中的 let target = "xxx" 一行将 "" 内目标改为 riscv64gc-unknown-none-elf
```sh
cd rustsbi-qemu
vim ./xtask/src/main.rs
```

3. 构建新的 rustsbi-qemu，最终生成的 rustsbi-qemu.bin 文件在 rustsbi-qemu/target/riscv64gc-unknown-none-elf/release/ 目录下。
```sh
cargo qemu
```

4. 将上面生成的 rustsbi-qemu.bin 文件复制到 rcore 实验项目目录的 bootloader 下。如果不想覆盖原本的文件就改一下名字，并且修改 rcore 实验项目 os 里的 Makefile，将 BOOTLOADER := ../bootloader/$(SBI)-$(BOARD).bin 一行修改指向新的文件名。（最好不要覆盖老的文件，因为有可能实验做到后面发现新的 rustsbi 跟教程不兼容了，到时候还得换回来:P）
```sh
cp <rusbsbi-qemu_dir>/target/riscv64gc-unknown-none-elf/release/rustsbi-qemu.bin <rcore_lab_dir>/bootloader/rustsbi-qemu-new.bin
vim <rcore_lab_dir>/os/Makefile
```

5. 最后再按照实验指导书中的步骤运行程序就可以看到 Hello, world! 输出了。
```sh
make run
```

### 编译 QEMU 7.0 时出现错误
编译 QEMU 7.0 的时候，由于宿主机器的 ebpf 版本较高，bpf_program__set_socket_filter 函数被弃用了，需要改成 bpf_program__set_type。[参考](https://gitlab.com/qemu-project/qemu/-/commit/a495eba03c31c96d6a0817b13598ce2219326691)
具体在我的环境中是修改 qemu/ebpf/ebpf_rss.c 的 52 行：
```c
//    bpf_program__set_socket_filter(rss_bpf_ctx->progs.tun_rss_steering_prog);
//  把上面这行修改成下面这样
    bpf_program__set_type(rss_bpf_ctx->progs.tun_rss_steering_prog, BPF_PROG_TYPE_SOCKET_FILTER);
```

## 2023-04-11 应用程序与基本执行环境

> 搞了好久 Hugo，把学习记录放到 [Github Pages](https://blog.creatio.top/docs/rcore_study/) 了。

### 构建无 OS 应用的准备工作

> 在项目目录下创建 .cargo/config 文件，用于配置编译/链接参数。可以省去每次构建时手动传递参数。
>
> 在这里就把编译目标平台指定为 riscv64gc-unknown-none-elf，不用每次构建都写目标了。只需要加入以下内容：
> ```toml
> [build]
> target = "riscv64gc-unknown-none-elf"
> ```

1. 移除标准库 std 依赖，只使用 core 核心库

```rust
use core::xxx;
#![no_std]
```

2. 添加 panic 处理函数
```rust
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
  loop {}
}
```

3. 移除 main 入口，并定义 _start 入口

```rust
#![no_main]

#[no_mangle]
extern "C" fn _start() {
    loop {};
}
```

> #[no_mangle] 标记表示不对下面的标识符进行修改，保留原样。（Rust 在编译的过程中默认会对标识符进行修改，这些标识符就会变得很复杂，这里因为要在别的地方使用 _start 标识符，所以要标记一下告诉编译器不要修改这个标识符，不然就没法找到这里了。）

### 系统调用

操作系统为应用程序提供了一些系统调用，这样应用程序不需要知道系统细节也可以通过这些系统调用使用系统功能。

### 第一章实验问题
教程书第一章书中的代码在 panic 处理中调用了 shutdown，而在 shutdown 中又调用了 panic 函数，这样就导致 QEMU 运行起来后一直循环打印 Panic 的内容。（ecall 没有让 QEMU 关闭退出。）
看了仓库提供的程序，发现里面的 shutdown 是调用了别的汇编指令而不是 ecall 让 QEMU 退出的：（参考原始代码后的简化版）

```rust
// QEMU exit.
unsafe {
    core::arch::asm!("sw {0}, 0({1})", in(reg)0x13333, in(reg)0x100000);
}

```

> 嘀嘀咕咕：实际上前两章没有实验，只要看懂程序就行了，后面的实验都提供了框架代码，这里搞不搞都行。（实际上是因为我嫌麻烦，不想搞了:P）


## 2023-04-19 批处理系统

> 嘀嘀咕咕：摸鱼好多天，该继续学习了。赶紧攀实验。（今天还把钥匙落公司了，又跑回去拿钥匙:(）

### 批处理系统
批处理系统可以一个接一个地自动运行不同的程序，使系统可以长时间不间断工作。

### Rust 取链接脚本中的符号地址
在链接脚本中定义的符号可以通过 extern "C" 的函数指针显式转换成 usize（usize 的长度是和硬件架构相关的），这样就得到了它们的地址。如下面这段 bss 清零的程序：
```rust
extern "C" {
    fn sbss(); // 链接脚本中 bss 段的起始地址
    fn ebss(); // 链接脚本中 bss 段的结束地址
}

unsafe {
    core::slice::from_raw_parts_mut(sbss as usize as *mut u8, ebss as usize - sbss as usize).fill(0);
}
```

> 嘀嘀咕咕：emmm，又没看多少内容--

## 2023-04-21 继续批处理系统

> 线上课学习

RISC-V 异常：
![exception](exception.png)

> 其中**断点**(Breakpoint) 和**执行环境调用**(Environment call) 两种异常是通过在上层软件中执行一条特定的指令触发的：执行 ebreak 这条指令之后就会触发断点异常；而执行 ecall 这条指令时候则会随着 CPU 当前所处特权级而触发不同的异常。这种有意为之的指令又被称为**陷入**或 **trap** 类指令。

Supervisor 的特权指令：
![supervisor privilage instruction](supervisor_priv_ins.png)

典型的程序布局相对内存布局：
![program](program_layout.png)

在函数调用时各寄存器的功能描述：
![register](register.png)

用户态（U）陷入到 S 态时相关的 CSR（控制状态寄存器）：
![supervisor_csr](supervisor_csr.png)

> 各个模式都有自己对应的 CSR，当发生特权级切换后由硬件自动对这些寄存器写入相应的数值（除了 stvec，我们要提前将它的值设置到 trap 处理函数的地址）。在这里我们只需要关心 S 特权级的 CSR 就行了，因为由 S 态陷入到 M 态时的处理由 RustSBI 完成。

> 默认情况下异常发生后是会陷入 M 态，但是可以通过设置异常代理模式，让异常发生后转入到 S 态来处理。

> 在批处理系统 Trap 处理汇编函数中，__alltraps 紧接着的就是 __restore 符号，也就是执行完了 trap_handler 紧接着就是开始恢复 Trap 上下文了。
> ```asm
> # 具体位置在 trap.S 的 9, 10 行
> .globl __alltraps
> .globl __restore
> ```


## 2023-04-24 多道程序与分时多任务

> 默认情况下，在软件开始响应中断前，硬件会自动禁用所有同特权级中断，所以不会触发同特权级中断导致嵌套中断（但是没法避免高特权级的中断发生）。[RISC-V 架构中的中断](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#risc-v)
