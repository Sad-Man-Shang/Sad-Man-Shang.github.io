---
title: "Markword如何存储32位hashcode"
date: 2023-09-04T16:42:56+08:00
draft: false
authors: [Sad_man]
tags: [hashcode, markword]
categories: [jvm]
series: [看看时间都浪费在哪里了]
series_weight: 1
---

说问题先说版本，**jdk8u**

众所周知，java中hashcode是32位的int类型。

在学习synchronized关键字的实现的时候，接触到了markword，在32位jvm中只有25位来存储hashcode。

机智的我立刻发现了异常

**25位存储空间如何才能存储32位的hashcode**啊？？？

难到是有什么padding操作？

我看了源码之后发现，是朴实无华的直接往markword里面存，用的之后直接取出。

```c
intptr_t hash() const {
  		// inline intptr_t mask_bits (intptr_t x, intptr_t m) { return x & m; }
  		// 这里直接返回了
      return mask_bits(value() >> hash_shift, hash_mask);
}
```

也就是说**32位虚拟机压根不存高7位**

一整个大崩溃……这与我刻板印象下的HashMap的put过程中发生的扰动运算不一致啊，那这样的话32位虚拟机环境下岂不是`h >>> 12`更好一点？

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



测试代码如下

```java
public class MyClass {
    public static void main(String args[]) {
      int minLeadingZeroes = 32;
      for (int i = 0; i < 1_000_000; i++) {
          int hash = System.identityHashCode(new Object());
          minLeadingZeroes = Math.min(minLeadingZeroes, Integer.numberOfLeadingZeros(hash));
      }

      System.out.println("Smallest number of leading zeroes in identity hash codes of 1000000 objects = " + minLeadingZeroes);
    }
}
```

