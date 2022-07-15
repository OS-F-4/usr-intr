# 尝试在kernel里加senduipi 并编译成功



### 尝试在makefile中修改gcc 为 gcc-11, 并添加-muintr flag(和用户程序一致)

```
HOSTCC	= gcc-11
HOSTCXX	= g++
endif

export KBUILD_USERCFLAGS := -Wall -Wmissing-prototypes -Wstrict-prototypes \
			      -O2 -fomit-frame-pointer -std=gnu89
export KBUILD_USERLDFLAGS :=

KBUILD_HOSTCFLAGS   := $(KBUILD_USERCFLAGS) $(HOST_LFS_CFLAGS) $(HOSTCFLAGS) -muintr -O1
KBUILD_HOSTCXXFLAGS := -Wall -O2 $(HOST_LFS_CFLAGS) $(HOSTCXXFLAGS) -muintr
KBUILD_HOSTLDFLAGS  := $(HOST_LFS_LDFLAGS) $(HOSTLDFLAGS)

```

结果找不到对应的头文件

```
  CC      security/selinux/ss/avtab.o
  CC      fs/io_uring.o
  CC      security/selinux/ss/policydb.o
../fs/io_uring.c:82:10: fatal error: x86gprintrin.h: No such file or directory
   82 | #include <x86gprintrin.h>
      |          ^~~~~~~~~~~~~~~~
compilation terminated.
make[2]: *** [../scripts/Makefile.build:277: fs/io_uring.o] Error 1
make[1]: *** [/home/xcd/qemu_uintr/uintr-linux-kernel/Makefile:1874: fs] Error 2
make[1]: *** Waiting for unfinished jobs....
  CC      security/selinux/ss/services.o
  CC      security/selinux/ss/conditional.o
  CC      security/selinux/ss/mls.o
  CC      security/selinux/ss/context.o
  CC      security/selinux/netlabel.o
  CC      security/lsm_audit.o
  AR      security/selinux/built-in.a
  CC      security/device_cgroup.o
  CC      security/integrity/iint.o
  CC      security/integrity/integrity_audit.o
  AR      security/integrity/built-in.a
  AR      security/built-in.a
make[1]: Leaving directory '/home/xcd/qemu_uintr/uintr-linux-kernel/build'
make: *** [Makefile:219: __sub-make] Error 2
```



## 切换系统默认gcc为gcc-11

```
(base) ➜  uintr-linux-kernel git:(uring) ✗ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9  49
update-alternatives: using /usr/bin/gcc-11 to provide /usr/bin/gcc (gcc) in auto mode
```



```
Initialize kernel stack variables at function entry
> 1. no automatic stack variable initialization (weakest) (INIT_STACK_NONE)
choice[1]: 1
Enable heap memory zeroing on allocation by default (INIT_ON_ALLOC_DEFAULT_ON) [N/y/?] n
Enable heap memory zeroing on free by default (INIT_ON_FREE_DEFAULT_ON) [N/y/?] n
Enable register zeroing on function exit (ZERO_CALL_USED_REGS) [N/y/?] (NEW) y
*
* KCSAN: dynamic data race detector
*
KCSAN: dynamic data race detector (KCSAN) [N/y/?] (NEW) y
  Perform short selftests on boot (KCSAN_SELFTEST) [Y/n/?] (NEW) y
  Early enable during boot (KCSAN_EARLY_ENABLE) [Y/n/?] (NEW) y
  Number of available watchpoints (KCSAN_NUM_WATCHPOINTS) [64] (NEW) 
  Delay in microseconds (for tasks) (KCSAN_UDELAY_TASK) [80] (NEW) 
  Delay in microseconds (for interrupts) (KCSAN_UDELAY_INTERRUPT) [20] (NEW) 
  Randomize above delays (KCSAN_DELAY_RANDOMIZE) [Y/n/?] (NEW) 
  Skip instructions before setting up watchpoint (KCSAN_SKIP_WATCH) [4000] (NEW) 
  Randomize watchpoint instruction skip count (KCSAN_SKIP_WATCH_RANDOMIZE) [Y/n/?] (NEW) 
  Interruptible watchers (KCSAN_INTERRUPT_WATCHER) [N/y/?] (NEW) 
  Duration in milliseconds, in which any given race is only reported once (KCSAN_REPORT_ONCE_IN_MS) [3000] (NEW) 
  Report races of unknown origin (KCSAN_REPORT_RACE_UNKNOWN_ORIGIN) [Y/n/?] (NEW) 
  Strict data-race checking (KCSAN_STRICT) [N/y/?] (NEW) 
    Only report races where watcher observed a data value change (KCSAN_REPORT_VALUE_CHANGE_ONLY) [Y/n/?] (NEW) 
    Assume that plain aligned writes up to word size are atomic (KCSAN_ASSUME_PLAIN_WRITES_ATOMIC) [Y/n/?] (NEW) 
    Do not instrument marked atomic accesses (KCSAN_IGNORE_ATOMICS) [N/y/?] (NEW) 
  Enable all additional permissive rules (KCSAN_PERMISSIVE) [N/y/?] (NEW) 
```

中途warning

```
../arch/x86/kernel/cpu/common.c: In function ‘setup_uintr’:
../arch/x86/kernel/cpu/common.c:330:9: warning: this ‘if’ clause does not guard... [-Wmisleading-indentation]
  330 |         if (!cpu_feature_enabled(X86_FEATURE_UINTR))
      |         ^~
../arch/x86/kernel/cpu/common.c:332:17: note: ...this statement, but the latter is misleadingly indented as if it were guarded by the ‘if’
  332 |                 goto disable_uintr;
      |                 ^~~~
../arch/x86/kernel/cpu/common.c:335:9: warning: this ‘if’ clause does not guard... [-Wmisleading-indentation]
  335 |         if (!cpu_has(c, X86_FEATURE_UINTR))
      |         ^~
      
../arch/x86/kernel/cpu/common.c:337:17: note: ...this statement, but the latter is misleadingly indented as if it were guarded by the ‘if’
  337 |                 goto disable_uintr;
      |                 ^~~~
../arch/x86/kernel/uintr_fd.c: In function ‘__do_sys_uintr_create_fd’:
../arch/x86/kernel/uintr_fd.c:73:9: warning: ISO C90 forbids mixed declarations and code [-Wdeclaration-after-statement]
   73 |         struct uintrfd_ctx *uintrfd_ctx;
      |         ^~~~~~
../arch/x86/kernel/uintr_fd.c: In function ‘__do_sys_uintr_register_handler’:
../arch/x86/kernel/uintr_fd.c:137:9: warning: ISO C90 forbids mixed declarations and code [-Wdeclaration-after-statement]
  137 |         int ret;
      |         ^~~
../arch/x86/kernel/uintr_fd.c: In function ‘__do_sys_uintr_unregister_handler’:
../arch/x86/kernel/uintr_fd.c:163:9: warning: ISO C90 forbids mixed declarations and code [-Wdeclaration-after-statement]
  163 |         int ret;
      |         ^~~
../arch/x86/kernel/uintr_fd.c: In function ‘__do_sys_uintr_register_sender’:
../arch/x86/kernel/uintr_fd.c:186:9: warning: ISO C90 forbids mixed declarations and code [-Wdeclaration-after-statement]
  186 |         struct uintr_sender_info *s_info;
      |         ^~~~~~
../arch/x86/kernel/uintr_fd.c: In function ‘__do_sys_uintr_unregister_sender’:
../arch/x86/kernel/uintr_fd.c:257:9: warning: ISO C90 forbids mixed declarations and code [-Wdeclaration-after-statement]
  257 |         struct uintr_sender_info *s_info;
```



```
In file included from /usr/lib/gcc/x86_64-linux-gnu/11/include/x86gprintrin.h:79,
                 from ../fs/io_uring.c:82:
../fs/io_uring.c: In function ‘__io_cqring_fill_event’:
/usr/lib/gcc/x86_64-linux-gnu/11/include/uintrintrin.h:65:1: error: inlining failed in call to ‘always_inline’ ‘_senduipi’: target specific option mismatch
   65 | _senduipi (unsigned long long __R)
      | ^~~~~~~~~
../fs/io_uring.c:1775:9: note: called from here
 1775 |         _senduipi(0);
      |         ^~~~~~~~~~~~
In file included from /usr/lib/gcc/x86_64-linux-gnu/11/include/x86gprintrin.h:79,
                 from ../fs/io_uring.c:82:
/usr/lib/gcc/x86_64-linux-gnu/11/include/uintrintrin.h:65:1: error: inlining failed in call to ‘always_inline’ ‘_senduipi’: target specific option mismatch
   65 | _senduipi (unsigned long long __R)
      | ^~~~~~~~~
../fs/io_uring.c:1775:9: note: called from here
 1775 |         _senduipi(0);
```



编译一般的用户程序:

```
#include <x86gprintrin.h>
int main(){
	_senduipi(0);
	return 0;
}
```



```shell
$gcc-11 test.c
In file included from /usr/lib/gcc/x86_64-linux-gnu/11/include/x86gprintrin.h:79,
                 from test.c:1:
test.c: In function ‘main’:
/usr/lib/gcc/x86_64-linux-gnu/11/include/uintrintrin.h:65:1: error: inlining failed in call to ‘always_inline’ ‘_senduipi’: target specific option mismatch
   65 | _senduipi (unsigned long long __R)
      | ^~~~~~~~~
test.c:5:9: note: called from here
    5 |         _senduipi(0);
      |         ^~~~~~~~~~~~
```

而使用命令‘gcc-11 test.c  -muintr’ 可以成功, 说明在编译uring.c 文件时, -muintr 这个flag并没有被添加到编译选项中。
