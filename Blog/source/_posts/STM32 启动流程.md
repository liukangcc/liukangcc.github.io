---
title: stm32
index_img: /img/rust.png
date: 2021-08-07 09:05:19
tags: stm32
---

分享这篇文章，谈一下STM32启动流程。如果读者朋友已经有过汇编相关基础，能够够好理解本文内容。汇编语言是比C语言更接近机器底层的编程语言，能让我们更好的理解和操纵硬件底层。



# STM32三种启动模式



下好程序后，重启芯片时，SYSCLK的第4个上升沿，BOOT引脚的值将被锁存，这就是所谓的启动过程。



STM32上电或者复位后，代码区始终从0x00000000开始，其实就是将存储空间的地址映射到0x00000000中。三种启动模式如下：



- 从**主闪存存储器**启动，将主Flash地址0x08000000映射到0x00000000，这样代码启动之后就相当于从0x08000000开始。主闪存存储器是STM32内置的Flash，作为芯片内置的Flash，是正常的工作模式。一般我们使用JTAG或者SWD模式下载程序时，就是下载到这个里面，重启后也直接从这启动程序。
- 从**系统存储器**启动。首先控制BOOT0、BOOT1管脚，复位后，STM32与上述两种方式类似，从系统存储器地址0x1FFF F000开始执行代码。系统存储器是芯片内部一块特定的区域，芯片出厂时在这个区域预置了一段Bootloader，就是通常说的ISP程序。这个区域的内容在芯片出厂后没有人能够修改或擦除，即它是一个ROM区。启动的程序功能由厂家设置。系统存储器存储的其实就是STM32自带的bootloader代码。
- 从**内置SRAM**启动，将SRAM地址0x20000000映射到0x00000000,这样代码启动之后就相当于从0x20000000开始。内置SRAM，也就是STM32的内存，既然是SRAM，自然也就没有程序存储的能力了，这个模式一般用于程序调试。假如我只修改了代码中一个小小的地方，然后就需要重新擦除整个Flash，比较的费时，可以考虑从这个模式启动代码，用于快速的程序调试，等程序调试完成后，在将程序下载到SRAM中。

  

用户可以通过设置BOOT1和BOOT0引脚的状态，来选择在复位后的启动模式。STM32三种启动模式对应的存储介质均是芯片内置的，如下图：



![图片](https://mmbiz.qpic.cn/mmbiz_png/K9mVOHgVt7y6bOlfWsByZCnpw6hyK1nfOxPR256SlLp2aEvsmWqM0IlwL3FiavYlVibM7wznkwJqvzqR5908TrKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 串口下载程序原理

从系统存储器启动，这种模式启动的程序功能是由厂家设置的。一般来说，这种启动方式用的比较少。系统存储器是芯片内部一块特定的区域，STM32在出厂时，由ST在这个区域内部预置了一段BootLoader，也就是我们常说的ISP程序，这是一块ROM，出厂后无法修改。



一般来说，我们选用这种启动模式时，是为了从串口下载程序，因为在厂家提供的BootLoader中，提供了串口下载程序的固件，可以通过这个BootLoader将程序下载到系统的Flash中。



这个下载方式需要以下步骤：



- 将BOOT0设置为1，BOOT1设置为0，然后按下复位键，这样才能从系统存储器启动BootLoader；

- 在BootLoader的帮助下，通过串口下载程序到Flash中；

- 程序下载完成后，又有需要将BOOT0设置为GND，手动复位，这样，STM32才可以从Flash中启动。

  

## 从汇编代码分析STM32启动过程



STM32的启动文件与编译器有关，不同编译器，它的启动文件不同。虽然启动文件（汇编）代码各有不同，但它们原理类似，都属于汇编程序。拿基于MDK-ARM的启动文件来举例，说一下要点内容。在基于MDK的启动文件开始，有一段汇编代码是分配堆栈大小的。



![图片](https://mmbiz.qpic.cn/mmbiz_png/K9mVOHgVt7y6bOlfWsByZCnpw6hyK1nffRoftaQRLbibB7Aiaf3V8gaicNLwnrpEtolqzictSFVZokReibx9wpRmzrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

这里重点知道堆栈数值大小就行。还有一段AREA（区域），表示分配一段堆栈数据段。可以使用STM32CubeMX对上面的数值大小进行配置**：**



![图片](https://mmbiz.qpic.cn/mmbiz_jpg/K9mVOHgVt7y6bOlfWsByZCnpw6hyK1nfbwbHV17McobBlMXkNQt6DOOMcIdGgcsgqkNf9TIYUicEw65TwPspycA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 

在IAR中，是通过工程配置堆栈大小：



![图片](https://mmbiz.qpic.cn/mmbiz_png/K9mVOHgVt7y6bOlfWsByZCnpw6hyK1nfTr5DfOu0GxjcZ1f4xjVRjd109CmuuRtFwWbx8YVGqCr9L6hXsibC3eA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



看下面的汇编代码，程序上电之后，是跳到Reset_Handler这个位置。



![图片](https://mmbiz.qpic.cn/mmbiz_png/K9mVOHgVt7y6bOlfWsByZCnpw6hyK1nfKLoiars6iaOlMjib9jk6ic0sTMK1ZuvLgq0yJ0zrU3oa32ByQNUIbQia4mw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



知道代码是从Reset_Handler开始执行，再来看如下Reset_Handler汇编代码。在启动的时候，执行了SystemInit这个函数。



![图片](https://mmbiz.qpic.cn/mmbiz_jpg/K9mVOHgVt7y6bOlfWsByZCnpw6hyK1nf0JlsyiafvmhvhPNlalib5avTkiaGExaP66Y1qX1Zjva0nmcaD1blcCyRQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



执行完SystemInit函数，初始化了系统时钟，之后跳转到main函数执行。