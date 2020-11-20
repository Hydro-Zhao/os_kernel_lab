# Lab1 实验关键点记录

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

## exercise7

参考[中断与异常](https://chyyuu.gitbooks.io/ucore_os_docs/content/lab1/lab1_3_3_2_interrupt_exception.html)的讲解，能否理解扩展实验的实现，就是要理解k2u和u2k的过程中因为特权级的切换，硬件会有什么额外的行为，我们需要为此保存什么，恢复什么。同时要理解trapframe结构体的结构和作用。理解tf->esp和esp的区别，并且在返回中断调用时理解tf->esp和esp的值所代表的意思是非常重要的。

注意TSS在中断时发生特权级转换的时候的行为，硬件在u2k进入中断处理之前会利用TSS中保存的内核栈ss和esp自动切换成内核栈。但是实验手册没有说k2u返回时发生的特权切换会对TSS有什么影响，我也还没有看文档，不过我想的是（eeeeeeee）也会保存内核的ss，esp到TSS中，而且在这样的理解执行做的实验是对的。

我的exercise7最终的结果和答案以及Harry-Chen的都不太一样。

为什么答案中并没有把四个ds，gs，es，fs全部更新？？？是忘了吗

## exercise8