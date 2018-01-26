# Integers and Floating-Point Numbers

#整数と浮動小数点数

<!-- End -->
<!-- Start -->
Integers and floating-point values are the basic building blocks of arithmetic and computation.
> 整数と浮動小数点値は、算術演算の基本的なビルディングブロックです。
<!-- End -->
<!-- Start -->
Built-in representations of such values are called numeric primitives, while representations of integers and floating-point numbers as immediate values in code are known as numeric literals.
> このような値の組み込み表現は数値プリミティブと呼ばれ、整数と浮動小数点数はコード内の即値として表現され、数値リテラルと呼ばれます。
<!-- End -->
<!-- Start -->
For example, `1` is an integer literal, while `1.0` is a floating-point literal; their binary in-memory representations as objects are numeric primitives.
> 例えば、 `1`は整数リテラルであり、` 1.0`は浮動小数点リテラルです。オブジェクトとしてのバイナリのメモリ内表現は数値プリミティブです。

<!-- End -->
<!-- Start -->
Julia provides a broad range of primitive numeric types, and a full complement of arithmetic and bitwise operators as well as standard mathematical functions are defined over them.
> Juliaは広範囲のプリミティブな数値型を提供し、算術演算子とビット演算子の完全な補完と標準的な数学関数が定義されています。
<!-- End -->
<!-- Start -->
These map directly onto numeric types and operations that are natively supported on modern computers, thus allowing Julia to take full advantage of computational resources.
> これらは、現代のコンピュータ上でネイティブにサポートされている数値型および演算に直接マップされているため、ジュリアは計算資源を最大限に活用できます。
<!-- End -->
<!-- Start -->
Additionally, Julia provides software support for [Arbitrary Precision Arithmetic](@ref), which can handle operations on numeric values that cannot be represented effectively in native hardware representations, but at the cost of relatively slower performance.
> さらにJuliaは、ネイティブのハードウェア表現では効果的に表現できない数値の演算を処理できるが、パフォーマンスは比較的遅いという[恣意的精度演算](@ ref)のソフトウェアサポートを提供しています。

<!-- End -->
<!-- Start -->
The following are Julia's primitive numeric types:
> Juliaのプリミティブな数値型は次のとおりです。
<!-- End -->

  * **Integer types:**

| Type              | Signed? | Number of bits | Smallest value | Largest value |
|:----------------- |:------- |:-------------- |:-------------- |:------------- |
| [`Int8`](@ref)    | ✓       | 8              | -2^7           | 2^7 - 1       |
| [`UInt8`](@ref)   |         | 8              | 0              | 2^8 - 1       |
| [`Int16`](@ref)   | ✓       | 16             | -2^15          | 2^15 - 1      |
| [`UInt16`](@ref)  |         | 16             | 0              | 2^16 - 1      |
| [`Int32`](@ref)   | ✓       | 32             | -2^31          | 2^31 - 1      |
| [`UInt32`](@ref)  |         | 32             | 0              | 2^32 - 1      |
| [`Int64`](@ref)   | ✓       | 64             | -2^63          | 2^63 - 1      |
| [`UInt64`](@ref)  |         | 64             | 0              | 2^64 - 1      |
| [`Int128`](@ref)  | ✓       | 128            | -2^127         | 2^127 - 1     |
| [`UInt128`](@ref) |         | 128            | 0              | 2^128 - 1     |
| [`Bool`](@ref)    | N/A     | 8              | `false` (0)    | `true` (1)    |

  * **Floating-point types:**

| Type              | Precision                                                                     | Number 
                                                                                                     of bits |
|:----------------- |:----------------------------------------------------------------------------- |:------ |
| [`Float16`](@ref) | [half](https://en.wikipedia.org/wiki/Half-precision_floating-point_format)    | 16     |
| [`Float32`](@ref) | [single](https://en.wikipedia.org/wiki/Single_precision_floating-point_format)| 32     |
| [`Float64`](@ref) | [double](https://en.wikipedia.org/wiki/Double_precision_floating-point_format)| 64     |

<!-- Start -->
Additionally, full support for [Complex and Rational Numbers](@ref) is built on top of these primitive numeric types.
> さらに、[Complex and Rational Numbers](@ ref)の完全なサポートは、これらの基本的な数値型の上に構築されています。
<!-- End -->
<!-- Start -->
All numeric types interoperate naturally without explicit casting, thanks to a flexible, user-extensible [type promotion system](@ref conversion-and-promotion).
> すべての数値型は、柔軟性のあるユーザー拡張可能な[型昇格システム](@ ref conversion-and-promotion)のおかげで、明示的なキャスティングなしで自然に相互運用できます。
<!-- End -->
<!-- Start -->

## Integers

> ## 整数

<!-- End -->
<!-- Start -->
Literal integers are represented in the standard manner:-->
> リテラル整数は標準的な方法で表現されます：
<!-- End -->

```jldoctest
julia> 1
1

julia> 1234
1234
```

<!-- Start -->
The default type for an integer literal depends on whether the target system has a 32-bit architecture or a 64-bit architecture:
> 整数リテラルのデフォルトのタイプは、ターゲットシステムが32ビットアーキテクチャであるか、64ビットアーキテクチャであるかによって異なります。
<!-- End -->

```julia-repl
# 32-bit system:
julia> typeof(1)
Int32

# 64-bit system:
julia> typeof(1)
Int64
```

<!-- Start -->
The Julia internal variable [`Sys.WORD_SIZE`](@ref) indicates whether the target system is 32-bit or 64-bit:
> Juliaの内部変数[`Sys.WORD_SIZE`](@ ref)は、ターゲットシステムが32ビットか64ビットかを示します。
<!-- End -->

```julia-repl
# 32-bit system:
julia> Sys.WORD_SIZE
32

# 64-bit system:
julia> Sys.WORD_SIZE
64
```

<!-- Start -->
Julia also defines the types `Int` and `UInt`, which are aliases for the system's signed and unsigned native integer types respectively:
> Juliaはまた、システムの符号付きおよび符号なしの各整数型のエイリアスである `Int`および` UInt`型を定義しています。
<!-- End -->

```julia-repl
# 32-bit system:
julia> Int
Int32
julia> UInt
UInt32

# 64-bit system:
julia> Int
Int64
julia> UInt
UInt64
```

<!-- Start -->
Larger integer literals that cannot be represented using only 32 bits but can be represented in 64 bits always create 64-bit integers, regardless of the system type:
> 32ビットのみでは表現できないが64ビットで表現できるより大きな整数リテラルは、システムタイプに関係なく、常に64ビットの整数を作成します。
<!-- End -->

```jldoctest
# 32-bit or 64-bit system:
julia> typeof(3000000000)
Int64
```

<!-- Start -->
Unsigned integers are input and output using the `0x` prefix and hexadecimal (base 16) digits `0-9a-f` (the capitalized digits `A-F` also work for input).
> 符号なし整数は、 `0x`接頭辞と16進数(16進数)の数字「0-9a-f」(大文字の数字「A-F」も入力用に有効)を使用して入力および出力されます。
<!-- End -->
<!-- Start -->
The size of the unsigned value is determined by the number of hex digits used:
> 符号なしの値のサイズは、使用される16進数の数によって決まります。
<!-- End -->

```jldoctest
julia> 0x1
0x01

julia> typeof(ans)
UInt8

julia> 0x123
0x0123

julia> typeof(ans)
UInt16

julia> 0x1234567
0x01234567

julia> typeof(ans)
UInt32

julia> 0x123456789abcdef
0x0123456789abcdef

julia> typeof(ans)
UInt64
```

<!-- Start -->
This behavior is based on the observation that when one uses unsigned hex literals for integer values, one typically is using them to represent a fixed numeric byte sequence, rather than just an integer value.
> この動作は、整数値に符号なし16進リテラルを使用する場合、一般に整数値ではなく固定数値バイトシーケンスを表すために使用されるという観測に基づいています。
<!-- End -->

<!-- Start -->
Recall that the variable [`ans`](@ref) is set to the value of the last expression evaluated in an interactive session.
> 変数[`ans`](@ ref)は、対話型セッションで評価された最後の式の値に設定されていることを思い出してください。
<!-- End -->
<!-- Start -->
This does not occur when Julia code is run in other ways.
> Juliaコードが他の方法で実行されている場合、これは発生しません。
<!-- End -->

<!-- Start -->
Binary and octal literals are also supported:
> バイナリと8進リテラルもサポートされています：
<!-- End -->

```jldoctest
julia> 0b10
0x02

julia> typeof(ans)
UInt8

julia> 0o10
0x08

julia> typeof(ans)
UInt8
```

<!-- Start -->
The minimum and maximum representable values of primitive numeric types such as integers are given by the [`typemin()`](@ref) and [`typemax()`](@ref) functions:
> 整数のようなプリミティブ数値型の最小値と最大値は、[`typemin()`](@ref)と[`typemax()`](@ref)関数によって与えられます：
<!-- End -->

```jldoctest
julia> (typemin(Int32), typemax(Int32))
(-2147483648, 2147483647)

julia> for T in [Int8,Int16,Int32,Int64,Int128,UInt8,UInt16,UInt32,UInt64,UInt128]
           println("$(lpad(T,7)): [$(typemin(T)),$(typemax(T))]")
       end
   Int8: [-128,127]
  Int16: [-32768,32767]
  Int32: [-2147483648,2147483647]
  Int64: [-9223372036854775808,9223372036854775807]
 Int128: [-170141183460469231731687303715884105728,170141183460469231731687303715884105727]
  UInt8: [0,255]
 UInt16: [0,65535]
 UInt32: [0,4294967295]
 UInt64: [0,18446744073709551615]
UInt128: [0,340282366920938463463374607431768211455]
```

<!-- Start -->
The values returned by [`typemin()`](@ref) and [`typemax()`](@ref) are always of the given argument type.
> [`typemin()`](@ ref)と[`typemax()`](@ ref)によって返される値は常に与えられた引数型です。
<!-- End -->
<!-- Start -->
(The above expression uses several features we have yet to introduce, including [for loops](@ref man-loops), [Strings](@ref man-strings), and [Interpolation](@ref), but should be easy enough to understand for users with some existing programming experience.)
> (上記の式は、forループ(@ ref man-loops)、[Strings](@ ref man-strings)、[Interpolation](@ ref)など、まだ紹介していないいくつかの機能を使用していますが、 既存のプログラミング経験を持つユーザーには十分理解できる)
<!-- End -->
<!-- Start -->

## Overflow behavior

> ## オーバーフローの動作

<!-- End -->
<!-- Start -->
In Julia, exceeding the maximum representable value of a given type results in a wraparound behavior:
> Juliaでは、指定された型の表現可能な最大値を超えると、折り返し動作が発生します。
<!-- End -->

```jldoctest
julia> x = typemax(Int64)
9223372036854775807

julia> x + 1
-9223372036854775808

julia> x + 1 == typemin(Int64)
true
```

<!-- Start -->
Thus, arithmetic with Julia integers is actually a form of [modular arithmetic](https://en.wikipedia.org/wiki/Modular_arithmetic).
> したがって、Julia整数による算術演算は、実際には[モジュラ算術](https://en.wikipedia.org/wiki/Modular_arithmetic)の形式です。
<!-- End -->
<!-- Start -->
This reflects the characteristics of the underlying arithmetic of integers as implemented on modern computers.
> これは、現代のコンピュータに実装されている整数の基礎となる算術の特性を反映しています。
<!-- End -->
<!-- Start -->
In applications where overflow is possible, explicit checking for wraparound produced by overflow is essential; otherwise, the [`BigInt`](@ref) type in [Arbitrary Precision Arithmetic](@ref) is recommended instead.
> オーバーフローが可能なアプリケーションでは、オーバーフローによって生成されたラップアラウンドを明示的にチェックすることが不可欠です;それ以外の場合は、[任意精度の計算](@ ref)の[`BigInt`](@ ref)型を代わりに使用することをお勧めします。
<!-- End -->
<!-- Start -->

### Division errors

> ### 除算エラー

<!-- End -->
<!-- Start -->
Integer division (the `div` function) has two exceptional cases: dividing by zero, and dividing the lowest negative number ([`typemin()`](@ref)) by -1.
> 整数除算( `div`関数)は2つの例外的なケースがあります：0で除算し、最も低い負の数([` typemin() `](@ ref))を-1で除算します。
<!-- End -->
<!-- Start -->
Both of these cases throw a [`DivideError`](@ref).
> これらの両方の場合、[`DivideError`](@ref)がスローされます。
<!-- End -->
<!-- Start -->
The remainder and modulus functions (`rem` and `mod`) throw a [`DivideError`](@ref) when their second argument is zero.
> 残余とモジュラス関数( `rem`と` mod`)は、第2引数がゼロのときに[DivideError`](@ ref)を投げます。
<!-- End -->
<!-- Start -->

## Floating-Point Numbers

> ## 浮動小数点数

<!-- End -->
<!-- Start -->
Literal floating-point numbers are represented in the standard formats:
> リテラルの浮動小数点数は標準形式で表されます。
<!-- End -->

```jldoctest
julia> 1.0
1.0

julia> 1.
1.0

julia> 0.5
0.5

julia> .5
0.5

julia> -1.23
-1.23

julia> 1e10
1.0e10

julia> 2.5e-4
0.00025
```

<!-- Start -->
The above results are all [`Float64`](@ref) values.
> 上記の結果はすべて[`Float64`](@ref)の値です。
<!-- End -->
<!-- Start -->
Literal [`Float32`](@ref) values can be entered by writing an `f` in place of `e`:
> リテラル [`Float32`](@ref) の値は `e` の代わりに `f` を書くことで入力できます：
<!-- End -->

```jldoctest
julia> 0.5f0
0.5f0

julia> typeof(ans)
Float32

julia> 2.5f-4
0.00025f0
```

<!-- Start -->
Values can be converted to [`Float32`](@ref) easily:
> 値は [`Float32`](@ref) に簡単に変換できます：
<!-- End -->

```jldoctest
julia> Float32(-1.5)
-1.5f0

julia> typeof(ans)
Float32
```

<!-- Start -->
Hexadecimal floating-point literals are also valid, but only as [`Float64`](@ref) values:
> 16進浮動小数点リテラルも有効ですが、[`Float64`](@ ref)の値としてのみ有効です：
<!-- End -->

```jldoctest
julia> 0x1p0
1.0

julia> 0x1.8p3
12.0

julia> 0x.4p-1
0.125

julia> typeof(ans)
Float64
```

<!-- Start -->
Half-precision floating-point numbers are also supported ([`Float16`](@ref)), but they are implemented in software and use [`Float32`](@ref) for calculations.
> 半精度の浮動小数点数もサポートされていますが ([`Float16`](@ref)) 、ソフトウェアで実装され、計算には [`Float32`](@ref) を使用します。
<!-- End -->

```jldoctest
julia> sizeof(Float16(4.))
2

julia> 2*Float16(4.)
Float16(8.0)
```

<!-- Start -->
The underscore `_` can be used as digit separator:
> アンダースコア `_ 'は数字区切り文字として使用できます：
<!-- End -->

```jldoctest
julia> 10_000, 0.000_000_005, 0xdead_beef, 0b1011_0010
(10000, 5.0e-9, 0xdeadbeef, 0xb2)
```

<!-- Start -->

### Floating-point zero

> ### 浮動小数点ゼロ

<!-- End -->
<!-- Start -->
Floating-point numbers have [two zeros](https://en.wikipedia.org/wiki/Signed_zero), positive zero and negative zero.
> 浮動小数点数は[2つのゼロ](https://en.wikipedia.org/wiki/Signed_zero)、正のゼロと負のゼロを持ちます。
<!-- End -->
<!-- Start -->
They are equal to each other but have different binary representations, as can be seen using the `bits` function: :
> それらは互いに等しいが、 `bits`関数を使って見ることができるように、異なるバイナリ表現を持っています。
<!-- End -->

```jldoctest
julia> 0.0 == -0.0
true

julia> bits(0.0)
"0000000000000000000000000000000000000000000000000000000000000000"

julia> bits(-0.0)
"1000000000000000000000000000000000000000000000000000000000000000"
```

<!-- Start -->

### Special floating-point values

>### 特殊浮動小数点値

<!-- End -->
<!-- Start -->
There are three specified standard floating-point values that do not correspond to any point on the real number line:
> 実数直線の任意の点に対応しない3つの指定された標準浮動小数点値があります。
<!-- End -->

| `Float16`| `Float32`| `Float64`| Name             | Description                                           |
|:---------|:---------|:---------|:-----------------|:----------------------------------------------------- |
| `Inf16`  | `Inf32`  | `Inf`    | positive infinity| a value greater than all finite floating-point values |
| `-Inf16` | `-Inf32` | `-Inf`   | negative infinity| a value less than all finite floating-point values    |
| `NaN16`  | `NaN32`  | `NaN`    | not a number     | a value not `==` to any floating-point value 
#                                                      (including itself)                                    |
| `Inf16`  | `Inf32`  | `Inf`    | 正の無限大 | すべての有限浮動小数点値より大きい値                     |
| `-Inf16` | `-Inf32` | `-Inf`   | 負の無限大 | すべての有限浮動小数点値より小さい値                     |
| `NaN16`  | `NaN32`  | `NaN`    | 非数      | 浮動小数点値に対して `==`ではない値 (それ自体を含む)     |

<!-- Start -->
For further discussion of how these non-finite floating-point values are ordered with respect to each other and other floats, see [Numeric Comparisons](@ref).
> これらの非有限浮動小数点値が互いおよび他の浮動小数点に対してどのように並べられているかの詳細については、 [Numeric Comparisons](@ref) を参照してください。
<!-- End -->
<!-- Start -->
By the [IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754-2008), these floating-point values are the results of certain arithmetic operations:
> [IEEE 754標準](https://en.wikipedia.org/wiki/IEEE_754-2008)により、これらの浮動小数点値は特定の算術演算の結果です。
<!-- End -->

```jldoctest
julia> 1/Inf
0.0

julia> 1/0
Inf

julia> -5/0
-Inf

julia> 0.000001/0
Inf

julia> 0/0
NaN

julia> 500 + Inf
Inf

julia> 500 - Inf
-Inf

julia> Inf + Inf
Inf

julia> Inf - Inf
NaN

julia> Inf * Inf
Inf

julia> Inf / Inf
NaN

julia> 0 * Inf
NaN
```

<!-- Start -->
The [`typemin()`](@ref) and [`typemax()`](@ref) functions also apply to floating-point types:
> [`typemin()`](@ref)と[`typemax()`](@ref)関数は浮動小数点型にも適用されます：
<!-- End -->

```jldoctest
julia> (typemin(Float16),typemax(Float16))
(-Inf16, Inf16)

julia> (typemin(Float32),typemax(Float32))
(-Inf32, Inf32)

julia> (typemin(Float64),typemax(Float64))
(-Inf, Inf)
```

<!-- Start -->

### Machine epsilon

> ### マシンイプシロン

<!-- End -->
<!-- Start -->
Most real numbers cannot be represented exactly with floating-point numbers, and so for many purposes it is important to know the distance between two adjacent representable floating-point numbers, which is often known as [machine epsilon](https://en.wikipedia.org/wiki/Machine_epsilon).
> ほとんどの実数は、浮動小数点数で正確に表現することはできません。したがって、多くの目的で、表現可能な2つの浮動小数点数の間の距離を知ることは重要です。これは[machineε(https：// en。 wikipedia.org/wiki/Machine_epsilon)。
<!-- End -->

<!-- Start -->
Julia provides [`eps()`](@ref), which gives the distance between `1.0` and the next larger representable floating-point value:
> Juliaは、 [`eps()`](@ref) を提供します。これは、 `1.0` と次の大きな表現可能な浮動小数点値の間の距離を与えます：
<!-- End -->

```jldoctest
julia> eps(Float32)
1.1920929f-7

julia> eps(Float64)
2.220446049250313e-16

julia> eps() # same as eps(Float64)
2.220446049250313e-16
```

<!-- Start -->
These values are `2.0^-23` and `2.0^-52` as [`Float32`](@ref) and [`Float64`](@ref) values, respectively.
> これらの値は、[`Float32`](@ref) と [`Float64`](@ref) の値としてそれぞれ `2.0^-23` と `2.0^-52` です。
<!-- End -->
<!-- Start -->
The [`eps()`](@ref) function can also take a floating-point value as an argument, and gives the absolute difference between that value and the next representable floating point value.
> [`eps()`](@ref) 関数は、浮動小数点値を引数として取ることもでき、その値と次の表現可能な浮動小数点値の間の絶対差を与えます。
<!-- End -->
<!-- Start -->
That is, `eps(x)` yields a value of the same type as `x` such that `x + eps(x)` is the next representable floating-point value larger than `x`:
> つまり、 `eps(x)`は、 `x + eps(x)`が `x`よりも大きい表現可能な次の浮動小数点値となるように` x`と同じ型の値を生成します：
<!-- End -->

```jldoctest
julia> eps(1.0)
2.220446049250313e-16

julia> eps(1000.)
1.1368683772161603e-13

julia> eps(1e-27)
1.793662034335766e-43

julia> eps(0.0)
5.0e-324
```

<!-- Start -->
The distance between two adjacent representable floating-point numbers is not constant, but is smaller for smaller values and larger for larger values.
> 隣接する2つの表現可能な浮動小数点数の間の距離は一定ではありませんが、小さい値の方が小さく、大きな値の方が大きくなります。
<!-- End -->
<!-- Start -->
In other words, the representable floating-point numbers are densest in the real number line near zero, and grow sparser exponentially as one moves farther away from zero.
> 換言すれば、表現可能な浮動小数点数は、ゼロに近い実数ラインにおいて最も密であり、ゼロから遠ざかるにつれて指数関数的に疎になる。
<!-- End -->
<!-- Start -->
By definition, `eps(1.0)` is the same as `eps(Float64)` since `1.0` is a 64-bit floating-point value.
> 定義上、 `eps(1.0)` は `eps(Float64)` と同じです。 `1.0` は64ビットの浮動小数点値だからです。
<!-- End -->

<!-- Start -->
Julia also provides the [`nextfloat()`](@ref) and [`prevfloat()`](@ref) functions which return the next largest or smallest representable floating-point number to the argument respectively:
> Juliaは次の最大または最小の表現可能な浮動小数点数をそれぞれ引数に返す [`nextfloat()`](@ref) 関数と [`prevfloat()`](@ref) 関数も提供しています：
<!-- End -->

```jldoctest
julia> x = 1.25f0
1.25f0

julia> nextfloat(x)
1.2500001f0

julia> prevfloat(x)
1.2499999f0

julia> bits(prevfloat(x))
"00111111100111111111111111111111"

julia> bits(x)
"00111111101000000000000000000000"

julia> bits(nextfloat(x))
"00111111101000000000000000000001"
```

<!-- Start -->
This example highlights the general principle that the adjacent representable floating-point numbers also have adjacent binary integer representations.
> この例では、隣接する表現可能な浮動小数点数にも隣接する2進整数表現があるという一般原則が強調されています。
<!-- End -->

### Rounding modes

> ### 丸めモード

<!-- End -->
<!-- Start -->
If a number doesn't have an exact floating-point representation, it must be rounded to an appropriate representable value, however, if wanted, the manner in which this rounding is done can be changed according to the rounding modes presented in the [IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754-2008).
> 数値が正確な浮動小数点表現を持たない場合は、適切な表現可能な値に丸めなければなりません、ただし、必要に応じて[IEEE 754標準](https://en.wikipedia.org/wiki/IEEE_754-2008)に示されている丸めモードに従って、この丸め処理の方法を変更することができます。
<!-- End -->

```jldoctest
julia> x = 1.1; y = 0.1;

julia> x + y
1.2000000000000002

julia> setrounding(Float64,RoundDown) do
           x + y
       end
1.2
```

<!-- Start -->
The default mode used is always [`RoundNearest`](@ref), which rounds to the nearest representable value, with ties rounded towards the nearest value with an even least significant bit.
> 使用されるデフォルトのモードは常に最も近い表現可能な値に丸められ、最も重要なビットが最も近い最も近い値に丸められた[[RoundNearest`](@ ref)です。
<!-- End -->

!!! warning
<!-- Start -->
   Rounding is generally only correct for basic arithmetic functions ([`+()`](@ref), [`-()`](@ref), [`*()`](@ref), [`/()`](@ref) and [`sqrt()`](@ref)) and type conversion operations.
   > 丸めは一般的に基本的な算術関数([`+()`](@ ref)、[` - ()`](@ ref)、[`*()`](@ ref)、[`/ ) `](@ ref)と[` sqrt() `](@ ref))とタイプ変換演算を実行します。
<!-- End -->
<!-- Start -->
   Many other functions assume the default [`RoundNearest`](@ref) mode is set, and can give erroneous results when operating under other rounding modes.
   > 他の多くの関数は、デフォルトの [`RoundNearest`](@ref) モードが設定されていると仮定し、他の丸めモードで動作すると誤った結果を与える可能性があります。
<!-- End -->
<!-- Start -->

### Background and References

> ### 背景と参考文献

<!-- End -->
<!-- Start -->
Floating-point arithmetic entails many subtleties which can be surprising to users who are unfamiliar with the low-level implementation details.
> 浮動小数点演算は、低レベルの実装の詳細に精通していないユーザーにとっては驚くべきことかもしれません。
<!-- End -->
<!-- Start -->
However, these subtleties are described in detail in most books on scientific computation, and also in the following references:
> しかし、これらの微妙な点については、科学計算に関する多くの書籍や以下の参考文献で詳しく説明されています。
<!-- End -->
<!-- Start -->
* The definitive guide to floating point arithmetic is the [IEEE 754-2008 Standard](http://standards.ieee.org/findstds/standard/754-2008.html); however, it is not available for free online.
* > 浮動小数点演算の決定的なガイドは[IEEE 754-2008標準]です。ただし、無料でオンラインでは利用できません。
<!-- End -->
<!-- Start -->
* For a brief but lucid presentation of how floating-point numbers are represented, see John D.Cook's [article](https://www.johndcook.com/blog/2009/04/06/anatomy-of-a-floating-point-number/) on the subject as well as his [introduction](https://www.johndcook.com/blog/2009/04/06/numbers-are-a-leaky-abstraction/) to some of the issues arising from how this representation differs in behavior from the idealized abstraction of real numbers.
* > 浮動小数点数の表現方法を簡潔かつ明快に説明するには、John D.Cookの[記事](johndcook)と、この表現がどのように振る舞いが異なるかによって生じるいくつかの問題についての彼の[introduction]を参照してください。理想化された実数の抽象化。
<!-- End -->
<!-- Start -->
* Also recommended is Bruce Dawson's [series of blog posts on floating-point numbers](https://randomascii.wordpress.com/2012/05/20/thats-not-normalthe-performance-of-odd-floats/).
* > また、Bruce Dawson [浮動小数点数に関するブログ記事のシリーズ]は、お推めです。
<!-- End -->
<!-- Start -->
For an excellent, in-depth discussion of floating-point numbers and issues of numerical accuracy encountered when computing with them, see David Goldberg's paper [What Every Computer Scientist Should Know About Floating-Point Arithmetic](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.22.6768&rep=rep1&type=pdf).
* > 浮動小数点数とそれらを使用して計算する際に発生する数値精度の優れた綿密な議論については、David Goldbergの論文[浮動小数点演算について各コンピュータ科学者が知るべきこと](pdf)を参照してください。
<!-- End -->
<!-- Start -->
* For even more extensive documentation of the history of, rationale for, and issues with floating-point numbers, as well as discussion of many other topics in numerical computing, see the [collected writings](https://people.eecs.berkeley.edu/~wkahan/) of [William Kahan](https://en.wikipedia.org/wiki/William_Kahan), commonly known as the "Father of Floating-Point".
* > 浮動小数点数の歴史や理論的根拠、浮動小数点数の問題のさらに詳細な文書化、数値計算における他の多くのトピックの議論については、[William Kahan]の[wkahan / William_Kahan)は、一般に「浮動小数点の父」として知られています。
<!-- End -->
<!-- Start -->
* Of particular interest may be [An Interview with the Old Man of Floating-Point](https://people.eecs.berkeley.edu/~wkahan/ieee754status/754story.html).
* > 特に興味のあるものは、[浮動小数点の老人とのインタビュー](https://people.eecs.berkeley.edu/~wkahan/ieee754status/754story.html)です。
<!-- End -->
<!-- Start -->

## Arbitrary Precision Arithmetic

> ## 任意の精密算術

<!-- End -->
<!-- Start -->
To allow computations with arbitrary-precision integers and floating point numbers, Julia wraps the [GNU Multiple Precision Arithmetic Library (GMP)](https://gmplib.org) and the [GNU MPFR Library](http://www.mpfr.org), respectively.
> Juliaは、任意精度の整数と浮動小数点数を使った計算を可能にするため、[GNU多重精度算術ライブラリ(GMP)](https://gmplib.org)と[GNU MPFRライブラリ](http：//www.mpfr .org)である。
<!-- End -->
<!-- Start -->
The [`BigInt`](@ref) and [`BigFloat`](@ref) types are available in Julia for arbitrary precision integer and floating point numbers respectively.
> Juliusでは、[`BigInt`](@ref) と [`BigFloat`](@ref) の型は、それぞれ任意の精度の整数と浮動小数点数で使用できます。
<!-- End -->

<!-- Start -->
Constructors exist to create these types from primitive numerical types, and [`parse()`](@ref) can be used to construct them from `AbstractString`s. 
> これらの型をプリミティブな数値型から作成するコンストラクタが存在し、[`parse()`](@ref)を使って `AbstractString`からそれらを構築することができます。
<!-- End -->
<!-- Start -->
Once created, they participate in arithmetic with all other numeric types thanks to Julia's [type promotion and conversion mechanism](@ref conversion-and-promotion):
> 作成されると、Juliaの[タイププロモーションと変換メカニズム](@ref conversion-and-promotion)のおかげで、他のすべての数値タイプの算術演算に参加します。
<!-- End -->

```jldoctest
julia> BigInt(typemax(Int64)) + 1
9223372036854775808

julia> parse(BigInt, "123456789012345678901234567890") + 1
123456789012345678901234567891

julia> parse(BigFloat, "1.23456789012345678901")
1.234567890123456789010000000000000000000000000000000000000000000000000000000004

julia> BigFloat(2.0^66) / 3
2.459565876494606882133333333333333333333333333333333333333333333333333333333344e+19

julia> factorial(BigInt(40))
815915283247897734345611269596115894272000000000
```

<!-- Start -->
However, type promotion between the primitive types above and [`BigInt`](@ref)/[`BigFloat`](@ref) is not automatic and must be explicitly stated.
> しかし、上記のプリミティブ型と [`BigInt`](@ref)/[`BigFloat`](@ref) 間の型変換は自動ではなく、明示的に述べなければなりません。

```jldoctest
julia> x = typemin(Int64)
-9223372036854775808

julia> x = x - 1
9223372036854775807

julia> typeof(x)
Int64

julia> y = BigInt(typemin(Int64))
-9223372036854775808

julia> y = y - 1
-9223372036854775809

julia> typeof(y)
BigInt
```

<!-- End -->
<!-- Start -->
The default precision (in number of bits of the significand) and rounding mode of [`BigFloat`](@ref) operations can be changed globally by calling [`setprecision()`](@ref) and [`setrounding()`](@ref), and all further calculations will take these changes in account. 
> [`setprecision()`](@ref) と [`setrounding()`](@ref) を呼び出すことによって、デフォルト精度(仮数のビット数)と [`BigFloat`](@ref) となり、これ以降の計算ではこれらの変更が考慮されます。
<!-- End -->
<!-- Start -->
Alternatively, the precision or the rounding can be changed only within the execution of a particular block of code by using the same functions with a `do` block:
> あるいは、正確さまたは丸めは、doブロックで同じ関数を使用することによって、特定のコードブロックの実行内でのみ変更することができます。
<!-- End -->

```jldoctest
julia> setrounding(BigFloat, RoundUp) do
           BigFloat(1) + parse(BigFloat, "0.1")
       end
1.100000000000000000000000000000000000000000000000000000000000000000000000000003

julia> setrounding(BigFloat, RoundDown) do
           BigFloat(1) + parse(BigFloat, "0.1")
       end
1.099999999999999999999999999999999999999999999999999999999999999999999999999986

julia> setprecision(40) do
           BigFloat(1) + parse(BigFloat, "0.1")
       end
1.1000000000004
```
<!-- Start -->

## [Numeric Literal Coefficients](@id man-numeric-literal-coefficients)

> ## [数値リテラル係数](@ id人数値 - リテラル係数)

<!-- End -->
<!-- Start -->
To make common numeric formulas and expressions clearer, Julia allows variables to be immediately preceded by a numeric literal, implying multiplication.
> Juliaは、一般的な数値式と式をより明確にするために、変数の直前に数値リテラルを付けて、乗算を意味します。
<!-- End -->
<!-- Start -->
This makes writing polynomial expressions much cleaner:
> これにより、多項式表現をもっときれいに書くことができます。
<!-- End -->

```jldoctest numeric-coefficients
julia> x = 3
3

julia> 2x^2 - 3x + 1
10

julia> 1.5x^2 - .5x + 1
13.0
```

<!-- Start -->
It also makes writing exponential functions more elegant:
> 指数関数をよりエレガントに書くこともできます：
<!-- End -->

```jldoctest numeric-coefficients
julia> 2^2x
64
```

<!-- Start -->
The precedence of numeric literal coefficients is the same as that of unary operators such as negation.
> 数値のリテラル係数の優先順位は、否定などの単項演算子の優先順位と同じです。
<!-- End -->
<!-- Start -->
So `2^3x` is parsed as `2^(3x)`, and `2x^3` is parsed as `2*(x^3)`.
> したがって、 `2 ^ 3x`は` 2 ^(3x) `として解析され、` 2x ^ 3`は `2 *(x ^ 3)`として解析されます。
<!-- End -->

<!-- Start -->
Numeric literals also work as coefficients to parenthesized expressions:
> 数値リテラルはカッコで囲まれた式の係数としても機能します。
<!-- End -->

```jldoctest numeric-coefficients
julia> 2(x-1)^2 - 3(x-1) + 1
3
```

<!-- Start -->
Additionally, parenthesized expressions can be used as coefficients to variables, implying multiplication of the expression by the variable:
> さらに、カッコで囲まれた式を変数への係数として使用することができ、式の変数による乗算を意味します。
<!-- End -->

```jldoctest numeric-coefficients
julia> (x-1)x
6
```

<!-- Start -->
Neither juxtaposition of two parenthesized expressions, nor placing a variable before a parenthesized expression, however, can be used to imply multiplication:
> しかし、カッコで囲まれた2つの式を並置したり、カッコで囲まれた式の前に変数を配置したりすることは、乗算を暗示するために使用できません。
<!-- End -->

```jldoctest numeric-coefficients
julia> (x-1)(x+1)
ERROR: MethodError: objects of type Int64 are not callable

julia> x(x+1)
ERROR: MethodError: objects of type Int64 are not callable
```

<!-- Start -->
Both expressions are interpreted as function application: any expression that is not a numeric literal, when immediately followed by a parenthetical, is interpreted as a function applied to the values in parentheses (see [Functions](@ref) for more about functions).
> 両方の式は関数アプリケーションとして解釈されます。数値リテラルではない式は、直後にかっこが付いたときに、カッコ内の値に適用される関数として解釈されます(関数の詳細については、[関数](@ ref)を参照)。
<!-- End -->
<!-- Start -->
Thus, in both of these cases, an error occurs since the left-hand value is not a function.
> したがって、これらの両方の場合、左辺値は関数ではないのでエラーが発生します。
<!-- End -->

<!-- Start -->
The above syntactic enhancements significantly reduce the visual noise incurred when writing common mathematical formulae.
> 上記の構文拡張は、一般的な数式を記述するときに発生する視覚的ノイズを大幅に削減します。
<!-- End -->
<!-- Start -->
Note that no whitespace may come between a numeric literal coefficient and the identifier or parenthesized expression which it multiplies.
> 数値リテラル係数とそれが乗算する識別子またはカッコ内の式の間に空白がないことに注意してください。
<!-- End -->
<!-- Start -->

### Syntax Conflicts

> ### 構文の衝突

<!-- End -->
<!-- Start -->
Juxtaposed literal coefficient syntax may conflict with two numeric literal syntaxes: hexadecimal integer literals and engineering notation for floating-point literals.
> 並置されたリテラル係数構文は、2つの数値リテラル構文(16進整数リテラルおよび浮動小数点リテラルのエンジニアリング表記)と競合することがあります。
<!-- End -->
<!-- Start -->
Here are some situations where syntactic conflicts arise:
> 構文上の矛盾が発生するいくつかの状況を次に示します。
<!-- End -->

   <!-- Start -->
   * The hexadecimal integer literal expression `0xff` could be interpreted as the numeric literal `0` multiplied by the variable `xff`.
   * > 16進整数のリテラル式 `0xff`は、数値リテラル` 0`に変数 `xff`を乗じたものとして解釈できます。
   <!-- End -->
   <!-- Start --> 
   * The floating-point literal expression `1e10` could be interpreted as the numeric literal `1` multiplied by the variable `e10`, and similarly with the equivalent `E` form.
   * > 浮動小数点リテラル表現 `1e10`は、数値リテラル` 1`に変数 `e10`を乗じたものと解釈することができ、同様に同等の` E`形式と解釈することができます。
   <!-- End -->

<!-- Start -->
In both cases, we resolve the ambiguity in favor of interpretation as a numeric literals:
> どちらの場合も、解釈を数値リテラルとして使用するというあいまいさを解消します。
<!-- End -->

   <!-- Start --> 
   * Expressions starting with `0x` are always hexadecimal literals.
   * > `0x 'で始まる式は常に16進リテラルです。
   <!-- End -->
   <!-- Start -->
   * Expressions starting with a numeric literal followed by `e` or `E` are always floating-point literals.
   * > 数値リテラルとそれに続く `e`または` E`で始まる式は常に浮動小数点リテラルです。
   <!-- End -->

<!-- Start -->

## Literal zero and one

> ## リテラルゼロ と 1

<!-- End -->
<!-- Start -->
Julia provides functions which return literal 0 and 1 corresponding to a specified type or the type of a given variable.
> Juliaは、指定された型または与えられた変数の型に対応するリテラル0および1を返す関数を提供します。
<!-- End -->

| Function          | Description                                      |
|:----------------- |:------------------------------------------------ |
| [`zero(x)`](@ref) | Literal zero of type `x` or type of variable `x` |
| [`one(x)`](@ref)  | Literal one of type `x` or type of variable `x`  |

<!-- Start -->
These functions are useful in [Numeric Comparisons](@ref) to avoid overhead from unnecessary [type conversion](@ref conversion-and-promotion).
> これらの関数は、不要な[型変換](@ ref変換と昇格)のオーバーヘッドを避けるために[数値比較](@ ref)で便利です。
<!-- End -->

Examples:

```jldoctest
julia> zero(Float32)
0.0f0

julia> zero(1.0)
0.0

julia> one(Int32)
1

julia> one(BigFloat)
1.000000000000000000000000000000000000000000000000000000000000000000000000000000
```
