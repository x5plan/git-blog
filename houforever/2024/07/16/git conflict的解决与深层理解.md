# git conflict的解决与深层理解

### 产生原因

是因为提交的两份代码修改了同一行code

*一般来说，conflict只会发生在merge/rebase的时候*,必须先解决才能继续merge/rebase

### 解决方法

把冲突部分基于新的main上修改

即:基于先提交pr的人的代码基础上修改

#### git fetch

本地把main分支合并到我的分支上去

#### git merge origin/main

相当于直接把你从main的某个历史节点拽到现在最新的main上去

#### git rebase

相当于把自己的分支拆下来装到最新的main上



### 概念辨析

1. main：你本地的main
2. origin/main:你下载到本机远程到main，但是没有和本地的main同步



### git pull详解

```
git pull = git fetch + git merge/rebase
```

- git fetch是将远程库的最新内容拉到本地库，用户在检查了以后决定是否合并到工作区中。
- git merge是将本地的两个分支合并，如果在分支A中执行git merge B，那就是将分支B中的代码合并到分支A中。
- git pull 则是将远程主机的最新内容拉去到本地库后直接合并到工作区中，即：git pull = git fetch + git merge，这样可能会产生冲突，需要手动解决。
- git rebase是将提交到某一分支上的所有修改都移至另一分支上。即如果在B分支上使用git rebase A就是将B分支上的修改都变基（移到）A分支上
  详见：https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA
  git pull --rebase则是将远程主机的最新内容拉去到本地库后直接变基到工作区中，即：git pull --rebase = git fetch + git rebase，可能会产生冲突手动解决。

//还得理解一下,有点痴呆.

[git pull 与 git pull --rebase_git pull rebase-CSDN博客](https://blog.csdn.net/mantou_riji/article/details/126762563?ops_request_misc=%7B%22request%5Fid%22%3A%22172113720416800227464931%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=172113720416800227464931&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-4-126762563-null-null.142^v100^pc_search_result_base1&utm_term=git pull&spm=1018.2226.3001.4449)

好东西,回头看