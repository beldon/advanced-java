
```bash
## 创建分支，远程分支和本地分支可以不一样
git checkout -b {new branch name} # 从当前分支创建新的分支（-b 若不存在时创建）
git checkout -b {new branch name} {target branch name} # 从指定分支创建分支
git checkout -b {new branch name} {target commit hash} # 从指定提交创建分支

git checkout -m {old branch name} {new branch name} # 修改分支

git merge {target branch name}
git rebase {target branch name}

git cherry-pick {target commit hash} # 把指定的 commit 从新提交到当前分支

git revert {commit hash} # 回退某个提交
git revert {revert commit hash} # 取消回退

git commit --amend -m "" # 提交合并到最新的提交记录中

git reset {commit hash} # 回退到某个提交，回保留工作区的内容

git stash

#git fix up & squash  用法

```



- [Learn Git Branch](https://learngitbranching.js.org/?locale=zh_CN)