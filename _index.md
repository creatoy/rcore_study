---
title: "rCore OS 学习记录"
# linkTitle:
date: 2023-04-08
nav_weight: 1
nav_icon:
  vendor: bs
  name: book
  color: deepskyblue
series:
  - 开源操作系统训练营
categories:
  - rcore_study
tags:
  - rCore
  - Rust
  - RISC-V
# menu:
#   main:
#     weight: 100
#     params:
#       icon:
#         vendor: bs
#         name: book
#         color: '#e24d0e'
authors:
  - yang
---


> 这些是自己学习的时候的一些简单记录，必然会有错漏的地方，如果有人发现了希望指出，非常感谢。

> 希望自己随着学习的不断深入会出现新的理解，到时可以对前面理解错误/不清楚的地方进行修正和梳理。（但愿自己能坚持下去并且没忘了这一茬。）

## 一些链接:
- 关于 rCore 操作系统学习
  - [开源操作系统训练营](https://github.com/LearningOS)
  - [rCore 教程书](http://rcore-os.cn/rCore-Tutorial-Book-v3/)
  - [rCore 实验指导](https://learningos.github.io/rCore-Tutorial-Guide-2023S/)
- 关于 Rust
  - Rust 程序设计语言（[中文版](https://kaisery.github.io/trpl-zh-cn/title-page.html)）（[英文版](https://doc.rust-lang.org/stable/book/title-page.html)）
  - [Rust 语言圣经](https://course.rs)
  - 通过例子学 Rust （[中文版](https://rustwiki.org/zh-CN/rust-by-example/)）（[英文版](https://doc.rust-lang.org/rust-by-example/)）
  - [Rust 标准库文档](https://doc.rust-lang.org/std/index.html)
- 关于 RISC-V
  - [RISC-V 特权指令集架构](https://content.riscv.org/wp-content/uploads/2018/05/riscv-privileged-BCN.v7-2.pdf)
  - [RISC-V 手册：一本开源指令集的指南](http://riscvbook.com/chinese/RISC-V-Reader-Chinese-v2p1.pdf)
  - [RISC-V 汇编手册](https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md)

