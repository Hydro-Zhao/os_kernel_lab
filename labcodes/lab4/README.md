kern/process/proc.c

# 实验记录

注释掉了kern/init/init.c中的grade_backtrace()

## 实验手册

创建第 1 个内核线程 initproc 和 调度并执行内核线程 initproc 部分值得反复阅读，可以弄清楚进程创建过程中context和tf的作用（这个问题非常重要）

## exercise1 分配并初始化一个进程控制块（需要编码）

## exercise2 为新创建的内核线程分配资源（需要编码）

## exercise3 阅读代码，理解 proc_run 函数和它调用的函数如何完成进程切换的。

## 扩展练习 Challenge：实现支持任意大小的内存分配算法