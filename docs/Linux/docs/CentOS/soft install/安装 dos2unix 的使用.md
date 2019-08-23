# dos2unix 的使用

由于在 DOS（windows系统）下，文本文件的换行符为 CRLF，而在 Linux 下换行符为 LF，使用 git 进行代码管理时，git 会自动进行 CRLF 和 LF 之间的转换，这个我们不用操心。而有时候，我们需要将 windows 下的文件上传到 linux 上，例如 shell 脚本，执行的时候有时会出现奇怪的问题，这时候，就需要安装 dos2unix 软件，centos 下：

```bash
yum install -y dos2unix
```

安装完成后，对文件进行转换

```bash
dos2unix abc.sh
```

现在执行就不会出问题了
