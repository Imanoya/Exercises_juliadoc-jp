# More about types

<!-- EN -->
If you've used Julia for a while, you understand the fundamental role that types play.
> Juliaをしばらく使用したことがある場合、タイプが果たす基本的な役割を理解しています。
<!-- EN -->
Here we try to get under the hood, focusing particularly on [Parametric Types](@ref).
> ここでは、特に[パラメトリックタイプ]（@ ref）に焦点を当てて、フードの中に入っていきます。

## Types and sets (and `Any` and `Union{}`/`Bottom`)

<!-- EN -->
It's perhaps easiest to conceive of Julia's type system in terms of sets.
> Juliaのタイプシステムをセットで考えるのはおそらく最も簡単です。
<!-- EN -->
While programs manipulate individual values, a type refers to a set of values.
> プログラムは個々の値を操作しますが、型は一連の値を参照します。
<!-- EN -->
This is not the same thing as a collection;
> これはコレクションと同じではありません。
<!-- EN -->
for example a `Set` of values is itself a single `Set` value.
> 例えば、値の `Set`それ自身は単一の` Set`値です。
<!-- EN -->
Rather, a type describes a set of *possible* values, expressing uncertainty about which value we have.
> むしろ、タイプは*可能な*値のセットを記述し、我々が持つ価値についての不確実性を表しています。

<!-- EN -->
A *concrete* type `T` describes the set of values whose direct tag, as returned by the `typeof` function, is `T`.
> *concrete*型 `T`は、` typeof`関数によって返された直接タグが `T`である値の集合を記述します。
<!-- EN -->
An *abstract* type describes some possibly-larger set of values.
> *abstract*型は、場合によってはより大きな値の集合を記述します。

<!-- EN -->
`Any` describes the entire universe of possible values. [`Integer`](@ref) is a subset of `Any` that includes `Int`, [`Int8`](@ref), and other concrete types.
<!-- EN -->
> `Any`は可能な値の宇宙全体を記述します。 [`Integer`]（@ref）は` Int`、[`Int8`]（@ref）などの具体的な型を含む` Any`のサブセットです。
<!-- EN -->
Internally, Julia also makes heavy use of another type known as `Bottom`, which can also be written as `Union{}`.
> 内部的には、Juliaは `Bottom`とも呼ばれる別の型を頻繁に使います。これは` Union {} `と書くこともできます。
<!-- EN -->
This corresponds to the empty set.
> 空のセットに相当します。

<!-- EN -->
Julia's types support the standard operations of set theory: you can ask whether `T1` is a "subset" (subtype) of `T2` with `T1 <: T2`.
> Juliaのタイプはset理論の標準的な操作をサポートしています： `T1`が` T1 <：T2`の `T2`の`サブセット `（サブタイプ）かどうかを調べることができます。
<!-- EN -->
Likewise, you intersect two types using `typeintersect`, take their union with `Union`, and compute a type that contains their union with `typejoin`:
> 同様に、 `typeintersect`を使って2つの型を交差させ、` Union`でそれらを結合し、 `typejoin`でその共用体を含む型を計算します：

```jldoctest
julia> typeintersect(Int, Float64)
Union{}

julia> Union{Int, Float64}
Union{Float64, Int64}

julia> typejoin(Int, Float64)
Real

julia> typeintersect(Signed, Union{UInt8, Int8})
Int8

julia> Union{Signed, Union{UInt8, Int8}}
Union{UInt8, Signed}

julia> typejoin(Signed, Union{UInt8, Int8})
Integer

julia> typeintersect(Tuple{Integer,Float64}, Tuple{Int,Real})
Tuple{Int64,Float64}

julia> Union{Tuple{Integer,Float64}, Tuple{Int,Real}}
Union{Tuple{Int64,Real}, Tuple{Integer,Float64}}

julia> typejoin(Tuple{Integer,Float64}, Tuple{Int,Real})
Tuple{Integer,Real}
```

<!-- EN -->
While these operations may seem abstract, they lie at the heart of Julia.
> これらの操作は抽象的に見えるかもしれませんが、Juliaの中心に位置しています。
<!-- EN -->
For example, method dispatch is implemented by stepping through the items in a method list until reaching one for which the type of the argument tuple is a subtype of the method signature.
> 例えば、メソッドディスパッチは、引数タプルの型がメソッドシグネチャのサブタイプであるものに達するまで、メソッドリスト内の項目をステップ実行することによって実装されます。
<!-- EN -->
For this algorithm to work, it's important that methods be sorted by their specificity, and that the search begins with the most specific methods.
> このアルゴリズムが機能するためには、メソッドがその特異性によってソートされ、検索が最も具体的なメソッドから始まることが重要です。
<!-- EN -->
Consequently, Julia also implements a partial order on types; this is achieved by functionality that is similar to `<:`, but with differences that will be discussed below.
> その結果、Juliaは型の部分的な順序も実装します。 これは `<：`と似ているが、以下で説明する相違点を持つ機能によって実現される。

## UnionAll types

<!-- EN -->
Julia's type system can also express an *iterated union* of types: a union of types over all values of some variable.
> Juliaの型システムは、*型の繰り返し反復*を表現することもできます：いくつかの変数のすべての値に対する型の和集合。
<!-- EN -->
This is needed to describe parametric types where the values of some parameters are not known.
> これは、いくつかのパラメータの値がわからないパラメトリック型を記述するために必要です。

<!-- EN -->
For example, :obj:`Array` has two parameters as in `Array{Int,2}`.
> 例えば、：obj： `Array`は` Array {Int、2} `のように二つのパラメータを持っています。
<!-- EN -->
If we did not know the element type, we could write `Array{T,2} where T`, which is the union of `Array{T,2}` for all values of `T`: `Union{Array{Int8,2}, Array{Int16,2}, ...}`.
> 要素の型が分からなければ、Tのすべての値に対して `Array {T、2} 'の和集合である` Array {T、2} T'を書くことができます：Union {Array {Int8 、2}、配列{Int16,2}、...} `である。

<!-- EN -->
Such a type is represented by a `UnionAll` object, which contains a variable (`T` in this example, of type `TypeVar`), and a wrapped type (`Array{T,2}` in this example).
<!-- EN -->
> そのような型は `UnionAll`オブジェクトで表され、変数（この例では` T`型、 `TypeVar`型）とラップ型（この例では` Array {T、2} `）を含みます。

<!-- EN -->
Consider the following methods:
> 次の方法を検討してください。

```julia
f1(A::Array) = 1
f2(A::Array{Int}) = 2
f3(A::Array{T}) where {T<:Any} = 3
f4(A::Array{Any}) = 4
```

<!-- EN -->
The signature of `f3` is a `UnionAll` type wrapping a tuple type.
> `f3`の署名はタプル型をラップする` UnionAll`型です。
<!-- EN -->
All but `f4` can be called with `a = [1,2]`; all but `f2` can be called with `b = Any[1,2]`.
> `f4`以外は` a = [1,2] `で呼び出すことができます。 `f2`を除くすべては` b = Any [1,2] `で呼び出すことができます。

<!-- EN -->
Let's look at these types a little more closely:
> これらのタイプをもう少し詳しく見てみましょう：

```jldoctest
julia> dump(Array)
UnionAll
  var: TypeVar
    name: Symbol T
    lb: Core.TypeofBottom Union{}
    ub: Any
  body: UnionAll
    var: TypeVar
      name: Symbol N
      lb: Core.TypeofBottom Union{}
      ub: Any
    body: Array{T,N} <: DenseArray{T,N}
```

<!-- EN -->
This indicates that `Array` actually names a `UnionAll` type.
> これは、 `Array`が実際に` UnionAll`型を指定していることを示しています。
<!-- EN -->
There is one `UnionAll` type for each parameter, nested.
> 入れ子にされた各パラメータに対して1つの `UnionAll`型があります。
<!-- EN -->
The syntax `Array{Int,2}` is equivalent to `Array{Int}{2}`; internally each `UnionAll` is instantiated with a particular variable value, one at a time, outermost-first.
> `Array {Int、2}`の構文は `Array {Int} {2}`と等価です。 内部的には、それぞれの「UnionAll」は、一度に1つずつ、特定の変数値でインスタンス化されます。
<!-- EN -->
This gives a natural meaning to the omission of trailing type parameters; `Array{Int}` gives a type equivalent to `Array{Int,N} where N`.
> これは、後続の型パラメータの省略に自然な意味を与えます。 `Array {Int}`は `Array {Int、N}と同じ型を返します。

<!-- EN -->
A `TypeVar` is not itself a type, but rather should be considered part of the structure of a `UnionAll` type.
> `TypeVar`自体は型ではなく、むしろ` UnionAll`型の構造の一部と考えるべきです。
<!-- EN -->
Type variables have lower and upper bounds on their values (in the fields `lb` and `ub`).
> 型変数は、値の下限と上限を持ちます（ `lb`と` ub`フィールド）。
<!-- EN -->
The symbol `name` is purely cosmetic.
> 記号 `name`は純粋に化粧品です。
<!-- EN -->
Internally, `TypeVar`s are compared by address, so they are defined as mutable types to ensure that "different" type variables can be distinguished.
> 内部的には、 `TypeVar`はアドレスで比較されるので、「異なる」型変数を区別できるように、可変型として定義されます。
<!-- EN -->
However, by convention they should not be mutated.
> しかし、慣例により、それらは突然変異してはならない。

<!-- EN -->
One can construct `TypeVar`s manually:
> 手動で `TypeVar`を構築することができます：


```jldoctest
julia> TypeVar(:V, Signed, Real)
Signed<:V<:Real
```

<!-- EN -->
There are convenience versions that allow you to omit any of these arguments except the `name` symbol.
> `name`シンボル以外のこれらの引数のどれかを省略できる便利なバージョンがあります。

<!-- EN -->
The syntax `Array{T} where T<:Integer` is lowered to
> 構文 `Array {T}（ここで、T <：Integer`は

```julia
let T = TypeVar(:T,Integer)
    UnionAll(T, Array{T})
end
```

<!-- EN -->
so it is seldom necessary to construct a `TypeVar` manually (indeed, this is to be avoided).
> したがって、 `TypeVar`を手作業で構築する必要はめったにありません（実際これは避けるべきです）。

## Free variables

<!-- EN -->
The concept of a *free* type variable is extremely important in the type system.
<!-- EN -->
We say that a variable `V` is free in type `T` if `T` does not contain the `UnionAll` that introduces variable `V`.
<!-- EN -->
For example, the type `Array{Array{V} where V<:Integer}` has no free variables, but the `Array{V}` part inside of it does have a free variable, `V`.

<!-- EN -->
A type with free variables is, in some sense, not really a type at all.
> 自由変数を持つ型は、ある意味では、まったく型ではありません。
<!-- EN -->
Consider the type `Array{Array{T}} where T`, which refers to all homogeneous arrays of arrays.
> 配列のすべての同次配列を参照する型 `Array {Array {T}}を考えてみましょう。
<!-- EN -->
The inner type `Array{T}`, seen by itself, might seem to refer to any kind of array.
> 内側の型 `Array {T}`はそれ自身で見れば、どんな種類の配列も参照するように見えるかもしれません。
<!-- EN -->
However, every element of the outer array must have the *same* array type, so `Array{T}` cannot refer to just any old array.
> しかし、外部配列のすべての要素は*同じ*配列型を持たなければならないので、 `Array {T}`は古い配列だけを参照することはできません。
<!-- EN -->
One could say that `Array{T}` effectively "occurs" multiple times, and `T` must have the same value each "time".
> `Array {T}`は効果的に複数回出現し、 `T`は毎回同じ値を持たなければなりません。

<!-- EN -->
For this reason, the function `jl_has_free_typevars` in the C API is very important.
> このため、C APIの関数 `jl_has_free_typevars`は非常に重要です。
<!-- EN -->
Types for which it returns true will not give meaningful answers in subtyping and other type functions.
> trueを返す型は、サブタイプや他の型関数で意味のある答えを出すことはありません。

## TypeNames

<!-- EN -->
The following two [`Array`](@ref) types are functionally equivalent, yet print differently:
> 次の2つの[`Array`]（@ref）型は機能的には同等ですが、表示は異なります。

```jldoctest
julia> TV, NV = TypeVar(:T), TypeVar(:N)
(T, N)

julia> Array
Array

julia> Array{TV,NV}
Array{T,N}
```

<!-- EN -->
These can be distinguished by examining the `name` field of the type, which is an object of type `TypeName`:
> これらは、 `TypeName`型のオブジェクトである型の` name`フィールドを調べることで区別できます：

```julia-repl
julia> dump(Array{Int,1}.name)
TypeName
  name: Symbol Array
  module: Module Core
  names: empty SimpleVector
  wrapper: UnionAll
    var: TypeVar
      name: Symbol T
      lb: Core.TypeofBottom Union{}
      ub: Any
    body: UnionAll
      var: TypeVar
        name: Symbol N
        lb: Core.TypeofBottom Union{}
        ub: Any
      body: Array{T,N} <: DenseArray{T,N}
  cache: SimpleVector
    ...

  linearcache: SimpleVector
    ...

  hash: Int64 -7900426068641098781
  mt: MethodTable
    name: Symbol Array
    defs: Nothing nothing
    cache: Nothing nothing
    max_args: Int64 0
    kwsorter: #undef
    module: Module Core
    : Int64 0
    : Int64 0
```

<!-- EN -->
In this case, the relevant field is `wrapper`, which holds a reference to the top-level type used to make new `Array` types.
> この場合、関連するフィールドは `wrapper`であり、これは新しい `Array`型を作るために使われたトップレベルの型への参照を保持します。

```julia-repl
julia> pointer_from_objref(Array)
Ptr{Cvoid} @0x00007fcc7de64850

julia> pointer_from_objref(Array.body.body.name.wrapper)
Ptr{Cvoid} @0x00007fcc7de64850

julia> pointer_from_objref(Array{TV,NV})
Ptr{Cvoid} @0x00007fcc80c4d930

julia> pointer_from_objref(Array{TV,NV}.name.wrapper)
Ptr{Cvoid} @0x00007fcc7de64850
```

<!-- EN -->
The `wrapper` field of [`Array`](@ref) points to itself, but for `Array{TV,NV}` it points back to the original definition of the type.
> [`Array`](@ref)の` wrapper`フィールドはそれ自身を指し示しますが、 `Array {TV、NV}`の場合は型の元の定義を指しています。

<!-- EN -->
What about the other fields? `hash` assigns an integer to each type.
> 他のフィールドはどうですか？ `hash`は各型に整数を代入します。
<!-- EN -->
To examine the `cache` field, it's helpful to pick a type that is less heavily used than Array.
> `cache`フィールドを調べるには、Arrayよりもあまり使われていないタイプを選ぶと便利です。
<!-- EN -->
Let's first create our own type:
> 最初に独自の型を作りましょう：

```jldoctest
julia> struct MyType{T,N} end

julia> MyType{Int,2}
MyType{Int64,2}

julia> MyType{Float32, 5}
MyType{Float32,5}

julia> MyType.body.body.name.cache
svec(MyType{Int64,2}, MyType{Float32,5}, #undef, #undef, #undef, #undef, #undef, #undef)
```

<!-- EN -->
(The cache is pre-allocated to have length 8, but only the first two entries are populated.)
> （キャッシュは長さが8になるように事前に割り当てられていますが、最初の2つのエントリだけが読み込まれます）。
<!-- EN -->
Consequently, when you instantiate a parametric type, each concrete type gets saved in a type cache.
> したがって、パラメトリック型をインスタンス化すると、各具体的な型は型キャッシュに保存されます。
<!-- EN -->
However, instances containing free type variables are not cached.
> ただし、フリー・タイプの変数を含むインスタンスはキャッシュされません。

## Tuple types

<!-- EN -->
Tuple types constitute an interesting special case.
> タプル型は面白い特別なケースを構成します。
<!-- EN -->
For dispatch to work on declarations like `x::Tuple`, the type has to be able to accommodate any tuple.
> ディスパッチが `x :: Tuple`のような宣言で動作するためには、型は任意のタプルに対応できる必要があります。
<!-- EN -->
Let's check the parameters:
> パラメータを確認しましょう：

```jldoctest
julia> Tuple
Tuple

julia> Tuple.parameters
svec(Vararg{Any,N} where N)
```

<!-- EN -->
Unlike other types, tuple types are covariant in their parameters, so this definition permits `Tuple` to match any type of tuple:
> 他の型とは違って、タプル型はそれらのパラメータに共変（covariant）なので、この定義は `Tuple`が任意の型のタプルにマッチすることを可能にします：

```jldoctest
julia> typeintersect(Tuple, Tuple{Int,Float64})
Tuple{Int64,Float64}

julia> typeintersect(Tuple{Vararg{Any}}, Tuple{Int,Float64})
Tuple{Int64,Float64}
```

<!-- EN -->
However, if a variadic (`Vararg`) tuple type has free variables it can describe different kinds of tuples:
> しかし、変数（Vararg`）タプル型が自由変数を持つ場合、それは異なる種類のタプルを記述することができます：

```jldoctest
julia> typeintersect(Tuple{Vararg{T} where T}, Tuple{Int,Float64})
Tuple{Int64,Float64}

julia> typeintersect(Tuple{Vararg{T}} where T, Tuple{Int,Float64})
Union{}
```

<!-- EN -->
Notice that when `T` is free with respect to the `Tuple` type (i.e. its binding `UnionAll` type is outside the `Tuple` type), only one `T` value must work over the whole type.
> 'Tuple'型（つまり、そのバインディング `UnionAll`型が` Tuple`型の外側にあります）に対して `T`が自由であるとき、1つの` T`値だけが型全体で動作しなければならないことに注意してください。
<!-- EN -->
Therefore a heterogeneous tuple does not match.
> したがって、異種タプルは一致しません。

<!-- EN -->
Finally, it's worth noting that `Tuple{}` is distinct:
> 最後に、 `Tuple {}`が別個であることに注目する価値があります：

```jldoctest
julia> Tuple{}
Tuple{}

julia> Tuple{}.parameters
svec()

julia> typeintersect(Tuple{}, Tuple{Int})
Union{}
```

<!-- EN -->
What is the "primary" tuple-type?
> 「プライマリ」タプル型とは何ですか？

```julia-repl
julia> pointer_from_objref(Tuple)
Ptr{Cvoid} @0x00007f5998a04370

julia> pointer_from_objref(Tuple{})
Ptr{Cvoid} @0x00007f5998a570d0

julia> pointer_from_objref(Tuple.name.wrapper)
Ptr{Cvoid} @0x00007f5998a04370

julia> pointer_from_objref(Tuple{}.name.wrapper)
Ptr{Cvoid} @0x00007f5998a04370
```

<!-- EN -->
so `Tuple == Tuple{Vararg{Any}}` is indeed the primary type.
> `Tuple == Tuple {Vararg {Any}}`が主な型です。

## Diagonal types

<!-- EN -->
Consider the type `Tuple{T,T} where T`.
> タイプTuple {T、T}を考えてみましょう。
<!-- EN -->
A method with this signature would look like:
> このシグネチャを持つメソッドは次のようになります。

```julia
f(x::T, y::T) where {T} = ...
```

<!-- EN -->
According to the usual interpretation of a `UnionAll` type, this `T` ranges over all types, including `Any`, so this type should be equivalent to `Tuple{Any,Any}`.
> `UnionAll`型の通常の解釈によれば、この` T`は `Any`を含むすべての型の範囲であるため、この型は` Tuple {Any、Any} 'と等価でなければなりません。
<!-- EN -->
However, this interpretation causes some practical problems.
> しかしながら、この解釈はいくつかの実用上の問題を引き起こす。

<!-- EN -->
First, a value of `T` needs to be available inside the method definition.
> まず、メソッド定義の中で値 'T'を利用できるようにする必要があります。
<!-- EN -->
For a call like `f(1, 1.0)`, it's not clear what `T` should be.
> `f（1、1.0）`のような呼び出しの場合、 `T`がどうあるべきかは不明です。
<!-- EN -->
It could be `Union{Int,Float64}`, or perhaps [`Real`](@ref).
> `Union {Int、Float64}`、おそらくは `` Real``（@ ref）かもしれません。
<!-- EN -->
Intuitively, we expect the declaration `x::T` to mean `T === typeof(x)`.
> 直感的に、 `x :: T`という宣言は` T === typeof（x） `を意味すると期待しています。
<!-- EN -->
To make sure that invariant holds, we need `typeof(x) === typeof(y) === T` in this method.
> 不変量が成り立つことを確かめるために、このメソッドでは `typeof（x）=== typeof（y）=== T`が必要です。
<!-- EN -->
That implies the method should only be called for arguments of the exact same type.
> つまり、このメソッドは、まったく同じ型の引数に対してのみ呼び出される必要があります。

<!-- EN -->
It turns out that being able to dispatch on whether two values have the same type is very useful (this is used by the promotion system for example), so we have multiple reasons to want a different interpretation of `Tuple{T,T} where T`.
> 2つの値の型が同じかどうかをディスパッチできることは非常に便利です（これは例えばプロモーションシステムで使用されています）ので、 `Tuple {T、T}の異なる解釈が必要な理由は複数ありますT 'である。
<!-- EN -->
To make this work we add the following rule to subtyping: if a variable occurs more than once in covariant position, it is restricted to ranging over only concrete types.
> この作業を行うために、以下のルールをサブタイプに追加します。変数が共変位置で複数回出現する場合は、具体的な型だけに限定されます。
<!-- EN -->
("Covariant position" means that only `Tuple` and `Union` types occur between an occurrence of a variable and the `UnionAll` type that introduces it.)
> （ "共変位置"は、変数の出現とそれを紹介する `UnionAll`型の間に` Tuple`と `Union`型だけがあることを意味します）。
<!-- EN -->
Such variables are called "diagonal variables" or "concrete variables".
> このような変数は「対角変数」または「具体変数」と呼ばれます。

<!-- EN -->
So for example, `Tuple{T,T} where T` can be seen as `Union{Tuple{Int8,Int8}, Tuple{Int16,Int16}, ...}`, where `T` ranges over all concrete types.
> 例えばTuple {T、T}は `Union {Tuple {Int8、Int8}、Tuple {Int16、Int16}、...}`と見なすことができます。 。
<!-- EN -->
This gives rise to some interesting subtyping results.
> これは、いくつかの興味深いサブタイプ化の結果をもたらす。
<!-- EN -->
For example `Tuple{Real,Real}` is not a subtype of `Tuple{T,T} where T`, because it includes some types like `Tuple{Int8,Int16}` where the two elements have different types.
> たとえば、 `Tuple {Real、Real}`は `Tuple {T、T} where T 'のサブタイプではありません。なぜなら、2つの要素が異なるタイプの` Tuple {Int8、Int16} `のような型を含んでいるからです。
<!-- EN -->
`Tuple{Real,Real}` and `Tuple{T,T} where T` have the non-trivial intersection `Tuple{T,T} where T<:Real`.
> `Tuple {Real、Real}`と `Tuple {T、T} 'は、T <：Realの場合はTuple {T、T}を持ちます。
<!-- EN -->
However, `Tuple{Real}` *is* a subtype of `Tuple{T} where T`, because in that case `T` occurs only once and so is not diagonal.
> しかし、 `Tuple {Real}` *は* Tuple {T}のサブタイプです。この場合、 `T`は一度だけ出現するので対角ではないからです。

<!-- EN -->
Next consider a signature like the following:
> 次に、次のような署名を考えてみましょう。

```julia
f(a::Array{T}, x::T, y::T) where {T} = ...
```

<!-- EN -->
In this case, `T` occurs in invariant position inside `Array{T}`.
> この場合、 `T 'は` Array {T}'の不変位置にあります。
<!-- EN -->
That means whatever type of array is passed unambiguously determines the value of `T` --- we say `T` has an *equality constraint* on it.
> つまり、どの型の配列が渡されても、Tの値を明白に決める---つまり、 `T`には*等価制約*があります。
<!-- EN -->
Therefore in this case the diagonal rule is not really necessary, since the array determines `T` and we can then allow `x` and `y` to be of any subtypes of `T`.
> したがって、この場合、対角ルールは実際には必要ではありません。なぜなら、配列が `T`を決定し、` x`と `y`が` T`の任意のサブタイプになることができるからです。
<!-- EN -->
So variables that occur in invariant position are never considered diagonal.
> したがって、不変の位置に生じる変数は決して対角とはみなされません。
<!-- EN -->
This choice of behavior is slightly controversial --- some feel this definition should be written as
> この行動の選択は、やや議論の余地があります。

```julia
f(a::Array{T}, x::S, y::S) where {T, S<:T} = ...
```

<!-- EN -->
to clarify whether `x` and `y` need to have the same type.
> `x`と` y`が同じ型を持つ必要があるかどうかを明らかにする。
<!-- EN -->
In this version of the signature they would, or we could introduce a third variable for the type of `y` if `x` and `y` can have different types.
> `x`と` y`が異なる型を持つことができるならば、このバージョンの署名では、 `y`型の第3の変数を導入することができます。

<!-- EN -->
The next complication is the interaction of unions and diagonal variables, e.g.
> 次の複雑さは、組合と対角変数との相互作用である。

```julia
f(x::Union{Nothing,T}, y::T) where {T} = ...
```

<!-- EN -->
Consider what this declaration means.
> この宣言が何を意味するのかを考えてください。
<!-- EN -->
`y` has type `T`. `x` then can have either the same type `T`, or else be of type `Nothing`.
> `y`は` T`型です。 `x`は同じ型` T`を持つことができ、そうでなければ `Nothing`型です。
<!-- EN -->
So all of the following calls should match:
> したがって、次の呼び出しのすべてが一致する必要があります。

```julia
f(1, 1)
f("", "")
f(2.0, 2.0)
f(nothing, 1)
f(nothing, "")
f(nothing, 2.0)
```

<!-- EN -->
These examples are telling us something: when `x` is `nothing::Nothing`, there are no extra constraints on `y`.
> これらの例は私たちに何かを伝えています： `x`が` nothing :: Nothing`であるとき、 `y`に余分な制約はありません。
<!-- EN -->
It is as if the method signature had `y::Any`.
> これは、メソッドのシグネチャに `y :: Any`があるかのようです。
<!-- EN -->
This means that whether a variable is diagonal is not a static property based on where it appears in a type.
> これは、変数が対角であるかどうかは、変数が型のどこに現れるかに基づいて静的なプロパティではないことを意味します。
<!-- EN -->
Rather, it depends on where a variable appears when the subtyping algorithm *uses* it.
> むしろ、サブタイプ化アルゴリズム*がそれを使用するときに変数がどこに現れるかによって異なります。
<!-- EN -->
When `x` has type `Nothing`, we don't need to use the `T` in `Union{Nothing,T}`, so `T` does not "occur".
> `x`が` Nothing`型であるとき、 `Union {Nothing、T} 'に` T`を使う必要はないので `T`は発生しません。
<!-- EN -->
Indeed, we have the following type equivalence:
> 実際、次のような型の同等性があります。

```julia
(Tuple{Union{Nothing,T},T} where T) == Union{Tuple{Nothing,Any}, Tuple{T,T} where T}
```

## Subtyping diagonal variables

<!-- EN -->
The subtyping algorithm for diagonal variables has two components:
> 対角変数のサブタイプ化アルゴリズムには、次の2つの要素があります。
<!-- EN -->
(1) identifying variable occurrences, and (2) ensuring that diagonal variables range over concrete types only.
> （1）変数の出現を特定し、（2）対角変数が具体的な型だけに及ぶことを保証する。

<!-- EN -->
The first task is accomplished by keeping counters `occurs_inv` and `occurs_cov` (in `src/subtype.c`) for each variable in the environment, tracking the number of invariant and covariant occurrences, respectively.
> 最初のタスクは、環境内の変数ごとに `occur_inv`と` occur_cov`（ `src / subtype.c`にある）を保持し、それぞれ不変量と共変量の発生数を追跡することによって実現されます。
<!-- EN -->
A variable is diagonal when `occurs_inv == 0 && occurs_cov > 1`.
> `occur_inv == 0 && occur_cov> 1 'のとき、変数は対角になります。

<!-- EN -->
The second task is accomplished by imposing a condition on a variable's lower bound.
> 第2のタスクは、変数の下限に条件を課すことによって達成される。
<!-- EN -->
As the subtyping algorithm runs, it narrows the bounds of each variable (raising lower bounds and lowering upper bounds) to keep track of the range of variable values for which the subtype relation would hold.
> サブタイプ化アルゴリズムが実行されると、サブタイプの関係が保持する変数値の範囲を追跡するために、各変数の境界が狭められます（下限を上げ、上限を下げる）。
<!-- EN -->
When we are done evaluating the body of a `UnionAll` type whose variable is diagonal, we look at the final values of the bounds.
> 変数が対角である `UnionAll`型のボディを評価し終えたら、境界の最終値を調べます。
<!-- EN -->
Since the variable must be concrete, a contradiction occurs if its lower bound could not be a subtype of a concrete type.
> 変数は具体的でなければならないので、その下限が具体的な型のサブタイプではない場合、矛盾が発生します。
<!-- EN -->
For example, an abstract type like [`AbstractArray`](@ref) cannot be a subtype of a concrete type, but a concrete type like `Int` can be, and the empty type `Bottom` can be as well.
> たとえば、[AbstractArray`]（@ref）のような抽象型は、具体的な型のサブタイプではありませんが、 `Int`のような具体的な型もあり、空の型` Bottom`も可能です。
<!-- EN -->
If a lower bound fails this test the algorithm stops with the answer `false`.
> 下限がこのテストに合格しなかった場合、アルゴリズムは応答が「偽」で停止する。

<!-- EN -->
For example, in the problem `Tuple{Int,String} <: Tuple{T,T} where T`, we derive that this would be true if `T` were a supertype of `Union{Int,String}`.
> たとえば、 `Tuple {Int、String} <：Tuple {T、T} where T`では、` T`が `Union {Int、String}`のスーパータイプであった場合、これが真となることを導き出します。
<!-- EN -->
However, `Union{Int,String}` is an abstract type, so the relation does not hold.
> しかし、 `Union {Int、String}`は抽象型なので、関係は成立しません。

<!-- EN -->
This concreteness test is done by the function `is_leaf_bound`.
> この具体性テストは関数 `is_leaf_bound`によって行われます。
<!-- EN -->
Note that this test is slightly different from `jl_is_leaf_type`, since it also returns `true` for `Bottom`.
> このテストは `jl_is_leaf_type`と少し異なります。これは` Bottom`に対しても `true`を返すためです。
<!-- EN -->
Currently this function is heuristic, and does not catch all possible concrete types.
> 現在のところ、この関数はヒューリスティックであり、すべての可能な具体的な型をキャッチしません。
<!-- EN -->
The difficulty is that whether a lower bound is concrete might depend on the values of other type variable bounds.
> 難しい点は、下限が具体的であるかどうかは、他の型の変数境界の値に依存する可能性があることです。
<!-- EN -->
For example, `Vector{T}` is equivalent to the concrete type `Vector{Int}` only if both the upper and lower bounds of `T` equal `Int`.
> たとえば `Vector {T} 'は` T`の上限と下限の両方が `Int`と等しい場合にのみ具体的な型` Vector {Int}'と等価です。
<!-- EN -->
We have not yet worked out a complete algorithm for this.
> このためのアルゴリズムはまだ完成していません。

## Introduction to the internal machinery

<!-- EN -->
Most operations for dealing with types are found in the files `jltypes.c` and `subtype.c`.
> 型を扱うためのほとんどの操作は `jltypes.c`と` subtype.c`ファイルにあります。
<!-- EN -->
A good way to start is to watch subtyping in action.
> サブタイピングの実際の動作を監視するのが良い方法です。
<!-- EN -->
Build Julia with `make debug` and fire up Julia within a debugger.
> Juliaを `make debug`でビルドし、デバッガ内でJuliaを起動します。
<!-- EN -->
[gdb debugging tips](@ref) has some tips which may be useful.
> [gdbのデバッグのヒント]（@ ref）に役立つヒントがいくつかあります。

<!-- EN -->
Because the subtyping code is used heavily in the REPL itself--and hence breakpoints in this code get triggered often--it will be easiest if you make the following definition:
> サブタイプコードはREPL自体で頻繁に使用されるため、このコードのブレークポイントは頻繁にトリガーされます。次の定義を行うと最も簡単になります。

```julia-repl
julia> function mysubtype(a,b)
           ccall(:jl_breakpoint, Cvoid, (Any,), nothing)
           a <: b
       end
```

<!-- EN -->
and then set a breakpoint in `jl_breakpoint`.
> `jl_breakpoint`にブレークポイントを設定します。
<!-- EN -->
Once this breakpoint gets triggered, you can set breakpoints in other functions.
> このブレークポイントがトリガーされると、他の関数でブレークポイントを設定できます。

<!-- EN -->
As a warm-up, try the following:
> ウォームアップとして、次のことを試してください：

```julia
mysubtype(Tuple{Int,Float64}, Tuple{Integer,Real})
```

<!-- EN -->
We can make it more interesting by trying a more complex case:
> より複雑なケースを試すことで、より面白くすることができます。

```julia
mysubtype(Tuple{Array{Int,2}, Int8}, Tuple{Array{T}, T} where T)
```

## Subtyping and method sorting

<!-- EN -->
The `type_morespecific` functions are used for imposing a partial order on functions in method tables (from most-to-least specific).
> `type_morespecific`関数は、メソッドテーブル内の関数の部分的な順序を（ほとんどのものから最小限のものまで）適用するために使用されます。
<!-- EN -->
Specificity is strict; if `a` is more specific than `b`, then `a` does not equal `b` and `b` is not more specific than `a`.
> 特異性は厳格である。 `a`が` b`よりも具体的であれば、 `a`は` b`と等しくなく、 `b`は` a`よりも具体的ではありません。

<!-- EN -->
If `a` is a strict subtype of `b`, then it is automatically considered more specific.
> `a`が` b`の厳密なサブタイプであれば、それはより自動的により具体的であると考えられます。
<!-- EN -->
From there, `type_morespecific` employs some less formal rules.
> そこから、 `type_morespecific`はあまり公式な規則を採用していません。
<!-- EN -->
For example, `subtype` is sensitive to the number of arguments, but `type_morespecific` may not be.
> 例えば、 `subtype`は引数の数に敏感ですが、` type_morespecific`はそうでないかもしれません。
<!-- EN -->
In particular, `Tuple{Int,AbstractFloat}` is more specific than `Tuple{Integer}`, even though it is not a subtype.
> 特に、 `Tuple {Int、AbstractFloat}`は、サブタイプではないにもかかわらず、 `Tuple {Integer} 'よりも具体的です。
<!-- EN -->
(Of `Tuple{Int,AbstractFloat}` and `Tuple{Integer,Float64}`, neither is more specific than the other.)
> （ `Tuple {Int、AbstractFloat}`と `Tuple {Integer、Float64} 'のうち、どちらも他のものより具体的ではありません）。
<!-- EN -->
Likewise, `Tuple{Int,Vararg{Int}}` is not a subtype of `Tuple{Integer}`, but it is considered more specific.
> 同様に、 `Tuple {Int、Vararg {Int}}`は `Tuple {Integer}`のサブタイプではありませんが、より具体的であると考えられます。
<!-- EN -->
However, `morespecific` does get a bonus for length: in particular, `Tuple{Int,Int}` is more specific than `Tuple{Int,Vararg{Int}}`.
> しかし、 `morespecific`は長さのボーナスを得ます：特に、` Tuple {Int、Int} `は` Tuple {Int、Vararg {Int}}より具体的です。

<!-- EN -->
If you're debugging how methods get sorted, it can be convenient to define the function:
> メソッドのソート方法をデバッグする場合は、関数を定義すると便利です。

```julia
type_morespecific(a, b) = ccall(:jl_type_morespecific, Cint, (Any,Any), a, b)
```

<!-- EN -->
which allows you to test whether tuple type `a` is more specific than tuple type `b`.
> これはタプル型 `a`がタプル型` b`よりも具体的であるかどうかをテストすることを可能にします。
