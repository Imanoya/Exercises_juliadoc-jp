# Arrays with custom indices

<!-- EN -->
Julia 0.5 adds experimental support for arrays with arbitrary indices. Conventionally, Julia's arrays are indexed starting at 1, whereas some other languages start numbering at 0, and yet others (e.g., Fortran) allow you to specify arbitrary starting indices.
> Julia 0.5は、任意のインデックスを持つ配列の実験的サポートを追加しています。 従来、Juliaの配列は1からインデックス付けされていましたが、他の言語では0に番号が付けられましたが、他の言語（Fortranなど）では任意の開始インデックスを指定できます。
<!-- EN -->
While there is much merit in picking a standard (i.e., 1 for Julia), there are some algorithms which simplify considerably if you can index outside the range `1:size(A,d)` (and not just `0:size(A,d)-1`, either).
> 標準を選ぶことにはメリットがありますが（Juliaの場合は1）、 `1：size（A、d）`の範囲外に索引を付けることができればかなり簡単になるアルゴリズムがあります（ `0：size A、d）-1 'のいずれか）。
<!-- EN -->
Such array types are expected to be supplied through packages.
> このような配列型は、パッケージを介して供給されることが期待されます。

<!-- EN -->
The purpose of this page is to address the question, "what do I have to do to support such arrays in my own code?"
> このページの目的は、「自分のコードでこのような配列をサポートするためには、どうすればよいですか？」という質問に対処することです。
<!-- EN -->
First, let's address the simplest case: if you know that your code will never need to handle arrays with unconventional indexing, hopefully the answer is "nothing." 
> まず、最も単純なケースに取り掛かりましょう。あなたのコードが、従来とは異なる索引付けで配列を処理する必要がないことが分かっているなら、うまくいけば答えは「何もありません」。
<!-- EN -->
Old code, on conventional arrays, should function essentially without alteration as long as it was using the exported interfaces of Julia.
> 従来の配列上の古いコードは、Juliaのエクスポートされたインタフェースを使用していれば、本質的に変更することなく機能するはずです。

## Generalizing existing code

<!-- EN -->
As an overview, the steps are:
> 概要として、次のステップがあります。

  <!-- EN -->
  * replace many uses of `size` with `indices`
  > * `size`の多くの用途を` indices`で置き換えます
  <!-- EN -->
  * replace `1:length(A)` with `eachindex(A)`, or in some cases `linearindices(A)`
  > * `1：length（A）`を `eachindex（A）`で置き換えるか、場合によっては `linearindices（A）`で置き換える
  <!-- EN -->
  * replace `length(A)` with `length(linearindices(A))`
  > * `length（A）`を `length（linearindices（A））`に置き換えます。
  <!-- EN -->
  * replace explicit allocations like `Array{Int}(size(B))` with `similar(Array{Int}, axes(B))`
  > * `Array {Int}（size（B））`と `similar（Array {Int}、axes（B））`のような明示的な割り当てを置き換える

<!-- EN -->
These are described in more detail below.
> これらについては、後で詳しく説明します。

### Background

<!-- EN -->
Because unconventional indexing breaks deeply-held assumptions throughout the Julia ecosystem, early adopters running code that has not been updated are likely to experience errors.
> 独創的でない索引付けは、Juliaエコシステム全体で深刻な前提条件を破るので、更新されていないコードを実行している早期採用者はエラーを経験する可能性があります。
<!-- EN -->
The most frustrating bugs would be incorrect results or segfaults (total crashes of Julia).
> 最も不満なバグは、不正確な結果またはセグメンテーション（Juliaの完全なクラッシュ）です。
<!-- EN -->
For example, consider the following function:
> たとえば、次の関数を考えます。

```julia
function mycopy!(dest::AbstractVector, src::AbstractVector)
    length(dest) == length(src) || throw(DimensionMismatch("vectors must match"))
    # OK, now we're safe to use @inbounds, right? (not anymore!)
    for i = 1:length(src)
        @inbounds dest[i] = src[i]
    end
    dest
end
```

<!-- EN -->
This code implicitly assumes that vectors are indexed from 1.
> このコードは、ベクトルが1から索引付けされることを暗黙的に前提としています。
<!-- EN -->
Previously that was a safe assumption, so this code was fine, but (depending on what types the user passes to this function) it may no longer be safe.
> これまでは安全な仮定だったので、このコードは問題ありませんでしたが、ユーザーがこの関数に渡すタイプによってはもはや安全ではないかもしれません。
<!-- EN -->
If this code continued to work when passed a vector with non-1 indices, it would either produce an incorrect answer or it would segfault.
> このコードが1以外のインデックスを持つベクトルを渡した場合、このコードが引き続き動作すると、誤った答えが生成されるか、segfaultになります。
<!-- EN -->
(If you do get segfaults, to help locate the cause try running julia with the option `--check-bounds=yes`.)
> （segfaultsを取得した場合は、原因を突き止めるために `--check-bounds = yes`オプションを付けてjuliaを実行してみてください）。

<!-- EN -->
To ensure that such errors are caught, in Julia 0.5 both `length` and `size`**should** throw an error when passed an array with non-1 indexing.
> このようなエラーが確実に発生するようにするために、Julia 0.5では、 `length`と` size` **の両方が、非1のインデックスを持つ配列を渡すとエラーが発生するはずです。
<!-- EN -->
This is designed to force users of such arrays to check the code, and inspect it for whether it needs to be generalized.
> これは、そのような配列のユーザーにコードをチェックさせ、コードを一般化する必要があるかどうかを検査するように設計されています。

### Using `indices` for bounds checks and loop iteration

<!-- EN -->
`axes(A)` (reminiscent of `size(A)`) returns a tuple of `AbstractUnitRange` objects, specifying the range of valid indices along each dimension of `A`.
> `axes（A）`（ `size（A）`を連想させる）は `AbstractUnitRange`オブジェクトのタプルを返し、` A`の各次元に沿った有効なインデックスの範囲を指定します。
<!-- EN -->
When `A` has unconventional indexing, the ranges may not start at 1.
> `A`が慣習的でないインデックス付けをしているとき、範囲は1で始まらないかもしれません。
<!-- EN -->
If you just want the range for a particular dimension `d`, there is `axes(A, d)`.
> 特定の次元 'd'の範囲を欲しければ、 `axes（A、d）`があります。

<!-- EN -->
Base implements a custom range type, `OneTo`, where `OneTo(n)` means the same thing as `1:n` but in a form that guarantees (via the type system) that the lower index is 1.
> Baseは `OneTo（n）`が `1：n`と同じことを意味するが、（型システムを介して）低いインデックスが1であることを保証する形式で、カスタム範囲型、` OneTo`を実装します。
<!-- EN -->
For any new [`AbstractArray`](@ref) type, this is the default returned by `indices`, and it indicates that this array type uses "conventional" 1-based indexing.
> 新しい[`AbstractArray`]（@ref）型の場合、これは` indices`によって返されるデフォルトです。この配列型は "従来の" 1ベースのインデックスを使用しています。
<!-- EN -->
Note that if you don't want to be bothered supporting arrays with non-1 indexing, you can add the following line:
<!-- JP -->
> 1以外のインデックスを持つ配列のサポートに気を配りたくない場合は、次の行を追加することができます：

```julia
@assert all(x->isa(x, Base.OneTo), axes(A))
```

<!-- EN -->
at the top of any function.
> 任意の機能の先頭に表示されます。

<!-- EN -->
For bounds checking, note that there are dedicated functions `checkbounds` and `checkindex` which can sometimes simplify such tests.
> 境界チェックのために、 `checkbounds`と` checkindex`という専用の関数があり、そのようなテストを単純化することがあることに注意してください。

### Linear indexing (`linearindices`)

<!-- EN -->
Some algorithms are most conveniently (or efficiently) written in terms of a single linear index, `A[i]` even if `A` is multi-dimensional.
> いくつかのアルゴリズムは、Aが多次元であっても、単一の線形インデックス、A [i]の観点から最も便利に（または効率的に）書かれます。
<!-- EN -->
Regardless of the array's native indices, linear indices always range from `1:length(A)`.
> 配列のネイティブインデックスにかかわらず、線形インデックスは常に `1：length（A）`の範囲です。
<!-- EN -->
However, this raises an ambiguity for one-dimensional arrays (a.k.a., [`AbstractVector`](@ref)): does `v[i]` mean linear indexing , or Cartesian indexing with the array's native indices?
> しかし、これは1次元配列（a.k.a.、[AbstractVector`]（@ ref））のあいまいさを引き起こします： `v [i]`は線形インデクシングか、配列のネイティブインデックスを用いたデカルトインデックス付けを意味しますか？

<!-- EN -->
For this reason, your best option may be to iterate over the array with `eachindex(A)`, or, if you require the indices to be sequential integers, to get the index range by calling `linearindices(A)`.
> このため、最良の選択肢は `eachindex（A）`を使って配列を繰り返し処理したり、インデックスが `linearindices（A）`を呼び出してインデックス範囲を取得するためにインデックスを連続した整数にしなければならない場合があります。
<!-- EN -->
This will return `axes(A, 1)` if A is an AbstractVector, and the equivalent of `1:length(A)` otherwise.
> これは、AがAbstractVectorの場合は `axes（A、1）`を返し、それ以外の場合は `1：length（A）`の等価物を返します。

<!-- EN -->
By this definition, 1-dimensional arrays always use Cartesian indexing with the array's native indices.
> この定義では、1次元配列は常に配列の固有のインデックスでデカルトインデックスを使用します。
<!-- EN -->
To help enforce this, it's worth noting that the index conversion functions will throw an error if shape indicates a 1-dimensional array with unconventional indexing (i.e., is a `Tuple{UnitRange}` rather than a tuple of `OneTo`).
> これを実行するのを助けるために、shapeが非定型索引付け（すなわち、OneToのタプルではなく、Tuple {UnitRange}）である1次元配列を示す場合、インデックス変換関数がエラーをスローすることは注目に値する。
<!-- EN -->
For arrays with conventional indexing, these functions continue to work the same as always.
> 従来の索引付き配列の場合、これらの関数は常に同じように動作し続けます。

<!-- EN -->
Using `indices` and `linearindices`, here is one way you could rewrite `mycopy!`:
> `indices`と` linearindices`を使って、 `mycopy！ 'を書き直す方法があります：

```julia
function mycopy!(dest::AbstractVector, src::AbstractVector)
    axes(dest) == axes(src) || throw(DimensionMismatch("vectors must match"))
    for i in linearindices(src)
        @inbounds dest[i] = src[i]
    end
    dest
end
```

### Allocating storage using generalizations of `similar`

<!-- EN -->
Storage is often allocated with `Array{Int}(uninitialized, dims)` or `similar(A, args...)`.
> ストレージは `Array {Int}（初期化されていない、dims）`または `similar（A、args ...）`で割り当てられることがよくあります。
<!-- EN -->
When the result needs to match the indices of some other array, this may not always suffice.
> 結果が他の配列のインデックスと一致する必要がある場合、必ずしも十分ではないかもしれません。
<!-- EN -->
The generic replacement for such patterns is to use `similar(storagetype, shape)`.
> そのようなパターンの一般的な置き換えは `類似した(storagetype、shape)` を使うことです。
<!-- EN -->
`storagetype` indicates the kind of underlying "conventional" behavior you'd like, e.g., `Array{Int}` or `BitArray` or even `dims->zeros(Float32, dims)` (which would allocate an all-zeros array).
> `storagetype`は` Array {Int} `や` BitArray`や `dims-> 0（Float32、dims）`などの基本的な "従来の"振る舞いの種類を示します（これはオールゼロアレイ）。
<!-- EN -->
`shape` is a tuple of [`Integer`](@ref) or `AbstractUnitRange` values, specifying the indices that you want the result to use.
> `shape`は[` Integer`]（@ref）または `AbstractUnitRange`値のタプルで、結果を使用するインデックスを指定します。
<!-- EN -->
Note that a convenient way of producing an all-zeros array that matches the indices of A is simply `zeros(A)`.
> Aのインデックスと一致するオールゼロ配列を生成する便利な方法は単に `0（A）`であることに注意してください。

<!-- EN -->
Let's walk through a couple of explicit examples.
> いくつかの明示的な例を見てみましょう。
<!-- EN -->
First, if `A` has conventional indices, then `similar(Array{Int}, axes(A))` would end up calling `Array{Int}(size(A))`, and thus return an array.
> まず、 `A` に従来のインデックスがある場合、 `Similar(Array {Int},axes(A))` は `Array{Int}(size(A))` を呼び出して配列を返します。
<!-- EN -->
If `A` is an `AbstractArray` type with unconventional indexing, then `similar(Array{Int}, axes(A))` should return something that "behaves like" an `Array{Int}` but with a shape (including indices) that matches `A`.
> もし `A` が非慣習的なインデックスを持つ `AbstractArray` 型であれば `similar(Array{Int}, axes(A))`は `Array{Int}` のように "振る舞い"インデックス）を返します。
<!-- EN -->
(The most obvious implementation is to allocate an `Array{Int}(uninitialized, size(A))` and then "wrap" it in a type that shifts the indices.)
> （最も明白な実装は、 `Array {Int}( 初期化されていない、サイズ(A))` を割り当ててから、インデックスをシフトする型で "ラップ"することです）。

<!-- EN -->
Note also that `similar(Array{Int}, (axes(A, 2),))` would allocate an `AbstractVector{Int}` (i.e., 1-dimensional array) that matches the indices of the columns of `A`.
> `Similar(Array{Int},(axes(A,2),))` は `A` の列のインデックスに一致する `AbstractVector{Int}`（すなわち、1次元配列） 。

### Deprecations

<!-- EN -->
In generalizing Julia's code base, at least one deprecation was unavoidable: earlier versions of Julia defined `first(::Colon) = 1`, meaning that the first index along a dimension indexed by `:` is 1.
> Juliaのコードベースを一般化すると、少なくとも1つの廃止が避けられませんでした。Juliaの以前のバージョンでは `first（:: Colon）= 1`が定義されていました。つまり、`： `でインデックス付けされた次元に沿った最初のインデックスは1です。
<!-- EN -->
This definition can no longer be justified, so it was deprecated.
> この定義はもはや正当化できないため、非推奨になりました。
<!-- EN -->
There is no provided replacement, because the proper replacement depends on what you are doing and might need to know more about the array.
> 適切な置換えは、あなたがやっていることと配列についてもっと知る必要があるかもしれないので、置換えはありません。
<!-- EN -->
However, it appears that many uses of `first(::Colon)` are really about computing an index offset; when that is the case, a candidate replacement is:
> しかし、 `first(::Colon)` の多くの使い方は実際にインデックスオフセットを計算することになります。 その場合、候補の置き換えは次のとおりです。

```julia
indexoffset(r::AbstractVector) = first(r) - 1
indexoffset(::Colon) = 0
```

<!-- EN -->
In other words, while `first(:)` does not itself make sense, in general you can say that the offset associated with a colon-index is zero.
> 言い換えれば、 `first(:)` はそれ自身が意味をなさないが、一般的にコロンインデックスに関連するオフセットはゼロであると言うことができる。

## Writing custom array types with non-1 indexing

<!-- EN -->
Most of the methods you'll need to define are standard for any `AbstractArray` type, see [Abstract Arrays](@ref man-interface-array).
> 定義する必要のあるメソッドのほとんどは、任意の `AbstractArray` 型の標準です。[Abstract Array]（@ref man-interface-array）を参照してください。
<!-- EN -->
This page focuses on the steps needed to define unconventional indexing.
> このページでは、非定型索引を定義するために必要な手順を中心に説明します。

### Do **not** implement `size` or `length`

<!-- EN -->
Perhaps the majority of pre-existing code that uses `size` will not work properly for arrays with non-1 indices.
> おそらく、 `size` を使う既存のコードの大部分は、1以外のインデックスを持つ配列に対してはうまく動作しません。
<!-- EN -->
For that reason, it is much better to avoid implementing these methods, and use the resulting `MethodError` to identify code that needs to be audited and perhaps generalized.
> そのため、これらのメソッドの実装を避け、結果として得られる `MethodError` を使用して、監査が必要で一般化される必要があるコードを識別する方がはるかに優れています。

### Do **not** annotate bounds checks

<!-- EN -->
Julia 0.5 includes `@boundscheck` to annotate code that can be removed for callers that exploit `@inbounds`.
> Julia 0.5には `@boundscheck` が含まれており、 `@inbounds` を利用する呼び出し元に対して削除できるコードに注釈を付けることができます。
<!-- EN -->
Initially, it seems far preferable to run with bounds checking always enabled (i.e., omit the `@boundscheck` annotation so the check always runs).
> 最初は、常に有効になっている境界チェックを実行して（つまり、チェックが常に実行されるように `@boundscheck` アノテーションを省略して）実行するほうがはるかに好ましいようです。

### Custom `AbstractUnitRange` types

<!-- EN -->
If you're writing a non-1 indexed array type, you will want to specialize `indices` so it returns a `UnitRange`, or (perhaps better) a custom `AbstractUnitRange`.
> non-1インデックス配列型を書いているならば、 `UnitRange`を返すように` indices`を特化したいでしょうし、カスタム `AbstractUnitRange`を（おそらくもっと良い）返すでしょう。
<!-- EN -->
The advantage of a custom type is that it "signals" the allocation type for functions like `similar`.
> カスタムタイプの利点は、それが `類似`のような関数の割り振りタイプを「シグナル」することです。
<!-- EN -->
If we're writing an array type for which indexing will start at 0, we likely want to begin by creating a new `AbstractUnitRange`, `ZeroRange`, where `ZeroRange(n)` is equivalent to `0:n-1`.
> インデックス作成が0で始まる配列型を作成する場合は、まず、新しい `AbstractUnitRange`、`ZeroRangeを作成します`。ここで、`ZeroRange(n)` は `0：n-1` と等価です。 。

<!-- EN -->
In general, you should probably *not* export `ZeroRange` from your package: there may be other packages that implement their own `ZeroRange`, and having multiple distinct `ZeroRange` types is (perhaps counterintuitively) an advantage: `ModuleA.ZeroRange` indicates that `similar` should create a `ModuleA.ZeroArray`, whereas `ModuleB.ZeroRange` indicates a `ModuleB.ZeroArray` type.
> 一般的には、あなたのパッケージから `ZeroRange` をエクスポートするべきではないでしょう：独自の `ZeroRange` を実装する他のパッケージがあり、複数の異なる `ZeroRange` 型を持つ（おそらく反直観的に）利点があります： `ModuleA.ZeroRange` `ModuleB.ZeroRange` は ` ModuleB.ZeroArray` 型を示しますが、 `Similar` は  `ModuleA.ZeroArray` を生成するはずです。
<!-- EN -->
This design allows peaceful coexistence among many different custom array types.
>  この設計により、さまざまなカスタムアレイタイプ間の平和な共存が可能になります。

<!-- EN -->
Note that the Julia package [CustomUnitRanges.jl](https://github.com/JuliaArrays/CustomUnitRanges.jl) can sometimes be used to avoid the need to write your own `ZeroRange` type.
> ジュリアパッケージ [CustomUnitRanges.jl](https://github.com/JuliaArrays/CustomUnitRanges.jl) は、独自の `ZeroRange` 型を書く必要を避けるために使うことができます。

### Specializing `indices`

<!-- EN -->
Once you have your `AbstractUnitRange` type, then specialize `indices`:
> `AbstractUnitRange` 型を取得したら、 `indices` を特化します：

```julia
Base.axes(A::ZeroArray) = map(n->ZeroRange(n), A.size)
```

<!-- EN -->
where here we imagine that `ZeroArray` has a field called `size` (there would be other ways to implement this).
> `ZeroArray` には `size` というフィールドがあります（これを実装する他の方法があります）。

<!-- EN -->
In some cases, the fallback definition for `axes(A, d)`:
> 場合によっては、 `axes(A,d)` のフォールバック定義：

```julia
axes(A::AbstractArray{T,N}, d) where {T,N} = d <= N ? axes(A)[d] : OneTo(1)
```

<!-- EN -->
may not be what you want: you may need to specialize it to return something other than `OneTo(1)` when `d > ndims(A)`.
> `d > ndims(A)` のときに `OneTo(1)` 以外のものを返すように特殊化する必要があるかもしれません。
<!-- EN -->
Likewise, in `Base` there is a dedicated function `indices1` which is equivalent to `axes(A, 1)` but which avoids checking (at runtime) whether `ndims(A) > 0`.
> 同様に、 `Base` に `axes(A, 1)` に相当するが `ndims(A) > 0` であるかどうかを（実行時に）チェックすることを避ける専用の関数 `indices1` があります。
<!-- EN -->
(This is purely a performance optimization.)
> (これは純粋にパフォーマンスの最適化です。)
<!-- EN -->
It is defined as:
> これは次のように定義されます。

```julia
indices1(A::AbstractArray{T,0}) where {T} = OneTo(1)
indices1(A::AbstractArray) = axes(A)[1]
```

<!-- EN -->
If the first of these (the zero-dimensional case) is problematic for your custom array type, be sure to specialize it appropriately.
> これらのうちの最初のもの（ゼロ次元のケース）がカスタム配列タイプに問題がある場合は、それを適切に特殊化するようにしてください。

### Specializing `similar`

<!-- EN -->
Given your custom `ZeroRange` type, then you should also add the following two specializations for `similar`:
> あなたのカスタム `ZeroRange` 型を与えられたら、 `similar` に以下の2つの特殊化を加えるべきです：

```julia
function Base.similar(A::AbstractArray, T::Type, shape::Tuple{ZeroRange,Vararg{ZeroRange}})
    # body
end

function Base.similar(f::Union{Function,DataType}, shape::Tuple{ZeroRange,Vararg{ZeroRange}})
    # body
end
```

<!-- EN -->
Both of these should allocate your custom array type.
> どちらもカスタム配列タイプを割り当てる必要があります。

### Specializing `reshape`

<!-- EN -->
Optionally, define a method
> オプションで、メソッドを定義する

```
Base.reshape(A::AbstractArray, shape::Tuple{ZeroRange,Vararg{ZeroRange}}) = ...
```

<!-- EN -->
and you can `reshape` an array so that the result has custom indices.
> 結果がカスタムインデックスを持つように配列を `reshape` することができます。

## Summary

<!-- EN -->
Writing code that doesn't make assumptions about indexing requires a few extra abstractions, but hopefully the necessary changes are relatively straightforward.
> 索引作成に関する前提条件を設定していないコードを作成するには、いくつかの抽象化が必要ですが、必要な変更は比較的簡単です。

<!-- EN -->
As a reminder, this support is still experimental.
> このサポートはまだ実験的なものです。
<!-- EN -->
While much of Julia's base code has been updated to support unconventional indexing, without a doubt there are many omissions that will be discovered only through usage.
> Juliaの基本コードの多くは、従来とは異なる索引付けをサポートするように更新されていますが、間違いなく、使用によってのみ発見されることが多くあります。
<!-- EN -->
Moreover, at the time of this writing, most packages do not support unconventional indexing.
> さらに、この記事の執筆時点では、ほとんどのパッケージは非従来型の索引付けをサポートしていません。
<!-- EN -->
As a consequence, early adopters should be prepared to identify and/or fix bugs.
> その結果、早期導入者はバグを特定および/または修正する用意ができているはずです。
<!-- EN -->
On the other hand, only through practical usage will it become clear whether this experimental feature should be retained in future versions of Julia; consequently, interested parties are encouraged to accept some ownership for putting it through its paces.
> 一方、この実験的特徴が将来のJuliaのバージョンで保持されるべきかどうかは、実際の使用を通してのみ明らかになります。 その結果、利害関係者は、ペースでそれを置くために何らかの所有権を受け入れることが奨励されます。