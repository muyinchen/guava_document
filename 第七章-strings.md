# Joiner
Joining together a sequence of strings with a separator can be unnecessarily tricky -- but it shouldn't be.  If your sequence contains nulls, it can be even harder.  The fluent style of <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Joiner.html'><code>Joiner</code></a> makes it simple.

```java

Joiner joiner = Joiner.on("; ").skipNulls();
return joiner.join("Harry", null, "Ron", "Hermione");
```
returns the string "Harry; Ron; Hermione".  Alternately, instead of using `skipNulls`, you may specify a string to use instead of null with `useForNull(String)`.

You may also use `Joiner` on objects, which will be converted using their `toString()` and then joined.

```java

Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"
```

**Warning:** joiner instances are always immutable.  The joiner configuration methods will always return a new `Joiner`, which you must use to get the desired semantics.  This makes any `Joiner` thread safe, and usable as a `static final` constant.

# Splitter
The built in Java utilities for splitting strings can have some quirky behaviors.  For example, `String.split` silently discards trailing separators, and `StringTokenizer` respects exactly five whitespace characters and nothing else.

Quiz: `",a,,b,".split(",")` returns...
  1. `"", "a", "", "b", ""`
  2. `null, "a", null, "b", null`
  3. `"a", null, "b"`
  4. `"a", "b"`
  5. None of the above

The correct answer is none of the above: `"", "a", "", "b"`.  Only trailing empty strings are skipped.  What is this I don't even.

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html'><code>Splitter</code></a> allows complete control over all this confusing behavior using a reassuringly straightforward fluent pattern.

```java

Splitter.on(',')
       .trimResults()
       .omitEmptyStrings()
       .split("foo,bar,,   qux");
```
returns an `Iterable<String>` containing "foo", "bar", "qux".  A `Splitter` may be set to split on any `Pattern`, `char`, `String`, or `CharMatcher`.

### Base Factories
| Method                                   | Description                              | Example            |
| :--------------------------------------- | :--------------------------------------- | :----------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on(char)'><code>Splitter.on(char)</code></a> | Split on occurrences of a specific, individual character. | `Splitter.on(';')` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on(com.google.common.base.CharMatcher)'><code>Splitter.on(CharMatcher)</code></a> | Split on occurrences of any character in some category. | `Splitter.on(CharMatcher.BREAKING_WHITESPACE)`<br><code>Splitter.on(CharMatcher.anyOf(";,."))</code>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on(java.lang.String)'><code>Splitter.on(String)</code></a> </td><td> Split on a literal <code>String</code>. </td><td> <code>Splitter.on(", ")</code> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on(java.util.regex.Pattern)'><code>Splitter.on(Pattern)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#onPattern(java.lang.String)'><code>Splitter.onPattern(String)</code></a> </td><td> Split on a regular expression. </td><td> <code>Splitter.onPattern("\r?\n")</code> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#fixedLength(int)'><code>Splitter.fixedLength(int)</code></a> </td><td> Splits strings into substrings of the specified fixed length.  The last piece can be smaller than <code>length</code>, but will never be empty. </td><td> <code>Splitter.fixedLength(3)</code> </td></tr></tbody></table>

### Modifiers
| Method                                   | Description                              | Example                                  |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#omitEmptyStrings()'><code>omitEmptyStrings()</code></a> | Automatically omits empty strings from the result. | `Splitter.on(',').omitEmptyStrings().split("a,,c,d")` returns `"a", "c", "d"` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#trimResults()'><code>trimResults()</code></a> | Trims whitespace from the results; equivalent to `trimResults(CharMatcher.WHITESPACE)`. | `Splitter.on(',').trimResults().split("a, b, c, d")` returns `"a", "b", "c", "d"` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#trimResults(com.google.common.base.CharMatcher)'><code>trimResults(CharMatcher)</code></a> | Trims characters matching the specified `CharMatcher` from results. | `Splitter.on(',').trimResults(CharMatcher.is('_')).split("_a ,_b_ ,c__")` returns `"a ", "b_ ", "c"`. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#limit(int)'><code>limit(int)</code></a> | Stops splitting after the specified number of strings have been returned. | `Splitter.on(',').limit(3).split("a,b,c,d")` returns `"a", "b", "c,d"` |

TODO: Map splitters

If you wish to get a `List`, just use `Lists.newArrayList(splitter.split(string))` or the like.

**Warning:** splitter instances are always immutable.  The splitter configuration methods will always return a new `Splitter`, which you must use to get the desired semantics.  This makes any `Splitter` thread safe, and usable as a `static final` constant.

<!--
<a href='Hidden comment: 
= Escaper =
Escaping strings correctly -- converting them into a format safe for inclusion in e.g. an XML document or a Java source file -- can be a tricky business, and critical for security reasons.  Guava provides a flexible API for escaping text, and a number of built-in escapers, in the com.google.common.escape package.

All escapers in Guava extend the [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/escape/Escaper.html Escaper] abstract class, and support the method String escape(String).  Built-in Escaper instances can be found in several classes, depending on your needs: [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/html/HtmlEscapers.html HtmlEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/xml/XmlEscapers.html XmlEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/escape/SourceCodeEscapers.html SourceCodeEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/net/UriEscapers.html UriEscapers], or you can build your own with [http://google.github.io/guava/releases/snapshot/api/docs/ an Escapers.Builder].  To inspect an Escaper, you can use Escapers.computeReplacement to find the replacement string for a given character.
'></a>
-->

# CharMatcher
In olden times, our `StringUtil` class grew unchecked, and had
many methods like these:

|                  |                  |                        |                      |                |
| :--------------- | :--------------- | :--------------------- | :------------------- | :------------- |
| `allAscii`       | `collapse`       | `collapseControlChars` | `collapseWhitespace` | `indexOfChars` |
| `lastIndexNotOf` | `numSharedChars` | `removeChars`          | `removeCrLf`         | `replaceChars` |
| `retainAllChars` | `strip`          | `stripAndCollapse`     | `stripNonDigits`     |                |

They represent a partial cross product of two notions:

  1. what constitutes a "matching" character?
  2. what to do with those "matching" characters?

To simplify this morass, we developed `CharMatcher`.

Intuitively, you can think of a `CharMatcher` as representing a particular class of characters, like digits or whitespace.  Practically speaking, a `CharMatcher` is just a boolean predicate on characters -- indeed, `CharMatcher` implements [[Predicate&lt;Character&gt;|FunctionalExplained#predicate]] -- but because it is so common to refer to "all whitespace characters" or "all lowercase letters," Guava provides this specialized syntax and API for characters.

But the utility of a `CharMatcher` is in the _operations_ it lets you perform on occurrences of the specified class of characters: trimming, collapsing, removing, retaining, and much more.  An object of type `CharMatcher` represents notion 1: what constitutes a matching character?  It then provides many operations answering notion 2: what to do with those matching characters?  The result is that API complexity increases linearly for quadratically increasing flexibility and power.  Yay!

```
String noControl = CharMatcher.JAVA_ISO_CONTROL.removeFrom(string); // remove control characters
String theDigits = CharMatcher.DIGIT.retainFrom(string); // only the digits
String spaced = CharMatcher.WHITESPACE.trimAndCollapseFrom(string, ' ');
  // trim whitespace at ends, and replace/collapse whitespace into single spaces
String noDigits = CharMatcher.JAVA_DIGIT.replaceFrom(string, "*"); // star out all digits
String lowerAndDigit = CharMatcher.JAVA_DIGIT.or(CharMatcher.JAVA_LOWER_CASE).retainFrom(string);
  // eliminate all characters that aren't digits or lowercase
```

**Note:** `CharMatcher` deals only with `char` values; it does not understand supplementary Unicode code points in the range 0x10000 to 0x10FFFF. Such logical characters are encoded into a `String` using surrogate pairs, and a `CharMatcher` treats these just as two separate characters.

## Obtaining CharMatchers
Many needs can be satisfied by the provided `CharMatcher` constants:

|                                          |                                          |                                          |                                          |                                          |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#ANY'><code>ANY</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#NONE'><code>NONE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#WHITESPACE'><code>WHITESPACE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#BREAKING_WHITESPACE'><code>BREAKING_WHITESPACE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#INVISIBLE'><code>INVISIBLE</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#DIGIT'><code>DIGIT</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_LETTER'><code>JAVA_LETTER</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_DIGIT'><code>JAVA_DIGIT</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_LETTER_OR_DIGIT'><code>JAVA_LETTER_OR_DIGIT</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_ISO_CONTROL'><code>JAVA_ISO_CONTROL</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_LOWER_CASE'><code>JAVA_LOWER_CASE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_UPPER_CASE'><code>JAVA_UPPER_CASE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#ASCII'><code>ASCII</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#SINGLE_WIDTH'><code>SINGLE_WIDTH</code></a> |                                          |

Other common ways to obtain a `CharMatcher` include:

| Method                                   | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#anyOf(java.lang.CharSequence)'><code>anyOf(CharSequence)</code></a> | Specify all the characters you wish matched.  For example, `CharMatcher.anyOf("aeiou")` matches lowercase English vowels. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#is(char)'><code>is(char)</code></a> | Specify exactly one character to match.  |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#inRange(char, char)'><code>inRange(char, char)</code></a> | Specify a range of characters to match, e.g. `CharMatcher.inRange('a', 'z')`. |

Additionally, `CharMatcher` has <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#negate()'><code>negate()</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#and(com.google.common.base.CharMatcher)'><code>and(CharMatcher)</code></a>, and <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#or(com.google.common.base.CharMatcher)'><code>or(CharMatcher)</code></a>.  These provide simple boolean operations on `CharMatcher`.

## Using CharMatchers
`CharMatcher` provides a [wide variety](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#method_summary) of methods to operate on occurrences of the specified characters in any `CharSequence`.  There are more methods provided than we can list here, but some of the most commonly used are:

| Method                                   | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#collapseFrom(java.lang.CharSequence, char)'><code>collapseFrom(CharSequence, char)</code></a> | Replace each group of consecutive matched characters with the specified character.  For example, `WHITESPACE.collapseFrom(string, ' ')` collapses whitespaces down to a single space. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#matchesAllOf(java.lang.CharSequence)'><code>matchesAllOf(CharSequence)</code></a> | Test if this matcher matches all characters in the sequence.  For example, `ASCII.matchesAllOf(string)` tests if all characters in the string are ASCII. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#removeFrom(java.lang.CharSequence)'><code>removeFrom(CharSequence)</code></a> | Removes matching characters from the sequence. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#retainFrom(java.lang.CharSequence)'><code>retainFrom(CharSequence)</code></a> | Removes all non-matching characters from the sequence. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#trimFrom(java.lang.CharSequence)'><code>trimFrom(CharSequence)</code></a> | Removes leading and trailing matching characters. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#replaceFrom(java.lang.CharSequence, java.lang.CharSequence)'><code>replaceFrom(CharSequence, CharSequence)</code></a> | Replace matching characters with a given sequence. |

(Note: all of these methods return a `String`, except for `matchesAllOf`, which returns a `boolean`.)

# Charsets
Don't do this:

```java

try {
  bytes = string.getBytes("UTF-8");
} catch (UnsupportedEncodingException e) {
  // how can this possibly happen?
  throw new AssertionError(e);
}
```

Do this instead:

```java

bytes = string.getBytes(Charsets.UTF_8);
```

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Charsets.html'><code>Charsets</code></a> provides constant references to the six standard `Charset` implementations guaranteed to be supported by all Java platform implementations.  Use them instead of referring to charsets by their names.

TODO: an explanation of charsets and when to use them

(Note: If you're using JDK7, you should use the constants in <a href='http://docs.oracle.com/javase/7/docs/api/java/nio/charset/StandardCharsets.html'><code>StandardCharsets</code></a> instead!)

# CaseFormat
`CaseFormat` is a handy little class for converting between ASCII case conventions -- like, for example, naming conventions for programming languages.  Supported formats include:

| Format                                   | Example            |
| :--------------------------------------- | :----------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_CAMEL'><code>LOWER_CAMEL</code></a> | `lowerCamel`       |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_HYPHEN'><code>LOWER_HYPHEN</code></a> | `lower-hyphen`     |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_UNDERSCORE'><code>LOWER_UNDERSCORE</code></a> | `lower_underscore` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#UPPER_CAMEL'><code>UPPER_CAMEL</code></a> | `UpperCamel`       |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#UPPER_UNDERSCORE'><code>UPPER_UNDERSCORE</code></a> | `UPPER_UNDERSCORE` |

Using it is relatively straightforward:

```java
CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "CONSTANT_NAME")); // returns "constantName"
```

We find this especially useful, for example, when writing programs that generate other programs.
