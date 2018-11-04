---
title: 马桶上的测试：用Stub加速你的单元测试
categories:
  - 技术
tags:
  - 马桶上的测试
  - 译文
date: 2018-08-26 21:39:32
---

Michael Feathers把高质量的单元测试定义为“快速运行，并能帮我们隔离问题”。当你的代码需要访问数据库、访问其他服务器，或对时间长短有依赖等等情况发生时，要做到这一点很难。<!-- more -->

通过用自定义的对象来替换你模块的部分依赖，你可以彻底测试你的代码，提高覆盖率，还不失速度。你甚至能模拟少见的场景，比如数据库错误，来测试错误处理代码。

有各种术语来指代这里的“自定义对象”。Gerard Meszaros试图统一叫法，提供了如下定义：

* **Test Double** 是一个通用术语，用于指代替换生产环境中实际对象的各种测试对象。
* **Dummy** 对象会被传来传去，但不会被真正使用。它们通常用于填充参数列表。
* **Fakes** 有实际的实现代码，但有所简化（如InMemoryDatabase）。
* **Stubs** 为测试中用到的函数调用提供预先设置好的返回值。
* **Mocks** 看哪些函数会被调用，哪些不会被调用是否符合预期。

例如，为了测试像`IdGetter`类的`getIdPrefix()`这种简单的函数：
```java
public class IdGetter {  // 省略了构造函数
    public String getIdPrefix() {
        try {
            String s = db.selectString("select id from foo");
            return s.substring(0, 5);
        } catch (SQLException e) { return ""; }
    }
}
```
你可以这样写：
```java
db.execute("create table foo (id varchar(40))");  // 数据库的建立在setUp()中
db.execute("insert into foo (id) values ('hello world!')");
IdGetter getter = new IdGetter(db);
assertEquals("hello", getter.getIdPrefix());
```
以上测试代码能工作，但耗时相对较长（访问网络），可能不可靠（db所在机器可能会宕机），难以测试错误处理代码。你可以用stub来避免这些缺点：
```java
public class StubDbThatReturnsId extends Database {
    public String selectString(String query) { return "hello world"; }
}
public class StubDbThatFails extends Database {
    public String selectString(String query) throws SQLException {
        throw new SQLException("Fake DB failure");
    }
}
public void testReturnsFirstFiveCharsOfId() throws Exception {
    IdGetter getter = new IdGetter(new StubDbThatReturnsId());
    assertEquals("hello", getter.getIdPrefix());
}
public void testReturnsEmptyStringIfIdNotFound() throws Exception {
    IdGetter getter = new IdGetter(new StubDbThatFails());
    assertEquals("", getter.getIdPrefix());
}
```
