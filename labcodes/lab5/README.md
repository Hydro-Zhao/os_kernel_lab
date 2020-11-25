kern/process/proc.c

kern/mm/pmm.c

kern/trap/trap.c

Makefiel

# 实验记录

## emmm

做完实验之后无法通过测试，甚至labcodes_answer也不能完全通过测试。

最后对比了Harry-Chen的答案，发现问题出在makefile，疯了

```
diff --git a/labcodes/lab5/Makefile b/labcodes/lab5/Makefile
index 17b8503..9e5b6ed 100644
--- a/labcodes/lab5/Makefile
+++ b/labcodes/lab5/Makefile
@@ -48,13 +48,15 @@ ifndef  USELLVM
 HOSTCC         := gcc
 HOSTCFLAGS     := -g -Wall -O2
 CC             := $(GCCPREFIX)gcc
-CFLAGS := -march=i686 -fno-builtin -fno-PIC -Wall -ggdb -m32 -gstabs -nostdinc $(DEFS)
+#CFLAGS        := -march=i686 -fno-builtin -fno-PIC -Wall -ggdb -m32 -gstabs -nostdinc $(DEFS)
+CFLAGS := -fno-builtin -fno-PIC -Wall -ggdb -m32 -gstabs -nostdinc $(DEFS)
 CFLAGS += $(shell $(CC) -fno-stack-protector -E -x c /dev/null >/dev/null 2>&1 && echo -fno-stack-protector)
 else
 HOSTCC         := clang
 HOSTCFLAGS     := -g -Wall -O2
 CC             := clang
-CFLAGS := -march=i686 -fno-builtin -fno-PIC -Wall -g -m32 -nostdinc $(DEFS)
+#CFLAGS        := -march=i686 -fno-builtin -fno-PIC -Wall -g -m32 -nostdinc $(DEFS)
+CFLAGS := -fno-builtin -fno-PIC -Wall -g -m32 -mno-sse -nostdinc $(DEFS)
 CFLAGS += $(shell $(CC) -fno-stack-protector -E -x c /dev/null >/dev/null 2>&1 && echo -fno-stack-protector)
 endif
 
@@ -62,7 +64,8 @@ GDB           := $(GCCPREFIX)gdb
 CTYPE  := c S
 
 LD      := $(GCCPREFIX)ld
-LDFLAGS        := -m $(shell $(LD) -V | grep elf_i386 2>/dev/null | head -n 1)
+#LDFLAGS       := -m $(shell $(LD) -V | grep elf_i386 2>/dev/null | head -n 1)
+LDFLAGS        := -m $(shell $(LD) -V | grep elf_i386 2>/dev/null)
 LDFLAGS        += -nostdlib
 
 OBJCOPY := $(GCCPREFIX)objcopy
 ```


## lab1，lab4，lab5联合Q&A

* lab1和lab4，lab5是有区别的

    * lab1怎么实现一个栈两种特权级？

        lab1中只有一个栈，需要考虑到k2u和u2k以及iret时硬件对有特权级改变的不同处理方式。主要使用移动trapframe或临时栈来解决这个问题。

        k2u时，将trapframe复制到一个临时栈，并且修改原本栈上trapentry.S中调用trap->trap_dispatch的返回地址。使trapentry.S中call trap返回后esp指向的是临时的trapframe，而在临时的trapframe的最高两个位置，是我们手动添加的ss和esp（指向中断时esp指向的位置），在iret之后，又回到了原来的栈。

        u2k时，硬件自动切换成tss中保存的内核栈，那么每次u2k时，trapframe都保存在同一位置，可以把保存的trapframe再次入栈，并将call trap的返回地址设置成这个临时的trapframe就可以了

    * lab5 嵌套中断

        首先明确，嵌套中断只能发生在k2k

        因为lab4和lab5中用户态栈和内核栈是分开的，而且内核栈上本来就为trapframe保留了空间，所以在u2k时trapframe可以保存在tss指定的位置（也就是内核栈顶预留的位置），k2k的话是在esp指向的位置保存trapframe不会影响之前的trapframe。所以u2k之后只有一串k2k或单独一串k2k都可以做到。但是要注意，在k2k时，trapframe虽然能正常保存在栈上，但是current->tf并不会主动更新，所以在lab5 trap.c的trap函数相对于之前的trap函数有很大的变化，在栈上维护了一个trapframe链（同时也保留了之前的trap函数的处理方式）（注意otf和current->tf都是指针）（我觉的是因为有很多地方会用到全局变量current->tf而不是依靠参数来传递tf）。

    * 关于u1k接一串k2k中途发生了k2u（不对应之前的u2k）然后是u2k（我不知道这种情况存不存在）？如果可以，这样就不能处理tss指向的trapframe被覆盖的问题了和之后的处理程序覆盖之前的处理程序的数据的问题。还是说如果发生这种情况，等同于前一个u2k的本身就不用“返回”，它的trapframe是可以丢弃的？对于用户进程，内核的操作只是辅助做一些事情，所以不同系统调用之间不需要有记忆性？

        **[进程切换过程](https://github.com/LearningOS/ucore_os_docs/blob/master/lab6/lab6_3_4_2_process_switch.md)很完美地解决了我大部分关于系统调用，k2u和u2k的问题**

        > 进程调度函数 schedule 选择了下一个将占用 CPU 执行的进程后，将调用进程切换，从而让新的进程得以执行。通过实验四和实验五的理解，应该已经对进程调度和上下文切换有了初步的认识。在实验五中，结合调度器框架的设计，可对 ucore 中的进程切换以及堆栈的维护和使用等有更加深刻的认识。假定有两个用户进程，在二者进行进程切换的过程中，具体的步骤如下：
        >
        > 首先在执行某进程 A 的用户代码时，出现了一个 trap (例如是一个外设产生的中断)，这个时候就会从进程 A 的用户态切换到内核态(过程(1))，并且保存好进程 A 的 trapframe；当内核态处理中断时发现需要进行进程切换时，ucore 要通过 schedule 函数选择下一个将占用 CPU 执行的进程（即进程 B），然后会调用 proc_run 函数，proc_run 函数进一步调用 switch_to 函数，切换到进程 B 的内核态(过程(2))，继续进程 B 上一次在内核态的操作，并通过 iret 指令，最终将执行权转交给进程 B 的用户空间(过程(3))。
        >
        > 当进程 B 由于某种原因发生中断之后(过程(4))，会从进程 B 的用户态切换到内核态，并且保存好进程 B 的 trapframe；当内核态处理中断时发现需要进行进程切换时，即需要切换到进程 A，ucore 再次切换到进程 A(过程(5))，会执行进程 A 上一次在内核调用 schedule (具体还要跟踪到 switch_to 函数)函数返回后的下一行代码，这行代码当然还是在进程 A 的上一次中断处理流程中。最后当进程 A 的中断处理完毕的时候，执行权又会反交给进程 A 的用户代码(过程(6))。这就是在只有两个进程的情况下，进程切换间的大体流程。
        >
        > 几点需要强调的是：
        >
        > a) 需要透彻理解在进程切换以后，程序是从哪里开始执行的？需要注意到虽然指令还是同一个 cpu 上执行，但是此时已经是另外一个进程在执行了，且使用的资源已经完全不同了。
        >
        > b) 内核在第一个程序运行的时候，需要进行哪些操作？有了实验四和实验五的经验，可以确定，内核启动第一个用户进程的过程，实际上是从进程启动时的内核状态切换到该用户进程的内核状态的过程，而且该用户进程在用户态的起始入口应该是 forkret。

    * lab1和lab4能够嵌套中断吗？

        既然能多次缺页，那么坑定是可以嵌套的，反正它们一直在内核态，只是k2k的情况
    
* lab4中forkrets之前有发生过中断吗？为什么要执行中断返回？这和lab5有关吗？lab4中，既然forkrets只是中断返回，为什么不把forkrets的任务交给kernel_thread_entry？

    lab4中是kernel_thread函数设置的trapframe->eip指向kernel_thread_entry，并且调用do_fork设置forkret为context.eip的。do_fork即会被内核程序调用，也会被用户系统调用调用，所以普适地，forkret是不可少的。

    lab4中是直接调用do_fork，没有发生中断，但是因为我们设置了init_main的trapframe，所以中断返回并不会有影响，甚至达到了把控制权交给kernel_thread_entry的作用。但是lab5中kernel_execve同样是在内核中，但是是通过系统调用，通过系统调用，lab5成功从内核态程序运行了用户态的程序

    所以如果有特权级的改变就必需使用系统调用

## 实验手册

除了系统调用以外，lab5还有其他很多细节问题。如实验手册所说，lab5主要是内存管理和进程管理两部分。进程管理部分包括对进程生命周期的管理，而进程少不了的就是内存空间，所以内存管理又是另外一个很重要的点。可以仔细阅读 实验执行流程概述 来思考需要关注哪些问题。

copy_from_user 和 copy_to_usery的实现

ucore中的调度点有好几个（参考lab6），但是它们都是通过schedule->proc_run->switch_to来切换的

### 创建用户进程

libc的范围包括哪些？

理解do_execve和load_icode，还可以帮助解决系统调用的那些疑问“至此，用户进程的用户环境已经搭建完毕。此时 initproc 将按产生系统调用的函数调用路径原路返回，执行中断返回指令“iret”（位于 trapentry.S 的最后一句）后，将切换到用户进程 hello 的第一条语句位置_start 处（位于 user/libs/initcode.S 的第三句）开始执行。”

idleproc只是空转并让权，initproc是作为所有进程的祖先，负责它们的资源回收（wait），user_main真正使用kernel_execve运行用户态的程序（这应该是lab5特有的，与实际应该有差别）。运行用户程序的过程有一个关键函数laod_icode（详见 创建用户进程），其中有一个很重要的描述“至此，用户进程的用户环境已经搭建完毕。此时 initproc （应该是user_main）将按产生系统调用的函数调用路径原路返回，执行中断返回指令“iret”（位于 trapentry.S 的最后一句）后，将切换到用户进程 hello 的第一条语句位置_start 处（位于 user/libs/initcode.S 的第三句）开始执行。”

用户程序在iret之后首先运行的是elf->e_entry，也就是_start，然后调用umain，最后调用main

## exercise1

## exercise2 父进程复制自己的内存空间给子进程（需要编码）

copy_mm(do_fork)-->dup_mmap(vmm.c)-->copy_range(pmm.c)，很多内存管理的实现细节还要仔细思考

## exercise3 阅读分析源代码，理解进程执行 fork/exec/wait/exit 的实现，以及系统调用的实现

弄清user/ user/libs libs的作用，尤其时user/libs中各代码与系统调用和libc的关系

## 扩展练习 Challenge ：实现 Copy on Write （COW）机制