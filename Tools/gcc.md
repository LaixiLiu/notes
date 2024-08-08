`gcc`对源文件进行处理的命令
```shell
gcc -E hello.c -o hello.i      # 生成预编译文件 hello.i,预处理
gcc -S hello.c       # 生成汇编文件 hello.s,预处理、编译
gcc -c hello.c       # 生成目标文件 hello.o,预处理、编译、汇编
gcc -o hello.out hello.c # 生成可执行文件 hello.out,预处理、编译、汇编、链接
gcc -o -Dname=stuff hello.c -o hello # 编译时制定名为name的符号，其值为stuff
```
开启编译优化可以减小程序体积与运行时间，使用`-On`选项，`n`为代表优化级别的字符。
当`-O`选型启用时，相当于开启了一批默认的优化标志（详情参见gcc手册）。
`-O2` `-O3`等都是在`-O`选型的基础上额外开启了一批优化标志。
几种优化选项的区别：
- `-O0`：沒有优化，`-O`的默认优化级别。注重于减少编译时间并使调试更加容易；
- `-O1`：减少代码大小和执行时间；
- 
```Shell
gcc -O2 test.c -o test
```
