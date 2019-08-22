# SSH 协议

## 1. 生成 RSA 密钥对

```bash
ssh-Keygen -t rsa -C "your email"
```

生成的密钥对默认存储在用户目录下的 `.ssh/` 文件夹下

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/koax/.ssh/id_rsa):
Created directory '/c/Users/koax/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/koax/.ssh/id_rsa.
Your public key has been saved in /c/Users/koax/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:VNe3pRAdLbkyCYVjhyxzzgHbKTXIvFffB2LXFM8EPdM oymtofight@163.com
The key's randomart image is:
+---[RSA 3072]----+
|       o.+++++oX=|
|        *=X+* B=E|
|        +OoB * BB|
|       ...+ + +.o|
|        S.   o  .|
|                 |
|                 |
|                 |
|                 |
+----[SHA256]-----+

```

## 2. 在 Githup 网站添加公钥

将公钥添加到 githup 中
