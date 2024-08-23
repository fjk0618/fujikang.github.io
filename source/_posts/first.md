---
title: first
date: 2024-08-22 15:05:37
author: 符吉康
tags: 
  # 标签没层级 属于多个
  - t1
  - t2
archives:
  - a1
  - a2
categories:
  #单级多个标签
  - [Linux]
  - [Tools]
  # 多级多个标签
  - [Mysql, Oracle]
  - [Java, Golang]
---

# 前言
## 本篇博文是操作系统基础系列之一；本文为作者的原创作品，转载需注明出处；

### Stack 栈的工作原理
#### 后进先出
###### 工作原理如下图所示，
how stack works可以将栈看做是一个队列，只是里面的队列的元素满足“后进先出”的原则，如果硬要找一个类比，就它吧，

how stack works simulate

x86 栈的工作原理
x86 栈的模型
head down
x86 中，栈的工作模式被称为 “head down” 既是“头朝下”的一种内存模型，如图，

x86 - stack - head down

一个栈从 x86 内存的堆栈区（SS）的某个位置开始，向低地址逐渐增长；因此，在 x86 的栈模型中，栈顶指的其实就是指该栈所在内存区域的最低地址，相反，栈底指的就是对应内存的最高地址；

栈单元
8086 既是 IA-16 的架构下，一个栈单元等于 2 个内存单元，既 2 字节
IA-32 或 IA-64 的架构下，一个栈单元等于 4 个内存单元，既 4 字节
备注，这里的栈单元只是对 push 和 pop 指令的描述，一次 push 或者 pop 操作，针对 IA-32 而言，指针是移动 4 个内存单元，但不意味着变量就必须被定义为 4 个内存单元，比如 c 中的 short 变量，只占用 2 个字节；下面简单看一下 short 变量翻译成汇编后，它是如何在栈中给它分配内存空间的，
z.s

int main()
{
    short b = 1;
    short a = 2;
    return a+b;
}
在 IA-32 编译后，得到的汇编指令大致如下，

_main:                                  ## @main
   push  ebp
   mov   ebp, esp
   ; 给 main 函数的本地变量预留空间，
   ; 这里只需要为两个 short 变量分配 4 个字节空间
   ;
   sub   esp, 4
   ; 给变量 b 进行初始化，b = 1；注意 WORD PTR 表示一个字，既表示占用两个字节
   ;
   mov   WORD PTR [ebp - 2], 0
   ; 给变量 a 进行初始化，a = 2
   ;
   mov   WORD PTR [ebp - 4], 1
   ; 将变量 b 赋值给寄存器，准备计算
   ; 
   mov   eax, word ptr [rbp - 4]
   ; 将变量 a 赋值给寄存器，准备计算
   ; 
   mov   ecx, word ptr [rbp - 2]
   add eax, ecx
   ; 还原 esp
   mov esp, ebp
   pop ebp
   ret
可见，当某个变量的大小小于栈单元的时候，使用 WORD PTR、BYTE PRT 对被赋值的内存指定即可；

x86 栈的工作方式
ESP
x86 中有一个保留的寄存器 ESP，Extended Stack Pointer，它始终存放着栈顶地址，如图，假设，这是某个栈的当前状态，

x86 - stack - esp 1假设栈顶存放着字符串“foo”，则 ESP 中保存着栈顶的地址0x9080ABCC（行话常叫做 ESP 指向栈顶元素）；

备注，该指针在不同的硬件架构上的叫法不同，

SP
x86_16 中的寄存器名称, Extended Stack Point
ESP
x86_32 中的寄存器名称, Extended Stack Point
RSP
x86_64 中的寄存器名称
push
push 指令向栈顶新增数据，示例，

push eax
假设 eax 寄存器中存放的是0xDEADBEEF，push 之后，栈的当前状态如图所示，

x86 - stack - after push可以看到，新的数据0xDEADBEEF放到了栈顶位置，同时 ESP 向下移动了一个栈单元继续指向栈顶的位置；因此，这个 push 指令实际上做了两件事情，

sub esp, 4
mov [esp], eax
先是向栈顶移动一个栈单元，

sub esp, 4
因为栈顶对应的是低地址，因此这里需要用 sub 指令减去一个栈单元的大小

因为这里使用的是 32 位的寄存器，因此一个栈单元对应 4 个内存单元，因此，这里需要减 4

然后，将 eax 中的值赋值给 esp 所指向的内存单元中去（需要连续 4 个内存单元来存放）
mov [esp], eax
这样，我们便完成了一次 push 操作，记住，将数据添加到栈顶的同时，ESP 也同样需要向栈顶移动一个栈单元，保证它始终是指向栈顶的；

pop
pop 弹出栈顶元素，示例，我们继续上一小节 push 的例子，

pop eax
我们将栈顶元素弹出并保存到 eax 中后，当前堆栈如图所示，

x86 - stack - after pop可以看到，当栈顶元素弹出以后，ESP 回退一个栈单元，继续指向栈顶，但，要注意的是，虽然栈顶元素已经被弹出，但它并不会从内存中清空；pop 指令等价于做了如下两件事

mov eax, [esp]
add esp, 4
先将 esp 所指向的栈单元（连续的 4 个内存单元）赋值给 eax，然后 esp 的地址回退一个栈单元；

C 函数详细的调用过程
栈是函数调用的核心，函数的参数和返回值是通过栈的方式在函数之间进行存储和传递的，看一个例子，
z.c

int foobar(int a, int b, int c)
{
    int xx = a + 2;
    int yy = b + 3;
    int zz = c + 4;
    int sum = xx + yy + zz;

    return xx * yy * zz + sum;
}

int main()
{
    int b = 1;
    int a = foobar(77, 88, 99);  
    return a+b;
}
使用 gcc 编译

gcc -masm=intel -S z.c -o z.s
编译后得到相关的汇编代码 z.s 的内容如下，（注意，这是在 IA-32 环境下编译后得到的结果）

_foobar:
    ; ebp 是调用函数 _main 的基地址
    ;
    push    ebp

    ; From now on, ebp points to the current stack
    ; frame of the function
    ;
    mov     ebp, esp

    ; Make space on the stack for local variables
    ;
    sub     esp, 16

    ; eax <-- a. eax += 2. then store eax in xx
    ;
    mov     eax, DWORD PTR [ebp+8]
    add     eax, 2
    mov     DWORD PTR [ebp-4], eax

    ; eax <-- b. eax += 3. then store eax in yy
    ;
    mov     eax, DWORD PTR [ebp+12]
    add     eax, 3
    mov     DWORD PTR [ebp-8], eax

    ; eax <-- c. eax += 4. then store eax in zz
    ;
    mov     eax, DWORD PTR [ebp+16]
    add     eax, 4
    mov     DWORD PTR [ebp-12], eax

    ; add xx + yy + zz and store it in sum
    ;
    mov     eax, DWORD PTR [ebp-8]
    mov     edx, DWORD PTR [ebp-4]
    lea     eax, [edx+eax]
    add     eax, DWORD PTR [ebp-12]
    mov     DWORD PTR [ebp-16], eax

    ; Compute final result into eax, which
    ; stays there until return
    ;
    mov     eax, DWORD PTR [ebp-4]
    imul    eax, DWORD PTR [ebp-8]
    imul    eax, DWORD PTR [ebp-12]
    add     eax, DWORD PTR [ebp-16]

    ; The leave instruction here is equivalent to:
    ;
    ;   mov esp, ebp
    ;   pop ebp
    ;
    ; Which cleans the allocated locals and restores ebp.
    ;
    leave
    ret
_main:                                  ## @main
   ; ebp 是调用 _main 函数的函数的栈基址，将其保存，以便 _main 函数退出后恢复该函数的栈基址
   ; 压栈后，同时 esp = esp - 4；
   ;
   push  ebp
   ; 此时的 esp 就是 _main 函数的栈基址既 _main 函数栈开始的地方，将其赋值给 ebp 作为栈基址保存
   ;
   mov   ebp, esp
   ; 给 main 函数的本地变量预留空间，
   ; 这里需要预留三个本地变量的空间，a、b 以及 _foobar 的返回值，也就是 3 个栈单元，因此 -12
   ;
   sub   esp, 12
   ; 给变量 a 进行初始化，a = 0；
   ; 可见，虽然没有在 c 代码中明确初始化便令，但是编译器会做这件事
   ;
   mov   DWORD PTR [ebp - 4], 0
   ; 给变量 b 进行初始化，b = 1
   ;
   mov   DWORD PTR [ebp - 8], 1
   ; 准备 call _foobar 函数，对调用参数77、88、99 从右至左依次压栈
   ; 
   mov   DWORD PTR [esp], 77
   mov   DWORD PTR [esp + 4], 88
   mov   DWORD PTR [esp + 8], 99
   ; call _foobar，call 函数做了如下几件事情
   ; 1) 将下一条指令的地址作为 return address 压栈
   ; 2) jump to _footbar
   ;
   call  _foobar
   ; 向上回退 3 个栈单元，跳过 _foobar 调用参数，既完全退出 _foobar 的栈帧部分
   ;
   add esp, 12
   ; eax 保存的是 _foobar 函数的返回值，赋值给栈的第 3 个栈单元
   ;
   mov   DWORD PTR [ebp - 12], eax
   mov   eax, DWORD PTR [ebp - 12]
   ; 实现 b + _foobar 的返回值，将结果存放在 eax 中
   ;
   add   eax, DWORD PTR [ebp - 8]
   ; 相当于执行
   ; mov   esp, ebp
   ; pop   ebp
   ; 然后清空 _main 栈帧
   ;
   leave
   ret
假设，某个系统的方法 𝛼
 调用了该 main 函数，然后 main 函数又调用了 foobar 函数，因此调用关系链为，
𝛼→main→foobar
调用过程笔者通过注解的方式已经写得非常清楚了，这里我们通过图解的方式看重点，

main 函数执行过程
首先，来看 main 函数中的执行过程，

push ebp
将调用 main 的某方法 𝛼
 的栈基址进行压栈，如图，esp 指向栈顶
stack call process
mov ebp, esp
创建 main 函数的栈基址，并将栈基址地址赋值给 ebp 保存，
stack call process此时，esp 和 ebp 均指向栈顶
sub esp, 12
通过将 esp 向下移动 3 个栈单元为 main 函数的 3 个局部变量分配空间
stack call process此时 esp 指向栈顶，并且为局部变量a、b和foobar函数的返回值预留了三个空的栈单元；
mov DWORD PTR [ebp - 4], 0
初始化a
stack call process
mov DWORD PTR [ebp - 8], 0
初始化b
stack call process
准备调用 _foobar，将调用 foobar 函数的三个参数从右至左分别压栈
压入参数 99,
stack call proc压入参数 88,
stack call proc压入参数 77,
stack call proc
call _foobar
做了如下两件事情，
然后将 call 指令的下一条指令既 add esp, 12 的指令地址作为 return address 进行压栈
stack call proc当 foobar 函数退出后，将会跳转到该指令地址上去执行；
然后，跳转到 _foobar 地址上去执行，既 jmp _foobar，该步骤之后的详细流程参考 foobar 函数执行过程
foobar 函数执行过程
push ebp
将 main 函数的栈基址压栈保存，
stack call proc
mov ebp, esp
创建 foobar 函数的栈基址 ebp，
stack call proc
sub esp, 16
为 foobar 函数在其栈中分配相应的局部变量空间，这里需要分配 4 个栈单元
stack call proc
实现 xx = a+2 并将其结果放到 xx 栈单元中

mov     eax, DWORD PTR [ebp+8]
add     eax, 2
mov     DWORD PTR [ebp-4], eax
stack call proc
同理，yy = b + 3 和 zz = c + 4 也分别放到所对应的栈单元中，如图，
stack call proc
最后，完成 sum = xx + yy + zz 并将结果放在 sum 所对应的栈单元中，
stack call proc注意，这里有一个很重要的概念，如图，蓝色部分被称为一个栈帧，它包含了一个函数在栈中所关联的所有信息；
这一步相关的汇编指令如下，

mov     eax, DWORD PTR [ebp-8]
mov     edx, DWORD PTR [ebp-4]
lea     eax, [edx+eax]
add     eax, DWORD PTR [ebp-12]
mov     DWORD PTR [ebp-16], eax
注意，sum 的结果同时也保存在 eax 中的；

执行，xx * yy * zz + sum，

mov     eax, DWORD PTR [ebp-4]
imul    eax, DWORD PTR [ebp-8]
imul    eax, DWORD PTR [ebp-12]
add     eax, DWORD PTR [ebp-16]
结果依然保存在 eax 中，

leave
leave 指令分别做了如下两件事情，

1) 重定位 esp，使其指向 foobar 栈基址的位置，

mov esp, ebp
stack call proc这一步也就等价于删除了 sum、xx、yy 和 zz 的栈单元；

2) 通过 pop ebp 恢复 main 函数的栈基址
stack call proc这一步等价于删除了 main's ebp 栈单元；
ret
弹出 return address，然后跳转到该地址取址执行，等价于执行如下两条命令，

pop edx
jmp edx
注意，前面提到过，在call _foobar的时候，会顺序将下一条指令，也就是add esp, 12，的地址进行压栈，因此，这里就是跳转到该命令去执行；

通过 ret 我们既回到了 main 函数的栈帧中去执行了，

回到 main 函数后的执行过程
add esp, 12
由 ret 指令跳转回 main 函数后跳转执行的第一条指令；向栈底移动 3 个栈单元，既是完全退出 _foobar 栈帧部分，使得 esp 指向 main 在调用 _foobar 之前的栈顶位置，也就是还原 main 函数栈顶指针 esp，
stack call proc备注，通过 gcc 生成 x86_64 的汇编指令的做法不同，得益于更多的寄存器可以使用，它是先将这三个 foobar 函数的调用参数赋值给三个寄存器中，生成的内容大致如下，

_main:                                  ## @main
   ...
   mov   edi, 77
   mov   esi, 88
   mov   edx, 99
   call  _foobar
可见，在调用 foobar 之前，将三个参数分别赋值给 adi、esi 和 edx 三个寄存器临时存储，然后在 foobar 中将这些参数分别赋值给堆栈中指定的位置即可，

_foobar:                                ## @foobar
   ...
   mov   dword ptr [rbp - 4], edi
   mov   dword ptr [rbp - 8], esi
   mov   dword ptr [rbp - 12], edx
这样做的好处是，我们会得到一个不同的栈结构，如图所示，
stack call proc这样，call 指令只需压入 return address 即可，从而，使得还原 main 函数的栈顶指针 esp 会变得异常的简单，只需要步骤 8.1 中通过 mov esp, ebp 重定位 esp 即可；

将 foobar 的返回值赋值给 main 栈帧中的第三个本地变量中

mov   DWORD PTR [ebp - 12], eax
foobar 的返回值 sum 保存在 eax 中，当前栈帧如图所示，黄色部分表示 main 函数的栈帧
stack call proc
a+b等价于b+foobar()的返回值，等价的汇编执行如下所示，

add   eax, DWORD PTR [ebp - 8]
注意，结果依然是存放在 eax 中的，

leave
同 foobar 执行过程一样，重定位 esp 到 main 的栈基址的位置，然后通过 pop ebp 恢复函数 𝛼
 的栈基址
ret
同 foobar 执行过程一样，恢复 𝛼
 的栈顶指针 esp，然后跳转到 return address 上取址执行
回到 𝛼
 函数后的执行过程
这个过程就好比 foobar→main
 返回执行过程，main 开始执行剩下的部分一样，不再赘述！

栈
如图，
stack call procfoobar 函数所对应的栈是由 ebp 所指向的栈底和 esp 所指向的栈顶的部分构成的，注意，蓝色部分对应的是 foobar 函数所对应的栈帧

ebp
x86 的一个保留寄存器，专门用于保存栈基址，它一直指向栈的开始处；

esp
x86 的一个保留寄存器，专门用于保存栈顶地址，它一直指向栈的最顶部；并且，它就像一个游标一样，当有新的栈元素添加以后，自动的指向这个新的栈单元上；

栈帧
在推导 foobar 函数的执行过程中，可以看到，由 foobar 的局部变量、return address 等构成了一个栈帧，如图所示，
stack call proc可见，一个函数栈帧的栈单元要比它的栈的栈单元多；这个示例中，栈帧中多出的栈单元分别是 77、88、99 和 return address 这 4 个栈单元；

函数的嵌套调用
函数中调用另外一个函数是编程的基础，无论是面向过程还是面向对象，都需要这样的机制，在硬件层面，就是通过栈来实现任意多层函数的嵌套调用的，所以，它的意义非常的巨大！

备注，通常，我们在调试代码的时候，调试中所打印出来的函数调用的层次关系，就是从对应线程的栈中输出的；

Stack Overflow
一般而言，一个程序运行时所使用的栈的大小是有限制的，比如最早的 8086，一个程序运行所能使用的栈大小最多只有 64K，所以，如果当函数嵌套调用的层次过多，分配的栈空间超过了这个最大容量限制，那么必然导致栈溢出既 Stack Overflow；

比如 JVM 默认把每个线程能够使用的栈空间的最大值设为 256K，以便能够同时启动多个线程，但是，256K 对于一些复杂的嵌套调用，特别是复杂的递归调用是不够的，因此，往往我们需要通过设置将占空间调大，比如 java -Xss1M，这样使得每个线程能够使用最多 1M 空间的栈；

程序运行的结构体系：cs + ss
在我的脑海里突然出现了这样的一幅完整的场景，

cs 段的存储器提供了程序执行的指令
ss 段的存储器提供了程序运行的环境
References
call 指令
https://docs.oracle.com/cd/E19455-01/806-3773/instructionset-66/index.html
https://www.aldeid.com/wiki/X86-assembly/Instructions/call
https://c9x.me/x86/html/file_module_x86_id_26.html
ret 指令
https://stackoverflow.comquestions/20129107/what-is-the-x86-ret-instruction-equivalent-to
