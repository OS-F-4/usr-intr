# usr-intr

### 项目描述

探索运用新一代 Intel 硬件特性**用户态中断**（User Interrupt） ，搭建运行环境，设计新的 IPC 场景和框架，不断进行性能比较和优化。

基于 nimbos 实现源码：https://github.com/OS-F-4/nimbos-uintr

基于 qemu 的 uintr 模拟器实现源码：https://github.com/OS-F-4/qemu-uintr

一些自己产出的 ppt 放在本仓库文件夹 `ppt` 下。

### 项目进度

#### 2022-4-16

编译了 Intel linux 和 qemu ，定位到二进制翻译的代码，定位到 MSR 寄存器定义的代码和部分中断代码。在 qemu 上跑 linux kernel 和用户态中断样例程序，目前可识别新添硬件指令（stui、senduipi等），未定义所需 MSR 寄存器，因此内核使用 `RDMSR` 和 `WRMSR` 读写失败，用户程序第一个系统调用失败获得对应输出结果。