# 不可变集合

```java
public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
  "red",
  "orange",
  "yellow",
  "green",
  "blue",
  "purple");

class Foo {
  final ImmutableSet<Bar> bars;
  Foo(Set<Bar> bars) {
    this.bars = ImmutableSet.copyOf(bars); // defensive copy!
  }
}
```

# 为什么要使用不可变集合

不可变对象有很多优点，包括：

* 安全使用不受信任的libraries。

* 线程安全：不可变对象被多个线程调用时，不存在竞态条件问题

* 不需要支持突变，并可以节省时间和空间。 所有不可变的集合都比它们的可变形式有更好的内存利用率（分析和测试细节）

* 因为有固定不变，可以作为常量来安全使用。

创建对象的不可变拷贝是一项很好的防御性编程技巧。Guava 为所有 JDK 标准集合类型和 Guava 新集合类型都提供了简单易用的不可变版本。

JDK提供了`Collections.unmodifiableXXX`方法，但是在我们看来，这些可以：

* 笨重而且累赘：不能舒适地用在所有想做防御性拷贝的场景

* 不安全：要保证没人通过原集合的引用进行修改，返回的集合才是事实上不可变的

* 效率低：数据结构仍然具有可变集合的所有开销，包括并发修改检查，散列表中的额外空间等。

如果你没有修改某个集合的需求，或者希望某个集合保持不变时，把它防御性地拷贝到不可变集合是个很好的实践。



**重要提示**：所有 Guava 不可变集合的实现都不接受 null 值。我们对 Google 内部的代码库做过详细研究，发现只有 5%的情况需要在集合中允许 `null`元素，剩下的 95%场景都是遇到 null 值就快速失败。如果你需要在不可变集合中使用 null，请使用 JDK 中的 `Collections.unmodifiableXXX`方法。更多细节建议请参考“使用和避免null”。

# 怎么使用不可变集合

不可变集合可以用如下多种方式创建：

* `copyOf` 方法，如 `ImmutableSet.copyOf(set)`;

* `of`方法，如 `ImmutableSet.of(“a”, “b”, “c”)`或 `ImmutableMap.of(“a”, 1, “b”, 2)`;

* `Builder`工具，如

```
public static final ImmutableSet<Color> GOOGLE_COLORS =
       ImmutableSet.<Color>builder()
           .addAll(WEBSAFE_COLORS)
           .add(new Color(0, 191, 255))
           .build();
```

此外，对有序不可变集合来说，排序是在构造集合的时候完成的，如：

```
ImmutableSortedSet.of("a", "b", "c", "a", "d", "b");
```

会在构造时就把元素排序为 a, b, c, d。

## 比想象中更智能的 `copyOf`

请注意，`ImmutableXXX.copyOf` 方法会尝试在安全的时候避免做拷贝——实际的实现细节不详，但通常来说是很智能的，比如：

```
ImmutableSet<String> foobar = ImmutableSet.of("foo", "bar", "baz");
thingamajig(foobar);

void thingamajig(Collection<String> collection) {
   ImmutableList<String> defensiveCopy = ImmutableList.copyOf(collection);
   ...
}
```

在这段代码中，`ImmutableList.copyOf(foobar)`会智能地直接返回`foobar.asList()`,它是一个 `ImmutableSet`的常量时间复杂度的 List 视图。

作为一种探索，`ImmutableXXX.copyOf(ImmutableCollection)`会试图对如下情况避免线性时间拷贝：

* 在常量时间内使用底层数据结构是可能的——例如，ImmutableSet.copyOf\(ImmutableList\)就不能在常量时间内完成。

* 不会造成内存泄露——例如，你有个很大的不可变集合 `ImmutableList <String> hugeList`， `ImmutableList.copyO  
  f(hugeList.subList(0, 10))`就会显式地拷贝，以免不必要地持有 `hugeList`的引用。

* 不改变语义——所以 `ImmutableSet.copyOf(myImmutableSortedSet)`会显式地拷贝，因为和基于比较器的 `ImmutableSortedSet`相比，`ImmutableSet`对`hashCode()`和 `equals`有不同语义。

在可能的情况下避免线性拷贝，可以最大限度地减少防御性编程风格所带来的性能开销。

## asList视图

所有不可变集合都有一个`asList()`方法提供 `ImmutableList` 视图，来帮助你用列表形式方便地读取集合元素。例如，你可以使用 `sortedSet.asList().get(k)`从 `ImmutableSortedSet`中读取第 `k` 个最小元素。

`asList()`返回的 `ImmutableList`通常是——并不总是——开销稳定的视图实现，而不是简单地把元素拷贝进 `List`。也就是说，asList 返回的列表视图通常比一般的列表平均性能更好，比如，在底层集合支持的情况下，它总是使用高效的 `contains`方法。

## 细节：关联可变集合和不可变集合

| Interface | JDK or Guava? | Immutable Version |
| :---: | :---: | :---: |
| Collection | JDK | [`ImmutableCollection`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableCollection.html) |
| List | JDK | [`ImmutableList`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableList.html) |
| Set | JDK | [`ImmutableSet`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSet.html) |
| SortedSet/NavigableSet | JDK | [`ImmutableSortedSet`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSortedSet.html) |
| Map | JDK | [`ImmutableMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMap.html) |
| SortedMap | JDK | [`ImmutableSortedMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSortedMap.html) |
| [Multiset](https://github.com/google/guava/wiki/NewCollectionTypesExplained#multiset) | Guava | [`ImmutableMultiset`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultiset.html) |
| SortedMultiset | Guava | [`ImmutableSortedMultiset`](http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/ImmutableSortedMultiset.html) |
| [Multimap](https://github.com/google/guava/wiki/NewCollectionTypesExplained#multimap) | Guava | [`ImmutableMultimap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultimap.html) |
| ListMultimap | Guava | [`ImmutableListMultimap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableListMultimap.html) |
| SetMultimap | Guava | [`ImmutableSetMultimap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSetMultimap.html) |
| [BiMap](https://github.com/google/guava/wiki/NewCollectionTypesExplained#bimap) | Guava | [`ImmutableBiMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableBiMap.html) |
| [ClassToInstanceMap](https://github.com/google/guava/wiki/NewCollectionTypesExplained#classtoinstancemap) | Guava | [`ImmutableClassToInstanceMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableClassToInstanceMap.html) |
| [Table](https://github.com/google/guava/wiki/NewCollectionTypesExplained#table) | Guava | [`ImmutableTable`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html) |
|  |  |  |



