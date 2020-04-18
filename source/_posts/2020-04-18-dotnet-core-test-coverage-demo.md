title: .NET Core 覆盖率测试
tags:
- .NET Core
---

## 项目搭建
首先执行命令 `dotnet new classlib --name App` 来创建一个类库程序作为测试的对象，该类库只包含一个 `SumService` 类：

```cs
using System;

namespace App
{
    public class SumService
    {
        public static int Sum(int a, int b)
        {
            return a + b;
        }
    }
}
```

然后创建一个基于 `xunit` 的单元测试项目来编写测试用例，并将 `App` 类库项目作为项目引用加入到 `App.Tests` 项目中：

```
dotnet new xunit --name App.Tests
dotnet add .\App.Tests\App.Tests.csproj reference .\App\App.csproj
```

并编写一个测试用例 `SumServiceTest`：

```cs
using System;
using Xunit;

namespace App.Tests
{
    public class SumServiceTest
    {
        [Fact]
        public void ShouldReturn5()
        {
            Assert.Equal(5, SumService.Sum(2, 3));
        }
    }
}
```

接着创建一个解决方案，并将 `App` 和 `App.Tests` 项目加入到该解决方案中：

```
dotnet new sln --name App
dotnet sln add .\App\App.csproj .\App.Tests\App.Tests.csproj
```

最后项目结构如下：

```
├── App
│   └── App.csproj
│   └── SumService.cs
├── App.Tests
│   └── App.Tests.csproj
│   └── SumServiceTest.cs
├── App.sln
```

## 覆盖率测试
覆盖率测试依赖 `coverlet`，在创建单元测试项目时已自动添加了该依赖，执行测试时添加 `coverlet` 相关的参数即可生成测试覆盖率报告：

```
dotnet test --collect:"XPlat Code Coverage"
```

执行成功后会在 `App.Tests/TestResults/{random-string}` 目录下生成名为 `coverage.cobertura.xml` 的测试覆盖率报告。

但是，自动创建的单元测试项目默认添加的 `coverlet` 依赖是 `coverlet.collector`，目前还不支持在控制台中打印测试覆盖率报告：

> At the moment VSTest integration doesn't support all features of msbuild and .NET tool, for instance show result on console, report merging and threshold validation. We're working to fill the gaps.

如果希望在控制台中打印测试覆盖率报告可将 `coverlet.collector` 依赖改为 `coverlet.msbuild`：

```
dotnet remove .\App.Tests package coverlet.collector
dotnet add .\App.Tests\ package coverlet.msbuild
```

然后执行测试命令：

```
dotnet test /p:CollectCoverage=true
```

即可在控制台打印测试覆盖率报告：
```
Calculating coverage result...
  Generating report 'D:\WorkSpace\dotnet-core-coverlet-msbuild-demo\App.Tests\coverage.json'

+--------+------+--------+--------+
| Module | Line | Branch | Method |
+--------+------+--------+--------+
| App    | 100% | 100%   | 100%   |
+--------+------+--------+--------+

+---------+------+--------+--------+
|         | Line | Branch | Method |
+---------+------+--------+--------+
| Total   | 100% | 100%   | 100%   |
+---------+------+--------+--------+
| Average | 100% | 100%   | 100%   |
+---------+------+--------+--------+
```

## 集成 codecov
### coverlet.msbuild
集成 `codecov` 需要指定测试覆盖率报告文件的路径，暂不支持 `coverlet.msbuild` 默认生成的 `json` 格式的文件，可以在执行测试时添加 `/p:CoverletOutputFormat=opencover` 参数来生成 `opencover` 格式的文件，相应的 `.appveyor.yml` 文件内容如下：
```
image: Visual Studio 2019
before_build:
  - choco install codecov
build_script:
  - dotnet build
test_script:
  - dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
  - codecov -f ./App.Tests/coverage.opencover.xml
```

### coverlet.collector
使用 `coverlet.collector` 时每次生成的测试覆盖率报告所在的路径是随机的，所以需要将测试覆盖率报告复制到一个固定的路径中，可以使用如下的 `PowerShell` 脚本：

```ps
$source = "./App.Tests/TestResults"
$destination = $source
$filter = "coverage.cobertura.xml"
Get-ChildItem -Recurse -Path $source | Where-Object { $_.Name -match $filter } | Copy-Item -Destination $destination
```

相应的 `.appveyor.yml` 文件内容如下：

```
image: Visual Studio 2019
before_build:
  - choco install codecov
build_script:
  - dotnet build
test_script:
  - dotnet test --collect:"XPlat Code Coverage"
  - ps: ./FindCoverageFile.ps1
  - codecov -f ./App.Tests/TestResults/coverage.cobertura.xml
```

完整的代码可参考 [dotnet-core-coverlet-msbuild-demo](https://github.com/Frederick-S/dotnet-core-coverlet-msbuild-demo) 及 [dotnet-core-coverlet-collector-demo](https://github.com/Frederick-S/dotnet-core-coverlet-collector-demo)。

参考：

- [Code Coverage in .NET Core Projects](https://codeburst.io/code-coverage-in-net-core-projects-c3d6536fd7d7?gi=4c835df2b1bf)
- [Coverlet integration with VSTest (a.k.a. Visual Studio Test Platform)](https://github.com/tonerdo/coverlet/blob/master/Documentation/VSTestIntegration.md)
- [codecov-exe](https://github.com/codecov/codecov-exe)
- [Windows: File copy/move with filename regular expressions?](https://superuser.com/questions/149537/windows-file-copy-move-with-filename-regular-expressions)
