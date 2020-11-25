kern/trap/trap.c

kern/process/proc.c

kern/schedule/default_sched.c -rename-> kern/schedule/default_sched_c

kern/schedule/default_sched_stride_c -rename&modify-> kern/schedule/default_sched.c

* TODO Stride Scheduling 调度算法的实现

    **首先阅读论文**，再看实验手册[基本思路](https://github.com/LearningOS/ucore_os_docs/blob/master/lab6/lab6_3_6_1_basic_method.md) [使用优先队列实现 Stride Scheduling](https://github.com/LearningOS/ucore_os_docs/blob/master/lab6/lab6_3_6_2_priority_queue.md)

    BIG_STRIDE数值的确定

    stride_pick_next的实现?(将堆顶元素的 stride 加上 BIG_STRIDE / PRIORITY，并返回)

* TODO 实际测试通过

    labcodes_answer和Harry-Chen的答案和labcodes_answer中，本repo中labcodes_answer的测试效果算是最好的了，但是仍然没有完全通过。见make_grade_output.txt和make_grade_output_answer.txt

# 实验记录

## 实验手册

内核抢占点 总结了ucore中进程切换点

## 练习1: 使用 Round Robin 调度算法（不需要编码）

## 练习2: 实现 Stride Scheduling 调度算法（需要编码）

## 扩展练习 Challenge 1 ：实现 Linux 的 CFS 调度算法

## 扩展练习 Challenge 2 ：在ucore上实现尽可能多的各种基本调度算法(FIFO, SJF,...)，并设计各种测试用例，能够定量地分析出各种调度算法在各种指标上的差异，说明调度算法的适用范围。