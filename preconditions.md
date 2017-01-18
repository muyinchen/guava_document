## Preconditions

Guava 在 Preconditions 类中提供了若干前置条件判断的实用方法，我们强烈建议在 Eclipse 中静态导入这些
方法。每个方法都有三个变种：



-  没有额外参数：抛出的异常中没有错误消息；
-  有一个` Object` 对象作为额外参数：抛出的异常使用 `Object.toString() `作为错误消息；
-  有一个 `String` 对象作为额外参数，并且有一组任意数量的附加 `Object` 对象：这个变种处理异常消息的方式

有点类似 printf，但考虑 GWT 的兼容性和效率，只支持%s 指示符。例如：

```
checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
checkArgument(i < j, "Expected i < j, but %s > %s", i, j);
```

| 方法签名(不包括额外参数)                            | 描述                                       | 检查失败时抛出的异常                             |
| :--------------------------------------- | :--------------------------------------- | :------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Preconditions.html#checkArgument(boolean)'><code>checkArgument(boolean)</code></a> | 检查 boolean 是否为 true，用来检查传递给方法的参数         | `IllegalArgumentException`             |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Preconditions.html#checkNotNull(T)'><code>checkNotNull(T)</code></a> | 检查 value 是否为 `null `  ，该方法直接返回 value，因此可以内嵌使用 | `NullPointerException`                 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Preconditions.html#checkState(boolean)'><code>checkState(boolean)</code></a> | 检查一些对象的状态，不依赖于方法的参数。例如， 一个`Iterator`可能使用它来检查`next` 已经在任何`remove`之前被调用 | `IllegalStateException`                |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Preconditions.html#checkElementIndex(int, int)'><code>checkElementIndex(int index, int size)</code></a> | 检查索引是否是一个有着明确大小的list，String 或着数组的有效的索引， 一个元素的索引范围[0,`size`)。你不用直接的传递list, string, 或着 array， 你仅仅传递它的大小。返回索引。 | `IndexOutOfBoundsException `           |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Preconditions.html#checkPositionIndex(int, int)'><code>checkPositionIndex(int index, int size)</code></a> | 检查索引是否是一个有着明确大小的list，String 或着数组的有效的索引， 一个元素的索引范围[0,`size`]。你不用直接的传递list, string, 或着 array， 你仅仅传递它的大小。返回索引。 | <code>IndexOutOfBoundsException</code> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Preconditions.html#checkPositionIndexes(int, int, int)'><code>checkPositionIndexes(int start, int end, int size)</code></a> | 检查[start, end) 是否是一个有明确大小的list， String 或 Array 的有效的子区间， 错误消息来自本身 | <code>IndexOutOfBoundsException</code> |

相比 Apache Commons 提供的类似方法，我们把 Guava 中的` Preconditions `作为首选。Piotr Jagielski 在
他的博客中简要地列举了一些理由：

- 在静态导入后，Guava 方法非常清楚明晰。checkNot`null `   清楚地描述做了什么，会抛出什么异常；

- `checkNotnull `   直接返回检查的参数，让你可以在构造函数中保持字段的单行赋值风格：`this.field = checkNotnull  (field)`

- 简单的、参数可变的 printf 风格异常信息。鉴于这个优点，在 JDK7 已经引入 `Objects.requireNonnull `   的

  情况下，我们仍然建议你使用 `checkNotnull `  。

  在编码时，如果某个值有多重的前置条件，我们建议你把它们放到不同的行，这样有助于在调试时定位。此

  外，把每个前置条件放到不同的行，也可以帮助你编写清晰和有用的错误消息

