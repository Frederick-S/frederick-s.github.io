title: JaCoCo 配置 Maven 多模块覆盖率测试
tags:
- Java
- JaCoCo
- Maven
---

首先构建一个多模块的 `Maven` 项目，项目结构如下：

```
├── product-service
│   └── pom.xml
├── sum-service
│   └── pom.xml
├── app
│   └── pom.xml
├── pom.xml
```

其中 `product-service` 和 `sum-service` 表示功能代码，`app` 负责测试用例的整合。`product-service` 和 `sum-service` 分别包含一个 `ProductService` 类和 `SumService` 类，具体代码如下：

```java
// ProductService
public class ProductService {
    public int product(int x, int y) {
        return x * y;
    }
}
```

```java
// SumService
public class SumService {
    public int sum(int x, int y) {
        return x + y;
    }
}
```

`app` 内的测试用例如下：

```java
public class AppTest {
    @Test
    public void shouldCalculateCorrectSumAndProduct() {
        Assert.assertEquals(10, new ProductService().product(2, 5));
        Assert.assertEquals(5, new SumService().sum(2, 3));
    }
}
```

然后在 `root` 模块的 `pom.xml` 文件中配置 `JaCoCo` 插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.5</version>
            <executions>
                <execution>
                    <id>prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

最后在 `app` 模块的 `pom.xml` 文件中配置 `JaCoCo` 插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
            <configuration>
                <argLine>${argLine} -Xms256m -Xmx2048m</argLine>
                <forkCount>1</forkCount>
                <runOrder>random</runOrder>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.5</version>
            <executions>
                <execution>
                    <id>report-aggregate</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report-aggregate</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

此时在 `root` 模块下执行 `mvn test`，执行成功后在 `app/target/site/jacoco-aggregate` 目录下就会生成各个模块的覆盖测试报告：

![alt](/images/jacoco-multiple-modules-demo.png)

完整的代码可参考 [GitHub](https://github.com/Frederick-S/jacoco-multiple-modules-demo)。

参考：

- [Jacoco report aggregation for code coverage](https://prismoskills.appspot.com/lessons/Maven/Chapter_06_-_Jacoco_report_aggregation.jsp)
