# rCore OS 学习记录

> 这些是自己学习的时候的一些简单记录，必然会有错漏的地方，如果有人发现了希望指出，非常感谢。

> 希望自己随着学习的不断深入会出现新的理解，到时可以对前面理解错误/不清楚的地方进行修正和梳理。（但愿自己能坚持下去并且没忘了这一茬。）

* [第一阶段学习](#1)
  * [Rust 语言](#1.1)
  * [RISC-V 处理器](#1.2)
* [第二阶段学习](#2)
  * [rCore OS](#2.1)

## 一些链接:
- 关于 rCore 操作系统学习
  - [开源操作系统训练营](https://github.com/LearningOS)
- 关于 Rust
  - Rust 程序设计语言（[中文版](https://kaisery.github.io/trpl-zh-cn/title-page.html)）（[英文版](https://doc.rust-lang.org/stable/book/title-page.html)）
  - [Rust 语言圣经](https://course.rs)
  - 通过例子学 Rust （[中文版](https://rustwiki.org/zh-CN/rust-by-example/)）（[英文版](https://doc.rust-lang.org/rust-by-example/)）
  - [Rust 标准库文档](https://doc.rust-lang.org/std/index.html)
- 关于 RISC-V
  - [RISC-V 特权指令集架构](https://content.riscv.org/wp-content/uploads/2018/05/riscv-privileged-BCN.v7-2.pdf)
  - [RISC-V 手册：一本开源指令集的指南](http://riscvbook.com/chinese/RISC-V-Reader-Chinese-v2p1.pdf)
  - [RISC-V 汇编手册](https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md)


<h2 id="1">第一阶段学习</h2>

<h3 id="1.1">Rust 语言</h3>

> 很多概念看一遍记不住，在学和练的过程中经常要回头看之前的章节才能想起来。

> Rust 的文档和教程中的代码片段都可以在线运行，每一个都可以运行看一下结果，加深印象和理解。

[2023-03-27 类型系统、流程控制、模式匹配等基本概念](rust/20230327.md)

[2023-03-29 集合类型 HashMap](rust/20230329.md)

[2023-03-30 泛型、枚举和 Options, Result](rust/20230330.md)

[2023-03-31 错误处理](rust/20230331.md)

[2023-04-01 泛型练习、特征、生命周期和自动化测试](rust/20230401.md)

[2023-04-02 函数式语言功能：闭包和迭代器](rust/20230402.md)

[2023-04-04 智能指针](rust/20230404.md)

[2023-04-05 多线程、宏](rust/20230405.md)

---

<h3 id="1.2">RISC-V 处理器</h3>

[2023-04-05 RISC-V特权指令级架构 特权级别](riscv/20230405.md)

[2023-04-06 RISC-V特权指令级架构 机器模式、用户模式、监管者模式](riscv/20230406.md)

---

<h2 id="2">第二阶段学习</h2>

<h2 id="2.1">rCore</h2>

> 还未开始。。。
