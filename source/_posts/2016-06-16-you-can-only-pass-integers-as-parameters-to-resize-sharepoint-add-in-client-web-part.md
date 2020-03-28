title: You can only pass integers as parameters to resize SharePoint Add-in Client Web Part
tags:
- SharePoint Add-in
---

I tried to resize a Client Web Part with non integers as size, but it failed. And that's why:

```js
var regex = RegExp(/(<\s*[Mm]essage\s+[Ss]ender[Ii]d\s*=\s*([\dAaBbCcDdEdFf]{8})(\d{1,3})\s*>[Rr]esize\s*\(\s*(\s*(\d*)\s*([^,\)\s\d]*)\s*,\s*(\d*)\s*([^,\)\s\d]*))?\s*\)\s*<\/\s*[Mm]essage\s*>)/);
var results = regex.exec(e.data);
if (results == null)
    return;
```

The regular expression shows that it doesn't allow float numbers.
