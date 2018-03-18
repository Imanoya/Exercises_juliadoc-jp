# Talking to the compiler (the `:meta` mechanism)

<!-- EN -->
In some circumstances, one might wish to provide hints or instructions that a given block of code has special properties: you might always want to inline it, or you might want to turn on special compiler optimization passes.
> 場合によっては、特定のコードブロックに特殊なプロパティがあるというヒントや指示を提供したい場合があります。常にインライン化したい場合や、特別なコンパイラ最適化パスをオンにしたい場合があります。
<!-- EN -->
Starting with version 0.4, Julia has a convention that these instructions can be placed inside a `:meta` expression, which is typically (but not necessarily) the first expression in the body of a function.
> バージョン0.4以降、Juliaは、これらの命令を関数本体の最初の式である（必ずしも必要ではない） `：meta`式の中に置くことができるという規則を持っています。

<!-- EN -->
`:meta` expressions are created with macros.
> `：meta`式はマクロで作成されます。
<!-- EN -->
As an example, consider the implementation of the `@inline` macro:
> 例として、 `@inline`マクロの実装を考えてみましょう：

```julia
macro inline(ex)
    esc(isa(ex, Expr) ? pushmeta!(ex, :inline) : ex)
end
```

<!-- EN -->
Here, `ex` is expected to be an expression defining a function.
> ここで、 `ex`は関数を定義する式であることが期待されます。
<!-- EN -->
A statement like this:
> 次のようなステートメント

```julia
@inline function myfunction(x)
    x*(x+3)
end
```

<!-- EN -->
gets turned into an expression like this:
> 次のような表現に変わります：

```julia
quote
    function myfunction(x)
        Expr(:meta, :inline)
        x*(x+3)
    end
end
```

<!-- EN -->
`Base.pushmeta!(ex, :symbol, args...)` appends `:symbol` to the end of the `:meta` expression,
creating a new `:meta` expression if necessary.
> `Base.pushmeta！（ex、：symbol、args ...）`は `：meta`式の末尾に`：symbol`を追加し、
> 必要に応じて新しい `：meta`式を作成します。
<!-- EN -->
If `args` is specified, a nested expression containing `:symbol` and these arguments is appended instead, which can be used to specify additional information.
> `args`が指定された場合、`：symbol`とこれらの引数を含むネストされた式が代わりに追加され、追加情報を指定するために使用できます。

<!-- EN -->
To use the metadata, you have to parse these `:meta` expressions.
> メタデータを使用するには、これらの `：meta`式を解析する必要があります。
<!-- EN -->
If your implementation can be performed within Julia, `Base.popmeta!` is very handy: `Base.popmeta!(body, :symbol)` will scan a function *body* expression (one without the function signature) for the first `:meta` expression containing `:symbol`, extract any arguments, and return a tuple `(found::Bool, args::Array{Any})`.
> `Base.popmeta！（body、：symbol）`は、最初の `` base.popmeta！ ''のための関数* body *式（関数シグネチャなしのもの）をスキャンします。 `：symbol`を含む`：meta`式に引数を抽出し、 `（found :: Bool、args :: Array {Any}）のタプルを返します。
<!-- EN -->
If the metadata did not have any arguments, or `:symbol` was not found, the `args` array will be empty.
> メタデータに引数がない場合、または `：symbol`が見つからなかった場合、` args`配列は空になります。

<!-- EN -->
Not yet provided is a convenient infrastructure for parsing `:meta` expressions from C++.
> まだ提供されていないのは、C ++の `：meta`式を解析するための便利なインフラストラクチャです。