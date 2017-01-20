TODO: Queues, Tables

Any programmer with experience with the JDK Collections Framework knows and loves the utilities available in <a href='http://docs.oracle.com/javase/7/docs/api/java/util/Collections.html'><code>java.util.Collections</code></a>.  Guava provides many more utilities along these lines: static methods applicable to all collections.  These are among the most popular and mature parts of Guava.

Methods corresponding to a particular interface are grouped in a relatively intuitive manner:

| Interface    | JDK or Guava?                          | Corresponding Guava utility class        |
| :----------- | :------------------------------------- | :--------------------------------------- |
| `Collection` | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Collections2.html'><code>Collections2</code></a> (avoiding conflict with `java.util.Collections`) |
| `List`       | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html'><code>Lists</code></a> |
| `Set`        | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html'><code>Sets</code></a> |
| `SortedSet`  | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html'><code>Sets</code></a> |
| `Map`        | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html'><code>Maps</code></a> |
| `SortedMap`  | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html'><code>Maps</code></a> |
| `Queue`      | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Queues.html'><code>Queues</code></a> |
| [[Multiset   | NewCollectionTypesExplained#multiset]] | Guava                                    |
| [[Multimap   | NewCollectionTypesExplained#multimap]] | Guava                                    |
| [[BiMap      | NewCollectionTypesExplained#bimap]]    | Guava                                    |
| [[Table      | NewCollectionTypesExplained#table]]    | Guava                                    |

**_Looking for transform, filter, and the like?  That stuff is [[here|FunctionalExplained]], under functional idioms._**

# Static constructors
Before JDK 7, constructing new generic collections requires unpleasant code duplication:
```java

List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<TypeThatsTooLongForItsOwnGood>();
```
I think we can all agree that this is unpleasant.  Guava provides static methods that use generics to infer the type on the right side:
```java

List<TypeThatsTooLongForItsOwnGood> list = Lists.newArrayList();
Map<KeyType, LongishValueType> map = Maps.newLinkedHashMap();
```

To be sure, the diamond operator in JDK 7 makes this less of a hassle:

```java

List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<>();
```

But Guava goes further than this.  With the factory method pattern, we can initialize collections with their starting elements very conveniently.

```java

Set<Type> copySet = Sets.newHashSet(elements);
List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");
```

Additionally, with the ability to name factory methods (Effective Java item 1), we can improve the readability of initializing collections to sizes:

```java

List<Type> exactly100 = Lists.newArrayListWithCapacity(100);
List<Type> approx100 = Lists.newArrayListWithExpectedSize(100);
Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(100);
```

The precise static factory methods provided are listed with their corresponding utility classes below.

_Note:_ New collection types introduced by Guava don't expose raw constructors, or have initializers in the utility classes.  Instead, they expose static factory methods directly, for example:

```java

Multiset<String> multiset = HashMultiset.create();
```

# Iterables
Whenever possible, Guava prefers to provide utilities accepting an `Iterable` rather than a `Collection`.  Here at Google, it's not out of the ordinary to encounter a "collection" that isn't actually stored in main memory, but is being gathered from a database, or from another data center, and can't support operations like `size()` without actually grabbing all of the elements.

As a result, many of the operations you might expect to see supported for all collections can be found in <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html'><code>Iterables</code></a>.  Additionally, most `Iterables` methods have a corresponding version in <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterators.html'><code>Iterators</code></a> that accepts the raw iterator.

The overwhelming majority of operations in the `Iterables` class are _lazy_: they only advance the backing iteration when absolutely necessary.  Methods that themselves return `Iterables` return lazily computed views, rather than explicitly constructing a collection in memory.

As of Guava 12, `Iterables` is supplemented by the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/FluentIterable.html'><code>FluentIterable</code></a> class, which wraps an `Iterable` and provides a "fluent" syntax for many of these operations.

The following is a selection of the most commonly used utilities, although many of the more "functional" methods in `Iterables` are discussed in [[Guava functional idioms|FunctionalExplained]].

### General
| Method                                   | Description                              | See Also                                 |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#concat(java.lang.Iterable)'><code>concat(Iterable&lt;Iterable&gt;)</code></a> | Returns a lazy view of the concatenation of several iterables. | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#concat(java.lang.Iterable...)'><code>concat(Iterable...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#frequency(java.lang.Iterable, java.lang.Object)'><code>frequency(Iterable, Object)</code></a> | Returns the number of occurrences of the object. | Compare `Collections.frequency(Collection, Object)`; see [[Multiset |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#partition(java.lang.Iterable, int)'><code>partition(Iterable, int)</code></a> | Returns an unmodifiable view of the iterable partitioned into chunks of the specified size. | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#partition(java.util.List, int)'><code>Lists.partition(List, int)</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#paddedPartition(java.lang.Iterable, int)'><code>paddedPartition(Iterable, int)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getFirst(java.lang.Iterable, T)'><code>getFirst(Iterable, T default)</code></a> | Returns the first element of the iterable, or the default value if empty. | Compare `Iterable.iterator().next()`<br><a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#first()'><code>FluentIterable.first()</code></a> <br>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getLast(java.lang.Iterable)'><code>getLast(Iterable)</code></a> </td><td> Returns the last element of the iterable, or fails fast with a <code>NoSuchElementException</code> if it's empty.  </td><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getLast(java.lang.Iterable, T)'><code>getLast(Iterable, T default)</code></a><br><a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#last()'><code>FluentIterable.last()</code></a> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#elementsEqual(java.lang.Iterable, java.lang.Iterable)'><code>elementsEqual(Iterable, Iterable)</code></a> </td><td> Returns true if the iterables have the same elements in the same order. </td><td> Compare <code>List.equals(Object)</code> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#unmodifiableIterable(java.lang.Iterable)'><code>unmodifiableIterable(Iterable)</code></a> </td><td> Returns an unmodifiable view of the iterable. </td><td> Compare <code>Collections.unmodifiableCollection(Collection)</code> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#limit(java.lang.Iterable, int)'><code>limit(Iterable, int)</code></a> </td><td> Returns an <code>Iterable</code> returning at most the specified number of elements. </td><td> <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#limit(int)'><code>FluentIterable.limit(int)</code></a> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getOnlyElement(java.lang.Iterable)'><code>getOnlyElement(Iterable)</code></a> </td><td> Returns the only element in <code>Iterable</code>.  Fails fast if the iterable is empty or has multiple elements.  </td><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getOnlyElement(java.lang.Iterable, T)'><code>getOnlyElement(Iterable, T default)</code></a> </td></tr></tbody></table>

```java

Iterable<Integer> concatenated = Iterables.concat(
  Ints.asList(1, 2, 3),
  Ints.asList(4, 5, 6));
// concatenated has elements 1, 2, 3, 4, 5, 6

String lastAdded = Iterables.getLast(myLinkedHashSet);

String theElement = Iterables.getOnlyElement(thisSetIsDefinitelyASingleton);
  // if this set isn't a singleton, something is wrong!
```

### Collection-Like
Typically, collections support these operations naturally on other collections, but not on iterables.

_Each of these operations delegates to the corresponding `Collection` interface method when the input is actually a `Collection`._  For example, if `Iterables.size` is passed a `Collection`, it will call the `Collection.size` method instead of walking through the iterator.

| Method                                   | Analogous `Collection` method      | `FluentIterable` equivalent              |
| :--------------------------------------- | :--------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#addAll(java.util.Collection, java.lang.Iterable)'><code>addAll(Collection addTo, Iterable toAdd)</code></a> | `Collection.addAll(Collection)`    |                                          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#contains(java.lang.Iterable, java.lang.Object)'><code>contains(Iterable, Object)</code></a> | `Collection.contains(Object)`      | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#contains(java.lang.Object)'><code>FluentIterable.contains(Object)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#removeAll(java.lang.Iterable, java.util.Collection)'><code>removeAll(Iterable removeFrom, Collection toRemove)</code></a> | `Collection.removeAll(Collection)` |                                          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#retainAll(java.lang.Iterable, java.util.Collection)'><code>retainAll(Iterable removeFrom, Collection toRetain)</code></a> | `Collection.retainAll(Collection)` |                                          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#size(java.lang.Iterable)'><code>size(Iterable)</code></a> | `Collection.size()`                | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#size()'><code>FluentIterable.size()</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#toArray(java.lang.Iterable, java.lang.Class)'><code>toArray(Iterable, Class)</code></a> | `Collection.toArray(T[])`          | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toArray(java.lang.Class)'><code>FluentIterable.toArray(Class)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#isEmpty(java.lang.Iterable)'><code>isEmpty(Iterable)</code></a> | `Collection.isEmpty()`             | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#isEmpty()'><code>FluentIterable.isEmpty()</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#get(java.lang.Iterable, int)'><code>get(Iterable, int)</code></a> | `List.get(int)`                    | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#get(int)'><code>FluentIterable.get(int)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#toString(java.lang.Iterable)'><code>toString(Iterable)</code></a> | `Collection.toString()`            | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toString()'><code>FluentIterable.toString()</code></a> |

### `FluentIterable`
Besides the methods covered above and in the functional idioms [[article|FunctionalExplained]], `FluentIterable` has a few convenient methods for copying into an immutable collection:

| `ImmutableList` | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toImmutableList()>`toImmutableList()`</a'> <br>
<tr><td> <code>ImmutableSet</code> </td><td> <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toImmutableSet()'><code>toImmutableSet()</code></a> </td></tr>
<tr><td> <code>ImmutableSortedSet</code> </td><td> <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toImmutableSortedSet(java.util.Comparator)'><code>toImmutableSortedSet(Comparator)</code></a> </td></tr></tbody></table>

### Lists
In addition to static constructor methods and functional programming methods, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html'><code>Lists</code></a> provides a number of valuable utility methods on `List` objects.

| Method                                   | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#partition(java.util.List, int)'><code>partition(List, int)</code></a> | Returns a view of the underlying list, partitioned into chunks of the specified size. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#reverse(java.util.List)'><code>reverse(List)</code></a> | Returns a reversed view of the specified list.  _Note_: if the list is immutable, consider <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableList.html#reverse()'><code>ImmutableList.reverse()</code></a> instead. |

```java

List<Integer> countUp = Ints.asList(1, 2, 3, 4, 5);
List<Integer> countDown = Lists.reverse(theList); // {5, 4, 3, 2, 1}

List<List<Integer>> parts = Lists.partition(countUp, 2); // {{1, 2}, {3, 4}, {5}}
```

### Static Factories
`Lists` provides the following static factory methods:

| Implementation | Factories                                |
| :------------- | :--------------------------------------- |
| `ArrayList`    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayList()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayList(E...)'>with elements</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayList(java.lang.Iterable)'>from <code>Iterable</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayListWithCapacity(int)'>with exact capacity</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayListWithExpectedSize(int)'>with expected size</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayList(java.util.Iterator)'>from <code>Iterator</code></a> |
| `LinkedList`   | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newLinkedList()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newLinkedList(java.lang.Iterable)'>from <code>Iterable</code></a> |

# Sets
The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html'><code>Sets</code></a> utility class includes a number of spicy methods.

### Set-Theoretic Operations
We provide a number of standard set-theoretic operations, implemented as views over the argument sets.  These return a <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.SetView.html'><code>SetView</code></a>, which can be used:
* as a `Set` directly, since it implements the `Set` interface
* by copying it into another mutable collection with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.SetView.html#copyInto(S)'><code>copyInto(Set)</code></a> 
    * by making an immutable copy with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.SetView.html#immutableCopy()'><code>immutableCopy()</code></a>

| Method                                   |
| :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#union(java.util.Set, java.util.Set)'><code>union(Set, Set)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#intersection(java.util.Set, java.util.Set)'><code>intersection(Set, Set)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#difference(java.util.Set, java.util.Set)'><code>difference(Set, Set)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#symmetricDifference(java.util.Set, java.util.Set)'><code>symmetricDifference(Set, Set)</code></a> |

For example:

```java

Set<String> wordsWithPrimeLength = ImmutableSet.of("one", "two", "three", "six", "seven", "eight");
Set<String> primes = ImmutableSet.of("two", "three", "five", "seven");

SetView<String> intersection = Sets.intersection(primes, wordsWithPrimeLength); // contains "two", "three", "seven"
// I can use intersection as a Set directly, but copying it can be more efficient if I use it a lot.
return intersection.immutableCopy();
```

### Other Set Utilities
| Method                                   | Description                              | See Also                                 |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#cartesianProduct(java.util.List)'><code>cartesianProduct(List&lt;Set&gt;)</code></a> | Returns every possible list that can be obtained by choosing one element from each set. | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#cartesianProduct(java.util.Set...)'><code>cartesianProduct(Set...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#powerSet(java.util.Set)'><code>powerSet(Set)</code></a> | Returns the set of subsets of the specified set. |                                          |

```java

Set<String> animals = ImmutableSet.of("gerbil", "hamster");
Set<String> fruits = ImmutableSet.of("apple", "orange", "banana");

Set<List<String>> product = Sets.cartesianProduct(animals, fruits);
// {{"gerbil", "apple"}, {"gerbil", "orange"}, {"gerbil", "banana"},
//  {"hamster", "apple"}, {"hamster", "orange"}, {"hamster", "banana"}}

Set<Set<String>> animalSets = Sets.powerSet(animals);
// {{}, {"gerbil"}, {"hamster"}, {"gerbil", "hamster"}}
```

### Static Factories
`Sets` provides the following static factory methods:

| Implementation  | Factories                                |
| :-------------- | :--------------------------------------- |
| `HashSet`       | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSet()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSet(E...)'>with elements</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSet(java.lang.Iterable)'>from <code>Iterable</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSetWithExpectedSize(int)'>with expected size</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSet(java.util.Iterator)'>from <code>Iterator</code></a> |
| `LinkedHashSet` | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newLinkedHashSet()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newLinkedHashSet(java.lang.Iterable)'>from <code>Iterable</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newLinkedHashSetWithExpectedSize(int)'>with expected size</a> |
| `TreeSet`       | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newTreeSet()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newTreeSet(java.util.Comparator)'>with <code>Comparator</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newTreeSet(java.lang.Iterable)'>from <code>Iterable</code></a> |

# Maps
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html'><code>Maps</code></a> has a number of cool utilities that deserve individual explanation.

### `uniqueIndex`
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#uniqueIndex(java.lang.Iterable, com.google.common.base.Function)'><code>Maps.uniqueIndex(Iterable, Function)</code></a> addresses the common case of having a bunch of objects that each have some unique attribute, and wanting to be able to look up those objects based on that attribute.

Let's say we have a bunch of strings that we know have unique lengths, and we want to be able to look up the string with some particular length.

```java

ImmutableMap<Integer, String> stringsByIndex = Maps.uniqueIndex(strings, new Function<String, Integer> () {
    public Integer apply(String string) {
      return string.length();
    }
  });
```

If indices are _not_ unique, see `Multimaps.index` below.

### `difference`
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#difference(java.util.Map, java.util.Map)'><code>Maps.difference(Map, Map)</code></a> allows you to compare all the differences between two maps.  It returns a `MapDifference` object, which breaks down the Venn diagram into:

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesInCommon()'><code>entriesInCommon()</code></a> | The entries which are in both maps, with both matching keys and values. |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesDiffering()'><code>entriesDiffering()</code></a> | The entries with the same keys, but differing values.  The values in this map are of type <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.ValueDifference.html'><code>MapDifference.ValueDifference</code></a>, which lets you look at the left and right values. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesOnlyOnLeft()'><code>entriesOnlyOnLeft()</code></a> | Returns the entries whose keys are in the left but not in the right map. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesOnlyOnRight()'><code>entriesOnlyOnRight()</code></a> | Returns the entries whose keys are in the right but not in the left map. |

```java

Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
Map<String, Integer> right = ImmutableMap.of("b", 2, "c", 4, "d", 5);
MapDifference<String, Integer> diff = Maps.difference(left, right);

diff.entriesInCommon(); // {"b" => 2}
diff.entriesDiffering(); // {"c" => (3, 4)}
diff.entriesOnlyOnLeft(); // {"a" => 1}
diff.entriesOnlyOnRight(); // {"d" => 5}
```

### `BiMap` utilities
The Guava utilities on `BiMap` live in the `Maps` class, since a `BiMap`
is also a `Map`.

| `BiMap` utility | Corresponding `Map` utility |
|:----------------|:----------------------------|
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#synchronizedBiMap(com.google.common.collect.BiMap)'><code>synchronizedBiMap(BiMap)</code> <table><thead><th> <code>Collections.synchronizedMap(Map)</code></a> </th></thead><tbody>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#unmodifiableBiMap(com.google.common.collect.BiMap)'><code>unmodifiableBiMap(BiMap)</code> </td><td> <code>Collections.unmodifiableMap(Map)</code></a> </td></tr></tbody></table>

### Static Factories
`Maps` provides the following static factory methods.

| Implementation                           | Factories                                |
| :--------------------------------------- | :--------------------------------------- |
| `HashMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMap(java.util.Map)'>from <code>Map</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMapWithExpectedSize(int)'>with expected size</a> |
| `LinkedHashMap`                          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newLinkedHashMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newLinkedHashMap(java.util.Map)'>from <code>Map</code></a> |
| `TreeMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap(java.util.Comparator)'>from <code>Comparator</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap(java.util.SortedMap)'>from <code>SortedMap</code></a> |
| `EnumMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newEnumMap(java.lang.Class)'>from <code>Class</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newEnumMap(java.util.Map)'>from <code>Map</code></a> |
| `ConcurrentMap` (supporting all operations) | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newConcurrentMap()'>basic</a> |
| `IdentityHashMap`                        | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newIdentityHashMap()'>basic</a> |

# Multisets
Standard `Collection` operations, such as `containsAll`, ignore the count of elements in the multiset, and only care about whether elements are in the multiset at all, or not.  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html'><code>Multisets</code></a> provides a number of operations that take into account element multiplicities in multisets.

| Method                                   | Explanation                              | Difference from `Collection` method      |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#containsOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>containsOccurrences(Multiset sup, Multiset sub)</code></a> | Returns `true` if `sub.count(o) <= super.count(o)` for all `o`. | `Collection.containsAll` ignores counts, and only tests whether elements are contained at all. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#removeOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>removeOccurrences(Multiset removeFrom, Multiset toRemove)</code></a> | Removes one occurrence in `removeFrom` for each occurrence of an element in `toRemove`. | `Collection.removeAll` removes all occurences of any element that occurs even once in `toRemove`. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#retainOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>retainOccurrences(Multiset removeFrom, Multiset toRetain)</code></a> | Guarantees that `removeFrom.count(o) <= toRetain.count(o)` for all `o`. | `Collection.retainAll` keeps all occurrences of elements that occur even once in `toRetain`. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#intersection(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>intersection(Multiset, Multiset)</code></a> | Returns a view of the intersection of two multisets; a nondestructive alternative to `retainOccurrences`. | Has no analogue                          |

```java

Multiset<String> multiset1 = HashMultiset.create();
multiset1.add("a", 2);

Multiset<String> multiset2 = HashMultiset.create();
multiset2.add("a", 5);

multiset1.containsAll(multiset2); // returns true: all unique elements are contained, 
  // even though multiset1.count("a") == 2 < multiset2.count("a") == 5
Multisets.containsOccurrences(multiset1, multiset2); // returns false

multiset2.removeOccurrences(multiset1); // multiset2 now contains 3 occurrences of "a"

multiset2.removeAll(multiset1); // removes all occurrences of "a" from multiset2, even though multiset1.count("a") == 2
multiset2.isEmpty(); // returns true
```

Other utilities in `Multisets` include:

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#copyHighestCountFirst(com.google.common.collect.Multiset)'><code>copyHighestCountFirst(Multiset)</code></a> | Returns an immutable copy of the multiset that iterates over elements in descending frequency order. |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#unmodifiableMultiset(com.google.common.collect.Multiset)'><code>unmodifiableMultiset(Multiset)</code></a> | Returns an unmodifiable view of the multiset. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#unmodifiableSortedMultiset(com.google.common.collect.SortedMultiset)'><code>unmodifiableSortedMultiset(SortedMultiset)</code></a> | Returns an unmodifiable view of the sorted multiset. |

```java

Multiset<String> multiset = HashMultiset.create();
multiset.add("a", 3);
multiset.add("b", 5);
multiset.add("c", 1);

ImmutableMultiset<String> highestCountFirst = Multisets.copyHighestCountFirst(multiset);

// highestCountFirst, like its entrySet and elementSet, iterates over the elements in order {"b", "a", "c"}
```

# Multimaps
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html'><code>Multimaps</code></a> provides a number of general utility operations that deserve individual explanation.

### `index`
The cousin to `Maps.uniqueIndex`, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#index(java.lang.Iterable, com.google.common.base.Function)'><code>Multimaps.index(Iterable, Function)</code></a> answers the case when you want to be able to look up all objects with some particular attribute in common, which is not necessarily unique.

Let's say we want to group strings based on their length.

```java

ImmutableSet<String> digits = ImmutableSet.of("zero", "one", "two", "three", "four",
  "five", "six", "seven", "eight", "nine");
Function<String, Integer> lengthFunction = new Function<String, Integer>() {
  public Integer apply(String string) {
    return string.length();
  }
};
ImmutableListMultimap<Integer, String> digitsByLength = Multimaps.index(digits, lengthFunction);
/*
 * digitsByLength maps:
 *  3 => {"one", "two", "six"}
 *  4 => {"zero", "four", "five", "nine"}
 *  5 => {"three", "seven", "eight"}
 */
```

### `invertFrom`
Since `Multimap` can map many keys to one value, and one key to many values, it can be useful to invert a `Multimap`.  Guava provides <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#invertFrom(com.google.common.collect.Multimap, M)'><code>invertFrom(Multimap toInvert, Multimap dest)</code></a> to let you do this, without choosing an implementation for you.

_NOTE:_ If you are using an `ImmutableMultimap`, consider <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultimap.html#inverse()'><code>ImmutableMultimap.inverse()</code></a> instead.

```java

ArrayListMultimap<String, Integer> multimap = ArrayListMultimap.create();
multimap.putAll("b", Ints.asList(2, 4, 6));
multimap.putAll("a", Ints.asList(4, 2, 1));
multimap.putAll("c", Ints.asList(2, 5, 3));

TreeMultimap<Integer, String> inverse = Multimaps.invertFrom(multimap, TreeMultimap.<String, Integer> create());
// note that we choose the implementation, so if we use a TreeMultimap, we get results in order
/*
 * inverse maps:
 *  1 => {"a"}
 *  2 => {"a", "b", "c"}
 *  3 => {"c"}
 *  4 => {"a", "b"}
 *  5 => {"c"}
 *  6 => {"b"}
 */
```

### `forMap`
Need to use a `Multimap` method on a `Map`?  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#forMap(java.util.Map)'><code>forMap(Map)</code></a> views a `Map` as a `SetMultimap`.  This is particularly useful, for example, in combination with `Multimaps.invertFrom`.

```java

Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 1, "c", 2);
SetMultimap<String, Integer> multimap = Multimaps.forMap(map);
// multimap maps ["a" => {1}, "b" => {1}, "c" => {2}]
Multimap<Integer, String> inverse = Multimaps.invertFrom(multimap, HashMultimap.<Integer, String> create());
// inverse maps [1 => {"a", "b"}, 2 => {"c"}]
```

### Wrappers
`Multimaps` provides the traditional wrapper methods, as well as tools to get custom `Multimap` implementations based on `Map` and `Collection` implementations of your choice.

| Unmodifiable          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableMultimap(com.google.common.collect.Multimap)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableListMultimap(com.google.common.collect.ListMultimap)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableSetMultimap(com.google.common.collect.SetMultimap)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableSortedSetMultimap(com.google.common.collect.SortedSetMultimap)'><code>SortedSetMultimap</code></a> |
| :-------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| Synchronized          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedMultimap(com.google.common.collect.Multimap)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedListMultimap(com.google.common.collect.ListMultimap)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedSetMultimap(com.google.common.collect.SetMultimap)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedSortedSetMultimap(com.google.common.collect.SortedSetMultimap)'><code>SortedSetMultimap</code></a> |
| Custom Implementation | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newMultimap(java.util.Map, com.google.common.base.Supplier)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newListMultimap(java.util.Map, com.google.common.base.Supplier)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSetMultimap(java.util.Map, com.google.common.base.Supplier)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSortedSetMultimap(java.util.Map, com.google.common.base.Supplier)'><code>SortedSetMultimap</code></a> |

The custom `Multimap` implementations let you specify a particular implementation that should be used in the returned `Multimap`.  Caveats include:

* The multimap assumes complete ownership over of map and the lists returned by factory. Those objects should not be manually updated, they should be empty when provided, and they should not use soft, weak, or phantom references.
* **No guarantees are made** on what the contents of the `Map` will look like after you modify the `Multimap`.
    * The multimap is not threadsafe when any concurrent operations update the multimap, even if map and the instances generated by factory are. Concurrent read operations will work correctly, though.  Work around this with the `synchronized` wrappers if necessary.
    * The multimap is serializable if map, factory, the lists generated by factory, and the multimap contents are all serializable.
    * The collections returned by `Multimap.get(key)` are _not_ of the same type as the collections returned by your `Supplier`, though if you supplier returns `RandomAccess` lists, the lists returned by `Multimap.get(key)` will also be random access.

Note that the custom `Multimap` methods expect a `Supplier` argument to generate fresh new collections.  Here is an example of writing a `ListMultimap` backed by a `TreeMap` mapping to `LinkedList`.

```java

ListMultimap<String, Integer> myMultimap = Multimaps.newListMultimap(
  Maps.<String, Collection<Integer>>newTreeMap(),
  new Supplier<LinkedList<Integer>>() {
    public LinkedList<Integer> get() {
      return Lists.newLinkedList();
    }
  });
```

# Tables
The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html'><code>Tables</code></a> class provides a few handy utilities.

### `customTable`
Comparable to the `Multimaps.newXXXMultimap(Map, Supplier)` utilities, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#newCustomTable(java.util.Map, com.google.common.base.Supplier)'><code>Tables.newCustomTable(Map, Supplier&lt;Map&gt;)</code></a> allows you to specify a `Table` implementation using whatever row or column map you like.

```java
// use LinkedHashMaps instead of HashMaps
Table<String, Character, Integer> table = Tables.newCustomTable(
  Maps.<String, Map<Character, Integer>>newLinkedHashMap(),
  new Supplier<Map<Character, Integer>> () {
    public Map<Character, Integer> get() {
      return Maps.newLinkedHashMap();
    }
  });
```

### `transpose`
The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#transpose(com.google.common.collect.Table)'><code>transpose(Table&lt;R, C, V&gt;)</code></a> method allows you to view a `Table<R, C, V>` as a `Table<C, R, V>`.

### Wrappers
These are the familiar unmodifiability wrappers you know and love.  Consider, however, using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html'><code>ImmutableTable</code></a> instead in most cases.

| Unmodifiable | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#unmodifiableTable(com.google.common.collect.Table)'><code>Table</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#unmodifiableRowSortedTable(com.google.common.collect.RowSortedTable)'><code>RowSortedTable</code></a> |
|:-------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
