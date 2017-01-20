## 常见 Object 方法

###equals 



当一个对象中的字段可以为 `null` 时，实现 `Object.equals` 方法会很痛苦，因为不得不分别对它们进行` null` 检
查。使用 `Objects.equal`  以一 种`null` 敏感的方式帮助你执行 `equals` 判断，从而避免抛出 `NullPointerException`。例如:

```java

Objects.equal("a", "a"); // returns true
Objects.equal(null, "a"); // returns false
Objects.equal("a", null); // returns false
Objects.equal(null, null); // returns true
```


注意：JDK7 引入的 Objects 类提供了一样的方法 Objects.equals。

### hashCode

用对象的所有字段作散列[hash]运算应当更简单。Guava 的 `Objects.hashCode(Object...)`会对传入的字段序
列计算出合理的、顺序敏感的散列值。你可以使用 `Objects.hashCode(field1, field2, …, fieldn)`来代替手动计算
散列值。
注意：JDK7 引入的 Objects 类提供了一样的方法 `Objects.hash(Object...)`

### toString

好的	` toString` 方法在调试时是无价之宝，但是编写 toString 方法有时候却很痛苦。使用 `Objects.toStringHelper`可以轻松编写有用的` toString `方法。例如：

```java

// Returns "ClassName{x=1}"
   MoreObjects.toStringHelper(this)
       .add("x", 1)
       .toString();

   // Returns "MyObject{x=1}"
   MoreObjects.toStringHelper("MyObject")
       .add("x", 1)
       .toString();
```


实现一个比较器`Comparator`，或者直接实现 `Comparable` 接口有时也伤不起。考虑一下这种情况：

```java

class Person implements Comparable<Person> {
  private String lastName;
  private String firstName;
  private int zipCode;

  public int compareTo(Person other) {
    int cmp = lastName.compareTo(other.lastName);
    if (cmp != 0) {
      return cmp;
    }
    cmp = firstName.compareTo(other.firstName);
    if (cmp != 0) {
      return cmp;
    }
    return Integer.compare(zipCode, other.zipCode);
  }
}
```


这部分代码太琐碎了，因此很容易搞乱，也很难调试。我们应该能把这种代码变得更优雅，为此，Guava 提供了
`ComparisonChain`。
`ComparisonChain `执行一种懒比较：它执行比较操作直至发现非零的结果，在那之后的比较输入将被忽略。

```java
   public int compareTo(Foo that) {
     return ComparisonChain.start()
         .compare(this.aString, that.aString)
         .compare(this.anInt, that.anInt)
         .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
         .result();
   }
```


这种 Fluent 接口风格的可读性更高，发生错误编码的几率更小，并且能避免做不必要的工作。更多 Guava 排序
器工具可以在下一节里找到。 













