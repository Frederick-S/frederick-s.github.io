title: 在 Ubuntu 下使用 Puppeteer 时无法加载类库 libX11-xcb.so.1
tags:
- Puppeteer
- Ubuntu
---

在 `Ubuntu` 下运行 `Puppeteer` 遇到了如下错误：

```
error while loading shared libraries: libX11-xcb.so.1: cannot open shared object file: No such file or directory
```

需要安装以下依赖来解决：

```sh
sudo apt-get install gconf-service libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxss1 libxtst6 libappindicator1 libnss3 libasound2 libatk1.0-0 libc6 ca-certificates fonts-liberation lsb-release xdg-utils wget
```

参考：

- [How to fix puppetteer error ibX11-xcb.so.1 on Ubuntu](https://medium.com/@cloverinks/how-to-fix-puppetteer-error-ibx11-xcb-so-1-on-ubuntu-152c336368)
