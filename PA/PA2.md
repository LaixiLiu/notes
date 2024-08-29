## RTFSC(2)
---
### nemu中指令执行的过程
`/src/cpu/cpu-exec.c`中定义了`execute()`函数，该函数通过for循环模拟cpu执行指令的过程。在for循环中，调用`exec_once()`函数，该函数接受一个结构体`Decode *s`，和当前的pc值，
```c
typedef struct Decode {
  vaddr_t pc;
  vaddr_t snpc; // static next pc
  vaddr_t dnpc; // dynamic next pc
  ISADecodeInfo isa;
  IFDEF(CONFIG_ITRACE, char logbuf[128]);
} Decode;
```
`Decode`结构体用于保存当前pc的值、下一条pc的值，以及用于存储指令信息的结构体`ISADecodeInfo isa`。

`exec_once`函数首先初始化`s`的pc与snpc值(pc表示当前指令地址，snpc表示下一条指令的地址，dnpc表示在分支、条件跳转后的指令地址)，然后调用`isa_exec_once()`函数进行取指、译码与执行。
**取指**
`isa_exec_once()`函数首先调用`inst_fetch()`函数(`inst_fetch()`函数通过接受pc值，从内存中读取pc对应地址的指令。)进行取指令，并将指令值保存至`s->isa.inst.val`中，然后调用`decode_exec()`并返回。
`decode_exec()`函数负责进行指令的译码与执行。
**译码**
`decode_exec()`函数首先初始化如下变量：
1. `int rd`用于保存指令中的目的寄存器编码；
2. `word_t src1, src2, imm`分别用于保存指令中的源寄存器与立即数
3. `s->dnpc = s->snpc`初始化dnpc为当前指令地址

`decode_exec()`利用宏展开与模式字符串进行指令的译码与执行
```c
INSTPAT_START();
INSTPAT("??????? ????? ????? ??? ????? 00101 11", auipc  , U, R(rd) = s->pc + imm);
// ...
INSTPAT_END();
```
`INSTPAT_START()`宏与`INSTPAT_END()`宏相对应，`INSTPAT_END`宏展开后会创建一个`__instpat_end_`标签(`lable`)，而`INSTPAT_START()`宏通过`const void ** __instpat_end = &&concat(__instpat_end_, name)`获取`__instpat_end_`标签的地址，用于在模式匹配成功并执行后跳转至`__instpat_end_`标签，可以将其功能理解为`switch-case`中的`break`。

将`decode_exec()`中的宏展开并简单整理后可以得到下面的代码：
```c
{ const void ** __instpat_end = &&__instpat_end_;
do {
  uint64_t key, mask, shift;
  pattern_decode("??????? ????? ????? ??? ????? 00101 11", 38, &key, &mask, &shift);
  if ((((uint64_t)s->isa.inst.val >> shift) & mask) == key) {
    {
      decode_operand(s, &rd, &src1, &src2, &imm, TYPE_U);
      R(rd) = s->pc + imm;
    }
    goto *(__instpat_end);
  }
} while (0);
// ...
__instpat_end_: ; }
```
`pattern_decode()`函数用于将模式字符串转换为`key``mask`与`shift`三个整形变量。
`key`用于保存模式字符串中的0与1；
`mask`表示`key`的掩码，用于标记模式字符串中的`?`；
`shift`用于表示`opcode`操作码距离最低位的位数。

`pattern_decode()`完成提取模式字符串后，将当前指令与上面提到的三个变量进行位运算，判断是否匹配。

若不匹配，则进行匹配下一条模式字符串；若匹配，则调用`decode_operand()`函数提取指令中的寄存器、立即数信息。
**执行**
提取完成之后，将`INSTPAT()`函数中的最后一个参数展开执行；执行完成之后，跳转至表示结束的标签，对x0寄存器重新置0,并退出函数。
**更新PC**
接着，函数一路返回，控制权回到了`exec_once()`函数，它将cpu的pc值更新为`dnpc`
### 实现指令并运行C程序
在实现指令的过程中，由于RISC-V32会对指令中的立即数进行符号扩展，并用指令的最高位作为符号位，所以需要在译码阶段对立即数进行符号扩展。
nemu在`/include/macro.h中定义了如下三个宏：
```c
#define BITMASK(bits) ((1ull << (bits)) - 1)
#define BITS(x, hi, lo) (((x) >> (lo)) & BITMASK((hi) - (lo) + 1)) // similar to x[hi:lo] in verilog
#define SEXT(x, len) ({ struct { int64_t n : len; } __x = { .n = x }; (uint64_t)__x.n; })
```
`BITS()`用于取出指令中`[hi, lo]`区间中的比特位；
`SEXT()`用于对`x`进行符号扩展，`len`为`x`的位数(符号位+数值位)。
#### 符号扩展
`SEXT()`宏其实利用到了c语言中`bit field`的概念：
在定义`struct`或`union`时，对于`int`,`signed int`或`unsigned int`，我们可以定义成员所要使用的比特位位数
```c
struct
{
	unsigned int a: 12, b: 10, c: 10;
} test;
```
上面的代码中定义了`test`结构体，与其成员a,b,c,；通过`sizeof(test)`可知`test`占用4字节；其成员a,b,c分别占12,10,10位。

宏 SEXT 接受两个参数：x 和 len。x 是需要进行符号扩展的整数，而 len 是该整数的位长度。宏的实现使用了一种非常规的方式，即通过定义一个匿名结构体来实现符号扩展。
1. 定义一个匿名结构体，该结构体包含一个位域成员 n，其长度为 len 位。
2. 使用初始化器将结构体实例 `__x` 的成员 n 赋值为 x。
3. 将结构体成员 n 转换为 uint64_t 类型并返回。
这种方法利用了位域的特性：当一个较小的有符号整数被赋值给一个较大的有符号整数时，编译器会自动进行符号扩展。因此，通过定义一个位域长度为 len 的结构体成员并赋值给它，编译器会自动进行符号扩展，然后将结果转换为 uint64_t 类型。

对于`bitfield`的自动符号扩展，我们可以用下面的代码进行验证：
```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

typedef struct {
  int32_t a: 12;
} sign;

typedef struct {
  uint32_t a: 12;
} unsign;

int main() {
  sign *x = malloc(sizeof(sign));
  x->a = 0xf10;
  uint64_t result = (uint64_t)x->a;
  printf("x->a = %08x\n", x->a);
  printf("result = %lx\n", result);
  unsign *y = malloc(sizeof(unsign));
  y->a = 0xf10;
  printf("y->a = %08x\n", y->a);
  printf("result = %lx\n", (uint64_t)y->a);
  free(x);
  free(y);
  return 0;
}
```
```
output:
x->a = ffffff10
result = ffffffffffffff10
y->a = 00000f10
result = f10
```
