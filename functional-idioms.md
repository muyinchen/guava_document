# Caveats
As of Java 7, functional programming in Java can only be approximated through awkward and verbose use of anonymous classes.  This is expected to change in Java 8, but Guava is currently aimed at users of Java 5 and above.

_Excessive_ use of Guava's functional programming idioms can lead to verbose, confusing, unreadable, and inefficient code.  These are by far the most easily (and most commonly) abused parts of Guava, and when you go to preposterous lengths to make your code "a one-liner," the Guava team weeps.

Compare the following code:
```java

Function<String, Integer> lengthFunction = new Function<String, Integer>() {
  public Integer apply(String string) {
    return string.length();
  }
};
Predicate<String> allCaps = new Predicate<String>() {
  public boolean apply(String string) {
    return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string);
  }
};
Multiset<Integer> lengths = HashMultiset.create(
  Iterables.transform(Iterables.filter(strings, allCaps), lengthFunction));
```
or the `FluentIterable` version
```java

Multiset<Integer> lengths = HashMultiset.create(
  FluentIterable.from(strings)
    .filter(new Predicate<String>() {
       public boolean apply(String string) {
         return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string);
       }
     })
    .transform(new Function<String, Integer>() {
       public Integer apply(String string) {
         return string.length();
       }
     }));
```
with:
```java
Multiset<Integer> lengths = HashMultiset.create();
for (String string : strings) {
  if (CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string)) {
    lengths.add(string.length());
  }
}
```

Even using static imports, even if the Function and the Predicate declarations are moved to a different file, the first implementation is less concise, less readable, and less efficient.

Imperative code should be your _default_, your _first choice_ as of Java 7.  You should not use functional idioms unless you are _absolutely_ sure of one of the following:

* Use of functional idioms will result in _net_ savings of lines of code for your entire project.  In the example above, the "functional" version used 11 lines, the imperative version 6.  Moving the definition of a function to another file, or a constant, does not help.
* For efficiency, you need a lazily computed view of the transformed collection and cannot settle for an explicitly computed collection.  Additionally, you have read and reread Effective Java, item 55, and besides following those instructions, you have actually done benchmarking to prove that this version is faster, and can cite  numbers to prove it.

Please be sure, when using Guava's functional utilities, that the traditional imperative way of doing things isn't more readable.  Try writing it out.  Was that so bad?  Was that more readable than the preposterously awkward functional approach you were about to try?

# Functions and Predicates
This article discusses only those Guava features dealing directly with `Function` and `Predicate`.  Some other utilities are associated with the "functional style," such as concatenation and other methods which return views in constant time.  Try looking in the [[collection utilities|CollectionUtilitiesExplained]] article.

Guava provides two basic "functional" interfaces:
* `Function<A, B>`, which has the single method `B apply(A input)`.  Instances of `Function` are generally expected to be referentially transparent -- no side effects -- and to be consistent with equals, that is, `a.equals(b)` implies that `function.apply(a).equals(function.apply(b))`.
* `Predicate<T>`, which has the single method `boolean apply(T input)`.  Instances of `Predicate` are generally expected to be side-effect-free and consistent with equals.

### Special predicates
Characters get their own specialized version of `Predicate`, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html'><code>CharMatcher</code></a>, which is typically more efficient and more useful for those needs. `CharMatcher` already implements `Predicate<Character>`, and can be used correspondingly, while conversion from a `Predicate` to a `CharMatcher` can be done using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#forPredicate(com.google.common.base.Predicate)'><code>CharMatcher.forPredicate</code></a>.  Consult [[the CharMatcher article|StringsExplained#charmatcher]] for details.

Additionally, for comparable types and comparison-based predicates, most needs can be fulfilled using the `Range` type, which implements an immutable interval.  The `Range` type implements `Predicate`, testing containment in the range.  For example, `Ranges.atMost(2)` is a perfectly valid `Predicate<Integer>`.  More details on using ranges can be found [[in the corresponding article|RangesExplained]].

### Manipulating Functions and Predicates

Simple `Function` construction and manipulation methods are provided in [Functions](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html), including

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#forMap(java.util.Map)'><code>forMap(Map&lt;A, B&gt;)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#compose(com.google.common.base.Function, com.google.common.base.Function)'><code>compose(Function&lt;B, C&gt;, Function&lt;A, B&gt;)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#constant(E)'><code>constant(T)</code></a> |  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#identity()'><code>identity()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#toStringFunction()'><code>toStringFunction()</code></a> |
|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|

Consult the Javadoc for details.

There are considerably more construction and manipulation methods available in [Predicates](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html), but a sample includes:

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#instanceOf(java.lang.Class)'><code>instanceOf(Class)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#assignableFrom(java.lang.Class)'><code>assignableFrom(Class)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#contains(java.util.regex.Pattern)'><code>contains(Pattern)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#in(java.util.Collection)'><code>in(Collection)</code></a> |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#isNull()'><code>isNull()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#alwaysFalse()'><code>alwaysFalse()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#alwaysTrue()'><code>alwaysTrue()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#equalTo(T)'><code>equalTo(Object)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#compose(com.google.common.base.Predicate, com.google.common.base.Function)'><code>compose(Predicate, Function)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#and(com.google.common.base.Predicate...)'><code>and(Predicate...)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#or(com.google.common.base.Predicate...)'><code>or(Predicate...)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#not(com.google.common.base.Predicate)'><code>not(Predicate)</code></a> |

Consult the Javadoc for details.

# Using
Guava provides many tools to manipulate collections using functions and predicates.  These can typically be found in the collection utility classes `Iterables`, `Lists`, `Sets`, `Maps`, `Multimaps`, and the like.

## Predicates
The most basic use of predicates is to filter collections.  All Guava filter methods return _views_.

| Collection type | Filter method |
|:----------------|:--------------|
| `Iterable`      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#filter(java.lang.Iterable, com.google.common.base.Predicate)'><code>Iterables.filter(Iterable, Predicate)</code></a> | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#filter(com.google.common.base.Predicate)'><code>FluentIterable.filter(Predicate)</code></a> 
