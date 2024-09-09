### Make工作方式
GNU make工作时的执行步骤如下：
1. 读入所有的Makefile。
2. 读入被include的其它Makefile。
3. 初始化文件中的变量。
4. 推导隐式规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。

1-5步为第一个阶段，6-7为第二个阶段。第一个阶段中，如果定义的变量被使用了，那么，make会把其展开在使用的位置。但make并不会完全马上展开，make使用的是拖延战术，如果变量出现在依赖关系的规则中，那么仅当这条依赖被决定要使用了，变量才会在其内部展开。
### Makefile语法
```Makefile
targets: prerequisites
	command
	command
	command
```
- `targets`：(目标)需要构建的目标文件文件名，用空格分隔；
- `commands`：(命令)用于构建目标的一系列命令；以制表符开头；
- `prerequisites`：(先决条件)文件名，以空格分隔。只有在这些文件存在时，才会执行`command`。也可以称之为依赖项(dependencies).

一个简单的示例：
```Makefile
hello:
	echo "Hello, world.\n"
```
当输入`make hello`时，执行`echo "Hello, world.\n"命令。

**make自动推导**
```Makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)
main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit $(objects)
```
当`object.o`被设置在目标中时，make会将`object.c`自动添加到依赖列表，并自动推导出`cc -c object.o`命令。
因此，上述Makefile可以简写为下面的形式：
```Makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
    rm edit $(objects)
```

**包含其他Makefile**
可以使用`include`将其他Makefile包含进来，类似C语言中的`#include`，被包含的Makefile会被原样展开：
```Makefile
include a.mk b.mk c.mk
```
make命令开始后，会首先寻找`include`的文件；如果没有找到对应文件，则会生成警告信息，直到其他所有文件完成载入，会再次尝试这些没有找到的文件，如果失败，则make会退出。

不想让make因为无法读取的文件而终止，可以使用`-include`：
```Makefile
-include <filenames> ...
```
无论`include`过程中出现什么错误，都不报错而是继续执行。
### 变量
Makefile中的变量只能是字符串，有点类似C语言中的宏，可以使用`:=`或`=`声明:
```Makefile
CC = gcc
```
如果使用变量，则可以通过`$(CC)`或`${CC}`两种方式引用变量。
#### 通配符
make支持通配符`~``*`与`?`，其含义与shell中的类似。
- `~`用于表示当前用户的家目录，在Linux下即为环境变量`$HOME`的值；
- `*`用于匹配零个或多个任意字符；
- `?`用于匹配单个任意字符；

可使用`\*`来进行转义，表示真实的`*`字符。
```Makefile
clean:
	rm -f *.o
```
### 目标

**all目标**
如果要编译多个目标，则可以设定一个名为`all`目标
```Makefile
all: one two three
.PHONY : all

one:
	touch one
two:
	touch two
three:
	touch three
```

