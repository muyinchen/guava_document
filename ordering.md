## OrderingExplained 排序说明

### 示例

```java
assertTrue(byLengthOrdering.reverse().isOrdered(list));
```

### 概述

排序器`Ordering`是 Guava 流畅风格比较器`Comparator`的实现，它可以用来为构建复杂的比较器，以完成集

合排序的功能。

从实现上说，`Ordering`实例就是一个特殊的 `Comparator`实例。Ordering 把很多基于 `Comparator`的静态方

法（如 `Collections.max`）包装为自己的实例方法（非静态方法），并且提供了链式调用方法，来定制和增强现

有的比较器。

### 创建排序器

常见的排序器可以由下面的静态方法创建

| Method                                   | Description                            |
| :--------------------------------------- | :------------------------------------- |
| [`natural()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#natural()\) | 对可排序类型做自然排序，如数字按大小，日期按先后排序             |
| [`usingToString()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#usingToString()\) | 按对象的字符串形式做字典排序，把给定的 `Comparator`转化为排序器 |

使用  `Ordering.from(Comparator)`  传入一个Comparator 到Ordering非常简单  
实现自定义的排序器时，除了用上面的 from 方法，也可以跳过实现 Comparator，而直接继承 Ordering：

```
Ordering<String> byLengthOrdering = new Ordering<String>() {
  public int compare(String left, String right) {
    return Ints.compare(left.length(), right.length());
  }
};
```

### 链式调用方法

通过链式调用，可以由给定的排序器衍生出其它排序器， 最常见的方法包括

|                                          | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| [`reverse()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#reverse()\) | 获取语义相反的排序器                               |
| [`nullsFirst()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#nullsFirst()\) | 使用当前排序器，但额外把 null 值排到最前面， 其它特性和原始的排序器表现相同，还可以查阅 [nullLast\(\)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#nullsLast()\) |
| [`compound(Comparator)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#compound(java.util.Comparator)\) | 合成另一个比较器，以处理当前排序器中的相等情况。                 |
| [`lexicographical()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#lexicographical()\) | 基于处理类型 T 的排序器，返回该类型的可迭代对象排序器             |
| [`onResultOf(Function)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#onResultOf(com.google.common.base.Function)\) | 对集合中元素调用 Function，再按返回值用当前排序器排序          |

例如，你需要下面这个类的排序器。

```
class Foo {
  @Nullable String sortedBy;
  int notSortedBy;
}
```

考虑到排序器应该能处理 sortedBy 为 null 的情况，我们可以使用下面的链式调用来合成排序器：

```
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(new Function<Foo, String>() {
  public String apply(Foo foo) {
    return foo.sortedBy;
  }
});
```

当阅读链式调用产生的排序器时，应该从后往前读。上面的例子中，排序器首先调用 apply 方法获取 `sortedBy`

值，并把 `sortedBy`为 `null` 的元素都放到最前面，然后把剩下的元素按 `sortedBy` 进行自然排序。之所以要从后

往前读，是因为每次链式调用都是用后面的方法包装了前面的排序器。

注：用 `compound`方法包装排序器时，就不应遵循从后往前读的原则。为了避免理解上的混乱，请不要把 `com`

`pound` 写在一长串链式调用的中间，你可以另起一行，在链中最先或最后调用 `compound`。

超过一定长度的链式调用，也可能会带来阅读和理解上的难度。我们建议按下面的代码这样，在一个链中最多使

用三个方法。此外，你也可以把 Function 分离成中间对象，让链式调用更简洁紧凑。

```
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(sortKeyFunction);
```

### 运用排序器

Guava 的排序器实现有若干操纵或检查集合或元素值的方法， 我们列出了最常用的一些：

| Method                                   | Description                              | See also                                 |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| [`greatestOf(Iterable iterable, int k)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#greatestOf(java.lang.Iterable,%20int)\) | 根据排序器，获取可迭代对象中最大的k个元素。排序从大到小且是不稳定排序      | [`leastOf`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#leastOf(java.lang.Iterable,%20int)\) |
| [`isOrdered(Iterable)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#isOrdered(java.lang.Iterable)\) | 判断可迭代对象是否已按排序器进行非递减式排序， 允许有值相等           | [`isStrictlyOrdered`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#isStrictlyOrdered(java.lang.Iterable)\) 不允许有值相等 |
| [`sortedCopy(Iterable)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#sortedCopy(java.lang.Iterable)\) | Returns a sorted copy of the specified elements as a`List`.返回一个List，已排序元素的拷贝 | [`immutableSortedCopy`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#immutableSortedCopy(java.lang.Iterable)\) |
| [`min(E, E)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#min(E,%20E)\) | 根据排序器返回两个参数中最小的那个。如果相等，则返回第一个元素          | [`max(E, E)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#max(E,%20E)\) |
| [`min(E, E, E, E...)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#min(E,%20E,%20E,%20E...)\) | Returns the minimum of its arguments according to this ordering. If there are multiple least values, the first is returned. 根据排序器返回多个参数的最小值， 如果有多个最小值 | [`max(E, E, E, E...)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#max(E,%20E,%20E,%20E...)\) |
| [`min(Iterable)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#min(java.lang.Iterable)\) | 返回迭代器中最小的元素。如果可迭代对象中没有元素，则抛出 NoSuchElementException | [`max(Iterable)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#max(java.lang.Iterable)\),[`min(Iterator)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#min(java.util.Iterator)\),[`max(Iterator)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#max(java.util.Iterator)\) |



