﻿[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">进程内存分布
</h1>

　　之前一直在分析栈，栈这个东西的作用也介绍得差不多了，
但是栈在哪儿还没有搞清楚，以及堆、代码、全局变量它们在哪儿，
这都牵涉到进程的内存分布。

## linux 0.01 的进程内存分布

　　内存分布随着操作系统的更新换代，越来越科学合理，
也越来越复杂，所以我们还是先了解一下早期操作系统的典型
linux 0.01 的进程的内存分布：

　　linux 0.01 的一个进程固定拥有64MB的线性内存空间
（ACM竞赛中单个程序的最大内存占用限制为64MB，
这肯定有猫腻O(∩_∩)O~），各个进程挨个放置在一张页目录表中，
一个页目录表可管理4G的线性空间，因此 linux0.01 最多有
64个进程。每个进程的内存分布如下：

![mem1](http://fmn.rrimg.com/fmn061/20121206/1925/original_tXyg_61c80000059b118d.jpg)

* .text 里存的是机器码序列
* .rodata 里存的是源字符串等只读内容
* .data 里存的是初始化的全局变量
* .bss 上一篇介绍过了，存的是未初始化的全局变量
* 堆、栈就不用介绍了吧！

`　　`.text .rodata .data .bss 是常驻内存的，
<b>也就是说进程从开始运行到进程僵死它们一直蹲在那里，
所以访问它们用的是常量地址</b>；而栈是不断的加帧（函数调用）
、减帧（函数返回）的，帧内的局部变量只能用相对于当前
esp（指向栈顶）或 ebp（指向当前帧）的相对地址来访问。

　　栈被放置在高地址也是有原因的：
<b>调用函数（加帧）是减 esp 的，函数返回（减帧）是加 esp 的，
调用在前，所以栈是向低地址扩展的，放在高地址再合适不过了</b>。

## 现代操作系统的进程内存分布

　　认识了 linux 0.01 的内存分布后，
再看看现代操作系统的内存分布发生了什么变化：

　　<b>首先</b>，linux 0.01 进程的64MB内存限制太过时了，
现在的程序都有潜力使用到 2GB、3GB 的内存空间
（每个进程一张页目录表），当然，机器有硬伤的话也没办法，
我的电脑就只有 2GB 的内存，想用 3GB 的内存是没指望了。
但也不是有4GB内存就可以用4GB（32位），
因为操作系统还要占个坑呢！
现代 linux 中 0xC0000000 以上的 1GB 空间是操作系统专用的，
而 linux 0.01 中第1个 64MB 是操作系统的坑，
所以别的进程完全占有它们的 64MB，
也不用跟操作系统客气。

　　<b>其次</b>，linux 0.01只有进程没有线程，
但是现代 linux 有多线程了
（linux 的线程其实是个轻量级的进程），
<b>一个进程的多个线程之间共享全局变量、堆、打开的文件……
但栈是不能共享的</b>：栈中各层函数帧代表着一条执行线索，
一个线程是一条执行线索，所以每个线程独占一个栈，
而这些栈又都必须在所属进程的内存空间中。

　　根据以上两点，进程的内存分布就变成了下面这个样子：

![mem2](http://fmn.xnpic.com/fmn056/20121206/1925/original_22bc_2556000005a8118c.jpg)

　　再者，如果把动态装载的动态链接库也考虑进去的话，
上面的分布图将会更加"破碎"。

　　<b>如果我们的程序没有采用多线程的话，
一般可以简单地认为它的内存分布模型是 linux 0.01 的那种</b>。

[回目录][content]