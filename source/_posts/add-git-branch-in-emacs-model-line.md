---
title: Emacs 的 Mode Line 上添加 Git Branch 的名字.
date: 2019-05-27 20:33:30
tags: [emacs, elisp]
---

# 在 Emacs 的 mode line 上显示当前的 git branch.

打算对自己的 Emacs 进行简单的定制, 就从比较简单的地方开始. 其实最正确地做法是用 doom-line.
这篇博客以一个初学者(懂一点基础的 elisp, 有一点编程基础)的角度写一下自己的研究过程.

第一步就是先分解一下这个需求, 将其拆分几个小问题:
1. 如何在 Emacs 里获取当前的分支名?
2. 如何定制 Mode Line 里的信息.
3. 将分支名加入 mode line 中.

## 从 Emacs 里获取当前的 Branch 信息.

获取当前 git 的分支信息可以利用命令 `git rev-parse --abbrev-ref HEAD` 获取当前分支的名称. 
所以这个问题其实可以转化成如何使用 elisp 运行 shell 命令并获取结果. 通过简单地 Google, 可以发现
elisp 提供了 `shell-command` 和 `shell-command-to-string` 两个函数来执行外部的 shell 命令.

所以可以通过以下的代码来获取当前的 branch 信息. 
``` elisp
(shell-command-to-string "git rev-parse --abbrev-ref HEAD")
```

## 如何定制 mode line 里的信息.

阅读关于官方两篇关于 mode line 的文档: [The Mode Line](https://www.gnu.org/software/emacs/manual/html_node/emacs/Mode-Line.html)
和 [Mode Line Format](https://www.gnu.org/software/emacs/manual/html_node/elisp/Mode-Line-Format.html#Mode-Line-Format)
可以比较全面地了解关于 mode line. 也可以知道如何定制 mode line.

通过第二篇文档, 可以得知 mode line 的显示在大部分时候由 `mode-line-format` 决定. 可以通过改变
`mode-line-format` 的值来改变 mode line 的显示内容. 

不过这个 mode-line-format 结构体比较复杂, 需要好好分析一下.


