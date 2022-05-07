# debug-log

## 问题

- 使用 `qemu-uintr` 执行测例代码 `uipi_sample.c`，发送方函数 `sender_thread` 理应被执行一次，但实际被执行两次。

- 执行 `senduipi` 时入栈的 `esp` 值与执行 `uiret` 时出栈的 `esp` 值不相等。

## 调试过程

### 定位异常代码

通过断点输出的方式，定位到 `esp` 值异常改变的位置。发现该处 `eip` 值也发生了异常改变。该处代码为

```c
// accel/tcg/cpu-exec.c
ret = tcg_qemu_tb_exec(env, tb_ptr);
```

即 qemu 动态执行用户程序的代码。

发生异常现象时，qemu 正在执行测例代码的 `uintr_handler` 函数。

### 查看 qemu 运行状态

进一步输出 qemu 此刻的运行状态，尤其是当前正在执行的 TranslationBlock 的状态。

可以发现，当前正在执行的 TranslationBlock 起始地址为 `0x4016f5` ，大小为 `0x55` 字节，指令条数为 23 条。

### 反汇编

对测例代码 `uipi_sample.c` 采用 `objdump` 反汇编，以下是一个关键片段。

```assembly
00000000004016f5 <uintr_handler>:
  4016f5:	55                   	push   %rbp
  4016f6:	48 89 e5             	mov    %rsp,%rbp
  4016f9:	50                   	push   %rax
  4016fa:	48 8d 45 10          	lea    0x10(%rbp),%rax
  4016fe:	48 89 45 f0          	mov    %rax,-0x10(%rbp)
  401702:	48 8b 45 08          	mov    0x8(%rbp),%rax
  401706:	48 89 45 e8          	mov    %rax,-0x18(%rbp)
  40170a:	c7 05 bc 5b 0d 00 01 	movl   $0x1,0xd5bbc(%rip)        # 4d72d0 <uintr_received>
  401711:	00 00 00 
  401714:	90                   	nop
  401715:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  401719:	c9                   	leave  
  40171a:	48 83 c4 08          	add    $0x8,%rsp
  40171e:	f3 0f 01 ec          	uiret  

0000000000401722 <sender_thread>:
  401722:	55                   	push   %rbp
  401723:	48 89 e5             	mov    %rsp,%rbp
  401726:	48 83 ec 20          	sub    $0x20,%rsp
  40172a:	48 89 7d e8          	mov    %rdi,-0x18(%rbp)
  40172e:	8b 05 a0 5b 0d 00    	mov    0xd5ba0(%rip),%eax        # 4d72d4 <uintr_fd>
  401734:	ba 00 00 00 00       	mov    $0x0,%edx
  401739:	89 c6                	mov    %eax,%esi
  40173b:	bf c4 01 00 00       	mov    $0x1c4,%edi
  401740:	b8 00 00 00 00       	mov    $0x0,%eax
  401745:	e8 c6 e8 04 00       	call   450010 <syscall>
  40174a:	89 45 f4             	mov    %eax,-0xc(%rbp)
  40174d:	83 7d f4 00          	cmpl   $0x0,-0xc(%rbp)
  401751:	79 19                	jns    40176c <sender_thread+0x4a>
  401753:	48 8d 05 ae 38 0a 00 	lea    0xa38ae(%rip),%rax        # 4a5008 <_IO_stdin_used+0x8>
  40175a:	48 89 c7             	mov    %rax,%rdi
  40175d:	e8 6e ea 00 00       	call   4101d0 <_IO_puts>
  401762:	bf 01 00 00 00       	mov    $0x1,%edi
  401767:	e8 a4 7c 00 00       	call   409410 <exit>
  40176c:	8b 45 f4             	mov    -0xc(%rbp),%eax
  40176f:	89 c6                	mov    %eax,%esi
  401771:	48 8d 05 a8 38 0a 00 	lea    0xa38a8(%rip),%rax        # 4a5020 <_IO_stdin_used+0x20>
  401778:	48 89 c7             	mov    %rax,%rdi
  40177b:	b8 00 00 00 00       	mov    $0x0,%eax
  401780:	e8 fb 85 00 00       	call   409d80 <_IO_printf>
  401785:	8b 45 f4             	mov    -0xc(%rbp),%eax
  401788:	48 98                	cltq   
  40178a:	48 89 45 f8          	mov    %rax,-0x8(%rbp)
  40178e:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  401792:	f3 0f c7 f0          	senduipi %rax
  401796:	90                   	nop
  401797:	8b 05 37 5b 0d 00    	mov    0xd5b37(%rip),%eax        # 4d72d4 <uintr_fd>
  40179d:	ba 00 00 00 00       	mov    $0x0,%edx
  4017a2:	89 c6                	mov    %eax,%esi
  4017a4:	bf c5 01 00 00       	mov    $0x1c5,%edi
  4017a9:	b8 00 00 00 00       	mov    $0x0,%eax
  4017ae:	e8 5d e8 04 00       	call   450010 <syscall>
  4017b3:	b8 00 00 00 00       	mov    $0x0,%eax
  4017b8:	c9                   	leave  
  4017b9:	c3                   	ret    
```

TranslationBlock 的起始地址加上大小，也就是 `0x4016f5 + 0x55 = 0x40174a`，得到 TranslationBlock 的终止地址。

这样，我们就能看出 qemu 此刻的 TranslationBlock 是哪一段了。

```assembly
00000000004016f5 <uintr_handler>:
  4016f5:	55                   	push   %rbp
  4016f6:	48 89 e5             	mov    %rsp,%rbp
  4016f9:	50                   	push   %rax
  4016fa:	48 8d 45 10          	lea    0x10(%rbp),%rax
  4016fe:	48 89 45 f0          	mov    %rax,-0x10(%rbp)
  401702:	48 8b 45 08          	mov    0x8(%rbp),%rax
  401706:	48 89 45 e8          	mov    %rax,-0x18(%rbp)
  40170a:	c7 05 bc 5b 0d 00 01 	movl   $0x1,0xd5bbc(%rip)        # 4d72d0 <uintr_received>
  401711:	00 00 00 
  401714:	90                   	nop
  401715:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  401719:	c9                   	leave  
  40171a:	48 83 c4 08          	add    $0x8,%rsp
  40171e:	f3 0f 01 ec          	uiret  

0000000000401722 <sender_thread>:
  401722:	55                   	push   %rbp
  401723:	48 89 e5             	mov    %rsp,%rbp
  401726:	48 83 ec 20          	sub    $0x20,%rsp
  40172a:	48 89 7d e8          	mov    %rdi,-0x18(%rbp)
  40172e:	8b 05 a0 5b 0d 00    	mov    0xd5ba0(%rip),%eax        # 4d72d4 <uintr_fd>
  401734:	ba 00 00 00 00       	mov    $0x0,%edx
  401739:	89 c6                	mov    %eax,%esi
  40173b:	bf c4 01 00 00       	mov    $0x1c4,%edi
  401740:	b8 00 00 00 00       	mov    $0x0,%eax
  401745:	e8 c6 e8 04 00       	call   450010 <syscall>
```

这段代码恰好由 23 条指令组成，与我们输出看到的 qemu 运行状态信息相吻合。

此外，由于这一基本块包含了部分 `sender_thread` 函数的指令，因此发送方函数 `sender_thread` 被执行两次的现象也得到了解释。

## 结论

qemu 在划分 TranslationBlock 的时候，并没有将 `uiret` 指令视为一个跳转语句。这导致上面这一整段指令均被视为一个基本块，而这显然是不合理的。

正是因为这整段指令被视为一个基本块，才导致 `esp` 值发生了预期之外的变化，且 `eip` 也跑飞了。

后续可以从 qemu 划分 TranslationBlock 的层面入手，修复这个问题。
