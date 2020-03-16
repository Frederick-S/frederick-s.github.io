title: 如何解决访问外部图片返回 403 Forbidden 错误
categories:
- HTTP
---

在某个 `Web` 应用中，引用了 `豆瓣读书` 的图片，大部分图片会无法显示并返回 `403 Forbidden` 错误，一个可能的原因是触发了 `豆瓣` 的图片防盗链机制。

一般来说，防盗链机制会判断图片请求的 `Request Headers` 里的 `Referer` 字段的值是否是允许的地址，如果不是，则不允许访问相应的资源。所以，一种可能的解决办法是在 `HTML` 页面中设置 `Referrer-Policy` 为 `no-referrer`，在发送 `HTTP` 请求时不发送 `Referer` 信息：

```html
<meta name="referrer" content="no-referrer" />
```

参考：

- [forbidden 403 on image URLS](https://stackoverflow.com/questions/49433452/forbidden-403-on-image-urls)
- [解决图片访问403 Forbidden问题](https://juejin.im/post/5cc50deff265da03a97af3e8)
