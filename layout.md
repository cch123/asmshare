# 汇编 is so easy。

## 为什么要学习汇编

- 装逼

![](images/do_you_want_power.jpg)
装逼是第一生产力，不服来打我

- 防骗：各种专家和架构师，江湖骗子，Go 语言以外的过往经验

1. 你这么写效率不高，下面可能会有逃逸，你看得按我教你的这么写
2. 我们 java 赋值为 null 就能帮助 GC 判断，所以我也需要用完了 x = nil
3. 这个东西 runtime 里是这样这样那样那样实现的，你信我的准没错
4. Go 的 xxx 类型参数是引用传递的， yyy 类型参数是值传递的，哥当年研究 C艹 的研究了好久，哥的理解肯定比你权威

- 串联应用层和 runtime 实现

## 基本概念

- 程序的编译阶段

```
编译 -> 汇编 -> 优化 -> 链接
```

汇编代码在汇编阶段生成，链接阶段会对生成的汇编代码进行修改。(eg0)

实际上优化过程是在所有过程中都有的，这里我们只说汇编优化。

链接阶段会对汇编结果做调整(如将函数地址从偏移量转换为逻辑地址)

- 机器码

所有汇编指令都可以转换为 0101 的序列，如：

```
>> mov rax,0x1
mnemonic : mov rax,0x1 => hex : 48c7c001000000
```
hex 是机器码的 16 进制编码。

- 指令集

RISC/CISC

intel? 难说。

risc 的新轮子 RISCV。

> https://github.com/riscv

CPU 优化指令集，如 x86 平台：SSE/SSE2/AVX。。。AVX512

> https://github.com/golang/go/wiki/AVX512

- 寄存器

寄存器是中央处理器内的组成部分。寄存器是有限存贮容量的高速存贮部件，它们可用来暂存指令、数据和地址。

```
ah/al => 8 位
ax/bx => 16 位
eax/ebx => 32 位
rax/rbx => 64 位，8 byte
```

```
rax : 0x0000000000000000
```

指令寄存器：pc，表示当前我的代码运行到哪里了。

标志寄存器：eflags，记录各种运算结果的标志位。

- 基础指令

```
mov rax, 0x1 // rax = 1
add rax, 11  // rax = rax + 11
sub rax,1    // rax = rax - 1

// 在寄存器之间搬数据
mov rax, rbx // rax = rbx

// 在寄存器和内存之间搬数据
mov rax, [rbx] // 把 rbx 寄存器的值看作一个地址，把该物理地址的值取出，然后赋值给 rax，即 rax = *rbx

// 在内存之间搬数据
- 你想多了，不可以的

push rax => 将 rax 的值存储到栈顶，并将 rsp 上移 8 个字节
pop rax => 将 rsp 指向的内存位置的 8 个字节移动到 rax，并将 rax 从栈顶下移 8 个字节

jmp addr => 跳转到指定地址
```

函数调用在汇编上的本质：

```
jmp 跳进去
jmp 跳回来
```

cpu 很傻，跳回来的时候我必须到当前过程的下一条指令位置，难道我还需要在被调用过程里知道调用者的过程下一条指令位置？

```
call => push pc; jmp
ret => pop pc; jmp
```

要点，在栈上记录下一条指令位置，即我们常说的 `return address`。

怎么和系统打交道：

```
section .data
message: db 'hello, world!', 10

section .text
global _start

_start:
    mov     rax, 1              ; 'write' syscall number
    mov     rdi, 1              ; stdout descriptor
    mov     rsi, message        ; string address
    mov     rdx, 14             ; string length in bytes
    syscall   ; 重点在这里
```

更具体的可以参考 [这里](https://github.com/cch123/llp-trans/blob/d6b7f46c72e83ac9145d5534c6bc4e690da8d815/part1/assembly-language/writing-hello-world/basic-instructions.md)

和[这里](http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)

## plan9 汇编

- 寄存器名差别

```
rax -> AX
rbx -> BX
r8  -> R8
```

因为移动多少个字节是由指令而不是操作数决定的，所以稍微省心一些。

- 指令对应关系

```
MOVB $1, DI -> mov al, 0x44    // 1 byte
MOVW $0x10, BX -> mov ax, 0x10 // 2 bytes
MOVD $100, DX -> mov eax, 100  // 4 bytes
MOVQ $-10, AX -> mov rax, -10  // 8 bytes
```

可以看到，指令的操作数是反着的，大部分 plan9 和 intel 的指令确实是反着的关系，学习过汇编的同学可能会认为很好啊，这样 plan9 的操作数顺序应该是和 AT&T 一致了，遗憾的是也有例外：

```
Mnemonic | Go operands           | AT&T operands
================================================
BOUNDW   | m16&16, r16           | r16, m16&16
BOUNDL   | m32&32, r32           | r32, m32&32
CMPB     | AL, imm8              | imm8, AL
CMPW     | AX, imm16             | imm16, AX
CMPL     | EAX, imm32            | imm32, EAX
CMPQ     | RAX, imm32            | imm32, RAX
CMPW     | r/m16, imm16          | imm16, r/m16
CMPW     | r/m16, imm8           | imm8, r/m16
CMPW     | r/m16, r16            | r16, r/m16
CMPL     | r/m32, imm32          | imm32, r/m32
CMPL     | r/m32, imm8           | imm8, r/m32
CMPL     | r/m32, r32            | r32, r/m32
CMPQ     | r/m64, imm32          | imm32, r/m64
CMPQ     | r/m64, imm8           | imm8, r/m64
CMPQ     | r/m64, r64            | r64, r/m64
CMPB     | r/m8, imm8            | imm8, r/m8
CMPB     | r/m8, imm8            | imm8, r/m8
CMPB     | r/m8, r8              | r8, r/m8
CMPB     | r/m8, r8              | r8, r/m8
CMPW     | r16, r/m16            | r/m16, r16
CMPL     | r32, r/m32            | r/m32, r32
CMPQ     | r64, r/m64            | r/m64, r64
CMPB     | r8, r/m8              | r/m8, r8
CMPB     | r8, r/m8              | r/m8, r8
CMPPD    | imm8, xmm1, xmm2/m128 | imm8, xmm2/m128, xmm1
CMPPS    | imm8, xmm1, xmm2/m128 | imm8, xmm2/m128, xmm1
CMPSD    | imm8, xmm1, xmm2/m64  | imm8, xmm2/m64, xmm1
CMPSS    | imm8, xmm1, xmm2/m32  | imm8, xmm2/m32, xmm1
ENTER    | 0, imm16              | imm16, 0
ENTER    | 1, imm16              | imm16, 1
```

## 使用汇编知识

- 程序的分段

代码在 .text 段中

- 工具

```
go tool compile -S
```

输出的汇编代码还没有链接，呈现的地址都是偏移量。

```
go tool objdump
```

相当于反汇编你的程序，会丢失很多信息。

当你手写汇编时，与 `go tool compile -S` 和 `objdump` 的是不一样的，不要直接照着这个结果来写！

- 查看 Go 语言的函数调用规约

- 补充知识：伪寄存器

- 查看 built-in 数据结构在传递给函数时的真面目

- 确定应用层代码被翻译为哪些 runtime 函数

例，查看 defer 的执行过程：

例，查看 map 被翻译为的 runtime 函数

例，查看内存是否在堆上分配：

例，查看 Go 语言的编译器是否进行了尾递归优化：

```
go tool compile -S | grep map
```

## 编写汇编代码

- 例1：实现一个简单的 a+b
- 例2：进阶，实现 slice 求和
- 使用汇编来优化，SIMD
- Go 未支持的优化指令编写


## 一些和汇编相关的轮子

c2goasm

SIMD
