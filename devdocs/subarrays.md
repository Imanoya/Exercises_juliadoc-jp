# SubArrays

Julia's `SubArray` type is a container encoding a "view" of a parent [`AbstractArray`](@ref).
> Juliaの `SubArray`型は親[` AbstractArray`](@ref)の "view"をエンコードするコンテナです。
This page documents some of the design principles and implementation of `SubArray`s.
> このページでは、いくつかの設計原則と `SubArray`の実装について説明します。

## Indexing: cartesian vs. linear indexing

Broadly speaking, there are two main ways to access data in an array.
> 概して、アレイ内のデータにアクセスする主な2つの方法があります。
The first, often called cartesian indexing, uses `N` indices for an `N` -dimensional `AbstractArray`.
> 最初に、デカルト索引付けと呼ばれるものは、 `N`次元の` AbstractArray`に `N`インデックスを使用します。
For example, a matrix `A` (2-dimensional) can be indexed in cartesian style as `A[i,j]`.
> 例えば、行列A（2次元）は、デカルト形式で、A [i、j] `として索引付けすることができる。
The second indexing method, referred to as linear indexing, uses a single index even for higher-dimensional objects.
> 線形索引付けと呼ばれる第2の索引付け方法は、高次のオブジェクトに対しても単一の索引を使用します。
For example, if `A = reshape(1:12, 3, 4)`, then the expression `A[5]` returns the value 5.
> たとえば、 `A = reshape（1:12,3,4）`の場合、式A [5]は値5を返します。
Julia allows you to combine these styles of indexing: for example, a 3d array `A3` can be indexed as `A3[i,j]`, in which case `i` is interpreted as a cartesian index for the first dimension, and `j` is a linear index over dimensions 2 and 3.
> Juliaでは、これらのスタイルのインデックスを組み合わせることができます。たとえば、3次元配列「A3」は、A3 [i、j]としてインデックス付けできます。この場合、iは最初の次元のデカルトインデックスとして解釈され、 「j」は次元2および3にわたる線形インデックスである。

For `Array`s, linear indexing appeals to the underlying storage format: an array is laid out as a contiguous block of memory, and hence the linear index is just the offset (+1) of the corresponding entry relative to the beginning of the array.
> `Array`の場合、線形インデクシングは基本的な記憶形式に訴えます。配列は連続したメモリブロックとしてレイアウトされます。したがって線形インデックスは、対応するエントリのオフセット（+1）だけです。アレイ。
However, this is not true for many other `AbstractArray` types: examples include [`SparseMatrixCSC`](@ref) from the `SparseArrays` standard library module, arrays that require some kind of computation (such as interpolation), and the type under discussion here, `SubArray`.
> しかし、これは他の多くの `AbstractArray`型では当てはまりません：例は、` SparseArrays`標準ライブラリモジュールの[`SparseMatrixCSC`]（@ref）、ある種の計算（補間など）を必要とする配列、ここで議論中の「SubArray」。
For these types, the underlying information is more naturally described in terms of cartesian indices.
> これらのタイプの場合、基礎となる情報はより自然にデカルトインデックスで記述されます。

The `getindex` and `setindex!` functions for `AbstractArray` types may include automatic conversion between indexing types.
> `AbstractArray`型の` getindex`と `setindex！ '関数は、インデックス型間の自動変換を含むかもしれません。
For explicit conversion, [`CartesianIndices`](@ref) can be used.
> 明示的な変換では、[`CartesianIndices`]（@ ref）を使うことができます。

While converting from a cartesian index to a linear index is fast (it's just multiplication and addition), converting from a linear index to a cartesian index is very slow: it relies on the `div` operation, which is one of the slowest low-level operations you can perform with a CPU.
> デカルトインデックスからリニアインデックスへの変換は高速です（乗算と加算だけです）。リニアインデックスからデカルトインデックスへの変換は非常に遅いです。これは、最も遅いローレベルインデックスの1つである `div`演算に依存します。 CPUで実行できるレベルの操作。
For this reason, any code that deals with `AbstractArray` types is best designed in terms of cartesian, rather than linear, indexing.
> このため、 `AbstractArray`型を扱うコードは、線形ではなくデカルトで表現するのが最適です。

## Index replacement

Consider making 2d slices of a 3d array:
> 3D配列の2次元スライスを作成することを検討してください：

```@meta
DocTestSetup = :(import Random; Random.srand(1234))
```
```jldoctest subarray
julia> A = rand(2,3,4);

julia> S1 = view(A, :, 1, 2:3)
2×2 view(::Array{Float64,3}, :, 1, 2:3) with eltype Float64:
 0.200586  0.066423
 0.298614  0.956753

julia> S2 = view(A, 1, :, 2:3)
3×2 view(::Array{Float64,3}, 1, :, 2:3) with eltype Float64:
 0.200586  0.066423
 0.246837  0.646691
 0.648882  0.276021
```
```@meta
DocTestSetup = nothing
```

`view` drops "singleton" dimensions (ones that are specified by an `Int`), so both `S1` and `S2` are two-dimensional `SubArray`s.
> `view`は` singleton 'ディメンション（ `Int`で指定されたディメンション）を削除するので、` S1`と `S2`は2次元の` SubArray`です。
Consequently, the natural way to index these is with `S1[i,j]`.
> したがって、これらをインデックス化する自然な方法は `S1 [i、j]`です。
To extract the value from the parent array `A`, the natural approach is to replace `S1[i,j]` with `A[i,1,(2:3)[j]]` and `S2[i,j]` with `A[1,i,(2:3)[j]]`.
> 親配列Aから値を抽出するには、S1 [i、j]をA [i、1、（2：3）[j]]およびS2 [i、 j]をA [1、i、（2：3）[j]]と置き換えたものである。

The key feature of the design of SubArrays is that this index replacement can be performed without any runtime overhead.
> サブアレイの設計の重要な特徴は、このインデックスの置き換えがランタイムオーバーヘッドなしで実行できることです。

## SubArray design

### Type parameters and fields

The strategy adopted is first and foremost expressed in the definition of the type:
> 採用された戦略は、第一に、タイプの定義で表現されています。

```julia
struct SubArray{T,N,P,I,L} <: AbstractArray{T,N}
    parent::P
    indices::I
    offset1::Int       # for linear indexing and pointer, only valid when L==true
    stride1::Int       # used only for linear indexing
    ...
end
```

`SubArray` has 5 type parameters.  The first two are the standard element type and dimensionality.
> `SubArray`には5つの型パラメータがあります。 最初の2つは標準的な要素の種類と次元です。
The next is the type of the parent `AbstractArray`.  The most heavily-used is the fourth parameter, a `Tuple` of the types of the indices for each dimension.
> 次は、親 `AbstractArray`の型です。 最も頻繁に使用されるのは、4番目のパラメータで、各次元のインデックスの型の「タプル」です。
The final one, `L`, is only provided as a convenience for dispatch; it's a boolean that represents whether the index types support fast linear indexing.
> 最後のもの、 `L`は発送の便宜のためにのみ提供されています。 索引タイプが高速リニア索引付けをサポートしているかどうかを表すブール値です。 
More on that later.
> 後で詳しく説明します。

If in our example above `A` is a `Array{Float64, 3}`, our `S1` case above would be a `SubArray{Float64,2,Array{Float64,3},Tuple{Base.Slice{Base.OneTo{Int64}},Int64,UnitRange{Int64}},false}`.
> 上の例で `A`が` Array {Float64、3} `の場合、上の` S1`は `SubArray {Float64,2、Array {Float64,3}、Tuple {Base.Slice {Base。 OneTo {Int64}}、Int64、UnitRange {Int64}}、false} `を返します。
Note in particular the tuple parameter, which stores the types of the indices used to create `S1`.
> 特に、 `S1`を生成するために使用されるインデックスのタイプを格納するtupleパラメータに注意してください。
Likewise,
> 同様に、

```jldoctest subarray
julia> S1.indices
(Base.Slice(Base.OneTo(2)), 1, 2:3)
```

Storing these values allows index replacement, and having the types encoded as parameters allows one to dispatch to efficient algorithms.
> これらの値を格納することで、インデックスの置換が可能になり、パラメータとしてコード化された型を使用することで、効率的なアルゴリズムにディスパッチすることができます。

### Index translation

Performing index translation requires that you do different things for different concrete `SubArray` types.
> インデックスの変換を行うには、具体的な `SubArray`タイプごとに異なることが必要です。
For example, for `S1`, one needs to apply the `i,j` indices to the first and third dimensions of the parent array, whereas for `S2` one needs to apply them to the second and third.
> 例えば、 `S1`では親配列の第1次元と第3次元に` i、j`インデックスを適用する必要がありますが、 `S2`では2番目と3番目にそれを適用する必要があります。
The simplest approach to indexing would be to do the type-analysis at runtime:
> 索引付けを行う最も簡単な方法は、実行時に型分析を行うことです。

```julia
parentindices = Vector{Any}()
for thisindex in S.indices
    ...
    if isa(thisindex, Int)
        # Don't consume one of the input indices
        push!(parentindices, thisindex)
    elseif isa(thisindex, AbstractVector)
        # Consume an input index
        push!(parentindices, thisindex[inputindex[j]])
        j += 1
    elseif isa(thisindex, AbstractMatrix)
        # Consume two input indices
        push!(parentindices, thisindex[inputindex[j], inputindex[j+1]])
        j += 2
    elseif ...
end
S.parent[parentindices...]
```

Unfortunately, this would be disastrous in terms of performance: each element access would allocate memory, and involves the running of a lot of poorly-typed code.
> 残念なことに、これはパフォーマンス面で悲惨なものになります。各要素のアクセスはメモリを割り当て、型の悪いコードの実行を伴います。

The better approach is to dispatch to specific methods to handle each type of stored index.
> より良いアプローチは、ストアド・インデックスの各タイプを処理する特定のメソッドにディスパッチすることです。
That's what `reindex` does: it dispatches on the type of the first stored index and consumes the appropriate number of input indices, and then it recurses on the remaining indices.
> これは `reindex`のやり方です。最初に格納されたインデックスの型にディスパッチし、適切な数の入力インデックスを消費し、残りのインデックスを再帰します。
In the case of `S1`, this expands to
> 「S1」の場合、これは

```julia
Base.reindex(S1, S1.indices, (i, j)) == (i, S1.indices[2], S1.indices[3][j])
```

for any pair of indices `(i,j)` (except [`CartesianIndex`](@ref)s and arrays thereof, see below).
> `(i,j)`([CartesianIndex`](@ref)とその配列を除く、以下を参照)。

This is the core of a `SubArray`; indexing methods depend upon `reindex` to do this index translation.
> これは `SubArray`の中核です。 索引付け方法は、この索引変換を行うための「再索引」に依存します。
Sometimes, though, we can avoid the indirection and make it even faster.
> しかし、時には間接を避けてより速くすることができます。

### Linear indexing

Linear indexing can be implemented efficiently when the entire array has a single stride that separates successive elements, starting from some offset.
>線形索引付けは、配列全体が、オフセットから始まる連続した要素を区切る単一のストライドを持つ場合、効率的に実装できます。
This means that we can pre-compute these values and represent linear indexing simply as an addition and multiplication, avoiding the indirection of `reindex` and (more importantly) the slow computation of the cartesian coordinates entirely.
>これは、これらの値を事前に計算し、単純に加算と乗算として線形インデックスを表現できることを意味します。「再索引」の間接参照と（より重要なことに）デカルト座標の完全な計算を完全に避けます。

For `SubArray` types, the availability of efficient linear indexing is based purely on the types of the indices, and does not depend on values like the size of the parent array.
>`SubArray`型では、効率的な線形インデクシングの利用可能性は、インデックスの種類にのみ基づいており、親配列のサイズなどの値には依存しません。
You can ask whether a given set of indices supports fast linear indexing with the internal `Base.viewindexing` function:
>与えられたインデックスセットが内部の `Base.viewindexing`関数で高速リニアインデックスをサポートしているかどうかを調べることができます：

```jldoctest subarray
julia> Base.viewindexing(S1.indices)
IndexCartesian()

julia> Base.viewindexing(S2.indices)
IndexLinear()
```

This is computed during construction of the `SubArray` and stored in the `L` type parameter as a boolean that encodes fast linear indexing support.
> これは、 `SubArray`の構築中に計算され、` L`型パラメータに、高速リニアインデックスのサポートをエンコードするブール値として格納されます。
While not strictly necessary, it means that we can define dispatch directly on `SubArray{T,N,A,I,true}` without any intermediaries.
> 厳密には必要ではありませんが、仲介なしで `SubArray {T、N、A、I、true} 'に直接ディスパッチを定義できることを意味します。

Since this computation doesn't depend on runtime values, it can miss some cases in which the stride happens to be uniform:
> この計算は実行時の値に依存しないため、ストライドが一様になる場合があります。

```jldoctest
julia> A = reshape(1:4*2, 4, 2)
4×2 reshape(::UnitRange{Int64}, 4, 2) with eltype Int64:
 1  5
 2  6
 3  7
 4  8

julia> diff(A[2:2:4,:][:])
3-element Array{Int64,1}:
 2
 2
 2
```

A view constructed as `view(A, 2:2:4, :)` happens to have uniform stride, and therefore linear indexing indeed could be performed efficiently.
> `view（A、2：2：4、：） 'として構築されたビューは、一様なストライドを持つようになり、実際に線形インデクシングを効率的に実行することができました。
However, success in this case depends on the size of the array: if the first dimension instead were odd,
> しかし、この場合の成功は配列のサイズに依存します。最初の次元が奇数の場合は、

```jldoctest
julia> A = reshape(1:5*2, 5, 2)
5×2 reshape(::UnitRange{Int64}, 5, 2) with eltype Int64:
 1   6
 2   7
 3   8
 4   9
 5  10

julia> diff(A[2:2:4,:][:])
3-element Array{Int64,1}:
 2
 3
 2
```

then `A[2:2:4,:]` does not have uniform stride, so we cannot guarantee efficient linear indexing.
>  `A [2：2：4、：]`は一様なストライドを持たないため、効率的な線形インデクシングを保証することはできません。
 Since we have to base this decision based purely on types encoded in the parameters of the `SubArray`, `S = view(A, 2:2:4, :)` cannot implement efficient linear indexing.
>   `S = view（A、2：2：4、：）`は、 `SubArray`のパラメータでコード化された型だけに基づいてこの決定を行う必要があるため、効率的な線形インデクシングを実装することはできません。

### A few details

  * Note that the `Base.reindex` function is agnostic to the types of the input indices; it simply determines how and where the stored indices should be reindexed.
  > * `Base.reindex` 関数は入力インデックスの型には無関心です。 格納されたインデックスをどのように再インデックスするかを単に決定します。
  It not only supports integer indices, but it supports non-scalar indexing, too.
  > 整数インデックスをサポートするだけでなく、非スカラーインデックスもサポートしています。
  This means that views of views don't need two levels of indirection; they can simply re-compute the indices into the original parent array!
  > これは、ビューのビューに2レベルの間接参照が必要ないことを意味します。 彼らは元の親配列へのインデックスを単純に再計算できます！
  * Hopefully by now it's fairly clear that supporting slices means that the dimensionality, given by the parameter `N`, is not necessarily equal to the dimensionality of the parent array or the length of the `indices` tuple.
  > * これまで、スライスをサポートするということは、パラメータ 'N'によって与えられる次元が、親配列の次元数または 'インデックス'タプルの長さと必ずしも等しくないことを意味することは明らかです。
  Neither do user-supplied indices necessarily line up with entries in the `indices` tuple (e.g., the second user-supplied index might correspond to the third dimension of the parent array, and the third element in the `indices` tuple).
  > また、ユーザが提供するインデックスは必ずしも `indices`タプル（例えば、第2のユーザ指定インデックスは親配列の第3次元に対応し、第3の要素は`インデックス 'タプルに対応する）のエントリと整列しない。

    What might be less obvious is that the dimensionality of the stored parent array must be equal to the number of effective indices in the `indices` tuple.
    > 目立たないことは、格納されている親配列の次元数が、 `indices`タプルの有効なインデックスの数と等しくなければならないということです。
    Some examples:
    > いくつかの例：

    ```julia
    A = reshape(1:35, 5, 7) # A 2d parent Array
    S = view(A, 2:7)         # A 1d view created by linear indexing
    S = view(A, :, :, 1:1)   # Appending extra indices is supported
    ```

    Naively, you'd think you could just set `S.parent = A` and `S.indices = (:,:,1:1)`, but supporting this dramatically complicates the reindexing process, especially for views of views.
    > 純粋に `S parent = A`と` S.indices =（：、：、1：1） `を設定できると思うでしょうが、これをサポートすることは再インデックス処理を劇的に複雑にします。
    Not only do you need to dispatch on the types of the stored indices, but you need to examine whether a given index is the final one and "merge" any remaining stored indices together.
    > 格納されたインデックスの型をディスパッチする必要があるだけでなく、指定されたインデックスが最後のインデックスであるかどうかを調べ、残りの格納されたインデックスを一緒に "マージ"する必要があります。
    This is not an easy task, and even worse: it's slow since it implicitly depends upon linear indexing.
    > これは簡単な作業ではなく、さらに悪いことです。暗黙のうちに線形索引付けに依存するため遅いです。

    Fortunately, this is precisely the computation that `ReshapedArray` performs, and it does so linearly if possible.
    > 幸いにも、これは正確には `ReshapedArray`が実行する計算であり、可能であれば直線的に実行されます。
    Consequently, `view` ensures that the parent array is the appropriate dimensionality for the given indices by reshaping it if needed.
    > したがって、 `view`は、親配列が、必要に応じて再構成することによって、指定されたインデックスの適切な次元であることを保証します。
    The inner `SubArray` constructor ensures that this invariant is satisfied.
    > 内側の `SubArray`コンストラクタは、この不変量が確実に満たされるようにします。

  * [`CartesianIndex`](@ref) and arrays thereof throw a nasty wrench into the `reindex` scheme.
  * > [`CartesianIndex`]（@ ref）とその配列は、厄介なレンチを` reindex`スキームに投げます。
    Recall that `reindex` simply dispatches on the type of the stored indices in order to determine how many passed indices should be used and where they should go.
    > `reindex`は、いくつの渡されたインデックスが使用されるべきか、どこに行くべきかを決定するために、格納されたインデックスの型を単にディスパッチすることを思い出してください。
    But with `CartesianIndex`, there's no longer a one-to-one correspondence between the number of passed arguments and the number of dimensions that they index into.
    > しかし、 `CartesianIndex`では、渡された引数の数とそれらがインデックスされる次元の数との間には、もはや1対1の対応がありません。
    If we return to the above example of `Base.reindex(S1, S1.indices, (i, j))`, you can see that the expansion is incorrect for `i, j = CartesianIndex(), CartesianIndex(2,1)`.
    > 上記の `Base.reindex（S1、S1.indices、（i、j））`の例に戻ると、展開が `i、j = CartesianIndex（）、CartesianIndex（2,1） ） `。
    It should *skip* the `CartesianIndex()` entirely and return:
    > それは `CartesianIndex（）`を完全に*スキップして*返す必要があります：

    ```julia
    (CartesianIndex(2,1)[1], S1.indices[2], S1.indices[3][CartesianIndex(2,1)[2]])
    ```

    Instead, though, we get:

    ```julia
    (CartesianIndex(), S1.indices[2], S1.indices[3][CartesianIndex(2,1)])
    ```

    Doing this correctly would require *combined* dispatch on both the stored and passed indices across all combinations of dimensionalities in an intractable manner.
    > これを正しく行うには、格納されたインデックスと渡されたインデックスの両方で、次元のすべての組み合わせに渡って扱いにくい方法で* combined * dispatchが必要です。
    As such, `reindex` must never be called with `CartesianIndex` indices.
    > したがって、 `再インデックス`は `CartesianIndex`インデックスで決して呼び出されてはなりません。
    Fortunately, the scalar case is easily handled by first flattening the `CartesianIndex` arguments to plain integers.
    > 幸運なことにスカラの場合は、まず `CartesianIndex`引数を単純な整数に平坦化することで簡単に扱えます。
    Arrays of `CartesianIndex`, however, cannot be split apart into orthogonal pieces so easily.
    > しかし、「CartesianIndex」の配列は、それほど容易に直交する断片に分割することはできません。
    Before attempting to use `reindex`, `view` must ensure that there are no arrays of `CartesianIndex` in the argument list.
    > `reindex`を使う前に、` view`は引数リストに `CartesianIndex`の配列がないことを保証しなければなりません。
    If there are, it can simply "punt" by avoiding the `reindex` calculation entirely, constructing a nested `SubArray` with two levels of indirection instead.
    > 存在する場合、 `reindex`の計算を完全に避けて単純に「駄目」にして、代わりに2段階の間接指定を持つ入れ子の` SubArray`を構築することができます。
