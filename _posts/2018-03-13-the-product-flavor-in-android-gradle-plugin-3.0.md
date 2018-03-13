---
title: Android Gradle Plugin 3.0 中的 productFlavors 知识
layout: post
guid: urn:uuid:f239a5e2-701e-4447-bdea-523861ca2481
comments: true
tags:
  - android-gradle-plugin
---

## productFlavors 新版配置

参考资料: [gradle-plugin-3-0-0-migration](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html#variant_aware)

如果我们在 module 的 build.gradle 指定了某个 product flavor。如下图所示：

```
productFlavors {
  // flavor name
  free {}
}
```

那么必须指定一个 flavor dimension。否则会报错 `Error:All flavors must now belong to a named flavor dimension. Learn more at https://d.android.com/r/tools/flavorDimensions-missing-error-message.html`。

根据文档修改为如下代码：

```
// freeDimension 为 dimension name
flavorDimensions "freeDimension"

productFlavors {
  // flavor name
  free {}
}
```

上面代码配置等同于下面的代码：

```
// freeDimension 为 dimension name
flavorDimensions "freeDimension"

productFlavors {
  // flavor name
  free {
    dimension "freeDimension"
  }
}
```

原因是我们只指定了一种 flavor dimension --- freeDimension。

假如 productFlavors 中包含多个 flavor，也是一样的道理，flavor 中指定的 dimension 默认为 freeDimension。

例如:

```
// freeDimension 为 dimension name
flavorDimensions "freeDimension"

productFlavors {
  // flavor name
  free {
    dimension "freeDimension"
  }
  paid {
    dimension "freeDimension"
  }
}
```

等同于

```
// freeDimension 为 dimension name
flavorDimensions "freeDimension"

productFlavors {
  // flavor name
  free {}
  paid {}
}
```
## 基本概念

看到这里，我们要明确两个概念，productFlavors 中的 flavor name 和 flavorDimensions 中的 dimension name 是两个东西，不一定要同名。

那么，它们的关系是什么呢？

productFlavors 中的某个 flavor 必须指定某个 dimension，flavorDimensions 下面的 dimension 个数决定了 productFlavors 最多可以被分为几大类。而 productFlavors 下面所有的 flavor 都需要选择“站队“。 (指定 dimension 就是在选择类型)

请往下看例子

## 例子

假设我们有 app module 和 library module，app module 依赖 library module。

### 配置一

app module 中的 build.gradle 配置为：

```
android {
  // ... 省略

  buildTypes {
    release {
      minifyEnabled false
    }
  }

  flavorDimensions "freeDimension"

  productFlavors {
    free {}
    paid {}
  }
}

dependencies {
  implementation project(':library')
}
```

library module 中的 build.gradle 配置为：

```
android {
  // ... 省略
  
  buildTypes {
    release {
      minifyEnabled false
    }
  }
}
```

最后看下 Build Variants 中的配置：

![](http://ozqz8ena7.bkt.clouddn.com/image/as3.0_productflavor/buildVariant_single_dimension_app.jpg)
![](http://ozqz8ena7.bkt.clouddn.com/image/as3.0_productflavor/buildVariant_single_dimension_library.jpg)

结果显示：当 app module 中所有 product flavor 都属于同一种 flavor dimension 时，其行为和之前使用 Android Plugin 2.3 系列基本一致。

### 配置二

app module 中的 build.gradle 配置变为：

```
flavorDimensions "freeDimension", "paidDimension"

productFlavors {
  free {
    dimension "freeDimension"
  }
  paid {
    dimension "paidDimension"
  }
}
```

library module 中的 build.gradle 和配置一相同。

最后看下 Build Variants 中的配置：

![](http://ozqz8ena7.bkt.clouddn.com/image/as3.0_productflavor/buildVariant_two_dimension_app_two_flavor.jpg)

library module 的配置没有变话，不贴图了。

结果显示：free 和 paid 两个 flavor 由于属于两种 flavor dimension 时，这两个 flavor 会同时编译。请看下图：

![](http://ozqz8ena7.bkt.clouddn.com/image/as3.0_productflavor/buildVariant_two_dimension_app_two_flavor_source_compile.jpg)

如果 app module 种包含三个 flavor 而且分别属于三种 flavor dimension 时，大家猜猜是怎么样？

结果如下：

![](http://ozqz8ena7.bkt.clouddn.com/image/as3.0_productflavor/buildVariant_three_dimension_app_three_flavor.jpg)
![](http://ozqz8ena7.bkt.clouddn.com/image/as3.0_productflavor/buildVariant_three_dimension_app_three_flavor_source_compile.jpg)

### 配置三

app module 中的 build.gradle 配置为：

```
flavorDimensions "freeDimension", "paidDimension"

productFlavors {
  free {
    dimension "freeDimension"
  }
  paid {
    dimension "freeDimension"
  }
  discount {
    dimension "paidDimension"
  }
}
```

结果如下：

![](http://ozqz8ena7.bkt.clouddn.com/image/as3.0_productflavor/buildVariant_two_dimension_app_three_flavor_with_two_dimension.jpg)

假如 paidDimension 下面增加一个 flavor 会如何呢？

```
flavorDimensions "freeDimension", "paidDimension"

productFlavors {
  free {
    dimension "freeDimension"
  }
  paid {
    dimension "freeDimension"
  }
  discount {
    dimension "paidDimension"
  }
  limited {
    dimension "paidDimension"
  }
}
```

结果如下：

![](http://ozqz8ena7.bkt.clouddn.com/image/as3.0_productflavor/buildVariant_two_dimension_app_four_flavor_with_two_dimension.jpg)

## 结论

productFlavors 中的 flavor 会根据 flavor dimension 类型分类，然后每个类型下面的 flavor 再分别组合形成新的 product flavor，再与 build type 进行组合，最后构成 Build Variants。

新的 product flavor 个数怎么计算呢？

假设 productFlavors 中所有的 flavor 根据 flavor dimension 分为 N 类(N >= 1)，每个类型下面的 flavor 个数为 M1，M2，M3 ... M++，新的 product flavor 个数为 F，那么有如下公式：

```
F = M1 * M2 * M3 * ... * M++
```

而且，无论 productFlavors 中有几种 flavor dimension 类型，都会在每个类型下选取 flavor 之后同时编译代码。

至此，我们弄清楚了单个 module 中 productFlavors 的配置规律。

## 代码示例

[TestAndroidPlugin-3_0_0-Demo](https://github.com/johnwatsondev/TestAndroidPlugin-3_0_0-Demo)

## 小提示

我们不可以声明多余的 flavor Dimension，否则会报错。

假如配置如下：

```
flavorDimensions "freeDimension", "paidDimension", "discountDimension"

productFlavors {
  free {
    dimension "freeDimension"
  }
  paid {
    dimension "freeDimension"
  }
  discount {
    dimension "discountDimension"
  }
}
```

报错如下：

`Error:No flavor is associated with flavor dimension 'paidDimension'.`

配置中只使用了 freeDimension 和 discountDimension，所以会报错。
