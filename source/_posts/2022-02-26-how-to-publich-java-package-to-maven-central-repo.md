title: 发布 JAR 包到 Maven 中央仓库
tags:
- Java
- Maven
---

`Sonatype OSSRH (OSS Repository Hosting)` 提供了 `JAR` 包发布服务，并支持自动将 `JAR` 包同步到 `Maven` 中央仓库，所以我们将 `JAR` 包发布到 `Sonatype OSSRH` 即可。

## 创建 Sonatype 工单
第一步在 [Sonatype](https://issues.sonatype.org/secure/Signup!default.jspa) 上注册一个账号，创建成功后在上面创建一个 `Issue`，`Project` 选择 `Community Support - Open Source Project Repository Hosting (OSSRH)`，`Issue Type` 选择 `New Project`：

![alt](/images/sonatype.png)

这里要注意的是 `Group Id` 的填写，根据 [Coordinates](https://central.sonatype.org/publish/requirements/coordinates/) 的描述，这里分两种情况：

1. 你拥有某个域名，如 `example.com`
2. 你没有域名，但是你的代码托管在了某个代码托管服务上，如 `GitHub`

对于第一种情况，你的 `Group Id` 可以是任何以 `com.example` 为前缀的字符串，如 `com.example.myawesomeproject`。不过，`Sonatype` 会要求你证明确实拥有 `example.com` 域名，你需要在你的域名注册商那创建一条 `TXT` 记录，其内容就是你创建的 `Issue` 的工单号，如 `OSSRH-12345`，具体步骤可参考 [How do I set the TXT record needed to prove ownership of my Web Domain?](https://central.sonatype.org/faq/how-to-set-txt-record/)。

对于第二种情况，以 `GitHub` 为例，你的 `Group Id` 必须是 `io.github.myusername`，`myusername` 是你的 `GitHub` 账户名或者是组织名，类似的，为了证明你对 `myusername` 的所有权，你需要在 `myusername` 下创建一个公开的仓库，仓库名称为你所创建 `Issue` 的工单号，如 `OSSRH-12345`，认证完成之后你就可以删掉这个仓库。`Sonatype` 所支持的代码托管服务如下：

|Service   |Example groupId   |
|---|---|
|GitHub   | 	io.github.myusername   |
|GitLab   | 	io.gitlab.myusername   |
|Gitee   | 	io.gitee.myusername   |
|Bitbucket   | 	io.bitbucket.myusername   |
|SourceForge   |io.sourceforge.myusername   |

工单示例可参考 [Publish my open source java package](https://issues.sonatype.org/browse/OSSRH-78488)。

## 安装 GPG
`GPG` 用于对所发布的包进行签名，在 [GnuPG](https://www.gnupg.org/download/index.html) 根据自己的操作系统下载 `GPG` 安装包，安装完成后执行 `gpg --full-gen-key` 生成秘钥对，选择默认选项即可，生成秘钥对时会要求输入姓名、邮箱、注释和密码，其中密码在发布阶段会用到，秘钥生成信息类似如下：

```
pub   rsa3072 2022-02-26 [SC]
      E892F685E5EA9005E0A2DE31F0F732425A15D81D
uid                      examplename <examplename@example.com>
sub   rsa3072 2022-02-26 [E]
```

其中 `E892F685E5EA9005E0A2DE31F0F732425A15D81D` 是秘钥的 `ID`，然后我们需要将公钥分发到公共的秘钥服务器上，这样 `Sonatype` 就可以通过这个公钥来验证我们所发布包的完整性：

```
gpg --keyserver keyserver.ubuntu.com --send-keys E892F685E5EA9005E0A2DE31F0F732425A15D81D
```

这里选择的秘钥服务器是 `keyserver.ubuntu.com`，也可以选择其他服务器，如 `keys.openpgp.org` 或者 `pgp.mit.edu`。

参考：

* [How to Publish Your Artifacts to Maven Central](https://dzone.com/articles/publish-your-artifacts-to-maven-central)
* [Coordinates](https://central.sonatype.org/publish/requirements/coordinates/)
* [GPG](https://central.sonatype.org/publish/requirements/gpg/)
