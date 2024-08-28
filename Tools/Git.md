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
		`git clone --no-checkout git@github.com:kepano/flexoki.git`
		`cd flexoki`
	2. 启用稀疏检出
		`git config core.sparseCheckout true`
	3. 制定要检出的文件夹(要检出的文件应记录在`.git/info/sparse-checkout`文件中)
		`echo "gtk/gkt-?.0" >> .git/info/sparse-checkout`
	4. 检出文件
		`git checkout main`
```shell
$: tree -L 2
.
└── gtk
    ├── gtk-3.0
    └── gtk-4.0

4 directories, 0 files
```
## git孤立分支
`git checkout --orphan` 命令的作用是创建一个**孤立的分支**（orphan branch），这个分支没有父提交。执行该命令后，你会处于一个全新的分支上，而该分支与当前的提交历史完全脱离。

具体作用如下：

1. **创建新分支**：创建一个新的分支，并且该分支没有任何历史记录。
2. **没有父提交**：新分支是孤立的，也就是没有任何祖先提交记录，因此它从历史上完全脱离。
3. **适用于重新开始历史**：`git checkout --orphan` 常用于清理项目历史、开始新的项目、或者发布不同版本等场景。
### Syntax
```bash
git checkout --orphan <new-branch-name>
```
### Steps
1. 创建孤立分支：
```bash
git checkout --orphan new-branch
```
2. 添加文件并提交：
```bash
git add .
git commit -m "Initial commit on orphan branch"
```
3. 清理旧的文件（可选）：
	孤立分支会继承当前工作目录的文件，如果不需要当前文件，可以清空目录：
```bash
git rm -rf .
```
### 使用场景

- **发布子项目**：当你想发布某个子项目作为独立项目，可以创建孤立分支而不携带整个历史。
- **重置项目历史**：有时你可能需要清理历史并从头开始，`--orphan` 是一个简单的方式。