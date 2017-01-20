# Overview
The _primitive_ types of Java are the basic types:

| `byte`  | `short`  | `int`  | `long`    |
| :------ | :------- | :----- | :-------- |
| `float` | `double` | `char` | `boolean` |

**_Before searching Guava for a method, you should check if it is in [Arrays](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/Arrays.html) or the corresponding JDK wrapper type, e.g. [Integer](http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/Integer.html)._**

These types cannot be used as objects or as type parameters to generic types, which means that many general-purpose utilities cannot be applied to them.  Guava provides a number of these general-purpose utilities, ways of interfacing between primitive arrays and collection APIs, conversion from types to byte array representations, and support for unsigned behaviors on certain types.

| Primitive Type | Guava Utilities (all in `com.google.common.primitives`) |
| :------------- | :--------------------------------------- |
| `byte`         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Bytes.html'><code>Bytes</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/SignedBytes.html '><code>SignedBytes</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedBytes.html'><code>UnsignedBytes</code></a> |
| `short`        | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Shorts.html'><code>Shorts</code></a> |
| `int`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Ints.html'><code>Ints</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInteger.html'><code>UnsignedInteger</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html'><code>UnsignedInts</code></a> |
| `long`         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Longs.html'><code>Longs</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLong.html'><code>UnsignedLong</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html'><code>UnsignedLongs</code></a> |
| `float`        | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Floats.html'><code>Floats</code></a> |
| `double`       | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Doubles.html'><code>Doubles</code></a> |
| `char`         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Chars.html'><code>Chars</code></a> |
| `boolean`      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Booleans.html'><code>Booleans</code></a> |

Methods that differ in behavior for signed and unsigned bytes are completely skipped in `Bytes`, but only present in the `SignedBytes` and `UnsignedBytes` utilities, since the signedness of bytes is somewhat more ambiguous than the signedness of other types.

Unsigned variants of methods on `int` and `long` are provided in the `UnsignedInts` and `UnsignedLongs` classes, but since most uses of those types are signed, the `Ints` and `Longs` classes treat their inputs as signed.

Additionally, Guava provides "wrapper types" for unsigned `int` and `long` values, `UnsignedInteger` and `UnsignedLong`, to help you use the type system to enforce distinctions between signed and unsigned values, in exchange for a small performance cost.  These classes directly support simple arithmetic operations in the style of `BigInteger`.

All method signatures use `Wrapper` to refer to the corresponding JDK wrapper type, and `prim` to refer to the primitive type.  (`Prims`, where applicable, refers to the corresponding Guava utilities class.)

# Primitive array utilities
Primitive arrays are the most efficient way (in both memory and performance) to work with primitive types in aggregate.  Guava provides a variety of utilities to work with these methods.

| Signature                                | Description                              | Collection analogue                      | Availability        |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :------------------ |
| `List<Wrapper> asList(prim... backingArray)` | Wraps a primitive array as a `List` of the corresponding wrapper type. | [Arrays.asList](http://docs.oracle.com/javase/6/docs/api/java/util/Arrays.html#asList(T...)) | Sign-independent`*` |
| `prim[] toArray(Collection<Wrapper> collection)` | Copies a collection into a new `prim[]`.  This method is as thread-safe as `collection.toArray()`. | [Collection.toArray()](http://docs.oracle.com/javase/6/docs/api/java/util/Collection.html#toArray()) | Sign-independent    |
| `prim[] concat(prim[]... arrays)`        | Concatenate several primitive arrays.    | [Iterables.concat](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#concat(java.lang.Iterable...)) | Sign-independent    |
| `boolean contains(prim[] array, prim target)` | Determines if the specified element is in the specified array. | [Collection.contains](http://docs.oracle.com/javase/6/docs/api/java/util/Collection.html#contains(java.lang.Object)) | Sign-independent    |
| `int indexOf(prim[] array, prim target)` | Finds the index of the first appearance of the value `target` in `array`, or returns `-1` if no such value exists. | [List.indexOf](http://docs.oracle.com/javase/6/docs/api/java/util/List.html#indexOf(java.lang.Object)) | Sign-independent    |
| `int lastIndexOf(prim[] array, prim target)` | Finds the index of the last appearance of the value `target` in `array`, or returns `-1` if no such value exists. | [List.lastIndexOf](http://docs.oracle.com/javase/6/docs/api/java/util/List.html#lastIndexOf(java.lang.Object)) | Sign-independent    |
| `prim min(prim... array)`                | Returns the minimum _element_ of the array. | [Collections.min](http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#min(java.util.Collection)) | Sign-dependent`**`  |
| `prim max(prim... array)`                | Returns the maximum _element_ of the array. | [Collections.max](http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#max(java.util.Collection)) | Sign-dependent      |
| `String join(String separator, prim... array)` | Constructs a string containing the elements of `array`, separated by `separator`. | [Joiner.on(separator).join](https://github.com/google/guava/wiki/StringsExplained#joiner) | Sign-dependent      |
| `Comparator<prim[]> lexicographicalComparator()` | A comparator which compares primitive arrays lexicographically. | [Ordering.natural().lexicographical()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#lexicographical()) | Sign-dependent      |

`*` Sign-independent methods are present in: `Bytes`, `Shorts`, `Ints`, `Longs`, `Floats`, `Doubles`, `Chars`, `Booleans`.  _Not_ `UnsignedInts`, `UnsignedLongs`, `SignedBytes`, or `UnsignedBytes`.

`**` Sign-dependent methods are present in: `SignedBytes`, `UnsignedBytes`, `Shorts`, `Ints`, `Longs`, `Floats`, `Doubles`, `Chars`, `Booleans`, `UnsignedInts`, `UnsignedLongs`.  _Not_ `Bytes`.

# General utility methods
Guava provides a number of basic utilities which were not part of JDK 6.  Some of these methods, however, are available in JDK 7.

| Signature                        | Description                              | Availability                             |
| :------------------------------- | :--------------------------------------- | :--------------------------------------- |
| `int compare(prim a, prim b)`    | A traditional `Comparator.compare` method, but on the primitive types.  _Provided in the JDK wrapper classes as of JDK 7._ | Sign-dependent                           |
| `prim checkedCast(long value)`   | Casts the specified value to `prim`, _unless_ the specified value does not fit into a `prim`, in which case an `IllegalArgumentException` is thrown. | Sign-dependent for integral types only`*` |
| `prim saturatedCast(long value)` | Casts the specified value to `prim`, unless the specified value does not fit into a `prim`, in which case the closest `prim` value is used. | Sign-dependent for integral types only   |


`*`Here, integral types include `byte`, `short`, `int`, `long`.  Integral types do _not_ include `char`, `boolean`, `float`, or `double`.

_Note:_ Rounding from `double` is provided in `com.google.common.math.DoubleMath`, and supports a variety of rounding modes.  See [[the article|MathExplained#floating-point-arithmetic]] for details.

# Byte conversion methods
Guava provides methods to convert primitive types to and from byte array representations **in big-endian order**.  All methods are sign-independent, except that `Booleans` provides none of these methods.

| Signature                               | Description                              |
| :-------------------------------------- | :--------------------------------------- |
| `int BYTES`                             | Constant representing the number of bytes needed to represent a `prim` value. |
| `prim fromByteArray(byte[] bytes)`      | Returns the `prim` value whose big-endian representation is the first `Prims.BYTES` bytes in the array `bytes`.  Throws an `IllegalArgumentException` if `bytes.length <= Prims.BYTES`. |
| `prim fromBytes(byte b1, ..., byte bk)` | Takes `Prims.BYTES` byte arguments.  Returns the `prim` value whose byte representation is the specified bytes in big-endian order. |
| `byte[] toByteArray(prim value)`        | Returns an array containing the big-endian byte representation of `value`. |

# Unsigned support
The `UnsignedInts` and `UnsignedLongs` utility classes provide some of the generic utilities that Java provides for signed types in their wrapper classes.  `UnsignedInts` and `UnsignedLongs` deal with the primitive type directly: it is up to you to make sure that only unsigned values are passed to these utilities.

Additionally, for `int` and `long`, Guava provides "unsigned" wrapper types ([UnsignedInteger](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInteger.html) and [UnsignedLong](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLong.html)) to help you enforce distinctions between unsigned and signed values in the type system, in exchange for a small performance penalty.

## Generic utilities
These methods' signed analogues are provided in the wrapper classes in the JDK.

| Signature | Explanation |
|:----------|:------------|
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#parseUnsignedInt(java.lang.String)'><code>int UnsignedInts.parseUnsignedInt(String)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#parseUnsignedLong(java.lang.String)'><code>long UnsignedLongs.parseUnsignedLong(String)</code></a> <table><thead><th> Parses an unsigned value from a string in base 10. </th></thead><tbody>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#parseUnsignedInt(java.lang.String, int)'><code>int UnsignedInts.parseUnsignedInt(String string, int radix)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#parseUnsignedLong(java.lang.String)'><code>long UnsignedLongs.parseUnsignedLong(String string, int radix)</code></a> </td><td> Parses an unsigned value from a string in the specified base. </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#toString(int)'><code>String UnsignedInts.toString(int)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#toString(long)'><code>String UnsignedLongs.toString(long)</code></a> </td><td> Returns a string representation of the unsigned value in base 10. </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#toString(int, int)'><code>String UnsignedInts.toString(int value, int radix)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#toString(long, int)'><code>String UnsignedLongs.toString(long value, int radix)</code></a> </td><td> Returns a string representation of the unsigned value in the specified base.</td></tr></tbody></table>

<h2>Wrapper</h2>
The provided unsigned wrapper types include a number of methods to make their use and conversion easier.<br>
<br>
<table><thead><th> Signature </th><th> Explanation </th></thead><tbody>
<tr><td> <code>UnsignedPrim plus(UnsignedPrim)</code>, <code>minus</code>, <code>times</code>, <code>dividedBy</code>, <code>mod</code> </td><td> Simple arithmetic operations. </td></tr>
<tr><td> <code>UnsignedPrim valueOf(BigInteger)</code> </td><td> Returns the value from a <code>BigInteger</code> as an <code>UnsignedPrim</code>, or throw an <code>IAE</code> if the specified <code>BigInteger</code> is negative or does not fit. </td></tr>
<tr><td> <code>UnsignedPrim valueOf(long)</code> </td><td> Returns the value from the <code>long</code> as an <code>UnsignedPrim</code>, or throw an <code>IAE</code> if the specified <code>long</code> is negative or does not fit. </td></tr>
<tr><td> <code>UnsignedPrim fromPrimBits(prim value)</code> </td><td> View the given value as unsigned.  For example, <code>UnsignedInteger.fromIntBits(1 &lt;&lt; 31)</code> has the value 2<sup>31</sup>, even though <code>1 &lt;&lt; 31</code> is negative as an <code>int</code>. </td></tr>
<tr><td> <code>BigInteger bigIntegerValue()</code> </td><td> Get the value of this <code>UnsignedPrim</code> as a <code>BigInteger</code>. </td></tr>
<tr><td> <code>toString()</code>, <code>toString(int radix)</code> </td><td> Returns a string representation of this unsigned value. </td></tr>
