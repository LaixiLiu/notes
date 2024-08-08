### 编译选项
在使用gcc编译时，通过`-g`选型可以额外生成可供`GDB`使用的调试信息
- `-g`:一般默认生成`DWARF`格式；
- `-ggdb`:生成适合GDB使用的调试信息，与`-g`相比包含更多的GDB专用信息；
- `-gstabs`:生成`STABS`格式的调试信息，适用于一些旧的编译器；
- `-gdwarf-2`、`-gdwarf-3`、`-gdwarf-4`、`-gdwarf-5`:生成制定版本的`DWARF`格式的调试信息；
### 调试信息格式
几种调试信息的区别：
- DWARF：
	- 现代且功能强大的调试信息格式，支持复杂的数据结构和优化后的代码调试。版本越高，支持的特性越多，但生成的调试信息也可能更大。
- STABS：
	- 较老的调试信息格式，适用于一些旧的调试器。不如 DWARF 强大，支持的特性较少。
- GDB 专用信息：
	- -ggdb 生成的调试信息包含更多的 GDB 专用信息，可能对使用 GDB 调试时更有帮助。
### 切换视图
通过`help layout`可以查看GDB提供的布局选项
```
Change the layout of windows.
Usage: layout prev | next | LAYOUT-NAME

List of layout subcommands:

layout asm -- Apply the "asm" layout.
layout next -- Apply the next TUI layout.
layout prev -- Apply the previous TUI layout.
layout regs -- Apply the TUI register layout.
layout split -- Apply the "split" layout.
layout src -- Apply the "src" layout.
```
- `asm`：开启反汇编窗口；
- `src`：开启源码窗口；
- `next`：切换至下一布局；
- `regs`：显示寄存器窗口；
- `split`：同时显示源码与反汇编代码窗口；