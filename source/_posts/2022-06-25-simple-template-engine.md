title: 'A Template Engine - 简易模板引擎实现'
tags:
- Template Engine
---

> 本文内容主要来源于 [A Template Engine](https://aosabook.org/en/500L/a-template-engine.html)。

## 支持的语法
首先来看一下这个模板引擎所支持的语法。

### 变量
使用 `{{ variable }}` 来表示变量，例如：

```
<p>Welcome, {{user_name}}!</p>
```

如果 `user_name` 是 `Tom`，则最后渲染的结果为：

```
<p>Welcome, Tom!</p>
```

### 对象属性和方法
除了字面量外，模板引擎的变量还支持复杂对象，可以通过点操作符来访问对象的属性或方法，例如：

```
<p>The price is: {{product.price}}, with a {{product.discount}}% discount.</p>
```

注意如果访问的是对象的方法，则不需要在方法名后添加 `()`，模板引擎会自动解析并调用方法。

同时，还可以使用管道操作符来链式调用过滤器，从而改变所渲染的变量值，例如：

```
<p>Short name: {{story.subject|slugify|lower}}</p>
```

### 条件判断
使用 `{% if condition %} body {% endif %}` 来表示条件判断，例如：

```
{% if user.is_logged_in %}
    <p>Welcome, {{ user.name }}!</p>
{% endif %}
```

### 循环
使用 `{% for item in list %} body {% endfor %}` 来表示循环，例如：

```
<p>Products:</p>
<ul>
{% for product in product_list %}
    <li>{{ product.name }}: {{ product.price|format_price }}</li>
{% endfor %}
</ul>
```

### 注释
使用 `{# comment #}` 来表示注释，例如：

```
{# This is the best template ever! #}
```

## 实现
一般来说，一个模板引擎主要做两件事：模板解析和渲染。这里要实现的模板引擎的渲染包括：

* 管理动态数据
* 执行逻辑语句，例如 `if`，`for`
* 实现点操作符访问和过滤器执行

类似于编程语言的实现，模板引擎的解析也可以分为解释型和编译型两种。对于解释型来说，模板解析阶段需要生成某个特定的数据结构，然后在渲染阶段遍历该数据结构并执行所遇到的每一条指令；而对于编译型来说，模板解析阶段直接生成可执行代码，而渲染阶段则大大简化，直接执行代码即可。

本文描述的模板引擎采用编译型的方式，原文的作者将模板编译为了 `Python` 代码，这里为了进一步加深理解，实现了 `.NET Core` 版本的简单编译。

## 编译为 C# 代码
在介绍模板引擎实现之前，先来看一下模板引擎编译出的 `C#` 代码示例，对于如下的模板：

```
<p>Welcome, {{userName}}!</p>
<p>Products:</p>
<ul>
{% for product in productList %}
    <li>{{ product.Name }}:
        {{ product.Price|FormatPrice }}</li>
{% endfor %}
</ul>
```

模板引擎会生成类似于下面的代码：

```cs
public string Render(Dictionary<string, object> Context Context, Func<object, string[], object> ResolveDots)
{
    var result = new List<string>();
    var userName = Context["userName"];
    var productList = Context["productList"];
    result.AddRange(new List<string> {"<p>Welcome, ", Convert.ToString(userName), "!</p><p>Products:</p><ul>"});
    foreach (var product in ConvertToEnumerable(productList)) {
        result.AddRange(new List<string> {"<li>", Convert.ToString(ResolveDots(product, new [] { "Name" })), ":", Convert.ToString(FormatPrice(ResolveDots(product, new [] { "Price" }))), "</li>"});
    }
    result.Add("</ul>");
    return string.Join(string.Empty, result);
}
```

其中 `Context` 表示全局上下文，用于获取渲染需要的动态数据，例如例子中的 `userName`，`Render` 方法会先从 `Context` 中提取出模板中所有需要的变量；`ResolveDots` 是一个函数指针，用于执行点操作符调用；而变量的值都会通过 `Convert.ToString` 转为字符串。

模板引擎的最终产物是一个字符串，所以在 `Render` 中先使用一个 `List` 保存每一行的渲染结果，最后再将 `List` 转换为字符串。

`.NET` 编译器提供了 `Microsoft.CodeAnalysis.CSharp.Scripting` 包来将某段字符串当做 `C#` 代码执行，所以最终模板引擎生成的代码将通过如下方式执行：

```cs
var code = "some code";
var scriptOptions = ScriptOptions.Default.WithImports("System", "System.Collections.Generic");
var script = CSharpScript.RunAsync(code, scriptOptions, yourCustomGlobals);

return script.Result.ReturnValue.ToString();
```

## 模板引擎编写
### Template
`Template` 是整个模板引擎的核心类，它首先通过模板和全局上下文初始化一个实例，然后调用 `Render` 方法来渲染模板：

```cs
var context = new Dictionary<string, object>()
{
    { "numbers", new[] { 1, 2, 3 } },
};
string text = @"<ol>{% for number in numbers %}<li>{{ number }}</li>{% endfor %}</ol>";
Template template = new Template(text, context);
string result = template.Render();
```

这里将 `text` 传入 `Template` 的构造函数后，会在构造函数中完成模板解析，后续的 `Render` 调用都不需要再执行模板解析。

### CodeBuilder
在介绍 `Template` 的实现之前，需要先了解下 `CodeBuilder`，`CodeBuilder` 用于辅助生成 `C#` 代码，`Template` 通过 `CodeBuilder` 添加代码行，以及管理缩进（原文的作者使用 `Python` 作为编译的目标语言所以这里需要维护正确的缩进，`C#` 则不需要），并最终通过 `CodeBuilder` 得到可执行代码。

`CodeBuilder` 内部维护了一个类型为 `List<object>` 的变量 `Codes` 来表示代码行，这里的 `List` 容器类型不是字符串是因为 `CodeBuilder` 间可以嵌套，一个 `CodeBuilder` 可以作为一个完整的逻辑单元添加到另一个 `CodeBuilder` 中，并最终通过自定义的 `ToString` 方法来生成可执行代码：

```cs
public class CodeBuilder
{
    private const int IndentStep = 4;

    public CodeBuilder()
        : this(0)
    {
    }

    public CodeBuilder(int indentLevel)
    {
        this.Codes = new List<object>();
        this.IndentLevel = indentLevel;
    }

    private List<object> Codes
    {
        get;
    }

    private int IndentLevel
    {
        get;
        set;
    }
}
```

`CodeBuilder` 的 `AddLine` 方法非常简单，即根据缩进层级补齐空格后添加一行代码（这里 `C#` 版本保留了 `Python` 版本缩进的功能）：

```cs
public void AddLine(string line)
{
    this.Codes.AddRange(new List<object> { new string(' ', this.IndentLevel), line, "\n" });
}
```

`Indent` 和 `Dedent` 用于管理 `Python` 代码的缩进层级：

```cs
public void Indent()
{
    this.IndentLevel += IndentStep;
}

public void Dedent()
{
    this.IndentLevel -= IndentStep;
}
```

`AddSection` 用于创建一个新的 `CodeBuilder` 对象，并将其添加到当前 `CodeBuilder` 的代码行中，后续对子 `CodeBuilder` 的修改都会反应到父 `CodeBuilder` 中：

```cs
public CodeBuilder AddSection()
{
    CodeBuilder section = new CodeBuilder(this.IndentLevel);

    this.Codes.Add(section);

    return section;
}
```

最后重写了 `ToString()` 方法来生成可执行代码：

```cs
public override string ToString()
{
    return string.Join(string.Empty, this.Codes.Select(code => code.ToString()));
}
```

### Template 实现
#### 编译
模板引擎的模板解析阶段发生在 `Template` 的构造函数中：

```cs
public Template(string text, Dictionary<string, object> context)
{
    this.Context = context;
    this.CodeBuilder = new CodeBuilder();
    this.AllVariables = new HashSet<string>();
    this.LoopVariables = new HashSet<string>();

    this.Initialize(text);
}
```

`Python` 版本的代码支持多个 `context`，会由构造函数统一合并为一个上下文对象，这里只简单实现仅支持一个 `context`；`AllVariables` 用于记录模板 `text` 中需要用到的变量名，例如 `userName`，然后在代码生成阶段就可以遍历 `AllVariables` 并通过 `var someName = Context[someName];` 生成局部变量；不过由于模板中的变量可能还会有循环语句用到的临时变量，这些变量会记录到 `LoopVariables` 中，最终代码生成阶段用到的变量为 `AllVariables - LoopVariables`。

接着我们再来看 `Initialize` 方法：

```cs
private void Initialize(string text)
{
    this.CodeBuilder.AddLine("var result = new List<string>();");

    var variablesSection = this.CodeBuilder.AddSection();

    // 解析 text
    // ...

    foreach (string variableName in new HashSet<string>(this.AllVariables.Except(this.LoopVariables)))
    {
        variablesSection.AddLine(string.Format("var {0} = Context[{1}];", variableName, this.ConvertToStringLiteral(variableName)));
    }

    this.CodeBuilder.AddLine("return string.Join(string.Empty, result);");
}
```

`Initialize` 首先会通过 `CodeBuilder` 分配一个 `List` 保存所有的代码行，然后新建一个子 `CodeBuilder` 来保存所有的局部变量，接着解析 `text`，在完成 `text` 的解析后就能知道模板中使用了哪些变量，从而根据 `AllVariables - LoopVariables` 生成局部变量，最后将所有的代码行转成字符串。

同时，原文作者在这里有一个优化，相比于在生成的代码中不断的调用 `result.Add(xxx)`，从性能上考虑可以将多个操作合并为一个即 `result.AddRange(new List<string> { xxx })`，从而引出了辅助变量 `buffered` 和辅助方法 `FlushOutput`：

```cs
var buffered = new List<string>();

private void FlushOutput(List<string> buffered)
{
    if (buffered.Count == 1)
    {
        this.CodeBuilder.AddLine(string.Format("result.Add({0});", buffered[0]));
    }
    else if (buffered.Count > 1)
    {
        this.CodeBuilder.AddLine(string.Format("result.AddRange(new List<string> {{{0}}});", string.Join(", ", buffered)));
    }

    buffered.Clear();
}
```

在解析 `text` 时，并不会处理完一个 `token` 就执行一次 `this.CodeBuilder.AddLine`，而是将多个 `token` 的处理结果批量的追加到最终的可执行代码中。

接着，再回到 `Initialize` 方法中，由于模板中 `if`，`for` 可能存在嵌套，为了正确处理嵌套语句，这里引入一个栈 `var operationStack = new Stack<string>();` 来处理嵌套关系。例如，假设模板中存在 `{% if xxx %} {% if xxx %} {% endif %} {% endif %}`，每次遇到 `if` 时则执行入栈，遇到 `endif` 时则执行出栈，如果出栈时栈为空则说明 `if` 语句不完整，并抛出语法错误。

那么，如何解析 `text` 呢？这里使用正则表达式来将 `text` 分割为 `token`：

```cs
private static Regex tokenPattern = new Regex("(?s)({{.*?}}|{%.*?%}|{#.*?#})", RegexOptions.Compiled);
var tokens = tokenPattern.Split(text);
```

其中正则表达式中的 `(?s)` 使得 `.` 能够匹配换行符。

例如对于模板：

```
<ol>{% for number in numbers %}<li>{{ number }}</li>{% endfor %}</ol>
```

分割后的 `tokens` 为：

```
[
    '<ol>',
    '{% for number in numbers %}',
    '<li>',
    '{{ number }}',
    '</li>',
    '{% endfor %}',
    '</ol>'
]
```

然后我们就可以遍历 `tokens` 处理了，每种 `token` 对应一种策略，如果是注释，则忽略：

```cs
if (token.StartsWith("{#"))
{
    continue;
}
```

如果是变量，则解析变量的表达式（表达式解析会在后面介绍）的值，然后再将其转为字符串：

```cs
else if (token.StartsWith("{{"))
{
    string expression = this.EvaluateExpression(token.Substring(2, token.Length - 4).Trim());

    buffered.Add(string.Format("Convert.ToString({0})", expression));
}
```

而如果是 `{%` 则是最复杂的场景，需要对 `if` 和 `for` 分别处理，并结合 `operationStack` 判断是否存在语法错误：

```cs
else if (token.StartsWith("{%"))
{
    // 执行 if，for 语句前先将 buffered 中的语句写到 CodeBuilder 中
    this.FlushOutput(buffered);

    // 将 {% %} 间的内容首尾去除空格后按照空格分割
    var words = token.Substring(2, token.Length - 4).Trim().Split();

    if (words[0] == "if")
    {
        // 只支持 if xxx 的形式
        if (words.Length != 2)
        {
            this.SyntaxError(string.Format("Don't understand if, token: {0}", token));
        }

        // 入栈，表示 if 语句的开始，需要有对应的出栈
        operationStack.Push("if");

        // if 条件也需要对表达式求值，C# 的 if 没有 Python 那么灵活，所以这里定义了一个 IsTure 方法来将表达式转为 bool 值
        this.CodeBuilder.AddLine(string.Format("if (IsTrue({0})) {{", this.EvaluateExpression(words[1])));
        this.CodeBuilder.Indent();
    }
    else if (words[0] == "for")
    {
        // 只支持 for xxx in xxx 的形式
        if (words.Length != 4 || words[2] != "in")
        {
            this.SyntaxError(string.Format("Don't understand for, token: {0}", token));
        }

        // 入栈，表示 for 语句的开始，需要有对应的出栈
        operationStack.Push("for");

        // 记录遇到的循环变量
        this.AddVariable(words[1], this.LoopVariables);

        // 这里定义了 ConvertToEnumerable 方法将循环的对象转为可迭代的对象，同时也需要对表达式求值
        this.CodeBuilder.AddLine(string.Format("foreach (var {0} in ConvertToEnumerable({1})) {{", words[1], this.EvaluateExpression(words[3])));
        this.CodeBuilder.Indent();
    }
    else if (words[0].StartsWith("end"))
    {
        if (words.Length != 1)
        {
            this.SyntaxError(string.Format("Don't understand end, token: {0}", token));
        }

        var endWhat = words[0].Substring(3);

        // 没有配对的开始语句
        if (operationStack.Count == 0)
        {
            this.SyntaxError(string.Format("Too many ends, token: {0}", token));
        }

        var startWhat = operationStack.Pop();

        // 开始和结束的语句不匹配，例如 if 以 endfor 结束
        if (startWhat != endWhat)
        {
            this.SyntaxError(string.Format("Mismatched end tag, token: {0}", token));
        }

        this.CodeBuilder.AddLine("}");
        this.CodeBuilder.Dedent();
    }
}
```

最后剩下的情况就按照单纯的字符串处理：

```cs
else
{
    if (!string.IsNullOrEmpty(token))
    {
        buffered.Add(this.ConvertToStringLiteral(token));
    }
}
```

#### 编译表达式
前面提到处理 `token` 时有时候需要对表达式求值，表达式分三种情况，第一种是管道流：

```cs
if (expression.Contains("|"))
{
    var pipes = expression.Split('|');
    var code = this.EvaluateExpression(pipes[0]);

    foreach (var function in pipes.ToList().GetRange(1, pipes.Length - 1))
    {
        code = string.Format("{0}({1})", function, code);
    }

    return code;
}
```

首先根据 `|` 切分，数组的第一个元素是一个子表达式，递归调用即可；数组后面的元素表示的是要调用哪一个预定义的函数（要支持这些函数需要在最终生成的代码中有定义），例如 `FormatPrice`，本质上是个函数调用，直接拼接即可。

第二种是点操作符：

```cs
else if (expression.Contains("."))
{
    var dots = expression.Split('.');
    var code = this.EvaluateExpression(dots[0]);
    var arguments = dots.ToList().GetRange(1, dots.Length - 1).Select(x => this.ConvertToStringLiteral(x)).ToArray();

    return string.Format("ResolveDots({0}, {1})", code, string.Format("new [] {{ {0} }}", string.Join(", ", arguments)));
}
```

首先按照 `.` 进行分割，数组的第一个元素是一个子表达式，递归调用即可；数组后面的元素作为一系列的链式点操作符调用的参数，交由辅助函数 `ResolveDots` 处理：

```cs
private object ResolveDots(object value, string[] arguments)
{
    foreach (string argument in arguments)
    {
        var members = value.GetType().GetMember(argument);

        // Suppose there is only one member
        var member = members[0];

        switch (member.MemberType)
        {
            case MemberTypes.Property:
                value = ((PropertyInfo)member).GetValue(value);

                break;
            case MemberTypes.Field:
                value = ((FieldInfo)member).GetValue(value);

                break;
            case MemberTypes.Method:
                value = ((MethodInfo)member).Invoke(value, null);

                break;
            default:
                throw new TemplateRuntimeException(string.Format("Unsupported member type {0}", member.MemberType));
        }
    }

    return value;
}
```

`ResolveDots` 通过反射判断点操作符对应的是属性还是方法，继而执行相应的调用。

第三种就是单纯的变量，直接输出即可：

```cs
else
{
    this.AddVariable(expression, this.AllVariables);

    return expression;
}
```

## 总结
以上就实现了一个简易的模板引擎，虽然还有很多高级的功能没有实现，但已足够作为一个引导式的教程。

完整的代码可参考原作者的 [代码](https://github.com/aosabook/500lines/tree/master/template-engine/code) 及 `C#` 版的 [SimpleTemplate](https://github.com/read-and-code/SimpleTemplate)。

## 参考
* [A Template Engine](https://aosabook.org/en/500L/a-template-engine.html)
