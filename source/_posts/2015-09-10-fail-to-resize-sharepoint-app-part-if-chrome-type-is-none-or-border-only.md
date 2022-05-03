title: Fail to resize SharePoint App Part if "Chrome Type" is "None" or "Border Only"
tags:
- SharePoint Add-in
---

## Issue

Today I met a bug in SharePoint 2013, I failed to resize my app part by using `postMessage`. Here is my code:

```js
var message = "<Message senderId=" + senderId + ">" + "resize(" + width + "," + height + ")</Message>";
window.parent.postMessage(message, hostUrl);
```

It fails to work if I set `Chrome Type` to `None` or `Border Only`. And I found an error message in console:

```js
Uncaught TypeError: Cannot read property 'style' of null
```

Then I located the code in my web part page which throws the error:

```js
if (resizeWidth)
{
    document.getElementById(webPartDivId + '_ChromeTitle').style.cssText = widthCssText;
    cssText = 'width:100% !important;'
}
```

It tries to find the title element and resize it. But `document.getElementById(webPartDivId + '_ChromeTitle')` returns null if `Chrome Type` is `None` or `Border Only`!
Because the app part doesn't have a title under these 2 modes. Of course it will throw exception.

## Solution

This bug is described [here](http://social.msdn.microsoft.com/Forums/en-US/ea4b1ab6-3d44-4792-bce2-79056269852a/dynamic-width-and-height-for-iframe-app-part?forum=appsforsharepoint), you can install a [patch](http://blogs.technet.com/b/stefan_gossner/archive/2013/03/21/march-public-update-for-sharepoint-2013-available-and-mandatory.aspx) to fix this bug.

After the patch is installed, you can find that the original code is changed, it will resize the element only if it's not null:

```js
if (resizeWidth)
{
    var webPartChromeTitle = document.getElementById(webPartDivId + '_ChromeTitle');
    if (null != webPartChromeTitle)
    {
        webPartChromeTitle.style.cssText = widthCssText;
    }

    cssText = 'width:100% !important;'
}
```

Hope it's helpful.

## Reference

- [Dynamic Width and Height for iFrame (App Part)](http://social.msdn.microsoft.com/Forums/en-US/ea4b1ab6-3d44-4792-bce2-79056269852a/dynamic-width-and-height-for-iframe-app-part?forum=appsforsharepoint)
