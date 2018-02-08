# Base.Cartesian

The (non-exported) Cartesian module provides macros that facilitate writing multidimensional algorithms.
> （エクスポートされていない）デカルト・モジュールは、多次元アルゴリズムの作成を容易にするマクロを提供します。
It is hoped that Cartesian will not, in the long term, be necessary; however, at present it is one of the few ways to write compact and performant multidimensional code.
> デカルトは、長期的には必要ではないと期待されている。 しかし、現時点では、コンパクトで実績のある多次元コードを記述する数少ない方法の1つです。

## Principles of usage

A simple example of usage is:
> 簡単な使用例は次のとおりです。

```julia
@nloops 3 i A begin
    s += @nref 3 A i
end
```

which generates the following code:
> 次のコードを生成します。

```julia
for i_3 = 1:size(A,3)
    for i_2 = 1:size(A,2)
        for i_1 = 1:size(A,1)
            s += A[i_1,i_2,i_3]
        end
    end
end
```

In general, Cartesian allows you to write generic code that contains repetitive elements, like the nested loops in this example.
> 一般に、デカルトでは、この例のネストされたループのような反復要素を含む汎用コードを記述することができます。
Other applications include repeated expressions (e.g., loop unwinding) or creating function calls with variable numbers of arguments without using the "splat" construct (`i...`).
> 他のアプリケーションには、 "splat"構造体（ `i ...`）を使用せずに、反復式（例えば、ループアンワインド）や引数の可変数による関数呼び出しの作成が含まれます。

## Basic syntax

The (basic) syntax of `@nloops` is as follows:
>`@ nloops`の（基本的な）構文は次のとおりです：

  * The first argument must be an integer (*not* a variable) specifying the number of loops.
  > *最初の引数は、ループの数を指定する整数（*変数ではなく）でなければなりません。
  * The second argument is the symbol-prefix used for the iterator variable. Here we used `i`, and variables `i_1, i_2, i_3` were generated.
  > * 2番目の引数は、イテレータ変数に使用されるシンボルプレフィックスです。ここでは `i`を使用し、変数` i_1、i_2、i_3`が生成されました。
  * The third argument specifies the range for each iterator variable.
    If you use a variable (symbol) here, it's taken as `1:size(A,dim)`.
    More flexibly, you can use the anonymous-function expression syntax described below.
  > * 3番目の引数は、各イテレータ変数の範囲を指定します。
  >   ここで変数（シンボル）を使用すると、 `1：size（A、dim）`とみなされます。
  >   より柔軟に、以下で説明する無名関数式構文を使用することができます。
  * The last argument is the body of the loop. Here, that's what appears between the `begin...end`.
  > *最後の引数はループの本体です。これは、 `begin ... end`の間に現れるものです。

There are some additional features of `@nloops` described in the [reference section](@ref dev-cartesian-reference).
> [参考セクション]（@ ref dev-cartesian-reference）で説明した `@ nloops`のいくつかの追加機能があります。

`@nref` follows a similar pattern, generating `A[i_1,i_2,i_3]` from `@nref 3 A i`.
> `@nref`は` @nref3Ai` から `A [i_1,i_2,i_3]` を生成する同様のパターンに従います。
The general practice is to read from left to right, which is why `@nloops` is `@nloops 3 i A expr` (as in `for i_2 = 1:size(A,2)`, where `i_2` is to the left and the range is to the right) whereas `@nref` is `@nref 3 A i` (as in `A[i_1,i_2,i_3]`, where the array comes first).
> 一般的な習慣は左から右へ読むことです。なぜなら、 `@nloops` は `@nloops i A expr` (`for i_2 = 1：size(A,2)` のように、 `i_2` は左側にあり、範囲は右側にあります)であり、`@nref`は` @nref 3 A i` (配列が最初に来る `A [i_1、i_2、i_3]`のように）です。

If you're developing code with Cartesian, you may find that debugging is easier when you examine the generated code, using `@macroexpand`:
> Cartesianでコードを開発している場合、 `@macroexpand`を使って、生成されたコードを調べるとデバッグが簡単になることがあります。

```@meta
DocTestSetup = quote
    import Base.Cartesian: @nref
end
```

```jldoctest
julia> @macroexpand @nref 2 A i
:(A[i_1, i_2])
```

```@meta
DocTestSetup = nothing
```

### Supplying the number of expressions

The first argument to both of these macros is the number of expressions, which must be an integer.
> これらのマクロの両方への最初の引数は、式の数です。これは整数でなければなりません。
When you're writing a function that you intend to work in multiple dimensions, this may not be something you want to hard-code.
> 複数の次元で作業する予定の関数を記述するときは、ハードコーディングする必要がありません。
If you're writing code that you need to work with older Julia versions, currently you should use the `@ngenerate` macro described in [an older version of this documentation](https://docs.julialang.org/en/release-0.3/devdocs/cartesian/#supplying-the-number-of-expressions).
> 以前のJuliaバージョンで作業する必要があるコードを書く場合は、現在、[このドキュメントの古いバージョン]で説明されている `@ ngenerate`マクロを使用する必要があります（https://docs.julialang.org/en/release -0.3 / devdocs / cartesian /＃-number-of-expressions）を指定します。

Starting in Julia 0.4-pre, the recommended approach is to use a `@generated function`.
> Julia 0.4-preから始めると、推奨される方法は `@generated function`を使うことです。
Here's an example:
> ここに例があります：

```julia
@generated function mysum(A::Array{T,N}) where {T,N}
    quote
        s = zero(T)
        @nloops $N i A begin
            s += @nref $N A i
        end
        s
    end
end
```

Naturally, you can also prepare expressions or perform calculations before the `quote` block.
> もちろん、式を準備したり、 `quote`ブロックの前に計算を行うこともできます。

### Anonymous-function expressions as macro arguments

Perhaps the single most powerful feature in `Cartesian` is the ability to supply anonymous-function expressions that get evaluated at parsing time.
> おそらく、「デカルト」の最も強力な機能は、解析時に評価される無名関数式を提供する能力です。
Let's consider a simple example:
> 簡単な例を考えてみましょう：

```julia
@nexprs 2 j->(i_j = 1)
```

`@nexprs` generates `n` expressions that follow a pattern. This code would generate the following statements:
> `@nexprs`はパターンに続く` n`式を生成します。 このコードは次の文を生成します：

```julia
i_1 = 1
i_2 = 1
```

In each generated statement, an "isolated" `j` (the variable of the anonymous function) gets replaced by values in the range `1:2`.
> 生成された各ステートメントでは、 "分離された" `j`（無名関数の変数）が` 1：2`の範囲の値に置き換えられます。
Generally speaking, Cartesian employs a LaTeX-like syntax.
> 一般に、デカルトはLaTeXのような構文を採用しています。
This allows you to do math on the index `j`.
> これは、あなたがインデックス `j`で数学をすることを可能にします。
Here's an example computing the strides of an array:
> 配列のストライドを計算する例を次に示します。

```julia
s_1 = 1
@nexprs 3 j->(s_{j+1} = s_j * size(A, j))
```

would generate expressions
> 式を生成する

```julia
s_1 = 1
s_2 = s_1 * size(A, 1)
s_3 = s_2 * size(A, 2)
s_4 = s_3 * size(A, 3)
```

Anonymous-function expressions have many uses in practice.
> 匿名関数式は実際には多くの用途があります。

#### [Macro reference](@id dev-cartesian-reference)

```@docs
Base.Cartesian.@nloops
Base.Cartesian.@nref
Base.Cartesian.@nextract
Base.Cartesian.@nexprs
Base.Cartesian.@ncall
Base.Cartesian.@ntuple
Base.Cartesian.@nall
Base.Cartesian.@nany
Base.Cartesian.@nif
```
