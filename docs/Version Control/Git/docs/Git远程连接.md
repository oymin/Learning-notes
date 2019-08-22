# Git远程连接

## remote 命令

### 关联远程仓库

**语法：**`git remote add [仓库名称] [仓库连接]`

```bash
git remote add origin https://github.com/asd/WorkSpace.git
```

查看远程仓库：

```bash
git remote -v
```

```bash
origin  https://github.com/asd/WorkSpace.git (fetch)
origin  https://github.com/asd/WorkSpace.git (push)
```

删除远程仓库：

```bash
git remote remove [仓库名称]
```

### 拉取远程仓库内容

**语法：**`git pull [仓库名] [分支名]`

```bash
git pull origin master
```

### 推送内容到远程仓库

**语法：**`git push [仓库名] [分支名]`

```bash
git push -u origin master
```

由于远程库是空的，我们第一次推送 `master` 分支时，加上了 `-u` 参数，Git 不但会把本地的 `master` 分支内容推送的远程新的 `master` 分支，还会把本地的 `master` 分支和远程的 `master` 分支关联起来，在以后的推送或者拉取时就可以简化命令。

## SSH 警告

当你第一次使用 Git 的 clone 或者 push 命令连接 GitHub 时，会得到一个警告：

```bash
The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
RSA key fingerprint is xx.xx.xx.xx.xx.
Are you sure you want to continue connecting (yes/no)?
```

这是因为 Git 使用 SSH 连接，而 SSH 连接在第一次验证 GitHub 服务器的 Key 时，需要你确认 GitHub 的 Key 的指纹信息是否真的来自 GitHub 的服务器，输入 `yes` 回车即可。

Git 会输出一个警告，告诉你已经把 GitHub 的 Key 添加到本机的一个信任列表里了：

```bash
Warning: Permanently added 'github.com' (RSA) to the list of known hosts.
```

这个警告只会出现一次，后面的操作就不会有任何警告了。
