<!--ts-->
* [git](#git)
   * [初始化](#初始化)
   * [保留换行格式](#保留换行格式)
   * [测试SSH协议](#测试ssh协议)
   * [多账号管理](#多账号管理)
   * [proxy](#proxy)
   * [四种合并方式](#四种合并方式)
   * [批量回退](#批量回退)
   * [对比两个分支](#对比两个分支)
   * [修正提交](#修正提交)
   * [替换历史提交内容](#替换历史提交内容)
   * [删除已加入忽略清单](#删除已加入忽略清单)
   * [bundle打包](#bundle打包)
   * [patch补丁](#patch补丁)
   * [列出合并已修改的文件](#列出合并已修改的文件)
<!--te-->

# git

## 初始化

`git config user.name "jack"`

`git config user.email "jack@domain.com"`

## 保留换行格式

`git config core.autocrlf input`

拉取时保留原换行格式，推送时自动改为lf

## 测试SSH协议

`ssh -T git@github.com`

`ssh -T git@foobar.github.com`

## 多账号管理

配置文件：`~/.ssh/config`

通过在github.com前加上用户名区分：

```
Host xx.github.com
	HostName github.com
	IdentityFile C:\\Users\\bar\\.ssh\\id_rsa
	
Host yy.github.com
	HostName github.com
	IdentityFile C:\\Users\\foo\\.ssh\\id_ed25519
```

使用时将origin中的`github.com`换成`xx.github.com`

## proxy

github使用ssh协议无须proxy

```shell
git config --global http.https://github.com.proxy http://127.0.0.1:1081

# 取消
git config --global --unset http.https://github.com.proxy
```

## 四种合并方式

想改变哪个分支，就要checkout到哪个分支上。

### merge

将feature1上的提交合并到main，保留所有的提交记录，commit id不变

```shell
$ git checkout main
Switched to branch 'main'
$ git merge feature1
```

`–no-ff`会创建新的合并提交，且保留对feature1的引用

### rebase

将feature1上的提交变基到main，rebase会将合入分支feature1上超前的节点在待合入分支main上重新提交一遍

```shell
$ git checkout feature1
Switched to branch 'feature1'
$ git rebase main
$ git checkout main
# fast-forward
$ git merge feature1
```

### squash

压缩提交，就是将多个提交合并成一个

`git rebase -i`时，s就是压缩

merge方式压缩，压缩后需要提交一次

```shell
$ git checkout main
Switched to branch 'main'
$ git merge --squash feature1
Squash commit -- not updating HEAD
Automatic merge went well; stopped before committing as requested
$ git commit -am 'Merged and squashed the feature1 branch changes'
```

### cherry-pick

适合分支内容不同，且不想合并，例如：

dev分支上开发，并提交

切换到beta分支，cherry-pick需要的commit到beta分支

最后，合并beta和main

`git cherry-pick <commit-id>`

## 批量回退

`git revert -n HEAD~2..HEAD`

回退最后两个提交，但不创建新的提交，提交后工作空间在HEAD~2

## 对比两个分支

使用IDE，jetbrains系列可以

## 修正提交

`git commit --amend`

同时修改提交作者：

`git commit --amend --author "jack <jack@domain.com>"`

## 替换历史提交内容

`git filter-branch --tree-filter "find -type f -name '*.py' -exec sed -i 's/oldpassword/fakepass/' {} \;"`

删除备份:

`git update-ref -d refs/original/refs/heads/main`

重置本地仓库：`git reset --hard origin/main`

## 删除已加入忽略清单

`git rm -r --cached .idea`

## bundle打包

`git bundle create app.bundle master`

`git clone -b master app.bundle`

## patch补丁

```
# diff未提交的内容
git diff > commit1.patch

# diff最近一次提交
git diff HEAD^ > commit1.patch

git apply --check commit1.patch
git apply commit1.patch
```

## 列出合并已修改的文件

对于提交，合并到master分支的模式，列出上次合并到现在修改的文件：

`git diff --name-only HEAD^`

显示历史记录：

`git log --pretty=oneline [start-commit-id]`
