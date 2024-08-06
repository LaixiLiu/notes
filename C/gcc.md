`gcc`对源文件进行处理的命令
```shell
gcc -E hello.c -o hello.i      # 生成预编译文件 hello.i,预处理
gcc -S hello.c       # 生成汇编文件 hello.s,预处理、编译
gcc -c hello.c       # 生成目标文件 hello.o,预处理、编译、汇编
gcc -o hello.out hello.c # 生成可执行文件 hello.out,预处理、编译、汇编、链接
gcc -o -Dname=stuff hello.c -o hello # 编译时制定名为name的符号，其值为stuff
```