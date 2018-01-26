# [Multi-dimensional Arrays](@id man-multi-dim-arrays)

Julia, like most technical computing languages, provides a first-class array implementation. 
> ジュリアは、ほとんどのテクニカルコンピューティング言語のように、ファーストクラスの配列実装を提供します。
Most technical computing languages pay a lot of attention to their array implementation at the expense of other containers. 
> ほとんどの技術計算言語は、他のコンテナを犠牲にして配列の実装に多くの注意を払っています。
Julia does not treat arrays in any special way.
> Juliaは特別な方法で配列を扱いません。
The array library is implemented almost completely in Julia itself, and derives its performance from the compiler, just like any other code written in Julia.
> 配列ライブラリはJulia自体でほぼ完全に実装されており、Juliaで書かれた他のコードと同様に、コンパイラからその性能を引き出します。

As such, it's also possible to define custom array types by inheriting from `AbstractArray.` 
> そのため、`AbstractArray`から継承してカスタム配列型を定義することもできます。
See the [manual section on the AbstractArray interface] for more details on implementing a custom array type.
> カスタム配列型の実装の詳細については、[AbstractArrayインタフェースのmanual section](@ref man-interface-array)を参照してください。

An array is a collection of objects stored in a multi-dimensional grid. 
> 配列は、オブジェクトの集合として多次元グリッドに格納されています。
In the most general case, an array may contain objects of type `Any`. 
> 最も一般的な場合、配列は `Any`型のオブジェクトを含むことができます。
For most computational purposes, arrays should contain objects of a more specific type, such as [`Float64`] or [`Int32`](@ref).
> ほとんどの計算目的のために、配列には、[`Float64`](@ref) や[` Int32`](@ref)のようなより具体的な型のオブジェクトが含まれていなければなりません。

In general, unlike many other technical computing languages, Julia does not expect programs to be written in a vectorized style for performance. 
> 一般に、他の多くのテクニカルコンピューティング言語とは異なり、Juliaはプログラムをベクトル化してパフォーマンスを向上させることは考えていません。
Julia's compiler uses type inference and generates optimized code for scalar array indexing, allowing programs to be written in a style that is convenient and readable, without sacrificing performance, and using less memory at times.
> Juliaのコンパイラは型推論を使用し、スカラー配列の索引付けのための最適化されたコードを生成するので、パフォーマンスを犠牲にすることなく、時には少ないメモリを使用して、プログラムを便利で読みやすいスタイルで書くことができます。

In Julia, all arguments to functions are passed by reference.
> Juliaでは、関数へのすべての引数は参照渡しです。
Some technical computing languages pass arrays by value, and this is convenient in many cases. 
> いくつかのテクニカルコンピューティング言語は、配列を値渡しします。これは多くの場合に便利です。
In Julia, modifications made to input arrays within a function will be visible in the parent function.
> Juliaでは、関数内で入力配列に加えられた変更が、親関数に表示されます。
The entire Julia array library ensures that inputs are not modified by library functions. 
> Julia配列ライブラリ全体は、ライブラリ関数によって入力が変更されないようにします。
User code, if it needs to exhibit similar behavior, should take care to create a copy of inputs that it may modify.
> 同様の振る舞いを示す必要がある場合は、ユーザーコードが変更可能な入力のコピーを作成するよう注意してください。

## Arrays
> ## 配列 

### Basic Functions
> ### 基本的な関数

| Function               | Description                                                                      |
|:---------------------- |:-------------------------------------------------------------------------------- |
| [`eltype(A)`](@ref)    | the type of the elements contained in `A`                                        |
| [`length(A)`](@ref)    | the number of elements in `A`                                                    |
| [`ndims(A)`](@ref)     | the number of dimensions of `A`                                                  |
| [`size(A)`](@ref)      | a tuple containing the dimensions of `A`                                         |
| [`size(A,n)`](@ref)    | the size of `A` along dimension `n`                                              |
| [`indices(A)`](@ref)   | a tuple containing the valid indices of `A`                                      |
| [`indices(A,n)`](@ref) | a range expressing the valid indices along dimension `n`                         |
| [`eachindex(A)`](@ref) | an efficient iterator for visiting each position in `A`                          |
| [`stride(A,k)`](@ref)  | the stride (linear index distance between adjacent elements) along dimension `k` |
#| [`strides(A)`](@ref)   | a tuple of the strides in each dimension                                         |
| [`strides(A)`](@ref)   | 各次元のタプル増分値                                         |

## Construction and Initialization
> ## コンストラクタと初期化

Many functions for constructing and initializing arrays are provided.
> 配列の作成と初期化のための多くの関数が用意されています。
In the following list of such functions, calls with a `dims...` argument can either take a single tuple of dimension sizes or a series of dimension sizes passed as a variable number of arguments.
> 次のような関数のリストでは、 `dims ...`引数を持つ呼び出しは、次元数の単一タプルか、可変数の引数として渡される一連の次元サイズのいずれかを取ることができます。
Most of these functions also accept a first input `T`, which is the element type of the array.
> これらの関数のほとんどは、配列の要素型である第1の入力`T`も受け入れます。
If the type `T` is omitted it will default to [`Float64`](@ref).
> `T`型が省略された場合は、デフォルトで[`Float64`](@ref)になります。


| Function                           | Description                                                                                                           |
|:---------------------------------- |:--------------------------------------------------------------------------------------------------------------------- |
| [`Array{T}(dims...)`](@ref)        | an uninitialized dense [`Array`](@ref)                                                                                |
| [`zeros(T, dims...)`](@ref)        | an `Array` of all zeros                                                                                               |
| [`zeros(A)`](@ref)                 | an array of all zeros with the same type, element type and shape as `A`                                               |
| [`ones(T, dims...)`](@ref)         | an `Array` of all ones                                                                                                |
| [`ones(A)`](@ref)                  | an array of all ones with the same type, element type and shape as `A`                                                |
| [`trues(dims...)`](@ref)           | a [`BitArray`](@ref) with all values `true`                                                                           |
| [`trues(A)`](@ref)                 | a `BitArray` with all values `true` and the same shape as `A`                                                         |
| [`falses(dims...)`](@ref)          | a `BitArray` with all values `false`                                                                                  |
| [`falses(A)`](@ref)                | a `BitArray` with all values `false` and the same shape as `A`                                                        |
| [`reshape(A, dims...)`](@ref)      | an array containing the same data as `A`, but with different dimensions                                               |
| [`copy(A)`](@ref)                  | copy `A`                                                                                                              |
| [`deepcopy(A)`](@ref)              | copy `A`, recursively copying its elements                                                                            |
| [`similar(A, T, dims...)`](@ref)   | an uninitialized array of the same type as `A` (dense, sparse, etc.), but with the specified element type and        
|                                    | dimensions. The second and third arguments are both optional, defaulting to the element type and dimensions of `A` 
|                                    | if omitted.                                                                                                           |
| [`reinterpret(T, A)`](@ref)        | an array with the same binary data as `A`, but with element type `T`                                                  |
| [`rand(T, dims...)`](@ref)         | an `Array` with random, iid [^1] and uniformly distributed values in the half-open interval ``[0, 1)``                |
| [`randn(T, dims...)`](@ref)        | an `Array` with random, iid and standard normally distributed values                                                  |
| [`eye(T, n)`](@ref)                | `n`-by-`n` identity matrix                                                                                            |
| [`eye(T, m, n)`](@ref)             | `m`-by-`n` identity matrix                                                                                            |
| [`linspace(start, stop, n)`](@ref) | range of `n` linearly spaced elements from `start` to `stop`                                                          |
| [`fill!(A, x)`](@ref)              | fill the array `A` with the value `x`                                                                                 |
| [`fill(x, dims...)`](@ref)         | an `Array` filled with the value `x`                                                                                  |

[^1]: *iid*, independently and identically distributed.

The syntax `[A, B, C, ...]` constructs a 1-d array (vector) of its arguments.
> 次の構文 `[A,B,C,...]` は、1次元配列(ベクトル)の引数を構築します。
If all arguments have a common [promotion type](@ref conversion-and-promotion) then they get converted to that type using `convert()`.
> すべての引数が共通の [プロモーションタイプ](@ref conversion-and-promotion) を持つ場合、それらは `convert()` を使ってそのタイプに変換されます。


## Concatenation
> ### 連結

Arrays can be constructed and also concatenated using the following functions:
> 配列は、次の関数を使用して構築し、連結することもできます。

| Function               | Description                                          |
|:---------------------- |:---------------------------------------------------- |
| [`cat(k, A...)`](@ref) | concatenate input n-d arrays along the dimension `k` |
| [`vcat(A...)`](@ref)   | shorthand for `cat(1, A...)`                         |
| [`hcat(A...)`](@ref)   | shorthand for `cat(2, A...)`                         |

Scalar values passed to these functions are treated as 1-element arrays.
> これらの関数に渡されるスカラー値は、1要素配列として扱われます。

The concatenation functions are used so often that they have special syntax:
> 連結関数は頻繁に使用され、特別な構文を持っています：

| Expression        | Calls             |
|:----------------- |:----------------- |
| `[A; B; C; ...]`  | [`vcat()`](@ref)  |
| `[A B C ...]`     | [`hcat()`](@ref)  |
| `[A B; C D; ...]` | [`hvcat()`](@ref) |

[`hvcat()`](@ref) concatenates in both dimension 1 (with semicolons) and dimension 2 (with spaces).
> [`hvcat()`](@ref)は次元1(セミコロン)と次元2(空白)を連結します。

### Typed array initializers
> ### 型付き配列初期化子(Typed array initializers)

An array with a specific element type can be constructed using the syntax `T[A, B, C, ...]`.
> 特定の要素型を持つ配列は`T [A,B,C, ...]`という構文で作ることができます。
This will construct a 1-d array with element type `T`, initialized to contain elements `A`, `B`, `C`, etc.
> これは要素型 `T` を持つ1次元配列を構築し、`A`,`B`,`C` などの要素を含むように初期化されます。
For example `Any[x, y, z]` constructs a heterogeneous array that can contain any values.
> 例えば `Any[x,y,z]` は任意の値を含むことができる異種配列を構築します。

Concatenation syntax can similarly be prefixed with a type to specify the element type of the result.
> 連結構文には同様に、結果の要素タイプを指定する型の接頭辞を付けることができます。

```jldoctest
julia> [[1 2] [3 4]]
1×4 Array{Int64,2}:
 1  2  3  4

julia> Int8[[1 2] [3 4]]
1×4 Array{Int8,2}:
 1  2  3  4
```

## Comprehensions
> ## 内包表記(Comprehensions)

Comprehensions provide a general and powerful way to construct arrays.
> 内包表記は、配列を構築するための一般的で強力な方法を提供します。
Comprehension syntax is similar to set construction notation in mathematics:
> 内包表記の構文は、数学の構造表記法と似ています。

```
A = [ F(x,y,...) for x=rx, y=ry, ... ]
```

The meaning of this form is that `F(x,y,...)` is evaluated with the variables `x`, `y`, etc. taking on each value in their given list of values. 
> この形式の意味は `F(x,y,...)`は変数 `x`, `y` などで評価され、与えられた値のリストの各値をとることです。
Values can be specified as any iterable object, but will commonly be ranges like `1:n` or `2:(n-1)`, or explicit arrays of values like `[1.2, 3.4, 5.7]`.
> 値は任意の反復可能オブジェクトとして指定できますが、通常は `1：n` や `2:(n-1)` のような範囲、`[1.2,3.4,5.7]` のような値の明示的な配列です。

The result is an N-d dense array with dimensions that are the concatenation of the dimensions of the variable ranges `rx`, `ry`, etc. and each `F(x,y,...)` evaluation returns a scalar.
> その結果、可変の寸法の連結が `rx`,`ry`,等と各 `F(X,Y,...)` 評価はスカラーを返すの範囲である寸法を有する N-D高密度配列です。

The following example computes a weighted average of the current element and its left and right neighbor along a 1-d grid. :
> 次の例では、現在の要素と1次元グリッドに沿って左、右隣の加重平均を計算します:

```julia-repl
julia> x = rand(8)
8-element Array{Float64,1}:
 0.843025
 0.869052
 0.365105
 0.699456
 0.977653
 0.994953
 0.41084
 0.809411

julia> [ 0.25*x[i-1] + 0.5*x[i] + 0.25*x[i+1] for i=2:length(x)-1 ]
6-element Array{Float64,1}:
 0.736559
 0.57468
 0.685417
 0.912429
 0.8446
 0.656511
```

The resulting array type depends on the types of the computed elements. 
> 結果の配列型は、計算された要素の型に依存します。
In order to control the type explicitly, a type can be prepended to the comprehension. 
> タイプを明示的に制御するために、タイプを内包表現の前に付けることができます。
For example, we could have requested the result in single precision by writing:
> たとえば、次のように書いて単精度の結果を求めることができます。

```julia
Float32[ 0.25*x[i-1] + 0.5*x[i] + 0.25*x[i+1] for i=2:length(x)-1 ]
```

## Generator Expressions
> ## ジェネレータ式

Comprehensions can also be written without the enclosing square brackets, producing an object known as a generator.
> 内包表現は、囲み角括弧を付けずに記入して、ジェネレータと呼ばれるオブジェクトを作成することもできます。
This object can be iterated to produce values on demand, instead of allocating an array and storing them in advance (see [Iteration](@ref)).
> このオブジェクトは、配列を割り当てて事前に格納する代わりに、必要に応じて値を生成するために反復することができます([反復](@ref)を参照)。
> このオブジェクトは、必要に応じて値を繰り返し生成できます、事前に配列を割り付けて格納しません。(参照 [Iteration](@ref))
For example, the following expression sums a series without allocating memory:
> たとえば、次の式は、メモリを割り当てずに系列を合計します。


```jldoctest
julia> sum(1/n^2 for n=1:1000)
1.6439345666815615
```

When writing a generator expression with multiple dimensions inside an argument list, parentheses are needed to separate the generator from subsequent arguments:
> 複数のディメンションを持つジェネレータ式を引数リストの中に書くときは、ジェネレータと後続の引数を区別するために括弧が必要です。

```julia-repl
julia> map(tuple, 1/(i+j) for i=1:2, j=1:2, [1:4;])
ERROR: syntax: invalid iteration specification
```

All comma-separated expressions after `for` are interpreted as ranges.
> `for` の後のすべてのコンマ区切り式は範囲として解釈されます。
Adding parentheses lets us add a third argument to `map`:
> カッコを追加すると、`map` に3番目の引数を追加できます：

```jldoctest
julia> map(tuple, (1/(i+j) for i=1:2, j=1:2), [1 3; 2 4])
2×2 Array{Tuple{Float64,Int64},2}:
 (0.5, 1)       (0.333333, 3)
 (0.333333, 2)  (0.25, 4)
```

Ranges in generators and comprehensions can depend on previous ranges by writing multiple `for` keywords:
> ジェネレータと補間の範囲は、複数の「for」キーワードを記述することによって、以前の範囲に依存することができます。

```jldoctest
julia> [(i,j) for i=1:3 for j=1:i]
6-element Array{Tuple{Int64,Int64},1}:
 (1, 1)
 (2, 1)
 (2, 2)
 (3, 1)
 (3, 2)
 (3, 3)
```

In such cases, the result is always 1-d.
> そのような場合、結果は常に1-dです。

Generated values can be filtered using the `if` keyword:
> 生成された値は `if` キーワードを使ってフィルタリングできます：

```jldoctest
julia> [(i,j) for i=1:3 for j=1:i if i+j == 4]
2-element Array{Tuple{Int64,Int64},1}:
 (2, 2)
 (3, 1)
```

## [Indexing](@ id man-array-indexing)
> ### [Indexing](@id man-array-indexing)

The general syntax for indexing into an n-dimensional array A is:
> n次元配列Aへの索引付けの一般的な構文は次のとおりです。

```
X = A[I_1, I_2, ..., I_n]
```

where each `I_k` may be a scalar integer, an array of integers, or any other [supported index](@ref man-supported-index-types). 
> それぞれの `I_k`はスカラ整数、整数の配列、または他の[supported index](@ref man-supported-index-types)です。
This includes [`Colon`](@ref) (`:`) to select all indices within the entire dimension, ranges of the form `a:c` or `a:b:c` to select contiguous or strided subsections, and arrays of booleans to select elements at their `true` indices.
This includes [`Colon`] to select all indices within the entire dimension, ranges of the form `a:c` or `a:b:c` to select contiguous or strided subsections, and arrays of booleans to select elements at their `true` indices.
> これには、次元全体の中のすべてのインデックス、 `a：c`または` a：b：c`の範囲を選択して連続した部分またはストライドされた部分と真のインデックスで要素を選択するブール値の配列、を選択する[`Colon`]が含まれます。

If all the indices are scalars, then the result `X` is a single element from the array `A`. 
> すべてのインデックスがスカラーである場合、結果の「X」は配列「A」からの単一の要素です。
Otherwise, `X` is an array with the same number of dimensions as the sum of the dimensionalities of all the indices.
> それ以外の場合、 `X`はすべてのインデックスの次元数の合計と同じ次元数の配列です。

If all indices are vectors, for example, then the shape of `X` would be `(length(I_1), length(I_2), ..., length(I_n))`, with location `(i_1, i_2, ..., i_n)` of `X` containing the value `A[I_1[i_1], I_2[i_2], ..., I_n[i_n]]`.
> `I_1`が2次元行列に変更された場合、` X`は `size(I_1、1)、size(I_1、2)、length(I_2)、` ...、長さ(I_n)) `である。行列は次元を追加します。
If `I_1` is changed to a two-dimensional matrix, then `X` becomes an `n+1`-dimensional array of shape `(size(I_1, 1), size(I_1, 2), length(I_2), ..., length(I_n))`. The matrix adds a dimension.
> たとえば、すべてのインデックスがベクトルである場合、 `X 'の形状は`(i_1、length、length)であり、位置は `(i_1、i_2、。 ..、I_n [i_n]] 'を含む `X'の値(例えば、。
The location `(i_1, i_2, i_3, ..., i_{n+1})` contains the value at `A[I_1[i_1, i_2], I_2[i_3], ..., I_n[i_{n+1}]]`.
`{i_1 [i_1、i_2]、I_2 [i_3]、...、I_n [i_ {n(i_1、i_2、 +1}]]。
> X = getindex(A, I_1, I_2, ..., I_n)
All dimensions indexed with scalars are dropped. 
> スカラで索引付けされたすべてのディメンションは削除されます。
For example, the result of `A[2, I, 3]` is an array with size `size(I)`. 
> たとえば、 `A [2、I、3]`の結果は `size(I)`というサイズの配列です。
Its `i`th element is populated by `A[2, I[i], 3]`.
> `i`番目の要素は`A [2、I [i]、3]`によって読み込まれます。

As a special part of this syntax, the `end` keyword may be used to represent the last index of each dimension within the indexing brackets, as determined by the size of the innermost array being indexed. 
> この構文の特別な部分として、`end` キーワードを使用して、索引付けされる最も内側の配列のサイズによって決定されるように、索引括弧内の各次元の最後の索引を表すことができます。
Indexing syntax without the `end` keyword is equivalent to a call to `getindex`:
> `end` キーワードのないシンタックスの索引付けは、`getindex` の呼び出しと同じです：

Example:

```jldoctest
julia> x = reshape(1:16, 4, 4)
4×4 Base.ReshapedArray{Int64,2,UnitRange{Int64},Tuple{}}:
 1  5   9  13
 2  6  10  14
 3  7  11  15
 4  8  12  16

julia> x[2:3, 2:end-1]
2×2 Array{Int64,2}:
 6  10
 7  11

julia> x[1, [2 3; 4 1]]
2×2 Array{Int64,2}:
  5  9
 13  1
```

Empty ranges of the form `n:n-1` are sometimes used to indicate the inter-index location between `n-1` and `n`. 
> `n：n-1` という形式の空の範囲は、`n-1` と `n` の間のインデックス間の位置を示すために時々使用されます。
For example, the [`searchsorted()`](@ref) function uses this convention to indicate the insertion point of a value not found in a sorted array:
> たとえば、[`searchsorted()`](@ref) 関数は、ソートされた配列に見つからない値の挿入ポイントを示すためにこの規約を使います：


```jldoctest
julia> a = [1,2,5,6,7];

julia> searchsorted(a, 3)
3:2
```

## Assignment
> ## 割り当て

The general syntax for assigning values in an n-dimensional array A is:
> n次元配列Aの値を代入する一般的な構文は次のとおりです。

```
A[I_1, I_2, ..., I_n] = X
```

where each `I_k` may be a scalar integer, an array of integers, or any other [supported index](@ref man-supported-index-types). 
> それぞれの `I_k` はスカラ整数、整数の配列、または他の [サポートされるインデックス](@ref man-supported-index-types) です。
#This includes [`Colon`](@ref) (`:`) to select all indices within the entire dimension, ranges of the form `a:c` or `a:b:c` to select contiguous or strided subsections, and arrays of booleans to select elements at their `true` indices.
> これには、次元全体の中のすべてのインデックスを選択するための [`Colon`](@ref)(`:`) 、連続したサブセクションまたはストライドサブセクションを選択するための `a:c` または `a:b:c` 論理値の要素を選択するブール値の配列

#If `X` is an array, it must have the same number of elements as the product of the lengths of the indices: `prod(length(I_1), length(I_2), ..., length(I_n))`. 
> `X`が配列の場合は、` prod(length(I_1)、length(I_2)、...、length(I_n)) `のインデックスの長さの積と同じ数の要素を持たなければなりません。
#The value in location `I_1[i_1], I_2[i_2], ..., I_n[i_n]` of `A` is overwritten with the value `X[i_1, i_2, ..., i_n]`. 
> 位置`I_1[i_1], I_2[i_2], ..., I_n[i_n]`の値`A`は`X[i_1, i_2, ..., i_n]`の値で上書きされます。
If `X` is not an array, its value is written to all referenced locations of `A`.
> `X`が配列でない場合、その値は` A`のすべての参照先に書き込まれます。

Just as in [Indexing](@ref man-array-indexing), the `end` keyword may be used to represent the last index of each dimension within the indexing brackets, as determined by the size of the array being assigned into. 
> [Indexing](@ref man-array-indexing)と同じように、 `end`キーワードは、割り付けられている配列のサイズによって決まるように、インデックス括弧内の各次元の最後のインデックスを表すために使用されます。
Indexed assignment syntax without the `end` keyword is equivalent to a call to [`setindex!()`](@ref):
> `end`キーワードのないインデックス付きの代入構文は、[`setindex!() `](@ref)の呼び出しと同じです：


```
setindex!(A, X, I_1, I_2, ..., I_n)
```

Example:
> 例:

```jldoctest
julia> x = collect(reshape(1:9, 3, 3))
3×3 Array{Int64,2}:
 1  4  7
 2  5  8
 3  6  9

julia> x[1:2, 2:3] = -1
-1

julia> x
3×3 Array{Int64,2}:
 1  -1  -1
 2  -1  -1
 3   6   9
```

## [Supported index types](@id man-supported-index-types)
> ### [サポートされているインデックスタイプ](@ idのman-supported-index-types)

In the expression `A[I_1, I_2, ..., I_n]`, each `I_k` may be a scalar index, an array of scalar indices, or an object that represents an array of scalar indices and can be converted to such by [`to_indices`](@ref):
> `A[I_1, I_2, ..., I_n]`という表現では、各`I_k`はスカラーインデックス、スカラインデックスの配列、またはスカラインデックスの配列を表すオブジェクトであり、[`to_indices`](@ref)で変換することができます。

1. A scalar index. By default this includes:
> 1. スカラーインデックス。デフォルトでは以下が含まれます：
   > * Non-boolean integers
   > * ブール値でない整数
   > * `CartesianIndex{N}`s, which behave like an `N`-tuple of integers spanning multiple dimensions (see below for more details)
   > * `CartesianIndex {N}`は、複数次元にまたがる整数の `N`タプルのように振る舞います(詳細は以下を参照してください)
2. An array of scalar indices. This includes:
> 2. スカラーインデックスの配列。これも：
   > * Vectors and multidimensional arrays of integers
   > * ベクトルと整数の多次元配列
   > * Empty arrays like `[]`, which select no elements
   > * 要素を選択しない `[]`のような空の配列
   > * `Range`s of the form `a:c` or `a:b:c`, which select contiguous or strided subsections from `a` to `c` (inclusive)
   > * `a：c`または` a：b：c`形式の `` Range`sは `a`から` c`までの連続したサブセクションまたはストライドサブセクションを選択します。
   > * Any custom array of scalar indices that is a subtype of `AbstractArray`
   > * `AbstractArray`のサブタイプであるスカラーインデックスのカスタム配列
   > * Arrays of `CartesianIndex{N}` (see below for more details)
   > * `CartesianIndex {N}`の配列(詳細は下記参照)
3. An object that represents an array of scalar indices and can be converted to such by [`to_indices`](@ref).
> 3. スカラーインデックスの配列を表し、[to_indices`](@ref)によってそれに変換できるオブジェクト。
By default this includes:
> デフォルトでは以下が含まれます：
    > * [`Colon()`](@ref) (`:`), which represents all indices within an entire dimension or across the entire array
   > * [`Colon()`](@ref)(`:`) 、ディメンション全体または配列全体にわたるすべてのインデックスを表します
   >  * Arrays of booleans, which select elements at their `true` indices (see below for more details)
   > * 真のインデックスで要素を選択するブール値の配列(詳細は下記参照)

### Cartesian indices
> ### デカルトインデックス

The special `CartesianIndex{N}` object represents a scalar index that behaves like an `N`-tuple of integers spanning multiple dimensions.
> 特殊な `CartesianIndex{N}`オブジェクトは、複数の次元にまたがる整数の `N` タプルのように振る舞うスカラーインデックスを表します。
For example:
> 次の例:

```jldoctest cartesianindex
julia> A = reshape(1:32, 4, 4, 2);

julia> A[3, 2, 1]
7

julia> A[CartesianIndex(3, 2, 1)] == A[3, 2, 1] == 7
true
```

# Considered alone, this may seem relatively trivial; `CartesianIndex` simply gathers multiple integers together into one object that represents a single multidimensional index.
> 単独で考えてみると、これは比較的些細なように見えるかもしれません。`CartesianIndex` は、複数の整数をひとつのオブジェクトに集めて、単一のオブジェクト 多次元指数。
When combined with other indexing forms and iterators that yield `CartesianIndex`es, however, this can lead directly to very elegant and efficient code.
> しかし、`CartesianIndex` を生成する他のインデクシングフォームやイテレータと組み合わせると、非常にエレガントで効率的なコードに直接つながります。
See [Iteration](@ref) below, and for some more advanced examples, see [this blog post on multidimensional algorithms and iteration](https://julialang.org/blog/2016/02/iteration). 
> 以下の [反復](@ref) を参照してください。いくつかの高度な例については、[多次元アルゴリズムと反復に関するこのブログ記事](https://julialang.org/blog/2016/02/iteration)を参照してください。
Arrays of `CartesianIndex{N}` are also supported. They represent a collection of scalar indices that each span `N` dimensions, enabling a form of indexing
> `CartesianIndex{N}` の配列もサポートされています。 これらは、それぞれがN次元に及ぶスカラーインデックスの集合を表し、インデックスの形式を可能にします
that is sometimes referred to as pointwise indexing.
> ポイントワイズインデックス付けと呼ばれることもあります。
For example, it enables accessing the diagonal elements from the first "page" of `A` from above:
> 例えば、上から `A` の最初の「ページ」から対角要素にアクセスすることができます。


```jldoctest cartesianindex
julia> page = A[:,:,1]
4×4 Array{Int64,2}:
 1  5   9  13
 2  6  10  14
 3  7  11  15
 4  8  12  16

julia> page[[CartesianIndex(1,1),
             CartesianIndex(2,2),
             CartesianIndex(3,3),
             CartesianIndex(4,4)]]
4-element Array{Int64,1}:
  1
  6
 11
 16
```

This can be expressed much more simply with [dot broadcasting](@ref man-vectorized) and by combining it with a normal integer index (instead of extracting the first `page` from `A` as a separate step).
> これは [dot broadcasting](@ref man-vectorized) や(`A`からの最初の `page`を別のステップとして抽出するのではなく)通常の整数インデックスと組み合わせることでもっと簡単に表現できます。
It can even be combined with a `:` to extract both diagonals from the two pages at the same time:
> 同時に2つのページから両方の対角線を抽出するために `:`と組み合わせることもできます：

```jldoctest cartesianindex
julia> A[CartesianIndex.(indices(A, 1), indices(A, 2)), 1]
4-element Array{Int64,1}:
  1
  6
 11
 16

julia> A[CartesianIndex.(indices(A, 1), indices(A, 2)), :]
4×2 Array{Int64,2}:
  1  17
  6  22
 11  27
 16  32
```

!!! warning
!!! 注意!!

   `CartesianIndex` and arrays of `CartesianIndex` are not compatible with the `end` keyword to represent the last index of a dimension.
   > `CartesianIndex`と` CartesianIndex`の配列は、次元の最後のインデックスを表す `end`キーワードと互換性がありません。
   Do not use `end` in indexing expressions that may contain either `CartesianIndex` or arrays thereof.
   > `CartesianIndex`かその配列のいずれかを含むインデックス式で` end`を使用しないでください。

### Logical indexing
> ### 論理インデックス

Often referred to as logical indexing or indexing with a logical mask, indexing by a boolean array selects elements at the indices where its values are `true`.
> 論理的なインデックス付けまたは論理的なマスクによるインデックス付けと呼ばれることが多いが、ブール配列によるインデックス付けは、その値が「真」であるインデックスの要素を選択する。
Indexing by a boolean vector `B` is effectively the same as indexing by the vector of integers that is returned by [`find(B)`](@ref). Similarly, indexing by a `N`-dimensional boolean array is effectively the same as indexing by the vector of `CartesianIndex{N}`s where its values are `true`. 
> ブール値ベクトル `B` によるインデックス付けは、 [find(B)`](@ref) によって返される整数ベクトルによるインデックス付けと事実上同じです。 同様に、 `N` 次元ブール配列によるインデックス付けは、その値が `true` である `CartesianIndex{N}` のベクトルによるインデックス付けと事実上同じです。
A logical index must be a vector of the same length as the dimension it indexes into, or it must be the only index provided and match the size and dimensionality of the array it indexes into. 
> 論理索引は、索引付けする次元と同じ長さのベクトルでなければならず、索引付けされる唯一の索引でなければならず、索引付けされる配列のサイズと次元に一致しなければなりません。
It is generally more efficient to use boolean arrays as indices directly instead of first calling [`find()`](@ref).
> 最初は [`find()`](@ref) を呼び出すのではなく、ブール値の配列を直接インデックスとして使う方が効率的です。


```jldoctest
julia> x = reshape(1:16, 4, 4)
4×4 Base.ReshapedArray{Int64,2,UnitRange{Int64},Tuple{}}:
 1  5   9  13
 2  6  10  14
 3  7  11  15
 4  8  12  16

julia> x[[false, true, true, false], :]
2×4 Array{Int64,2}:
 2  6  10  14
 3  7  11  15

julia> mask = map(ispow2, x)
4×4 Array{Bool,2}:
  true  false  false  false
  true  false  false  false
 false  false  false  false
  true   true  false   true

julia> x[mask]
5-element Array{Int64,1}:
  1
  2
  4
  8
 16
```

## Iteration
> ### 反復

The recommended ways to iterate over a whole array are
> 配列全体を反復処理するための推奨される方法は次のとおりです。

```julia
for a in A
    # Do something with the element a
end

for i in eachindex(A)
    # Do something with i and/or A[i]
end
```

The first construct is used when you need the value, but not index, of each element. 
> 最初の構造体は、各要素の値ではなくインデックスを必要とするときに使用されます。
In the second construct, `i` will be an `Int` if `A` is an array type with fast linear indexing; otherwise, it will be a `CartesianIndex`:
> 2番目の構文では、`A` が高速リニアインデックスを持つ配列型であれば `i` が `Int` になります。 さもなければ `CartesianIndex`でしょう：

```jldoctest
julia> A = rand(4,3);

julia> B = view(A, 1:3, 2:3);

julia> for i in eachindex(B)
           @show i
       end
i = CartesianIndex{2}((1, 1))
i = CartesianIndex{2}((2, 1))
i = CartesianIndex{2}((3, 1))
i = CartesianIndex{2}((1, 2))
i = CartesianIndex{2}((2, 2))
i = CartesianIndex{2}((3, 2))
```

In contrast with `for i = 1:length(A)`, iterating with `eachindex` provides an efficient way to iterate over any array type.
> `for i = 1：length(A)` とは対照的に、`eachindex` を使って反復することで、どの配列型でも効率的に反復処理ができます。

## Array traits
> ### 配列の特徴

If you write a custom [`AbstractArray`](@ref) type, you can specify that it has fast linear indexing using
> カスタム [`AbstractArray`](@ref) 型を記述すると、高速リニアインデックスがあることを指定することができます

```julia
Base.IndexStyle(::Type{<:MyArray}) = IndexLinear()
```

This setting will cause `eachindex` iteration over a `MyArray` to use integers.
> この設定は、 `MyArray` に対して `eachindex` 反復に整数を使用させます。
If you don't specify this trait, the default value `IndexCartesian()` is used.
> この特性を指定しないと、デフォルト値 `IndexCartesian()` が使用されます。

## Array and Vectorized Operators and Functions
> ## 配列とベクトル化された演算子と関数

The following operators are supported for arrays:
> 配列では次の演算子がサポートされています。

1. Unary arithmetic -- `-`, `+`
2. Binary arithmetic -- `-`, `+`, `*`, `/`, `\`, `^`
3. Comparison -- `==`, `!=`, `≈` ([`isapprox`](@ref)), `≉`

Most of the binary arithmetic operators listed above also operate elementwise when one argument is scalar: `-`, `+`, and `*` when either argument is scalar,
> 上記のバイナリ算術演算子の大部分は、いずれかの引数がスカラーの場合に、引数がスカラーの場合にはelementwiseで動作します： `-`、` + `、` * `
and `/` and `\` when the denominator is scalar.
> 分母がスカラーのときは `/` と `\` を使います。
For example, `[1, 2] + 3 == [4, 5]` and `[6, 4] / 2 == [3, 2]`.
> 例えば, `[1, 2] + 3 == [4, 5]` and `[6, 4] / 2 == [3, 2]`.

Additionally, to enable convenient vectorization of mathematical and other operations, Julia [provides the dot syntax](@ref man-vectorized) `f.(args...)`, e.g. `sin.(x)` or `min.(x,y)`, for elementwise operations over arrays or mixtures of arrays and scalars (a [Broadcasting](@ref) operation); these have the additional advantage of "fusing" into a single loop when combined with other dot calls, e.g. `sin.(cos.(x))`.
> さらに、数学的演算や他の演算の便利なベクトル化を可能にするために、Julia [ドット構文を提供する](@ref man-vectorized) `f(args ...)` などがあります。 ( [Broadcasting](@ref) 演算)、または配列とスカラの混合物に対する要素ワイルド演算のために、 `sin(x)` または `min(x、y)` これらは、他のドット呼び出しと組み合わせると、単一のループに「融合する」という付加的な利点を有する。 `sin(cos(x))` である。

Also, *every* binary operator supports a [dot version](@ref man-dot-operators) that can be applied to arrays (and combinations of arrays and scalars) in such
> また、すべての* 2項演算子は、(配列とスカラーの組み合わせ)配列に適用できる [dot version](@ref man-dot-operators) をサポートしています
[fused broadcasting operations](@ref man-vectorized), e.g. `z .== sin.(x .* y)`.
> [fused broadcasting operations](@ref man-vectorized) 。 `z .== sin.(x .* y)`である。

Note that comparisons such as `==` operate on whole arrays, giving a single boolean answer. 
> `==` のような比較は配列全体に対して作用し、単一のブール型の答えを与えることに注意してください。
Use dot operators like `.==` for elementwise comparisons. (For comparison operations like `<`, *only* the elementwise `.<` version is applicable to arrays.)
> 要素比較のために `.==` のようなドット演算子を使います。 (`<`、*のみのような* 比較演算では、要素ごとの `.<` バージョンは配列にも適用可能です)。

Also notice the difference between `max.(a,b)`, which `broadcast`s [`max()`](@ref) elementwise over `a` and `b`, and `maximum(a)`, which finds the largest value within `a`. The same relationship holds for `min.(a,b)` and `minimum(a)`.
> `a`と`b`の要素ごとの `max`と `max(a)`の `max`と `max`(@ref)の違いにも注目してください。 `a`の中で最大値を見つけます。 `min(a、b)`と `minimum(a)`についても同じ関係が成り立ちます。

## Broadcasting
> ## ブロードキャスト

It is sometimes useful to perform element-by-element binary operations on arrays of different sizes, such as adding a vector to each column of a matrix. 
> 行列の各列にベクトルを追加するなど、サイズの異なる配列に対して要素バイナリ演算を実行すると便利なことがあります。
An inefficient way to do this would be to replicate the vector to the size of the matrix:
> これを行う非効率的な方法は、ベクトルを行列のサイズに複製することです。

```julia-repl
julia> a = rand(2,1); A = rand(2,3);

julia> repmat(a,1,3)+A
2×3 Array{Float64,2}:
 1.20813  1.82068  1.25387
 1.56851  1.86401  1.67846
```

This is wasteful when dimensions get large, so Julia offers [`broadcast()`](@ref), which expands singleton dimensions in array arguments to match the corresponding dimension in the other array without using extra memory, and applies the given function elementwise:
> これは、ディメンションが大きくなると無駄なので、Juliaは [`broadcast()`](@ref) を提供します。 これは、余分なメモリを使用せずに、配列引数のシングルトンディメンションを他の配列の対応するディメンションと一致するように展開し、指定された関数を要素ごとに適用します。

```julia-repl
julia> broadcast(+, a, A)
2×3 Array{Float64,2}:
 1.20813  1.82068  1.25387
 1.56851  1.86401  1.67846

julia> b = rand(1,2)
1×2 Array{Float64,2}:
 0.867535  0.00457906

julia> broadcast(+, a, b)
2×2 Array{Float64,2}:
 1.71056  0.847604
 1.73659  0.873631
```

[Dotted operators](@ref man-dot-operators) such as `.+` and `.*` are equivalent to `broadcast` calls (except that they fuse, as described below).
> `.+` や `.*` のような [dot演算子](@ref man-dot-operators) は、`broadcast` 呼び出しと同じです(以下で説明するように、それらが融合する点を除きます)。
There is also a [`broadcast!()`](@ref) function to specify an explicit destination (which can also be accessed in a fusing fashion by `.=` assignment), and functions [`broadcast_getindex()`](@ref) and [`broadcast_setindex!()`](@ref) that broadcast the indices before indexing. 
> 明示的なブロードキャストを指定する [`broadcast!()`](@ref) 関数もあります (`.=` 代入で融合する方法でもアクセスできます)。関数 [`broadcast_getindex()`]@ref ) と [`broadcast_setindex!()`](@ref) は、インデックス作成の前にインデックスをブロードキャストします。
Moreover, `f.(args...)` is equivalent to `broadcast(f, args...)`, providing a convenient syntax to broadcast any function ([dot syntax](@ref man-vectorized)). 
> さらに、 `f.(args...)`は `broadcast(f, args...)`と同等で、任意の関数([dot syntax](@ref man-vectorized))を放送する便利な構文を提供します。
Nested "dot calls" `f.(...)` (including calls to `.+` etcetera) [automatically fuse](@ref man-dot-operators) into a single `broadcast` call.
> ネストされた "ドット呼出し" `f.(...)`(`.+`etceterへの呼出しを含む)[automatically fuse](@ref man-dot-operators)

Additionally, [`broadcast()`](@ref) is not limited to arrays (see the function documentation), it also handles tuples and treats any argument that is not an array, tuple or `Ref` (except for `Ptr`) as a "scalar".
> さらに、[`broadcast()`](@ref) は配列に限定されず(関数のドキュメントを参照)、タプルも扱い、配列、タプル、または `Ref`以外の引数を扱います(` Ptr` )を「スカラー」として定義します。

```jldoctest
julia> convert.(Float32, [1, 2])
2-element Array{Float32,1}:
 1.0
 2.0

julia> ceil.((UInt8,), [1.2 3.4; 5.6 6.7])
2×2 Array{UInt8,2}:
 0x02  0x04
 0x06  0x07

julia> string.(1:3, ". ", ["First", "Second", "Third"])
3-element Array{String,1}:
 "1. First"
 "2. Second"
 "3. Third"
```

## Implementation
> ## 実装

The base array type in Julia is the abstract type [`AbstractArray{T,N}`](@ref). 
> Juliaの基本配列型は抽象型 [`AbstractArray{T,N}`](@ref) です。
It is parametrized by the number of dimensions `N` and the element type `T`. 
> これは次元数Nと要素タイプTでパラメータ化されます。 
[`AbstractVector`](@ref) and [`AbstractMatrix`](@ref) are aliases for the 1-d and 2-d cases. 
> [AbstractVector`](@ref) と[`AbstractMatrix`](@ref) は、1次元と2次元の場合のエイリアスです。
Operations on `AbstractArray` objects are defined using higher level operators and functions, in a way that is independent of the underlying storage. 
> `AbstractArray` オブジェクトに対する操作は、基礎となる記憶装置から独立したやり方で、より高いレベルの演算子と関数を使用して定義されます。
These operations generally work correctly as a fallback for any specific array implementation.
> これらの操作は、通常、特定の配列実装のフォールバックとして正しく機能します。

The `AbstractArray` type includes anything vaguely array-like, and implementations of it might be quite different from conventional arrays. 
> `AbstractArray` 型は漠然とアレイ的なものを含み、その実装は従来の配列とはかなり異なるかもしれません。
For example, elements might be computed on request rather than stored. 
> 例えば、要素は格納されるのではなく要求に応じて計算されることがあります。
However, any concrete `AbstractArray{T,N}` type should generally implement at least [`size(A)`](@ref) (returning an `Int` tuple), [`getindex(A,i)`](@ref) and [`getindex(A,i1,...,iN)`](@ref getindex); mutable arrays should also implement [`setindex!()`](@ref). 
> しかし、具体的な `AbstractArray {T、N}`型は、一般に、少なくとも[`size(A)`](@ref)( `Int`タプルを返す)、[` getindex(A、i) `] @ref)と[`getindex(A、i1、...、iN)`](@ref getindex);可変配列は[`setindex！()`](@ref)も実装しなければなりません。
It is recommended that these operations have nearly constant time complexity, or technically Õ(1) complexity, as otherwise some array functions may be unexpectedly slow. 
> これらの操作は、時間の複雑さがほぼ一定であるか、技術的には複雑であることが推奨されます。そうしないと、一部の配列関数が予期せず遅くなることがあります。
Concrete types should also typically provide a [`similar(A,T=eltype(A),dims=size(A))`](@ref) method, which is used to allocate a similar array for [`copy()`](@ref) and other out-of-place operations. 
> コンクリート型は、通常、[`copy()`のために同様の配列を割り当てるために使用される[`類似の(A、T = eltype(A)、dims = size(A))`](@ref) ](@ref)とその他のアウトオブプレース操作が含まれます。
No matter how an `AbstractArray{T,N}` is represented internally, `T` is the type of object returned by *integer* indexing (`A[1, ..., 1]`, when `A` is not empty) and `N` should be the length of the tuple returned by [`size()`](@ref).
> `AbstractArray {T、N}`が内部的にどのように表現されていても、`T`は整数の索引付けによって返されるオブジェクトの型です(`A [1、...、1]空)`、 `N`は[size()`](@ref) によって返されるタプルの長さでなければなりません。

`DenseArray` is an abstract subtype of `AbstractArray` intended to include all arrays that are laid out at regular offsets in memory, and which can therefore be passed to external C and Fortran functions expecting this memory layout. 
> `DenseArray`は` AbstractArray`の抽象サブタイプであり、通常のオフセットでメモリに配置されたすべての配列を含み、このメモリレイアウトを期待する外部のCおよびFortran関数に渡すことができます。
Subtypes should provide a method [`stride(A,k)`](@ref) that returns the "stride" of dimension `k`: increasing the index of dimension `k` by `1` should increase the index `i` of [`getindex(A,i)`](@ref) by [`stride(A,k)`](@ref). 
> サブタイプは、次元「k」の「ストライド」を返すメソッド[`stride(A、k)`](@ref)を提供する必要があります：次元 `k`のインデックスを `1`で増加させるとインデックス `i` [`stind(A、k)`](@ref)による[`getindex(A、i)`](@ref)
If a pointer conversion method [`Base.unsafe_convert(Ptr{T}, A)`](@ref) is provided, the memory layout should correspond in the same way to these strides.
> ポインタ変換方法[`Base.unsafe_convert(Ptr {T}、A)`](@ref) が提供されている場合、メモリレイアウトはこれらのストライドと同じ方法で対応する必要があります。

The [`Array`](@ref) type is a specific instance of `DenseArray` where elements are stored in column-major order (see additional notes in [Performance Tips](@ref man-performance-tips)). 
> [`Array`](@ref)型は`DenseArray`の特定のインスタンスであり、要素は列メジャー順に格納されます([Performance Tips](@ref man-performance-tips)の補足を参照してください)。
[`Vector`](@ref) and [`Matrix`](@ref) are aliases for the 1-d and 2-d cases. 
> [`Vector`](@ref) と [`Matrix`](@ref) は1-dと2-dの場合のエイリアスです。
Specific operations such as scalar indexing, assignment, and a few other basic storage-specific operations are all that have to be implemented for [`Array`](@ref), so that the rest of the array library can be implemented in a generic manner.
> スカラーインデックス、代入、その他いくつかの基本的なストレージ固有の操作などの特定の操作はすべて [`Array`](@ref) に対して実装する必要があります。そのため、配列ライブラリの残りの部分は汎用方法。

`SubArray` is a specialization of `AbstractArray` that performs indexing by reference rather than by copying. 
> `SubArray` は `AbstractArray` の特殊化で、コピーするのではなく参照でインデックスを実行します。
A `SubArray` is created with the [`view()`](@ref) function, which is called the same way as [`getindex()`](@ref) (with an array and a series of index arguments). 
> [`view()`](@ref) の結果は、データがそのまま残っていることを除いて [`getindex()`](@ref) の結果と同じに見えます。
#The result of [`view()`](@ref) looks the same as the result of [`getindex()`](@ref), except the data is left in place. 
> `` getindex() `](@ref)(配列と一連のインデックス引数を持つ)と同じように呼ばれる[` view() `](@ref)関数で` SubArray`を作成します。 。
> [`view()`](@ref)は入力インデックスベクトルを `SubArray`オブジェクトに格納します。これは後で元の配列を間接的に索引付けするために使用できます。
[`view()`](@ref) stores the input index vectors in a `SubArray` object, which can later be used to index the original array indirectly.  
#By putting the [`@views`](@ref) macro in front of an expression or block of code, any `array[...]` slice in that expression will be converted to create a `SubArray` view instead.
> 式またはブロックのコードの前に[`@views`](@ref)マクロを置くことによって、その式の` array [...] `スライスは` SubArray`ビューを生成するように変換されます。

#`StridedVector` and `StridedMatrix` are convenient aliases defined to make it possible for Julia to call a wider range of BLAS and LAPACK functions by passing them either [`Array`](@ref) or `SubArray` objects, and thus saving inefficiencies from memory allocation and copying.
> `StridedVector`と` StridedMatrix`はJuliaが `` Array``(@ref)または `SubArray`オブジェクトを渡すことでより広い範囲のBLAS関数とLAPACK関数を呼び出すことができるように定義された便利なエイリアスなので、非効率メモリの割り当てとコピーから。

The following example computes the QR decomposition of a small section of a larger array, without creating any temporaries, and by calling the appropriate LAPACK function with the right leading dimension size and stride parameters.
> 次の例では、より大きな配列の小さなセクションのQR分解を計算します。一時的なものは作成せずに、適切なLAPACK関数を呼び出し、サイズおよびストライドパラメータをリードします。


```julia-repl
julia> a = rand(10,10)
10×10 Array{Float64,2}:
 0.561255   0.226678   0.203391  0.308912   …  0.750307  0.235023   0.217964
 0.718915   0.537192   0.556946  0.996234      0.666232  0.509423   0.660788
 0.493501   0.0565622  0.118392  0.493498      0.262048  0.940693   0.252965
 0.0470779  0.736979   0.264822  0.228787      0.161441  0.897023   0.567641
 0.343935   0.32327    0.795673  0.452242      0.468819  0.628507   0.511528
 0.935597   0.991511   0.571297  0.74485    …  0.84589   0.178834   0.284413
 0.160706   0.672252   0.133158  0.65554       0.371826  0.770628   0.0531208
 0.306617   0.836126   0.301198  0.0224702     0.39344   0.0370205  0.536062
 0.890947   0.168877   0.32002   0.486136      0.096078  0.172048   0.77672
 0.507762   0.573567   0.220124  0.165816      0.211049  0.433277   0.539476

julia> b = view(a, 2:2:8,2:2:4)
4×2 SubArray{Float64,2,Array{Float64,2},Tuple{StepRange{Int64,Int64},StepRange{Int64,Int64}},false}:
 0.537192  0.996234
 0.736979  0.228787
 0.991511  0.74485
 0.836126  0.0224702

julia> (q,r) = qr(b);

julia> q
4×2 Array{Float64,2}:
 -0.338809   0.78934
 -0.464815  -0.230274
 -0.625349   0.194538
 -0.527347  -0.534856

julia> r
2×2 Array{Float64,2}:
 -1.58553  -0.921517
  0.0       0.866567
```

## Sparse Vectors and Matrices
> ## 疎ベクトルと疎配列

Julia has built-in support for sparse vectors and [sparse matrices](https://en.wikipedia.org/wiki/Sparse_matrix). 
> Juliaには、疎ベクトルと[sparse matrices](https://en.wikipedia.org/wiki/Sparse_matrix)が組み込まれています。
Sparse arrays are arrays that contain enough zeros that storing them in a special data structure leads to savings in space and execution time, compared to dense arrays.
> 疎配列は、特殊なデータ構造に格納するのに十分な零点を含む配列であり、高密度配列に比べて領域と実行時間の節約につながります。

## [Compressed Sparse Column (CSC) Sparse Matrix Storage](@id man-csc)
> ## [圧縮列格納方式(CSC)疎行列`ストレージ](@ id man-csc)

In Julia, sparse matrices are stored in the [Compressed Sparse Column (CSC) format](https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_column_.28CSC_or_CCS.29).
> Juliaでは、疎行列は[Compressed Sparse Column(CSC)形式]に格納されます(https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_column_.28CSC_or_CCS.29)。
Julia sparse matrices have the type [`SparseMatrixCSC{Tv,Ti}`](@ref), where `Tv` is the type of the stored values, and `Ti` is the integer type for storing column pointers and row indices. 
> Juliaの疎行列は、 `` SparseMatrixCSC {Tv、Ti} `(@ref)型を持ち、` Tv`は格納された値の型、 `Ti`は列ポインタと行インデックスを格納する整数型です。
The internal representation of `SparseMatrixCSC` is as follows:
> `SparseMatrixCSC`の内部表現は以下の通りです：

```julia
struct SparseMatrixCSC{Tv,Ti<:Integer} <: AbstractSparseMatrix{Tv,Ti}
    m::Int                  # Number of rows
    n::Int                  # Number of columns
    colptr::Vector{Ti}      # Column i is in colptr[i]:(colptr[i+1]-1)
    rowval::Vector{Ti}      # Row indices of stored values
    nzval::Vector{Tv}       # Stored values, typically nonzeros
end
```

The compressed sparse column storage makes it easy and quick to access the elements in the column of a sparse matrix, whereas accessing the sparse matrix by rows is considerably slower. 
> 圧縮された疎列の記憶域を使用すると、疎行列の列の要素に簡単かつ迅速にアクセスできますが、行単位で疎行列にアクセスするのはかなり遅くなります。
Operations such as insertion of previously unstored entries one at a time in the CSC structure tend to be slow. 
> CSC 構造内に以前に記憶されていないエントリを一度に1つ挿入するなどの操作は遅くなる傾向があります。
This is because all elements of the sparse matrix that are beyond the point of insertion have to be moved one place over.
> これは、挿入位置を超えた疎マトリックスのすべての要素を1か所上に移動する必要があるためです。

All operations on sparse matrices are carefully implemented to exploit the CSC data structure for performance, and to avoid expensive operations.
> 疎行列のすべての演算は、パフォーマンスのために CSC データ構造を利用し、高価な演算を避けるために慎重に実装されています。

If you have data in CSC format from a different application or library, and wish to import it in Julia, make sure that you use 1-based indexing. 
> 異なるアプリケーションまたはライブラリの CSC 形式のデータがあり、Juliaでインポートする場合は、必ず1ベースのインデックスを使用してください。
The row indices in every column need to be sorted. 
> すべての列の行インデックスをソートする必要があります。
If your `SparseMatrixCSC` object contains unsorted row indices, one quick way to sort them is by doing a double transpose.
> `SparseMatrixCSC` オブジェクトにソートされていない行インデックスが含まれている場合、それらをソートする素早い方法の1つは、ダブルトランスポーズです。

In some applications, it is convenient to store explicit zero values in a `SparseMatrixCSC`. 
> アプリケーションによっては、明示的なゼロ値を `SparseMatrixCSC` に格納すると便利です。
These *are* accepted by functions in `Base` (but there is no guarantee that they will be preserved in mutating operations). 
> これらは `Base` の関数によって *受け入れられます* (しかし、それらが変異操作で保持されるという保証はありません)。
Such explicitly stored zeros are treated as structural nonzeros by many routines. 
> このように明示的に格納されたゼロは、多くのルーチンによって構造的非ゼロとして扱われます。
The [`nnz()`](@ref) function returns the number of elements explicitly stored in the sparse data structure, including structural nonzeros. 
> [`nnz()`](@ref) 関数は、構造的な非ゼロを含む疎なデータ構造体に明示的に格納された要素の数を返します。
In order to count the exact number of numerical nonzeros, use [`countnz()`](@ref), which inspects every stored element of a sparse matrix. 
> 数値非ゼロの正確な数を数えるには、[`countnz()`](@ref) を使います。これは疎行列のすべての保存要素を検査します。
[`dropzeros()`](@ref), and the in-place [`dropzeros!()`](@ref), can be used to remove stored zeros from the sparse matrix.
> [`dropzeros()`](@ref) 、およびインプレース [`dropzeros！()`](@ref) は、疎行列から保存されたゼロを削除するために使用できます。

----

```jldoctest
julia> A = sparse([1, 2, 3], [1, 2, 3], [0, 2, 0])
3×3 SparseMatrixCSC{Int64,Int64} with 3 stored entries:
  [1, 1]  =  0
  [2, 2]  =  2
  [3, 3]  =  0

julia> dropzeros(A)
3×3 SparseMatrixCSC{Int64,Int64} with 1 stored entry:
  [2, 2]  =  2
```

## Sparse Vector Storage
> ## スパースベクトルストレージ ( Sparse Vector Storage )

Sparse vectors are stored in a close analog to compressed sparse column format for sparse matrices. 
> 疎ベクトルは、疎行列のための密接なアナログから圧縮疎カラムフォーマットで記憶されます。
In Julia, sparse vectors have the type [`SparseVector{Tv,Ti}`](@ref) where `Tv` is the type of the stored values and `Ti` the integer type for the indices. 
> Juliaでは、スパースベクトルの型は [`SparseVector {Tv、Ti}`](@ref) です。ここで `Tv` は格納された値の型で、`Ti` はインデックスの整数型です。
The internal representation is as follows:
> 内部表現は次のとおりです。

```julia
struct SparseVector{Tv,Ti<:Integer} <: AbstractSparseVector{Tv,Ti}
    n::Int              # Length of the sparse vector
    nzind::Vector{Ti}   # Indices of stored values
    nzval::Vector{Tv}   # Stored values, typically nonzeros
end
```

As for [`SparseMatrixCSC`](@ref), the `SparseVector` type can also contain explicitly stored zeros. 
> [`Sparse Matrix CSC`](@ref) に関しては、`Sparse Vector` 型は明示的に格納されたゼロを含むこともできます。
(See [Sparse Matrix Storage](@ref man-csc).).
> ([Sparse Matrix Storage](@ref man-csc) を参照してください)。

## Sparse Vector and Matrix Constructors
> ## 疎ベクトルと行列コンストラクタ (Sparse Vector and Matrix Constructors) 

The simplest way to create sparse arrays is to use functions equivalent to the [`zeros()`](@ref) and [`eye()`](@ref) functions that Julia provides for working with dense arrays. 
> 疎配列を作成する最も簡単な方法は、[`zeros()`](@ref) 関数や [`eye()`](@ref) 関数に相当するjuliaが高密度配列へ提供する操作を使用することです。 
To produce sparse arrays instead, you can use the same names with an `sp` prefix:
> 疎配列を生成する代りに、同じ名前に `sp` という接頭辞を付けることができます：


```jldoctest
julia> spzeros(3)
3-element SparseVector{Float64,Int64} with 0 stored entries

julia> speye(3,5)
3×5 SparseMatrixCSC{Float64,Int64} with 3 stored entries:
  [1, 1]  =  1.0
  [2, 2]  =  1.0
  [3, 3]  =  1.0
```

The [`sparse()`](@ref) function is often a handy way to construct sparse arrays. 
> [`sparse()`](@ref) 関数は、しばしば、疎配列を構築する便利な方法です。
For example, to construct a sparse matrix we can input a vector `I` of row indices, a vector `J` of column indices, and a vector `V` of stored values (this is also known as the [COO (coordinate) format](https://en.wikipedia.org/wiki/Sparse_matrix#Coordinate_list_.28COO.29)).
> 例えば、疎行列を構成するために、行インデックスのベクトル「I」、列インデックスのベクトル「J」、および格納された値のベクトル「V」を入力することができる(これは[COO(座標) format](https://en.wikipedia.org/wiki/Sparse_matrix#Coordinate_list_.28COO.29))。
`sparse(I,J,V)` then constructs a sparse matrix such that `S[I[k], J[k]] = V[k]`. 
> `sparse(I、J、V)`は、S [I [k]、J [k]] = V [k] `のような疎行列を構築する。
The equivalent sparse vector constructor is [`sparsevec`](@ref), which takes the (row) index vector `I` and the vector `V` with the stored values and constructs a sparse vector `R` such that `R[I[k]] = V[k]`.
> 同等のスパースベクトルコンストラクタは、(行)インデックスベクトル「I」とベクトル「V」を格納された値でとり、スパースベクトル「R」を構築して「R [ I [k]] = V [k] `となる。

```jldoctest sparse_function
julia> I = [1, 4, 3, 5]; J = [4, 7, 18, 9]; V = [1, 2, -5, 3];

julia> S = sparse(I,J,V)
5×18 SparseMatrixCSC{Int64,Int64} with 4 stored entries:
  [1 ,  4]  =  1
  [4 ,  7]  =  2
  [5 ,  9]  =  3
  [3 , 18]  =  -5

julia> R = sparsevec(I,V)
5-element SparseVector{Int64,Int64} with 4 stored entries:
  [1]  =  1
  [3]  =  -5
  [4]  =  2
  [5]  =  3
```

The inverse of the [`sparse()`](@ref) and [`sparsevec`](@ref) functions is [`findnz()`](@ref), which retrieves the inputs used to create the sparse array.
> [`sparse()`](@ref) と[`sparsevec`](@ref) 関数の逆は[`findnz()`](@ref)です。これはスパース配列の作成に使用される入力を取得します。
There is also a [`findn`](@ref) function which only returns the index vectors.
> インデックスベクトルだけを返す [findn`](@ref) 関数もあります。

```jldoctest sparse_function
julia> findnz(S)
([1, 4, 5, 3], [4, 7, 9, 18], [1, 2, 3, -5])

julia> findn(S)
([1, 4, 5, 3], [4, 7, 9, 18])

julia> findnz(R)
([1, 3, 4, 5], [1, -5, 2, 3])

julia> findn(R)
4-element Array{Int64,1}:
 1
 3
 4
 5
```

Another way to create a sparse array is to convert a dense array into a sparse array using the [`sparse()`](@ref) function:
> 疎配列を作成する別の方法は、密な配列を [`sparse()`](@ref) 関数を使って疎配列に変換することです：

```jldoctest
julia> sparse(eye(5))
5×5 SparseMatrixCSC{Float64,Int64} with 5 stored entries:
  [1, 1]  =  1.0
  [2, 2]  =  1.0
  [3, 3]  =  1.0
  [4, 4]  =  1.0
  [5, 5]  =  1.0

julia> sparse([1.0, 0.0, 1.0])
3-element SparseVector{Float64,Int64} with 2 stored entries:
  [1]  =  1.0
  [3]  =  1.0
```

You can go in the other direction using the [`Array`](@ref) constructor. 
> [`Array`](@ref) コンストラクタを使う他の方法で初める事も出来ます。
The [`issparse()`](@ref) function can be used to query if a matrix is sparse.
> [`issparse()`](@ref) 関数は、行列が疎であるかどうかを調べるために使用できます。


```jldoctest
julia> issparse(speye(5))
true
```

## Sparse matrix operations
> ## スパース行列演算 (Sparse matrix operations) 

Arithmetic operations on sparse matrices also work as they do on dense matrices. Indexing of, assignment into, and concatenation of sparse matrices work in the same way as dense matrices.
> スパース行列の算術演算も、密行列の場合と同じように機能します。 スパース行列の索引付け、代入および連結は、密行列と同じ方法で行われます。
Indexing operations, especially assignment, are expensive, when carried out one element at a time.
> インデクシング操作、特に割り当ては、一度に1つの要素で実行されると高価です。
In many cases it may be better to convert the sparse matrix into `(I,J,V)` format using [`findnz()`](@ref), manipulate the values or the structure in the dense vectors `(I,J,V)`, and then reconstruct the sparse matrix.
> 多くの場合、[`findnz()`](@ref) を使用して疎行列を `(I,J,V)` フォーマットに変換し、密ベクトルの `(I, J,V)` を生成し、その後、疎行列を再構築する。

## Correspondence of dense and sparse methods
> ## 高密度メソッドと疎メソッドの対応 (Correspondence of dense and sparse methods)

The following table gives a correspondence between built-in methods on sparse matrices and their corresponding methods on dense matrix types. 
> 次の表は、疎行列の組み込みメソッドと、密行列型の対応するメソッドとの対応を示しています。
In general, methods that generate sparse matrices differ from their dense counterparts in that the resulting matrix follows the same sparsity pattern as a given sparse matrix `S`, or that the resulting sparse matrix has density `d`, i.e. each matrix element has a probability `d` of being non-zero.
> 一般に、疎行列を生成する方法は、得られた行列が、所与の疎行列 `S` と同じ疎行列パターンに従うか、または結果として生じる疎行列が密度 `d` を有するという点で、それらの密な対応物とは異なる、 確率 `d` は非ゼロである。
Details can be found in the [Sparse Vectors and Matrices](@ref stdlib-sparse-arrays) section of the standard library reference.
> 詳細は標準ライブラリリファレンスの[Sparse Vectors and Matrices](@ref stdlib-sparse-arrays)セクションにあります。




| Sparse                     | Dense                  | Description                                                                                           |
|:------------------------- |:---------------------- |:-------------------------------------------------- |
| [`spzeros(m,n)`](@ref)     | [`zeros(m,n)`](@ref)   | Creates a *m*-by-*n* matrix of zeros. 
                                                        ([`spzeros(m,n)`](@ref) is empty.)               |
| [`spones(S)`](@ref)        | [`ones(m,n)`](@ref)    | Creates a matrix filled with ones. Unlike the
                                                        dense version, [`spones()`](@ref) has the same
                                                        sparsity pattern as *S*.                         |
| [`speye(n)`](@ref)         | [`eye(n)`](@ref)       | Creates a *n*-by-*n* identity matrix.             |
| [`full(S)`](@ref)          | [`sparse(A)`](@ref)    | Interconverts between dense and sparse formats.   |
| [`sprand(m,n,d)`](@ref)    | [`rand(m,n)`](@ref)    | Creates a *m*-by-*n* random matrix (of density *d*)
                                                        with iid non-zero elements distributed uniformly
                                                      | on the half-open interval ``[0, 1)``.             |
| [`sprandn(m,n,d)`](@ref)   | [`randn(m,n)`](@ref)   | Creates a *m*-by-*n* random matrix (of density *d*)
                                                        with iid non-zero elements distributed according
                                                        to the standard normal (Gaussian) distribution.   |
| [`sprandn(m,n,d,X)`](@ref) | [`randn(m,n,X)`](@ref) | Creates a *m*-by-*n* random matrix (of density *d*)
                                                        with iid non-zero elements distributed according
                                                        to the *X* distribution. 
                                                        (Requires the`Distributions` package.) | 
                1G                                        * X *分布に従って分布するiidの非ゼロ要素を持つ* m * -by- * n *ランダム行列(密度* d *)を作成します。
(`Distributions`パッケージが必要です)。