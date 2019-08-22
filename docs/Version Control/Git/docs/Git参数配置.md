# Git参数配置

## git config 命令

### 查看配置列表

```bash
git config --list
```

### 设置全局用户名和邮箱

```bash
#设置提交仓库时的用户名信息
git config --global user.name "我的名字"

#设置提交仓库时的邮箱信息
git config --global user.email "我的邮箱"
```

### Git 换行符设置

提交时转换为 LF，检出时转换为 CRLF，默认设置不用修改

```bash
git config --global core.autocrlf true
```

允许提交包含混合换行符的文件

```bash
git config --global core.safecrlf false
```

## 设置别名

将 `commit` 命令设置别名 `ci`

```bash
git  config --global alias.ci commit
```

## --date 参数

--date 支持以下参数：

- relative
- local
- default
- iso
- iso-strict
- rfc
- short
- raw

支持自定义格式：

```bash
--date=format:'%Y-%m-%d %H:%M:%S'
```

自定义格式有以下占位符：

| 参数 | 说明 |
| --- | --- |
| %a |         Abbreviated weekday name |
| %A |         Full weekday name |
| %b |         Abbreviated month name |
| %B |         Full month name |
| %c |         Date and time representation appropriate for locale |
| %d |         Day of month as decimal number (01 – 31) |
| %H |         Hour in 24-hour format (00 – 23) |
| %I |         Hour in 12-hour format (01 – 12) |
| %j |         Day of year as decimal number (001 – 366) |
| %m |         Month as decimal number (01 – 12) |
| %M |         Minute as decimal number (00 – 59) |
| %p |         Current locale's A.M./P.M. indicator for 12-hour clock |
| %S |         Second as decimal number (00 – 59) |
| %U |         Week of year as decimal number, with Sunday as first day of week (00 – 53) |
| %w |         Weekday as decimal number (0 – 6; Sunday is 0) |
| %W |         Week of year as decimal number, with Monday as first day of week (00 – 53) |
| %x |         Date representation for current locale |
| %X |         Time representation for current locale |
| %y |         Year without century, as decimal number (00 – 99) |
| %Y |         Year with century, as decimal number |
| %z, %Z |         Either the time-zone name or time zone abbreviation, depending on registry settings; no characters if time zone is unknown |
| %% |         Percent sign |

## git 设置保存凭证

```bash
git config --global credential.helper wincred
```
