kern/debug/kdebug.c

kern/trap/trap.c

kern/init/init.c

# Lab1 实验关键点记录

## 疑问

Q: TSS段的作用是什么？（lab7任然有问题没有解决）

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

为什么答案中并没有把四个ds，gs，es，fs全部更新？？？是忘了吗

> 注意TSS在中断时发生特权级转换的时候的行为，硬件在u2k进入中断处理之前会利用TSS中保存的内核栈ss和esp自动切换成内核栈。但是实验手册没有说k2u返回时发生的特权切换会对TSS有什么影响，看文档也没找到。我觉得保存内核的ss，esp到TSS中应该是软件实现
>
> TODO 这个exercise困扰了我很久，我现在任然没有想明白，k2u时中断返回后，ss更新成用户态的，但是esp任然指向调用中断之前的esp，虽然这样中断调用的包装函数可以正常返回返回，但是用户态的esp难道要一直和内核态的esp一样吗？还是其实完整的过程还有将esp保存到TSS中，然后将esp修改为用户态esp（或者用户态ebp）的值，这样的话怎中断之前的栈就没了，函数也就没办法返回，那么这个函数本身就不会返回了，或者是在用户态返回内核态之前不会返回了（但是在lab1中没有体现出来？？）
> 
> 然后一个更严重的问题出现了，改了cs之后返回了执行的还是之前的代码吗？
>
> 综合这两个问题，我应该是漏掉了一些细节，已经想一天了，没有必要**一直**在这上面纠结，先做后面的实验。
>
> 暂时给一个自圆其说的答案，因为ucore中实际上等于没有分段，所以ss在用户和内核空间作用是一样的，相同的esp在内核和用户态指向的是相同位置，实际上内核和用户用的还是同一个栈。那么如何实现真正的切换栈呢？？？

## exercise8