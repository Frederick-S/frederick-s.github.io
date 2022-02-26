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

参考：

* [How to Publish Your Artifacts to Maven Central](https://dzone.com/articles/publish-your-artifacts-to-maven-central)
* [Coordinates](https://central.sonatype.org/publish/requirements/coordinates/)
