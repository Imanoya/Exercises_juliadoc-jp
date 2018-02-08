# Bounds checking

Like many modern programming languages, Julia uses bounds checking to ensure program safety when accessing arrays.
> 現代の多くのプログラミング言語と同様、ジュリアは配列にアクセスするときにプログラムの安全性を保証するために境界チェックを使用します。
In tight inner loops or other performance critical situations, you may wish to skip these bounds checks to improve runtime performance.
> タイトな内部ループやその他のパフォーマンス上重大な状況では、これらの境界チェックをスキップして実行時のパフォーマンスを向上させることができます。
For instance, in order to emit vectorized (SIMD) instructions, your loop body cannot contain branches, and thus cannot contain bounds checks.
> たとえば、ベクトル化（SIMD）命令を発行するには、ループ本体に分岐を含めることができないため、境界チェックを含めることはできません。
Consequently, Julia includes an `@inbounds(...)` macro to tell the compiler to skip such bounds checks within the given block.
> その結果、Juliaには `@inbounds(...)` マクロが含まれています。このマクロは、指定されたブロック内でそのような境界チェックをスキップするようにコンパイラに指示します。
User-defined array types can use the `@boundscheck(...)` macro to achieve context-sensitive code selection.
> ユーザ定義の配列型は `@boundscheck(...)` マクロを使用してコンテキスト依存のコード選択を行うことができます。

## Eliding bounds checks

The `@boundscheck(...)` macro marks blocks of code that perform bounds checking.
> `@boundscheck(...)` マクロは、境界チェックを実行するコードブロックをマークします。
When such blocks are inlined into an `@inbounds(...)` block, the compiler may remove these blocks.
> そのようなブロックが `@inbounds(...)` ブロックにインライン展開されると、コンパイラはこれらのブロックを削除することがあります。
The compiler removes the `@boundscheck` block *only if it is inlined* into the calling function.
> コンパイラは `@boundscheck` ブロック*を呼び出し関数にインライン展開した場合にのみ削除します*。
For example, you might write the method `sum` as:
> たとえば、 `sum` メソッドを以下のように書くことができます：

```julia
function sum(A::AbstractArray)
    r = zero(eltype(A))
    for i = 1:length(A)
        @inbounds r += A[i]
    end
    return r
end
```

With a custom array-like type `MyArray` having:
> カスタム配列のような型の `MyArray` は次のものを持っています：

```julia
@inline getindex(A::MyArray, i::Real) = (@boundscheck checkbounds(A,i); A.data[to_index(i)])
```

Then when `getindex` is inlined into `sum`, the call to `checkbounds(A,i)` will be elided.
> 次に `getindex` が `sum` にインライン展開されると、 `checkbounds(A,i)` の呼び出しは省略されます。
If your function contains multiple layers of inlining, only `@boundscheck` blocks at most one level of inlining deeper are eliminated.
> あなたの関数が複数の層のインライン化を含んでいる場合、インライン化の多くのレベルのうちの1つの `@boundscheck` ブロックだけが削除されます。
The rule prevents unintended changes in program behavior from code further up the stack.
> このルールは、プログラム動作の意図しない変更をスタックからさらに上に向かうコードから防ぎます。

## Propagating inbounds

There may be certain scenarios where for code-organization reasons you want more than one layer between the `@inbounds` and `@boundscheck` declarations.
> コード構成の理由から、 `@inbounds` 宣言と `@boundscheck` 宣言の間に複数のレイヤーが必要なシナリオがあるかもしれません。
For instance, the default `getindex` methods have the chain `getindex(A::AbstractArray, i::Real)` calls `getindex(IndexStyle(A), A, i)` calls `_getindex(::IndexLinear, A, i)`.
> たとえば、デフォルトの `getindex` メソッドは `getindex(A::AbstractArray, i::Real)` という連鎖をもち、 `getindex(IndexStyle(A), A,i)` は `_getindex(::IndexLinear,A,i)` 。

To override the "one layer of inlining" rule, a function may be marked with `@propagate_inbounds` to propagate an inbounds context (or out of bounds context) through one additional layer of inlining.
> "1層のインライン化"ルールをオーバーライドするために、インラインコンテキストの1つの追加層を介してインバウンドコンテキスト（または範囲外コンテキスト）を伝播するために、関数に `@production_inbounds` というマークを付けることができます。

## The bounds checking call hierarchy

The overall hierarchy is:

  * `checkbounds(A, I...)` which calls

      * `checkbounds(Bool, A, I...)` which calls

          * `checkbounds_indices(Bool, axes(A), I)` which recursively calls

              * `checkindex` for each dimension

Here `A` is the array, and `I` contains the "requested" indices.
> ここで `A` は配列、 `I` は `要求された 'インデックスを含んでいます。
`axes(A)` は `A` の"許可 "インデックスのタプルを返します。
> `axes(A)` returns a tuple of "permitted" indices of `A`.

`checkbounds(A, I...)` throws an error if the indices are invalid, whereas `checkbounds(Bool, A, I...)` returns `false` in that circumstance. 
> `checkbounds(Bool,A,I ...)` がその状況で `false` を返すのに対して、 `checkbounds(A,I ...)` はインデックスが無効であればエラーをスローします。
`checkbounds_indices` discards any information about the array other than its `indices` tuple, and performs a pure indices-vs-indices comparison: this allows relatively few compiled methods to serve a huge variety of array types.
> `checkbounds_indices` は、 `indices` タプル以外の配列に関する情報を破棄し、純粋なインデックスと vs インデックスの比較を実行します。これは、コンパイルされたメソッドが、多種多様な配列タイプに対応することを可能にします。
Indices are specified as tuples, and are usually compared in a 1-1 fashion with individual dimensions handled by calling another important function, `checkindex`: typically,
> インデックスはタプルとして指定され、通常、1-1の方法で、別の重要な関数 `checkindex` を呼び出すことによって扱われる個々の次元と比較されます：

```julia
checkbounds_indices(Bool, (IA1, IA...), (I1, I...)) = checkindex(Bool, IA1, I1) &
                                                      checkbounds_indices(Bool, IA, I)
```

so `checkindex` checks a single dimension.
> `checkindex` は単一の次元をチェックします。
All of these functions, including the unexported `checkbounds_indices` have docstrings accessible with `?` .
> これらの関数はすべて、非チェックの `checkbounds_indices` を含み、`?` でアクセス可能な docstring を持っています。

If you have to customize bounds checking for a specific array type, you should specialize `checkbounds(Bool, A, I...)`.
> 特定の配列型の境界チェックをカスタマイズする必要がある場合は、 `checkbounds（Bool、A、I ...）`を特化する必要があります。
However, in most cases you should be able to rely on `checkbounds_indices` as long as you supply useful `indices` for your array type.
しかし、大抵の場合、あなたの配列型に有用な `indices` を提供する限り、 `checkbounds_indices` に頼ることができます。

If you have novel index types, first consider specializing `checkindex`, which handles a single index for a particular dimension of an array.
> 新しい索引タイプがある場合は、まず、配列の特定の次元の単一の索引を処理する `checkindex`を特化してください。
If you have a custom multidimensional index type (similar to `CartesianIndex`), then you may have to consider specializing `checkbounds_indices`.
> カスタム多次元インデックス型( `CartesianIndex` に似ています)をお持ちの場合は、` checkbounds_indices`を特化しなければならないかもしれません。

Note this hierarchy has been designed to reduce the likelihood of method ambiguities.
> この階層は、メソッドのあいまいさの可能性を減らすように設計されていることに注意してください。
We try to make `checkbounds` the place to specialize on array type, and try to avoid specializations on index types; conversely, `checkindex` is intended to be specialized only on index type (especially, the last argument).
> `checkbounds`を配列型に特化し、インデックス型の特殊化を避けようとしています。逆に、 `checkindex`はインデックス型（特に最後の引数）のみを対象としています。
