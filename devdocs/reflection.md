# Reflection and introspection

Julia provides a variety of runtime reflection capabilities.
> Juliaは、さまざまなランタイム反映機能を提供します。

## Module bindings

The exported names for a `Module` are available using [`names(m::Module)`](@ref), which will return an array of [`Symbol`](@ref) elements representing the exported bindings.
> `Module`のエクスポートされた名前は、エクスポートされたバインディングを表す[`Symbol`](@ref)要素の配列を返す[`names(m::Module)`](@ref)を使って利用できます。
`names(m::Module, all = true)` returns symbols for all bindings in `m`, regardless of export status.
> `names(m::Module、all = true)`は、エクスポートステータスにかかわらず、 `m` のすべてのバインディングのシンボルを返します。

## DataType fields

The names of `DataType` fields may be interrogated using [`fieldnames`](@ref).
> `DataType`フィールドの名前は[fieldnames`](@ref)を使って調べることができます。
For example, given the following type, `fieldnames(Point)` returns an arrays of [`Symbol`](@ref) elements representing the field names:
> たとえば、以下の型があると、 `fieldnames(Point)`はフィールド名を表す[`Symbol`](@ref)要素の配列を返します：

```jldoctest struct_point
julia> struct Point
           x::Int
           y
       end

julia> fieldnames(Point)
2-element Array{Symbol,1}:
 :x
 :y
```

The type of each field in a `Point` object is stored in the `types` field of the `Point` variable itself:
> `Point`オブジェクトの各フィールドの型は` Point`変数自体の `types`フィールドに格納されます：

```jldoctest struct_point
julia> Point.types
svec(Int64, Any)
```

While `x` is annotated as an `Int`, `y` was unannotated in the type definition, therefore `y` defaults to the `Any` type.
> `x`は` Int`としてアノテーションされていますが、 `y`は型定義に注釈が付けられていないので、` y`のデフォルトは `Any`です。

Types are themselves represented as a structure called `DataType`:
> タイプ自体は `DataType`と呼ばれる構造体として表されます。

```jldoctest struct_point
julia> typeof(Point)
DataType
```

Note that `fieldnames(DataType)` gives the names for each field of `DataType` itself, and one of these fields is the `types` field observed in the example above.
> `fieldnames(DataType)` は `DataType` 自身の各フィールドの名前を与え、これらのフィールドの1つは上記の例で見られる `types` フィールドです。

## Subtypes

The *direct* subtypes of any `DataType` may be listed using [`subtypes`](@ref).
For example, the abstract `DataType` [`AbstractFloat`](@ref) has four (concrete) subtypes:
> 任意の `DataType` の*直接*サブタイプは、[`subtypes`](@ref)を使ってリストされます。
> 例えば、抽象的な `DataType` [`AbstractFloat`](@ref)には4つの具体的なサブタイプがあります：

```jldoctest
julia> subtypes(AbstractFloat)
4-element Array{Union{DataType, UnionAll},1}:
 BigFloat
 Float16
 Float32
 Float64
```

Any abstract subtype will also be included in this list, but further subtypes thereof will not; recursive application of [`subtypes`](@ref) may be used to inspect the full type tree.
> すべての抽象サブタイプもこのリストに含まれますが、それ以上のサブタイプは含まれません。 [`subtypes](@ref)を再帰的に適用すると、フルタイプのツリーを調べることができます。

## DataType layout

The internal representation of a `DataType` is critically important when interfacing with C code and several functions are available to inspect these details.
> `DataType`の内部表現は、Cコードとインターフェースする際に非常に重要です。これらの詳細を調べるためにいくつかの関数が利用できます。
[`isbits(T::DataType)`](@ref) returns true if `T` is stored with C-compatible alignment. [`fieldoffset(T::DataType, i::Integer)`](@ref) returns the (byte) offset for field *i* relative to the start of the type.
> [`isbits(T::DataType)`](@ref)は` T`がC互換アラインメントで格納されている場合にtrueを返します。 [`fieldoffset(T::DataType, i::Integer)`](@ref)は、型の先頭からのフィールド *i* の（バイト）オフセットを返します。

## Function methods

The methods of any generic function may be listed using [`methods`](@ref). The method dispatch table may be searched for methods accepting a given type using [`methodswith`](@ref).
> 任意の汎用関数のメソッドは、[`methods`](@ref)を使ってリストすることができます。 メソッドディスパッチテーブルは、[`methodswith`](@ref)を使って与えられた型を受け入れるメソッドを検索することができます。

## Expansion and lowering

As discussed in the [Metaprogramming](@ref) section, the [`macroexpand`](@ref) function gives the unquoted and interpolated expression (`Expr`) form for a given macro.
> [Metaprogramming](@ref)セクションで説明したように、[`macroexpand`](@ref)関数は、与えられたマクロの引用符で囲まれていない式（` `Expr`）を返します。
To use `macroexpand`, `quote` the expression block itself (otherwise, the macro will be evaluated and the result will be passed instead!).
> `macroexpand`を使うには、式ブロック自体を` quote`します（そうでなければ、マクロが評価され、代わりに結果が渡されます）。
For example:
> 例えば：

```jldoctest
julia> macroexpand(@__MODULE__, :(@edit println("")) )
:((Base.edit)(println, (Base.typesof)("")))
```

The functions `Base.Meta.show_sexpr` and [`dump`](@ref) are used to display S-expr style views and depth-nested detail views for any expression.
> `Base.Meta.show_sexpr`と[`dump`](@ref)関数は、S-expr形式のビューと深さにネストされた詳細ビューを表示するために使われます。

Finally, the [`Meta.lower`](@ref) function gives the `lowered` form of any expression and is of particular interest for understanding both macros and top-level statements such as function declarations and variable assignments:
> 最後に、[`Meta.lower`](@ref)関数は式の` `低下した形を与え、関数宣言や変数代入などのマクロとトップレベルのステートメントの両方を理解するのに特に重要です：

```jldoctest
julia> Meta.lower(@__MODULE__, :(f() = 1) )
:(begin
        $(Expr(:method, :f))
        $(Expr(:method, :f, :((Core.svec)((Core.svec)((Core.Typeof)(f)), (Core.svec)())), CodeInfo(:(begin
        #= none:1 =#
        return 1
    end)), false))
        return f
    end)
```

## Intermediate and compiled representations

Inspecting the lowered form for functions requires selection of the specific method to display, because generic functions may have many methods with different type signatures.
> 汎用関数には異なる型シグネチャを持つ多くのメソッドがあるため、関数の下位型を調べるには、表示する特定のメソッドを選択する必要があります。
For this purpose, method-specific code-lowering is available using [`code_lowered(f::Function, (Argtypes...))`](@ref), and the type-inferred form is available using [`code_typed(f::Function, (Argtypes...))`](@ref).
> この目的のために、メソッド固有のコード下げは、[`code_lowered（f :: Function、（Argtypes ...））]]（@ ref）を使用して利用でき、型推論された形式は[` code_typed :: Function、（Argtypes ...）） `]（@ ref）。
[`code_warntype(f::Function, (Argtypes...))`](@ref) adds highlighting to the output of [`code_typed`](@ref) (see [`@code_warntype`](@ref)).
> [`code_warntype（f :: Function、（Argtypes ...））]]（@ ref）は[` code_typed`]（@ ref）の出力にハイライトを追加します（[`@ code_warntype`]（@ ref）参照） 。

Closer to the machine, the LLVM intermediate representation of a function may be printed using by [`code_llvm(f::Function, (Argtypes...))`](@ref), and finally the compiled machine code is available using [`code_native(f::Function, (Argtypes...))`](@ref) (this will trigger JIT compilation/code generation for any function which has not previously been called).
> マシンに近づくと、関数のLLVM中間表現は、 `` code_llvm（f :: Function、（Argtypes ...）） `]（@ ref）によって出力され、最後にコンパイルされたマシンコードは[ `` code_native（f :: Function、（Argtypes ...）） `]（@ ref）（これは以前呼び出されていない関数のJITコンパイル/コード生成を引き起こします）。

For convenience, there are macro versions of the above functions which take standard function calls and expand argument types automatically:
> 便宜上、上記の関数のマクロバージョンがあり、これらは標準の関数呼び出しを受け取り、引数型を自動的に展開します：

```julia-repl
julia> @code_llvm +(1,1)

; Function Attrs: sspreq
define i64 @"julia_+_130862"(i64, i64) #0 {
top:
    %2 = add i64 %1, %0, !dbg !8
    ret i64 %2, !dbg !8
}
```

(likewise `@code_typed`, `@code_warntype`, `@code_lowered`, and `@code_native`)
> (同様に `@code_typed`, `@code_warntype`, `@code_lowered`, and `@code_native`)
 
