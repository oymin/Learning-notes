# ElasticSearch安装head插件

## 1. 安装 NodeJS

从 ES6.0 开始，head 插件支持使得 node.js 运行。

NodeJS 下载地址：[https://nodejs.org/en/download/](https://nodejs.org/en/download/)

### 1.1 下载 msi 格式，直接下一步安装完成即可：

```yml
#确认是否安装成功 可执行下面的命令 查看是否又版本信息
node -v
npm -v
```

安装 cnpm

```yml
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 1.2 在 cmd 模式下进入 nodejs 的安装目录执行命令安装 grunt 命令

grunt 是基于 Node.js 的项目构建工具，可以进行打包压缩、测试、执行等等的工作，head 插件就是通过 grunt 启动

```yml
# 命令方式一
npm install -g grunt -cli --registry=https://registry.npm.taobao.org --no-proxy

# 命令方式二
npm install -g grunt -cli
```

确认安装成功

```yml
grunt -version
```

---

## 2. 安装 elasticsearch-head 插件

下载地址：[https://github.com/mobz/elasticsearch-head](https://github.com/mobz/elasticsearch-head)

### 2.1 解压后在根目录dos模式下执行 cnpm install 命令安装插件

![image](..\images\01.png)

### 2.2 安装完成，执行 grunt server 或者 npm run start 命令启动 head

![image](..\images\02.png)

### 2.3 可以修改 Gruntfile.js 文件修改 head 的配置

![image](..\images\03.png)

### 2.4 安装方法-2

执行下面命令：

```bash
git clone git://github.com/mobz/elasticsearch-head.git cd elasticsearch-head npm install npm run start open HTTP://本地主机:9100/
```
