# Style Guide
#スタイルガイド

#The following sections explain a few aspects of idiomatic Julia coding style. 
次のセクションでは、慣用的なJuliaコーディングスタイルのいくつかの側面について説明します。
#None of these rules are absolute; they are only suggestions to help familiarize you with the language and to help you choose among alternative designs.
これらのルールのどれも絶対的なものではありません。 言語に慣れ親しんで、代替デザインの中から選択するのに役立つヒントです。

## Write functions, not just scripts
##スクリプトだけでなく関数を書く

#Writing code as a series of steps at the top level is a quick way to get started solving a problem, but you should try to divide a program into functions as soon as possible. 
最上位レベルの一連のステップとしてコードを書くことは、問題を解決するための素早い方法ですが、プログラムをできるだけ早く関数に分割してください。
#Functions are more reusable and testable, and clarify what steps are being done and what their inputs and outputs are.
関数は再利用可能であり、テスト可能であり、実行されているステップとその入力と出力が何であるかを明確にします。
#Furthermore, code inside functions tends to run much faster than top level code, due to how Julia's compiler works.
さらに、関数内のコードは、Juliaのコンパイラの仕組みのために、最上位のコードよりもはるかに高速に動作する傾向があります。

#It is also worth emphasizing that functions should take arguments, instead of operating directly on global variables (aside from constants like [`pi`](@ref)).
関数はグローバル変数（[`pi`]（@ ref）のような定数を除いて）で直接操作するのではなく、引数を取るべきであることを強調する価値もあります。

## Avoid writing overly-specific types
##過度に特殊な型を書くのを避ける

#Code should be as generic as possible.
コードは可能な限り汎用的でなければなりません。
#Instead of writing:
書くのではなく、

```julia
convert(Complex{Float64}, x)
```

#it's better to use available generic functions:
利用可能な汎用関数を使用する方が良いでしょう：

```julia
complex(float(x))
```

#The second version will convert `x` to an appropriate type, instead of always the same type.
2つ目のバージョンは `x`を常に同じ型ではなく適切な型に変換します。

#This style point is especially relevant to function arguments. For example, don't declare an argument to be of type `Int` or [`Int32`](@ref) if it really could be any integer, expressed with the abstract type [`Integer`](@ref).
このスタイルポイントは、特に関数の引数に関係します。 たとえば、抽象型[`Integer`]（@ ref）で表現された引数が本当に整数であれば、型は` Int`または[`Int32`]（@ ref）であると宣言しないでください。
#In fact, in many cases you can omit the argument type altogether, unless it is needed to disambiguate from other method definitions, since a [`MethodError`](@ref) will be thrown anyway if a type is passed that does not support any of the requisite operations. 
実際には、他のメソッド定義との曖昧さを解消する必要がある場合を除いて、引数型を完全に省略することができます。型を渡すと[[MethodError`]（@ ref）がスローされます 必要な操作の
#(This is known as [duck typing](https://en.wikipedia.org/wiki/Duck_typing).)
（これは[ダックタイピング]（https://en.wikipedia.org/wiki/Duck_typing）として知られています）。

#For example, consider the following definitions of a function `addone` that returns one plus its argument:
例えば、引数に1を加えたものを返す `addone`関数の定義を考えてみましょう：

```julia
addone(x::Int) = x + 1                 # works only for Int
addone(x::Integer) = x + oneunit(x)    # any integer type
addone(x::Number) = x + oneunit(x)     # any numeric type
addone(x) = x + oneunit(x)             # any type supporting + and oneunit
```

#The last definition of `addone` handles any type supporting [`oneunit`](@ref) (which returns 1 in the same type as `x`, which avoids unwanted type promotion) and the [`+`](@ref) function with those arguments. 
`addun`の最後の定義は、` `oneunit` '（@ref）（` x`と同じ型で1を返し、不要な型宣伝を避ける）と[`+`] これらの引数を使用して機能します。
#The key thing to realize is that there is *no performance penalty* to defining *only* the general `addone(x) = x + oneunit(x)`, because Julia will automatically compile specialized versions as needed.
実現する重要なことは、Juliaが必要に応じて自動的に特殊なバージョンをコンパイルするため、*一般的な `addone（x）= x + oneunit（x）`のみを定義する*パフォーマンスペナルティ*がないことです。
#For example, the first time you call `addone(12)`, Julia will automatically compile a specialized `addone` function for `x::Int` arguments, with the call to `oneunit` replaced by its inlined value `1`. 
たとえば、初めて `addone（12）`を呼び出すと、Juliaは `x :: Int`引数に対して特殊な` addone`関数を自動的にコンパイルします。 `oneunit`の呼び出しはそのインライン値` 1`で置き換えられます。
#Therefore, the first three definitions of `addone` above are completely redundant with the fourth definition.
したがって、上記の「addone」の最初の3つの定義は、第4の定義と完全に重複しています。

## Handle excess argument diversity in the caller
##呼び出し元の過剰な引数の多様性を処理する

Instead of:

```julia
function foo(x, y)
    x = Int(x); y = Int(y)
    ...
end
foo(x, y)
```

use:

```julia
function foo(x::Int, y::Int)
    ...
end
foo(Int(x), Int(y))
```

#This is better style because `foo` does not really accept numbers of all types; it really needs `Int` s.
`foo`は実際にすべての型の数を受け入れないので、これはより良いスタイルです。 本当に `Int`が必要です。
#One issue here is that if a function inherently requires integers, it might be better to force the caller to decide how non-integers should be converted (e.g. floor or ceiling). 
ここでの問題の1つは、関数が整数を本質的に必要とする場合、呼び出し元に非整数の変換方法（床や天井など）を強制的に決定させる方がよい場合があるということです。
#Another issue is that declaring more specific types leaves more "space" for future method definitions.
もう1つの問題は、より具体的な型を宣言すると、将来のメソッド定義のための "スペース"が増えることです。

## Append `!` to names of functions that modify their arguments
##引数を変更する関数の名前に `！`を追加する

#Instead of:
の代わりに：

```julia
function double(a::AbstractArray{<:Number})
    for i = 1:endof(a)
        a[i] *= 2
    end
    return a
end
```

use:

```julia
function double!(a::AbstractArray{<:Number})
    for i = 1:endof(a)
        a[i] *= 2
    end
    return a
end
```

#The Julia standard library uses this convention throughout and contains examples of functions with both copying and modifying forms (e.g., [`sort()`](@ref) and [`sort!()`](@ref)), and others which are just modifying (e.g., [`push!()`](@ref), [`pop!()`](@ref), [`splice!()`](@ref)).  
Juliaの標準ライブラリでは、この形式を全面的に使用し、コピーと変更の両方の形式（[`sort（）`]（@ ref）と[`sort！（）`]（@ ref）など） （@ ref）、[`pop！（）`]（@ ref）、[`splice！（）`]（@ ref））を変更するだけです。
#It is typical for such functions to also return the modified array for convenience.
このような関数は、便宜上、変更された配列も返すのが一般的です。

## Avoid strange type `Union`s
##奇妙なタイプの `Union`sを避ける

#Types such as `Union{Function,AbstractString}` are often a sign that some design could be cleaner.
`Union {Function、AbstractString} 'のような型は、しばしば、ある種のデザインがより洗練されたものになることを示す記号です。

## Avoid type Unions in fields
##フィールド内の型共用体を避ける

#When creating a type such as:
次のような型を作成するとき：

```julia
mutable struct MyType
    ...
    x::Union{Void,T}
end
```

#ask whether the option for `x` to be `nothing` (of type `Void`) is really necessary. 
`x`が` nothing`（ `Void`型）のオプションが本当に必要かどうかを尋ねます。
#Here are some alternatives to consider:
考慮すべきいくつかの選択肢は次のとおりです。

   * 
   #Find a safe default value to initialize `x` with
   `x`を初期化するための安全なデフォルト値を見つける
   * 
   #Introduce another type that lacks `x`
   `x`がない別のタイプを導入する
   * 
   #If there are many fields like `x`, store them in a dictionary
   `x`のようなフィールドがたくさんある場合は、それらを辞書に格納します
   * 
   #Determine whether there is a simple rule for when `x` is `nothing`. 
   `x`が` nothing`の場合の単純なルールがあるかどうかを判断します。
   #For example, often the field will start as `nothing` but get initialized at some well-defined point. 
   例えば、しばしばフィールドは「何もない」として開始するが、ある明確な点で初期化される。
   #In that case, consider leaving it undefined at first.
   その場合、最初は未定義のままにしておくことを検討してください。
   * 
   #If `x` really needs to hold no value at some times, define it as `::Nullable{T}` instead, as this guarantees type-stability in the code accessing this field (see [Nullable types](@ref man-nullable-types)).
   もし `x`が実際に値を保持していなければならない場合は、このフィールドにアクセスするコードの型安定性を保証するために、` :: Nullable {T} `として定義してください（[Nullable types] nullable-types））。

## Avoid elaborate container types
##精巧なコンテナの種類を避ける

#It is usually not much help to construct arrays like the following:
通常、次のような配列を作成するのはあまり役に立ちません。

```julia
a = Array{Union{Int,AbstractString,Tuple,Array}}(n)
```

#In this case `Array{Any}(n)` is better. 
この場合、 `Array {Any}（n）`がより良いです。
#It is also more helpful to the compiler to annotate specific uses (e.g. `a[i]::Int`) than to try to pack many alternatives into one type.
コンパイラは、多くの選択肢を1つのタイプにパックしようとするよりも、特定の用途に注釈を付ける方が便利です（たとえば `a [i] :: Int`など）。

## Use naming conventions consistent with Julia's `base/`
## Juliaの `base /`と同じ命名規則を使う

  * 
  # modules and type names use capitalization and camel case: `module SparseArrays`, `struct UnitRange`.
   モジュールと型名は、大文字と小文字の大文字小文字を使用します： `module SparseArrays`、` struct UnitRange`。
   *
   *

  * 
  #functions are lowercase ([`maximum()`](@ref), [`convert()`](@ref)) and, when readable, with multiple words squashed together ([`isequal()`](@ref), [`haskey()`](@ref)). When necessary, use underscores as word separators.
   （@ ref）、[`convert（）`]（@ ref））、複数の単語が一緒に押しつぶされる（[`isequal（）`]（@ ref） 、[`haskey（）`]（@ ref））。 必要に応じて、単語区切り記号としてアンダースコアを使用します。
  #Underscores are also used to indicate a combination of concepts ([`remotecall_fetch()`](@ref) as a more efficient implementation of `fetch(remotecall(...))`) or as modifiers ([`sum_kbn()`](@ref)).
   アンダースコアは、fetch（remotecall（...））のより効率的な実装としてのコンセプト（[`remotecall_fetch（）`]（@ref）の組み合わせを示すため、または修飾子（[`sum_kbn（） ]（@ ref））。
  * 
  #conciseness is valued, but avoid abbreviation ([`indexin()`](@ref) rather than `indxin()`) as it becomes difficult to remember whether and how particular words are abbreviated.
   簡潔さは評価されますが、特定の単語が省略されているかどうか、どのように省略されているかを覚えにくくなるので、省略形（ `` indexin（） ``（@ ref）ではなく `` indxin（）

#If a function name requires multiple words, consider whether it might represent more than one concept and might be better split into pieces.
関数名が複数の単語を必要とする場合、複数の単語を必要とするかどうかを検討し、複数の単語に分割する方がよいかどうかを検討します。

## Don't overuse try-catch
## try-catchを多用しないでください

#It is better to avoid errors than to rely on catching them.
それを捉えることに頼るよりも、エラーを避ける方が良いです。

## Don't parenthesize conditions
##条件をカッコで囲まない

#Julia doesn't require parens around conditions in `if` and `while`. Write:
Juliaは `if`と` while`の条件の周りに括弧を必要としません。 書きます：

```julia
if a == b
```

instead of:

```julia
if (a == b)
```

## Don't overuse `...`
## `...`をあまり使わないでください。

#Splicing function arguments can be addictive. 
スプライス関数の引数は中毒性があります。
#Instead of `[a..., b...]`, use simply `[a; b]`, which already concatenates arrays. 
`[a ...、b ...]`の代わりに `[a; b] `はすでに配列を連結しています。
#[`collect(a)`](@ref) is better than `[a...]`, but since `a` is already iterable it is often even better to leave it alone, and not convert it to an array.
[`collect（a）`]（@ ref）は `[a ...]`より優れていますが、 `a`は既にiterableなので、それをそのままにして配列に変換しない方が良いことがよくあります。

## Don't use unnecessary static parameters
##不必要な静的パラメータを使用しないでください

#A function signature:
関数シグネチャ：

```julia
foo(x::T) where {T<:Real} = ...
```

#should be written as:
次のように記述する必要があります。

```julia
foo(x::Real) = ...
```

#instead, especially if `T` is not used in the function body. 
代わりに、 `T 'が関数本体で使われていない場合は特にそうです。
#Even if `T` is used, it can be replaced with [`typeof(x)`](@ref) if convenient. 
`T 'を使用しても便利な場合は[` typeof（x） `]（@ ref）に置き換えることができます。
#There is no performance difference. 
パフォーマンスの違いはありません。
#Note that this is not a general caution against static parameters, just against uses where they are not needed.
これは静的パラメータに対する一般的な警告ではなく、必要でない場所での使用に対してのみ注意することに注意してください。

#Note also that container types, specifically may need type parameters in function calls. 
コンテナ型は、特に関数呼び出しで型パラメータが必要な場合があることにも注意してください。
#See the FAQ [Avoid fields with abstract containers](@ref) for more information.
詳細は、FAQ [抽象コンテナのフィールドを避ける]（@ ref）を参照してください。

### Avoid confusion about whether something is an instance or a type
##何かがインスタンスか型かどうか混同しないようにする

#Sets of definitions like the following are confusing:
以下のような定義は混乱します。

```julia
foo(::Type{MyType}) = ...
foo(::MyType) = foo(MyType)
```

#Decide whether the concept in question will be written as `MyType` or `MyType()`, and stick to it.
問題のコンセプトが `MyType`か` MyType（） `として記述されるかどうかを決め、それに固執する。

#The preferred style is to use instances by default, and only add methods involving `Type{MyType}` later if they become necessary to solve some problem.
推奨されるスタイルは、デフォルトでインスタンスを使用し、後で問題を解決するために必要になった場合に、 `Type {MyType} 'を含むメソッドを追加することです。

#If a type is effectively an enumeration, it should be defined as a single (ideally immutable struct or primitive) type, with the enumeration values being instances of it. 
型が実質的に列挙型である場合、その列挙型の値をインスタンスとする単一の（理想的には変更できない構造体またはプリミティブの）型として定義する必要があります。
#Constructors and conversions can check whether values are valid. This design is preferred over making the enumeration an abstract type, with the "values" as subtypes.
コンストラクターとコンバージョンは、値が有効かどうかをチェックできます。 この設計は、列挙型を抽象型にするよりも、「値」をサブタイプとして使用する方が望ましいです。

## Don't overuse macros
##マクロを酷使しないでください

#Be aware of when a macro could really be a function instead.
マクロが本当に代わりに関数になる可能性があることに注意してください。

#Calling [`eval()`](@ref) inside a macro is a particularly dangerous warning sign; it means the macro will only work when called at the top level. 
マクロ内で[`eval（）`]（@ ref）を呼び出すことは、特に危険な警告標識です。 これは、マクロがトップレベルで呼び出されたときにのみ機能することを意味します。
#If such a macro is written as a function instead, it will naturally have access to the run-time values it needs.
そのようなマクロが代わりに関数として記述されている場合、必然的に必要な実行時の値にアクセスすることができます。

## Don't expose unsafe operations at the interface level
##安全でない操作をインターフェースレベルで公開しない


#If you have a type that uses a native pointer:
ネイティブポインタを使用する型がある場合：

```julia
mutable struct NativeType
    p::Ptr{UInt8}
    ...
end
```

#don't write definitions like the following:
次のような定義は書き込まないでください。

```julia
getindex(x::NativeType, i) = unsafe_load(x.p, i)
```

#The problem is that users of this type can write `x[i]` without realizing that the operation is unsafe, and then be susceptible to memory bugs.
問題は、このタイプのユーザーは、操作が危険であることを認識せずに `x [i]`と書いて、メモリのバグの影響を受けやすいということです。

#Such a function should either check the operation to ensure it is safe, or have `unsafe` somewhere in its name to alert callers.
そのような関数は、安全であることを確認するために操作をチェックするか、呼び出し側に警告するためにその名前のどこかに `unsafe`を置かなければなりません。

## Don't overload methods of base container types
##基本コンテナ型のメソッドをオーバーロードしないでください

#It is possible to write definitions like the following:
次のような定義を書くことができます：

```julia
show(io::IO, v::Vector{MyType}) = ...
```

#This would provide custom showing of vectors with a specific new element type. 
これは、特定の新しい要素タイプを持つベクトルのカスタム表示を提供します。
#While tempting, this should be avoided. The trouble is that users will expect a well-known type like `Vector()` to behave in a certain way, and overly customizing its behavior can make it harder to work with.
誘惑している間、これは避けるべきです。 問題は、ユーザーが `Vector（）`のようなよく知られた型を特定の方法で動作させることを期待し、その振る舞いを過度にカスタマイズすると作業が難しくなることです。

## Avoid type piracy
##海賊行為を避ける

#"Type piracy" refers to the practice of extending or redefining methods in Base or other packages on types that you have not defined. 
"タイプ違法コピー"とは、定義していないタイプのBaseパッケージや他のパッケージのメソッドを拡張または再定義する方法を指します。
#In some cases, you can get away with type piracy with little ill effect. 
場合によっては、悪影響をほとんど及ぼすことなくタイプ違法コピーを手に入れることができます。
#In extreme cases, however, you can even crash Julia (e.g. if your method extension or redefinition causes invalid input to be passed to a `ccall`). 
しかし、極端な場合にはJuliaをクラッシュさえすることもできます（例えば、メソッドの拡張や再定義によって無効な入力が `ccall`に渡された場合など）。
#Type piracy can complicate reasoning about code, and may introduce incompatibilities that are hard to predict and diagnose.
タイプの海賊行為は、コードに関する推論を複雑にし、予測や診断が困難な非互換性をもたらす可能性があります。

#As an example, suppose you wanted to define multiplication on symbols in a module:
たとえば、モジュール内のシンボルに乗算を定義するとします。

```julia
module A
import Base.*
*(x::Symbol, y::Symbol) = Symbol(x,y)
end
```

#The problem is that now any other module that uses `Base.*` will also see this definition. 
問題は、 `Base。* 'を使用する他のモジュールもこの定義を見ることになります。
#Since `Symbol` is defined in Base and is used by other modules, this can change the behavior of unrelated code unexpectedly. 
`Symbol`はBaseで定義されており、他のモジュールで使われているので、無関係なコードの動作が予期せず変更される可能性があります。
#There are several alternatives here, including using a different function name, or wrapping the `Symbol`s in another type that you define.
別の関数名を使うことや、定義した別の型で `Symbol`をラップすることなど、いくつかの選択肢があります。

#Sometimes, coupled packages may engage in type piracy to separate features from definitions, especially when the packages were designed by collaborating authors, and when the definitions are reusable. 
時には、カップルされたパッケージは、特にパッケージが共同作者によって設計されたとき、および定義が再利用可能なときに、フィーチャを定義から分離するためにタイプ違法コピーに関与することがあります。
#For example, one package might provide some types useful for working with colors; another package could define methods for those types that enable conversions between color spaces. 
たとえば、あるパッケージでは、色の操作に便利ないくつかの型が用意されています。 別のパッケージでは、カラースペース間の変換を可能にするタイプのメソッドを定義できます。
#Another example might be a package that acts as a thin wrapper for some C code, which another package might then pirate to implement a higher-level, Julia-friendly API.
もう1つの例は、Cコードのための薄いラッパーとして機能するパッケージで、別のパッケージがより高度なジュリアフレンドリーなAPIを実装するために海賊行為をする可能性があります。

## Be careful with type equality
##型の等価性に注意してください

#You generally want to use [`isa()`](@ref) and `<:` ([`issubtype()`](@ref)) for testing types, not `==`.
`==`ではなく、 `` isa（） ``（@ ref）と `<：`（[`issubtype（）`]（@ ref） 
#Checking types for exact equality typically only makes sense when comparing to a known concrete type (e.g. `T == Float64`), or if you *really, really* know what you're doing.
タイプを正確に一致させるかどうかを確認するのは、通常、既知の具体的なタイプ（たとえば `T == Float64`）と比較する場合、または実際に行っていることを本当に知っている場合にのみ意味があります。

## Do not write `x->f(x)`
## `x-> f（x）`を書かないでください。

#Since higher-order functions are often called with anonymous functions, it is easy to conclude that this is desirable or even necessary. 
高階関数は匿名関数で呼び出されることが多いため、これが望ましく、必要でさえあると結論付けるのは簡単です。
#But any function can be passed directly, without being "wrapped" in an anonymous function.
しかし、任意の関数は、無名関数で「ラップされる」ことなく直接渡すことができます。 
#Instead of writing `map(x->f(x), a)`, write [`map(f, a)`](@ref).
`map（x-> f（x）、a）`を書くのではなく、[`map（f、a）`]（@ ref）と書いてください。

## Avoid using floats for numeric literals in generic code when possible
##できるだけジェネリックコードの数値リテラルに浮動小数点数を使用しないでください

#If you write generic code which handles numbers, and which can be expected to run with many different numeric type arguments, try using literals of a numeric type that will affect the arguments as little as possible through promotion.
数字を扱うジェネリックコードを記述し、多くの異なる数値型引数で実行することが期待できる場合は、宣言によってできるだけ引数に影響を与えない数値型のリテラルを試してみてください。

#For example,
例えば、

```jldoctest
julia> f(x) = 2.0 * x
f (generic function with 1 method)

julia> f(1//2)
1.0

julia> f(1/2)
1.0

julia> f(1)
2.0
```

while

```jldoctest
julia> g(x) = 2 * x
g (generic function with 1 method)

julia> g(1//2)
1//1

julia> g(1/2)
1.0

julia> g(1)
2
```

#As you can see, the second version, where we used an `Int` literal, preserved the type of the input argument, while the first didn't. 
ご覧のように、 `Int`リテラルを使用した2番目のバージョンは入力引数の型を保持していましたが、最初の引数は保持しませんでした。
#This is because e.g. `promote_type(Int, Float64) == Float64`, and promotion happens with the multiplication. Similarly, [`Rational`](@ref) literals are less type disruptive than [`Float64`](@ref) literals, but more disruptive than `Int`s:
これは、例えば `promote_type（Int、Float64）== Float64`となり、宣伝は乗算で行われます。 同様に、[`Rational`]（@ ref）のリテラルは[` Float64`]（@ ref）リテラルよりも型破壊的ではありませんが、 `Int`よりも破壊的です：

```jldoctest
julia> h(x) = 2//1 * x
h (generic function with 1 method)

julia> h(1//2)
1//1

julia> h(1/2)
1.0

julia> h(1)
2//1
```

#Thus, use `Int` literals when possible, with `Rational{Int}` for literal non-integer numbers, in order to make it easier to use your code.
したがって、あなたのコードを使いやすくするために、可能な場合には `Int`リテラルを使用し、実数の非整数の場合は` Rational {Int} 'を使用してください。
