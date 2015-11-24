---
title: 混淆带泛型的 Gson 工具类
layout: post
guid: urn:uuid:c04b47e9-4192-47d5-ae0d-1e64285ab2f4
comments: false
tags:
  - gson
---

### 场景
本着 DRY 原则抽取 Gson 的序列化和反序列化方法，结果被打脸。。。先来看代码：

```java
public static <T> String convertToString(T src) {
  return new Gson().toJson(src, new TypeToken<T>() {
  }.getType());
}
```

上面的代码在为混淆前运行正常，混淆后抛出错误：

```java
java.lang.AssertionError: illegal type variable reference
  at libcore.reflect.TypeVariableImpl.resolve(TypeVariableImpl.java:110)
  at libcore.reflect.TypeVariableImpl.getGenericDeclaration(TypeVariableImpl.java:124)
  at libcore.reflect.TypeVariableImpl.hashCode(TypeVariableImpl.java:47)
  at com.google.gson.reflect.TypeToken.<init>(SourceFile:64)
```

变量类型引用不合法错误。一翻查资料后，有网友指出，这个泛型是在运行时期是无效的，只有在编译时期才有效。那为什么在混淆之前不报错呢？所以对这个持有疑问。  

### 问题分析
写的匆忙，没有认真分析。无法解答。

### 参考
[Gson User Guide](https://sites.google.com/site/gson/gson-user-guide)  
[Using generics with GSON](http://stackoverflow.com/questions/4226738/using-generics-with-gson)  
[Using a generic type with Gson](http://stackoverflow.com/questions/5370768/using-a-generic-type-with-gson)  
[Using GSON with proguard enabled](http://stackoverflow.com/questions/31844352/using-gson-with-proguard-enabled)
