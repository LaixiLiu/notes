![Vim键位图](attachments/vi-vim-cheat-sheet-sch1.gif)
### Vim常用命令总结
-    基本移动: hjkl （左， 下， 上， 右）
-    词： w （下一个词）， b （词初）， e （词尾）
-    行： 0 （行初）， ^ （第一个非空格字符）， $ （行尾）
-    屏幕： H （屏幕首行）， M （屏幕中间）， L （屏幕底部）
-    翻页： Ctrl-u （上翻）， Ctrl-d （下翻）
-    文件： gg （文件头）， G （文件尾）
-    行数： :{行数}`<CR>` 或者 {行数}G ({行数}为行数)
-    杂项： % （找到配对，比如括号或者 /* */ 之类的注释对）
-    查找： f{字符}， t{字符}， F{字符}， T{字符}
	-         查找/到 向前/向后 在本行的{字符}
	        , / ; 用于导航匹配
-    搜索: /{正则表达式}, n / N 用于导航匹配

### VIM中执行Shell命令
#### 使用`:!`命令
`:!` 命令用于在 Vim 中执行单个 Shell 命令。执行完命令后，Vim 会显示命令的输出，并提示用户按键返回编辑模式。
```Vim
:!ls
```
![](attachments/Pasted%20image%2020240808142750.png)
#### 使用 `:sh` 命令

`:sh` 命令用于启动一个新的 Shell 会话。在新的 Shell 会话中，用户可以执行多条 Shell 命令。输入 `exit` 或按 `Ctrl-D` 可以返回 Vim。
```Vim
:sh
```
启动 Shell 会话后，可以执行多条命令。退出 Shell 会话后，会返回 Vim。
#### 使用 `system()` 函数
`system()` 函数可以在 Vim 脚本中执行 Shell 命令，并将命令的输出作为字符串返回。可以在 Vim 脚本或映射中使用此函数。
```Vim
:echo system('ls')
```
![](attachments/Pasted%20image%2020240808143007.png)
#### 使用 `read !` 命令
`read !` 命令用于将 Shell 命令的输出插入到当前缓冲区的光标所在行之后。
```Vim
:read !ls
```
![](attachments/Pasted%20image%2020240808143122.png)
#### 使用 `:terminal` 命令
在支持的 Vim 版本（通常是 Vim 8.0 及以上版本）中，可以使用 `:terminal` 命令打开一个嵌入的终端窗口。
```Vim
:terminal
```
打开终端窗口后，可以执行多条 Shell 命令。可以使用 `:q` 命令隐藏编辑区窗口，`<ctr-w>`可用于切换焦点。
