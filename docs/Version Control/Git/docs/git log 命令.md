# git log 命令

查看 git 日志

## git log 优化显示

### 查看指定内容

```bash
git long --oneline add
```

结果：

```bash
021b963 add java file
718f760 add new file
```

### 以自定义方式打印 Git 提交日志

```bash
git log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
```

### 自定义 log 日志颜色

还可以自定义颜色输出：

```bash
git log --graph --date=format:'%Y-%m-%d %H:%M:%S' --pretty=format:'%C(yellow)%h%Creset %ad %C(cyan)%s%Creset %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

## git 的预定义占位符

|参数|说明|
|---|---|
| %H |	commit hash |
| %h |	commit的短hash |
| %T |	tree hash |
| %t |	tree的短hash |
| %P |	parent hashes |
| %p |	parent的短hashes |
| %an |	作者名字 |
| %aN |	mailmap中对应的作者名字 (.mailmap对应，详情参照git-shortlog(1)或者git-blame(1)) |
| %ae |	作者邮箱 |
| %aE |	作者邮箱 (.mailmap对应，详情参照git-shortlog(1)或者git-blame(1)) |
| %ad |	日期 (–date= 制定的格式) |
| %aD |	日期, RFC2822格式 |
| %ar |	日期, 相对格式(1 day ago) |
| %at |	日期, UNIX timestamp |
| %ai |	日期, ISO 8601 格式 |
| %cn |	提交者名字 |
| %cN |	提交者名字 (.mailmap对应，详情参照git-shortlog(1)或者git-blame(1)) |
| %ce |	提交者 email |
| %cE |	提交者 email (.mailmap对应，详情参照git-shortlog(1)或者git-blame(1)) |
| %cd |	提交日期 (–date= 制定的格式) |
| %cD |	提交日期, RFC2822格式 |
| %cr |	提交日期, 相对格式(1 day ago) |
| %ct |	提交日期, UNIX timestamp |
| %ci |	提交日期, ISO 8601 格式 |
| %d |	ref名称 |
| %e |	encoding |
| %s |	commit信息标题 |
| %f |	过滤commit信息的标题使之可以作为文件名 |
| %b |	commit信息内容 |
| %N |	commit notes |
| %gD |	reflog selector, e.g., refs/stash@{1} |
| %gd |	shortened reflog selector, e.g., stash@{1} |
| %gs |	reflog subject |
| %Cred |	切换到红色 |
| %Cgreen |	切换到绿色 |
| %Cblue |	切换到蓝色 |
| %Creset |	重设颜色 |
| %C(…) |	制定颜色, as described in color.branch.* config option |
| %m |	left, right or boundary mark |
| %n |	换行 |
| %% |	a raw % |
| %x00 |	print a byte from a hex code |
| %w([<w>[,<i1>[,<i2>]]]) |	switch line wrapping, like the -w option of git-shortlog(1). |

## git 其他功能多颜色配置

git 默认的输出是单一颜色的，不仅不够美观，也不容易阅读。实际上，git 本身就支持用多种颜色来显示其输出的信息，只需在命令行中运行以下命令来修改 git 的设置，即可开启多颜色输出：

```bash
git config --global color.status auto
```

```bash
git config --global color.diff auto
```

```bash
git config --global color.branch auto
```

```bash
git config --global color.interactive auto
```

执行以上命令后，git 的 `status`, `diff` 和 `branch` 等诸命令的输出就都是带有颜色的了。

## git 支持的颜色

| 颜色 | 说明 |
|:----:|:----:|
| normal  | 正常   |
| black   | 黑色   |
| red     | 红色   |
| green   | 绿色   |
| yellow  | 黄色   |
| blue    | 蓝色   |
| magenta | 洋红色 |
| cyan    | 青色   |
| white   | 白色   |

颜色可以结合一下属性之一：

| 颜色 | 说明 |
|:----:|:----:|
| bold   | 加粗 |
| dim    | dim |
| ul     | ul  |
| blink  | 闪烁 |
| reverse| 反转 |
