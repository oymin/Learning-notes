# Nexus Repository Manager OSS 3.x 私服搭建

> - [下载地址](https://www.sonatype.com/download-oss-sonatype)
> - 安装系统：windows

## 安装环境及版本

1. 操作系统：Windows 10
2. nexus版本：nexus-3.16.2-01-win64 [下载地址](https://www.sonatype.com/download-oss-sonatype)

----

## 安装步骤

1、解压缩安装包，目录如下

>`nexus-3.16.2-01`  
    `sonatype-work`

2、使用`以管理员身份运行` cmd 命令窗口，进入 `nexus-3.16.2-01` 目录的 `bin` 文件夹运行命令

```bash
nexus.exe /run
```

3、访问图形界面

[http://localhost:8081](http://localhost:8081)

默认有2个登账号：

```yaml
# 管理员权限账号
账号：admin  密码：admin123

# 只有查看权限账号
账号：anonymous  密码：anonymous
```

----

## nexus 仓库类型

### nexus 有4中仓库类型

![仓库类型](.\images\01.jpg)

**hosted：** 宿主仓库，部署自己的 jar 到这个类型的仓库，包括 releases 和 snapshot 两部分，Releases 公司内部发布版本仓库、 Snapshots 公司内部测试版本仓库

**proxy：** 代理仓库，用于代理远程的公共仓库，如 maven 中央仓库，用户连接私服，私
服自动去中央仓库下载 jar 包或者插件。

**group:** 仓库组，用来合并多个 hosted/proxy 仓库，通常我们配置自己的 maven 连接仓
库组。

**virtual(虚拟)：** 兼容 Maven1 版本的 jar 或者插件

----

## 仓库功能分类

**maven-releases：** 本地仓库，存储 releases构件。

**maven-snapshots：** 本地仓库，存储 snapshots构件。

**maven-central：** 代理仓库，代理中央仓库，默认从[https://repo1.maven.org/maven2/](https://repo1.maven.org/maven2/)拉取jar 

**maven-public：** 仓库分组，把上面三个仓库组合在一起对外提供服务，在本地maven基础配置settings.xml中使用。

----

## 将项目发布到私服

### 1. Maven 配置

修改 `settings.xml` 文件，配置连接私服的用户和密码

```xml
<servers>
    <!-- 连接发布版本项目仓库 -->
    <server>
        <id>releases</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
    <!-- 连接测试版本项目仓库 -->
    <server>
        <id>snapshots</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>
```

### 2. 配置项目 pom.xml

配置私服仓库的地址，本公司的自己的 jar 包会上传到私服的宿主仓库，根据工程的版本号决定上传到哪个宿主仓库，如果版本为 release 则上传到私服的 release 仓库，如果版本为snapshot 则上传到私服的 snapshot 仓库

```xml
<distributionManagement>
    <repository>
        <id>releases</id>
        <url>http://localhost:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <url>http://localhost:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

### 3. 将项目将打成 jar 包发布到私服

对项目执行 maven 的 `deploy` 命令打包发布到私服

![执行deploy命令](.\images\02.jpg)

发布成功

![发布成功](.\images\03.jpg)

----

## 从私服下载 jar 包

配置 maven 的 `settings.xml` 文件

```xml
<profiles>
    <profile>
        <!--profile 的 id-->
        <id>dev</id>
        <repositories>
            <repository>
                <!--仓库 id，repositories 可以配置多个仓库，保证 id 不重复-->
                <id>nexus</id>
                <!--仓库地址，即 nexus 仓库组的地址-->
                <url>http://localhost:8081/repository/maven-public/</url>
                <!--是否下载 releases 构件-->
                <releases>
                    <enabled>true</enabled>
                </releases>
                <!--是否下载 snapshots 构件-->
                <snapshots>
                    <enabled>true</enabled>
                </snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <!-- 插件仓库，maven 的运行依赖插件，也需要从私服下载插件 -->
            <pluginRepository>
                <!-- 插件仓库的 id 不允许重复，如果重复后边配置会覆盖前边 -->
                <id>public</id>
                <name>Public Repositories</name>
                <url>http://localhost:8081/repository/maven-public/</url>
            </pluginRepository>
        </pluginRepositories>
    </profile>
</profiles>
```

激活配置的 `profile`

```xml
<settings>
	<activeProfiles>
		<activeProfile>dev</activeProfile>
	</activeProfiles>
</settings>
```

----

## 安装第三方 jar 包到本地仓库

先 CMD 进入到 jar 包所在位置，运行

```yaml
mvn install:install-file -DgroupId=com.alibaba -DartifactId=fastjson -Dversion=1.1.37 -Dfile= fastjson-1.1.37.jar -Dpackaging=jar
```

----

## 第三方 jar 包安装到私服

### 首先配置 Maven `settings.xml` 配置文件

```xml
<server>
    <id>3rd_part</id>
    <username>admin</username>
    <password>admin123</password>
</server>
```

### 执行安装 jar 包命令

```yaml
mvn deploy:deploy-file -DgroupId=com.alibaba -DartifactId=fastjson -Dversion=1.1.37 -Dpackaging=jar -Dfile=fastjson-1.1.37.jar -Durl=http://localhost:8081/repository/3rd_part/ -DrepositoryId=3rd_part
```
