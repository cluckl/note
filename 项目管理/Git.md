# git

## 工作流程

Git本地有三个工作区域：

* 工作目录（Working Directory）
* 暂存区(Stage/Index)：所有分支共享，在一个分支上操作之后，如果还没有将修改提交到分支上，此时进行切换分支，那么另一个分支上也能看到新的修改。使用 `git stash` 将当前分支的修改储藏起来，`git stash pop`恢复当前分支的修改。
* 资源库(Repository)

![image-20211103123903863](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20211103124605.png)

## 文件状态

- Untracked: 未跟踪，此文件不参与版本控制
  - 执行`git add <file_name> ` 将文件加入版本控制中，并转为unmodified转态
- Unmodified: 文件已入库且未修改
  - 当文件被修改, 转为modified转态
  - 执行`git rm <file_name>`将文件移出版本库, 并转为untracked状态
- Modified: 文件已修改
  - 执行`git add <file_name> `，并转为staged状态
  - 执行`git checkout`丢弃修改, 并转为unmodified状态
- Staged: 暂存状态
  - 执行`git commit`将修改同步到库中, 这时库中的文件和本地文件变为一致, 并转为unmodified状态
  - 执行`git reset HEAD` 取消已缓存的内容, 并转为modified状态

![Illustration of the process of tracking, modifying, and committing changes to a Git project](http://git-scm.com/figures/18333fig0201-tn.png)

## 分支管理

Git的每个分支存在一个指针用于指向当前节点，分支的操作实际上是对该指针的操作。

![GIT Branching and Merging](https://i2.wp.com/digitalvarys.com/wp-content/uploads/2019/06/image-7.png?fit=640%2C311&ssl=1)

### 命令

```shell

# 列出所有本地分支
git branch

# 列出所有远程分支
git branch -r

# 新建分支
git branch <branch_name>

# 切换到一个已经存在的分支
git checkout <branch_name>
git switch <branch_name>

# 新建一个分支并切换到该分支
git checkout -b <branch_name> 
git switch -c <branch_name>

# 合并指定分支到当前分支
git merge <branch_name> 

# 仅当分支被合并时删除分支
git branch -d <branch_name> 

# 强行删除分支而不检查分支是否被合并
git branch -D <branch_name> 

# 删除远程分支
git push origin --delete <branch_name> 
```

