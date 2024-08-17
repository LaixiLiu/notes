C字符串处理函数
#### strtok
`strtok`能够以制定的一系列分隔符分割字符串，并在每次调用时返回指向下一个字符串的指针。
**函数原型**
```c
char *strtok(char *str, const char *delim);
```
**参数**
- **str** -- 要被分解成一组小字符串的字符串。
- **delim** -- 包含分隔符的 C 字符串。
**返回值**
该函数返回被分解的第一个子字符串，如果没有可检索的字符串，则返回一个空指针。
e.g.
```c
#include <stdio.h>
#include <string.h>

int main() {
    char str[] = "ls  -a -l";
    char *token;

    // 以逗号作为分隔符分割字符串
    token = strtok(str, " ");
    while(token != NULL) {
        printf("token=%s\n", token); 
        token = strtok(NULL, " ");
    }

    return 0;
}
```
输出结果：
```
token=ls
token=-a
token=-l
```
对某个字符串调用该函数后，原字符串中第一个分隔符会被修改为`\0`，后续调用时，将`str`传入`NULL`，内存中每个子串后的第一个分隔符会被修改为`\0`
e.g.
```c
#include <stdio.h>
#include <string.h>

int main() {
    char str[] = "ls  -a   -l";
    char *token;
    int len = strlen(str);
    // 以逗号作为分隔符分割字符串
    token = strtok(str, " ");
    while(token != NULL) {
        printf("token: %s\n", token);
        token = strtok(NULL, " ");
    }
    for(int i = 0; i < len; i++) {
        if(str[i] == '\0') {
            printf("\\0");
        }
        printf("%c", str[i]);
    }
    puts("");
    return 0;
}

```
输出结果：
```
token: ls
token: -a
token: -l
ls\0 -a\0  -l
```
`strtok`可以用于分隔命令行参数，结合指针运算判断参数是否为空
e.g.
```c
#include <stdio.h>
#include <string.h>

int main() {
    char str[] = "ls  -a -l";
    char *str_end = str + strlen(str); // 字符串结束位置
    char *token;
    // 以逗号作为分隔符分割字符串
    token = strtok(str, " ");
    while(token != NULL) {
        char *args = token + strlen(token) + 1; // 下一个潜在参数开始位置
        printf("str_end: %p, token: %s %p, args: %p\n", str_end, token, token, args);
        if(args >= str_end) {
            // 没有参数
            break;
        }
        token = strtok(NULL, " ");
    }

    return 0;
}

```
输出结果：
```
str_end: 0x7fffffffd777, token: ls 0x7fffffffd76e, args: 0x7fffffffd771
str_end: 0x7fffffffd777, token: -a 0x7fffffffd772, args: 0x7fffffffd775
str_end: 0x7fffffffd777, token: -l 0x7fffffffd775, args: 0x7fffffffd778
```
从结果可以看出，`token`在每次`strtok`调用后指向下一个字串；而args指向当前`token`终止符`\0`下一个byte。