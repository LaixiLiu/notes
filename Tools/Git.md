## git-sparse-checkout
### git clone仓库内指定路径下的文件
在使用`git clone`获取项目源代码时，有时只希望获取项目中某个文件夹下的内容或着某个文件，这个时候我们可以借助`git-sparse-checkout`来实现目的。
### Manual of git-sparse-checkout
#### Description
查看`git-sparse-checkout`的手册，可以看到对它的介绍：
`git-sparse-checkout - Reduce your working tree to a subset of tracked files`
翻译为中文：
`git-sparse-checkout` - 减少你的工作树至追踪文件的子集。

接着看`Description`部分
```
       This command is used to create sparse checkouts, which change the
       working tree from having all tracked files present to only having a
       subset of those files. It can also switch which subset of files are
       present, or undo and go back to having all tracked files present in the
       working copy.

       The subset of files is chosen by providing a list of directories in
       cone mode (the default), or by providing a list of patterns in non-cone
       mode.

       When in a sparse-checkout, other Git commands behave a bit differently.
       For example, switching branches will not update paths outside the
       sparse-checkout directories/patterns, and git commit -a will not record
       paths outside the sparse-checkout directories/patterns as deleted.

       THIS COMMAND IS EXPERIMENTAL. ITS BEHAVIOR, AND THE BEHAVIOR OF OTHER
       COMMANDS IN THE PRESENCE OF SPARSE-CHECKOUTS, WILL LIKELY CHANGE IN THE
       FUTURE.
```
可以看到该命令用于创建稀疏检出，它将工作树由拥有所有跟踪文件变为只有一部分文件。
同时，它也可以切换哪些文件的子集是否出现，或者撤销并返回至所有跟踪文件存在的情况。
#### Synopsis
```
git sparse-checkout (init | list | set | add | reapply | disable | check-rules) [<options>]
```
- `list`: 列出在`.git/info/sparse-checkout`文件中描述的所要保留的目录文件
- `set`: 设置稀疏检出的路径
- `add`: 添加路径至稀疏检出
- `remove`: 从稀疏检出删除路径
- `disable`: 禁用稀疏检出

### Example
目标：拉取`flexoki`项目下的`gtk/gtk-3.0`与`gtk/gtk-4.0`目录
实现：
	1. 克隆仓库但不检出文件：
		`git clone --nocheckout git@github.com:kepano/flexoki.git`
	2. 启用稀疏检出
		`git config core.sparseCheckout true`
	3. 制定要检出的文件夹(要检出的文件应记录在`.git/info/sparse-checkout`文件中)
		`echo "gtk/gkt-?.0" >> .git/info/sparse-checkout`
	4. 检出文件
		`git checkout main`
