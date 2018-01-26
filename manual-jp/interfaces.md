<!-- Start -->

# Interfaces

> # インターフェース

<!-- End -->
<!-- Start -->
A lot of the power and extensibility in Julia comes from a collection of informal interfaces.
> Juliaの力と拡張性の多くは、非公式のインターフェースの集まりから来ています。
<!-- End -->
<!-- Start -->
By extending a few specific methods to work for a custom type, objects of that type not only receive those functionalities, but they are also able to be used in other methods that are written to generically build upon those behaviors.
> カスタムタイプのためにいくつかの特定のメソッドを拡張することによって、そのタイプのオブジェクトはそれらの機能を受け取るだけでなく、それらのビヘイビアを一般的に構築するように記述された他のメソッドでも使用できます。
<!-- End -->
<!-- Start -->

## [Iteration](@id man-interface-iteration)

> ## [反復](@ id man-interface-iteration)

<!-- End -->
| Required methods          |              | Brief description                                               |
|:--------------------------|:------------ |:--------------------------------------------------------------- |
| `start(iter)`             |              | Returns the initial iteration state                             |
| `next(iter, state)`       |              | Returns the current item and the next state                     |
| `done(iter, state)`       |              | Tests if there are any items remaining                          |
| **Important optional methods** | **Default definition** | **Brief description**                            |
| `iteratorsize(IterType)`  | `HasLength()`| One of `HasLength()`, `HasShape()`, `IsInfinite()`, or
                                             `SizeUnknown()` as appropriate                                  |
| `iteratoreltype(IterType)`| `HasEltype()`| Either `EltypeUnknown()` or `HasEltype()` as appropriate        |
| `eltype(IterType)`        | `Any`        | The type the items returned by `next()`                         |
| `length(iter)`            | (*undefined*)| The number of items, if known                                   |
| `size(iter, [dim...])`    | (*undefined*)| The number of items in each dimension, if known                 |

| Value returned by `iteratorsize(IterType)` | Required Methods                           |
|:------------------------------------------ |:------------------------------------------ |
| `HasLength()`                              | `length(iter)`                             |
| `HasShape()`                               | `length(iter)`  and `size(iter, [dim...])` |
| `IsInfinite()`                             | (*none*)                                   |
| `SizeUnknown()`                            | (*none*)                                   |

| Value returned by `iteratoreltype(IterType)` | Required Methods   |
|:-------------------------------------------- |:------------------ |
| `HasEltype()`                                | `eltype(IterType)` |
| `EltypeUnknown()`                            | (*none*)           |

<!-- Start -->
Sequential iteration is implemented by the methods [`start()`](@ref), [`done()`](@ref), and [`next()`](@ref). 
> 逐次反復はメソッド[`start()`](@ ref)、[`done()`](@ ref)、[`next()`](@ ref)メソッドによって実装されます。
<!-- End -->
<!-- Start -->
Instead of mutating objects as they are iterated over, Julia provides these three methods to keep track of the iteration state externally from the object. 
> Juliaは、オブジェクトを反復処理する際にオブジェクトを変更する代わりに、オブジェクトから外部に反復状態を追跡するための3つのメソッドを提供します。
<!-- End -->
<!-- Start -->
The `start(iter)` method returns the initial state for the iterable object `iter`. 
> `start(iter)`メソッドは、反復可能オブジェクト `iter`の初期状態を返します。
<!-- End -->
<!-- Start -->
That state gets passed along to `done(iter, state)`, which tests if there are any elements remaining, and `next(iter, state)`, which returns a tuple containing the current element and an updated `state`. 
> その状態は、要素が残っているかどうかをテストする `done(iter、state)`と、現在の要素と更新された `state`を含むタプルを返す` next(iter、state) `に渡されます。
<!-- End -->
<!-- Start -->
The `state` object can be anything, and is generally considered to be an implementation detail private to the iterable object.
> `state`オブジェクトは何でもかまいませんが、一般的にiterableオブジェクトに対して実装の詳細と見なされます。
<!-- End -->

<!-- Start -->
Any object defines these three methods is iterable and can be used in the [many functions that rely upon iteration](@ref lib-collections-iteration).
> これらの3つのメソッドを定義するオブジェクトはすべて反復可能であり、[反復に依存する多くの関数](@ ref lib-collections-iteration)で使用できます。
<!-- End -->
<!-- Start -->
It can also be used directly in a `for` loop since the syntax:
> `for`ループの中で直接使用することもできます：
<!-- End -->

```julia
for i in iter   # or  "for i = iter"
    # body
end
```

<!-- Start -->
is translated into:
> 翻訳された：
<!-- End -->

```julia
state = start(iter)
while !done(iter, state)
    (i, state) = next(iter, state)
    # body
end
```

<!-- Start -->
A simple example is an iterable sequence of square numbers with a defined length:
> 簡単な例は、長さが定義された四角形の反復可能なシーケンスです。
<!-- End -->

```jldoctest squaretype
julia> struct Squares
           count::Int
       end

julia> Base.start(::Squares) = 1

julia> Base.next(S::Squares, state) = (state*state, state+1)

julia> Base.done(S::Squares, state) = state > S.count

julia> Base.eltype(::Type{Squares}) = Int # Note that this is defined for the type

julia> Base.length(S::Squares) = S.count
```

<!-- Start -->
With only [`start`](@ref), [`next`](@ref), and [`done`](@ref) definitions, the `Squares` type is already pretty powerful.
> [`start`](@ref)、[`next`](@ref)、 [`done`](@ref) の定義だけでは、 `Squares` 型はすでにかなり強力です。
<!-- End -->
<!-- Start -->
We can iterate over all the elements:
> すべての要素を繰り返し処理できます。
<!-- End -->

```jldoctest squaretype
julia> for i in Squares(7)
           println(i)
       end
1
4
9
16
25
36
49
```

<!-- Start -->
We can use many of the builtin methods that work with iterables, like [`in()`](@ref), [`mean()`](@ref) and [`std()`](@ref):
> [`in()`](@ ref)、[`mean()`](@ ref)、[`std()`](@ ref)のように、iterablesで動作する組み込みメソッドの多くを使用できます。 ：
<!-- End -->

```jldoctest squaretype
julia> 25 in Squares(10)
true

julia> mean(Squares(100))
3383.5

julia> std(Squares(100))
3024.355854282583
```

<!-- Start -->
There are a few more methods we can extend to give Julia more information about this iterable collection.  
> Juliaにこの繰り返し可能なコレクションについての詳しい情報を与えるために拡張できるメソッドはいくつかあります。
<!-- End -->
<!-- Start -->
We know that the elements in a `Squares` sequence will always be `Int`. 
> `Squares`シーケンスの要素は常に` Int`です。
<!-- End -->
<!-- Start -->
By extending the [`eltype()`](@ref) method, we can give that information to Julia and help it make more specialized code in the more complicated methods. 
> [`eltype()`](@ ref)メソッドを拡張することで、その情報をJuliaに与え、より複雑なメソッドでより特殊化したコードを作ることができます。
<!-- End -->
<!-- Start -->
We also know the number of elements in our sequence, so we can extend [`length()`](@ref), too.
> シーケンスの要素数も知っているので、[`length()`](@ref)も拡張できます。
<!-- End -->

<!-- Start -->
Now, when we ask Julia to [`collect()`](@ref) all the elements into an array it can preallocate a `Vector{Int}` of the right size instead of blindly [`push!`](@ref)ing each element into a `Vector{Any}`:
> 私たちがJuliaにすべての要素を配列に集めるように依頼すると(@ ref)、盲目的に[`push！`](@ ref)の代わりに適切なサイズのVector Intを事前に割り当てることができます )各要素を `Vector {Any}`に変換する：
<!-- End -->

```jldoctest squaretype
julia> collect(Squares(10))' # transposed to save space
1×10 RowVector{Int64,Array{Int64,1}}:
 1  4  9  16  25  36  49  64  81  100
```

<!-- Start -->
While we can rely upon generic implementations, we can also extend specific methods where we know there is a simpler algorithm. 
> 一般的な実装に頼ることもできますが、より単純なアルゴリズムがあることがわかっている特定のメソッドを拡張することもできます。
<!-- End -->
<!-- Start -->
For example, there's a formula to compute the sum of squares, so we can override the generic iterative version with a more performant solution:
> たとえば、平方和を計算する式があるので、より一般的な反復バージョンをより効果的なソリューションで上書きすることができます。
<!-- End -->

```jldoctest squaretype
julia> Base.sum(S::Squares) = (n = S.count; return n*(n+1)*(2n+1)÷6)

julia> sum(Squares(1803))
1955361914
```

<!-- Start -->
This is a very common pattern throughout the Julia standard library: a small set of required methods define an informal interface that enable many fancier behaviors. 
> これは、Julia標準ライブラリ全体を通して非常に一般的なパターンです。少数の必須メソッドは、多くの魅力的な動作を可能にする非公式のインターフェイスを定義します。
<!-- End -->
<!-- Start -->
In some cases, types will want to additionally specialize those extra behaviors when they know a more efficient algorithm can be used in their specific case.
> 場合によっては、より効率的なアルゴリズムを特定のケースで使用できることが分かっている場合、型はそれらの余分な動作をさらに特殊化することを望みます。
<!-- End -->
<!-- Start -->

## Indexing

> ## 索引付け

<!-- End -->
| Methods to implement | Brief description                |
|:-------------------- |:-------------------------------- |
| `getindex(X, i)`     | `X[i]`, indexed element access   |
| `setindex!(X, v, i)` | `X[i] = v`, indexed assignment   |
| `endof(X)`           | The last index, used in `X[end]` |

<!-- Start -->
For the `Squares` iterable above, we can easily compute the `i`th element of the sequence by squaring it.  
> 上記の「正方形」については、シーケンスの第i要素を2乗で簡単に計算することができます。
<!-- End -->
<!-- Start -->
We can expose this as an indexing expression `S[i]`. 
> これをインデックス式 `S [i]`として公開することができます。
<!-- End -->
<!-- Start -->
To opt into this behavior, `Squares` simply needs to define [`getindex()`](@ref):
> この振る舞いを選ぶために、 `Squares`は単に[` getindex() `](@ ref)を定義するだけです：
<!-- End -->

```jldoctest squaretype
julia> function Base.getindex(S::Squares, i::Int)
           1 <= i <= S.count || throw(BoundsError(S, i))
           return i*i
       end

julia> Squares(100)[23]
529
```

<!-- Start -->
Additionally, to support the syntax `S[end]`, we must define [`endof()`](@ref) to specify the last valid index:
> さらに、構文 `S [end]`をサポートするためには、最後の有効なインデックスを指定するために[`end of()`](@ ref)を定義する必要があります：
<!-- End -->

```jldoctest squaretype
julia> Base.endof(S::Squares) = length(S)

julia> Squares(23)[end]
529
```

<!-- Start -->
Note, though, that the above *only* defines [`getindex()`](@ref) with one integer index. 
> ただし、上記の* only *は1つの整数インデックスを持つ[`getindex()`](@ ref)を定義しています。
<!-- End -->
<!-- Start -->
Indexing with anything other than an `Int` will throw a [`MethodError`](@ref) saying that there was no matching method.
> `Int`以外のものでインデックスを作成すると、一致するメソッドがないという[MethodError`](@ ref)がスローされます。
<!-- End -->
<!-- Start -->
In order to support indexing with ranges or vectors of `Int`s, separate methods must be written:
> `Int`の範囲やベクトルでインデックスをサポートするためには、別々のメソッドを記述しなければなりません：
<!-- End -->

```jldoctest squaretype
julia> Base.getindex(S::Squares, i::Number) = S[convert(Int, i)]

julia> Base.getindex(S::Squares, I) = [S[i] for i in I]

julia> Squares(10)[[3,4.,5]]
3-element Array{Int64,1}:
  9
 16
 25
```

<!-- Start -->
While this is starting to support more of the [indexing operations supported by some of the builtin types](@ref man-array-indexing), there's still quite a number of behaviors missing. 
> これは、[組み込み型のいくつかでサポートされているインデックス作成操作](@ ref man-array-indexing)の多くをサポートし始めていますが、依然として欠けている動作がかなりあります。
<!-- End -->
<!-- Start -->
This `Squares` sequence is starting to look more and more like a vector as we've added behaviors to it. 
> この「正方形」シーケンスは、我々がそれに行動を加えたときにますますベクトルのように見え始めています。
<!-- End -->
<!-- Start -->
Instead of defining all these behaviors ourselves, we can officially define it as a subtype of an [`AbstractArray`](@ref).
> これらの振る舞いをすべて自分で定義するのではなく、[`AbstractArray`](@ ref)のサブタイプとして正式に定義することができます。
<!-- End -->
<!-- Start -->

## [Abstract Arrays](@id man-interface-array)

> ## [抽象配列](@ id man-interface-array)

<!-- End -->
| Methods to implement                            |                                          | Brief description                                                                     |
|:----------------------------------------------- |:---------------------------------------- |:------------------------------------------------------------------------------------- |
| `size(A)`                                       |                                          
                    | Returns a tuple containing the dimensions of `A`                                      |
| `getindex(A, i::Int)`                           |                                          
                    | (if `IndexLinear`) Linear scalar indexing                                              |
| `getindex(A, I::Vararg{Int, N})`                |                                          
                    | (if `IndexCartesian`, where `N = ndims(A)`) N-dimensional scalar indexing              |
| `setindex!(A, v, i::Int)`                       |                                          
                    | (if `IndexLinear`) Scalar indexed assignment                                           |
| `setindex!(A, v, I::Vararg{Int, N})`            |                                          
                    | (if `IndexCartesian`, where `N = ndims(A)`) N-dimensional scalar indexed assignment   |
| **Optional methods**                            | **Default definition**                   
                    | **Brief description**                                                                 |
| `IndexStyle(::Type)`                            | `IndexCartesian()`                       
                    | Returns either `IndexLinear()` or `IndexCartesian()`. See the description below.      |
| `getindex(A, I...)`                             | defined in terms of scalar `getindex()`  
                    | [Multidimensional and nonscalar indexing](@ref man-array-indexing)                    |
| `setindex!(A, I...)`                            | defined in terms of scalar `setindex!()` 
                    | [Multidimensional and nonscalar indexed assignment](@ref man-array-indexing)          |
| `start()`/`next()`/`done()`                     | defined in terms of scalar `getindex()`  
                    | Iteration                                                                             |
| `length(A)`                                     | `prod(size(A))`                          
                    | Number of elements                                                                    |
| `similar(A)`                                    | `similar(A, eltype(A), size(A))`         
                    | Return a mutable array with the same shape and element type                           |
| `similar(A, ::Type{S})`                         | `similar(A, S, size(A))`                 
                    | Return a mutable array with the same shape and the specified element type             |
| `similar(A, dims::NTuple{Int})`                 | `similar(A, eltype(A), dims)`            
                    | Return a mutable array with the same element type and size *dims*                     |
| `similar(A, ::Type{S}, dims::NTuple{Int})`      | `Array{S}(dims)`                         
                    | Return a mutable array with the specified element type and size                       |
| **Non-traditional indices**                     | **Default definition**                   
                    | **Brief description**                                                                 |
| `indices(A)`                                    | `map(OneTo, size(A))`                    
                    | Return the `AbstractUnitRange` of valid indices                                       |
| `Base.similar(A, ::Type{S}, inds::NTuple{Ind})` | `similar(A, S, Base.to_shape(inds))`     
                    | Return a mutable array with the specified indices `inds` (see below)                  |
| `Base.similar(T::Union{Type,Function}, inds)`   | `T(Base.to_shape(inds))`                 
                    | Return an array similar to `T` with the specified indices `inds` (see below)          |

<!-- Start -->
If a type is defined as a subtype of `AbstractArray`, it inherits a very large set of rich behaviors including iteration and multidimensional indexing built on top of single-element access.  
> 型が `AbstractArray`のサブタイプとして定義されている場合、型は単一要素アクセスの上に構築された繰り返しと多次元索引付けを含む非常に大きな一連の豊富な動作を継承します。
<!-- End -->
<!-- Start -->
See the [arrays manual page](@ref man-multi-dim-arrays) and [standard library section](@ref lib-arrays) for more supported methods.
> サポートされている他のメソッドについては、[配列マニュアルページ](@ ref man-multi-dim-arrays)と[標準ライブラリセクション](@ ref lib-arrays)を参照してください。
<!-- End -->

<!-- Start -->
A key part in defining an `AbstractArray` subtype is [`IndexStyle`](@ref). 
> `AbstractArray`サブタイプを定義する上での重要な部分は[IndexStyle`](@ ref)です。
<!-- End -->
<!-- Start -->
Since indexing is such an important part of an array and often occurs in hot loops, it's important to make both indexing and indexed assignment as efficient as possible.  
> 索引付けは配列の重要な部分であり、頻繁にホット・ループで行われるため、索引付けと索引付けの両方を効率的に行うことが重要です。
<!-- End -->
<!-- Start -->
Array data structures are typically defined in one of two ways: either it most efficiently accesses its elements using just one index (linear indexing) or it intrinsically accesses the elements with indices specified for every dimension.
>配列データ構造は、通常、1つのインデックス(線形インデクシング)を使用して要素に最も効率的にアクセスするか、すべての次元に指定されたインデックスを持つ要素に本質的にアクセスするかの2つの方法のいずれかで定義されます。
<!-- End -->
<!-- Start -->
These two modalities are identified by Julia as `IndexLinear()` and `IndexCartesian()`.
> これら2つのモダリティは、Juliaによって `IndexLinear()`と `IndexCartesian()`として識別されます。
<!-- End -->
<!-- Start -->
Converting a linear index to multiple indexing subscripts is typically very expensive, so this provides a traits-based mechanism to enable efficient generic code for all array types.
> リニアインデックスを複数のインデックス付きサブスクリプトに変換するのは通常非常にコストがかかるため、すべての配列タイプに対して効率的な汎用コードを有効にするための特性ベースのメカニズムが提供されます。
<!-- End -->

<!-- Start -->
This distinction determines which scalar indexing methods the type must define. 
> この区別は、型が定義しなければならないスカラー索引付け方法を決定します。
<!-- End -->
<!-- Start -->
`IndexLinear()` arrays are simple: just define `getindex(A::ArrayType, i::Int)`.  
> `IndexLinear()`配列は単純です： `getindex(A :: ArrayType、i :: Int)`を定義するだけです。
<!-- End -->
<!-- Start -->
When the array is subsequently indexed with a multidimensional set of indices, the fallback `getindex(A::AbstractArray, I...)()` efficiently converts the indices into one linear index and then calls the above method. 
> その後、配列が多次元インデックスのインデックスで索引付けされると、フォールバック `getindex(A :: AbstractArray、I ...)()`は索引を1つの線形索引に効率的に変換し、上記のメソッドを呼び出します。
<!-- End -->
<!-- Start -->
`IndexCartesian()` arrays, on the other hand, require methods to be defined for each supported dimensionality with `ndims(A)` `Int` indices. 
> 一方、 `IndexCartesian()`配列は `ndims(A)` `Int`インデックスでサポートされている次元ごとにメソッドを定義する必要があります。
<!-- End -->
<!-- Start -->
For example, the built-in [`SparseMatrixCSC`](@ref) type only supports two dimensions, so it just defines
> たとえば、組み込みの[`SparseMatrixCSC`](@ ref)型は2つの次元しかサポートしないので、
<!-- End -->
<!-- Start -->
`getindex(A::SparseMatrixCSC, i::Int, j::Int)`. 
> `getindex(A :: SparseMatrixCSC、i :: Int、j :: Int)`です。
<!-- End -->
<!-- Start -->
The same holds for `setindex!()`.
> `setindex！()`も同様です。
<!-- End -->

<!-- Start -->
Returning to the sequence of squares from above, we could instead define it as a subtype of an `AbstractArray{Int, 1}`:
> 上記の四角形のシーケンスに戻って、代わりに `AbstractArray {Int、1}`のサブタイプとして定義することができます：
<!-- End -->

```jldoctest squarevectype
julia> struct SquaresVector <: AbstractArray{Int, 1}
           count::Int
       end

julia> Base.size(S::SquaresVector) = (S.count,)

julia> Base.IndexStyle(::Type{<:SquaresVector}) = IndexLinear()

julia> Base.getindex(S::SquaresVector, i::Int) = i*i
```

<!-- Start -->
Note that it's very important to specify the two parameters of the `AbstractArray`; the first defines the [`eltype()`](@ref), and the second defines the [`ndims()`](@ref). 
> `AbstractArray`の2つのパラメータを指定することは非常に重要です。 最初の要素は[eltype() `](@ ref)を定義し、2番目の要素は[` ndims() `](@ ref)を定義します。
<!-- End -->
<!-- Start -->
That supertype and those three methods are all it takes for `SquaresVector` to be an iterable, indexable, and completely functional array:
> そのスーパータイプとその3つのメソッドは、 `SquaresVector`が反復可能で、索引可能で、完全に機能的な配列であるために必要なすべてです。
<!-- End -->

```jldoctest squarevectype
julia> s = SquaresVector(7)
7-element SquaresVector:
  1
  4
  9
 16
 25
 36
 49

julia> s[s .> 20]
3-element Array{Int64,1}:
 25
 36
 49

julia> s \ [1 2; 3 4; 5 6; 7 8; 9 10; 11 12; 13 14]
1×2 Array{Float64,2}:
 0.305389  0.335329

julia> s ⋅ s # dot(s, s)
4676
```

<!-- Start -->
As a more complicated example, let's define our own toy N-dimensional sparse-like array type built on top of [`Dict`](@ref):
> より複雑な例として、[Dict`](@ ref)の上に構築された独自のおもちゃのN次元スパース型配列型を定義しましょう：
<!-- End -->

```jldoctest squarevectype
julia> struct SparseArray{T,N} <: AbstractArray{T,N}
           data::Dict{NTuple{N,Int}, T}
           dims::NTuple{N,Int}
       end

julia> SparseArray{T}(::Type{T}, dims::Int...) = SparseArray(T, dims);

julia> SparseArray{T,N}(::Type{T}, dims::NTuple{N,Int}) = SparseArray{T,N}(Dict{NTuple{N,Int}, T}(), dims);

julia> Base.size(A::SparseArray) = A.dims

julia> Base.similar(A::SparseArray, ::Type{T}, dims::Dims) where {T} = SparseArray(T, dims)

julia> Base.getindex(A::SparseArray{T,N}, I::Vararg{Int,N}) where {T,N} = get(A.data, I, zero(T))

julia> Base.setindex!(A::SparseArray{T,N}, v, I::Vararg{Int,N}) where {T,N} = (A.data[I] = v)
```

<!-- Start -->
Notice that this is an `IndexCartesian` array, so we must manually define [`getindex()`](@ref) and [`setindex!()`](@ref) at the dimensionality of the array. 
> これは `IndexCartesian`配列なので、配列の次元数で[getindex()`](@ref)と[`setindex！()`](@ref)を手動で定義する必要があります。
<!-- End -->
<!-- Start -->
Unlike the `SquaresVector`, we are able to define [`setindex!()`](@ref), and so we can mutate the array:
> `SquaresVector`とは異なり、[` setindex！() `](@ ref)を定義することができるので、配列を変更することができます：
<!-- End -->

```jldoctest squarevectype
julia> A = SparseArray(Float64, 3, 3)
3×3 SparseArray{Float64,2}:
 0.0  0.0  0.0
 0.0  0.0  0.0
 0.0  0.0  0.0

julia> fill!(A, 2)
3×3 SparseArray{Float64,2}:
 2.0  2.0  2.0
 2.0  2.0  2.0
 2.0  2.0  2.0

julia> A[:] = 1:length(A); A
3×3 SparseArray{Float64,2}:
 1.0  4.0  7.0
 2.0  5.0  8.0
 3.0  6.0  9.0
```

<!-- Start -->
The result of indexing an `AbstractArray` can itself be an array (for instance when indexing by a `Range`). 
> `AbstractArray`を索引付けした結果は、それ自身が配列になります(例えば、` Range`による索引付けの場合)。
<!-- End -->
<!-- Start -->
The `AbstractArray` fallback methods use [`similar()`](@ref) to allocate an `Array` of the appropriate size and element type, which is filled in using the basic indexing method described above. 
> `AbstractArray`フォールバックメソッドは適切なサイズと要素型の` Array`を割り当てるために[`similar()`](@ref)を使います。これは上記の基本的なインデックス方法を使って埋められます。
<!-- End -->
<!-- Start -->
However, when implementing an array wrapper you often want the result to be wrapped as well:
> しかし、配列ラッパーを実装するときには、結果をラップすることもしばしばあります。
<!-- End -->

```jldoctest squarevectype
julia> A[1:2,:]
2×3 SparseArray{Float64,2}:
 1.0  4.0  7.0
 2.0  5.0  8.0
```

<!-- Start -->
In this example it is accomplished by defining `Base.similar{T}(A::SparseArray, ::Type{T}, dims::Dims)` to create the appropriate wrapped array. 
> この例では、 `Base.similar {T}(A :: SparseArray、:: Type {T}、dims :: Dims)`を定義して、適切なラップされた配列を作成します。
<!-- End -->
<!-- Start -->
(Note that while `similar` supports 1- and 2-argument forms, in most case you only need to specialize the 3-argument form.) 
> ( `similar 'は1引数型と2引数型をサポートしますが、ほとんどの場合、3引数型を特化する必要があることに注意してください)。
<!-- End -->
<!-- Start -->
For this to work it's important that `SparseArray` is mutable (supports `setindex!`). 
> これを行うには、 `SparseArray`が変更可能であることが重要です(` setindex！ `をサポートしています)。
<!-- End -->
<!-- Start -->
Defining `similar()`, `getindex()` and `setindex!()` for `SparseArray` also makes it possible to [`copy()`](@ref) the array:
> `SparseArray`に` similar() `、` getindex() `と` setindex！() `を定義することによって、配列を[copy()`](@ ref)することもできます：
<!-- End -->

```jldoctest squarevectype
julia> copy(A)
3×3 SparseArray{Float64,2}:
 1.0  4.0  7.0
 2.0  5.0  8.0
 3.0  6.0  9.0
```

<!-- Start -->
In addition to all the iterable and indexable methods from above, these types can also interact with each other and use most of the methods defined in the standard library for `AbstractArrays`:
> 上記のすべての反復可能なメソッドとインデックス可能なメソッドに加えて、これらの型は相互にやりとりして、 `AbstractArrays`に標準ライブラリで定義されているメソッドの大部分を使用することもできます：
<!-- End -->

```jldoctest squarevectype
julia> A[SquaresVector(3)]
3-element SparseArray{Float64,1}:
 1.0
 4.0
 9.0

julia> dot(A[:,1],A[:,2])
32.0
```

<!-- Start -->
If you are defining an array type that allows non-traditional indexing (indices that start at something other than 1), you should specialize `indices`. 
> 非従来の索引付け(1以外の索引で始まる索引付け)を可能にする配列型を定義する場合は、 `indices`を特化する必要があります。
<!-- End -->
<!-- Start -->
You should also specialize [`similar`](@ref) so that the `dims` argument (ordinarily a `Dims` size-tuple) can accept `AbstractUnitRange` objects, perhaps range-types `Ind` of your own design. 
> また、 `dims`引数(通常は` Dims`サイズのタプル)が `AbstractUnitRange`オブジェクト、おそらくあなた自身のデザインの範囲型` Ind`を受け入れることができるように、[`similar`](@ref)を特殊化するべきです。
<!-- End -->
<!-- Start -->
For more information, see [Arrays with custom indices](@ref).
> 詳細については、[カスタムインデックスを持つ配列](@ ref)を参照してください。
<!-- End -->
