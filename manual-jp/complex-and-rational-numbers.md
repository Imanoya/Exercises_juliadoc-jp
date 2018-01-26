<!-- Start -->

# Complex and Rational Numbers

> # 複素数と有理数

<!-- End -->
<!-- Start -->
Julia ships with predefined types representing both complex and rational numbers, and supports all standard [Mathematical Operations and Elementary Functions](@ref) on them.
> Juliaは、複素数と有理数の両方を表す定義済みの型とともに出荷され、標準の[Mathematical Operations and Elementary Functions](@ref)をサポートしています。
<!-- End -->
<!-- Start -->
[Conversion and Promotion](@ref conversion-and-promotion) are defined so that operations on any combination of predefined numeric types, whether primitive or composite, behave as expected.
> [変換とプロモーション](@ ref変換とプロモーション)は、プリミティブでもコンポジットでも、あらかじめ定義された数値型の任意の組み合わせに対する操作が期待どおりに動作するように定義されています。
<!-- End -->
<!-- Start -->

## Complex Numbers

## 複素数

<!-- End -->
<!-- Start -->
The global constant [`im`](@ref) is bound to the complex number *i*, representing the principal square root of -1.
> グローバル定数[`im`](@ref)は、-1の主平方根を表す複素数* i *に束縛されます。
<!-- End -->
<!-- Start -->
It was deemed harmful to co-opt the name `i` for a global constant, since it is such a popular index variable name.
> グローバル定数のために `i`という名前を選ぶことは、一般的なインデックス変数名であるため有害であるとみなされました。
<!-- End -->
<!-- Start -->
Since Julia allows numeric literals to be [juxtaposed with identifiers as coefficients](@ref man-numeric-literal-coefficients), this binding suffices to provide convenient syntax for complex numbers, similar to the traditional mathematical notation:-->
Juliaは数値リテラルを[識別子として係数と並置]することができるので(@ ref man-numeric-literal-coefficients)、この結合は複素数に便利な構文を提供するのに十分です。 伝統的な数学的表記法に似ています：

```jldoctest
julia> 1 + 2im
1 + 2im
```

<!-- Start -->
You can perform all the standard arithmetic operations with complex numbers:
> 複素数で標準的な算術演算をすべて実行できます。
<!-- End -->

```jldoctest
julia> (1 + 2im)*(2 - 3im)
8 + 1im

julia> (1 + 2im)/(1 - 2im)
-0.6 + 0.8im

julia> (1 + 2im) + (1 - 2im)
2 + 0im

julia> (-3 + 2im) - (5 - 1im)
-8 + 3im

julia> (-1 + 2im)^2
-3 - 4im

julia> (-1 + 2im)^2.5
2.7296244647840084 - 6.960664459571898im

julia> (-1 + 2im)^(1 + 1im)
-0.27910381075826657 + 0.08708053414102428im

julia> 3(2 - 5im)
6 - 15im

julia> 3(2 - 5im)^2
-63 - 60im

julia> 3(2 - 5im)^-1.0
0.20689655172413796 + 0.5172413793103449im
```

<!-- Start -->
The promotion mechanism ensures that combinations of operands of different types just work:
> プロモーションの仕組みによって、異なるタイプのオペランドの組み合わせが正しく動作することが保証されます。
<!-- End -->

```jldoctest
julia> 2(1 - 1im)
2 - 2im

julia> (2 + 3im) - 1
1 + 3im

julia> (1 + 2im) + 0.5
1.5 + 2.0im

julia> (2 + 3im) - 0.5im
2.0 + 2.5im

julia> 0.75(1 + 2im)
0.75 + 1.5im

julia> (2 + 3im) / 2
1.0 + 1.5im

julia> (1 - 3im) / (2 + 2im)
-0.5 - 1.0im

julia> 2im^2
-2 + 0im

julia> 1 + 3/4im
1.0 - 0.75im
```

<!-- Start -->
Note that `3/4im == 3/(4*im) == -(3/4*im)`, since a literal coefficient binds more tightly than division.
> リテラル係数は除算よりも厳密に束縛されるので、 `3/4im == 3/(4*im) == -(3/4*im)`に注意してください。
<!-- End -->

<!-- Start -->
Standard functions to manipulate complex values are provided:
> 複素数を操作するための標準関数が用意されています。
<!-- End -->


```jldoctest
julia> z = 1 + 2im
1 + 2im

julia> real(1 + 2im) # real part of z
1

julia> imag(1 + 2im) # imaginary part of z
2

julia> conj(1 + 2im) # complex conjugate of z
1 - 2im

julia> abs(1 + 2im) # absolute value of z
2.23606797749979

julia> abs2(1 + 2im) # squared absolute value
5

julia> angle(1 + 2im) # phase angle in radians
1.1071487177940904
```

<!-- Start -->
As usual, the absolute value ([`abs()`](@ref)) of a complex number is its distance from zero.
> いつものように、複素数の絶対値([`abs()`](@ ref))はゼロからの距離です。
<!-- End -->
<!-- Start -->
[`abs2()`](@ref) gives the square of the absolute value, and is of particular use for complex numbers where it avoids taking a square root.
> [`abs2()`](@ ref)は絶対値の二乗を与え、複素数の場合は特に平方根を避けるために使います。
<!-- End -->
<!-- Start -->
[`angle()`](@ref) returns the phase angle in radians (also known as the *argument* or *arg* function).
> [`angle()`](@ref)は位相角をラジアンで返します(*引数*または* arg *関数とも呼ばれます)。
<!-- End -->
<!-- Start -->
The full gamut of other [Elementary Functions](@ref) is also defined for complex numbers:
> 複素数に対しても、他の[基本関数](@ref)の全範囲が定義されています：
<!-- End -->

```jldoctest
julia> sqrt(1im)
0.7071067811865476 + 0.7071067811865475im

julia> sqrt(1 + 2im)
1.272019649514069 + 0.7861513777574233im

julia> cos(1 + 2im)
2.0327230070196656 - 3.0518977991518im

julia> exp(1 + 2im)
-1.1312043837568135 + 2.4717266720048188im

julia> sinh(1 + 2im)
-0.4890562590412937 + 1.4031192506220405im
```

<!-- Start -->
Note that mathematical functions typically return real values when applied to real numbers and complex values when applied to complex numbers.
> 数学関数は、複素数に適用されたときに実数と複素数値に適用されると、通常、実数値を返します。
<!-- End -->
<!-- Start -->
For example, [`sqrt()`](@ref) behaves differently when applied to `-1` versus `-1 + 0im` even though `-1 == -1 + 0im`:-->
> 例えば、 [`sqrt()`](@ref)は `-1 == -1 + 0im` であっても `-1 + 0im` に対して `-1` に適用されたときの動作が異なります。
<!-- End -->

```jldoctest
julia> sqrt(-1)
ERROR: DomainError:

#sqrt will only return a complex result if called with a complex argument.
#sqrtは、複雑な引数で呼び出された場合にのみ、複雑な結果を返します。
#Try sqrt(complex(x)).
#sqrt(complex(x))を試してください。
#Stacktrace:

 [1] sqrt(::Int64) at ./math.jl:447

julia> sqrt(-1 + 0im)
0.0 + 1.0im
```

<!-- Start -->
The [literal numeric coefficient notation](@ref man-numeric-literal-coefficients) does not work when constructing a complex number from variables.
> 変数から複素数を構成するときに[数値数値表記法] (@ref man-numeric-literal-coefficients) は機能しません。
<!-- End -->
<!-- Start -->
Instead, the multiplication must be explicitly written out:
> 代わりに、乗算を明示的に書き出す必要があります。
<!-- End -->

```jldoctest
julia> a = 1; b = 2; a + b*im
1 + 2im
```

<!-- Start -->
However, this is *not* recommended; Use the [`complex()`](@ref) function instead to construct a complex value directly from its real and imaginary parts:
> しかし、これは推奨されません。 実数部と虚数部から直接複素数値を構築する代わりに、[`complex()`] (@ref) 関数を使用してください：
<!-- End -->

```jldoctest
julia> a = 1; b = 2; complex(a, b)
1 + 2im
```

<!-- Start -->
This construction avoids the multiplication and addition operations.
> この構成は、乗算および加算演算を回避する。
<!-- End -->

<!-- Start -->
[`Inf`](@ref) and [`NaN`](@ref) propagate through complex numbers in the real and imaginary parts of a complex number as described in the [Special floating-point values](@ref) section:
> [`Inf`] (@ref) と [`NaN`] (@ref) は[特殊浮動小数点値]（@ref）セクションで説明した複素数の実数部と虚数部の複素数を伝播します。
<!-- End -->

```jldoctest
julia> 1 + Inf*im
1.0 + Inf*im

julia> 1 + NaN*im
1.0 + NaN*im
```
<!-- Start -->

# Rational Numbers
> # 有理数

<!-- End -->
<!-- Start -->
Julia has a rational number type to represent exact ratios of integers.
> Juliaには、整数の正確な比率を表す有理数型があります。
<!-- End -->
<!-- Start -->
Rationals are constructed using the [`//`](@ref) operator:
> 有理数は、[`//`] (@ref) 演算子を使って生成されます：
<!-- End -->

```jldoctest
julia> 2//3
2//3
```

<!-- Start -->
If the numerator and denominator of a rational have common factors, they are reduced to lowest terms such that the denominator is non-negative:
> 有理数の分子と分母が共通の要素を持つ場合、それらは分母が非負であるように最も低い項に還元されます。
<!-- End -->

```jldoctest
julia> 6//9
2//3

julia> -4//8
-1//2

julia> 5//-15
-1//3

julia> -4//-12
1//3
```

<!-- Start -->
This normalized form for a ratio of integers is unique, so equality of rational values can be tested by checking for equality of the numerator and denominator.
> 整数の比に対するこの正規化された形式は固有であるため、有理値の等価性は、分子と分母の等価性を調べることによってテストできます。
<!-- End -->
<!-- Start -->
The standardized numerator and denominator of a rational value can be extracted using the [`numerator()`](@ref) and [`denominator()`](@ref) functions:
> 有理値の標準化された分子と分母は、[`numerator()`] (@ref) と[`denominator()`] (@ref) 関数を使用して抽出することができます：
<!-- End -->

```jldoctest
julia> numerator(2//3)
2

julia> denominator(2//3)
3
```

<!-- Start -->
Direct comparison of the numerator and denominator is generally not necessary, since the standard arithmetic and comparison operations are defined for rational values:
> 標準的な算術演算と比較演算が有理値のために定義されているので、分子と分母の直接比較は一般に必要ではありません。
<!-- End -->


```jldoctest
julia> 2//3 == 6//9
true

julia> 2//3 == 9//27
false

julia> 3//7 < 1//2
true

julia> 3//4 > 2//3
true

julia> 2//4 + 1//6
2//3

julia> 5//12 - 1//4
1//6

julia> 5//8 * 3//12
5//32

julia> 6//5 / 10//7
21//25
```

<!-- Start -->
Rationals can be easily converted to floating-point numbers:
> 有理数は簡単に浮動小数点数に変換することができます。
<!-- End -->

```jldoctest
julia> float(3//4)
0.75
```

<!-- Start -->
Conversion from rational to floating-point respects the following identity for any integral values of `a` and `b`, with the exception of the case `a == 0` and `b == 0`:-->
有理式から浮動小数点への変換は、'a == 0' および 'b == 0' の場合を除いて、'a' および 'b' の整数値に対する以下の同一性を尊重する。


```jldoctest
julia> a = 1; b = 2;

julia> isequal(float(a//b), a/b)
true
```

<!-- Start -->
Constructing infinite rational values is acceptable:
> 無限の合理的な値を構築することは許容されます：
<!-- End -->

```jldoctest
julia> 5//0
1//0

julia> -3//0
-1//0

julia> typeof(ans)
Rational{Int64}
```

<!-- Start -->
Trying to construct a [`NaN`](@ref) rational value, however, is not:
> しかし、[`NaN`] (@ref) 有理値を構築しようとすると、以下のようになりません：
<!-- End -->

```jldoctest
julia> 0//0
ERROR: ArgumentError: invalid rational: zero(Int64)//zero(Int64)
Stacktrace:
 [1] Rational{Int64}(::Int64, ::Int64) at ./rational.jl:13
 [2] //(::Int64, ::Int64) at ./rational.jl:40
```

<!-- Start -->
As usual, the promotion system makes interactions with other numeric types effortless:
> いつものように、プロモーションシステムは他の数値型とのやりとりを楽にします：
<!-- End -->

```jldoctest
julia> 3//5 + 1
8//5

julia> 3//5 - 0.5
0.09999999999999998

julia> 2//7 * (1 + 2im)
2//7 + 4//7*im

julia> 2//7 * (1.5 + 2im)
0.42857142857142855 + 0.5714285714285714im

julia> 3//2 / (1 + 2im)
3//10 - 3//5*im

julia> 1//2 + 2im
1//2 + 2//1*im

julia> 1 + 2//3im
1//1 - 2//3*im

julia> 0.5 == 1//2
true

julia> 0.33 == 1//3
false

julia> 0.33 < 1//3
true

julia> 1//3 - 0.33
0.0033333333333332993
```
