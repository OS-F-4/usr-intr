

[toc]



# 对输出进行反汇编分析


```shell
# 添加icount 的版本
Sending IPI from sender thread index:0 
[    5.884737] rdmsrl misc 5
qemu: caught 0xf30fc7 SENDUIPI eip:0x4017bc #在sender_thread print之后第一条指令
 --------------


0x4017bc  block size:46  icont:12
[    5.886608] rdmsrl misc 5
0x485e45  block size:12  icont: 3 # 在__clock_nanosleep
0x4017bc  block size:92  icont:20
qemu:helper senduipi called receive  regidx:240, uipiindex: 0
qemu: data of uitt valid:1 user_vec:0  UPID address 0xffff9cf1c2b32ac0 
qemu: content of upid:  status:0x0    nv:0xec    ndst:0x100    0x0000000000000000
qemu: data write back in upid:  status:0x1    nv:0xec    ndst:0x100    0x0000000000000001
0x450050  block size:46  icont:12 # 在syscall
0x450050  block size:92  icont:20
[    5.888169] rdmsrl misc 5
[    5.888670x485e45  block size:26  icont:10
!!! interrupt 2  intno:236 
recognize uintr
the physical address of APIC 0x10   the EOI content: 0xf000ff53f000ff53
rrnzero called handler: 0x4016f5  rr: 0x1
qemu:origin exp 0x7ffd801e2860   eip 0x485e45  eflags: 0x202
qemu:move statck 0x7ffd801e27e0
qemu:after align statck 0x7ffd801e27e0
qemu:push finish now esp is: 0x7ffd801e27c0qemu: eip: 0x4016f5
2] uintr_unregister_sender called
[    5.891230] send: unregister sender uintrfd 3 for task=79 ret 0
0x44f36f  block size: 2  icont: 1 # 在__libc_write
0x475649  block size: 8  icont: 2 # __pthread_disable_asynccancel
0x401734  block size: 1  icont: 1 # uintr_handler 中写函数的后一条指令（将uintr_received改为1）
--------------


qemu:caught 0xf30f01ec UIRET  # 注意！ pc: 0x401759 已经进入sender_thread
before:  pc_start: 0x401755  sc_base:0   pc: 0x401759  pc.next:0x401755  rip:0x401734
helper uiret called, now eip: 0x401734 # eip 还在uintr_handler （将uintr_received改为1）
qemu: now esp is: 0x7ffd801e2760
qemu:poped values:uirrv:0x1 rip:0x485e45   eflags:0x202  rsp:0x7ffd801e2860 
pc_start: 0x401755  sc_base:0   pc: 0x401759  rip:0x485e45 # 希望返回的eip值在nanosleep中
-------------


[    5.889107] rdmsrl misc 5
[    5.893860] debug!
[    5.894780] rdmsrl 1
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
0x45006d  block size:14  icont: 6 # syscall
0x4017ea  block size: 8  icont: 2 # sender_thread
0x41801a  block size: 7  icont: 3 # jmp  417ed3 <start_thread+0x193>  start_thread中
0x417ed3  block size: 5  icont: 1 # start_thread中 417 到 418 都是
0x417eec  block size:19  icont: 3
0x417a00  block size: 5  icont: 1
0x417b43  block size:22  icont: 5
0x417ef1  block size: 1  icont: 1
0x421c40  block size: 5  icont: 1 # __libc_thread_freeres
0x421c52  block size:18  icont: 4
0x47b440  block size: 6  icont: 1 #__res_thread_freeres
0x47db00  block size:10  icont: 3
0x47db60  block size:38  icont:11
0x47b44a  block size: 9  icont: 6
0x47b45c  block size:18  icont: 5
0x421c58  block size: 2  icont: 2 # __libc_thread_freeres
0x4205b0  block size:22  icont: 3
0x420678  block size:13  icont: 3
0x421c6e  block size: 1  icont: 1
0x421c84  block size:22  icont: 3
0x421c84  block size:10  icont: 2 
0x421c84  block size:92  icont:20
0x421c8e  block size:10  icont: 2
0x48c8d0  block size: 6  icont: 1 # __libc_dlerror_result_free
0x48c908  block size:21  icont: 6
0x421c94  block size: 2  icont: 2
0x421c9e  block size:10  icont: 2
0x4206b0  block size: 9  icont: 2
0x420738  block size:31  icont: 8
0x42079c  block size:24  icont: 5
0x417ef6  block size: 5  icont: 4
0x417f08  block size:18  icont: 3
0x417f27  block size:31  icont: 6
0x417f55  block size: 2  icont: 2
0x417f6e  block size:25  icont: 5
0x417f8b  block size:29  icont: 6
0x417f99  block size:14  icont: 3
0x417f9e  block size: 5  icont: 1
0x417f9e  block size:92  icont:20
0x417fc7  block size:41  icont:10
0x41808b  block size:12  icont: 2
0x450150  block size:20  icont: 4 # __madvise
0x45015b  block size: 3  icont: 2
0x450163  block size: 8  icont: 2
0x41809f  block size: 1  icont: 1
0x417fd3  block size: 5  icont: 1
0x417fe5  block size:18  icont: 3
0x417ffb  block size:14  icont: 3
0x417ffb  block size:11  icont: 4 # start_thread
[    5.889107] sample2[78]: segfault at 7f29aa395640 ip 00007f29aa395640 sp 00007ffd801e28f8 error 15
0x417ffb  block size: 3  icont: 2
0x417ffb  block size:10  icont: 3
[    5.889107] Code: 00 00 a0 e0 4a 00 00 00 00 00 a0 e6 4a 00 00 00 00 00 a0 ef 4a 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 <40> 56 39 aa 29 7f 0f
[    5.889107] recv: Release uintrfd for r_task 78 uvec 0
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
qemu:wrmsr pd 0x0000000000000000
qemu:wrmsr RR 0x0
qemu:wrmsr stackadjust 0x0
qemu:wrmsr handler 0x0000000000000000
0x587ead  block size: 2  icont: 2 # 可能是shell程序
0x49596a  block size: 2  icont: 2
0x49596a  block size:92  icont:20
0x5882b6  block size: 8  icont: 2
0x5882cb  block size: 6  icont: 2
0x5882cb  block size: 2  icont: 1
0x5882cb  block size:10  icont: 3
0x5882cf  block size: 4  icont: 2
0x5882cf  block size: 5  icont: 2
0x5882cf  block size:92  icont:20
0x5882d4  block size: 5  icont: 2
0x5882d9  block size: 5  icont: 2
0x5882d9  block size:92  icont:20
0x452950  block size:12  icont: 3
0x452a60  block size:29  icont: 7
0x452a6e  block size:14  icont: 3
0x452a95  block size:10  icont: 2
0x452a9c  block size:63  icont:17
0x452980  block size:21  icont: 4
0x42ec80  block size: 5  icont: 1
0x452985  block size:11  icont: 3
0x45298a  block size: 5  icont: 2
0x452990  block size: 6  icont: 2
0x4529a3  block size:19  icont: 5
0x4e3de0  block size:74  icont:17
0x422fff  block size:27  icont: 6
0x4233cb  block size:63  icont:17
0x4233d8  block size:13  icont: 3
0x5882e5  block size:10  icont: 3
0x5881c6  block size:18  icont: 5
0x58821c  block size:31  icont: 9
0x448157  block size:92  icont:19
0x448157  block size:14  icont: 4
0x448157  block size: 5  icont: 1
0x448165  block size:14  icont: 4
0x448173  block size:14  icont: 4
0x44819a  block size:18  icont: 5
0x588278  block size:16  icont: 4
0x58827f  block size: 7  icont: 2
0x58829b  block size:23  icont: 6
0x5882f7  block size:10  icont: 4
0x588356  block size:14  icont: 5
0x588698  block size: 9  icont: 4
0x5883e5  block size:31  icont: 7
0x4b9837  block size: 1  icont: 1
0x445b2e  block size:63  icont:17
0x445bc8  block size:28  icont: 7
0x588401  block size:13  icont: 2
0x588406  block size: 5  icont: 1
0x448c1c  block size:63  icont:17
0x448c1c  block size:48  icont: 9
0x448c1c  block size: 1  icont: 1
0x448c4c  block size:48  icont: 9
0x5886b7  block size:14  icont: 2
0x588518  block size: 5  icont: 1
0x5875d4  block size: 9  icont: 2
0x5875d8  block size: 4  icont: 2
0x5875e5  block size: 5  icont: 2
0x44c1cc  block size:63  icont:17
0x44c043  block size:35  icont: 7
0x592fe0  block size:20  icont: 3
0x5e9037  block size:15  icont: 3
0x44f80b  block size:63  icont:17
0x44f784  block size:13  icont: 2
0x58c86e  block size:63  icont:17
0x58c885  block size:12  icont: 3
0x44f80b  block size:25  icont: 5
0x5876b0  block size:12  icont: 3
0x58ee73  block size:63  icont:17
0x5ed66e  block size:63  icont:17
0x4bb63a  block size:10  icont: 3
0x5ed74f  block size: 8  icont: 2
0x44d932  block size:37  icont: 8
0x44de60  block size:10  icont: 2
0x44de60  block size:11  icont: 3
0x44de60  block size:92  icont:20
0x44de6f  block size:11  icont: 3
0x44de7f  block size:16  icont: 5
0x44de91  block size: 6  icont: 2
0x44affe  block size:16  icont: 4
0x44dea1  block size:17  icont: 5
0x44df7f  block size:18  icont: 4
0x44df1e  block size:18  icont: 4
0x44d502  block size:18  icont: 4
0x52f2be  block size:21  icont: 9
0x52f2c9  block size:92  icont:20
0x4bb460  block size:10  icont: 3
0x4bb4d4  block size: 1  icont: 1
0x4bb4fd  block size:10  icont: 3
0x496beb  block size:10  icont: 3
0x531127  block size: 1  icont: 1
0x4b967b  block size:11  icont: 7
0x4bbb6b  block size:10  icont: 3
0x52eff3  block size: 8  icont: 2
0x44f80b  block size:63  icont:17
0x44affe  block size:13  icont: 2
0x4b9d42  block size:31  icont:10
0x496beb  block size:10  icont: 3
0x445992  block size: 1  icont: 1
0x4459a1  block size:92  icont:20
0x42e6f6  block size:20  icont: 3
0x4b9837  block size: 1  icont: 1
QEMU: Terminated
```





## 一次比较好的结果

```shell

--------------
qemu:caught 0xf30f01ec UIRET
before:  pc_start: 0x401764  sc_base:0   pc: 0x401768  pc.next:0x401764  rip:0x401743
helper uiret called, now eip: 0x401743
qemu: now esp is: 0x7ffec0070290
qemu:poped values:uirrv:0x1 rip:0x485e85   eflags:0x202  rsp:0x7ffec0070390 
pc_start: 0x401764  sc_base:0   pc: 0x401768  rip:0x485e85
-------------


[   11.615969] uintr_register_sender called
qemu:wrmsr tt 0xffff9d63413ac001
[   11.615969] rdmsrl 2
qemu:wrmsr misc 0x000000ec00000100
[   11.615969] send: register sender task=78 flags 0 ret(uipi_id)=0
[   11.615969] rdmsrl misc 5
[   11.615969] rdmsrl misc 5
qemu:helper senduipi called receive  regidx:240, uipiindex: 0
qemu: data of uitt valid:1 user_vec:0  UPID address 0xffff9d6342c26cc0 
qemu: content of upid:  status:0x0    nv:0xec    ndst:0x0    0x0000000000000000
qemu: data write back in upid:  status:0x1    nv:0xec    ndst:0x0    0x0000000000000001
[   11.615969] uintr_unregister_sender called
[   11.615969] send: unregister sender uintrfd 3 for task=78 ret 0
- - - - user: now in the handler function
- - - - Sending IPI from sender thread index:0 
[   11.615969] debug!
[   11.629945] rdmsrl 1
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
[   11.615969] rdmsrl 1
qemu:wrmsr misc 0x000000ec00000000
qemu:wrmsr tt 0x0000000000000000
[   11.615969] rdmsrl misc 5
[   11.615969] rdmsrl misc 5
- - - - user receive uintr, joining thread
[   11.615969] traps: sample[78] general protection fault ip:401907 sp:7ffec0070400 error:0 in sample[401000+a4000]
[   11.658274] recv: Release uintrfd for r_task 78 uvec 0
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
qemu:wrmsr pd 0x0000000000000000
qemu:wrmsr RR 0x0
qemu:wrmsr stackadjust 0x0
qemu:wrmsr handler 0x0000000000000000
[   11.666518] sample (78) used greatest stack depth: 14624 bytes left
Segmentation fault
/ # [   32.669644] rcu: INFO: rcu_sched detected stalls on CPUs/tasks:
```





捕捉到SENDUIPI之前，执行后面几次的输出如下：

```
[  304.087150] uintr_register_handler called
qemu:wrmsr handler 0x0000000000401de5
qemu:wrmsr pd 0xffff9c00c38795c0
qemu:wrmsr stackadjust 0x0000000000000080
qemu:rdmsr misc 0x0000000000000000
qemu:wrmsr misc 0x000000ec00000000
[  304.087617] recv: register handler task=84 flags 0 handler 401de5 ret 0
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
regeister returned 0
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
[  304.089302] uintr_create_fd called
[  304.089550] recv: Alloc vector success uintrfd 3 uvec 0 for task=84
qemu:rdmsr misc 0x000000ec00000000
create fd returned 3 qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
Receiver enabled interrupts
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
[  304.097607] uintr_register_seqemu:rdmsr misc 0x000000ec00000000
nder called
qemu:rdmsr misc 0x000000ec00000000
qemu:wrmsr tt 0xffff9c00c19d3001
qemu:rdmsr misc 0x0000000000000000
qemu:wrmsr misc 0x0000000000000100
[  304.1004qemu:rdmsr misc 0x000000ec00000000
05] send: register qemu:rdmsr misc 0x000000ec00000000
sender task=85 flags 0 ret(uqemu:rdmsr misc 0x000000ec00000000
ipi_id)=0
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
Sending IPI from sender thread
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
qemu:rdmsr misc 0x000000ec00000000
[  304.104475] traps: uipi_samplqemu:rdmsr misc 0x000000ec00000000
e[85] trap invalid opcode ip:401eb7 sqemu:rdmsr misc 0x000000ec00000000
p:7f6056609d90 error:0 in uipi_sample[401000+af000]
qemu:rdmsr misc 0x000000ec00000000
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
qemu:wrmsr pd 0x0000000000000000
qemu:wrmsr RR 0x0000000000000000
qemu:wrmsr stackadjust 0x0000000000000000
qemu:wrmsr handler 0x0000000000000000
[  304.113500] recv: Release uintrfd for r_task 84 uvec 0
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
qemu:wrmsr pd 0x0000000000000000
qemu:wrmsr RR 0x0000000000000000
qemu:wrmsr stackadjust 0x0000000000000000
qemu:wrmsr handler 0x0000000000000000
Illegal instruction
```





```
在senduipi后设置答应eip
--------------
qemu: caught 0xf30fc7 SENDUIPI eip:0x4017bc
 --------------


qemu:helper senduipi called receive  regidx:240, uipiindex: 0
qemu: data of uitt valid:1 user_vec:0  UPID address 0xffff9febc2710b80 
qemu: content of upid:  status:0x2    nv:0xec    ndst:0x0    0x0000000000000000
qemu: data write back in upid:  status:0x2    nv:0xec    ndst:0x0    0x0000000000000001
    6.174750] rdmsrl misc 5
0x485e45 
!!! interrupt 2  intno:236 
recognize uintr
the physical address of APIC 0x10   the EOI content: 0xf000ff53f000ff53
rrnzero called handler: 0x4016f5  rr: 0x1
qemu:origin exp 0x7fff285ec040   eip 0x485e45  eflags: 0x202
qemu:move statck 0x7fff285ebfc0
qemu:after align statck 0x7fff285ebfc0
qemu:push finish now esp is: 0x7fff285ebfa0qemu: eip: 0x4016f5
[    6.175069] uintr_unregister_sender called
[    6.174921] rdmsrl misc 5
0x44f36f 
0x475649 
0x401734 
--------------


qemu:caught 0xf30f01ec UIRET
before:  pc_start: 0x401755  sc_base:0   pc: 0x401759  pc.next:0x401755  rip:0x401734
helper uiret called, now eip: 0x401734
qemu: now esp is: 0x7fff285ebf40
qemu:poped values:uirrv:0x1 rip:0x485e45   eflags:0x202  rsp:0x7fff285ec040 
pc_start: 0x401755  sc_base:0   pc: 0x401759  rip:0x485e45
-------------


        -- User Interrupt handler --
[    6.180153] send: unregister sender uintrfd 3 for task=78 ret 0
[    6.181136] debug!
[    6.182027] rdmsrl 1
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
0x45006d 
0x4017ea 
0x41801a 
0x417ed3 
0x417eec 
0x417a00 
0x417b43 
0x417ef1 
0x417ef1 
0x417ef1 
0x421c40 
0x421c52 
0x47b440 
0x47db00 
0x47db00 
0x47db60 
0x47db60 
0x47db60 
0x47b44a 
0x47b45c 
0x421c58 
0x4205b0 
0x420678 
0x421c6e 
0x421c6e 
0x421c84 
0x421c8e 
0x48c8d0 
0x48c908 
0x421c94 
0x421c94 
0x421c9e 
0x4206b0 
0x420738 
0x42079c 
0x42079c 
0x42079c 
0x417ef6 
0x417f08 
0x417f27 
0x417f55 
0x417f6e 
0x417f6e 
0x417f8b 
0x417f99 
0x417f9e 
0x417fc7 
0x41808b 
0x41808b 
0x450150 
[    6.174921] sample2[77]: segfault at 7f4b15d06640 ip 00007f4b15d06640 sp 00007fff285ec0d8 error 15
0x45015b 
0x450163 
0x41809f 
0x417fd3 
0x417fe5 
0x417ffb 
[    6.174921] Code: 00 00 a0 e0 4a 00 00 00 00 00 a0 e6 4a 00 00 00 00 00 a0 ef 4a 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 <40> 66 d0 15 4b 7f 0f
[    6.174921] recv: Release uintrfd for r_task 77 uvec 0
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
qemu:wrmsr pd 0x0000000000000000
qemu:wrmsr RR 0x0
qemu:wrmsr stackadjust 0x0
qemu:wrmsr handler 0x0000000000000000
0x587ead 
[ 0x49596a 
  0x5882b6 
0x5882cb 
0x5882cb 
0x5882cb 
0x5882cf 
0x5882d4 
0x5882d9 
0x452950 
0x452a60 
0x452a6e 
0x452a95 
0x452a9c 
0x452980 
0x42ec80 
0x452985 
0x45298a 
0x452990 
0x4529a3 
 6.263346] sample2 (77) used greatest stack depth: 14512 bytes left
0x4e3de0 
0x422fff 
0x4233cb 
0x4233cb 
0x4233cb 
0x4233d8 
0x5882e5 
0x5881c6 
0x5881c6 
0x58821c 
0x448157 
0x448165 
0x448173 
0x44819a 
0x588278 
0x58827f 
0x58829b 
0x5882f7 
0x588356 
0x588698 
0x588698 
0x588698 
0x5883e5 
0x4b9837 
Segmentation fault
0x4b9837 
0x4b9837 
0x4b9837 
0x445b2e 
0x445bc8 
0x588401 
0x588406 
0x448c1c 
0x448c4c 
0x5886b7 
0x588518 
0x5875d4 
0x5875d8 
0x5875e5 
0x592f92 
0x592f92 
0x44c1cc 
0x44c043 
0x592fe0 
0x5e9037 
0x44f80b 
0x44f784 
0x58c86e 
0x58c885 
0x44f80b 
0x5876b0 
0x58ee73 
0x5ed66e 
0x4bb63a 
0x5ed74f 
0x44d932 
0x44de60 
0x44de6f 
0x44de6f 
0x44de6f 
0x44de7f 
0x44de91 
0x44affe 
0x44dea1 
0x44df7f 
0x44df1e 
0x44d502 
0x4bb460 
0x4bb4d4 
0x4bb4fd 
0x496beb 
0x531127 
0x4b967b 
0x4bbb6b 
0x52eff3 
0x44f80b 
0x44affe 
0x4b9d42 
0x496beb 
0x42e6f6 
0x4b9837 
/ # 0x4b9837 
0x4b9837 
0x4b9837 
```







```shell
# 只改了eip，同样senduipi called 之后输出各个块的cp
Sending IPI from sen

--------------
qemu: caught 0xf30fc7 SENDUIPI eip:0x4017bc
 --------------


qemu:helper senduipi called receive  regidx:240, uipiindex: 0
qemu: data of uitt valid:1 user_vec:0  UPID address 0xffffa20a82c1c1c0 
qemu: content of upid:  status:0x0    nv:0xec    ndst:0x100    0x0000000000000000
qemu: data write back in upid:  status:0x1    nv:0xec    ndst:0x100    0x0000000000000001
der thread i[    8.789172] uintr_unregister_sender called
[    8.793096] rdmsrl misc 5
0x485e45  block size:26  icont:10
!!! interrupt 2  intno:236 
recognize uintr
the physical address of APIC 0x10   the EOI content: 0xf000ff53f000ff53
rrnzero called handler: 0x4016f5  rr: 0x1
qemu:origin exp 0x7fff9ed2ca50   eip 0x485e45  eflags: 0x202
qemu:move statck 0x7fff9ed2c9d0
qemu:after align statck 0x7fff9ed2c9d0
qemu:push finish now esp is: 0x7fff9ed2c9b0qemu: eip: 0x4016f5
[    8.798471] send: unregister sender uintrfd 3 for task=78 ret 0
[    8.800438] debug!
[    8.793096] rdmsrl misc 5
0x44f36f  block size:12  icont: 3
0x475649  block size: 8  icont: 2
0x401734  block size: 1  icont: 1
--------------


qemu:caught 0xf30f01ec UIRET  # 注意！ pc: 0x401759 已经进入sender_thread
before:  pc_start: 0x401755  sc_base:0   pc: 0x401759  pc.next:0x401755  rip:0x401734
helper uiret called, now eip: 0x401734 # __libc_write后
qemu: now esp is: 0x7fff9ed2c950  # 回去的地方在nanosleep
qemu:poped values:uirrv:0x1 rip:0x485e45   eflags:0x202  rsp:0x7fff9ed2ca50 
pc_start: 0x401755  sc_base:0   pc: 0x401759  rip:0x485e45
-------------


[    8.793096] uintr_register_sender called
qemu:wrmsr tt 0xffffa20a82625001
qemu:wrmsr misc 0x000000ec00000100
0x45006d  block size: 2  icont: 1
0x44f36f  block size: 1  icont: 1
0x4178e8  block size: 8  icont: 2
0x41b720  block size: 5  icont: 1
0x41b73e  block size:30  icont: 5
0x41b746  block size: 8  icont: 3
qemu:helper senduipi called receive  regidx:240, uipiindex: 0
qemu: data of uitt valid:1 user_vec:0  UPID address 0xffffa20a82c1c1c0 
qemu: content of upid:  status:0x0    nv:0xec    ndst:0x100    0x0000000000000000
qemu: data write back in upid:  status:0x1    nv:0xec    ndst:0x100    0x0000000000000001
qemu:wrmsr misc 0x000000ec00000000
qemu:wrmsr tt 0x0000000000000000
[    8.801274] rdmsrl 1
[    8.793096] rdmsrl 2
[    8.793096] send: register sender task=77 flags 0 ret(uipi_id)=0
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
0x45006d  block size: 1  icont: 1 # syscall
[    8.793096] rdmsrl misc 5
[    8.793096] rdmsrl misc 5
[    8.793096] uintr_unregister_sender called
[    8.793096] send: unregister sender u0x45006d  block size:92  icont:20
0x4017ea  block size: 8  icont: 2 # sender_thread unrigester
int0x41801a  block size: 7  icont: 3 # 417ed3 <start_thread+0x193>
r0x417ed3  block size: 5  icont: 1
f0x417eec  block size:19  icont: 3
d 0x417a00  block size: 5  icont: 1
3 0x417b43  block size:22  icont: 5
0x417ef1  block size: 1  icont: 1
0x421c40  block size: 5  icont: 1
for task=77 ret 0
0x47b440  block size: 6  icont: 1
[ 0x47b440  block size:92  icont:20
  0x47db60  block size:38  icont:11
 0x47b44a  block size: 9  icont: 6
8.793096] debug!
0x47b45c  block size:18  icont: 5
[    8.793096] rdmsrl 1
0x4205b0  block size:22  icont: 3
[    8.7930960x420678  block size:13  icont: 3
0x421c6e  block size: 1  icont: 1
]0x421c84  block size:22  icont: 3
0x421c84  block size:10  icont: 2
0x421c84  block size:92  icont:20
0x421c8e  block size:10  icont: 2
 rdmsrl misc 5
0x48c908  block size:21  icont: 6
0x421c94  block size: 2  icont: 2
0x45006d  block size:12  icont: 3
0x4018e9  block size: 8  icont: 2
0x421c9e  block size:10  icont: 2
0x4206b0  block size: 9  icont: 2
0x420738  block size:31  icont: 8
0x42079c  block size:24  icont: 5
0x417ef6  block size: 5  icont: 4
0x417f08  block size:18  icont: 3
0x417f27  block size:31  icont: 6
0x417f55  block size: 2  icont: 2
0x417f6e  block size:25  icont: 5
0x417f6e  block size:29  icont: 6
0x417f6e  block size:92  icont:20
0x417f8b  block size:29  icont: 6
0x417f99  block size:14  icont: 3
0x417f9e  block size: 5  icont: 1
0x417fc7  block size:41  icont:10
0x41808b  block size:12  icont: 2
0x450150  block size:20  icont: 4
0x45015b  block size: 3  icont: 2
0x450163  block size: 8  icont: 2
0x41809f  block size: 1  icont: 1
0x417fd3  block size: 5  icont: 1
0x417fe5  block size:18  icont: 3
0x417ffb  block size:14  icont: 3  # sleep 结束的第一条指令 ip:4018e9
[    8.793096] traps: sample2[77] general protection fault ip:4018e9 sp:7fff9ed2cac0 error:0 in sample2[401000+a4000]
[    8.793096] recv: Release uintrfd for r_task 77 uvec 0
qemu:wrmsr misc 0x0000000000000000
qemu:wrmsr tt 0x0000000000000000
qemu:wrmsr pd 0x0000000000000000
qemu:wrmsr RR 0x0
qemu:wrmsr stackadjust 0x0
qemu:wrmsr handler 0x0000000000000000
[    8.879583] sample2 (77) used greatest stack depth: 14456 bytes left
0x587ead  block size: 2  icont: 2
0x587ead  block size:92  icont:20
0x49596a  block size: 2  icont: 2
0x49596a  block size:10  icont: 3
0x5882b6  block size: 8  icont: 2
0x5882cb  block size: 6  icont: 2
0x5882cb  block size:92  icont:20
0x5882cf  block size: 4  icont: 2
0x5882d4  block size: 5  icont: 2
0x5882d4  block size:11  icont: 7
0x5882d9  block size: 5  icont: 2
0x452950  block size:12  icont: 3
0x452a60  block size:29  icont: 7
0x452a6e  block size:14  icont: 3
0x452a95  block size:10  icont: 2
0x452a9c  block size:63  icont:17
0x452980  block size:21  icont: 4
0x42ec80  block size: 5  icont: 1
0x452985  block size:11  icont: 3
0x45298a  block size: 5  icont: 2
0x452990  block size: 6  icont: 2
0x4529a3  block size:19  icont: 5
0x4e3de0  block size:63  icont:17
0x422fff  block size:27  icont: 6
0x4233cb  block size:63  icont:17
0x4233d8  block size:13  icont: 3
0x5882e5  block size:10  icont: 3
0x5881c6  block size:18  icont: 5
0x58821c  block size:31  icont: 9
0x448157  block size:92  icont:19
0x448165  block size:14  icont: 4
0x448173  block size:14  icont: 4
0x44819a  block size:18  icont: 5
0x588278  block size:16  icont: 4
0x58827f  block size: 7  icont: 2
0x58829b  block size:23  icont: 6
0x5882f7  block size:10  icont: 4
0x588356  block size:14  icont: 5
0x588698  block size: 9  icont: 4
0x5883e5  block size:31  icont: 7
0x4b9837  block size: 1  icont: 1
0x445b2e  block size:63  icont:17
0x445bc8  block size:28  icont: 7
0x588401  block size:13  icont: 2
0x588406  block size: 5  icont: 1
0x448c1c  block size:63  icont:17
0x448c4c  block size:48  icont: 9
0x5886b7  block size:14  icont: 2
0x588518  block size: 5  icont: 1
0x5875d4  block size: 9  icont: 2
0x5875d8  block size: 4  icont: 2
0x5875e5  block size: 5  icont: 2
0x44c1cc  block size:63  icont:17
0x44c043  block size:35  icont: 7
0x592fe0  block size:20  icont: 3
0x5e9037  block size:15  icont: 3
0x44f80b  block size:63  icont:17
0x44f784  block size:13  icont: 2
0x58c86e  block size:63  icont:17
0x58c885  block size:12  icont: 3
0x44f80b  block size:63  icont:17
0x5876b0  block size:63  icont:17
0x58ee73  block size:63  icont:17
0x5ed66e  block size:63  icont:17
0x4bb63a  block size:10  icont: 3
0x5ed74f  block size: 8  icont: 2
0x44d932  block size:37  icont: 8
0x44de60  block size:10  icont: 2
0x44de6f  block size:11  icont: 3
0x44de7f  block size:16  icont: 5
0x44de91  block size: 6  icont: 2
0x44affe  block size:16  icont: 4
0x44dea1  block size:17  icont: 5
0x44df7f  block size:18  icont: 4
0x44df1e  block size:18  icont: 4
0x44d502  block size:18  icont: 4
0x4bb460  block size:10  icont: 3
0x4bb4d4  block size: 1  icont: 1
0x4bb4fd  block size:10  icont: 3
0x496beb  block size:10  icont: 3
0x531127  block size: 1  icont: 1
0x4b967b  block size:11  icont: 7
0x4bbb6b  block size:10  icont: 3
0x52eff3  block size: 8  icont: 2
0x44f80b  block size:63  icont:17
0x44affe  block size:13  icont: 2
0x4b9d42  block size:31  icont:10
0x496beb  block size:10  icont: 3
0x42e6f6  block size:20  icont: 3
0x4b9837  block size: 1  icont: 1
QEMU: Terminated
```





