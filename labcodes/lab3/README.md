kern/mm/vmm.c

kern/mm/swap_fifo.c

# 实验记录

注释掉了kern/init/init.c中的grade_backtrace()

## exercise1

> https://github.com/Harry-Chen/ucore_os_lab/blob/master/labcodes/lab3/lab3.md
> 
> 如果在页错误处理程序中又发生了页错误，则依旧会像在其他代码中发生页错误一样进入异常服务程序转到页错误处理函数，不同的是不会发生特权级切换。事实上，一个鲁棒的操作系统不应该在任何时候将内核使用的内存页面换出，因此这是不该发生的。
> 
> 特别地，如果在发生页错误，CPU 试图处理时又发生了页错误（比如访问 IDT 指向的处理程序地址导致页错误），这会被处理器认为是双重错误 （Double Fault），并跳转到双重错误处理程序。一般而言双重错误是不可恢复的，系统应该关闭对应的程序。如果再次触发页错误，则是 Triple Fault，会直接导致处理器重置。

## exercise2


## 扩展练习 Challenge 1：实现识别 dirty bit 的 extended clock 页替换算法（需要编程）

## 扩展练习 Challenge 2：实现不考虑实现开销和效率的 LRU 页替换算法（需要编程）