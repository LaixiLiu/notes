### 基础概念
#### 流
对于C程序而言，所有的I/O操作只是从程序移进/移出字节，这种字节流便被称为`流(stream)`。
C标准I/O库支持三种类型的缓冲：完全缓冲（fully buffered）、行缓冲（line buffered）和无缓冲（unbuffered）。
- **完全缓冲**：I/O操作被缓冲到一个内存块（缓冲区）中，直到缓冲区满或者显式刷新缓冲区时才会进行实际的I/O操作（如读取或写入）。这意味着数据不会立即被写入文件或设备，而是先存储在内存缓冲区中，以减少系统调用的次数，提高性能。
- **行缓冲**：数据被缓冲到内存中，直到遇到换行符（`'\n'`）、缓冲区满，或者显式刷新缓冲区时，数据才会被实际写入文件或设备中。这种模式通常用于交互式设备，如终端或控制台。
- **无缓冲**：每次I/O操作都是直接进行的，不使用任何中间缓冲区。也就是说，每次读取或写入操作都会立即调用底层的系统I/O函数，这样可以确保数据立即被处理，但效率可能较低。
	> 使用`fflush`函数可以显示刷新缓冲区。

流分为文本(`text`)流与二进制(`binary`)流。
#### 文件

#### 标准I/O常量

### 文件打开与关闭
#### **fopen**
`fopen`函数定义在`stdio.h`头文件中，用于打开一个文件并进行读取、写入或追加操作，返回值为一个指向该文件的`FILE`类型的指针。
**函数原型**
```c
FILE *fopen(const char *pathname, const char *mode);
```
**参数说明**
- pathname：要打开的文件的路径。
- mode：文件打开模式，表示文件的访问类型。
**文件打开模式**
- "r"：以只读模式打开文件。文件必须存在。
- "w"：以写入模式打开文件。如果文件不存在则创建，若文件已存在则清空文件内容。
- "a"：以追加模式打开文件。如果文件不存在则创建，若文件已存在则从文件末尾开始写入。
- "r+"：以读写模式打开文件。文件必须存在。
- "w+"：以读写模式打开文件。如果文件不存在则创建，若文件已存在则清空文件内容。
- "a+"：以读写模式打开文件。如果文件不存在则创建，若文件已存在则从文件末尾开始读写。
	> `fopen`默认将流视为文本流，在上面的模式字符后面加一个 `"b"`，即可将文件以二进制流模式打开。

**返回值**
- 成功时返回一个指向 FILE 对象的指针。
- 失败时返回 NULL，并设置相应的错误码，可以用 perror 或 strerror 进行错误处理。
e.g.
```c
#include <stdio.h>

int main() {
    FILE *file = fopen("./test.txt", "r");
    if(file == NULL) {
        // 文件打开失败
        perror("test");
    }
    else {
        // 文件打开成功
        printf("文件打开成功\n");
        fclose(file);
    }
}
```
#### **freopen**
`freopen` 是 C 标准库中的一个函数，定义在 `stdio.h` 头文件中。
它的主要功能是将一个新的文件或设备重新分配给已打开的文件流（通常是标准输入、标准输出或标准错误输出）。
这在需要重定向输入输出的情况下非常有用，比如在程序中将输出重定向到文件，或者将文件作为输入源。
**函数原型**
```c
FILE *freopen(const char *pathname, const char *mode, FILE *stream);
```
**参数说明**
- `pathname`：要打开的文件名。
- `mode`：文件打开模式，类似于 `fopen` 的模式字符串，如 `"r"`（读取），`"w"`（写入），`"a"`（追加）等。
- `stream`：要重定向的文件流，通常是 `stdin`、`stdout` 或 `stderr`。
**返回值**
- 成功时返回一个指向 `FILE` 对象的指针。
- 失败时返回 `NULL`，并设置相应的错误码，可以用 `perror` 或 `strerror` 进行错误处理。
**使用场景**
1. **重定向标准输入**：将标准输入重定向为从文件读取。
2. **重定向标准输出**：将标准输出重定向到文件。
3. **重定向标准错误输出**：将标准错误输出重定向到文件。
e.g.重定向`stdin`
```c
#include <stdio.h>

int main() {
    freopen("input.txt", "r", stdin);
    char buffer[100];
    while (fgets(buffer, sizeof(buffer), stdin) != NULL) {
        printf("%s", buffer);
    }
    return 0;
}
```
e.g.重定向`stderr`
```c
#include <stdio.h>

int main() {
    freopen("errors.txt", "w", stderr);
    fprintf(stderr, "This error message will be written to errors.txt\n");
    return 0;
}
```
#### **fclose**
`fclose`用于关闭流。
对于输出流，`fclose`会在文件关闭之前刷新缓冲区。如果执行成功，则返回零值，否则返回`EOF`。
**函数原型**
```c
int fclose(FILE *stream);
```
**参数说明**
- `stream`：指向要关闭的文件的指针，该文件指针之前应该通过 `fopen`、`freopen` 或类似函数获得。
**返回值**
- 成功时返回 `0`。
- 失败时返回 `EOF`，并设置相应的错误码，可以用 `perror` 或 `strerror` 进行错误处理。
 
**可能导致`fclose`失败的情况**
- 缓冲区刷新失败
    - 在关闭文件时，`fclose` 会尝试刷新所有与该文件相关的缓冲区内容。如果由于某种原因无法将缓冲区内容写入文件，`fclose` 会失败。例如，磁盘已满或发生了写入错误。
- 文件系统错误
    - 文件系统的错误，例如文件系统损坏、网络文件系统连接丢失（如果文件位于网络文件系统上），也可能导致 `fclose` 失败。
- 非法文件指针
    - 如果传递给 `fclose` 的文件指针不是一个有效的打开文件指针，或者是一个已经关闭的文件指针，`fclose` 的行为是未定义的。这通常会导致崩溃，但有时也可能导致 `fclose` 失败。
- 系统资源耗尽
    - 系统资源耗尽，如文件描述符耗尽或内存不足，也可能导致 `fclose` 失败。

### 字符I/O
#### 读取
当一个流被打开后，它可以用于输入或输出。
它最简单的形式就是字符I/O，可通过`getchar`系列函数实现。其函数原型如下：
```c
int fgetc(FILE *stream);
int getc(FILE *stream);
int getchar(void);
```
需要将要操作的流作为参数传递给`fgetc`与`getc`；`getchar`始终从`stdin`读取。
每个函数从流中读取下一个字符，并将其作为返回值返回，如果流中不存在更多的字符，则返回`EOF`。
e.g.
```c
#include <stdio.h>
#include <stdlib.h>
int main() {
    FILE *file = fopen("./test.txt", "r");
    if (file == NULL) {
        perror("test.txt");
        exit(1);
    }
    int c;
    while((c = getc(file)) != EOF) {
        putchar(c);
    }
    if(fclose(file) != 0) {
        perror("fclose");
        exit(1);
    }
    return 0;
}
```
#### 写入
**函数原型**
```c
int fputc(int character, FILE *stream);
int putc(int character, FILE *stream);
int putchar(int character);
```
`character`为要打印的字符，函数会将这个`int`类型转换为对应的无符号整形值。
如果写入失败(如向一个已关闭的流写入)，则返回`EOF`。
e.g.
```c
#include <stdio.h>
#include <stdlib.h>
int main() {
    FILE *file = fopen("./test.txt", "w");
    if (file == NULL) {
        perror("test.txt");
        exit(1);
    }
    char c = '0';
    for(; c < '9' && fputc(c, file); c++) ;
    if(fclose(file) != 0) {
        perror("fclose");
        exit(1);
    }
    return 0;
}
```
#### 撤销字符I/O
在读取字符流时，可能需要查看下一个字符来决定如何处理当前输入，但这个字符可能不应该消耗掉。在这种情况下，可以使用 `ungetc` 将字符放回流中。
**函数原型**
```c
int ungetc(int c, FILE *stream);
```
`ungetc` 只能在输入流上使用。试图在输出流上使用 `ungetc` 会导致未定义行为。