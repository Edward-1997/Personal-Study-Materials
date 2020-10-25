一、Linux内核启动流程

1、x86进入实模式从0xFFFF0开始自动执行程序代码，这个地址通常是 ROM-BIOS 中的地址。 

2、在物理地址 0 处开始设置和初始化中断向量。

3、它将可启动设备的第一个扇区（磁盘引导扇区， 512 字节）读入内存绝对地址 0x7C00 处，并跳转到这个地方开始引导启动机器运行了。 

4、boot/bootsect.S  将由 BIOS 读入到内存绝对地址 0x7C00（ 31KB） 处，当它被执行时就会把自己移动到内存绝对地址 0x90000（ 576KB） 处，并把启动设备盘中后 2KB 字节代码（ boot/setup.S） 读入到内存 0x90200 处  

5、内核的其他部分（ system 模块）则被读入到从内存地址 0x10000（ 64KB）开始处  

6、setup 程序将会把 system 模块移动到物理内存起始位置处，这样 system 模块中代码的地址也
即等于实际的物理地址，便于对内核代码和数据进行操作  

![6-1系统加电启执行程序顺序.png](./picture/6-1系统加电启执行程序顺序.png)

各部分在内存中的位置变化如下图：

![6-2启动引导时内核在内存中的位置和移动后的位置](.\picture\6-2启动引导时内核在内存中的位置和移动后的位置.png)

6、加载IDT、 GDT 以及 LDT ，确认处理器和协处理器，设置分页工作， 最终会调用执行 init/main.c 中的 main()代码  

![6-3Linux0.12内核磁盘分布图.png](./picture/6-3Linux0.12内核磁盘分布图.png)

1.44MB 磁盘盘片两面各有 80个磁道（柱面），每磁道有 18 个扇区，共有 2880 个扇区  

boot:1扇区

setup：4扇区

system：260扇区

```assembly
1 !
2 ! SYS_SIZE is the number of clicks (16 bytes) to be loaded.
3 ! 0x3000 is 0x30000 bytes = 196kB, more than enough for current
4 ! versions of linux
! SYS_SIZE 是要加载的系统模块长度。 长度单位是节（ paragraph） ，每节 16 字节。 这里 0x3000 共为
! 0x30000 字节=196KB。 若以 1024 字节为 1KB 计，则应该是 192KB。对于当前内核版本这个空间长度
! 已足够了。当该值为 0x8000 时，表示内核最大为 512KB。因为内存 0x90000 处开始存放移动后的
! bootsect 和 setup 的代码，因此该值最大不得超过 0x9000（表示 584KB）。
5 !
! 头文件 linux/config.h 中定义了内核用到的一些常数符号和 Linus 自己使用的默认硬盘参数块。
! 例如其中定义了以下一些常数：
! DEF_SYSSIZE = 0x3000 - 默认系统模块长度。单位是节，每节为 16 字节；
! DEF_INITSEG = 0x9000 - 默认本程序代码移动目的段位置；
! DEF_SETUPSEG = 0x9020 - 默认 setup 程序代码段位置；
! DEF_SYSSEG = 0x1000 - 默认从磁盘加载系统模块到内存的段位置。
6 #include <linux/config.h>
7 SYSSIZE = DEF_SYSSIZE ! 定义一个标号或符号。指明编译连接后 system 模块的大小。
8 !
9 ! bootsect.s (C) 1991 Linus Torvalds
10 ! modified by Drew Eckhardt
11 !
12 ! bootsect.s is loaded at 0x7c00 by the bios-startup routines, and moves
13 ! iself out of the way to address 0x90000, and jumps there.
14 !
15 ! It then loads 'setup' directly after itself (0x90200), and the system
16 ! at 0x10000, using BIOS interrupts.
17 !
18 ! NOTE! currently system is at most 8*65536 bytes long. This should be no
19 ! problem, even in the future. I want to keep it simple. This 512 kB
20 ! kernel size should be enough, especially as this doesn't contain the
21 ! buffer cache as in minix
22 !
23 ! The loader has been made as simple as possible, and continuos
24 ! read errors will result in a unbreakable loop. Reboot by hand. It
25 ! loads pretty fast by getting whole sectors at a time whenever possible.
    !
	! 以下是前面文字的译文：
	! bootsect.s (C) 1991 Linus Torvalds
 	! Drew Eckhardt 修改
!
! bootsect.s 被 ROM BIOS 启动子程序加载至 0x7c00 (31KB)处，并将自己移到了地址 0x90000
! (576KB)处，并跳转至那里。
!
! 它然后使用 BIOS 中断将'setup'直接加载到自己的后面(0x90200)(576.5KB)，并将 system 加
! 载到地址 0x10000 处。
!
! 注意! 目前的内核系统最大长度限制为(8*65536)(512KB)字节，即使是在将来这也应该没有问
! 题的。我想让它保持简单明了。这样 512KB 的最大内核长度应该足够了，尤其是这里没有象
! MINIX 中一样包含缓冲区高速缓冲。
!
! 加载程序已经做得够简单了，所以持续地读操作出错将导致死循环。只能手工重启。只要可能，
! 通过一次读取所有的扇区，加载过程可以做得很快。
26
! 伪指令（伪操作符） .globl 或.global 用于定义随后的标识符是外部的或全局的，并且即使不
! 使用也强制引入。 .text、 .data 和.bss 用于分别定义当前代码段、数据段和未初始化数据段。
! 在链接多个目标模块时，链接程序（ ld86） 会根据它们的类别把各个目标模块中的相应段分别
! 组合（合并）在一起。这里把三个段都定义在同一重叠地址范围中，因此本程序实际上不分段。
! 另外，后面带冒号的字符串是标号，例如下面的'begtext:'。
! 一条汇编语句通常由标号（可选）、指令助记符（指令名）和操作数三个字段组成。标号位于
! 一条指令的第一个字段。它代表其所在位置的地址，通常指明一个跳转指令的目标位置。
27 .globl begtext, begdata, begbss, endtext, enddata, endbss
28 .text ! 文本段（代码段）。
29 begtext:
30 .data ! 数据段。
31 begdata:
32 .bss ! 未初始化数据段。
33 begbss:
34 .text ! 文本段（代码段）。
35
! 下面等号'='或符号'EQU'用于定义标识符或标号所代表的值。
36 SETUPLEN = 4 ! nr of setup-sectors
! setup 程序代码占用磁盘扇区数(setup-sectors)值；
37 BOOTSEG = 0x07c0 ! original address of boot-sector
! bootsect 代码所在内存原始段地址；
38 INITSEG = DEF_INITSEG ! we move boot here - out of the way
! 将 bootsect 移到位置 0x90000 - 避开系统模块占用处；
39 SETUPSEG = DEF_SETUPSEG ! setup starts here
! setup 程序从内存 0x90200 处开始；
40 SYSSEG = DEF_SYSSEG ! system loaded at 0x10000 (65536).
! system 模块加载到 0x10000（ 64 KB） 处；
41 ENDSEG = SYSSEG + SYSSIZE ! where to stop loading
! 停止加载的段地址；
42
43 ! ROOT_DEV & SWAP_DEV are now written by "build".
! 根文件系统设备号 ROOT_DEV 和交换设备号 SWAP_DEV 现在由 tools 目录下的 build 程序写入。
! 设备号 0x306 指定根文件系统设备是第 2 个硬盘的第 1 个分区。 当年 Linus 是在第 2 个硬盘上
! 安装了 Linux 0.11 系统，所以这里 ROOT_DEV 被设置为 0x306。在编译这个内核时你可以根据
! 自己根文件系统所在设备位置修改这个设备号。 例如，若你的根文件系统在第 1 个硬盘的第 1 个
! 分区上，那么该值应该为 0x0301，即（ 0x01, 0x03） 。 这个设备号是 Linux 系统老式的硬盘设备
! 号命名方式，硬盘设备号具体值的含义如下：
! 设备号=主设备号*256 + 次设备号（也即 dev_no = (major<<8) + minor ）
! （主设备号： 1-内存,2-磁盘,3-硬盘,4-ttyx,5-tty,6-并行口,7-非命名管道）
! 0x300 - /dev/hd0 - 代表整个第 1 个硬盘；
! 0x301 - /dev/hd1 - 第 1 个盘的第 1 个分区；
! …
! 0x304 - /dev/hd4 - 第 1 个盘的第 4 个分区；
! 0x305 - /dev/hd5 - 代表整个第 2 个硬盘；
! 0x306 - /dev/hd6 - 第 2 个盘的第 1 个分区；
! …
! 0x309 - /dev/hd9 - 第 2 个盘的第 4 个分区；
! 从 Linux 内核 0.95 版后就已经使用与现在内核相同的命名方法了。
44 ROOT_DEV = 0 ! 根文件系统设备使用与系统引导时同样的设备；
45 SWAP_DEV = 0 ! 交换设备使用与系统引导时同样的设备；
46
! 伪指令 entry 迫使链接程序在生成的执行程序（ a.out）中包含指定的标识符或标号。这里是
! 程序执行开始点。 49 -- 58 行作用是将自身(bootsect)从目前段位置 0x07c0(31KB) 移动到
! 0x9000(576KB) 处，共 256 字（ 512 字节），然后跳转到移动后代码的 go 标号处，也即本程
! 序的下一语句处。
47 entry start ! 告知链接程序，程序从 start 标号开始执行。
48 start:
49 mov ax,#BOOTSEG ! 将 ds 段寄存器置为 0x7C0；
50 mov ds,ax
51 mov ax,#INITSEG ! 将 es 段寄存器置为 0x9000；
52 mov es,ax
53 mov cx,#256 ! 设置移动计数值=256 字（ 512 字节）；
54 sub si,si ! 源地址 ds:si = 0x07C0:0x0000
55 sub di,di ! 目的地址 es:di = 0x9000:0x0000
56 rep ! 重复执行并递减 cx 的值，直到 cx = 0 为止。
57 movw ! 即 movs 指令。从内存[si]处移动 cx 个字到[di]处。
58 jmpi go,INITSEG ! 段间跳转（ Jump Intersegment） 。这里 INITSEG
! 指出跳转到的段地址， 标号 go 是段内偏移地址。
59
    ! 从下面开始， CPU 在已移动到 0x90000 位置处的代码中执行。
    ! 这段代码设置几个段寄存器，包括栈寄存器 ss 和 sp。栈指针 sp 只要指向远大于 512 字节偏移
    ! （即地址 0x90200） 处都可以。因为从 0x90200 地址开始处还要放置 setup 程序，而此时 setup
    ! 程序大约为 4 个扇区，因此 sp 要指向大于（ 0x200 + 0x200 * 4 +堆栈大小）位置处。这里 sp
    ! 设置为 0x9ff00 - 12（参数表长度），即 sp = 0xfef4。在此位置之上会存放一个自建的驱动
    ! 器参数表，见下面说明。实际上 BIOS 把引导扇区加载到 0x7c00 处并把执行权交给引导程序时，
    ! ss = 0x00， sp = 0xfffe。
    ! 另外，第 65 行上 push 指令的期望作用是想暂时把段值保留在栈中，然后等下面执行完判断磁道
    ! 扇区数后再弹出栈，并给段寄存器 fs 和 gs 赋值（第 109 行）。但是由于第 67、 68 两语句修改
    ! 了栈段的位置，因此除非在执行栈弹出操作之前把栈段恢复到原位置，否则这样设计就是错误的。
    ! 因此这里存在一个 bug。改正的方法之一是去掉第 65 行，并把第 109 行修改成“ mov ax,cs”。
60 go: mov ax,cs ! 将 ds、 es 和 ss 都置成移动后代码所在的段处(0x9000)。
61 mov dx,#0xfef4 ! arbitrary value >>512 - disk parm size
62
63 mov ds,ax
64 mov es,ax
65 push ax ! 临时保存段值（ 0x9000），供 109 行使用。（滑头!）
66
67 mov ss,ax ! put stack at 0x9ff00 - 12.
68 mov sp,dx
69 /*
70 * Many BIOS's default disk parameter tables will not
71 * recognize multi-sector reads beyond the maximum sector number
72 * specified in the default diskette parameter tables - this may
73 * mean 7 sectors in some cases.
74 *
75 * Since single sector reads are slow and out of the question,
76 * we must take care of this by creating new parameter tables
77 * (for the first disk) in RAM. We will set the maximum sector
78 * count to 18 - the most we will encounter on an HD 1.44.
79 *
80 * High doesn't hurt. Low does.
81 *
82 * Segments are as follows: ds=es=ss=cs - INITSEG,
83 * fs = 0, gs = parameter table segment
84 */
    /*
    * 对于多扇区读操作所读的扇区数超过默认磁盘参数表中指定的最大扇区数时，
    * 很多 BIOS 将不能进行正确识别。在某些情况下是 7 个扇区。
    *
    * 由于单扇区读操作太慢，不予以考虑，因此我们必须通过在内存中重创建新的
    * 参数表（为第 1 个驱动器）来解决这个问题。我们将把其中最大扇区数设置为
    * 18 -- 即在 1.44MB 磁盘上会碰到的最大数值。
    *
    * 这个数值大了不会出问题，但是太小就不行了。
    *
    * 段寄存器将被设置成： ds=es=ss=cs - 都为 INITSEG（ 0x9000） ，
    * fs = 0， gs = 参数表所在段值。
    */
85 
    ! BIOS 设置的中断 0x1E 实际上并不是一个中断，其对应中断向量的地方被放置了软驱参数表的地址。
    ! 该中断向量位于内存 0x1E * 4 = 0x78 处。这段代码首先从内存 0x0000:0x0078 处复制原软驱参数表
    ! 到 0x9000:0xfef4 处，然后修改表中偏移 4 处的每磁道最大扇区数为 18。 表长 12 字节。
86
87 push #0 ! 置段寄存器 fs = 0。
88 pop fs ! fs:bx 指向存有软驱参数表地址处（指针的指针）。
89 mov bx,#0x78 ! fs:bx is parameter table address
    ! 指令 seg fs 表示其下一条语句的操作数在 fs 段中。 该指令仅影响其下一条语句。这里把 fs:bx
    ! 所指内存位置处的表地址放到寄存器对 gs:si 中作为原地址。寄存器对 es:di = 0x9000:0xfef4
    ! 作为目的地址。
90 seg fs
91 lgs si,(bx) ! gs:si is source
92
93 mov di,dx ! es:di is destination ! dx=0xfef4，在 61 行被设置。
94 mov cx,#6 ! copy 12 bytes
95 cld ! 清方向标志。复制时指针递增。
96
97 rep ! 复制 12 字节的软驱参数表到 0x9000:0xfef4 处。
98 seg gs
99 movw
100
101 mov di,dx ! es:di 指向新表，然后修改表中偏移 4 处的最大扇区数。
102 movb 4(di),*18 ! patch sector count
103
104 seg fs ! 让中断向量 0x1E 的值指向新表。
105 mov (bx),di
```

