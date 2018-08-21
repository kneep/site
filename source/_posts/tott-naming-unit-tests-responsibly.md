---
title: 马桶上的测试：以对象功能命名单元测试
date: 2018-07-28 22:06:22
categories:
 - 技术
tags:
 - 马桶上的测试 
 - 译文
---

对于一个类，尝试这样来命名测试方法集，每个方法描述对象的一个功能，句首的类名则省略。<!-- more -->以Java为例：
```java
class HtmlLinkRewriterTest ... {
    void testAppendsAdditionalParameterToUrlsInHrefAttributes(){?}
    void testDoesNotRewriteImageOrJavascriptLinks(){?}
    void testThrowsExceptionIfHrefContainsSessionId(){?}
    void testEncodesParameterValue(){?}
}
```
这可以解读为：
```
HtmlLinkRewriter appends additional param to URLs in href attrs.
HtmlLinkRewriter does not rewrite image or JavaScript links.
HtmlLinkRewriter throws exception if href contains session ID.
HtmlLinkRewriter encodes parameter value.   
```

**好处**

这些测试方法描述了对象的功能，而不是公开的接口和输入输出。将来的工程师能更容易地知道这些方法是干什么用的，不需要去研究代码。

这样的命名习惯能给你预警。例如，测试方法名前面加上类名无法构成一句通顺的句子，则说明这个测试方法放错位置了。说不清楚功能的类，一般要分解成更小更清晰的类。
