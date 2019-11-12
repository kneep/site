---
title: 在持续集成中使用Error Prone
date: 2019-11-07 14:35:49
categories:
 - 技术
tags:
 - 静态检查
 - DevOps
---


# Error Prone介绍

代码静态检查是持续集成流程中一个不可或缺的环节，对团队的代码质量有很大的帮助。对于开发基于Android的设备的厂商来说，选择第三方的静态检查工具往往是一件痛苦的事情，因为Android本身的代码库非常庞大，第三方静态检查工具虽然功能非常强大，但是和Android集成起来都比较费解，这类工具往往作为定时的全量扫描比较合适。持续集成要求轻、快，在此可以考虑使用Google出品的Error Prone作为一个轻量级的替代品。

[Error Prone](https://errorprone.info/)是Google研发的静态代码扫描工具，顾名思义，就是用于扫描Java各种易于出错的代码。Google在[这篇论文](https://ai.google/research/pubs/pub46576)中也提到了内部使用Error Prone的一些实践。

目前Google已经把它集成在AOSP里面，这对于设备厂商来说是一个好消息。

# 用法

编译模块的时候加上```RUN_ERROR_PRONE=true```，比如编译Android框架：

```bash
source build/envsentup.sh
lunch aosp_arm64-eng
RUN_ERROR_PRONE=true make -j8 framework
```

就可以在编译framework过程中嵌入Error Prone检查，不需要做其他任何设置。

# 检查结果示例

Error Prone是嵌入```javac```的编译过程中的，所以如果它的输出就是编译器的输出，如果发生错误，就会中止编译过程。以下是一个```warning```：
```plain
frameworks/opt/net/ims/src/java/com/android/ims/internal/VideoPauseTracker.java:160: warning: [SynchronizeOnNonFinalField] Synchronizing on non-final fields is not safe: if the field is ever updated, different threads may end up locking on different objects.
        synchronized (mPauseRequestsLock) {
                     ^
    (see https://errorprone.info/bugpattern/SynchronizeOnNonFinalField)
```
每条日志都有一个标签，比如```SynchronizeOnNonFinalField```，然后是简短的说明、有问题的代码片段，最后列出了线上的详细文档URL，文档内有详细讲解。

# 定制

一类问题是```error```还是```warning```是可以定制的。通过在```Android.mk```中定义```LOCAL_ERROR_PRONE_FLAGS```可以指定，以下是```tradefed```模块使用的配置：

```makefile
LOCAL_ERROR_PRONE_FLAGS:= -XDandroidCompatible=false \
                          -Xep:ArrayToString:ERROR \
                          -Xep:BoxedPrimitiveConstructor:ERROR \
                          -Xep:ConstantField:ERROR \
                          -Xep:DeadException:ERROR \
                          -Xep:EqualsIncompatibleType:ERROR \
                          -Xep:ExtendingJUnitAssert:ERROR \
                          -Xep:FormatString:ERROR \
                          -Xep:GetClassOnClass:ERROR \
                          -Xep:IdentityBinaryExpression:ERROR \
                          -Xep:JUnit3TestNotRun:ERROR \
                          -Xep:JUnit4ClassUsedInJUnit3:ERROR \
                          -Xep:JUnitAmbiguousTestClass:ERROR \
                          -Xep:MissingFail:ERROR \
                          -Xep:MissingOverride:ERROR \
                          -Xep:ModifiedButNotUsed:ERROR \
                          -Xep:MustBeClosedChecker:ERROR \
                          -Xep:Overrides:ERROR \
                          -Xep:PackageLocation:ERROR \
                          -Xep:ParameterName:ERROR \
                          -Xep:ReferenceEquality:ERROR \
                          -Xep:RemoveUnusedImports:ERROR \
                          -Xep:ReturnValueIgnored:ERROR \
                          -Xep:SelfEquals:ERROR \
                          -Xep:SizeGreaterThanOrEqualsZero:ERROR \
                          -Xep:TryFailThrowable:ERROR
```


# 性能影响

分别打开和关闭Error Prone编译framework，编译时间几乎没有任何差别。

# 和Android Lint的关系

Android Lint是Google推出的针对Android应用开发的静态检查工具，它主要针对Android应用特有的各种问题，除了源码，它还可以检查各种资源文件。Android Lint是一个单独运行的工具，分析源码而不参与编译。

Error Prone是为了发现Java开发中共性的问题，只要是Java项目都可以检查。对于设备厂商来说，主要的修改在Android框架层，这部分代码的质量是他们最关心的，对此，Error Prone是一个合适的工具。

两者几乎没有重叠的功能，可以运用在不同项目上，互为补充。

# 结论

Error Prone可以结合基于模块的CI，做快速的代码检查，至少有以下几个优点：

* 官方出品，代表Google对Java代码质量的思考，且已经与AOSP集成
* 轻量级，对CI时间影响很小
* 本身就是编译的一部分，无需额外设计检查流程
* 研发工程师提交代码前也可自行验证
