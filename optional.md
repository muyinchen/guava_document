
##Optional

大多数情况下，开发人员使用 ``null `  ` 表明的是某种缺失情形：可能是已经有一个默认值，或没有值，或找不到
值。例如，Map.get 返回 ``null `  ` 就表示找不到给定键对应的值。
Guava 用 `Optional<T>`表示可能为 ``null ` 的 T 类型引用。一个 Optional 实例可能包含非 `null `   T类型 的引用（我们称之为引用存在），也可能什么也不包括（称之为引用缺失）。它从不说包含的是 `null `   值，而是用存在或缺失来表示。但
Optional 从不会包含 `null `   值引用。

```
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5
```
`Optional` 无意直接模拟其他编程环境中的”可选” or “可能”语义，但它们的确有相似之处。
`Optional` 最常用的一些操作被罗列如下：

### 创建 Optional 实例（以下都是`Optional`中的静态方法）：

| Method                                   | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| [`Optional.of(T)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#of(T)) | 如果` Optional `包含非 `null `   的引用（引用存在），返回true |
| [`Optional.absent()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#absent()) | 返回 `Optional` 所包含的引用，若引用缺失，则抛出 `java.lang.IllegalStateExceptionT ` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#or(T)'><code>T or(T)</code></a> | 返回 Optional 所包含的引用，若引用缺失，返回指定的值          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#orNull()'><code>T orNull()</code></a> | 返回 Optional 所包含的引用，若引用缺失，返回 `null `      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#asSet()'><code>Set&lt;T&gt; asSet()</code></a> | 返回 `Optional` 所包含引用的单例不可变集，如果引用存在，返回一个只有单一元素的集合，如果引用缺失，返回一个空集合。 |

不止这些，`Optional`提供了更多便利的工具方法; 详情翻阅javadoc




### 用 Optional 实例查询引用（以下都是非静态方法）：

| Method                                   | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| [`boolean isPresent()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#isPresent()) | 如果` Optional `包含非 `null `   的引用（引用存在），返回true |
| [`T get()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#get()) | 返回 `Optional` 所包含的引用，若引用缺失，则抛出 `java.lang.IllegalStateExceptionT ` |
| [`T or(T)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#or(T)) | 返回 Optional 所包含的引用，若引用缺失，返回指定的值          |
| [`T orNull()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#orNull()) | 返回 Optional 所包含的引用，若引用缺失，返回 `null `      |
| [`Set asSet()`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#asSet()) | 返回 `Optional` 所包含引用的单例不可变集，如果引用存在，返回一个只有单一元素的集合，如果引用缺失，返回一个空集合。 |

不止这些，`Optional`提供了更多便利的工具方法; 详情翻阅javadoc

###使用 Optional 的意义在哪儿？

使用` Optional `除了赋予 `null `   语义，增加了可读性，最大的优点在于它是一种傻瓜式的防护。如果你想让你的程序完整编译 `Optional` 迫使你积极思考引用缺失的情况，因为你必须显式地从 Optional 获取引用。直接使用 `null `   很容易让人忘掉某些情形，尽管 FindBugs 可以帮助查找 `null `   相关的问题，但是我们还是认为它并不能准确地定位问题根源。

如同输入参数，方法的返回值也可能是 `null `  。和其他人一样，你绝对很可能会忘记别人写的方法 `method(a,b)`会
返回一个 `null `  ，就好像当你实现` method(a,b)`时，也很可能忘记输入参数 a 可以为 `null `  。将方法的返回类型指定
为 `Optional`，使调用者不可能忘记这种情况， 因为他们必须亲自“抽取”出对象来使他们的代码通过编译。

## 处理 `null `   的便利方法

当你需要用一个默认值来替换可能的 `null `  ，请使用`Objects.firstNonnull(T, T)` 方法。根据方法名所示，如果两个值都是 `null `  ，该方法会很快失败并抛出 `null PointerException`  。`Optional `也是一个比较好的替代方案，例如：`first.or(second)`.

在`Strings` 中还有其它一些方法专门处理 `null `   或空字符串：

| Method                                   |
| :--------------------------------------- |
| [`emptyToNull(String)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Strings.html#emptyToNull(java.lang.String)) |
| [`isNullOrEmpty(String)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Strings.html#isNullOrEmpty(java.lang.String)) |
| [`nullToEmpty(String)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Strings.html#nullToEmpty(java.lang.String)) |

我们想要强调的是，这些方法主要用来与混淆 `null `  /空的 API 进行交互。当每次你写下混淆 `null `  /空的
代码时，Guava 团队都泪流满面。（好的做法是积极地把 `null `   和空的含义区分开，以表示不同的含义，在代码中把 `null `  和空同等对待是一种令人不安的臭代码。