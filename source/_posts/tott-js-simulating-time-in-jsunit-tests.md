---
title: 马桶上的测试：JavaScript：在jsUnit测试中模拟时间
date: 2018-08-23 22:15:20
categories:
 - 技术
tags:
 - 马桶上的测试 
 - 译文
---

有时候你需要在客户端侧测试带有`setTimeout()`的JavaScript代码，这些代码使用这个函数来调度将来要做的某些工作。`jsUnit`有`Clock.tick()`方法可以模拟时间流逝，又不至于让测试代码真的去睡眠。<!-- more -->例如，以下函数设置了回调，在整个4秒过程中更新一个状态信息：
```js
function showProgress(status) {
  status.message = "Loading";
  for (var time = 1000; time <= 3000; time += 1000) {
    // 前3秒每秒给消息添加一个“.”
    setTimeout(function() {
      status.message += ".";
    }, time);
  }
  setTimeout(function() {
    // 第4秒的特殊情况
    status.message = "Done";
  }, 4000);
}
```
用jsUnit来测试这个函数的代码是这样的：
```js
function testUpdatesStatusMessageOverFourSeconds() {
  Clock.reset(); // 清空事件队列中已有的超时函数
  var status = {};
  showProgress(status); // 调我们的函数.
  assertEquals("Loading", status.message);
  Clock.tick(2000); // 调用前2秒超时的函数
  assertEquals("Loading..",  status.message);
  Clock.tick(2000); // 再来一次，下2秒
  assertEquals("Done", status.message);
}
```
这项测试跑起来非常快——它不需要4秒。
`Clock`支持函数`setTimeout()`、`setInterval()`、`clearTimeout()`和`clearInterval()`。Clock对象是在`jsUnitMockTimeout.js`中定义的，它跟`jsUnitCore.js`在同一目录下。
