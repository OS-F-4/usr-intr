# qemu-tutorial

[toc]

欢迎来到qemu-tutorial! 这是一个介绍qemu的代码级的中文教程, 希望它能够对你了解qemu有所帮助。

qemu体量庞大, 这个教程不可能面面俱到, 欢迎大家补充和纠正。

下面的部分主要依据我在qemu上实现intel x86用户态中断硬件特性涉及的内容。主要涉及新增指令, 指令翻译, 寄存器定义, 中断处理, 访存等操作,  架构主要和x86相关。教程更偏向定位和一些调试技巧, 更希望通过一些小的调试过程的介绍来提升对qemu的理解。



## qemu初体验

### 编译和运行

我们开始吧! qemu github 主页`https://github.com/qemu/qemu`。

```
git clone git@github.com:qemu/qemu.git
cd qemu
mkdir build
cd build
../configure --enable-debug  --target-list=x86_64-softmmu
make
```

完整编译qemu需要非常长的时间, 在这里选择`--target-list=x86_64-softmmu` 可以只编译支持运行`x86`架构的部分, 可以提升编译的速度, 如果需要编译不同架构的qemu, 则可以修改list中的内容。

编译完成后, 可执行文件中在`/qemu/build/x86_64-softmmu/qemu-system-x86_64`, 我们在此备用。

我们这里使用linux作为内核进行加载, 为了更好的契合修改以及应用的需要, 我们这里使用intel编写的支持用户态中断的内核, 并创建相应的文件的系统。具体的编译方式见`编译技巧和环境.md`。

假设编译出的内核位置在`$kernelbase`, 文件系统在`$fsbase`, qemu文件夹在`$qemubase`, 我们可以用以下命令将编译好的内核运行在我们的qemu上。

```
$qemubase/qemu/build/x86_64-softmmu/qemu-system-x86_64 -smp 2 \
-m 1024M   -nographic  \
-kernel $kernelbase/bzImage \
-initrd $fsbase/initramfs-busybox-x86_64.cpio.gz \
-append "root=/dev/ram0 rw rootfstype=ext4 console=ttyS0 init=/linuxrc"
```

我们可以将以上命令包装为一个`run.sh`方便我们快速启动qemu并运行内核。

"smp"指定了处理器的个数，"m"指定了内存的大小。"kernel"指定编译生成的内核镜像的存放位置。的"append"是内核启动的附加参数。”initrd” 指定了文件系统镜像所在的位置。

运行qemu后可以进入内核镜像的shell程序。

可以通过`control + a` 随后按`x`退出qemu。

### 源代码概览

在本教程中, 我们主要关注和指令和体系架构的部分以及中断的部分的。

指令架构部分的代码主要在`target`目录下, 可见目录下有各类指令集对应的子文件夹, 其中`x86`对应的文件夹为`i386`。

```shell
$ ls target
Kconfig  arm  cris     hppa  m68k         microblaze  nios2     ppc    rx     sh4    tricore
alpha    avr  hexagon  i386  meson.build  mips        openrisc  riscv  s390x  sparc  xtensa
```

在`i386`文件夹下又有大量文件, 我们主要关注`tcg`文件夹下的内容, 主要涉及指令翻译以及中断异常处理逻辑的实现, 其中`translate.c` 是最为核心的部分, 代码量也非常大, 但是仔细解构可以发现, 这部分的代码主要工作就是解析二进制字节, 匹配翻译, 并将指令转化为一系列宿主机器指令的集合或者若干函数的集合。`cpu.h` 和`cpu.c`中的内容也是重要的, 主要涉及体系寄存器的定义以及不同的硬件配置。

`hw`目录主要涉及硬件周围设备相关的代码, 其中`apic`, 即中断控制器, 相关的代码在`hw/intc`下, 在涉及apic时, 我们会修改这一部分的代码。

### 在qemu中输出调试

在这里我们希望在qemu初始化cpu的时候进行输出调试。在qemu中, 输出日志的方式是通过`qemu_log()`函数, 用法和`printf()`相同, 需要包括`"qemu/log.h"` 头文件。

```c
//target/i386/helper.c
void do_cpu_init(X86CPU *cpu)
{   
    qemu_log("hello qemu!!\n");
    CPUState *cs = CPU(cpu);
    CPUX86State *env = &cpu->env;
    CPUX86State *save = g_new(CPUX86State, 1);
    int sipi = cs->interrupt_request & CPU_INTERRUPT_SIPI;
    *save = *env;
    cpu_reset(cs);
    cs->interrupt_request = sipi;
    memcpy(&env->start_init_save, &save->start_init_save,
           offsetof(CPUX86State, end_init_save) -
           offsetof(CPUX86State, start_init_save));
    g_free(save);
    if (kvm_enabled()) {
        kvm_arch_do_init_vcpu(cpu);
    }
    apic_init_reset(cpu->apic_state);
}
```

重新编译qemu, 执行`run.sh`, 你可能可以在某个瞬间看到自己想要的输出。但其实输出非常短暂, 因为linux启动时会清空输出界面, 导致看不到我们想要的输出。这里可以用重定向的方法来把linux的输出导出到别的地方, 或者简单的利用管道即可。

```shell
$ ./run.sh | echo nothing
nothing
hello qemu!!
hello qemu!!
```

因为启动了两个虚拟cpu, 所以有两行输出。至此我们已经找到了cpu启动部分的代码, 并且成功修改代码后进行输出。



## 第一条指令

我们尝试在qemu中定位到一条指令, 每次该指令被翻译, 就输出trace信息。

```c
//target/i386/tcg/translate.c
    case 0xc3: /* ret */
        qemu_log("ret instruction finded when translate\n");
        ot = gen_pop_T0(s);
        gen_pop_update(s, ot);
        /* Note that gen_pop_T0 uses a zero-extending load.  */
        gen_op_jmp_v(s->T0);
        gen_bnd_jmp(s);
        gen_jr(s, s->T0);
        break;
```

再次编译运行qemu然后进行输出, 会发现屏幕上出现了大量的输出语句, 记得及时`control + a` 后`x` 退出qemu。

我们在这里介绍translate.c中主要的翻译机制:

本质上来说, 识别指令就是一个查二进制表的过程, 每个二进制码对应一条汇编指令, 翻译过程类似自动机, 识别一个字节后进行二进制case的switch, 如果非法则跳入非法指令处理流程, 如果合法则继续匹配一下一个字节。这里我们以`rdtsc`这条指令为例子, 从源代码查看指令的翻译过程。

通过查阅资料, 我们得知`rdtsc`指令的二进制编码为`0f 31`, 主要的功能是读取时间寄存器, 随后写入edx:eax。

总的翻译过程是从`static target_ulong disas_insn(DisasContext *s, CPUState *cpu)` 函数开始的, 这个函数十分庞大, 所以我们只取其中比较关键的部分进行讲解。

```c
next_byte:
    b = x86_ldub_code(env, s);
    /* Collect prefixes.  */
    switch (b) {
    case 0xf3:
        prefixes |= PREFIX_REPZ;
        goto next_byte;
    case 0xf2:
        prefixes |= PREFIX_REPNZ;
        goto next_byte;
    case 0xf0:
        prefixes |= PREFIX_LOCK;
        goto next_byte;
        //....
```

我们说过, 翻译过程是按字节翻译的`x86_ldub_code`这个函数就是取出待翻译的块中的一个字节, 赋值到b中, 然后进行指令的匹配。而字节之间很容易重复, qemu通过前缀的位表示来区分之前翻译的几个字节中记录的信息, 对此我们只需要大概有印象, 具体的指令相关的信息可以通过查阅`x86`的手册来辅助理解。在`rdtsc`这条指令中, `0f`这个字节不会匹配到任何前缀, 程序继续向下运行。

前缀识别完毕后, 开始识别操作符。

```c
    /* now check op code */
 reswitch:
    switch(b) {
    case 0x0f:
        /**************************/
        /* extended op code */
        b = x86_ldub_code(env, s) | 0x100;
        goto reswitch;
        /**************************/
        /* arith & logic */
    case 0x00 ... 0x05:
    case 0x08 ... 0x0d:
```

这里的`0x0f`恰好是`rdtsc`的第一个字节, 则程序会再读出下一个字节, 并或上`0x100`, 这里的作用是一种添加前缀信息的体现, 标志当前的前一个字节是`0x0f`这意味着`rdtsc`中的`0x31`和其他情况下的`0x31`走的匹配路径是不同的分支。

我们直接搜索`0x131`, 发现确实直接定位到了`rdtsc`中的内容。

```c
    case 0x131: /* rdtsc */
        gen_update_cc_op(s);
        gen_jmp_im(s, pc_start - s->cs_base);
        if (tb_cflags(s->base.tb) & CF_USE_ICOUNT) {
            gen_io_start();
        }
        gen_helper_rdtsc(cpu_env);
        if (tb_cflags(s->base.tb) & CF_USE_ICOUNT) {
            gen_jmp(s, s->pc - s->cs_base);
        }
        break;
```

在此添加相关输出, 在运行相关的程序包含此指令时, 就可以看到相应的输出。理论上大多数的指令都会在注释中列出, 直接搜索相关相关的指令的名称如`rdtsc`就可找到对应的操作, 我们介绍此过程主要帮助读者理解翻译, 在尝试自己添加新的指令或者操作时, 能够更加高效和符合qemu中的规范。

## 理解翻译块机制

QEMU 是一个动态翻译器。当它第一次遇到一段代码时，它将其转换为主机指令集。通常是动态翻译器非常复杂且高度依赖CPU。 QEMU 使用了一些技巧这使得它相对容易携带和简单，同时实现良好的表现。

QEMU 的动态翻译后端叫做 TCG, `Tiny Code Generator`的简称, 翻译过程为`目标机器代码->tcg->宿主机器代码`, 如果是简单的指令, 可能通过几条tcg中间代码代替, 如果是复杂的指令, 则通过helper函数来进行实现。其中一个重要的过程为翻译块机制, 为了方便加速, qemu不是翻译一条指令执行一条指令, 而是翻译一些指令, 然后同时执行这些指令。指令分块的依据和编译原理中基本快的划分比较类似, 当在块内翻译到跳转语句时, qemu会停止翻译, 执行已经翻译的部分, 根据执行结果寻找下个需要翻译的块进行翻译。这样的翻译粒度能够避免全局代码翻译带来的巨大翻译量, 同时也比边翻译边执行的效率要高, 如果翻译结果块(宿主机器代码)在缓存中停留时间较长, 也能够很好的起到加速的作用, 变相借助了宿主机器的缓存机制, 使得模拟的行为更加接近真实。

所以这里需要注意的有两点。 第一, 翻译和执行是分离的, 我们在`translate.c `中进行的输出均是在翻译阶段进行的, 也就是说如果我们像在执行阶段进行操作, 需要有别的方法, 这也是为什么翻译阶段存在许多`gen_`开头的函数, 这些函数的本质就是将翻译出的代码加入翻译块中, 而并不执行这些代码。经过全局搜索我们容易发现, `gen_`开头的函数没有相应的定义, 这里其实是宏在起作用, 但是具体宏的转化以及背后的运行机制笔者也没有彻底弄懂, 这里我们将举一个简单的例子进行演示。第二, 在我们尝试修改qemu代码时, 要注意翻译过程中是否是跳转语句, 是否需要让翻译过程终止, 先执行当前翻译块的内容。

我们依然通过`rdtsc`这条指令来进行翻译块机制的演示。首先编写一个简单的应用程序, 编译后放入qemu挂载的文件系统后制作文件系统镜像。

```c
#include<stdio.h>
typedef unsigned long long int  uint64_t;
uint64_t rdtsc() // linux
{
    unsigned int lo, hi;
    __asm__ volatile ("rdtsc" : "=a" (lo), "=d" (hi));
    return ((uint64_t)hi << 32) | lo;
}
int main(){
        uint64_t time = rdtsc();
        printf("the time is %llu", time);
        return 0;
}
```

我们在以下几个地方添加pin, `(env->hflags & HF_CPL_MASK) == 3` 主要作用是标准当前是用户态(内核态调用rdtsc非常多, 所以通过这个条件进行过滤):

在`translate.c`中添加, 这个不再过多赘述`rdtsc_translated`用于标志是否进行了翻译, 这个变量将会在其他地方引用, 。

```c
//target/i386/tcg/translate.c
bool rdtsc_translated = false;
/* convert one instruction. s->base.is_jmp is set if the translation must
   be stopped. Return the next pc value */
static target_ulong disas_insn(DisasContext *s, CPUState *cpu)
  //...
     case 0x131: /* rdtsc */
        gen_update_cc_op(s);
        if((env->hflags & HF_CPL_MASK) == 3){
            qemu_log("caught rdtsc in translate.c\n");
            rdtsc_translated = true;
        }
        gen_jmp_im(s, pc_start - s->cs_base);
        if (tb_cflags(s->base.tb) & CF_USE_ICOUNT) {
            gen_io_start();
        }
        gen_helper_rdtsc(cpu_env);
        if (tb_cflags(s->base.tb) & CF_USE_ICOUNT) {
            gen_jmp(s, s->pc - s->cs_base);
        }
        break;
```

我们再找到这个`gen_helper_rdtsc(cpu_env);`这个函数的真实位置, 观察函数语义, 很明显可以知道这和硬件定义的。

```c
//target/i386/tcg/misc_helper.c
bool rdtsc_execed = false;
void helper_rdtsc(CPUX86State *env)
{
    uint64_t val;
    if((env->hflags & HF_CPL_MASK) == 3){
        qemu_log("rdtsc execed\n");
        rdtsc_execed = true;
    }
    if ((env->cr[4] & CR4_TSD_MASK) && ((env->hflags & HF_CPL_MASK) != 0)) {
        raise_exception_ra(env, EXCP0D_GPF, GETPC());
    }
    cpu_svm_check_intercept_param(env, SVM_EXIT_RDTSC, 0, GETPC());
    val = cpu_get_tsc(env) + env->tsc_offset;
    env->regs[R_EAX] = (uint32_t)(val);
    env->regs[R_EDX] = (uint32_t)(val >> 32);
}
```

我们找到块翻译和执行相关的代码, 其中核心的是`cpu_loop_exec_tb`这个函数, 我们分别在函数的前后进行添加pin进行输出。

```c
//accel/tcg/cpu-exec.c
extern bool rdtsc_translated;
extern bool rdtsc_execed;
int cpu_exec(CPUState *cpu){
    int ret;
    SyncClocks sc = { 0 };
//....
    /* if an exception is pending, we execute it here */
    while (!cpu_handle_exception(cpu, &ret)) {
        TranslationBlock *last_tb = NULL;
        int tb_exit = 0;
        while (!cpu_handle_interrupt(cpu, &last_tb)) {
            TranslationBlock *tb;
            target_ulong cs_base, pc;
            uint32_t flags, cflags;
            cpu_get_tb_cpu_state(cpu->env_ptr, &pc, &cs_base, &flags);
            cflags = cpu->cflags_next_tb;
            if (cflags == -1) {
                cflags = curr_cflags(cpu);
            } else {
                cpu->cflags_next_tb = -1;
            }
            if (check_for_breakpoints(cpu, pc, &cflags)) { break; }
            tb = tb_lookup(cpu, pc, cs_base, flags, cflags);
            if (tb == NULL) {
                mmap_lock();
                tb = tb_gen_code(cpu, pc, cs_base, flags, cflags);
                mmap_unlock();
                qatomic_set(&cpu->tb_jmp_cache[tb_jmp_cache_hash_func(pc)], tb);
            }
#ifndef CONFIG_USER_ONLY
            if (tb->page_addr[1] != -1) {
                last_tb = NULL;
            }
#endif
            /* See if we can patch the calling TB. */
            if (last_tb) {tb_add_jump(last_tb, tb_exit, tb);}
            if(rdtsc_translated){
                qemu_log("translated before exec tb\n");
                rdtsc_translated = false;
            }else if (rdtsc_execed){
                qemu_log("execd before exec tb\n");
                rdtsc_execed = false;
            }

            cpu_loop_exec_tb(cpu, tb, &last_tb, &tb_exit);

             if(rdtsc_translated){
                qemu_log("translated after exec tb\n");
                rdtsc_translated = false;
            }else if (rdtsc_execed){
                qemu_log("execd after exec tb\n");
                rdtsc_execed = false;
            }
            /* Try to align the host and virtual clocks
               if the guest is in advance */
            align_clocks(&sc, cpu);
        }
    }
    cpu_exec_exit(cpu);
    rcu_read_unlock();
    return ret;
}
```

输出体量较多, 但是顺序是符合预期的:

```shell
caught rdtsc in translate.c
translated before exec tb
rdtsc execed
execd after exec tb
the time is 67290627360
```



## cpu定义

### 寄存器定义

`x86`寄存器相关的定义在`target/i386/cpu.h`中, 结构体为`CPUX86State`, 其中包括了所有的寄存器以及其他硬件状态相关的信息变量。由于结构体庞大, 我们只保留一些常用的做简单介绍。从本章开始, 我们也会在相关的内容中体现出我们之前在实现uintr中的工作, 所以一些代码在qemu原版中是没有的, 这部分多余的代码可以视作对qemu做修改的一些例子。因为增加新的硬件特性涉及寄存器, 指令, 操作系统, 用户程序, 编译器等内容, 全流程介绍较为冗余, 不够聚焦, 我们在介绍的过程中就简单穿插修改部分和qemu本身代码的讲解, 如果对我们实现uintr的部分感兴趣, 欢迎查看我们相关的仓库和主页。

```c
typedef struct CPUArchState {
    /* standard registers */
    target_ulong regs[CPU_NB_REGS]; // 通用寄存器
    target_ulong eip; // pc寄存器
    target_ulong eflags; /* eflags register. During CPU emulation, CC
                        flags and DF are set to zero because they are
                        stored elsewhere */

    /* emulator internal eflags handling */
    target_ulong cc_dst;
    target_ulong cc_src;
    target_ulong cc_src2;
    uint32_t cc_op;
    int32_t df; /* D flag : 1 if D = 0, -1 if D = 1 */
    uint32_t hflags; /* TB flags, see HF_xxx constants. These flags
                        are known at translation time. */
    uint32_t hflags2; /* various other flags, see HF2_xxx constants. */

    /* segments */
    SegmentCache segs[6]; /* selector values */
    SegmentCache ldt;
    SegmentCache tr;
    SegmentCache gdt; /* only base and limit are used */
    SegmentCache idt; /* only base and limit are used */

    target_ulong cr[5]; /* NOTE: cr1 is unused !!! */
//....
#ifdef TARGET_X86_64
    target_ulong lstar;
    target_ulong cstar;
    target_ulong fmask;
    target_ulong kernelgsbase;
#endif
    // 我们所实现的uintr依赖的寄存器
    uint64_t uintr_rr;
    uint64_t uintr_handler;
    uint64_t uintr_stackadjust;
    uint64_t uintr_misc;
    uint64_t uintr_pd;
    uint64_t uintr_tt;
    uint64_t uintr_uif;

//....
} CPUX86State;
```

### 寄存器访问

寄存器的访问比较简单, 在给定结构体指针的情况下, 直接通过结构体访问即可

```c
void helper_cpuid(CPUX86State *env)
{
    uint32_t eax, ebx, ecx, edx;
    cpu_svm_check_intercept_param(env, SVM_EXIT_CPUID, 0, GETPC());
    cpu_x86_cpuid(env, (uint32_t)env->regs[R_EAX], (uint32_t)env->regs[R_ECX],
                  &eax, &ebx, &ecx, &edx);
    env->regs[R_EAX] = eax;
    env->regs[R_EBX] = ebx;
    env->regs[R_ECX] = ecx;
    env->regs[R_EDX] = edx;
}
```

但是对于一些msr, 是基于专门的指令来访问的, 这就需要我们按照指令的翻译流程进行识别和访问。

首先对于msr, 每个都有特定的编号, 我们一般在`cpu.h`中找到相关的定义:

```c
#define MSR_IA32_SGXLEPUBKEYHASH0       0x8c
#define MSR_IA32_SGXLEPUBKEYHASH1       0x8d
#define MSR_IA32_SGXLEPUBKEYHASH2       0x8e
#define MSR_IA32_SGXLEPUBKEYHASH3       0x8f
// 用户态寄存器编号定义
#define MSR_IA32_UINTR_RR               0x985
#define MSR_IA32_UINTR_HANDLER          0x986
#define MSR_IA32_UINTR_STACKADJUST      0x987
#define MSR_IA32_UINTR_MISC             0x988
#define MSR_IA32_UINTR_PD               0x989
#define MSR_IA32_UINTR_TT               0x98a
```

对应的访问指令翻译部分在`translate.c`中。我们去掉`gen_`前缀, 直接搜索`helper_rdmsr`, 可以找到相关函数的定义位置。

```c
    case 0x130: /* wrmsr */
    case 0x132: /* rdmsr */
        if (check_cpl0(s)) {
            gen_update_cc_op(s);
            gen_jmp_im(s, pc_start - s->cs_base);
            if (b & 2) {
                gen_helper_rdmsr(cpu_env);
            } else {
                gen_helper_wrmsr(cpu_env);
                gen_jmp_im(s, s->pc - s->cs_base);
                gen_eob(s);
            }
        }
        break;
```

在`rdmsr`和`wrmsr`相关的函数中, 只是应用了简单的`switch`语句, 

```c
void helper_rdmsr(CPUX86State *env)
{
    X86CPU *x86_cpu = env_archcpu(env);
    uint64_t val;

    cpu_svm_check_intercept_param(env, SVM_EXIT_MSR, 0, GETPC());

    switch ((uint32_t)env->regs[R_ECX]) {
//...我们实现的读写msr部分
    case MSR_IA32_UINTR_HANDLER:
        val = env->uintr_handler;
        break;
    case MSR_IA32_UINTR_STACKADJUST:
        val = env->uintr_stackadjust;
        break;
    case MSR_IA32_UINTR_MISC:
        val = env->uintr_misc;
        break;
    case MSR_IA32_UINTR_PD:
        val = env->uintr_pd;
        break;
    case MSR_IA32_UINTR_TT:
        val = env->uintr_tt;
        break;
//...
}

void helper_wrmsr(CPUX86State *env)
{
    // if(Debug)qemu_log("wrmsr %hx \n",(uint32_t)env->regs[R_ECX]);
    uint64_t val;
    CPUState *cs = env_cpu(env);

    cpu_svm_check_intercept_param(env, SVM_EXIT_MSR, 1, GETPC());
//...我们实现的写msr相关的语句
  	case MSR_IA32_UINTR_RR:
        env->uintr_rr = val;
        if(val!= 0){
            helper_rrnzero(env);
        }
        break;
    case MSR_IA32_UINTR_HANDLER:
        env->uintr_handler = val;
        break;
    case MSR_IA32_UINTR_STACKADJUST:
        env->uintr_stackadjust = val;
        break;
    case MSR_IA32_UINTR_MISC:
        env->uintr_misc = val;
        break;
    case MSR_IA32_UINTR_PD:
        env->uintr_pd = val;
        break;
    case MSR_IA32_UINTR_TT:
        env->uintr_tt = val;
        break;
}      
```



## 中断处理

中断处理主要的内容在`target/i386/tcg/seg_helper.c`中, 在64位情况下执行的路线为:

```
do_interrupt_x86_hardirq->do_interrupt_all->do_interrupt64
```

这是大多数情况的执行路线, 在其他情况下可能会调用其他的中断处理函数。



## apic相关

apic主要是中断控制相关的函数, 用于发核间中断等。这里我们只举一个例子, `sipi`相关的操作。

这里我们找到sipi相关的流程线

```c
//target/i386/sysemu/seg_helper.c/x86_cpu_exec_interrupt
        if(Debug) printf("x86 cpu exec interrupt called sipi \n");
        do_cpu_sipi(cpu);

//target/i386/helper.c/do_cpu_sipi
void do_cpu_sipi(X86CPU *cpu)
{
    apic_sipi(cpu->apic_state); //此处调用的是第一个函数
}

//hw/intc/apic.c
void apic_sipi(DeviceState *dev)
{   
    if(Debug)printf("qemu: apic sipi called\n");
    APICCommonState *s = APIC(dev);

    cpu_reset_interrupt(CPU(s->cpu), CPU_INTERRUPT_SIPI);

    if (!s->wait_for_sipi)
        return;
    cpu_x86_load_seg_cache_sipi(s->cpu, s->sipi_vector);
    s->wait_for_sipi = 0;
}
```

可以在linux启动的日志中看到相关的输出:

```shell
[    0.334116] x86: Booting SMP configuration:
x86 cpu exec interrupt called sipi 
qemu: apic sipi called
x86 cpu exec interrupt called sipi 
qemu: apic sipi called
[    0.334220] .... node  #0, CPUs:  
```

根据以上的文件组织路径, 我们尝试在clear_eoi中构建相同的调用路径, 调用到apic.c中的函数。

apic中相关的操作在`apic.c`中有相关的定义, 但多是静态函数, 如果要对外暴露接口, 建议新写函数调用现成的函数。

还值得注意的是, 新的函数调用前, 需要将相应的函数声明添加到以下函数的位置:

```c
//include/hw/i386/apic.h
#ifndef APIC_H
#define APIC_H
/* apic.c */
void apic_deliver_irq(uint8_t dest, uint8_t dest_mode, uint8_t delivery_mode,
                      uint8_t vector_num, uint8_t trigger_mode);
int apic_accept_pic_intr(DeviceState *s);
void apic_deliver_pic_intr(DeviceState *s, int level);
void apic_deliver_nmi(DeviceState *d);
int apic_get_interrupt(DeviceState *s);
void apic_reset_irq_delivered(void);
int apic_get_irq_delivered(void);
void cpu_set_apic_base(DeviceState *s, uint64_t val);
uint64_t cpu_get_apic_base(DeviceState *s);
void cpu_set_apic_tpr(DeviceState *s, uint8_t val);
uint8_t cpu_get_apic_tpr(DeviceState *s);
void apic_init_reset(DeviceState *s);
void apic_sipi(DeviceState *s);
void apic_clear_eoi(DeviceState *s); //新添加的对外暴露的接口
int get_apic_id(DeviceState *dev); //新添加的对外暴露的接口
void send_ipi(DeviceState *dev, uint8_t dest, uint8_t nv); // 新添加的对外暴露的接口
void apic_poll_irq(DeviceState *d);
void apic_designate_bsp(DeviceState *d, bool bsp);
int apic_get_highest_priority_irr(DeviceState *dev);
/* pc.c */
DeviceState *cpu_get_current_apic(void);
#endif
```

