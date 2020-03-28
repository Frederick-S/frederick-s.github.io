title: 如何在 TypeScript 中定义属性名未知的对象类型
tags:
- TypeScript
---

在某些场景下，我们会使用 `{}` 来存储某类对象的映射关系，例如车主和车子的映射关系可记为：

```js
{
    'Alice': new Car(),
    'Bob': new Car()
}
```

而在 `TypeScript` 中声明这种对象的类型时遇到了问题：该对象的属性名是未知的，不过所有属性名对应的值的类型是确定的。这种情况下可以借助 [Indexable Types](https://www.typescriptlang.org/docs/handbook/interfaces.html#indexable-types) 解决，以上述的例子为例：

```ts
interface CarOwners {
    [key: string]: Car;
}

const carOwners: CarOwners = {
    'Alice': new Car(),
    'Bob': new Car()
}
```

参考：

- [Type annotation for object with unknown properties but known property types?](https://github.com/Microsoft/TypeScript/issues/7803)
