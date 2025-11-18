# 核心

它允许我们创建一个新类（子类），这个新类可以获取另一个已存在类（父类）的属性和方法。

在Python中，实现继承非常简单。在定义子类时，将父类的名字放在类名后面的括号里即可。

```python
class ParentClass:
    # 父类的属性和方法
    pass

class ChildClass(ParentClass):
    # 子类可以继承ParentClass的属性和方法
    # 也可以定义自己的属性和方法
    pass
```

# 方法解析顺序

先在子类查找，再去父类查找，一直向上直到发现

# Object-Oriented Design

组合表示has a关系

继承表示is a关系

# Multiple Inheritance

有多个父类

## 方法解析顺序

你可以使用 `ClassName.mro()` 或者 `help(ClassName)` 来查看这个顺序。