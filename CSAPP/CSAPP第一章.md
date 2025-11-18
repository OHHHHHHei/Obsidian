# 计算机系统漫游
## 字节
01组成的为序列（比特），8个比特称为1字节。现代计算机使用ASCII标准表示文本字符，
## 编译
预处理器->编译器->汇编器->链接器

![[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=39&rect=29,246,453,328&color=yellow|p.39]]

hello.c->hello.i->hello.s->hello.0->hello.exe
## 系统的硬件组成
### 1.总线
> ([[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=41&selection=388,1,420,1&color=yellow|📖]])
> 它携带信息字节并负责在各个部件间传递。通常总线被设计成传送定长的字节块 ，也就是字（word)。字中的字节数（即字长）是一个基本的系统参数 ，各个系统中都不尽相同。

### 2.I/O设备
输入输出设备
![[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=42&rect=23,378,463,616&color=yellow|📖]]

### 3.主存
> ([[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=42&selection=207,0,219,1&color=yellow|📖]])
> 主存是一个临时存储设备，在处理器执行程序时，用来存放程序和程序处理的数据。
### 4.处理器
> ([[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=42&selection=353,0,399,1&color=yellow|📖]])
> 中央处理单元（CPU), 简称处理器，是解释（或执行）存储在主存中指令的引擎。处理器的核心是一个大小为一个字的存储设备（或寄存器 ）， 称为程序计数器（PC)。在任何时刻，PC 都指向主存中的某条机器语言指令（即含有该条指令的地址）
## 运行程序
## 高速缓存
> ([[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=45&selection=461,1,475,3&color=yellow|📖]])
> 意识到高速缓存存储器存在的应用程序员能够利用高速缓存将程序的性能提高一个数量级


![[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=45&rect=54,140,412,269&color=yellow|📖]]
## 存储设备的层次结构
![[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=46&rect=46,361,454,603&color=yellow|📖]]
> ([[深入理解计算机系统（原书第3版）扫描版可编辑 (Randal E.Bryant David OHallaron) (Z-Library) 1.pdf#page=46&selection=22,1,91,3&color=yellow|📖]])
> 在这个层次结构中，从上至下，设备的访问速度越来越慢、容量越来越大，并且每字节的造价也越来越便宜。寄存器文件在层次结构中位于最顶部 ，也就是第〇级或记为 L0ÿ 这里我们展示的是三层高速缓存 L1 到 L3, 占据存储器层次结构的第 1 层到第 3 层。主存在第 4 层，以此类推。
## 操作系统管理硬件


