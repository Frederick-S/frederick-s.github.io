title: 如何在 Dockerfile 中定义变量
categories:
- Docker
---

在某些场景下，编写 `Dockerfile` 时需要定义变量来避免重复出现的值，例如下面的例子中，`Gradle` 的版本号出现了三次，如果未来需要更新 `Gradle` 的版本号，则需要修改三次。

```docker
RUN wget https://services.gradle.org/distributions/gradle-6.3-bin.zip
RUN unzip gradle-6.3-bin.zip
RUN gradle-6.3/bin/gradle task
```

可以通过 `ARG variable-name=variable-value` 来定义一个变量，使用变量时通过 `$variable-name` 访问即可，对开头的例子使用变量修改后如下：

```docker
ARG GRADLE_VERSION=6.3
RUN wget https://services.gradle.org/distributions/gradle-$GRADLE_VERSION-bin.zip
RUN unzip gradle-$GRADLE_VERSION-bin.zip
RUN gradle-$GRADLE_VERSION/bin/gradle task
```

参考：

- [How to define a variable in a Dockerfile?](https://stackoverflow.com/questions/33935807/how-to-define-a-variable-in-a-dockerfile)
