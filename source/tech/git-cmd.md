# GIT常用命令

## 常用

添加修改
```
git add FILE
```

提交修改
```
git commit -m "MESAGE"
```

拉取更新
```
git pull origin BRANCH
```

推送更新
```
git push origin BRANCH
```

## 分支

新建分支
```
git branch BRANCH
```

新建空白分支
```
git checkout --orphan BRANCH
```

切换分支
```
git checkout BRANCH
```

新建并切换分支
```
git checkout -b BRANCH
```

重命名分支
```
git branch -m OLDBRANCH NEWBRANCH
```

删除分支
```
git branch -d BRANCH
git branch -D BRANCH
```

合并分支
```
git merge BRANCH    # 将branch合并到当前分支
```

拉取远程新分支
```
git checkout -b LOCALBRANCH origin/BRANCH
git checkout REMOTEBRANCH -b LOCALBRANCH
```

合并commit
```
git cherry-pick COMMIT      # 将一个分支的特定COMMIT合并到当前分支
```


删除远程分支
```
git push origin :REMOTEBRANCH               # 适合所有版本
git push origin --delete REMOTEBRANCH       # 适合v1.7.0之后版本
```

清理远程分支
```
git remote prune origin
```

## 历史

查看修改历史
```
git log
```

查看修改内容
```
git show COMMIT
```

查看文件的特定版本内容
```
git show COMMIT:FILE
```


查看当前commit
```
git rev-parse --short HEAD
```

查看修改文件历史
```
git log --name-only
```

## 回滚

撤销修改
```
git checkout FILE       # 未添加到暂存区
git reset HEAD FILE     # 已添加到暂存区
```

取消刚刚的合并
```
git reset --merge
```

回滚到某个commit
```
git reset --hard COMMITID       # 新增内容全部丢失
git reset --soft COMMITID       # 新增内容处于已添加到暂存区状态
git reset --mixed COMMITID      # 新增内容处于未添加到暂存区状态
```

## 重写历史

修改第1次commit
```
git rebase -i --root
```

修改最新commit的信息
```
git commit --amend
```

修改最新commit的作者信息
```
git commit --amend --author="NEWAUTHOR <NEWEMAIL@address.com>"
```

批量修改commit的作者信息
```
git filter-branch --commit-filter '
    if [ "$GIT_AUTHOR_NAME" = "old name" ];
    then
            GIT_AUTHOR_NAME="new name";
            git commit-tree "$@";
    else
            git commit-tree "$@";
    fi' HEAD
```

修改历史时保留merge commit
```
git rebase --rebase-merges master
```


## 对比

查看本地修改
```
git diff            # 针对未添加到暂存区的情况
git diff --cached   # 针对已添加到暂存区的情况
```

对比提交差异
```
git diff COMMIT1 COMMIT2    # 针对已添加到暂存区的情况
```

对比分支差异
```
git diff BRANCH1 BRANCH2    # 显示出所有有差异的文件的详细差异
```

## 文件

删除文件
```
git rm FILE
```
重命令文件
```
git mv FILEA FILEB
```
删除未跟踪的文件和目录
```
git clean -fd
```

## 归档
导出代码
```
git archive -o output.tar master
```
导出新修改的文件的未修改版本
```
git archive -o update.tar HEAD $(git diff --name-only HEAD)
```
提取本地修改的文件并打包
```
tar -cvf update.tar $(git diff --name-only HEAD)
```

## 配置
查看git配置
```
git config --system --list      # 查看系统配置
git config --global --list      # 查看当前用户配置
git config --local --list       # 查看当前仓库配置
```

设置用户名和邮箱
```
git config --global user.name USERNAME
git config --global user.email EMAIL
git config --local  user.name USERNAME
git config --local  user.email EMAIL
```

清除用户名和邮箱设置
```
git config --global --unset user.name 
git config --global --unset user.email
```

## 子模块
添加子模块
```
git submodule add URL PATH
```
克隆所有子模块
```
git clone --recursive URL
```
更新所有子模块
```
git submodule foreach git pull
```
初始化子模块
```
git submodule init
```
去初始化子模块
```
git submodule deinit
```
更新子模块
```
git submodule update
```
初始化(克隆)并更新子模块
```
git submodule update --init --recursive
```
将submodule新的URL更新到文件.git/config
```
git submodule sync
```

## subtree
将仓库old-repo中关于目录elake的commit抽出为新的分支new-branch
```
cd old-repo
git subtree split -P elake -b new-branch
```

新建仓库new-repo，并从old-repo的分支new-branch中拉取内容
```
mkdir new-repo
cd new-repo
git init
git pull ../old-repo new-branch
```

## 远程地址
添加关联的远程仓库URL
```
git remote add origin URL
```
修改关联的远程仓库URL
```
git remote set-url origin URL
```
删除关联的远程仓库
```
git remote rm origin
```

## SSH keys配置
生成密钥（一组公钥和私钥对）
```
ssh-keygen -t rsa -C "EMAIL"
```

添加密钥到ssh-agent的高速缓存：默认只读取id_rsa，为了让SSH识别新的私钥，需将其手动添加到ssh-agent中
```
ssh-agent bash
ssh-add ~/.ssh/XXX_id_rsa
```

## 别名配置

### 通过命令行设置
```
git config --global alias.pl pull
git config --global alias.ps push
git config --global alias.co checkout
git config --global alias.st status
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.rs restore
git config --global alias.df diff
git config --global alias.dft difftool
git config --global alias.dfs 'diff --staged'
git config --global alias.dfts 'difftool --staged'
git config --global alias.mg merge
git config --global alias.mgt mergetool
git config --global alias.lg 'log --abbrev-commit --name-status'
git config --global alias.gp 'log --graph --pretty=format:"%C(auto)%h %C(green)[%an] %C(magenta)(%ad) %Creset%C(auto)%d %s" --date=format:"%Y-%m-%d %H:%M:%S"'
git config --global alias.last 'log -1 HEAD'
git config --global alias.rb 'rebase -i'
git config --global alias.cp 'cherry-pick'
```

### 通过.gitconfig文件设置
```
pl = pull
ps = push
co = checkout
st = status
ci = commit
br = branch
rs = restore
df = diff
dft = difftool
dfs = diff --staged
dfts = difftool --staged
mg = merge
mgt = mergetool
lg = log --abbrev-commit --name-status
gp = log --graph --pretty=format:\"%C(auto)%h %C(green)[%an] %C(magenta)(%ad) %Creset%C(auto)%d %s\" --date=format:\"%Y-%m-%d %H:%M:%S\"
last = log -1 HEAD
rb = rebase -i
cp = cherry-pick
```
