# linux-uring

## 项目简介

本项目基于linux的io_uring子系统, 利用用户态中断的硬件特性作为io完成的通知机制, 实现io的异步提交和io完成的异步返回的双向异步过程, 预期在达到io和计算重叠这样基本的异步框架下更低io延迟。

我们的内核程序位于[仓库](https://github.com/OS-F-4/uintr-linux-kernel/tree/uring)。我们的用户程序以及测试程序位于[仓库](https://github.com/OS-F-4/uring)。

详细的原理说明详见我们的[技术分享报告](https://github.com/OS-F-4/usr-intr/blob/main/%E6%8A%80%E6%9C%AF%E4%BA%A4%E6%B5%81%E6%8A%A5%E5%91%8A.pptx)。以及最终报告

## 使用方法





