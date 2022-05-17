<!--ts-->
* [git](#git)
   * [初始化](#初始化)
   * [保留换行格式](#保留换行格式)
   * [测试SSH协议](#测试ssh协议)
   * [多账号管理](#多账号管理)
   * [proxy](#proxy)
   * [批量回退](#批量回退)
   * [对比两个分支](#对比两个分支)
   * [修正提交](#修正提交)
   * [替换历史提交内容](#替换历史提交内容)
   * [删除已加入忽略清单](#删除已加入忽略清单)
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