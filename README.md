# usr-intr

### 项目描述

选题为 **[proj6-RV64N-user-level-interrupt](https://github.com/oscomp/proj6-RV64N-user-level-interrupt)** 。

探索运用新一代 Intel 硬件特性**用户态中断**（User Interrupt） ，搭建运行环境，设计新的 IPC 场景和框架，不断与传统 IPC 及 XPC、underbridge、skybridge 等新 IPC 进行性能比较并优化。

基于 qemu 的 uintr 模拟器实现源码：https://gitlab.eduxiji.net/quintr/qemu

uintr-linux-kernel：https://gitlab.eduxiji.net/quintr/uintr-linux-kernel

如需复现当前 qemu 的成果，可参考 qemu 仓库根目录下的 workinglog.md 中的部分内容，配置环境，编译 uintr-linux-kernel ，编译 qemu ，编译 uipi_sample.c ，并用 qemu 执行 uintr-linux-kernel ，运行 uipi_sample.c （当前还未能完全跑通）。之后我们会考虑将这部分的文档独立化与细致化，以便外界复现和参考。

如需复现当前 qemu 的成果，可参考 qemu 仓库根目录下的 workinglog.md 中的部分内容，配置环境，编译 uintr-linux-kernel ，编译 qemu ，编译 uipi_sample.c ，并用 qemu 执行 uintr-linux-kernel ，运行 uipi_sample.c （当前还未能完全跑通）。之后我们会考虑将这部分的文档独立化与细致化，以便外界复现和参考。

一些自己产出的 ppt 放在本仓库文件夹 `ppt` 下。

### 与导师的沟通情况

每周都有定时例会与导师进行沟通。

和导师建立了微信群。
### 工作规划

#### 第一步

受限于疫情影响，我们没办法拿到有 uintr 特性的物理机。为了方便后续工作开展，第一步我们将基于 qemu 实现支持 uintr 的模拟器。

预期产出：能在我们的 qemu 上跑 uintr-linux-kernel ，并正确通过 linux 实现的简单功能测例 uipi_sample.c 。

### 项目进度

#### 2022-4-9

完成了一些前期调研工作。

阅读了 XPC 、 underbridge 、skybridge 论文。

阅读与整理了 Intel Architecture Instruction Set Extensions  and Future Features 中与用户态中断相关的硬件规范，主要是第2章与第11章。详见 `ppt/uintr-intel-linux.pptx` 。

阅读与整理了 uintr-linux-kernel （基于 linux 运用 uintr 特性的内核版本）仓库中的 commits 。详见 `ppt/uintr-intel-kernel commit summary.pptx` 。

对之后希望实现的 IPC 场景与框架进行了简单的调研与设计，受限于当前工作重心，该部分暂没有成熟的想法。

#### 2022-4-16

编译了 Intel linux 和 qemu ，定位到二进制翻译的代码，定位到 MSR 寄存器定义的代码和部分中断代码。在 qemu 上跑 linux kernel 和用户态中断样例程序，目前可识别新添硬件指令（stui、senduipi等），未定义所需 MSR 寄存器，因此内核使用 `RDMSR` 和 `WRMSR` 读写失败，用户程序第一个系统调用失败获得对应输出结果。

详见 `ppt/2022-4-16.pptx`

#### 2022-4-23

qemu-uintr 完成 MSR 寄存器的定义和访问，正在实现不发送 IPI 的 senduipi 指令，主要困难是进行访存。初步调研了一些基于 linux 的传统 IPC 方式，使用工具 perf 进行了性能分析的上手，为之后的工作打下基础。

详见 `ppt/2022-4-23.pptx`

#### 2022-4-30

qemu-uintr 研究清楚了访存如何做，将 senduipi 指令实现到手册伪代码的第二个 if 。

详见 `ppt/2022-4-30.pptx`

#### 2022-5-7

实现了 senduipi 与 uiret 的执行逻辑，完成中断定位与识别。

详见 `ppt/worklog5-6.md` 与 `ppt/debug-log-5-5.md`

#### 2022-5-14

修复了一些 bug，现在可以完整跑通 `uipi_sample.c`。

对输出进行了反汇编分析，详情见`ppt/asm分析.md`

详见 `ppt/2022-5-14.pptx`

#### 2022-5-21

修复了一些bug,现在可以在多线程情况下正常结束程序, 栈偏移问题也得到了一定的解释, 对文档进行了分块整理。

详见`ppt/2022-5-21.pptx` , `qemu工作文档分块`



#### 2022-5-28

解决了cpu调度的问题, 最后总结问题为中断处理问题。 

构建了一个 ubuntu 文件系统，尝试在其中运行 Intel 的 uintr ipc benchmark。

详见`ppt/2022-5-28.pptx` , `调度问题5-28.md`, `how-to-build-a-ubuntu-rootfs.md`



#### 2022-6-4

实现了直接发中断的逻辑，修复了部分特权级判断的bug，详见`ppt/直接发中断6-4.md`。

跑通 intel ipc-bench，取得了初步的性能测试结果，并对性能瓶颈进行了分析。

编写了`qemu-tutorial.md`主要面向手把手修改qemu代码并进行调试, 详见`qemu-tutorial.md`

详见`ppt/2022-6-4.pptx`



#### 2022-6-11

修补bug, 多线程情况下跑通`uintrfd-uni`

新增5个测试程序, 均测试通过

修补bug, 使得`uintrfd-bi` 能够高概率直接发送中断, 性能大幅度提升。

测试了基于共享内存的`uintrfd-bi`, 新能依旧碾压传统方法。
