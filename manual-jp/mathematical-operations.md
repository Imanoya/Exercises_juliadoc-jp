<!-- Start -->

# Mathematical Operations and Elementary Functions

> # 数学演算と基本関数

<!-- End -->
<!-- Start -->
Julia provides a complete collection of basic arithmetic and bitwise operators across all of its numeric primitive types, as well as providing portable, efficient implementations of a comprehensive collection of standard mathematical functions.
> Juliaは、すべての数値プリミティブ型の基本的な算術演算とビット演算子の完全なコレクションを提供し、標準的な数学関数の包括的なコレクションのポータブルで効率的な実装を提供します。
<!-- End -->

<!-- Start -->
## Arithmetic Operators

> ##算術演算子

<!-- End -->
<!-- Start -->
The following [arithmetic operators](https://en.wikipedia.org/wiki/Arithmetic#Arithmetic_operations) are supported on all primitive numeric types:
> 次の [算術演算子] (https://en.wikipedia.org/wiki/Arithmetic#Arithmetic_operations)は、すべてのプリミティブな数値型でサポートされています。
<!-- End -->

| Expression | Name           | Description                            |
|:---------- |:-------------- |:-------------------------------------- |
| `+x`       | unary plus     | the identity operation                 |
| `-x`       | unary minus    | maps values to their additive inverses |
| `x + y`    | binary plus    | performs addition                      |
| `x - y`    | binary minus   | performs subtraction                   |
| `x * y`    | times          | performs multiplication                |
| `x / y`    | divide         | performs division                      |
| `x \ y`    | inverse divide | equivalent to `y / x`                  |
| `x ^ y`    | power          | raises `x` to the `y`th power          |
| `x % y`    | remainder      | equivalent to `rem(x,y)`               |

<!-- Start -->
as well as the negation on [`Bool`](@ref) types:
> [`Bool`](@ref) の型の否定と同様に、
<!-- End -->

| Expression | Name     | Description                              |
|:---------- |:-------- |:---------------------------------------- |
| `!x`       | negation | changes `true` to `false` and vice versa |

<!-- Start -->
Julia's promotion system makes arithmetic operations on mixtures of argument types "just work" naturally and automatically. 
> Juliaのプロモーションシステムは、引数型の混合物の算術演算を自動的かつ自動的に「うまくいく」ようにします。
<!-- End -->
<!-- Start -->
See [Conversion and Promotion](@ref conversion-and-promotion) for details of the promotion system.
> [変換とプロモーション] (@ref conversion-and-promotion)で、プロモーションシステムの詳細をご覧いただけます。
<!-- End -->

<!-- Start -->
Here are some simple examples using arithmetic operators:
> 算術演算子を使用した簡単な例を次に示します。
<!-- End -->

```jldoctest
julia> 1 + 2 + 3
6

julia> 1 - 2
-1

julia> 3*2/12
0.5
```

<!-- Start -->
(By convention, we tend to space operators more tightly if they get applied before other nearby operators. 
> (仕様により、近くの他のオペレータの前に適用されると、オペレータをより緊密に配置する傾向があります。
<!-- End -->
<!-- Start -->
For instance, we would generally write `-x + 2` to reflect that first `x` gets negated, and then `2` is added to that result.)
> たとえば、最初に `x`がネゲートされ、その結果に` 2`が追加されることを反映するために `-x + 2`と書くことにします。
<!-- End -->

<!-- Start -->
## Bitwise Operators

> ##ビット演算子

<!-- End -->
<!-- Start -->
The following [bitwise operators](https://en.wikipedia.org/wiki/Bitwise_operation#Bitwise_operators) are supported on all primitive integer types:
> 次の[ビット単位の演算子](https://en.wikipedia.org/wiki/Bitwise_operation#Bitwise_operators)は、すべてのプリミティブ型でサポートされています。
<!-- End -->

| Expression | Name                                                                     |
|:---------- |:------------------------------------------------------------------------ |
| `~x`       | bitwise not                                                              |
| `x & y`    | bitwise and                                                              |
| `x \| y`   | bitwise or                                                               |
| `x ⊻ y`    | bitwise xor (exclusive or)                                               |
| `x >>> y`  | [logical shift](https://en.wikipedia.org/wiki/Logical_shift) right       |
| `x >> y`   | [arithmetic shift](https://en.wikipedia.org/wiki/Arithmetic_shift) right |
| `x << y`   | logical/arithmetic shift left                                            |

<!-- Start -->
Here are some examples with bitwise operators:
> ビット演算子の例をいくつか示します：
<!-- End -->

```jldoctest
julia> ~123
-124

julia> 123 & 234
106

julia> 123 | 234
251

julia> 123 ⊻ 234
145

julia> xor(123, 234)
145

julia> ~UInt32(123)
0xffffff84

julia> ~UInt8(123)
0x84
```

<!-- Start -->

## Updating operators

> ## 更新演算子

<!-- End -->
<!-- Start -->
Every binary arithmetic and bitwise operator also has an updating version that assigns the result of the operation back into its left operand. 
> 全てのバイナリ算術演算子とビット演算子には、左のオペランドへ演算の結果を更新するバージョンがあります。
<!-- End -->
<!-- Start -->
The updating version of the binary operator is formed by placing a `=` immediately after the operator. 
> バイナリ演算子の更新版は、演算子の直後に `=' を置くことによって形成されます。
<!-- End -->
<!-- Start -->
For example, writing `x += 3` is equivalent to writing `x = x + 3`:
> 例えば、 `x += 3` を書くことは `x = x + 3`と書くことと同じです：
<!-- End -->

```jldoctest
julia> x = 1
1

julia> x += 3
4

julia> x
4
```

<!-- Start -->
The updating versions of all the binary arithmetic and bitwise operators are:
> すべてのバイナリ算術演算子とビット演算子の更新バージョンは次のとおりです。
<!-- End -->

```
+=  -=  *=  /=  \=  ÷=  %=  ^=  &=  |=  ⊻=  >>>=  >>=  <<=
```

<!-- Start -->
!!! note
<!-- End -->
  <!-- Start -->
  An updating operator rebinds the variable on the left-hand side. As a result, the type of the variable may change.
  > 更新演算子は、変数を左側に再バインドします。 その結果、変数の型が変更される可能性があります。
  <!-- End -->

  ```jldoctest
  julia> x = 0x01; typeof(x)
  UInt8

  julia> x *= 2 # Same as x = x * 2
  2

  julia> typeof(x)
  Int64
  ```

<!-- Start -->

## [Vectorized "dot" operators](@id man-dot-operators)

> ## [ベクトル化された "ドット"演算子](@id man-dot-operators)

<!-- End -->
<!-- Start -->
For *every* binary operation like `^`, there is a corresponding "dot" operation `.^` that is *automatically* defined to perform `^` element-by-element on arrays. 
> `^` のような *バイナリ演算のたび* に、配列に要素ごとに `^`を実行するために *自動的に* 定義された対応する "ドット"演算 `.^`があります。
<!-- End -->
<!-- Start -->
For example, `[1,2,3] ^ 3` is not defined, since there is no standard mathematical meaning to "cubing" an array, but `[1,2,3] .^ 3` is defined as computing the elementwise (or "vectorized") result `[1^3, 2^3, 3^3]`.
> 例えば、 `[1,2,3] ^ 3`は定義されていません。なぜなら、配列を「3乗」するための標準的な数学的な意味はないからですが、`[1,2,3].^3`は、 要素ごとの(またはベクトル化された)結果 `[1^3,2^3,3^3]`を計算することとして定義されます。
<!-- End -->
<!-- Start -->
Similarly for unary operators like `!` or `√`, there is a corresponding `.√` that applies the operator elementwise.
> 同様に、 `!`や `√`のような単項演算子の場合、対応する`.√`があり、これは演算子を要素ごとに適用します。
<!-- End -->

```jldoctest
julia> [1,2,3] .^ 3
3-element Array{Int64,1}:
  1
  8
 27
```

<!-- Start -->
More specifically, `a .^ b` is parsed as the ["dot" call](@ref man-vectorized) `(^).(a,b)`, which performs a [broadcast](@ref Broadcasting) operation:
> より具体的には、`a .^ b` は [ブロードキャスト](@ref Broadcasting)操作を実行する["dot" call](@ ref man-vectorized) `(^).(a,b)` として解析されます。 ：
<!-- End -->
<!-- Start -->
it can combine arrays and scalars, arrays of the same size (performing the operation elementwise), and even arrays of different shapes (e.g. combining row and column vectors to produce a matrix). 
> 配列とスカラ、同じ大きさの配列(要素単位の演算を実行する)、さらには異なる形状の配列(行列を生成するために行ベクトルと列ベクトルを組み合わせるなど)を組み合わせることができます。
<!-- End -->
<!-- Start -->
Moreover, like all vectorized "dot calls," these "dot operators" are *fusing*. 
> さらに、すべてのベクトル化された"dot calls"と同様に、これらの"ドット演算子"は*融合*です。
<!-- End -->
<!-- Start -->
For example, if you compute `2 .* A.^2 .+ sin.(A)` (or equivalently `@. 2A^2 + sin(A)`, using the [`@.`](@ref @__dot__) macro) for an array `A`, it performs a *single* loop over `A`, computing `2a^2 + sin(a)` for each element of `A`. 
> 例えば、 ``。*。^ 2。+ sin(A) `(あるいはそれに相当する` @。2A ^ 2 + sin(A) `) __dot__)マクロ)を配列 `A`に渡すと、Aの各要素に対して` 2a ^ 2 + sin(a) `を計算して` A`に*単一ループを実行します。
<!-- End -->
<!-- Start -->
In particular, nested dot calls like `f.(g.(x))` are fused, and "adjacent" binary operators like `x .+ 3 .* x.^2` are equivalent to nested dot calls `(+).(x, (*).(3, (^).(x, 2)))`.
> 特に、 `f。(g。(x))`のようなネストされたドットコールは融合され、 `x。+ 3。* x。^ 2`のような「隣接」バイナリ演算子はネストされたドットコール` 。(x、(*)。(3、(^)。(x、2))) `。
<!-- End -->

<!-- Start -->
Furthermore, "dotted" updating operators like `a .+= b` (or `@. a += b`) are parsed as `a .= a .+ b`, where `.=` is a fused *in-place* assignment operation (see the [dot syntax documentation](@ref man-vectorized)).
> さらに、 `。= b`(または` @ .a + = b`)のような点線の更新演算子は、 `。= a。+ b`として解析されます。ここで`。= `はfused * place * assignmentオペレーション([dot syntax documentation](@ ref man-vectorized)を参照してください)。
<!-- End -->

<!-- Start -->
Note the dot syntax is also applicable to user-defined operators.
> ドット構文はユーザー定義演算子にも適用できます。
For example, if you define `⊗(A,B) = kron(A,B)` to give a convenient infix syntax `A ⊗ B` for Kronecker products ([`kron`](@ref)), then `[A,B] .⊗ [C,D]` will compute `[A⊗C, B⊗D]` with no additional coding.
> たとえば、Kroneckerの製品([`` kron`](@ ref))に便利なインフィックス構文 `A⊗B`を与えるために`⊗(A、B)= kron(A、B) `を定義すると、` [ A、B].⊗[C、D] `は、追加のコーディングなしで[A⊗C、B⊗D]を計算します。
<!-- End -->

<!-- Start -->
Combining dot operators with numeric literals can be ambiguous.
> ドット演算子と数値リテラルを組み合わせると、あいまいになる可能性があります。
<!-- End -->
<!-- Start -->
For example, it is not clear whether `1.+x` means `1. + x` or `1 .+ x`.
> 例えば、`1.+x`の意味は `1. + x` または `1 .+ x`かどうかはっきりしません。
<!-- End -->
<!-- Start -->
Therefore this syntax is disallowed, and spaces must be used around the operator in such cases.
> したがって、この構文は許されません、この場合オペレータの前後へスペースを使用して下さい。
<!-- End -->

<!-- Start -->

## Numeric Comparisons

> ## 数値の比較

<!-- End -->
<!-- Start -->
Standard comparison operations are defined for all the primitive numeric types:
> 標準比較演算は、すべてのプリミティブな数値型に対して定義されています。
<!-- End -->

| Operator                     | Name                     |
|:---------------------------- |:------------------------ |
| [`==`](@ref)                 | equality                 |
| [`!=`](@ref), [`≠`](@ref !=) | inequality               |
| [`<`](@ref)                  | less than                |
| [`<=`](@ref), [`≤`](@ref <=) | less than or equal to    |
| [`>`](@ref)                  | greater than             |
| [`>=`](@ref), [`≥`](@ref >=) | greater than or equal to |

<!-- Start -->
Here are some simple examples:
> ここにいくつかの簡単な例があります：
<!-- End -->

```jldoctest
julia> 1 == 1
true

julia> 1 == 2
false

julia> 1 != 2
true

julia> 1 == 1.0
true

julia> 1 < 2
true

julia> 1.0 > 3
false

julia> 1 >= 1.0
true

julia> -1 <= 1
true

julia> -1 <= -1
true

julia> -1 <= -2
false

julia> 3 < -0.5
false
```

<!-- Start -->
Integers are compared in the standard manner -- by comparison of bits. 
> 整数は、ビットの比較によって標準的な方法で比較されます。
<!-- End -->
<!-- Start -->
Floating-point numbers are compared according to the [IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754-2008):
> 浮動小数点数は、[IEEE 754標準](https://en.wikipedia.org/wiki/IEEE_754-2008)に従って比較されます。
<!-- End -->

<!-- Start -->
* Finite numbers are ordered in the usual manner.
* > 有限数は通常の方法で順序付けられます。
<!-- End -->
<!-- Start -->
* Positive zero is equal but not greater than negative zero.
* > 正のゼロは等しいが負のゼロより大きくはない。
<!-- End -->
<!-- Start -->
* `Inf` is equal to itself and greater than everything else except `NaN`.
* > `Inf`はそれ自身と等しく、`NaN`を除いて他の全てよりも大きい。
<!-- End -->
<!-- Start -->
* `-Inf` is equal to itself and less then everything else except `NaN`.
* > `-Inf`はそれ自身と等しく、`NaN`を除いて他のものより小さい。
<!-- End -->
<!-- Start -->
* `NaN` is not equal to, not less than, and not greater than anything, including itself.
* > `NaN`は、自身を含め全てと、等しくなく、小さくなく、大きくない。
<!-- End -->

<!-- Start -->
The last point is potentially surprising and thus worth noting:
> 最後の点は潜在的に驚くべきことで、注意が必要です。
<!-- End -->

```jldoctest
julia> NaN == NaN
false

julia> NaN != NaN
true

julia> NaN < NaN
false

julia> NaN > NaN
false
```

<!-- Start -->
and can cause especial headaches with [Arrays](@ref):
> [配列](@ref)で特別な頭痛を引き起こす可能性があります：
<!-- End -->

```jldoctest
julia> [1 NaN] == [1 NaN]
false
```

<!-- Start -->
Julia provides additional functions to test numbers for special values, which can be useful in situations like hash key comparisons:
> Juliaは、特殊な値の数値をテストするための追加機能を、ハッシュキー比較のような状況で便利に使えるよう提供しています。
<!-- End -->

| Function                | Tests if                  |
|:----------------------- |:------------------------- |
| [`isequal(x, y)`](@ref) | `x` and `y` are identical |
| [`isfinite(x)`](@ref)   | `x` is a finite number    |
| [`isinf(x)`](@ref)      | `x` is infinite           |
| [`isnan(x)`](@ref)      | `x` is not a number       |

<!-- Start -->
[`isequal()`](@ref) considers `NaN`s equal to each other:
> [`isequal()`](@ref)は `NaN`が互いに等しいとみなします：
<!-- End -->

```jldoctest
julia> isequal(NaN, NaN)
true

julia> isequal([1 NaN], [1 NaN])
true

julia> isequal(NaN, NaN32)
true
```

<!-- Start -->
`isequal()` can also be used to distinguish signed zeros:
> `isequal()`は、符号付きの零の区別にも使うことができます：
<!-- End -->

```jldoctest
julia> -0.0 == 0.0
true

julia> isequal(-0.0, 0.0)
false
```

<!-- Start -->
Mixed-type comparisons between signed integers, unsigned integers, and floats can be tricky. 
> 符号付き整数と符号なし整数,浮動小数点数,どうしの比較は扱いにくいことがあります。
<!-- End -->
<!-- Start -->
A great deal of care has been taken to ensure that Julia does them correctly.
> Juliaは正しく行える事を保障するために、多くの努力がなされています。
<!-- End -->

<!-- Start -->
For other types, `isequal()` defaults to calling [`==()`](@ref), so if you want to define equality for your own types then you only need to add a [`==()`](@ref) method.  
> 他の型の `isequal()`はデフォルトで[`==()`](@ ref)を呼び出すため、 ](@ ref)メソッド。
<!-- End -->
<!-- Start -->
If you define your own equality function, you should probably define a corresponding [`hash()`](@ref) method to ensure that `isequal(x,y)` implies `hash(x) == hash(y)`.
> あなた自身の等価関数を定義するならば、 `isequal(x、y)`が `hash(x)== hash(y)`を意味するように対応する[`hash()`](@ ref) 。
<!-- End -->

<!-- Start -->
## Chaining comparisons

> ## 連鎖比較

<!-- End -->
<!-- Start -->
Unlike most languages, with the [notable exception of Python](https://en.wikipedia.org/wiki/Python_syntax_and_semantics#Comparison_operators), comparisons can be arbitrarily chained:
> ほとんどの言語とは異なり、[Pythonの注目すべき例外](https://en.wikipedia.org/wiki/Python_syntax_and_semantics#Comparison_operators)では、比較は任意に連鎖することができます：
<!-- End -->

```jldoctest
julia> 1 < 2 <= 2 < 3 == 3 > 2 >= 1 == 1 < 3 != 5
true
```

<!-- Start -->
Chaining comparisons is often quite convenient in numerical code. 
> 連鎖比較は、数値コードではしばしば便利です。
<!-- End -->
<!-- Start -->
Chained comparisons use the `&&` operator for scalar comparisons, and the [`&`](@ref) operator for elementwise comparisons, which allows them to work on arrays. 
> 連鎖比較は、スカラー比較のために `&&`演算子を使用し、配列での演算を可能にする要素間比較の[`＆`](@ref)演算子を使用します。
<!-- End -->
<!-- Start -->
For example, `0 .< A .< 1` gives a boolean array whose entries are true where the corresponding elements of `A` are between 0 and 1.
> 例えば、 `0 .< A .< 1`は、`A`の対応する要素が0と1の間で真であるブール値の配列を返します。
<!-- End -->

<!-- Start -->
Note the evaluation behavior of chained comparisons:
> 連鎖比較の評価動作に注意してください。
<!-- End -->

```jldoctest
julia> v(x) = (println(x); x)
v (generic function with 1 method)

julia> v(1) < v(2) <= v(3)
2
1
3
true

julia> v(1) > v(2) <= v(3)
2
1
false
```

<!-- Start -->
The middle expression is only evaluated once, rather than twice as it would be if the expression were written as `v(1) < v(2) && v(2) <= v(3)`. 
> 中間表現の評価は式が `v(1)<v(2) && v(2)<= v(3)`と書かれている場合、2回ではなく1回だけ評価されます。
<!-- End -->
<!-- Start -->
However, the order of evaluations in a chained comparison is undefined. 
> ただし、連鎖比較での評価の順序は未定義です。
<!-- End -->
<!-- Start -->
It is strongly recommended not to use expressions with side effects (such as printing) in chained comparisons. 
> 連鎖比較で副作用のある式(印刷など)を使用しないことを強く推奨します。
<!-- End -->
<!-- Start -->
If side effects are required, the short-circuit `&&` operator should be used explicitly (see [Short-Circuit Evaluation](@ref)).
> 副作用が必要な場合は、短絡の&&演算子を明示的に使用する必要があります([Short-Circuit Evaluation](@ref)を参照)。
<!-- End -->

<!-- Start -->

## Elementary Functions

> ## 初等関数

<!-- End -->
<!-- Start -->
Julia provides a comprehensive collection of mathematical functions and operators. 
> ジュリアは、数学的関数と演算子の包括的なコレクションを提供します。
<!-- End -->
<!-- Start -->
These mathematical operations are defined over as broad a class of numerical values as permit sensible definitions, including integers, floating-point numbers, rationals, and complex numbers, wherever such definitions make sense.
> これらの数学的演算は、整数、浮動小数点数、有理数、複素数などの許容可能な定義として、そのような定義が意味を成す限り、幅広い数値クラスとして定義されています。
<!-- End -->

<!-- Start -->
Moreover, these functions (like any Julia function) can be applied in "vectorized" fashion to arrays and other collections with the [dot syntax](@ref man-vectorized) `f.(A)`, e.g. `sin.(A)` will compute the sine of each element of an array `A`.
> さらに、これらの関数(任意のJulia関数のような)は、[dot syntax](@ref man-vectorized) `f.(A)`で配列や他のコレクションに "ベクトル化"された形で適用できます。 `sin(A)`は配列 `A`の各要素の正弦を計算します。
<!-- End -->

<!-- Start -->

## Operator Precedence

> ##演算子の優先順位

<!-- End -->
<!-- Start -->
Julia applies the following order of operations, from highest precedence to lowest:
> ジュリアは、優先順位の高いものから低いものまで、次の操作順序を適用します。
<!-- End -->

| Category       | Operators                                                                       |
|:-------------- |:------------------------------------------------------------------------------- |
| Syntax         | `.` followed by `::`                                                            |
| Exponentiation | `^`                                                                             |
| Fractions      | `//`                                                                            |
| Multiplication | `* / % & \`                                                                     |
| Bitshifts      | `<< >> >>>`                                                                     |
| Addition       | `+ - \| ⊻`                                                                      |
| Syntax         | `: ..` followed by `\|>`                                                        |
| Comparisons    | `> < >= <= == === != !== <:`                                                    |
| Control flow   | `&&` followed by `\|\|` followed by `?`                                         |
| Assignments    | `= += -= *= /= //= \= ^= ÷= %= \|= &= ⊻= <<= >>= >>>=`                          |

<!-- Start -->
For a complete list of *every* Julia operator's precedence, see the top of this file: [`src/julia-parser.scm`](https://github.com/JuliaLang/julia/blob/master/src/julia-parser.scm)
> *すべての* Juliaオペレータの優先順位の完全なリストについては、このファイルの先頭を参照してください。 [`src/julia-parser.scm`](https://github.com/JuliaLang/julia/blob/master/src/julia-parser.scm)
<!-- End -->

<!-- Start -->
You can also find the numerical precedence for any given operator via the built-in function `Base.operator_precedence`, where higher numbers take precedence:
> 与えられた演算子の数値優先順位は、組み込み関数 `Base.operator_precedence`を使って見つけることができます。ここでは、より高い数値が優先されます。
<!-- End -->

```jldoctest
julia> Base.operator_precedence(:+), Base.operator_precedence(:*), Base.operator_precedence(:.)
(9, 11, 15)

julia> Base.operator_precedence(:+=), Base.operator_precedence(:(=))  # (Note the necessary parens on `:(=)`)
(1, 1)
```

<!-- Start -->

## Numerical Conversions

> ##数値変換

<!-- End -->
<!-- Start -->
Julia supports three forms of numerical conversion, which differ in their handling of inexact conversions.
> Juliaは、不正確な変換の処理が異なる3つの数値変換形式をサポートしています。
<!-- End -->

<!-- Start -->
* The notation `T(x)` or `convert(T,x)` converts `x` to a value of type `T`.
* > `T(x)`や `convert(T、x)`という表記法は`x`を `T`型の値に変換します。
<!-- End -->
<!-- Start -->
* If `T` is a floating-point type, the result is the nearest representable value, which could be positive or negative infinity.
* > `T`が浮動小数点型の場合、結果は最も近い表現可能な値であり、正または負の無限大になります。
<!-- End -->
<!-- Start -->
* If `T` is an integer type, an `InexactError` is raised if `x` is not representable by `T`.
* > `T`が整数型の場合、`x`が `T`で表現できない場合、`InexactError`が発生します。
<!-- End -->
<!-- Start -->
* `x % T` converts an integer `x` to a value of integer type `T` congruent to `x` modulo `2^n`, where `n` is the number of bits in `T`. 
* > `x ％ T`は整数`x`を`2^n`を法とする整数型`「T」`の値に変換します、ここで`「n」`は`「T」`のビット数です。 
<!-- End -->
<!-- Start -->
* In other words, the binary representation is truncated to fit. 
* > 言い換えれば、バイナリ表現は収まるように切り捨てられます。
<!-- End -->
    
<!-- Start -->
* The [Rounding functions](@ref) take a type `T` as an optional argument. For example, `round(Int,x)` is a shorthand for `Int(round(x))`.
* > [丸め関数](@ref)はオプションの引数として型 `T`をとります。 たとえば、 `round(Int、x)`は `Int(round(x))`の短縮形です。
<!-- End -->

<!-- Start -->
The following examples show the different forms.
> 次の例は、さまざまな形式を示しています。
<!-- End -->

```jldoctest
julia> Int8(127)
127

julia> Int8(128)
ERROR: InexactError()
Stacktrace:
 [1] Int8(::Int64) at ./sysimg.jl:102

julia> Int8(127.0)
127

julia> Int8(3.14)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int8}, ::Float64) at ./float.jl:659
 [2] Int8(::Float64) at ./sysimg.jl:102

julia> Int8(128.0)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int8}, ::Float64) at ./float.jl:659
 [2] Int8(::Float64) at ./sysimg.jl:102

julia> 127 % Int8
127

julia> 128 % Int8
-128

julia> round(Int8,127.4)
127

julia> round(Int8,127.6)
ERROR: InexactError()
Stacktrace:
 [1] trunc(::Type{Int8}, ::Float64) at ./float.jl:652
 [2] round(::Type{Int8}, ::Float64) at ./float.jl:338
```

<!-- Start -->
See [Conversion and Promotion](@ref conversion-and-promotion) for how to define your own conversions and promotions.
> [コンバージョンとプロモーション](@ref conversion-and-promotion) で、独自のコンバージョンやプロモーションの定義方法を見て下さい。
<!-- End -->

<!-- Start -->
## Rounding functions
> ## 丸め関数

| Function              | Description                      | Return type |
|:--------------------- |:-------------------------------- |:----------- |
| [`round(x)`](@ref)    | round `x` to the nearest integer | `typeof(x)` |
| [`round(T, x)`](@ref) | round `x` to the nearest integer | `T`         |
| [`floor(x)`](@ref)    | round `x` towards `-Inf`         | `typeof(x)` |
| [`floor(T, x)`](@ref) | round `x` towards `-Inf`         | `T`         |
| [`ceil(x)`](@ref)     | round `x` towards `+Inf`         | `typeof(x)` |
| [`ceil(T, x)`](@ref)  | round `x` towards `+Inf`         | `T`         |
| [`trunc(x)`](@ref)    | round `x` towards zero           | `typeof(x)` |
| [`trunc(T, x)`](@ref) | round `x` towards zero           | `T`         |

<!-- End -->
<!-- Start -->

## Division functions

> ## 除算関数

<!-- End -->
| Function              | Description                                                                                               |
|:--------------------- |:--------------------------------------------------------------------------------------------------------- |
| [`div(x,y)`](@ref)    | truncated division; quotient rounded towards zero                                                         |
| [`fld(x,y)`](@ref)    | floored division; quotient rounded towards `-Inf`                                                         |
| [`cld(x,y)`](@ref)    | ceiling division; quotient rounded towards `+Inf`                                                         |
| [`rem(x,y)`](@ref)    | remainder; satisfies `x == div(x,y)*y + rem(x,y)`; sign matches `x`                                       |
| [`mod(x,y)`](@ref)    | modulus; satisfies `x == fld(x,y)*y + mod(x,y)`; sign matches `y`                                         |
| [`mod1(x,y)`](@ref)   | `mod()` with offset 1; returns `r∈(0,y]` for `y>0` or `r∈[y,0)` for `y<0`, where `mod(r, y) == mod(x, y)` |
| [`mod2pi(x)`](@ref)   | modulus with respect to 2pi;  `0 <= mod2pi(x)    < 2pi`                                                   |
| [`divrem(x,y)`](@ref) | returns `(div(x,y),rem(x,y))`                                                                             |
| [`fldmod(x,y)`](@ref) | returns `(fld(x,y),mod(x,y))`                                                                             |
| [`gcd(x,y...)`](@ref) | greatest positive common divisor of `x`, `y`,...                                                          |
| [`lcm(x,y...)`](@ref) | least positive common multiple of `x`, `y`,...                                                            |

<!-- Start -->

## Sign and absolute value functions

> ## 符号と絶対値関数

<!-- End -->

| Function                | Description                                                |
|:----------------------- |:---------------------------------------------------------- |
| [`abs(x)`](@ref)        | a positive value with the magnitude of `x`                 |
| [`abs2(x)`](@ref)       | the squared magnitude of `x`                               |
| [`sign(x)`](@ref)       | indicates the sign of `x`, returning -1, 0, or +1          |
| [`signbit(x)`](@ref)    | indicates whether the sign bit is on (true) or off (false) |
| [`copysign(x,y)`](@ref) | a value with the magnitude of `x` and the sign of `y`      |
| [`flipsign(x,y)`](@ref) | a value with the magnitude of `x` and the sign of `x*y`    |

<!-- Start -->

## Powers, logs and roots

> ##  べき乗、ログ、ルート

<!-- End -->
| Function                 | Description                                                                |
|:------------------------ |:-------------------------------------------------------------------------- |
| [`sqrt(x)`](@ref), `√x`  | square root of `x`                                                         |
| [`cbrt(x)`](@ref), `∛x`  | cube root of `x`                                                           |
| [`hypot(x,y)`](@ref)     | hypotenuse of right-angled triangle with other sides of length `x` and `y` |
| [`exp(x)`](@ref)         | natural exponential function at `x`                                        |
| [`expm1(x)`](@ref)       | accurate `exp(x)-1` for `x` near zero                                      |
| [`ldexp(x,n)`](@ref)     | `x*2^n` computed efficiently for integer values of `n`                     |
| [`log(x)`](@ref)         | natural logarithm of `x`                                                   |
| [`log(b,x)`](@ref)       | base `b` logarithm of `x`                                                  |
| [`log2(x)`](@ref)        | base 2 logarithm of `x`                                                    |
| [`log10(x)`](@ref)       | base 10 logarithm of `x`                                                   |
| [`log1p(x)`](@ref)       | accurate `log(1+x)` for `x` near zero                                      |
| [`exponent(x)`](@ref)    | binary exponent of `x`                                                     |
| [`significand(x)`](@ref) | binary significand (a.k.a. mantissa) of a floating-point number `x`        |

<!-- Start -->
For an overview of why functions like [`hypot()`](@ref), [`expm1()`](@ref), and [`log1p()`](@ref) are necessary and useful, see John D. 
> [`hypot()`](@ ref)、[`expm1()`](@ ref)、[`log1p()`](@ ref)などの関数がなぜ必要で有用なのかについては、John D.
<!-- End -->
<!-- Start -->
Cook's excellent pair of blog posts on the subject: [expm1, log1p, erfc](https://www.johndcook.com/blog/2010/06/07/math-library-functions-that-seem-unnecessary/), and [hypot](https://www.johndcook.com/blog/2010/06/02/whats-so-hard-about-finding-a-hypotenuse/).
> 件名に関するCookの優れたブログ記事：[expm1、log1p、erfc](https://www.johndcook.com/blog/2010/06/07/math-library-functions-that-seem-unnecessary/)、 と[hypot](https://www.johndcook.com/blog/2010/06/02/whats-so-hard-about-finding-a-hypotenuse/)を参照してください。
<!-- End -->
<!-- Start -->

## Trigonometric and hyperbolic functions

> ## 三角関数と双曲線関数

<!-- End -->
<!-- Start -->
All the standard trigonometric and hyperbolic functions are also defined:
> すべての標準的な三角関数と双曲線関数も定義されています。
<!-- End -->

```
sin    cos    tan    cot    sec    csc
sinh   cosh   tanh   coth   sech   csch
asin   acos   atan   acot   asec   acsc
asinh  acosh  atanh  acoth  asech  acsch
sinc   cosc   atan2
```

<!-- Start -->
These are all single-argument functions, with the exception of [atan2](https://en.wikipedia.org/wiki/Atan2), which gives the angle in [radians](https://en.wikipedia.org/wiki/Radian) between the *x*-axis and the point specified by its arguments, interpreted as *x* and *y* coordinates.
> これらはすべて、[atan2](https://en.wikipedia.org/wiki/Atan2) を除いて、単一引数の関数です。角度は[ラジアン](https://en.wikipedia.org/)になります。 wiki / Radian)を* x *軸とその引数で指定された点の間に置き、* x *および* y *座標として解釈します。
<!-- End -->

<!-- Start -->
Additionally, [`sinpi(x)`](@ref) and [`cospi(x)`](@ref) are provided for more accurate computations of [`sin(pi*x)`](@ref) and [`cos(pi*x)`](@ref) respectively.
> さらに、[`sin(pi * x)`](@ ref)と[sin(x)]]をより正確に計算するために[[sinpi(x) `](@ref)と[` cospi `` cos(pi * x) `](@ ref)になります。
<!-- End -->

<!-- Start -->
In order to compute trigonometric functions with degrees instead of radians, suffix the function with `d`. 
> ラジアンではなく度を用いて三角関数を計算するには、関数に `d`をつけます。
<!-- End -->
<!-- Start -->
For example, [`sind(x)`](@ref) computes the sine of `x` where `x` is specified in degrees. 
> 例えば、[`sind(x)`](@ref)は `x`の正弦を計算します。ここで` x`は度で指定します。
<!-- End -->
<!-- Start -->
The complete list of trigonometric functions with degree variants is: 
> 次数のある三角関数の完全なリストは次のとおりです。
<!-- End -->

```
sind   cosd   tand   cotd   secd   cscd
asind  acosd  atand  acotd  asecd  acscd
```

<!-- Start -->

## Special functions

> ## 特殊関数

<!-- End -->
| Function             | Description                                                                                     |
|:---------------------|:----------------------------------------------------------------------------------------------- |
| [`gamma(x)`](@ref)   | [gamma function] (https://en.wikipedia.org/wiki/Gamma_function) at `x`                          |
| [`lgamma(x)`](@ref)  | accurate `log(gamma(x))` for large `x`                                                          |
| [`lfact(x)`](@ref)   | accurate `log(factorial(x))` for large `x`; same as `lgamma(x+1)` for `x > 1`, zero otherwise   |
| [`beta(x,y)`](@ref)  | [beta function](https://en.wikipedia.org/wiki/Beta_function) at `x,y`                           |
| [`lbeta(x,y)`](@ref) | accurate `log(beta(x,y))` for large `x` or `y`                                                  |
