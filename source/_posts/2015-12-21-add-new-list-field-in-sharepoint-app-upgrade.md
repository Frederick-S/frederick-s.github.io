title: Add new list field in SharePoint App upgrade
categories:
- SharePoint Add-in
---

I'm learning how to upgrade SharePoint App, and I met a problem when I added a new list field in the upgrade.

The [document](http://msdn.microsoft.com/en-us/library/office/dn265911(v=office.15).aspx) mentioned that if you added a field to a content type in the feature, you should add an `AddContentTypeField` element to the `VersionRange` section. But there is no `ContentType` in my app, it only has a `ListDefinition`. I tried to add an `AddContentTypeField`, unfortunately it throws exception.

So I tried another way. The document also mentioned that if you have changed a file that is referenced in an elements manifest file, you have to copy the `ElementManifest` element for the component from the `ElementManifests` section to the `ApplyElementManifests` section. When we added a new field to list, the `Schema.xml` is changed, although it's not referenced in a `ElementManifest`, I still copied `MyList/Elements.xml` to `ApplyElementManifests`, so it looks like this:

```xml
<UpgradeActions>
  <VersionRange>
    <ApplyElementManifests>
      ...
      <ElementManifest Location="MyList\Elements.xml" />
    </ApplyElementManifests>
  </VersionRange>
</UpgradeActions>
```

And it works. Hope it’s helpful.

Reference:

- [How to: Update app web components in SharePoint 2013](http://msdn.microsoft.com/en-us/library/office/dn265911(v=office.15).aspx)
