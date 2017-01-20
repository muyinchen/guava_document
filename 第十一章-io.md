# `ByteStreams` and `CharStreams`
Guava uses the term "stream" to refer to a `Closeable` stream for I/O data which has positional state in the underlying resource. The term "`byte` stream" refers to an `InputStream` or `OutputStream`, while "`char` stream" refers to a `Reader` or `Writer` (though their supertypes `Readable` and `Appendable` are often used as method parameter types). Corresponding utilities are divided into the utility classes <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html'><code>ByteStreams</code></a> and <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html'><code>CharStreams</code></a>.

Most Guava stream-related utilities deal with an entire stream at a time and/or handle buffering themselves for efficiency. Also note that Guava methods that take a stream do _not_ close the stream: closing streams is generally the responsibility of the code that opens the stream.

Some of the methods provided by these classes include:

| **`ByteStreams`**                        | **`CharStreams`**                        |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#toByteArray(java.io.InputStream)'><code>byte[] toByteArray(InputStream)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#toString(java.lang.Readable)'><code>String toString(Readable)</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#readLines(java.lang.Readable)'><code>List&lt;String&gt; readLines(Readable)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#copy(java.io.InputStream, java.io.OutputStream)'><code>long copy(InputStream, OutputStream)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#copy(java.lang.Readable, java.lang.Appendable)'><code>long copy(Readable, Appendable)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#readFully(java.io.InputStream, byte[])'><code>void readFully(InputStream, byte[])</code></a> | N/A                                      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#skipFully(java.io.InputStream, long)'><code>void skipFully(InputStream, long)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#skipFully(java.io.Reader, long)'><code>void skipFully(Reader, long)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#nullOutputStream()'><code>OutputStream nullOutputStream()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#nullWriter()'><code>Writer nullWriter()</code></a> |

#### A note on `InputSupplier` and `OutputSupplier`

Many of the methods in `ByteStreams`, `CharStreams` and other classes in the `common.io` package still use the `InputSupplier` and `OutputSupplier` interfaces. These interfaces and the methods that use them are deprecated: they are being replaced by the source and sink types described below and will eventually be removed.

# Sources and sinks

It's common to create I/O utility methods that help you to avoid dealing with streams at all when doing basic operations. For example, Guava has `Files.toByteArray(File)` and `Files.write(File, byte[])`. However, you end up with similar methods scattered all over, each dealing with a different kind of _source_ of data or _sink_ to which data can be written. For example, Guava has `Resources.toByteArray(URL)` which does the same thing as `Files.toByteArray(File)`, but using a `URL` as the source of data rather than a file.

To address this, Guava has a set of abstractions over different types of data sources and sinks. A source or sink is a resource of some sort that you know how to open a new stream to, such as a `File` or `URL`. Sources are readable, while sinks are writable. Additionally, sources and sinks are broken down according to whether you are dealing with `byte` or `char` data.

|             | **Bytes**                                | **Chars**                                |
| :---------- | :--------------------------------------- | :--------------------------------------- |
| **Reading** | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html'><code>ByteSource</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html'><code>CharSource</code></a> |
| **Writing** | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html'><code>ByteSink</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html'><code>CharSink</code></a> |

The advantage of these APIs is that they provide a common set of operations. Once you've wrapped your data source as a `ByteSource`, for example, you get the same set of methods no matter what that source happens to be.

### Creating sources and sinks

Guava provides a number of source and sink implementations:

| **Bytes**                                | **Chars**                                |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asByteSource(java.io.File)'><code>Files.asByteSource(File)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asCharSource(java.io.File, java.nio.charset.Charset)'><code>Files.asCharSource(File, Charset)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asByteSink(java.io.File, com.google.common.io.FileWriteMode...)'><code>Files.asByteSink(File, FileWriteMode...)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asCharSink(java.io.File, java.nio.charset.Charset, com.google.common.io.FileWriteMode...)'><code>Files.asCharSink(File, Charset, FileWriteMode...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Resources.html#asByteSource(java.net.URL)'><code>Resources.asByteSource(URL)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Resources.html#asCharSource(java.net.URL, java.nio.charset.Charset)'><code>Resources.asCharSource(URL, Charset)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#wrap(byte[])'><code>ByteSource.wrap(byte[])</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#wrap(java.lang.CharSequence)'><code>CharSource.wrap(CharSequence)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#concat(com.google.common.io.ByteSource...)'><code>ByteSource.concat(ByteSource...)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#concat(com.google.common.io.CharSource...)'><code>CharSource.concat(CharSource...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#slice(long, long)'><code>ByteSource.slice(long, long)</code></a> | N/A                                      |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#asCharSource(java.nio.charset.Charset)'><code>ByteSource.asCharSource(Charset)</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#asCharSink(java.nio.charset.Charset)'><code>ByteSink.asCharSink(Charset)</code></a> |

In addition, you can extend the source and sink classes yourself to create new implementations.

**Note:** While it can be tempting to create a source or sink that wraps an _open_ stream (such as an `InputStream`), this should be avoided. Your source/sink should instead open a new stream each time its `openStream()` method is called. This allows the source or sink to control the full lifecycle of that stream and allows it to be usable multiple times rather that becoming unusable the first time any method on it is called. Additionally, if you're opening the stream before creating the source or sink you may still have to deal with ensuring that the stream is closed correctly if an exception is thrown elsewhere in your code, which defeats many of the advantages of using a source or sink in the first place.

### Using Sources and Sinks

Once you have a source or sink instance, you have access to a number of operations for reading or writing.

#### Common operations

All sources and sinks provide the ability to open a new stream for reading or writing. By default, other operations are all implemented by calling one of these methods to get a stream, doing something, and then ensuring that the stream is closed.

These methods are all named:

* `openStream()` - returns an `InputStream`, `OutputStream`, `Reader` or `Writer` depending on the type of source or sink.
* `openBufferedStream()` - returns an `InputStream`, `OutputStream`, `BufferedReader` or `Writer` depending on the type of source or sink. The returned stream is guaranteed to be buffered if necessary. For example, a source that reads from a byte array has no need for additional buffering in memory. This is why the methods do not return `BufferedInputStream` etc. except in the case of `BufferedReader`, because it defines the `readLine()` method.

#### Source operations

| **`ByteSource`**                         | **`CharSource`**                         |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#read()'><code>byte[] read()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#read()'><code>String read()</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#readLines()'><code>ImmutableList&lt;String&gt; readLines()</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#readFirstLine()'><code>String readFirstLine()</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#copyTo(com.google.common.io.ByteSink)'><code>long copyTo(ByteSink)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#copyTo(com.google.common.io.CharSink)'><code>long copyTo(CharSink)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#copyTo(java.io.OutputStream)'><code>long copyTo(OutputStream)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#copyTo(java.lang.Appendable)'><code>long copyTo(Appendable)</code> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#size()'><code>long size()</code></a> (in bytes) | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#length--'><code>long length()</code></a> (in chars) |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#isEmpty()'><code>boolean isEmpty()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#isEmpty()'><code>boolean isEmpty()</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#contentEquals(com.google.common.io.ByteSource)'><code>boolean contentEquals(ByteSource)</code></a> | N/A                                      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#hash(com.google.common.hash.HashFunction)'><code>HashCode hash(HashFunction)</code></a> | N/A                                      |

#### Sink operations

| **`ByteSink`**                           | **`CharSink`**                           |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#write(byte[])'><code>void write(byte[])</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#write(java.lang.CharSequence)'><code>void write(CharSequence)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#writeFrom(java.io.InputStream)'><code>long writeFrom(InputStream)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeFrom(java.lang.Readable)'><code>long writeFrom(Readable)</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeLines(java.lang.Iterable)'><code>void writeLines(Iterable&lt;? extends CharSequence&gt;)</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeLines(java.lang.Iterable, java.lang.String)'><code>void writeLines(Iterable&lt;? extends CharSequence&gt;, String)</code></a> |

### Examples

```java
// Read the lines of a UTF-8 text file
ImmutableList<String> lines = Files.asCharSource(file, Charsets.UTF_8)
    .readLines();

// Count distinct word occurrences in a file
Multiset<String> wordOccurrences = HashMultiset.create(
  Splitter.on(CharMatcher.WHITESPACE)
    .trimResults()
    .omitEmptyStrings()
    .split(Files.asCharSource(file, Charsets.UTF_8).read()));

// SHA-1 a file
HashCode hash = Files.asByteSource(file).hash(Hashing.sha1());

// Copy the data from a URL to a file
Resources.asByteSource(url).copyTo(Files.asByteSink(file));
```

# `Files`

In addition to methods for creating file sources and sinks, the `Files` class contains a number of convenience methods that you might be interested in.

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#createParentDirs(java.io.File)'><code>createParentDirs(File)</code></a> | Creates necessary but nonexistent parent directories of the file. |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#getFileExtension(java.lang.String)'><code>getFileExtension(String)</code></a> | Gets the file extension of the file described by the path. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#getNameWithoutExtension(java.lang.String)'><code>getNameWithoutExtension(String)</code></a> | Gets the name of the file with its extension removed |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#simplifyPath(java.lang.String)'><code>simplifyPath(String)</code></a> | Cleans up the path.  Not always consistent with your filesystem; test carefully! |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#fileTreeTraverser()'><code>fileTreeTraverser()</code></a> | Returns a `TreeTraverser` that can traverse file trees |
