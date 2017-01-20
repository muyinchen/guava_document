# Example
```java

int logFloor = LongMath.log2(n, FLOOR);

int mustNotOverflow = IntMath.checkedMultiply(x, y);

long quotient = LongMath.divide(knownMultipleOfThree, 3, RoundingMode.UNNECESSARY); // fail fast on non-multiple of 3

BigInteger nearestInteger = DoubleMath.roundToBigInteger(d, RoundingMode.HALF_EVEN);

BigInteger sideLength = BigIntegerMath.sqrt(area, CEILING);
```

# Why?
* Guava's math utilities are already exhaustively tested for unusual overflow conditions.  Overflow semantics, if relevant, are clearly specified in the associated documentation.  When a precondition fails, it fails fast.
* Guava's math utilities have been painstakingly benchmarked and optimized.  While performance inevitably varies depending on particular hardware details, their speed is competitive with -- and in some cases, significantly better than -- analogous functions in Apache Commons `MathUtils`.
    * Guava's math utilities are designed to encourage readable, correct programming habits.  The meaning of `IntMath.log2(x, CEILING)` is unambiguous and obvious even on a casual read-through.  The meaning of `32 - Integer.numberOfLeadingZeros(x - 1)` is not.

_Note: Guava's math utilities are not especially compatible with GWT, nor are they optimized for GWT, due to differing overflow logic._

# Math on Integral Types
Guava's math utilities deal primarily with three integral types: `int`, `long`, and `BigInteger`.  The math utilities on these types are conveniently named [IntMath](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html), [LongMath](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html), and [BigIntegerMath](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html).

## Checked Arithmetic
Guava provides arithmetic methods for `IntMath` and `LongMath` that fail fast on overflow instead of silently ignoring it.

| `IntMath`                                | `LongMath`                               |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedAdd(int, int)'><code>IntMath.checkedAdd</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedAdd(long, long)'><code>LongMath.checkedAdd</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedSubtract(int, int)'><code>IntMath.checkedSubtract</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedSubtract(long, long)'><code>LongMath.checkedSubtract</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedMultiply(int, int)'><code>IntMath.checkedMultiply</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedMultiply(long, long)'><code>LongMath.checkedMultiply</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedPow(int, int)'><code>IntMath.checkedPow</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedPow(long, long)'><code>LongMath.checkedPow</code></a> |

```java

IntMath.checkedAdd(Integer.MAX_VALUE, Integer.MAX_VALUE); // throws ArithmeticException
```

# Real-valued methods

`IntMath`, `LongMath`, and `BigIntegerMath` have support for a variety of methods with a "precise real value," but that round their result to an integer. These methods accept a [java.math.RoundingMode](http://docs.oracle.com/javase/7/docs/api/java/math/RoundingMode.html).  This is the same `RoundingMode` used in the JDK, and is an enum with the following values:
* `DOWN`: round towards 0.  (This is the behavior of Java division.)
* `UP`: round away from 0.
    * `FLOOR`: round towards negative infinity.
    * `CEILING`: round towards positive infinity.
    * `UNNECESSARY`: rounding should not be necessary; if it is, fail fast by throwing an `ArithmeticException`.
    * `HALF_UP`: round to the nearest half, rounding `x.5` away from 0.
    * `HALF_DOWN`: round to the nearest half, rounding `x.5` towards 0.
    * `HALF_EVEN`: round to the nearest half, rounding `x.5` to its nearest even neighbor.

These methods are meant to be readable when used: for example, `divide(x, 3, CEILING)` is completely unambiguous even on a casual read-through.

Additionally, each of these functions internally use only integer arithmetic, except in constructing initial approximations for use in `sqrt`.

| Operation         | `IntMath`                                | `LongMath`                               | `BigIntegerMath`                         |
| :---------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| Division          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#divide(int, int, java.math.RoundingMode)'><code>divide(int, int, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#divide(long, long, java.math.RoundingMode)'><code>divide(long, long, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#divide(java.math.BigInteger, java.math.BigInteger, java.math.RoundingMode)'><code>divide(BigInteger, BigInteger, RoundingMode)</code></a> |
| Base-2 logarithm  | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#log2(int, java.math.RoundingMode)'><code>log2(int, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#log2(long, java.math.RoundingMode)'><code>log2(long, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#log2(java.math.BigInteger, java.math.RoundingMode)'><code>log2(BigInteger, RoundingMode)</code></a> |
| Base-10 logarithm | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#log10(int, java.math.RoundingMode)'><code>log10(int, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#log10(long, java.math.RoundingMode)'><code>log10(long, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#log10(java.math.BigInteger, java.math.RoundingMode)'><code>log10(BigInteger, RoundingMode)</code></a> |
| Square root       | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#sqrt(int, java.math.RoundingMode)'><code>sqrt(int, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#sqrt(long, java.math.RoundingMode)'><code>sqrt(long, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#sqrt(java.math.BigInteger, java.math.RoundingMode)'><code>sqrt(BigInteger, RoundingMode)</code></a> |

```java
BigIntegerMath.sqrt(BigInteger.TEN.pow(99), RoundingMode.HALF_EVEN);
   // returns 31622776601683793319988935444327185337195551393252
```

## Additional functions
We provide support for a few other mathematical functions we've found useful.

| Operation                                | `IntMath`                                | `LongMath`                               | `BigIntegerMath`                         |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| Greatest common divisor                  | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#gcd(int, int)'><code>gcd(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#gcd(long, long)'><code>gcd(long, long)</code></a> | In JDK: <a href='http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#gcd(java.math.BigInteger)'><code>BigInteger.gcd(BigInteger)</code></a> |
| Modulus (always nonnegative, -5 mod 3 is 1) | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#mod(int, int)'><code>mod(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#mod(long, long)'><code>mod(long, long)</code></a> | In JDK: <a href='http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#mod(java.math.BigInteger)'><code>BigInteger.mod(BigInteger)</code></a> |
| Exponentiation (may overflow)            | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#pow(int, int)'><code>pow(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#pow(long, int)'><code>pow(long, int)</code></a> | In JDK: <a href='http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#pow(int)'><code>BigInteger.pow(int)</code></a> |
| Power-of-two testing                     | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#isPowerOfTwo(int)'><code>isPowerOfTwo(int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#isPowerOfTwo(long)'><code>isPowerOfTwo(long)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#isPowerOfTwo(java.math.BigInteger)'><code>isPowerOfTwo(BigInteger)</code></a> |
| Factorial (returns `MAX_VALUE` if input too big) | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#factorial(int)'><code>factorial(int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#factorial(int)'><code>factorial(int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#factorial(int)'><code>factorial(int)</code></a> |
| Binomial coefficient (returns `MAX_VALUE` if too big) | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#binomial(int, int)'><code>binomial(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#binomial(int, int)'><code>binomial(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#binomial(int, int)'><code>binomial(int, int)</code></a> |

# Floating-point arithmetic
Floating point arithmetic is pretty thoroughly covered by the JDK, but we added a few useful methods to [DoubleMath](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html).

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#isMathematicalInteger(double)'><code>isMathematicalInteger(double)</code></a> | Tests if the input is finite and an exact integer. |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToInt(double, java.math.RoundingMode)'><code>roundToInt(double, RoundingMode)</code></a> | Rounds the specified number and casts it to an int, if it fits into an int, failing fast otherwise. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToLong(double, java.math.RoundingMode)'><code>roundToLong(double, RoundingMode)</code></a> | Rounds the specified number and casts it to a long, if it fits into a long, failing fast otherwise. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToBigInteger(double, java.math.RoundingMode)'><code>roundToBigInteger(double, RoundingMode)</code></a> | Rounds the specified number to a `BigInteger`, if it is finite, failing fast otherwise. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#log2(double, java.math.RoundingMode)'><code>log2(double, RoundingMode)</code></a> | Takes the base-2 logarithm, and rounds to an `int` using the specified `RoundingMode`.  Faster than `Math.log(double)`. |
