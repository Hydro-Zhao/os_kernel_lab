kern/process/proc.c

# 实验记录

注释掉了kern/init/init.c中的grade_backtrace()

## 实验手册

### 创建第 0 个内核线程 idleproc

了解idleproc的作用（emmm，“没什么作用”）

### 创建第 1 个内核线程 initproc

虽然initproc是要执行init_main，但是实际的tf中eip指向的是kernel_thread_entry。kernel_thread_entry的作用更像是一个包装壳，它为调用真正的函数准备args，然后调用init_main，并且设置返回值，并调用do_exit（注意这个过程对象是trapframe而不是context）。实际switch_to执行后运行的是context中eip指向的指令，所以kernel_thread_entry还不是最先执行的，真正最先执行的是在copy_thread中context.eip指向的forkret。因为只有idleproc是通过特定的代码创建的，而之后的包括initproc在内的都是通过do_fork(kernel_thread)创建的，所以需要先调用forkret，而forkret调用forkrets汇编，forkrets使esp指向即将运行的线程的trapframe并执行中断返回，而中断返回的正是kernel_thread_entry。

### 调度并执行内核线程 initproc

弄清楚进程创建过程中context和tf的作用和区别，一个用在switch过程，一个用在中断异常过程

注意在switch_to中有更改任务状态段 ts 中特权态 0 下的栈顶指针 esp0的过程

## exercise1 分配并初始化一个进程控制块（需要编码）

## exercise2 为新创建的内核线程分配资源（需要编码）

## exercise3 阅读代码，理解 proc_run 函数和它调用的函数如何完成进程切换的。

## 扩展练习 Challenge：实现支持任意大小的内存分配算法