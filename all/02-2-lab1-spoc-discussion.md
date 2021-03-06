# lec4: lab1 SPOC思考题

##**提前准备**
（请在上课前完成）

 - 完成lec4的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。
 - 了解x86的保护模式，段选择子，全局描述符，全局描述符表，中断描述符表等概念，以及如何读写，设置等操作
 - 了解Linux中的ELF执行文件格式
 - 了解外设:串口，并口，时钟，键盘,CGA，已经如何对这些外设进行编程
 - 了解x86架构中的mem地址空间和io地址空间
 - 了解x86的中断处理过程（包括硬件部分和软件部分）
 - 了解GCC的x86/RV内联汇编
 - 了解C语言的可函数变参数编程
 - 了解qemu的启动参数的含义
 - 在piazza上就lec3学习中不理解问题进行提问
 - 学会使用 qemu
 - 在linux系统中，看看 /proc/cpuinfo的内容

## 思考题

### 启动顺序

1. x86段寄存器的字段含义和功能有哪些？

   CS: Code Segment代码段寄存器. 以此为基址加上IP为指令代码的位置.

   DS: Data Segment数据段寄存器. 以此为基址访问程序数据.

   SS: Stack Segment堆栈段寄存器. 为堆栈的底部地址.

   ES: Extra Segment附加段寄存器. 还有FS, GS.

2. x86描述符特权级DPL、当前特权级CPL和请求特权级RPL的含义是什么？在哪些寄存器中存在这些字段？对应的访问条件是什么？

   CPL: 表示当前代码的特权级别. 在CS寄存器低两位.

   DPL: 存在段描述符里, 规定其访问权限.

   RPL: 存在段选择子里, 表示某个进程在希望访问某段时拥有的权限, 可以比CPL严格.

3. 分析可执行文件格式elf的格式 无需回答

### 4.1 C函数调用的实现

### 4.2 x86中断处理过程

1. x86/RV中断处理中硬件压栈内容？用户态中断和内核态中断的硬件压栈有什么不同？

   产生中断后, CPU会跳转到中断处理入口并压栈error_code和trap_no. 然后压入trapframe结构体. 最后压入esp作为参数. 同时如果从用户态切换到内核态的话, 还需要有一个切换进程的操作, 也就是保存用户态的上下文, 寄存器等信息, 将其压栈, 然后进入内核态. 处理完成后再恢复状态回到用户态.

2. 为什么在用户态的中断响应要使用内核堆栈？

   因为中断响应涉及到内核态的操作, 这样可以控制用户态程序的权限, 保证内核的安全.

3. x86中trap类型的中断门与interrupt类型的中断门有啥设置上的差别？如果在设置中断门上不做区分，会有什么可能的后果?

   interrupt类型的中断是内部中断, 往往是由意想不到的程序异常产生的, 这时CPU会禁止其它interrupt, 避免反复触发中断; trap类型的中断是系统有意引起的, 类似于函数调用, 这时CPU不会做任何操作. 不加以区分的话可能导致interrupt被不断触发, 或者trap无法嵌套trap的情况.

### 4.3 练习四和五 ucore内核映像加载和函数调用栈分析

1. ucore中，在kdebug.c文件中用到的函数`read_ebp`是内联的，而函数`read_eip`不是内联的。为什么要设计成这样？

   因为ebp是常规寄存器可以通过movl指令直接获取. eip不是, 必须通过"将eip压栈然后从栈中取出"的方法.

### 4.4 练习六 完善中断初始化和处理

1. CPU加电初始化后中断是使能的吗？为什么？

   一开始中断是不可用的. 需要等待BIOS建立中断向量表等支持.

## 开放思考题

1. 在ucore/rcore中如何修改lab1, 实现在出现除零异常时显示一个字符串的异常服务例程？
2. 在ucore lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
3. GRUB是一个通用的x86 bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)
4. 如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？

## 课堂实践
### 练习一
在Linux系统的应用程序中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。

### 练习二
在ucore/rcore内核中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。
