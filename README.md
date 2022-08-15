# usr-intr

### 项目描述

选题为 **[proj6-RV64N-user-level-interrupt](https://github.com/oscomp/proj6-RV64N-user-level-interrupt)** 。

探索运用新一代 Intel 硬件特性**用户态中断**( User Interrupt )，搭建运行环境，设计新的 IPC 场景和框架，不断与传统 IPC 及 XPC、underbridge、skybridge 等新 IPC 进行性能比较并优化, 我们的结果表明, 利用用户态中断来进行进程间通信, 因其不需要进过内核, 从而性能相比传统进程间通信有数量级上的优势。此外我们还通过将用户态中断和linux io_uring子系统结合, 以用户态中断作为io完成的通知机制, 实现了双向的异步调用过程, 在保证异步基本的特性的情况下获取了更低的io延迟。

为了更好更方便地调试代码, 我们**基于qemu实现了x86的用户态中断**的支持, 可以让更多没有支持用户态中断硬件设备的人可以尝试用户态中断的特性。同时在修改qemu代码的过程中, 我们还总结qemu代码的框架和部分实现的细节, 制作为[**qemu tutorial**](https://github.com/OS-F-4/qemu-tutorial), 实现了代码级别的qemu教程, 帮助更多的人了解qemu的原理和实现。

一些产出的 ppt 放在本仓库文件夹 `ppt` 下, 主要内容为前期调研, 每周进度汇报, debug日志, 程序输出log, 问题探索和解决过程等。

我们的[**最终完整报告**](https://github.com/OS-F-4/usr-intr/blob/main/%E6%9C%80%E7%BB%88%E6%8A%A5%E5%91%8A.md), 位于仓库的根目录下, 命名为`最终报告.md`或`最终报告.pdf`。



# 成果请看这里

| 成果                           | github地址                                              | 详细说明以及使用                                             | gitlab地址                                                   | 详细说明以及使用                                             |
| ------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 实现用户态中断的qemu           | https://github.com/OS-F-4/qemu-uintr                    | https://github.com/OS-F-4/usr-intr/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/qemu.md | https://gitlab.eduxiji.net/quintr/qemu                       | https://gitlab.eduxiji.net/quintr/report/-/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/qemu.md |
| 实现用户态中断的linux内核      | https://github.com/OS-F-4/uintr-linux-kernel            | https://github.com/OS-F-4/usr-intr/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/linux-kernel.md | https://gitlab.eduxiji.net/quintr/uintr-linux-kernel/-/tree/uintr-next | https://gitlab.eduxiji.net/quintr/report/-/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/linux-kernel.md |
| ipc-bench性能测试              | https://github.com/OS-F-4/ipc-bench/tree/linux-rfc-v1   | https://github.com/OS-F-4/usr-intr/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/ipc-bench.md | https://gitlab.eduxiji.net/quintr/ipc-bench                  | https://gitlab.eduxiji.net/quintr/report/-/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/ipc-bench.md |
| 基于io_uring的异步系统调用内核 | https://github.com/OS-F-4/uintr-linux-kernel/tree/uring | https://github.com/OS-F-4/usr-intr/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/linux-uring.md | https://gitlab.eduxiji.net/quintr/uintr-linux-kernel/-/tree/uring | https://gitlab.eduxiji.net/quintr/report/-/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/linux-uring.md |
| 异步系统调用用户程序           | https://github.com/OS-F-4/uring                         | https://github.com/OS-F-4/usr-intr/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/uring.md | https://gitlab.eduxiji.net/quintr/uring                      | https://gitlab.eduxiji.net/quintr/report/-/blob/main/ppt/%E5%B1%95%E7%A4%BA%E6%96%87%E6%A1%A3/uring.md |
| qemu tutorial                  | https://github.com/OS-F-4/qemu-tutorial                 | https://github.com/OS-F-4/qemu-tutorial                      | https://gitlab.eduxiji.net/quintr/qemu-tutorial              | https://gitlab.eduxiji.net/quintr/qemu-tutorial              |







### 与导师的沟通情况

每周都有定时例会与导师进行沟通。

和导师建立了微信群。
### 工作规划

#### 第一步

受限于疫情影响，我们没办法拿到有 uintr 特性的物理机。为了方便后续工作开展，第一步我们将基于 qemu 实现支持 uintr 的模拟器。

预期产出：能在我们的 qemu 上跑 uintr-linux-kernel ，并正确通过 linux 实现的简单功能测例 uipi_sample.c 。

#### 第二步

进行性能测试，基于 Linux RFC 报告使用的 ipc-bench 作测试，比较 uintr 与 pipe等传统 IPC 方式的性能异同。自行编写 uintr+shmem 或其他运用 uintr 形式的用户测例，与传统 IPC 方式作比较。

预期产出：复现并得到比 Linux RFC 更丰富的性能测试结果，验证 IPC 框架设计的可行性。

#### 第三步

修改 Linux 内核，为 IPC 框架提供更通用的系统调用接口，并通过测试。

预期产出：一个初步成型的原型系统。

#### 第四步

寻找能发挥我们系统优势的有意义应用场景，在调度、负载均衡等方面继续改进 IPC 框架和内核。

预期产出：一篇有价值或影响力的学术性论文



## 项目进度

#### 2022-4-9

完成了一些前期调研工作。

阅读了 XPC 、 underbridge 、skybridge 论文。

阅读与整理了 Intel Architecture Instruction Set Extensions  and Future Features 中与用户态中断相关的硬件规范，主要是第2章与第11章。详见 `ppt/uintr-intel-linux.pptx` 。

阅读与整理了 uintr-linux-kernel （基于 linux 运用 uintr 特性的内核版本）仓库中的 commits 。详见 `ppt/uintr-intel-kernel commit summary.pptx` 。

对之后希望实现的 IPC 场景与框架进行了简单的调研与设计，受限于当前工作重心，该部分暂没有成熟的想法。阶段性想法产出如下（感谢张译仁学长）：

对 XPC 和 uintr 进行了比较分析，提出 uintr+shmem 实现的 IPC 框架，详见 `ppt/初步设想.pdf` 。

对 uintr 的实际应用场景和会遇到的问题进行了简单分析，详见 `ppt/应用场景.md` 。

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

将 linux 内核的终端正确挂载到 `/dev/ttyS0` 上，从而能够跑起其它传统形式的 ipc 测例程序。相关参考资料 [1](https://unix.stackexchange.com/questions/529935
)、[2](https://stackoverflow.com/questions/36529881)

#### 2022-7-9

与蚂蚁沟通协调，可以使用蚂蚁的物理机了。

与贾越凯学长交流，了解网络请求性能优化的大致思路。

初步调研 Nginx 对网络请求的处理流程。

见 `ppt/利用uintr优化网络服务性能.pptx`

### 2022-7-23

和intel沟通，发现他们也在做io_uring的优化。

qemu调试，单核，不打开sqpoll可以收到中断，打开sqpoll后无法发出中断，可能是内核线程没有注册sender导致，正在解决（将注册sender的函数在内核线程初始化的地方进行调用和尝试）

之后还有编写用户程序以及设计benchmark 的问题

负载变化剧烈的情况

物理机器还没消息，intel不给，

### 2022-7-30

完成异步内核的编写, 编写了相关了用户程序, 并用cgo编写了基于协程的程序

go调度和操作系统调度的独立性还不太清楚。



### 2022-8

获得物理机的支持, 经过努力成功将内核安装到物理机器, 并重新测试了ipc-bench的结果, 同时测试了异步IO的结果。

编写技术报告, 文档, 以及最终报告。