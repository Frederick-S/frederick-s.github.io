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

本文描述的模板引擎采用编译型的方式，原文的作者将模板编译为了 `Python` 代码，这里为了加深理解，实现了 `.NET Core` 版本的编译。

## 参考
* [A Template Engine](https://aosabook.org/en/500L/a-template-engine.html)
