## 基本概念

- 程序的编译阶段

汇编代码在编译的最后阶段生成

链接阶段会对汇编结果做调整(如将函数地址从偏移量转换为逻辑地址)

- 机器码

所有汇编指令都可以转换为 0101 的序列，如：

```
>> mov rax,0x1
mnemonic : mov rax,0x1 => hex : 48c7c001000000
```
hex 是机器码的 16 进制编码。

- 指令集

RISC/CISC，新架构，如 RISCV。

cpu 优化指令集：SSE/SSE2/AVX。。。AVX512

- 寄存器
ah/al
ax/bx
eax/ebx
rax/rbx

## plan9 汇编

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

- 寄存器名差别

```
rax -> AX
rbx -> BX
r8  -> R8
```

因为移动多少个字节是由指令而不是操作数决定的，所以稍微省心一些。

## 使用汇编知识

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

```
go tool compile -S | grep map
```

## 编写汇编代码

- 例1：实现一个简单的 a+b
- 例2：进阶，实现 slice 求和
- 使用汇编来优化，SIMD
- Go 未支持的优化指令编写

c2goasm

SIMD
