
- 配置姓名邮箱

```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

- 秘钥

```bash
ls -al ~/.ssh # 查看秘钥， id_rsa 是私钥，id_rsa.pub 是公钥


## 生成秘钥
ssh-keygen -t rsa -b 4096 -C "you_email@example.com" 

```

- 克隆远程仓库操作

```bash
git clone https//:xxx.git  # 需要授权时需要输入账号密码

git clone git@xxx.git # 需要授权时读取 ~/.ssh 私钥
```

- 本地仓库操作

```bash
# 创建文件夹
mkdir test && cd test

# 初始化 git
git init 

# 移除原有的 origin
git remote rm origin

# 添加远程地址
git remote add origin "your remote repo"

# 更新本地库
git pull

# 首次推送
git push --set-upstream origin master

```

- 分支操作

```bash
# 查看当前所在分支
git branch

#查看所有分支
git branch -a

# 新建分支
git branch {new branch name}

# 切换分支
git checkout {target branch name}

# 新建并切换分支
git checkout -b {new branch name}

# 从远程指定分支新建分支
git checkout -b {new branch name} {origin/branch}
git checkout -b {commit hash}

# 合并分支
git merge {target branch}
git rebase {target branch}

# 删除分支
git branch -d {branch name}

# 强制删除未 commit 的分支
git branch -D {branch name}

```

- 正常流程

```bash
# 查看当前工作区状态
git status

# 添加文件到缓冲区
git add {file name} # 添加单个文件
git add .  # 添加所有文件

# 提交
git commit -m "{message}"

# 推送
git push

# 关联远程分支
git push --set-upstream origin master

```

- 文件对比

```bash
# 工作区和暂存区之间的对比
git diff {file name}

# 暂存区和版本库之间的对比
git diff --cached(--staged) {file name}

# 工作区和版本库之间的对比
git diff master

```

- 查看历史提交记录

```bash
git log
git reflog
```

- 版本回退
```bash
# 回退上一个版本， HEAD 是最新版本， HEAD~是上一个版本
git reset --heard HEAD~

# 回退指定id， git log 或 git reflog 查看版本id
git reset --hard {hash id}

```

- 撤销工作区修改

```bash
# 让文件回到最近一次 conmmit 或者 git add 时的状态
git checkut {file name}
```

- 删除文件
```bash

# 删除工作区的文件
rm {file name}

# 提交删除操作到缓冲区
git rm {file name}
```

- 分支管理策略

[Learn Git Branch](https://learngitbranching.js.org/?locale=zh_CN)


- 存储工作现场

```bash

# 存储工作现场
git stash

# 工作现场列表
git stash list

# 恢复但不删除工作现场
git stash apply

# 恢复并删除存储栈的工作现场
git stash pop

```

- 忽略版本管理文件

创建 **.gitignore** 文件，并添加需要忽略管理的文件或文件夹

- 其他

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