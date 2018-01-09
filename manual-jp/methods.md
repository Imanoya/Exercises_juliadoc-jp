<!-- Methods -->
# メソッド

<!-- Recll from [Functions](@ref man-functions) that a function is an object that maps a tuple of arguments to a return value, or throws an exception if no appropriate value can be returned. -->
[関数](@ref man-functions)はオブジェクトであり、引数のタプルを戻り値にマップし、適切な値が返されない場合は例外をスローすることを思い出してください。
#It is common for the same conceptual function or operation to be implemented quite differently for different types of arguments: adding two integers is very different from adding two floating-point numbers, both of which are distinct from adding an integer to a floating-point number. 
同じ概念的な関数または操作が、異なるタイプの引数に対して全く異なって実装されることは一般的であり、2つの整数を加算することは、2つの浮動小数点数を加算することとは非常に異なります。 どちらも浮動小数点数に整数を追加するのとも異なります。
<!-- Desite their implementation differences, these operations all fall under the general concept of "addition".  -->
実装の違いにもかかわらず、これらの操作はすべて「追加」という一般的な概念に当てはまります。
<!-- Accrdingly, in Julia, these behaviors all belong to a single object: the `+` function. -->
したがって、Juliaでは、これらの動作はすべて単一のオブジェクトに属します:`+`関数。

<!-- To acilitate using many different implementations of the same concept smoothly, functions need not be defined all at once, but can rather be defined piecewise by providing specific behaviors for certain combinations of argument types and counts.  -->
同じコンセプトのさまざまな実装をスムーズに使用するために、関数を一度にすべて定義する必要はありませんが、引数の型と数の特定の組み合わせに対して特定の動作を指定することによって、区分的に定義できます。
<!-- A dfinition of one possible behavior for a function is called a *method*.  -->
関数の1つの可能な振る舞いの定義は、*メソッド*と呼ばれます。
<!-- Thu far, we have presented only examples of functions defined with a single method, applicable to all types of arguments.  -->
ここまでは、単一のメソッドで定義された関数の例のみを示しましたが、すべての型の引数に適用できます。
<!-- Howver, the signatures of method definitions can be annotated to indicate the types of arguments in addition to their number, and more than a single method definition may be provided.  -->
ただし、メソッド定義のシグネチャに注釈を付けて、数に加えて引数のタイプを示すことができ、複数のメソッド定義を提供することができます。
<!-- Whe a function is applied to a particular tuple of arguments, the most specific method applicable to those arguments is applied.  -->
関数が特定の組の引数に適用されるとき、それらの引数に適用可能な最も具体的な方法が適用されます。
#Thus, the overall behavior of a function is a patchwork of the behaviors of its various method definitions.
したがって、関数の全体的な動作は、さまざまなメソッド定義の動作のパッチワークです。
#If the patchwork is well designed, even though the implementations of the methods may be quite different, the outward behavior of the function will appear seamless and consistent.
パッチワークがうまく設計されている場合、メソッドの実装が全く異なる場合でも、関数の外向きの動作はシームレスで一貫性があります。

<!-- Thechoice of which method to execute when a function is applied is called *dispatch*.  -->
関数が適用されたときに実行するメソッドの選択は、*dispatch*と呼ばれます。
#Julia allows the dispatch process to choose which of a function's methods to call based on the number of arguments given, and on the types of all of the function's arguments. 
Juliaは、ディスパッチプロセスが与えられた引数の数に基づいてどの関数のメソッドのうちどれを呼び出すかを選択し、関数のすべての引数のタイプを選択することができます。
#This is different than traditional object-oriented languages, where dispatch occurs based only on the first argument, which often has a special argument syntax, and is sometimes implied rather than explicitly written as an argument.
これは、通常は特殊な引数構文を持つ最初の引数のみに基づいてディスパッチが行われる従来のオブジェクト指向言語とは異なり、引数として明示的に記述されるのではなく暗示されることがあります。
#[^1] Using all of a function's arguments to choose which method should be invoked, rather than just the first, is known as [multiple dispatch](https://en.wikipedia.org/wiki/Multiple_dispatch).
[^ 1]すべての関数の引数を使って、最初に呼び出すメソッドを呼び出すのではなく、呼び出すメソッドを選択することは[マルチディスパッチ](https://en.wikipedia.org/wiki/Multiple_dispatch)として知られています。
#Multiple dispatch is particularly useful for mathematical code, where it makes little sense to artificially deem the operations to "belong" to one argument more than any of the others: does the addition operation in `x + y` belong to `x` any more than it does to `y`? The implementation of a mathematical operator generally depends on the types of all of its arguments. 
マルチプルディスパッチは、数学的コードにとって特に有用であり、人為的に他のものよりも1つの引数に「属し」ていると人工的に判断するのはほとんど意味がない：x + yの加算演算はもはやxに属する「y」よりも数学的演算子の実装は、一般に、そのすべての引数の型に依存する。
#Even beyond mathematical operations, however, multiple dispatch ends up being a powerful and convenient paradigm for structuring and organizing programs.
しかし、数学的な操作を越えても、複数のディスパッチは、プログラムの構造化と整理のためのパワフルで便利なパラダイムになります。

[^1]:
   <!--  
    In C++ or Java, for example, in a method call like `obj.meth(arg1,arg2)`, the object obj "receives" the method call and is implicitly passed to the method via the `this` keyword, rather than as an explicit method argument.
   -->
    例えば、 `obj.meth(arg1、arg2)`のようなメソッド呼び出しで、C ++やJavaでは、オブジェクトobjはメソッド呼び出しを受け取り、 `this`キーワードでメソッドに暗黙的に渡されます。 明示的なメソッド引数

   <!--
    When the current `this` object is the receiver of a method call, it can be omitted altogether, writing just `meth(arg1,arg2)`, with `this` implied as the receiving object.
    -->
    現在の `this`オブジェクトがメソッド呼び出しの受信者である場合、` meth(arg1、arg2) `だけを` this`を受信オブジェクトとして暗黙的に書くことで、これを省略することができます。

<!-- # Dfining Methods -->
##メソッドの定義

<!-- Untl now, we have, in our examples, defined only functions with a single method having unconstrained argument types.  -->
これまでの例では、制約のない型を持つ単一のメソッドを持つ関数しか定義していませんでした。
<!-- Suc functions behave just like they would in traditional dynamically typed languages. -->
そのような関数は、従来の動的に型指定された言語と同じように動作します。
#Nevertheless, we have used multiple dispatch and methods almost continually without being aware of it: all of Julia's standard functions and operators, like the aforementioned `+` function, have many methods defining their behavior over various possible combinations of argument type and count.
それにもかかわらず、私たちはそれを認識することなく、ほとんど連続的にディスパッチとメソッドを使用してきました。前述の `+`関数のようなJuliaの標準関数と演算子のすべては、引数の型とカウントのさまざまな組み合わせに対して動作を定義する多くのメソッドを持っています。

#When defining a function, one can optionally constrain the types of parameters it is applicable to, using the `::` type-assertion operator, introduced in the section on [Composite Types](@ref):
関数を定義するとき、[Composite Types](@ref)の節で紹介した:: :: type-assertion演算子を使って、適用可能なパラメータの型を任意に制約することができます：

```jldoctest fofxy
julia> f(x::Float64, y::Float64) = 2x + y
f (generic function with 1 method)
```

<!-- Thi function definition applies only to calls where `x` and `y` are both values of type [`Float64`](@ref): -->
この関数定義は`x`と`y`が両方とも[`Float64`](@ref)型の値である呼び出しにのみ適用されます：

```jldoctest fofxy
julia> f(2.0, 3.0)
7.0
```

<!-- Appying it to any other types of arguments will result in a [`MethodError`](@ref): -->
それを他の型の引数に適用すると、[`MethodError`](@ref)になります：

```jldoctest fofxy
julia> f(2.0, 3)
ERROR: MethodError: no method matching f(::Float64, ::Int64)
Closest candidates are:
  f(::Float64, !Matched::Float64) at none:1

julia> f(Float32(2.0), 3.0)
ERROR: MethodError: no method matching f(::Float32, ::Float64)
Closest candidates are:
  f(!Matched::Float64, ::Float64) at none:1

julia> f(2.0, "3.0")
ERROR: MethodError: no method matching f(::Float64, ::String)
Closest candidates are:
  f(::Float64, !Matched::Float64) at none:1

julia> f("2.0", "3.0")
ERROR: MethodError: no method matching f(::String, ::String)
```

<!-- As ou can see, the arguments must be precisely of type [`Float64`](@ref).  -->
ご覧のように、引数は正確に[`Float64`](@ref)型でなければなりません。
<!-- Othr numeric types, such as integers or 32-bit floating-point values, are not automatically converted to 64-bit floating-point, nor are strings parsed as numbers. -->
整数や32ビット浮動小数点値などの他の数値型は自動的に64ビット浮動小数点に変換されず、数字として解析される文字列もありません。
<!-- Becuse `Float64` is a concrete type and concrete types cannot be subclassed in Julia, such a definition can only be applied to arguments that are exactly of type `Float64`. -->
`Float64`は具体的な型であり、具体的な型はJuliaでサブクラス化できないので、そのような定義は`Float64`型の引数にしか適用できません。
<!-- It ay often be useful, however, to write more general methods where the declared parameter types are abstract: -->
しかし、宣言されたパラメータ型が抽象的なより一般的なメソッドを記述することは、しばしば役に立ちます：

```jldoctest fofxy
julia> f(x::Number, y::Number) = 2x - y
f (generic function with 2 methods)

julia> f(2.0, 3)
1.0
```

<!-- Thi method definition applies to any pair of arguments that are instances of [`Number`](@ref). -->
このメソッド定義は、[`Number`](@ref)のインスタンスである任意の引数のペアに適用されます。
<!-- The need not be of the same type, so long as they are each numeric values.  -->
それぞれが数値である限り、同じ型である必要はありません。
<!-- The problem of handling disparate numeric types is delegated to the arithmetic operations in the expression `2x - y`. -->
異種の数値型を扱う問題は、式 `2x - y`の算術演算に委譲されます。

#To define a function with multiple methods, one simply defines the function multiple times, with different numbers and types of arguments. 
複数のメソッドを持つ関数を定義するには、異なる数と型の引数を使用して関数を複数回定義するだけです。
#The first method definition for a function creates the function object, and subsequent method definitions add new methods to the existing function object. 
関数の最初のメソッド定義は関数オブジェクトを作成し、後続のメソッド定義は既存の関数オブジェクトに新しいメソッドを追加します。
#The most specific method definition matching the number and types of the arguments will be executed when the function is applied. 
引数の数と型に一致する最も具体的なメソッド定義は、関数が適用されたときに実行されます。
#Thus, the two method definitions above, taken together, define the behavior for `f` over all pairs of instances of the abstract type `Number` -- but with a different behavior specific to pairs of [`Float64`](@ref) values. 
したがって、上記の2つのメソッド定義はまとめて、抽象型 `Number`のインスタンスのすべてのペアに対して` f`の振る舞いを定義しますが、[`Float64`](@ref)値。
#If one of the arguments is a 64-bit float but the other one is not, then the `f(Float64,Float64)` method cannot be called and the more general `f(Number,Number)` method must be used:
引数の1つが64ビット浮動小数点型でもう1つが浮動小数点型でない場合、 `f(Float64、Float64)`メソッドは呼び出すことができず、より一般的な `f(Number、Number)`メソッドを使用する必要があります：

```jldoctest fofxy
julia> f(2.0, 3.0)
7.0

julia> f(2, 3.0)
1.0

julia> f(2.0, 3)
1.0

julia> f(2, 3)
1
```

#The `2x + y` definition is only used in the first case, while the `2x - y` definition is used in the others. 
`2x + y`定義は最初の場合にのみ使用され、` 2x-y`定義は他の場合に使用されます。
#No automatic casting or conversion of function arguments is ever performed: all conversion in Julia is non-magical and completely explicit. 
関数の引数の自動キャストまたは変換はこれまでに実行されていません.Juliaのすべての変換は非魔法で完全に明示的です。
#[Conversion and Promotion](@ref conversion-and-promotion), however, shows how clever application of sufficiently advanced technology can be indistinguishable from magic. [^Clarke61]
しかし、「変換とプロモーション」(@ref conversion-and-promotion)は、十分に高度な技術の巧妙な応用が魔法とどのように区別できないかを示しています。 [^ Clarke61]

#For non-numeric values, and for fewer or more than two arguments, the function `f` remains undefined, and applying it will still result in a [`MethodError`](@ref):
数値以外の値や2つ以上の引数に対しては、関数fは未定義のままであり、それを適用すると[`MethodError`](@ref)になります。

```jldoctest fofxy
julia> f("foo", 3)
ERROR: MethodError: no method matching f(::String, ::Int64)
Closest candidates are:
  f(!Matched::Number, ::Number) at none:1

julia> f()
ERROR: MethodError: no method matching f()
Closest candidates are:
  f(!Matched::Float64, !Matched::Float64) at none:1
  f(!Matched::Number, !Matched::Number) at none:1
```

<!-- Youcan easily see which methods exist for a function by entering the function object itself in an interactive session: -->
対話型セッションで関数オブジェクト自体を入力すると、関数のどのメソッドが存在するかを簡単に確認できます。

```jldoctest fofxy
julia> f
f (generic function with 2 methods)
```

<!-- Thi output tells us that `f` is a function object with two methods. To find out what the signatures of those methods are, use the [`methods()`](@ref) function: -->
この出力は `f`が2つのメソッドを持つ関数オブジェクトであることを示しています。 これらのメソッドのシグネチャを調べるには、[`methods()`](@ref)関数を使います：

```julia-repl
julia> methods(f)
# 2 methods for generic function "f":
[1] f(x::Float64, y::Float64) in Main at none:1
[2] f(x::Number, y::Number) in Main at none:1
```

<!-- whih shows that `f` has two methods, one taking two `Float64` arguments and one taking arguments of type `Number`.  -->
`f`は2つのメソッドを持ち、[1]つは2つの`Float64`引数を取り、[2]は `Number`型の引数を取ることを示しています。
#It also indicates the file and line number where the methods were defined: because these methods were defined at the REPL, we get the apparent line number `none:1`.
また、メソッドが定義されたファイルと行番号を示します。これらのメソッドはREPLで定義されているため、見かけの行番号は`none：1`になります。

#In the absence of a type declaration with `::`, the type of a method parameter is `Any` by default, meaning that it is unconstrained since all values in Julia are instances of the abstract type `Any`. 
`::`で型宣言がない場合、メソッドパラメータの型はデフォルトで `Any`です。つまり、Juliaのすべての値は抽象型` Any`のインスタンスなので、制約されません。
#Thus, we can define a catch-all method for `f` like so:
したがって、 `f`のキャッチオールメソッドを以下のように定義することができます：

```jldoctest fofxy
julia> f(x,y) = println("Whoa there, Nelly.")
f (generic function with 3 methods)

julia> f("foo", 1)
Whoa there, Nelly.
```

#This catch-all is less specific than any other possible method definition for a pair of parameter values, so it will only be called on pairs of arguments to which no other method definition applies.
このcatch-allは、パラメータ値のペアの他の可能なメソッド定義よりも具体的ではないため、他のメソッド定義が適用されない引数のペアに対してのみ呼び出されます。

#Although it seems a simple concept, multiple dispatch on the types of values is perhaps the single most powerful and central feature of the Julia language. 
それは単純なコンセプトに見えますが、値の型に対する複数のディスパッチは、おそらくジュリア言語の最も強力で中心的な機能です。
#Core operations typically have dozens of methods:
コアオペレーションには通常数十種類のメソッドがあります。

```julia-repl
julia> methods(+)
# 180 methods for generic function "+":
[1] +(x::Bool, z::Complex{Bool}) in Base at complex.jl:227
[2] +(x::Bool, y::Bool) in Base at bool.jl:89
[3] +(x::Bool) in Base at bool.jl:86
[4] +(x::Bool, y::T) where T<:AbstractFloat in Base at bool.jl:96
[5] +(x::Bool, z::Complex) in Base at complex.jl:234
[6] +(a::Float16, b::Float16) in Base at float.jl:373
[7] +(x::Float32, y::Float32) in Base at float.jl:375
[8] +(x::Float64, y::Float64) in Base at float.jl:376
[9] +(z::Complex{Bool}, x::Bool) in Base at complex.jl:228
[10] +(z::Complex{Bool}, x::Real) in Base at complex.jl:242
[11] +(x::Char, y::Integer) in Base at char.jl:40
[12] +(c::BigInt, x::BigFloat) in Base.MPFR at mpfr.jl:307
[13] +(a::BigInt, b::BigInt, c::BigInt, d::BigInt, e::BigInt) in Base.GMP at gmp.jl:392
[14] +(a::BigInt, b::BigInt, c::BigInt, d::BigInt) in Base.GMP at gmp.jl:391
[15] +(a::BigInt, b::BigInt, c::BigInt) in Base.GMP at gmp.jl:390
[16] +(x::BigInt, y::BigInt) in Base.GMP at gmp.jl:361
[17] +(x::BigInt, c::Union{UInt16, UInt32, UInt64, UInt8}) in Base.GMP at gmp.jl:398
...
[180] +(a, b, c, xs...) in Base at operators.jl:424
```

#Multiple dispatch together with the flexible parametric type system give Julia its ability to abstractly express high-level algorithms decoupled from implementation details, yet generate efficient, specialized code to handle each case at run time.
柔軟なパラメトリック・タイプ・システムと複数のディスパッチを行うことで、Juliaは実装の詳細から切り離された高水準のアルゴリズムを抽象的に表現できますが、実行時に各ケースを処理する効率的で特殊なコードを生成できます。

## [Method Ambiguities](@id man-ambiguities)
## [メソッドの曖昧さ](@ idの曖昧さ)

#It is possible to define a set of function methods such that there is no unique most specific method applicable to some combinations of arguments:
いくつかの引数の組み合わせに適用可能な一意の最も具体的なメソッドが存在しないような関数メソッドのセットを定義することは可能です。

```jldoctest gofxy
julia> g(x::Float64, y) = 2x + y
g (generic function with 1 method)

julia> g(x, y::Float64) = x + 2y
g (generic function with 2 methods)

julia> g(2.0, 3)
7.0

julia> g(2, 3.0)
8.0

julia> g(2.0, 3.0)
ERROR: MethodError: g(::Float64, ::Float64) is ambiguous.
[...]
```

#Here the call `g(2.0, 3.0)` could be handled by either the `g(Float64, Any)` or the `g(Any, Float64)` method, and neither is more specific than the other.
ここで、 `g(2.0,3.0)`は `g(Float64、Any)`や `g(Any、Float64)`メソッドで扱うことができます。
#In such cases, Julia raises a [`MethodError`](@ref) rather than arbitrarily picking a method. 
そのような場合、Juliaはメソッドを任意に選択するのではなく、[`MethodError`](@ref)を送出します。
#You can avoid method ambiguities by specifying an appropriate method for the intersection case:
交差事例に適切な方法を指定することによって、メソッドのあいまい性を避けることができます。

```jldoctest gofxy
julia> g(x::Float64, y::Float64) = 2x + 2y
g (generic function with 3 methods)

julia> g(2.0, 3)
7.0

julia> g(2, 3.0)
8.0

julia> g(2.0, 3.0)
10.0
```

#It is recommended that the disambiguating method be defined first, since otherwise the ambiguity exists, if transiently, until the more specific method is defined.
一時的であれば、より具体的な方法が定義されるまで、あいまい性が存在するので、最初に曖昧さ回避法を定義することを推奨します。

#In more complex cases, resolving method ambiguities involves a certain element of design; this topic is explored further [below](@ref man-method-design-ambiguities).
より複雑なケースでは、解決方法のあいまいさは設計の特定の要素を伴います。 この話題は、さらに[以下の](@ref man-method-design-ambiguities)を探求しています。

<!-- # Prametric Methods -->
##パラメトリックメソッド

#Method definitions can optionally have type parameters qualifying the signature:
メソッド定義には、シグネチャを修飾する型パラメータをオプションで持つことができます。

```jldoctest same_typefunc
julia> same_type(x::T, y::T) where {T} = true
same_type (generic function with 1 method)

julia> same_type(x,y) = false
same_type (generic function with 2 methods)
```

<!-- Thefirst method applies whenever both arguments are of the same concrete type, regardless of what type that is, while the second method acts as a catch-all, covering all other cases.  -->
第1の方法は、両方の引数が同じ具体的な型であるときはいつでも適用されます。どちらの型であろうと、第2の方法はキャッチオールとして機能し、他のすべてのケースをカバーします。
<!-- Thu, overall, this defines a boolean function that checks whether its two arguments are of the same type: -->
したがって、これは全体的に、2つの引数が同じ型であるかどうかをチェックするブール関数を定義します。

```jldoctest same_typefunc
julia> same_type(1, 2)
true

julia> same_type(1, 2.0)
false

julia> same_type(1.0, 2.0)
true

julia> same_type("foo", 2.0)
false

julia> same_type("foo", "bar")
true

julia> same_type(Int32(1), Int64(2))
false
```

<!-- Suc definitions correspond to methods whose type signatures are `UnionAll` types (see [UnionAll Types](@ref)). -->
このような定義は、型シグネチャが `UnionAll`型であるメソッドに対応します([UnionAll Types](@ref)を参照)。

<!-- Thi kind of definition of function behavior by dispatch is quite common -- idiomatic, even -- in Julia.  -->
Juliaでは、このようなディスパッチによる関数の振る舞いの定義は、かなり一般的です - 慣用的です。
#Method type parameters are not restricted to being used as the types of arguments: they can be used anywhere a value would be in the signature of the function or body of the function.
メソッド型パラメータは、引数の型として使用されることに限定されず、関数または関数の本体のシグネチャに値がある場合はどこでも使用できます。
#Here's an example where the method type parameter `T` is used as the type parameter to the parametric type `Vector{T}` in the method signature:
メソッドの型パラメータ `T`がメソッドシグネチャのパラメータ型`Vector {T}`の型パラメータとして使用される例を次に示します。

```jldoctest
julia> myappend(v::Vector{T}, x::T) where {T} = [v..., x]
myappend (generic function with 1 method)

julia> myappend([1,2,3],4)
4-element Array{Int64,1}:
 1
 2
 3
 4

julia> myappend([1,2,3],2.5)
ERROR: MethodError: no method matching myappend(::Array{Int64,1}, ::Float64)
Closest candidates are:
  myappend(::Array{T,1}, !Matched::T) where T at none:1

julia> myappend([1.0,2.0,3.0],4.0)
4-element Array{Float64,1}:
 1.0
 2.0
 3.0
 4.0

julia> myappend([1.0,2.0,3.0],4)
ERROR: MethodError: no method matching myappend(::Array{Float64,1}, ::Int64)
Closest candidates are:
  myappend(::Array{T,1}, !Matched::T) where T at none:1
```

<!-- As ou can see, the type of the appended element must match the element type of the vector it is appended to, or else a [`MethodError`](@ref) is raised.  -->
ご覧のように、追加される要素の型は、それが追加されるベクトルの要素の型と一致しなければなりません。さもなければ、[`MethodError`](@ref)が呼び出されます。
<!-- In he following example, the method type parameter `T` is used as the return value: -->
次の例では、メソッド型パラメータ`T`が戻り値として使用されています。

```jldoctest
julia> mytypeof(x::T) where {T} = T
mytypeof (generic function with 1 method)

julia> mytypeof(1)
Int64

julia> mytypeof(1.0)
Float64
```

<!-- Jus as you can put subtype constraints on type parameters in type declarations (see [Parametric Types](@ref)), you can also constrain type parameters of methods: -->
型宣言の型パラメータにサブタイプ制約を入れることができるように([パラメトリック型](@ref)を参照)、メソッドの型パラメータを制約することもできます：

```jldoctest
julia> same_type_numeric(x::T, y::T) where {T<:Number} = true
same_type_numeric (generic function with 1 method)

julia> same_type_numeric(x::Number, y::Number) = false
same_type_numeric (generic function with 2 methods)

julia> same_type_numeric(1, 2)
true

julia> same_type_numeric(1, 2.0)
false

julia> same_type_numeric(1.0, 2.0)
true

julia> same_type_numeric("foo", 2.0)
ERROR: MethodError: no method matching same_type_numeric(::String, ::Float64)
Closest candidates are:
  same_type_numeric(!Matched::T<:Number, ::T<:Number) where T<:Number at none:1
  same_type_numeric(!Matched::Number, ::Number) at none:1

julia> same_type_numeric("foo", "bar")
ERROR: MethodError: no method matching same_type_numeric(::String, ::String)

julia> same_type_numeric(Int32(1), Int64(2))
false
```

<!-- The`same_type_numeric` function behaves much like the `same_type` function defined above, but is only defined for pairs of numbers. -->
`same_type_numeric`関数は、上で定義した`same_type`関数と同じように動作しますが、数字のペアに対してのみ定義されます。

<!-- Parmetric methods allow the same syntax as `where` expressions used to write types (see [UnionAll Types](@ref)). -->
パラメトリックメソッドでは、型の書き込みに `where`式と同じ構文を使用できます([UnionAll Types](@ref)を参照)。
#If there is only a single parameter, the enclosing curly braces (in `where {T}`) can be omitted, but are often preferred for clarity.
パラメータが1つしかない場合は、(`{where {T}`の)中括弧は省略できますが、わかりやすくするためにしばしば好まれます。
#Multiple parameters can be separated with commas, e.g. `where {T, S<:Real}`, or written using nested `where`, e.g. `where S<:Real where T`.
複数のパラメータはカンマで区切ることができます。 `where {T、S <：Real}`、または入れ子の `where`を使って書かれています。 `S <：Real where T`。

#Redefining Methods
メソッドの再定義
------------------

<!-- Whe redefining a method or adding new methods, it is important to realize that these changes don't take effect immediately. -->
メソッドを再定義したり、新しいメソッドを追加したりするときは、これらの変更が即座に有効にならないことを認識することが重要です。
#This is key to Julia's ability to statically infer and compile code to run fast, without the usual JIT tricks and overhead.
これは、通常のJITトリックやオーバーヘッドがなくても、静的に推論してコードをコンパイルするJuliaの能力にとって重要です。
#Indeed, any new method definition won't be visible to the current runtime environment, including Tasks and Threads (and any previously defined `@generated` functions).
実際、新しいメソッド定義は、タスクやスレッド(以前に定義された `@generated`関数など)を含め、現在の実行時環境では見えません。
<!-- Lets start with an example to see what this means: -->
これが何を意味するのかを見てみましょう。

```julia-repl
julia> function tryeval()
           @eval newfun() = 1
           newfun()
       end
tryeval (generic function with 1 method)

julia> tryeval()
ERROR: MethodError: no method matching newfun()
The applicable method may be too new: running in world age xxxx1, while current world is xxxx2.
Closest candidates are:
  newfun() at none:1 (method too new to be called from this world context.)
 in tryeval() at none:1
 ...

julia> newfun()
1
```

#In this example, observe that the new definition for `newfun` has been created, but can't be immediately called.
この例では、`newfun`の新しい定義が作成されましたが、すぐに呼び出すことはできません。
#The new global is immediately visible to the `tryeval` function, so you could write `return newfun` (without parentheses).
新しいグローバルは `tryeval`関数にすぐに見えるので、` return newfun`(括弧なし)を書くことができます。
#But neither you, nor any of your callers, nor the functions they call, or etc. can call this new method definition!
しかし、あなたも呼び出し元の関数も、呼び出す関数なども、この新しいメソッド定義を呼び出すことはできません！

#But there's an exception: future calls to `newfun` *from the REPL* work as expected, being able to both see and call the new definition of `newfun`.
しかし例外があります：REPL *からの `newfun` *への将来の呼び出しは、` newfun`の新しい定義を見たり呼び出したりすることができます。

#However, future calls to `tryeval` will continue to see the definition of `newfun` as it was *at the previous statement at the REPL*, and thus before that call to `tryeval`.
しかし、 `tryeval`への今後の呼び出しでは、` tryeval`への呼び出しの前に、REPL *の前のステートメントと同じように `newfun`の定義が見られ続けます。

#You may want to try this for yourself to see how it works.
それがどのように動作するかを見るためにこれを試してみてください。

#The implementation of this behavior is a "world age counter".
この行動の実装は「世界時代のカウンター」です。
#This monotonically increasing value tracks each method definition operation.
この単調に増加する値は、各メソッド定義操作を追跡します。
#This allows describing "the set of method definitions visible to a given runtime environment" as a single number, or "world age".
これにより、「与えられた実行時環境に可視のメソッド定義のセット」を単一の数値または「世界時代」として記述することができます。
#It also allows comparing the methods available in two worlds just by comparing their ordinal value.
また、2つの世界で利用可能なメソッドを、順序値を比較するだけで比較することもできます。
#In the example above, we see that the "current world" (in which the method `newfun()` exists), is one greater than the task-local "runtime world" that was fixed when the execution of `tryeval` started.
上記の例では、 `new world`(` newfun() `メソッドが存在する)は、` tryeval`の実行が開始されたときに修正されたタスクローカル "ランタイムワールド"よりも大きいものです。

#Sometimes it is necessary to get around this (for example, if you are implementing the above REPL).
場合によっては、これを回避する必要があります(たとえば、上記のREPLを実装している場合など)。
#Fortunately, there is an easy solution: call the function using [`Base.invokelatest`](@ref):
幸いにも、簡単な解決策があります：[`Base.invokelatest`](@ref)を使って関数を呼び出します：

```jldoctest
julia> function tryeval2()
           @eval newfun2() = 2
           Base.invokelatest(newfun2)
       end
tryeval2 (generic function with 1 method)

julia> tryeval2()
2
```

<!-- Finlly, let's take a look at some more complex examples where this rule comes into play. -->
最後に、このルールが適用されるいくつかのより複雑な例を見てみましょう。
<!-- Defne a function `f(x)`, which initially has one method: -->
最初に1つのメソッドを持つ関数`f(x)`を定義する：

```jldoctest redefinemethod
julia> f(x) = "original definition"
f (generic function with 1 method)
```

<!-- Stat some other operations that use `f(x)`: -->
`f(x)`を使う他の操作を始めてください：

```jldoctest redefinemethod
julia> g(x) = f(x)
g (generic function with 1 method)

julia> t = @async f(wait()); yield();
```

<!-- Nowwe add some new methods to `f(x)`: -->
さて、 `f(x)`にいくつかの新しいメソッドを追加します：

```jldoctest redefinemethod
julia> f(x::Int) = "definition for Int"
f (generic function with 2 methods)

julia> f(x::Type{Int}) = "definition for Type{Int}"
f (generic function with 3 methods)
```

<!-- Comare how these results differ: -->
これらの結果がどのように異なるかを比較する：

```jldoctest redefinemethod
julia> f(1)
"definition for Int"

julia> g(1)
"definition for Int"

julia> wait(schedule(t, 1))
"original definition"

julia> t = @async f(wait()); yield();

julia> wait(schedule(t, 1))
"definition for Int"
```

<!-- ## arametrically-constrained Varargs methods -->
##パラメトリック制約付きVarargsメソッド


<!-- Funtion parameters can also be used to constrain the number of arguments that may be supplied to a "varargs" function ([Varargs Functions](@ref)).   -->
関数パラメータは、 "varargs"関数([Varargs Functions](@ref))に供給される引数の数を制限するためにも使用できます。
<!-- Thenotation `Vararg{T,N}` is used to indicate such a constraint.   -->
`Vararg{T,N}`という表記は、そのような制約を示すために使用されます。
<!-- Forexample: -->
例えば：

```jldoctest
julia> bar(a,b,x::Vararg{Any,2}) = (a,b,x)
bar (generic function with 1 method)

julia> bar(1,2,3)
ERROR: MethodError: no method matching bar(::Int64, ::Int64, ::Int64)
Closest candidates are:
  bar(::Any, ::Any, ::Any, !Matched::Any) at none:1

julia> bar(1,2,3,4)
(1, 2, (3, 4))

julia> bar(1,2,3,4,5)
ERROR: MethodError: no method matching bar(::Int64, ::Int64, ::Int64, ::Int64, ::Int64)
Closest candidates are:
  bar(::Any, ::Any, ::Any, ::Any) at none:1
```

#More usefully, it is possible to constrain varargs methods by a parameter.
もっと便利なことに、パラメータでvarargsメソッドを制約することは可能です。
<!-- Forexample: -->
例えば：

```julia
function getindex(A::AbstractArray{T,N}, indexes::Vararg{Number,N}) where {T,N}
```

#would be called only when the number of `indexes` matches the dimensionality of the array.
`インデックスの数が配列の次元数と一致する場合にのみ呼び出されます。

#When only the type of supplied arguments needs to be constrained `Vararg{T}` can be equivalently written as `T...`. 
与えられた引数の型だけを制約する必要があるとき、 `Vararg {T}`は `T ...`と等価に書くことができます。
#For instance `f(x::Int...) = x` is a shorthand for `f(x::Vararg{Int}) = x`.
例えば `f(x :: Int ...)= x`は` f(x :: Vararg {Int})= x`の省略形です。

<!-- # Nte on Optional and keyword Arguments -->
##オプション引数とキーワード引数の注意

#As mentioned briefly in [Functions](@ref man-functions), optional arguments are implemented as syntax for multiple method definitions. 
[Functions](@ref man-functions)で簡単に述べたように、オプション引数は複数のメソッド定義の構文として実装されています。
#For example, this definition:
たとえば、次の定義があります。

```julia
f(a=1,b=2) = a+2b
```

<!-- traslates to the following three methods: -->
次の3つの方法に変換されます。

```julia
f(a,b) = a+2b
f(a) = f(a,2)
f() = f(1,2)
```

#This means that calling `f()` is equivalent to calling `f(1,2)`. In this case the result is `5`, because `f(1,2)` invokes the first method of `f` above. However, this need not always be the case.
これは `f()`を呼び出すことは `f(1,2)`を呼び出すことと等価であることを意味します。 この場合、 `f`(1,2)は上の` f`の最初のメソッドを呼び出すため、結果は `5`です。 しかしながら、必ずしもそうである必要はない。
#If you define a fourth method that is more specialized for integers:
整数に特化した4番目のメソッドを定義した場合は、次のようになります。

```julia
f(a::Int,b::Int) = a-2b
```

#then the result of both `f()` and `f(1,2)` is `-3`. In other words, optional arguments are tied to a function, not to any specific method of that function. 
`f()`と `f(1,2)`の両方の結果は `-3`です。 言い換えれば、オプションの引数は、その関数の特定のメソッドではなく、関数に結び付けられます。
#It depends on the types of the optional arguments which method is invoked. When optional arguments are defined in terms of a global variable, the type of the optional argument may even change at run-time.
メソッドが呼び出されるオプション引数の型によって異なります。 オプションの引数がグローバル変数で定義されている場合、オプションの引数の型は実行時に変更されることさえあります。

#Keyword arguments behave quite differently from ordinary positional arguments. 
キーワード引数は、通常の位置引数とはまったく異なる動作をします。
#In particular, they do not participate in method dispatch. Methods are dispatched based only on positional arguments, with keyword arguments processed after the matching method is identified.
特に、メソッドディスパッチには参加しません。 メソッドは、位置引数だけに基づいて送出され、一致するメソッドが識別された後にキーワード引数が処理されます。

<!-- # Fnction-like objects -->
##関数のようなオブジェクト

#Methods are associated with types, so it is possible to make any arbitrary Julia object "callable" by adding methods to its type. 
メソッドは型に関連付けられているので、その型にメソッドを追加することで、任意のJuliaオブジェクトを「呼び出し可能」にすることができます。
<!-- (Suh "callable" objects are sometimes called "functors.") -->
(このような「呼び出し可能な」オブジェクトは「ファンクタ」と呼ばれることもあります)

#For example, you can define a type that stores the coefficients of a polynomial, but behaves like a function evaluating the polynomial:
たとえば、多項式の係数を格納する型を定義できますが、多項式を評価する関数のように動作します。

```jldoctest polynomial
julia> struct Polynomial{R}
           coeffs::Vector{R}
       end

julia> function (p::Polynomial)(x)
           v = p.coeffs[end]
           for i = (length(p.coeffs)-1):-1:1
               v = v*x + p.coeffs[i]
           end
           return v
       end
```

<!-- Notce that the function is specified by type instead of by name.  -->
関数が名前ではなく型によって指定されていることに注意してください。 
<!-- In he function body, `p` will refer to the object that was called.  -->
関数本体では、`p`は呼び出されたオブジェクトを参照します。
<!-- A `olynomial` can be used as follows: -->
`多項式`は次のように使用できます。

```jldoctest polynomial
julia> p = Polynomial([1,10,100])
Polynomial{Int64}([1, 10, 100])

julia> p(3)
931
```

#This mechanism is also the key to how type constructors and closures (inner functions that refer to their surrounding environment) work in Julia, discussed [later in the manual](@ref constructors-and-conversion).
このメカニズムは、型のコンストラクタとクロージャ(周囲の環境を参照する内部関数)がJuliaでどのように機能するかの鍵でもあります(後述の@refコンストラクタと変換)。

<!-- # Epty generic functions -->
##空の汎用関数

<!-- Occsionally it is useful to introduce a generic function without yet adding methods.  -->
時には、まだメソッドを追加しないでジェネリック関数を導入すると便利です。
<!-- Thi can be used to separate interface definitions from implementations.  -->
これは、インタフェース定義を実装から分離するために使用できます。
<!-- It ight also be done for the purpose of documentation or code readability.  -->
ドキュメンテーションやコードの読みやすさのために行うこともできます。
#The syntax for this is an empty `function` block without a tuple of arguments:
これの構文は空の `function`ブロックで、引数のタプルはありません：

```julia
function emptyfunc
end
```

<!-- ## [ethod design and the avoidance of ambiguities](@id man-method-design-ambiguities) -->
## [方法の設計とあいまいさの回避](@ id man-method-design-ambiguities)

#Julia's method polymorphism is one of its most powerful features, yet exploiting this power can pose design challenges.  
Juliaのメソッド、ポリモルフィズムは最も強力な機能の1つですが、この力を利用することで設計上の課題が生じる可能性があります。
<!-- In articular, in more complex method hierarchies it is not uncommon for [ambiguities](@ref man-ambiguities) to arise. -->
特に、より複雑なメソッド階層では、[曖昧さ](@ref man-ambiguities)が発生することは珍しくありません。

<!-- Aboe, it was pointed out that one can resolve ambiguities like -->
先に、あいまいさを解決することができるという説明をしました。

```julia
f(x, y::Int) = 1
f(x::Int, y) = 2
```

<!-- by efining a method -->
メソッドを定義することによって

```julia
f(x::Int, y::Int) = 3
```

<!-- Thi is often the right strategy; however, there are circumstances where following this advice blindly can be counterproductive.  -->
これはしばしば正しい戦略です。 しかし、このアドバイスを盲目的に追跡することは、非生産的になる可能性があります。
<!-- In articular, the more methods a generic function has, the more possibilities there are for ambiguities.  -->
特に、ジェネリック関数のメソッドが多くなればなるほど、あいまいさの可能性が増します。
<!-- Whe your method hierarchies get more complicated than this simple example, it can be worth your while to think carefully about alternative strategies. -->
メソッドの階層がこの単純な例より複雑になった場合、代替戦略について注意深く検討する価値があります。

<!-- Belw we discuss particular challenges and some alternative ways to resolve such issues. -->
以下では、特定の課題と、そのような問題を解決するための代替方法について説明します。

<!-- ### uple and NTuple arguments -->
###タプルとNTupleの引数

<!-- `Tule` (and `NTuple`) arguments present special challenges. For example, -->
`Tuple`(と`NTuple`)の引数には特別な問題があります。 例えば、

```julia
f(x::NTuple{N,Int}) where {N} = 1
f(x::NTuple{N,Float64}) where {N} = 2
```

#are ambiguous because of the possibility that `N == 0`: there are no elements to determine whether the `Int` or `Float64` variant should be called. 
これはあいまいです、なぜなら`N == 0` の可能性があるからで： その場合`Int` か `Float64`変種のどちらを呼び出すべきかを決定する要素がありません。
#To resolve the ambiguity, one approach is define a method for the empty tuple:
あいまいさを解決するには、空のタプルのメソッドを定義する方法があります。

```julia
f(x::Tuple{}) = 3
```

#Alternatively, for all methods but one you can insist that there is at least one element in the tuple:
あるいは、1つ以外のすべてのメソッドに対して、タプルに少なくとも1つの要素があると主張することができます。

```julia
f(x::NTuple{N,Int}) where {N} = 1           # this is the fallback
f(x::Tuple{Float64, Vararg{Float64}}) = 2   # this requires at least one Float64
```

<!-- ## Orthogonalize your design](@id man-methods-orthogonalize) -->
### [あなたのデザインの直交化](@ id man-methods-orthogonalize)


<!-- Whe you might be tempted to dispatch on two or more arguments, consider whether a "wrapper" function might make for a simpler design.  -->
2つ以上の引数にディスパッチしたい場合は、ラッパー関数を使って簡単な設計ができるかどうかを検討してください。
<!-- Forexample, instead of writing multiple variants: -->
たとえば、複数のバリアントを書くのではなく、

```julia
f(x::A, y::A) = ...
f(x::A, y::B) = ...
f(x::B, y::A) = ...
f(x::B, y::B) = ...
```

<!-- youmight consider defining -->
定義することを考えるかもしれません。

```julia
f(x::A, y::A) = ...
f(x, y) = f(g(x), g(y))
```

<!-- whee `g` converts the argument to type `A`.  -->
`g`は引数を` A`型に変換します。
#This is a very specific example of the more general principle of [orthogonal design](https://en.wikipedia.org/wiki/Orthogonality_(programming)), in which separate concepts are assigned to separate methods. 
これは、[直交設計](https://en.wikipedia.org/wiki/Orthogonality_(プログラミング))のより一般的な原則の非常に具体的な例です。別々の概念が別々の方法に割り当てられています。
#Here, `g` will most likely need a fallback definition
ここで、 `g`はフォールバック定義が必要になるでしょう

```julia
g(x::A) = x
```

<!-- A rlated strategy exploits `promote` to bring `x` and `y` to a common type: -->
関連する戦略は、`x`と`y`を共通の型にするために `promote`を利用します：

```julia
f(x::T, y::T) where {T} = ...
f(x, y) = f(promote(x, y)...)
```

#One risk with this design is the possibility that if there is no suitable promotion method converting `x` and `y` to the same type, the second method will recurse on itself infinitely and trigger a stack overflow. 
この設計の1つのリスクは、`x`と`y`を同じ型に変換する適切な宣伝方法がない場合、2番目の方法は無限に繰り返してスタックオーバーフローを引き起こす可能性があります。
#The non-exported function `Base.promote_noncircular` can be used as an alternative; when promotion fails it will still throw an error, but one that fails faster with a more specific error message.
エクスポートされていない関数 `Base.promote_noncircular`を代わりに使うことができます。 プロモーションに失敗した場合でもエラーは発生しますが、より具体的なエラーメッセージが表示されるとエラーが発生します。

<!-- ## ispatch on one argument at a time -->
###一度に1つの引数にディスパッチする

#If you need to dispatch on multiple arguments, and there are many fallbacks with too many combinations to make it practical to define all possible variants, then consider introducing a "name cascade" where (for example) you dispatch on the first argument and then call an internal method:
複数の引数にディスパッチする必要があり、組み合わせが多すぎる可能性があるため、可能なすべてのバリアントを定義するのが現実的となるようなフォールバックが多い場合は、最初の引数にディスパッチして(たとえば) 内部的な方法：

```julia
f(x::A, y) = _fA(x, y)
f(x::B, y) = _fB(x, y)
```

#Then the internal methods `_fA` and `_fB` can dispatch on `y` without concern about ambiguities with each other with respect to `x`.
すると内部メソッド `_fA`と` _fB`は `x`に関して互いにあいまいさを気にせずに` y`にディスパッチすることができます。

#Be aware that this strategy has at least one major disadvantage: in many cases, it is not possible for users to further customize the behavior of `f` by defining further specializations of your exported function `f`. 
この戦略には、少なくとも1つの大きな欠点があることに注意してください。多くの場合、エクスポートされた関数fのさらなる特殊化を定義することによって、ユーザーが `f`の動作をさらにカスタマイズすることはできません。
#Instead, they have to define specializations for your internal methods `_fA` and `_fB`, and this blurs the lines between exported and internal methods.
代わりに、内部メソッド `_fA`と` _fB`の特殊化を定義しなければなりません。これは、エクスポートされたメソッドと内部メソッドの間の行をぼかします。

<!-- ## bstract containers and element types -->
###抽象コンテナと要素型

<!-- Whee possible, try to avoid defining methods that dispatch on specific element types of abstract containers.  -->
可能であれば、抽象コンテナの特定の要素型にディスパッチするメソッドを定義しないようにしてください。
<!-- Forexample, -->
例えば、

```julia
-(A::AbstractArray{T}, b::Date) where {T<:Date}
```

<!-- genrates ambiguities for anyone who defines a method -->
メソッドを定義する人のためにあいまいさを生成する

```julia
-(A::MyArrayType{T}, b::T) where {T}
```

#The best approach is to avoid defining *either* of these methods: instead, rely on a generic method `-(A::AbstractArray, b)` and make sure this method is implemented with generic calls (like `similar` and `-`) that do the right thing for each container type and element type *separately*. 
最良の方法は、これらのメソッドの*どちらかを定義するのを避けることです：代わりに汎用メソッド ` - (A :: AbstractArray、b)`に依存し、このメソッドがジェネリックコール( `similar`や` - `)は、それぞれのコンテナタイプと要素タイプ*別々の*に対して正しいことをします。
#This is just a more complex variant of the advice to [orthogonalize](@ref man-methods-orthogonalize) your methods.
これはあなたのメソッドを[直交化](@ref man-methods-orthogonalize)するアドバイスのちょっと複雑な変形です。

#When this approach is not possible, it may be worth starting a discussion with other developers about resolving the ambiguity; just because one method was defined first does not necessarily mean that it can't be modified or eliminated.  
このアプローチが不可能な場合は、あいまいさを解決することについて他の開発者と議論を開始する価値があります。 1つのメソッドが最初に定義されているからといって、必ずしもそのメソッドを変更または削除することはできません。
#As a last resort, one developer can define the "band-aid" method
最後の手段として、1人の開発者が「バンドエイド」メソッドを定義できます

```julia
-(A::MyArrayType{T}, b::Date) where {T<:Date} = ...
```

#that resolves the ambiguity by brute force.
ブルートフォースによるあいまいさを解消する。

<!-- ## omplex method "cascades" with default arguments -->
###複雑なメソッド "カスケード"とデフォルトの引数

#If you are defining a method "cascade" that supplies defaults, be careful about dropping any arguments that correspond to potential defaults. 
デフォルトを提供するメソッド "カスケード"を定義する場合、潜在的なデフォルトに対応する引数を削除することに注意してください。
#For example, suppose you're writing a digital filtering algorithm and you have a method that handles the edges of the signal by applying padding:
たとえば、デジタルフィルタリングアルゴリズムを作成していて、パディングを適用して信号のエッジを処理する方法があるとします。

```julia
function myfilter(A, kernel, ::Replicate)
    Apadded = replicate_edges(A, size(kernel))
    myfilter(Apadded, kernel)  # now perform the "real" computation
end
```

<!-- Thi will run afoul of a method that supplies default padding: -->
これは、デフォルトのパディングを提供するメソッドと違反します：

```julia
myfilter(A, kernel) = myfilter(A, kernel, Replicate()) # replicate the edge by default
```

<!-- Togther, these two methods generate an infinite recursion with `A` constantly growing bigger. -->
一緒に、これらの2つの方法は、`A`が絶えず大きくなる無限再帰を生成します。

<!-- Thebetter design would be to define your call hierarchy like this: -->
より良い設計は、次のように呼び出し階層を定義することです。

```julia
struct NoPad end  # indicate that no padding is desired, or that it's already applied

myfilter(A, kernel) = myfilter(A, kernel, Replicate())  # default boundary conditions

function myfilter(A, kernel, ::Replicate)
    Apadded = replicate_edges(A, size(kernel))
    myfilter(Apadded, kernel, NoPad())  # indicate the new boundary conditions
end

# other padding methods go here

function myfilter(A, kernel, ::NoPad)
    # Here's the "real" implementation of the core computation
end
```

<!-- `Noad` is supplied in the same argument position as any other kind of padding, so it keeps the dispatch hierarchy well organized and with reduced likelihood of ambiguities.  -->
`NoPad`は、他の種類のパディングと同じ引数位置に指定されているため、ディスパッチ階層を整理し、あいまいさを減らします。
<!-- Morover, it extends the "public" `myfilter` interface: a user who wants to control the padding explicitly can call the `NoPad` variant directly. -->
さらに、"パブリック"な`myfilter`インターフェースを拡張します。パディングを明示的に制御したいユーザーは、`NoPad`バリアントを直接呼び出すことができます。

[^Clarke61]: Arthur C. Clarke, *Profiles of the Future* (1961): Clarke's Third Law.
