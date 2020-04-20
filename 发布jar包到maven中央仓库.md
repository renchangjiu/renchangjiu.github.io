# 如何发布 jar 包到 maven 中央仓库详细教程

java 开发中没少用各种 jar 包，特别是使用 maven，gradle 构建工具，方便有效。总是只取不予，也应该懂得奉献，当你写好了一个十分好用的 jar 包，想贡献出去给大家使用的时候，应该怎么做呢？当然是发布到 maven 的中央仓库了

**开始要注意这个几个 Maven 相关地址**：

- **工单管理**：[https://issues.sonatype.org](https://issues.sonatype.org/)

> 就是申请上传资格和 groupId 的地方,注册账号、创建和管理 issue，Jar 包的发布是以解决 issue 的方式起步的

- **构件仓库** : https://oss.sonatype.org/#welcome

> 把 jar 包上传到这里，Release 之后就会同步到 maven 中央仓库。

- **仓库镜像**: http://search.maven.org/

> 最终工件可以在这里搜索到。

## 一. 注册 sonatype 帐号(register sonatype)

访问[Sonatype](https://issues.sonatype.org/)，注册帐号  

## 二. 创建工单（nexus)

点击 header 头 “create”按钮创建一个工单，主要用途注册你上传 jar 包基本信息，主要 groupid，通过审核有两个目的：1.防止重复，约束 groupid 规范，定义 grupid 最好有所属的域名

比如：你申请 com.hippo 那么你最有有 hippo.com 这个域名的所有权。如果你不符合还有一个解决办法，groupid 申请以：com.github.{账号名来定义}

![创建](https://static.oschina.net/uploads/img/201712/27115346_dBwl.png)

创建成功后，接下来等待后台管理员审核，一般一个工作日以内，当 Issue 的 Status 变为 RESOLVED 后，就可以进行下一步操作了，否则，就等待… ![输入图片说明](https://static.oschina.net/uploads/img/201712/27120115_WJle.png)

审批通过后，通常管理员会给你留言配置方法，大体文字如下：

```txt
Configuration has been prepared, now you can:

Deploy snapshot artifacts into repository https://oss.sonatype.org/content/repositories/snapshots
Deploy release artifacts into the staging repository https://oss.sonatype.org/service/local/staging/deploy/maven2
Promote staged artifacts into repository 'Releases'......
please comment on this ticket when you promoted your first release, thanks
```

## 三. 配置项目工程 pom.xml

在工程的 pom.xml 文件中，引入 Sonatype 官方的一个通用配置 oss-parent，这样做的好处是很多 pom.xml 的发布配置不需要自己配置了

```xml
<parent>
    <groupId>org.sonatype.oss</groupId>
    <artifactId>oss-parent</artifactId>
    <version>7</version>
</parent>
```

并增加 Licenses、SCM、Developers 信息

```xml
<licenses>
    <license>
        <name>The Apache Software License, Version 2.0</name>
        <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
        <distribution>repo</distribution>
    </license>
</licenses>
<scm>
    <tag>master</tag>
    <url>git@github.com:cloudnil/marathon-client.git</url>
    <connection>scm:git:git@github.com:cloudnil/marathon-client.git</connection>
    <developerConnection>scm:git:git@github.com:cloudnil/marathon-client.git</developerConnection>
</scm>
<developers>
    <developer>
        <name>cloudnil</name>
        <email>cloudnil@126.com</email>
        <organization>CloudNil</organization>
    </developer>
</developers>
```

## 四. 配置 Maven setting.xml

setting.xml 放在 Maven 安装文件/conf 目录下

```xml
<servers>
    <server>
      <id>sonatype-nexus-snapshots</id>
      <username>Sonatype 账号</username>
      <password>Sonatype 密码</password>
    </server>
    <server>
      <id>sonatype-nexus-staging</id>
      <username>Sonatype 账号</username>
      <password>Sonatype 密码</password>
    </server>
</servers>
```

## 五. 配置 gpg-key

如果是使用的 windows,建议下载 git 客户端，可以在 git bash 提供窗口操作,在命令行中执行 `gpg --gen-key` 生成，过程中需要填写名字、邮箱等，其他步骤可以使用默认值，不过有个叫：Passphase 的参数需要记住，这个相当于是是密钥的密码，下一步发布过程中进行签名操作的时候会用到

![输入图片说明](https://static.oschina.net/uploads/img/201712/29102734_UxSx.png)

## 六. Deploy 部署

这步就简单了，就是一套命令：

```bash
mvn clean deploy -P sonatype-oss-release -Darguments="gpg.passphrase=密钥密码"
```

默认启动：maven-javadoc-plugin 插件 如果要忽略，可以跟参数：

```bash
-Dmaven.javadoc.skip=true
```

## 七. Release 发行

进入[Repositories](https://oss.sonatype.org/#stagingRepositories)查看发布好的构件，点击左侧的 Staging Repositories, 一般最后一个就是刚刚发布的 jar 了，此时的构件状态为 open。 打开命令行窗口，查看 gpg key 并上传到第三方的 key 验证库：

```bash
$ gpg --list-keys
---------------------------------------------
pub   2048R/824B4D7A 2016-01-06
uid       [ultimate] cloudnil <cloudnil@126.com>
sub   2048R/7A10AD69 2016-01-06

$ gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 824B4D7A
---------------------------------------------
gpg: sending key 824B4D7A to hkp server keyserver.ubuntu.com
```

![输入图片说明](https://static.oschina.net/uploads/img/201712/29165622_9sZu.png)

以上操作完成后, 回到[Repositories](https://oss.sonatype.org/#stagingRepositories), 选中刚才发布的构件，并点击上方的 close–>Confirm，在下边的 Activity 选项卡中查看状态，当状态变成 closed 后，执行 Release–>Confirm，并在下边的 Activity 选项卡中查看状态，成功后构件自动删除，一小段时间（约 1-2 个小时）后即可同步到 maven 的中央仓库。
