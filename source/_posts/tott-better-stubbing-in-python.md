---
title: 马桶上的测试：在Python中更好地使用stub
date: 2018-07-08 22:48:28
categories:
 - 技术
tags:
 - 马桶上的测试 
 - 译文
---

你学过函数stub、mock对象和fake（译注：stub、mock和fake都是用于生成测试复制品的技术，概念上也略有不同，除stub外没有通用的中文名称，此处统一不译）。你忍不住要用stub替换掉耗时或与I/O相关的内置函数。<!-- more -->例如：

```python
def Foo(path):
    if os.path.exists(path):
        return DoSomething()
    else:
        return DoSomethingElse()

def testFoo(self): # 在你单元测试类的某处
    old_exists = os.path.exists
    try:
        os.path.exists = lambda x: True
        self.assertEqual(Foo('bar'), something)
        os.path.exists = lambda x: False
        self.assertEqual(Foo('bar'), something_else)
    finally:
        # 记得清理
        os.path.exists = old_exists
```

恭喜，你的测试代码达到了100%的覆盖率。不幸的是，你会发现这段测试代码可能会很诡异地失败。举个例子，假设如下`DoSomethingElse`会去检查另一个文件是否存在：
```python
def DoSomethingElse():
    assert os.path.exists(some_other_file)
    return some_other_file
```

现在第二次调用`Foo`的地方抛出异常，因为`os.path.exists`返回了`False`所以断言失败。

通过stub或mock`DoSomethingElse`，你可以解决这个问题，但在现实世界中，这项任务还是有点吓人的。其实，把你要stub的内置函数参数化，是更安全快速的做法：
```python
def Foo(path, path_checker=os.path.exists):
    if path_checker(path):
        return DoSomething()
    else:
        return DoSomethingElse()

def testFoo(self):
    self.assertEqual(Foo('bar', lambda x: True), something)
    self.assertEqual(Foo('bar', lambda x: False), something_else)
```