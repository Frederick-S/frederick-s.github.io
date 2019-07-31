title: Add property to web with JavaScript in SharePoint 2013 App
categories:
- SharePoint Add-in
---

## Introduction

`SharePoint App Model` provides a new approach to SharePoint development. And here is the question: where to save app data? There are several ways to save data in an app, you can create a list or connect to a database in Windows Azure or set custom properties in `AppManifest.xml`. But we can also save data to the property bag of a SharePoint web.

## Add property to web

Adding custom property to web is easy. Let's just see the code:

```js
var context = SP.ClientContext.get_current();
var web = context.get_web();

// Add a new property
var props = web.get_allProperties();
props.set_item("MyProperty", "My property value");

// Apply change to web
web.update();
context.load(web);

context.executeQueryAsync(function (sender, args) {
    alert("Success.");
}, function (sender, args) {
    alert("Request failed.");
});
```

How do we know "MyProperty" is really added to web? We can check it here: `http://your_web/_api/web/AllProperties?$select=MyProperty`. The result looks like this:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<entry xml:base="http://your_web/test/_api/" xmlns="http://www.w3.org/2005/Atom" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns:georss="http://www.georss.org/georss" xmlns:gml="http://www.opengis.net/gml">
    <id>http://your_web/_api/web/AllProperties</id>
    <category term="SP.PropertyValues" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme" />
    <link rel="edit" href="web/AllProperties" />
    <title />
    <updated>2013-12-04T04:37:07Z</updated>
    <author>
        <name />;
    </author>
    <content type="application/xml">
        <m:properties>
            <d:myproperty>My property value</d:myproperty>
        </m:properties>
    </content>
</entry>
```

## Read property from web

We have two approaches to read properties from web: `props.get_item("PropertyKey")` or `REST API`.

### The get_item method

```js
var context = SP.ClientContext.get_current();
var web = context.get_web();

var props = web.get_allProperties();
context.load(props);

context.executeQueryAsync(function (sender, args) {
    var prop = props.get_item("MyProperty");
    alert(prop);
}, function (sender, args) {
    alert("Request failed.");
});
```

### The REST API

```js
var executor = new SP.RequestExecutor('http://your_web');
executor.executeAsync({
    url: 'http://your_web/_api/web/AllProperties?$select=MyProperty',
    method: 'GET',
    headers: { "Accept": "application/json; odata=verbose" },
    success: function (response) {
        var obj = JSON.parse(response.body);
        alert(obj.d.MyProperty);
    },
    error: function (response) {
        alert(response.body);
    }
});
```

And here is the json query result, very straightforward:

```json
{
    "d": {
        "__metadata": {
            "id": "http://your_web/_api/web/AllProperties",
            "uri": "http://your_web/_api/web/AllProperties",
            "type": "SP.PropertyValues"
        },
        "MyProperty": "My property value"
    }
}
```

Reference:

- [HOW TO: Add properties to SPWeb property bag using JSOM in SharePoint 2013](http://blogs.technet.com/b/sharepointdevelopersupport/archive/2013/05/06/how-to-add-properties-to-spweb-property-bag-using-jsom-in-sharepoint-2013.aspx)
