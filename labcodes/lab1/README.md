kern/debug/kdebug.c

kern/trap/trap.c

kern/init/init.c

# Lab1 实验关键点记录

## 疑问

Q: TSS如果只在一开始初始化，如果没有更新，之后每次u2k的时候不是会丢失内核栈之前保存的内容吗？如果有更新，是在代码的哪里呢？（结合lab4&5来寻找答案）

A: 在lab1中设置tss是在GDT的初始化中也就是pmm_init->gdb_init，tss指向的是内核栈的栈顶。同时在lab4中，在switch_to中会“设置任务状态段 ts 中特权态 0 下的栈顶指针 esp0 为 next 内核线程 initproc 的内核栈的栈顶，即 next->kstack + KSTACKSIZE ”解释了何时设置tss，也就是说tss是固定的。那么每次u2k是在同一个位置保存trapframe。那么多次发生u2k会使trapframe被破坏吗（见lab5 README）？

## 实验手册

熟悉Lab0中介绍的工具将大大减少Lab1所花时间（Lab1的exercise就是对Lab0中介绍的工具进行实际操作）

## exercise1 理解通过make生成执行文件的过程

要熟悉Makefile的语法，其实不难，主要就是变量定义和实际操作。比较难处理的就是理解内置和自定义函数的行为。

主要关注生成kernel.img的部分，具体可以看labcode_answer或Harry-Chen的report。

了解编译选项中取消了哪些标准库

链接时怎么指定起始位置和代码段地址

如何使用链接脚本.ld

标准bootloader的必需部分

ELF文件的格式

objdump，objcopy等工具的使用

## exercise2 使用qemu执行并调试lab1中的软件

next和nexti，step和stepi

x/FMT $ADDR: x /10i $pc中i的含义

set architecture

qemu作为gdb的服务器

（对gdb操作熟悉的话可以略过这一步）

## exercise3 分析bootloader进入保护模式的过程

> Harry-Chen

初始化的作用？？？

A20地址线，参考实验手册

配置GDT

开启保护模式

设置段寄存器，长跳转指令的作用

## exercise4 分析bootloader加载ELF格式的OS的过程

## exercise5 实现函数调用堆栈跟踪函数

仔细阅读参考手册[函数堆栈]()即可

思考堆栈底指向的是哪个位置（并不是bootloader起始，而是bootmain的起始，因为那时才设置了栈）

## exercise6 完善中断初始化和处理

> Harry-Chen 注意到在设置 IDT 时答案没有将 SYSCALL 对应的中断号标记为 istrap，也就是说没有允许再次中断，这其实是不正确的。

理解trapframe各部分的意思

ucore中的段选择子？？？

不太了解idt_init中使用的宏的值代表的意思，具体就是KERNEL_CS和GD_KTEXT的区别，GD_KTEXT表示什么意思？

## exercise7 TODO

参考[中断与异常](https://chyyuu.gitbooks.io/ucore_os_docs/content/lab1/lab1_3_3_2_interrupt_exception.html)的讲解，能否理解扩展实验的实现，就是要理解k2u和u2k的过程中因为特权级的切换，硬件会有什么额外的行为，我们需要为此保存什么，恢复什么。同时要理解trapframe结构体的结构和作用。理解tf->esp和esp的区别，并且在返回中断调用时理解tf->esp和esp的值所代表的意思是非常重要的。

Q:为什么答案中并没有把四个ds，gs，es，fs全部更新？ 

A: lab4_3_3_2_create_kthread_initproc, "并设置中断帧的代码段（tf.tf_cs）和数据段(tf.tf_ds/tf_es/tf_ss)为内核空间的段（KERNEL_CS/KERNEL_DS）"

Q: k2u时中断返回后，ss更新成用户态的，但是esp任然指向调用中断之前的esp，虽然这样中断调用的包装函数可以正常返回，但是用户态的esp难道要一直和内核态的esp一样吗？还是其实完整的过程还有将esp保存到TSS中，然后将esp修改为用户态esp（或者用户态ebp）的值，这样的话中断之前的栈就没了，函数也就没办法返回，那么这个函数本身就不会返回了，或者是在用户态返回内核态之前不会返回了（但是在lab1中没有体现出来）。然后一个更严重的问题出现了，改了cs之后返回了执行的还是之前的代码吗？

A: 这里我弄混了一个东西，在lab1中，是只有一个内核进程（线程），并且没有用户线程。虽然在exercise7中进行了特权级的切换，但是任然是一个线程，甚至，用户态和内核态所用的栈也是同一个。（更多内容可以参考lab5）

注意TSS在中断时发生特权级转换的时候的行为，硬件在u2k进入中断处理之前会利用TSS中保存的内核栈ss和esp自动切换成内核栈。阅读代码，找到TSS初始化的汇编代码

## exercise8