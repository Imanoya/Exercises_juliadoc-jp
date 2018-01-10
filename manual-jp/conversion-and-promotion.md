
<!-- > [Conversion and Promotion](@id conversion-and-promotion) -->
# [Conversion and Promotion](@id conversion-and-promotion)


<!-- Julia has a system for promoting arguments of mathematical operators to a common type, which has been mentioned in various other sections, including [Integers and Floating-Point Numbers](@ref), [Mathematical Operations and Elementary Functions](@ref), [Types](@ref man-types), and [Methods](@ref). -->
Juliaには、数学的演算子の引数を共通の型に昇格させるためのシステムがあり、さまざまなセクション[Integers and Floating-Point Numbers](@ref) 、[数学演算と基本関数](@ref) 、[Types](@ref man-types)、[Methods](@ref) で言及されています。
<!-- In this section, we explain how this promotion system works, as well as how to extend it to new types and apply it to functions besides built-in mathematical operators.  -->
このセクションでは、このプロモーションシステムがどのように機能するのか、それを新しいタイプに拡張して組み込みの数学演算子以外の関数にも適用する方法について説明します。
<!-- Traditionally, programming languages fall into two camps with respect to promotion of arithmetic arguments: -->
伝統的に、プログラミング言語は、算術引数の宣伝に関して2つのキャンプに分類されます。

<!-- >  ***Automatic promotion for built-in arithmetic types and operators.** -->
   ***組み込み算術型と演算子の自動昇格**
   #  In most languages, built-in numeric types, when used as operands to arithmetic operators with infix syntax, such as `+`, `-`, `*`, and `/`, are automatically promoted to a common type to produce the expected results.
    ほとんどの言語では、組み込みの数値型が、infix構文の算術演算子のオペランドとして使用されます, `+`、 ` - `、 `*`、 `/`のようなものは、自動的に共通の型に昇格され、期待される結果が得られます。
   #  C, Java, Perl, and Python, to name a few, all correctly compute the sum `1 + 1.5` as the floating-point value `2.5`, even though one of the operands to `+` is an integer. 
    C、Java、Perl、Pythonのように、 `+`へのオペランドの1つが整数であっても、浮動小数点値「2.5」として合計「1 + 1.5」を正しく計算する必要があります。
   # These systems are convenient and designed carefully enough that they are generally all-but-invisible to the programmer: hardly anyone consciously thinks of this promotion taking place when writing such an expression, but compilers and interpreters must perform conversion before addition since integers and floating-point values cannot be added as-is. 
    これらのシステムは、プログラマにとっては一般的には完全に見えないほど慎重に設計されていますが、このような表現を書くときに意識的にこの昇格を考える人はほとんどいません しかし、コンパイラやインタプリタは、整数と浮動小数点値をそのままでは追加できないため、追加する前に変換を実行する必要があります。
   # Complex rules for such automatic conversions are thus inevitably part of specifications and implementations for such languages.
    したがって、そのような自動変換の複雑な規則は、必然的に、そのような言語の仕様および実装の一部であります。

<!-- >  * **No automatic promotion.** -->
* **非自動昇格**
   # This camp includes Ada and ML -- very "strict" statically typed languages.
    このキャンプにはAdaとMLが含まれています。非常に厳密な型付けされた言語です。
   # In these languages, every conversion must be explicitly specified by the programmer. 
    これらの言語では、すべての変換をプログラマが明示的に指定する必要があります。
   # Thus, the example expression `1 + 1.5` would be a compilation error in both Ada and ML. 
    したがって、例の式「1 + 1.5」は、AdaとMLの両方でコンパイルエラーになります。
   # Instead one must write `real(1) + 1.5`, explicitly converting the integer `1` to a floating-point value before performing addition. 
    その代わりに、加算を実行する前に整数 `1`を浮動小数点値に明示的に変換する` real(1)+ 1.5`を書き込む必要があります。
   # Explicit conversion everywhere is so inconvenient, however, that even Ada has some degree of automatic conversion: integer literals are promoted to the expected integer type automatically, and floating-point literals are similarly promoted to appropriate floating-point types.
    しかし、どこでも明示的に変換するのは非常に面倒ですが、Adaでもある程度の自動変換が行われます。整数リテラルは予想される整数型に自動的に昇格され、浮動小数点リテラルも同様に適切な浮動小数点型に昇格されます。

<!-- In a sense, Julia falls into the "no automatic promotion" category: mathematical operators are just functions with special syntax, and the arguments of functions are never automatically converted.-->
ある意味では、Juliaは"非自動型昇格"カテゴリに分類されます。数学演算子は特殊な構文を持つ関数に過ぎず、関数の引数は決して自動的に変換されません。
<!-- However, one may observe that applying mathematical operations to a wide variety of mixed argument types is just an extreme case of polymorphic multiple dispatch -- something which Julia's dispatch and type systems are particularly well-suited to handle. -->
多種多様な混合引数型に数学的演算を適用することは、多態的な多重ディスパッチの極端なケースに過ぎませんが、Juliaのディスパッチおよびタイプシステムは、とても適切に処理します。
<!-- "Automatic" promotion of mathematical operands simply emerges as a special application: Julia comes with pre-defined catch-all dispatch rules for mathematical operators, invoked when no specific implementation exists for some combination of operand types. -->
数学的オペランドの「自動」昇格は、特殊なアプリケーションとしてシンプルに現わします: Juliaは事前定義されたキャッチオールディスパッチルールを数式演算子のために用意しています。これは、オペランドタイプの組み合わせによっては特定の実装が存在しないときに呼び出されます。
#These catch-all rules first promote all operands to a common type using user-definable promotion rules, and then invoke a specialized implementation of the operator in question for the resulting values, now of the same type. 
これらのキャッチオールルールは、まずユーザー定義可能なプロモーションルールを使用して、すべてのオペランドを共通タイプに昇格させ、結果の値について、同じ型の問題の演算子の特殊な実装を呼び出します。
#User-defined types can easily participate in this promotion system by defining methods for conversion to and from other types, and providing a handful of promotion rules defining what types they should promote to when mixed with other types.
ユーザー定義型は、他の型との間で変換するメソッドを定義し、他の型と混合したときにどの型を宣言するべきかを定義するプロモーションルールをいくつか提供することで、このプロモーションシステムに簡単に参加できます。

<!-- ## Conversion -->
## (conversion) 変換

<!-- Conversion of values to various types is performed by the `convert` function.  -->
値のさまざまな型への変換は、`convert`関数によって実行されます。
<!-- The `convert` function generally takes two arguments: the first is a type object while the second is a value to convert to that type; the returned value is the value converted to an instance of given type. -->
`convert`関数は一般に2つの引数をとります：最初のものは型オブジェクトであり、2番目の型はその型に変換する値です。 戻り値は、指定された型のインスタンスに変換された値です。
<!-- The simplest way to understand this function is to see it in action: -->
この関数を理解する最も簡単な方法は、実際の動作を確認することです。

```jldoctest
julia> x = 12
12

julia> typeof(x)
Int64

julia> convert(UInt8, x)
0x0c

julia> typeof(ans)
UInt8

julia> convert(AbstractFloat, x)
12.0

julia> typeof(ans)
Float64

julia> a = Any[1 2 3; 4 5 6]
2×3 Array{Any,2}:
 1  2  3
 4  5  6

julia> convert(Array{Float64}, a)
2×3 Array{Float64,2}:
 1.0  2.0  3.0
 4.0  5.0  6.0
```

<!-- Conversion isn't always possible, in which case a no method error is thrown indicating that `convert` doesn't know how to perform the requested conversion:-->
変換は常に可能ではありません。この場合noメソッドエラーをスローし、`convert`が要求された変換に対応する方法を知らないと示します。


```jldoctest
julia> convert(AbstractFloat, "foo")
ERROR: MethodError: Cannot `convert` an object of type String to an object of type AbstractFloat
This may have arisen from a call to the constructor AbstractFloat(...), since type constructors fall back to convert methods.
```

<!-- Some languages consider parsing strings as numbers or formatting numbers as strings to be conversions (many dynamic languages will even perform conversion for you automatically), however Julia does not: even though some strings can be parsed as numbers, most strings are not valid representations of numbers, and only a very limited subset of them are. -->
一部の言語では、文字列を数値として解析したり、数値を文字列として変換して変換することを検討しています(多くの動的言語では自動的に変換します)。Juliaは違います。 いくつかの文字列は数字として解析することができますが、ほとんどの文字列は数字の有効な表現ではなく、非常に限られたサブセットのみだからです。
<!-- Therefore in Julia the dedicated `parse()` function must be used to perform this operation, making it more explicit.-->
したがってJuliaでは、この操作をより明確にする為、専用の `parse()`関数を使わなければなりません。

<!-- ## Defining New Conversions-->
### 新しいコンバージョンを定義する

<!-- To define a new conversion, simply provide a new method for `convert()`. -->
新しい変換を定義するには、単に`convert()`へ新しいメソッドを提供するだけです。
<!-- That's really all there is to it. -->
本当にこれだけです。
<!-- For example, the method to convert a real number to a boolean is this:-->
たとえば、実数をブール値に変換する方法は次のとおりです。

```julia
convert(::Type{Bool}, x::Real) = x==0 ? false : x==1 ? true : throw(InexactError())
```

#The type of the first argument of this method is a [singleton type](@ref man-singleton-types), `Type{Bool}`, the only instance of which is [`Bool`](@ref). 
このメソッドの第1引数の型は、[singleton type](@ref man-singleton-types)、`type{Bool}`、[Bool`](@ref) のみです。
<!-- Thus, this method is only invoked when the first argument is the type value `Bool`. -->
したがって、このメソッドは、最初の引数が型値 `Bool`のときにのみ呼び出されます。
#Notice the syntax used for the first argument: the argument name is omitted prior to the `::` symbol, and only the type is given.
最初の引数に使用されている構文に注目してください。引数名は `::`前のシンボルが省略され、型だけが与えられます。
#This is the syntax in Julia for a function argument whose type is specified but whose value is never used in the function body. 
これは、Julia構文で、関数引数の型が指定されているがその値が関数本体で決して使用されていないのです。
#In this example, since the type is a singleton, there would never be any reason to use its value within the body. 
この例では、型がシングルトンであるため、その値を本体内で使用する理由はありません。
#When invoked, the method determines whether a numeric value is true or false as a boolean, by comparing it to one and zero:
このメソッドは、呼び出されると、数値を1と0を比較することで、真偽値をブール値として判断します。

```jldoctest
julia> convert(Bool, 1)
true

julia> convert(Bool, 0)
false

julia> convert(Bool, 1im)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Bool}, ::Complex{Int64}) at ./complex.jl:31

julia> convert(Bool, 0im)
false
```

<!-- The method signatures for conversion methods are often quite a bit more involved than this example, especially for parametric types. -->
変換メソッドのメソッドシグネチャは、この例よりもかなり複雑です。特にパラメトリック型のシグネチャです。
<!-- The example above is meant to be pedagogical, and is not the actual Julia behaviour. -->
上の例は教育的なものであり、実際のJuliaの動作ではありません。
<!-- This is the actual implementation in Julia:-->
これがJuliaの実際の実装です：

```julia
convert(::Type{T}, z::Complex) where {T<:Real} =
    (imag(z) == 0 ? convert(T, real(z)) : throw(InexactError()))
```

<!-- ### [Case Study: Rational Conversions](@id man-rational-conversion)-->
### [ケーススタディ：合理的な変換](@ id man-rational-conversion)

#To continue our case study of Julia's [`Rational`](@ref) type, here are the conversions declared in [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl), right after the declaration of the type and its constructors:
Juliaの `` Rational``(@ref) 型のケーススタディを続けるために、ここには `` rational.jl`で宣言された変換があります(https://github.com/JuliaLang/julia/blob/master/base/)。 rational.jl)、型の宣言の直後およびそのコンストラクタ：
```julia
convert(::Type{Rational{T}}, x::Rational) where {T<:Integer} = Rational(convert(T,x.num),convert(T,x.den))
convert(::Type{Rational{T}}, x::Integer) where {T<:Integer} = Rational(convert(T,x), convert(T,1))

function convert(::Type{Rational{T}}, x::AbstractFloat, tol::Real) where T<:Integer
    if isnan(x); return zero(T)//zero(T); end
    if isinf(x); return sign(x)//zero(T); end
    y = x
    a = d = one(T)
    b = c = zero(T)
    while true
        f = convert(T,round(y)); y -= f
        a, b, c, d = f*a+c, f*b+d, a, b
        if y == 0 || abs(a/b-x) <= tol
            return a//b
        end
        y = 1/y
    end
end
convert(rt::Type{Rational{T}}, x::AbstractFloat) where {T<:Integer} = convert(rt,x,eps(x))

convert(::Type{T}, x::Rational) where {T<:AbstractFloat} = convert(T,x.num)/convert(T,x.den)
convert(::Type{T}, x::Rational) where {T<:Integer} = div(convert(T,x.num),convert(T,x.den))
```

#jThe initial four convert methods provide conversions to rational types. 
最初の4つの変換メソッドは、有理型への変換を提供します。
#The first method converts one type of rational to another type of rational by converting the numerator and denominator to the appropriate integer type. 
第1の方法は、分子と分母を適切な整数型に変換することによって、有理型の1つの型を別の型の有理型に変換する。
#The second method does the same conversion for integers by taking the denominator to be 1. 
第2の方法は、分母を1とすることにより、整数に対して同じ変換を行う。
#The third method implements a standard algorithm for approximating a floating-point number by a ratio of integers to within a given tolerance, and the fourth method applies it, using machine epsilon at the given value as the threshold. 
第3の方法は、与えられた公差内の整数比で浮動小数点数を近似するための標準的なアルゴリズムを実装し、第4の方法は、与えられた値のマシンイプシロンを閾値として用いてそれを適用する。
#In general, one should have `a//b == convert(Rational{Int64}, a/b)`.
一般に、 `a // b == convert(Rational {Int64}、a / b)`が必要です。

#The last two convert methods provide conversions from rational types to floating-point and integer types. 
最後の2つの変換メソッドは、有理型から浮動小数点型および整数型への変換を提供します。
#To convert to floating point, one simply converts both numerator and denominator to that floating point type and then divides. 
浮動小数点に変換するには、分子と分母の両方をその浮動小数点型に変換してから分割するだけです。
#To convert to integer, one can use the `div` operator for truncated integer division (rounded towards zero).
整数に変換するには、切り捨てられた整数の除算(ゼロに丸められます)に `div`演算子を使用できます。


<!-- ## Promotion -->
## プロモーション

<!-- Promotion refers to converting values of mixed types to a single common type. -->
プロモーションは、混合型の値を単一の共通型に変換することを指します。
<!-- Although it is not strictly necessary, it is generally implied that the common type to which the values are converted can faithfully represent all of the original values. -->
厳密には必要ではありませんが、値が変換される一般的な型は元の値のすべてを忠実に表すことができます。
#In this sense, the term "promotion" is appropriate since the values are converted to a "greater" type -- i.e. one which can represent all of the input values in a single common type. 
この意味で、「プロモーション」という用語は値が「より大きい」タイプに変換されるので適切です。つまり、すべての入力値を単一の共通タイプで表すことができます。
#It is important, however, not to confuse this with object-oriented (structural) super-typing, or Julia's notion of abstract super-types: promotion has nothing to do with the type hierarchy, and everything to do with converting between alternate representations.
しかし、これをオブジェクト指向(構造的)スーパータイプ、または抽象スーパータイプのジュリアの概念と混同しないことが重要です。プロモーションはタイプ階層とは関係がなく、代替表現間の変換に関係するすべてです。
#For instance, although every [`Int32`](@ref) value can also be represented as a [`Float64`](@ref) value, `Int32` is not a subtype of `Float64`.
例えば、すべての[`Int32`](@ref) 値は[`Float64`](@ref) 値として表現することもできますが、`Int32`は `Float64`のサブタイプではありません。

#Promotion to a common "greater" type is performed in Julia by the `promote` function, which takes any number of arguments, and returns a tuple of the same number of values, converted to a common type, or throws an exception if promotion is not possible. 
一般的な「より大きな」タイプへのプロモーションは、Juliaで `promote`機能によって実行されます、 任意の数の引数をとり、同じ数の値のタプルを返すか、共通の型に変換します。プロモーションができない場合は例外をスローします。
#The most common use case for promotion is to convert numeric arguments to a common type:
プロモーションの最も一般的な使用例は、数値引数を共通の型に変換することです：

```jldoctest
julia> promote(1, 2.5)
(1.0, 2.5)

julia> promote(1, 2.5, 3)
(1.0, 2.5, 3.0)

julia> promote(2, 3//4)
(2//1, 3//4)

julia> promote(1, 2.5, 3, 3//4)
(1.0, 2.5, 3.0, 0.75)

julia> promote(1.5, im)
(1.5 + 0.0im, 0.0 + 1.0im)

julia> promote(1 + 2im, 3//4)
(1//1 + 2//1*im, 3//4 + 0//1*im)
```

#Floating-point values are promoted to the largest of the floating-point argument types. 
浮動小数点値は浮動小数点引数型のうち最大のものに昇格されます。
#Integer values are promoted to the larger of either the native machine word size or the largest integer argument type. 
整数値は、ネイティブマシンのワードサイズまたは最大の整数引数型のいずれか大きい方に昇格されます。
#Mixtures of integers and floating-point values are promoted to a floating-point type big enough to hold all the values. 
整数と浮動小数点値の行列は、すべての値を保持するのに十分な大きさの浮動小数点型に昇格されます。
#Integers mixed with rationals are promoted to rationals. 
有理数と混合された整数は有理数に昇進されます。
#Rationals mixed with floats are promoted to floats. 
浮動小数点数と混合された競合は、浮動小数点数に昇格されます。
#Complex values mixed with real values are promoted to the appropriate kind of complex value.
実数値と混合された複素数の値は、適切な種類の複素数に昇格されます。

#That is really all there is to using promotions. 
それは本当にプロモーションを使用することです。
#The rest is just a matter of clever application, the most typical "clever" application being the definition of catch-all methods for numeric operations like the arithmetic operators `+`, `-`, `*` and `/`. 
残りは単なる賢明なアプリケーションの問題であり、最も典型的な "巧妙な"アプリケーションは、算術演算子 `+`、 ` - `、 `*`や `/`のような数値演算のキャッチオールメソッドの定義です。
#Here are some of the catch-all method definitions given in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl):
[`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)にあるキャッチオールメソッドの定義をいくつか示します：

```julia
+(x::Number, y::Number) = +(promote(x,y)...)
-(x::Number, y::Number) = -(promote(x,y)...)
*(x::Number, y::Number) = *(promote(x,y)...)
/(x::Number, y::Number) = /(promote(x,y)...)
```

#These method definitions say that in the absence of more specific rules for adding, subtracting, multiplying and dividing pairs of numeric values, promote the values to a common type and then try again. 
これらのメソッド定義では、数値のペアを加算、減算、掛け算、および除算するためのより具体的なルールがない場合、値を共通タイプに昇格してからやり直します。
#That's all there is to it: nowhere else does one ever need to worry about promotion to a common numeric type for arithmetic operations -- it just happens automatically. 
これだけです：算術演算の一般的な数値型への昇格を心配する必要はありません。それはちょうど自動的に起こります。
#There are definitions of catch-all promotion methods for a number of other arithmetic and mathematical functions in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl), but beyond that, there are hardly any calls to `promote` required in the Julia standard library. 
[`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)には数多くの算術関数と数学関数のキャッチオールプロモーションメソッドの定義がありますが、それを超えて、ジュリア標準ライブラリで必要とされる「促進する」という呼びかけはほとんどありません。
#The most common usages of `promote` occur in outer constructors methods, provided for convenience, to allow constructor calls with mixed types to delegate to an inner type with fields promoted to an appropriate common type. 
`promote`のもっとも一般的な使い方は、混合型のコンストラクタ呼び出しが適切な共通型に昇格されたフィールドを持つ内部型に委譲できるように、便宜上提供される外部コンストラクタメソッドで発生します。
#For example, recall that [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl) provides the following outer constructor method:
たとえば、[`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl)は、以下の外部コンストラクタメソッドを提供しています：


```julia
Rational(n::Integer, d::Integer) = Rational(promote(n,d)...)
```

This allows calls like the following to work:

```jldoctest
julia> Rational(Int8(15),Int32(-5))
-3//1

julia> typeof(ans)
Rational{Int32}
```
#For most user-defined types, it is better practice to require programmers to supply the expected types to constructor functions explicitly, but sometimes, especially for numeric problems, it can be convenient to do promotion automatically.
ほとんどのユーザー定義型では、コンストラクターの特定関数で必要になると予想される型をプログラマーが指定することをお勧めしますが、数値問題の場合は、自動的に昇格させるのが便利です。

### Defining Promotion Rules
###プロモーションルールの定義

#Although one could, in principle, define methods for the `promote` function directly, this would require many redundant definitions for all possible permutations of argument types. 
原則として `promote`関数のメソッドを直接定義することはできますが、これは引数型のすべての可能な置換のために多くの冗長な定義を必要とします。
#Instead, the behavior of `promote` is defined in terms of an auxiliary function called `promote_rule`, which one can provide methods for. 
その代わりに、 `promote`の振る舞いは` promote_rule`と呼ばれる補助的な関数の観点から定義されています。
#The `promote_rule` function takes a pair of type objects and returns another type object, such that instances of the argument types will be promoted to the returned type. 
`promote_rule`関数は一対の型オブジェクトをとり、別の型オブジェクトを返します。その結果、引数型のインスタンスは返される型に昇格します。 

<!-- Thus, by defining the rule: -->
したがって、ルールを定義することによって：

```julia
promote_rule(::Type{Float64}, ::Type{Float32}) = Float64
```j
#one declares that when 64-bit and 32-bit floating-point values are promoted together, they should be promoted to 64-bit floating-point. 
64ビットと32ビットの浮動小数点値を一緒に昇格させると、64ビットの浮動小数点に昇格する必要があります。

#The promotion type does not need to be one of the argument types, however; the following promotion rules both occur in Julia's standard library:
ただし、プロモーションタイプは引数タイプの1つである必要はありません。 Juliaの標準ライブラリでは、次のプロモーションルールが両方とも発生します。

```julia
promote_rule(::Type{UInt8}, ::Type{Int8}) = Int
promote_rule(::Type{BigInt}, ::Type{Int8}) = BigInt
```

#In the latter case, the result type is [`BigInt`](@ref) since `BigInt` is the only type large enough to hold integers for arbitrary-precision integer arithmetic. 
後者の場合、結果の型は[BigInt`](@ref) です。なぜなら、 `BigInt`は任意の精度の整数算術のための整数を保持できる唯一の型であるからです。
#Also note that one does not need to define both `promote_rule(::Type{A}, ::Type{B})` and `promote_rule(::Type{B}, ::Type{A})` -- the symmetry is implied by the way `promote_rule` is used in the promotion process.
また、 `promote_rule(:: Type {A}、:: Type {B})`と `promote_rule(:: Type {B}、:: Type {A})`の両方を定義する必要はありません。対称性はプロモーションプロセスで `promote_rule`が使用される方法によって暗示されます。

#The `promote_rule` function is used as a building block to define a second function called `promote_type`, which, given any number of type objects, returns the common type to which those values, as arguments to `promote` should be promoted. 
`promote_rule`関数は、` promote_type`と呼ばれる第2の関数を定義するビルディングブロックとして使用されます。任意の数の型オブジェクトがあれば、 `promote`への引数としてそれらの値を宣言しなければなりません。
#Thus, if one wants to know, in absence of actual values, what type a collection of values of certain types would promote to, one can use `promote_type`:
したがって、実際の値が存在しない場合、特定の型の値の集合がどの型に昇格するかを知りたければ、 `promote_type`を使うことができます：

```jldoctest
julia> promote_type(Int8, UInt16)
Int64
```

#Internally, `promote_type` is used inside of `promote` to determine what type argument values should be converted to for promotion. 
内部的に `promote_type`はプロモーションのためにどのタイプの引数値を変換すべきかを決定するために` promote`の内部で使用されます。
<!-- It can, however, be useful in its own right.  -->
しかし、それはそれ自体では有用なことがあります。
#The curious reader can read the code in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl), which defines the complete promotion mechanism in about 35 lines.
注意深い読者は、[promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)のコードを読むことができます。これは約35行で完全なプロモーションの仕組みを定義しています。

### Case Study: Rational Promotions
###ケーススタディ：合理的プロモーション

#Finally, we finish off our ongoing case study of Julia's rational number type, which makes relatively sophisticated use of the promotion mechanism with the following promotion rules:
最後に、Juliaの有理数型に関する継続的なケーススタディを終了します。これは、次のプロモーションルールでプロモーションメカニズムを比較的洗練された形で使用します。

```julia
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{Rational{S}}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:AbstractFloat} = promote_type(T,S)
```

#The first rule says that promoting a rational number with any other integer type promotes to a rational type whose numerator/denominator type is the result of promotion of its numerator/denominator type with the other integer type. 
第1の規則は、他の整数型で有理数を宣伝することは、分子/分母型が他の整数型の分子/分母型の宣伝の結果である有理型に昇格することを示している。
#The second rule applies the same logic to two different types of rational numbers, resulting in a rational of the promotion of their respective numerator/denominator types. 
第2の規則は、同じ論理を2つの異なるタイプの有理数に適用し、その結果、それぞれの分子/分母型の推進を合理的にする。
#The third and final rule dictates that promoting a rational with a float results in the same type as promoting the numerator/denominator type with the float.
最後の3つ目のルールは、floatを使用して有理数を昇格すると、floatで分子/分母型を昇格させるのと同じ型になります。

#This small handful of promotion rules, together with the [conversion methods discussed above](@ref man-rational-conversion), are sufficient to make rational numbers interoperate completely naturally with all of Julia's other numeric types -- integers, floating-point numbers, and complex numbers. 
このような少数のプロモーションルールは、[上記の変換方法](@ref man-rational-conversion)と共に、有理数をJuliaの他の数値型 - 整数、浮動小数点数、および複素数。
#By providing appropriate conversion methods and promotion rules in the same manner, any user-defined numeric type can interoperate just as naturally with Julia's predefined numerics.
適切な変換方法とプロモーションルールを同じ方法で提供することで、ユーザー定義の数値型は、Juliaの事前定義された数値と自然に同じように相互運用できます。
