<!-- Start -->
# [Performance Tips](@id man-performance-tips)
> # [パフォーマンスのヒント](@ idのman-performance-tips)
<!-- End -->
<!-- Start -->
In the following sections, we briefly go through a few techniques that can help make your Julia code run as fast as possible.
> 次のセクションでは、Juliaコードをできるだけ速く動かすのに役立ついくつかのテクニックを簡単に説明します。
<!-- End -->

<!-- Start -->
## Avoid global variables
## グローバル変数を避ける
<!-- End -->

<!-- Start -->
A global variable might have its value, and therefore its type, change at any point. 
>グローバル変数は、その値を持つ可能性があり、その型はいつでも変更できます。
<!-- End -->
<!-- Start -->
This makes it difficult for the compiler to optimize code using global variables. Variables should be local, or passed as arguments to functions, whenever possible.
> これにより、コンパイラがグローバル変数を使用してコードを最適化することが困難になります。 可能であれば、変数はローカルであるか、関数の引数として渡されます。
<!-- End -->

<!-- Start -->
Any code that is performance critical or being benchmarked should be inside a function.
> パフォーマンス上重要であるか、またはベンチマークされているコードは、関数内にある必要があります。
<!-- End -->

<!-- Start -->
We find that global names are frequently constants, and declaring them as such greatly improves performance:
> グローバル名は定数であることが多く、そのように宣言するとパフォーマンスが大幅に向上します。
<!-- End -->

```julia
const DEFAULT_VAL = 0
```

<!-- Start -->
Uses of non-constant globals can be optimized by annotating their types at the point of use:
> 定数でないグローバルの使用は、使用時に型を注釈することで最適化することができます。
<!-- End -->

```julia
global x
y = f(x::Int + 1)
```

<!-- Start -->
Writing functions is better style. 
> 関数を書く方が良いスタイルです。
<!-- End -->
<!-- Start -->
It leads to more reusable code and clarifies what steps are being done, and what their inputs and outputs are.
> より再利用可能なコードにつながり、どのステップが実行されているか、そしてその入力と出力が何であるかを明確にします。
<!-- End -->

<!-- Start -->
!!! note
!!! 注意
<!-- End -->

   All code in the REPL is evaluated in global scope, so a variable defined and assigned at toplevel will be a **global** variable.
   > REPLのすべてのコードはグローバルスコープで評価されるため、トップレベルで定義され割り当てられた変数は **グローバル**変数になります。

<!-- Start -->
In the following REPL session:
> 次のREPLセッションでは、
<!-- End -->

```julia-repl
julia> x = 1.0
```

is equivalent to:

```julia-repl
julia> global x = 1.0
```

<!-- Start -->
so all the performance issues discussed previously apply.
> 以前に説明したすべてのパフォーマンスの問題が適用されます。
<!-- End -->

<!-- Start -->
## Measure performance with [`@time`](@ref) and pay attention to memory allocation
> ## [`@time`](@ref)でパフォーマンスを測定し、メモリ割り当てに注意してください
<!-- End -->

<!-- Start -->
A useful tool for measuring performance is the [`@time`](@ref) macro. The following example illustrates good working style:
> パフォーマンスを測定するのに便利なツールは [`@time`](@ref) マクロです。 次の例は、良い作業スタイルを示しています。
<!-- End -->

```julia-repl
julia> function f(n)
           s = 0
           for i = 1:n
               s += i/2
           end
           s
       end
f (generic function with 1 method)

julia> @time f(1)
  0.012686 seconds (2.09 k allocations: 103.421 KiB)
0.5

julia> @time f(10^6)
  0.021061 seconds (3.00 M allocations: 45.777 MiB, 11.69% gc time)
2.5000025e11
```

<!-- Start -->
On the first call (`@time f(1)`), `f` gets compiled.  
> 最初の呼び出し (`@time f(1)`) では、 `f` がコンパイルされます。
<!-- End -->
<!-- Start -->
(If you've not yet used [`@time`](@ref) in this session, it will also compile functions needed for timing.)  
> (このセッションでまだ [`@time`](@ref) を使用していない場合は、タイミングに必要な関数もコンパイルされます)。
<!-- End -->
<!-- Start -->
You should not take the results of this run seriously. 
> この実行の結果を真剣に受け止めてはいけません。
<!-- End -->
<!-- Start -->
For the second run, note that in addition to reporting the time, it also indicated that a large amount of memory was allocated. 
> 2回目の実行では、時間の報告に加えて、大量のメモリが割り当てられていることも示しています。
<!-- End -->
<!-- Start -->
This is the single biggest advantage of [`@time`](@ref) vs. functions like [`tic()`](@ref) and [`toc()`](@ref), which only report time.
> これは[`tic()`](@ref) や [`toc()`](@ref) のような関数と比べて [`@time`](@ref) の最大の利点です。
<!-- End -->

<!-- Start -->
Unexpected memory allocation is almost always a sign of some problem with your code, usually a problem with type-stability. 
> 予期しないメモリ割り当ては、ほとんどの場合、コードに問題があることの兆候であり、通常は型安定性の問題です。
<!-- End -->
<!-- Start -->
Consequently, in addition to the allocation itself, it's very likely that the code generated for your function is far from optimal. 
> その結果、割り当て自体に加えて、あなたの関数用に生成されたコードは最適ではない可能性が非常に高いです。
<!-- End -->
<!-- Start -->
Take such indications seriously and follow the advice below.
<!-- End -->
> そのような兆候を真剣に受けて、以下の助言に従ってください。

<!-- Start -->
For more serious benchmarking, consider the [BenchmarkTools.jl](https://github.com/JuliaCI/BenchmarkTools.jl) package which evaluates the function multiple times in order to reduce noise.
> より深刻なベンチマークでは、ノイズを減らすために関数を複数回評価する [BenchmarkTools.jl](https://github.com/JuliaCI/BenchmarkTools.jl) パッケージを検討してください。
<!-- End -->

<!-- Start -->
As a teaser, an improved version of this function allocates no memory (the allocation reported below is due to running the `@time` macro in global scope) and has an order of magnitude faster execution after the first call:
> ティーザーとして、この関数の改良版はメモリを割り当てません(以下に報告される割り当てはグローバルスコープで `@time` マクロを実行するためである)、最初の呼び出し後には一段速く実行されます：
<!-- End -->

```julia-repl
julia> @time f_improved(1)
  0.007008 seconds (1.32 k allocations: 63.640 KiB)
0.5

julia> @time f_improved(10^6)
  0.002997 seconds (6 allocations: 192 bytes)
2.5000025e11
```

<!-- Start -->
Below you'll learn how to spot the problem with `f` and how to fix it.
> 以下で `f` で問題を見つけ出す方法とそれを修正する方法を学びます。
<!-- End -->

<!-- Start -->
In some situations, your function may need to allocate memory as part of its operation, and this can complicate the simple picture above. 
> 状況によっては、関数がその操作の一部としてメモリを割り当てる必要があり、これが上記の単純な画像を複雑にする可能性があります。
<!-- End -->
<!-- Start -->
In such cases, consider using one of the [tools](@ref tools) below to diagnose problems, or write a version of your function that separates allocation from its algorithmic aspects (see [Pre-allocating outputs](@ref)).
> そのような場合は、以下の [tools](@ref tools) のいずれかを使用して問題を診断するか、アルゴリズムの側面から割り振りを分ける関数のバージョンを記述することを検討してください( [Pre-allocating outputs](@ref) を参照 )。
<!-- End -->

<!-- Start -->
## [Tools](@id tools)
> ## [ツール](@id tools)
<!-- End -->

<!-- Start -->
Julia and its package ecosystem includes tools that may help you diagnose problems and improve the performance of your code:
> ジュリアとそのパッケージエコシステムには、問題の診断とコードのパフォーマンス向上に役立つツールが含まれています。
<!-- End -->

<!-- Start -->
* [Profiling](@ref) allows you to measure the performance of your running code and identify lines that serve as bottlenecks.  
* > [プロファイリング](@ref) では、実行中のコードのパフォーマンスを測定し、ボトルネックとなる行を識別することができます。
<!-- End -->
<!-- Start -->
* For complex projects, the [ProfileView](https://github.com/timholy/ProfileView.jl) package can help you visualize your profiling results.
* > 複雑なプロジェクトの場合、[ProfileView](https://github.com/timholy/ProfileView.jl)パッケージは、プロファイリングの結果を視覚化するのに役立ちます。
<!-- End -->
<!-- Start -->
* Unexpectedly-large memory allocations--as reported by [`@time`](@ref), [`@allocated`](@ref), or the profiler (through calls to the garbage-collection routines)--hint that there might be issues with your code.
* > [`@time`](@ref), [`@allocated`](@ref), またはプロファイラ(ガベージコレクションルーチンへの呼び出しを介して)によって予期せずに大量のメモリ割り当てが発生する - あなたのコードに問題があるかもしれません。
<!-- End -->
<!-- Start -->
If you don't see another reason for the allocations, suspect a type problem. 
   割り当てに別の理由がない場合は、タイプの問題が考えられます。
<!-- End -->
<!-- Start -->
* You can also start Julia with the `--track-allocation=user` option and examine the resulting `*.mem` files to see information about where those allocations occur.  
* > また、 `--track-allocation = user` オプションでJuliaを起動し、結果の `*.mem` ファイルを調べて、その割り当てがどこにあるかについての情報を見ることができます。
<!-- End -->
<!-- Start -->
* See [Memory allocation analysis](@ref).
* > [メモリ割り当て分析](@ref) を参照してください。
<!-- End -->
<!-- Start -->
* `@code_warntype` generates a representation of your code that can be helpful in finding expressions that result in type uncertainty. See [`@code_warntype`](@ref) below.
* > `@code_warntype` は、型の不確定性をもたらす式を見つけるのに役立つコードの表現を生成します。以下の [`@ code_warntype`](@ref) を参照してください。
<!-- End -->
   <!-- Start -->
* The [Lint](https://github.com/tonyhffong/Lint.jl) package can also warn you of certain types of programming errors.
* > [Lint](https://github.com/tonyhffong/Lint.jl) パッケージは、特定のタイプのプログラミングエラーについて警告することもできます。
<!-- End -->

<!-- Start -->
## Avoid containers with abstract type parameters
> ## 抽象型パラメータを持つコンテナを避ける
<!-- End -->

<!-- Start -->
When working with parameterized types, including arrays, it is best to avoid parameterizing with abstract types where possible.
> 配列を含むパラメータ化された型で作業する場合は、可能であれば抽象型でパラメータ化するのは避けるのが最善です。
<!-- End -->

<!-- Start -->
Consider the following:
> 次の点を考慮してください。
<!-- End -->

```julia
a = Real[]
# typeof(a) = Array{Real,1}
if (f = rand()) < .8
    push!(a, f)
end
```

<!-- Start -->
Because `a` is a an array of abstract type [`Real`](@ref), it must be able to hold any `Real` value.  
> `a`は抽象型[` Real`](@ref)の配列であるため、 `Real`値を保持できる必要があります。
<!-- End -->
<!-- Start -->
Since `Real` objects can be of arbitrary size and structure, `a` must be represented as an array of pointers to individually allocated `Real` objects. 
> `Real`オブジェクトは任意のサイズと構造を取ることができるので、` a`は個々に割り当てられた `Real`オブジェクトへのポインタの配列として表現されなければなりません。
<!-- End -->
<!-- Start -->
Because `f` will always be a [`Float64`](@ref), we should instead, use:
> `f` は常に [`Float64`](@ref) になるので、代わりに次のように使うべきです：
<!-- End -->

```julia
a = Float64[]
# typeof(a) = Array{Float64,1}
```

<!-- Start -->
which will create a contiguous block of 64-bit floating-point values that can be manipulated efficiently.
> 効率的に操作できる64ビット浮動小数点値の連続したブロックが作成されます。
<!-- End -->

<!-- Start -->
See also the discussion under [Parametric Types](@ref).
> [パラメトリック型](@ref) の説明も参照してください。
<!-- End -->

<!-- Start -->
## Type declarations
> ## 型宣言
<!-- End -->

<!-- Start -->
In many languages with optional type declarations, adding declarations is the principal way to make code run faster. 
> オプションの型宣言を使用する多くの言語で、宣言を追加することは、コードをより高速に実行するための主要な方法です。
<!-- End -->
<!-- Start -->
This is *not* the case in Julia. 
> juliaの場合、そうでは*ありません*
<!-- End -->
<!-- Start -->
In Julia, the compiler generally knows the types of all function arguments, local variables, and expressions. However, there are a few specific instances where declarations are helpful.
> Juliaでは、コンパイラは一般にすべての関数引数、ローカル変数、および式の型を知っています。 しかし、宣言が有用ないくつかの具体的な例があります。
<!-- End -->

<!-- Start -->
### Avoid fields with abstract type
> ### 抽象型のフィールドを避ける
<!-- End -->

<!-- Start -->
Types can be declared without specifying the types of their fields:
> 型は、フィールドの型を指定せずに宣言できます。
<!-- End -->

```jldoctest myambig
julia> struct MyAmbiguousType
           a
       end
```

<!-- Start -->
This allows `a` to be of any type. 
> これにより、 `a` はどんな型でも構いません。
<!-- End -->
<!-- Start -->
This can often be useful, but it does have a downside: for objects of type `MyAmbiguousType`, the compiler will not be able to generate high-performance code.  
> これはしばしば便利ですが、欠点があります： `MyAmbiguousType` 型のオブジェクトの場合、コンパイラは高性能コードを生成できません。
<!-- End -->
<!-- Start -->
The reason is that the compiler uses the types of objects, not their values, to determine how to build code. 
> その理由は、コンパイラは値を作成するのではなく、オブジェクトの型を使用してコードを作成する方法を決定するからです。
<!-- End -->
<!-- Start -->
Unfortunately, very little can be inferred about an object of type `MyAmbiguousType`:
> 残念ながら、 `MyAmbiguousType` 型のオブジェクトについてはほとんど推測できません：
<!-- End -->

```jldoctest myambig
julia> b = MyAmbiguousType("Hello")
MyAmbiguousType("Hello")

julia> c = MyAmbiguousType(17)
MyAmbiguousType(17)

julia> typeof(b)
MyAmbiguousType

julia> typeof(c)
MyAmbiguousType
```

<!-- Start -->
`b` and `c` have the same type, yet their underlying representation of data in memory is very different. 
> `b`と `c` は同じ型を持っていますが、メモリ内のデータの基底表現は非常に異なります。
<!-- End -->
<!-- Start -->
Even if you stored just numeric values in field `a`, the fact that the memory representation of a [`UInt8`](@ref) differs from a [`Float64`](@ref) also means that the CPU needs to handle them using two different kinds of instructions. 
> フィールド `a` に数値だけを格納しても、[`UInt8`](@ref) のメモリ表現が [`Float64`](@ref) と異なるということは、CPU が それらは2つの異なる種類の命令を使用します。
<!-- End -->
<!-- Start -->
Since the required information is not available in the type, such decisions have to be made at run-time. 
> 必要な情報はタイプで利用できないため、このような決定は実行時に行わなければなりません。
<!-- End -->
<!-- Start -->
This slows performance.
> これはパフォーマンスを低下させます。
<!-- End -->

<!-- Start -->
You can do better by declaring the type of `a`. Here, we are focused on the case where `a` might be any one of several types, in which case the natural solution is to use parameters. 
> `a` の型を宣言すれば、よりうまくいくでしょう。 ここでは、`a` がいくつかの型のいずれかである場合に焦点を合わせます。その場合、自然な解決策はパラメータを使用することです。
<!-- End -->
<!-- Start -->
For example:
> 例えば：
<!-- End -->

```jldoctest myambig2
julia> mutable struct MyType{T<:AbstractFloat}
           a::T
       end
```

This is a better choice than

```jldoctest myambig2
julia> mutable struct MyStillAmbiguousType
           a::AbstractFloat
       end
```

<!-- Start -->
because the first version specifies the type of `a` from the type of the wrapper object.  
> なぜなら最初のバージョンはラッパーオブジェクトの型から `a`の型を指定するからです。
<!-- End -->
<!-- Start -->
For example:
> 例えば：
<!-- End -->

```jldoctest myambig2
julia> m = MyType(3.2)
MyType{Float64}(3.2)

julia> t = MyStillAmbiguousType(3.2)
MyStillAmbiguousType(3.2)

julia> typeof(m)
MyType{Float64}

julia> typeof(t)
MyStillAmbiguousType
```

<!-- Start -->
The type of field `a` can be readily determined from the type of `m`, but not from the type of `t`.  
> フィールド `a` のタイプは、 `m` のタイプから容易に決定されるが、 `t` のタイプからは決定されない。
<!-- End -->
<!-- Start -->
#Indeed, in `t` it's possible to change the type of field `a`:
実際、 `t`では、フィールド` a`の型を変更することができます：
<!-- End -->

```jldoctest myambig2
julia> typeof(t.a)
Float64

julia> t.a = 4.5f0
4.5f0

julia> typeof(t.a)
Float32
```

<!-- Start -->
In contrast, once `m` is constructed, the type of `m.a` cannot change:
> 対照的に、 `m` が構築されると `m.a` の型は変更できません：
<!-- End -->

```jldoctest myambig2
julia> m.a = 4.5f0
4.5f0

julia> typeof(m.a)
Float64
```

<!-- Start -->
The fact that the type of `m.a` is known from `m`'s type--coupled with the fact that its type cannot change mid-function--allows the compiler to generate highly-optimized code for objects like `m` but not for objects like `t`.
> `m` 型が `m` 型から知られているという事実と、その型が中間関数を変更できないという事実は、コンパイラが `m` のようなオブジェクトに対して高度に最適化されたコードを生成することを可能にします。 `t` のようなオブジェクトではありません。
<!-- End -->

<!-- Start -->
Of course, all of this is true only if we construct `m` with a concrete type.  
> もちろん、これはすべて、具体的な型で `m` を構築する場合にのみ当てはまります。
<!-- End -->
<!-- Start -->
We can break this by explicitly constructing it with an abstract type:
> 抽象型で明示的に構築することで、これを解消できます。
<!-- End -->

```jldoctest myambig2
julia> m = MyType{AbstractFloat}(3.2)
MyType{AbstractFloat}(3.2)

julia> typeof(m.a)
Float64

julia> m.a = 4.5f0
4.5f0

julia> typeof(m.a)
Float32
```

<!-- Start -->
For all practical purposes, such objects behave identically to those of `MyStillAmbiguousType`.
> すべての実際的な目的のために、そのようなオブジェクトは `MyStillAmbiguousType` のものと同じように動作します。
<!-- End -->

<!-- Start -->
It's quite instructive to compare the sheer amount code generated for a simple function
> 簡単な関数のために生成された完全な量のコードを比較することは非常に有益です
<!-- End -->

```julia
func(m::MyType) = m.a+1
```

<!-- Start -->
using
> を使って
<!-- End -->

```julia
code_llvm(func,Tuple{MyType{Float64}})
code_llvm(func,Tuple{MyType{AbstractFloat}})
code_llvm(func,Tuple{MyType})
```

<!-- Start -->
For reasons of length the results are not shown here, but you may wish to try this yourself. 
> 長さの理由から結果はここには表示されませんが、あなた自身で試してみることをお勧めします。
<!-- End -->
<!-- Start -->
Because the type is fully-specified in the first case, the compiler doesn't need to generate any code to resolve the type at run-time. 
> 最初のケースでは型が完全指定されているため、コンパイラは実行時に型を解決するコードを生成する必要はありません。
<!-- End -->
<!-- Start -->
This results in shorter and faster code.
> その結果、コードが短くて高速になります。
<!-- End -->

<!-- Start -->
### Avoid fields with abstract containers
> ### 抽象コンテナを持つフィールドを避ける
<!-- End -->

<!-- Start -->
The same best practices also work for container types:
> 同様のベストプラクティスもコンテナタイプで使用できます。
<!-- End -->

```jldoctest containers
julia> mutable struct MySimpleContainer{A<:AbstractVector}
           a::A
       end

julia> mutable struct MyAmbiguousContainer{T}
           a::AbstractVector{T}
       end
```

<!-- Start -->
For example:
> 例えば:
<!-- End -->

```jldoctest containers
julia> c = MySimpleContainer(1:3);

julia> typeof(c)
MySimpleContainer{UnitRange{Int64}}

julia> c = MySimpleContainer([1:3;]);

julia> typeof(c)
MySimpleContainer{Array{Int64,1}}

julia> b = MyAmbiguousContainer(1:3);

julia> typeof(b)
MyAmbiguousContainer{Int64}

julia> b = MyAmbiguousContainer([1:3;]);

julia> typeof(b)
MyAmbiguousContainer{Int64}
```
<!-- Start -->
For `MySimpleContainer`, the object is fully-specified by its type and parameters, so the compiler can generate optimized functions. 
> `MySimpleContainer` の場合、オブジェクトは型とパラメータによって完全に指定されるため、コンパイラは最適化された関数を生成できます。
<!-- End -->
<!-- Start -->
In most instances, this will probably suffice.
> ほとんどの場合、これで十分でしょう。
<!-- End -->

<!-- Start -->
While the compiler can now do its job perfectly well, there are cases where *you* might wish that your code could do different things depending on the *element type* of `a`.  
> コンパイラは今、その仕事を完璧に行うことができますが、 `a` の *要素の型* に応じて、あなたのコードが異なることを望むかもしれない場合があります。
<!-- End -->
<!-- Start -->
Usually the best way to achieve this is to wrap your specific operation (here, `foo`) in a separate function:
> これを実現する最も良い方法は、あなたの特定の操作 (ここでは `foo` ) を別の関数にラップすることです:
<!-- End -->

```julia jldoctest containers
julia> function sumfoo(c::MySimpleContainer)
           s = 0
           for x in c.a
               s += foo(x)
           end
           s
       end
sumfoo (generic function with 1 method)

julia> foo(x::Integer) = x
foo (generic function with 1 method)

julia> foo(x::AbstractFloat) = round(x)
foo (generic function with 2 methods)
```

<!-- Start -->
This keeps things simple, while allowing the compiler to generate optimized code in all cases.
> これにより、コンパイル時に最適化されたコードを生成できるようになります。
<!-- End -->

<!-- Start -->
However, there are cases where you may need to declare different versions of the outer function for different element types of `a`. 
> しかし、異なる要素型の `a` に対して異なるバージョンの外部関数を宣言する必要がある場合があります。
<!-- End -->
<!-- Start -->
You could do it like this:
> あなたはこれを次のようにすることができます：
<!-- End -->

```
function myfun(c::MySimpleContainer{Vector{T}}) where T<:AbstractFloat
    ...
end
function myfun(c::MySimpleContainer{Vector{T}}) where T<:Integer
    ...
end
```

<!-- Start -->
This works fine for `Vector{T}`, but we'd also have to write explicit versions for `UnitRange{T}` or other abstract types. 
> これは `Vector {T}` にはうまくいきますが、 `UnitRange {T}` やその他の抽象タイプの明示的なバージョンも記述しなければなりません。
<!-- End -->
<!-- Start -->
To prevent such tedium, you can use two parameters in the declaration of `MyContainer`:
> このような兆候を避けるために、 `MyContainer` の宣言で2つのパラメータを使うことができます：
<!-- End -->

```jldoctest containers2
julia> mutable struct MyContainer{T, A<:AbstractVector}
           a::A
       end

julia> MyContainer(v::AbstractVector) = MyContainer{eltype(v), typeof(v)}(v)
MyContainer

julia> b = MyContainer(1:5);

julia> typeof(b)
MyContainer{Int64,UnitRange{Int64}}
```

<!-- Start -->
Note the somewhat surprising fact that `T` doesn't appear in the declaration of field `a`, a point that we'll return to in a moment. 
> `a` の宣言に `T` が現れないという驚くべき事実に注意してください。これはすぐに返される点です。
<!-- End -->
<!-- Start -->
With this approach, one can write functions such as:
> このアプローチでは、次のような関数を書くことができます。
<!-- End -->

```jldoctest containers2
julia> function myfunc(c::MyContainer{<:Integer, <:AbstractArray})
           return c.a[1]+1
       end
myfunc (generic function with 1 method)

julia> function myfunc(c::MyContainer{<:AbstractFloat})
           return c.a[1]+2
       end
myfunc (generic function with 2 methods)

julia> function myfunc(c::MyContainer{T,Vector{T}}) where T<:Integer
           return c.a[1]+3
       end
myfunc (generic function with 3 methods)
```

<!-- Start -->
!!! note
> !!! 注意
<!-- End -->
   <!-- Start -->
   Because we can only define `MyContainer` for `A<:AbstractArray`, and any unspecified parameters are arbitrary, the first function above could have been written more succinctly as `function myfunc(c::MyContainer{<:Integer})`
   > 私たちは `A<:AbstractArray` に対して `MyContainer` を定義することができ、未指定のパラメータは任意です。上記の最初の関数は `function myfunc(c::MyContainer{<:Integer})`
<!-- End -->


```jldoctest containers2
julia> myfunc(MyContainer(1:3))
2

julia> myfunc(MyContainer(1.0:3))
3.0

julia> myfunc(MyContainer([1:3;]))
4
```

<!-- Start -->
As you can see, with this approach it's possible to specialize on both the element type `T` and the array type `A`.
> ご覧のとおり、このアプローチでは、エレメントタイプ `T` と配列タイプ `A` の両方を専門にすることができます。
<!-- End -->

<!-- Start -->
However, there's one remaining hole: we haven't enforced that `A` has element type `T`, so it's perfectly possible to construct an object like this:
> しかし、残っている穴が1つ残っています。 `A` に要素タイプ `T` があることを強制していないので、このようなオブジェクトを構築することは完全に可能です。
<!-- End -->

```jldoctest containers2
julia> b = MyContainer{Int64, UnitRange{Float64}}(UnitRange(1.3, 5.0));

julia> typeof(b)
MyContainer{Int64,UnitRange{Float64}}
```

<!-- Start -->
To prevent this, we can add an inner constructor:
> これを防ぐために、内部コンストラクタを追加することができます：
<!-- End -->

```jldoctest containers3
julia> mutable struct MyBetterContainer{T<:Real, A<:AbstractVector}
           a::A
           MyBetterContainer{T,A}(v::AbstractVector{T}) where {T,A} = new(v)
       end

julia> MyBetterContainer(v::AbstractVector) = MyBetterContainer{eltype(v),typeof(v)}(v)
MyBetterContainer

julia> b = MyBetterContainer(UnitRange(1.3, 5.0));

julia> typeof(b)
MyBetterContainer{Float64,UnitRange{Float64}}

julia> b = MyBetterContainer{Int64, UnitRange{Float64}}(UnitRange(1.3, 5.0));
ERROR: MethodError: Cannot `convert` an object of type UnitRange{Float64} to an object of type MyBetterContainer{Int64,UnitRange{Float64}}
[...]
```

<!-- Start -->
The inner constructor requires that the element type of `A` be `T`.
> 内側のコンストラクタでは、要素型 `A` が `T` であることが必要です。
<!-- End -->

<!-- Start -->
### Annotate values taken from untyped locations
> ### タイプのない場所から取得した値に注釈を付ける
<!-- End -->

<!-- Start -->
It is often convenient to work with data structures that may contain values of any type (arrays of type `Array{Any}`). 
> 任意の型の値( `Array{Any}` 型の配列 )を含む可能性のあるデータ構造を扱うことは、しばしば便利です。
<!-- End -->
<!-- Start -->
But, if you're using one of these structures and happen to know the type of an element, it helps to share this knowledge with the compiler:
> しかし、これらの構造体のいずれかを使用していて、要素の型を知っている場合は、この知識をコンパイラと共有することができます。
<!-- End -->

```julia
function foo(a::Array{Any,1})
    x = a[1]::Int32
    b = x+1
    ...
end
```

<!-- Start -->
Here, we happened to know that the first element of `a` would be an [`Int32`](@ref). 
> ここで、 `a` の最初の要素は [`Int32`](@ref) になることを知りました。
<!-- End -->
<!-- Start -->
Making an annotation like this has the added benefit that it will raise a run-time error if the value is not of the expected type, potentially catching certain bugs earlier.
> このようなアノテーションを作成すると、値が期待される型でない場合にランタイムエラーが発生し、特定のバグを早期に突き止める可能性があるという利点があります。
<!-- End -->

<!-- Start -->
In the case that the type of `a[1]` is not known precisely, `x` can be declared via `x = convert(Int32,a[1])::Int32`. 
> `a [1]` の型が正確に分からない場合、`x = convert(Int32,a[1])::Int32` で `x` を宣言することができます。
<!-- End -->
<!-- Start -->
The use of the [`convert`](@ref) function allows `a[1]` to be any object convertible to an `Int32` (such as `UInt8`), thus increasing the genericity of the code by loosening the type requirement. 
> [`convert`](@ref) 関数を使うと `a[1]` が `Int32` ( 例えば `UInt8` )に変換可能なオブジェクトになり、型を緩めることでコードの汎用性が高まります 要件。
<!-- End -->
<!-- Start -->
Notice that `convert` itself needs a type annotation in this context in order to achieve type stability. 
> 型の安定性を達成するためには、 `convert` 自体がこの文脈で型アノテーションを必要とすることに注意してください。
<!-- End -->
<!-- Start -->
This is because the compiler cannot deduce the type of the return value of a function, even `convert`, unless the types of all the function's arguments are known.
> これは、すべての関数の引数の型がわかっていない限り、コンパイラは関数の戻り値の型を推測することはできないからです。
<!-- End -->

<!-- Start -->
### Declare types of keyword arguments
> ### キーワードの引数の型を宣言する
<!-- End -->

<!-- Start -->
Keyword arguments can have declared types:
> キーワード引数は宣言された型を持つことができます：
<!-- End -->

```julia
function with_keyword(x; name::Int = 1)
    ...
end
```

<!-- Start -->
Functions are specialized on the types of keyword arguments, so these declarations will not affect performance of code inside the function. 
> 関数はキーワード引数の型に特化しているため、これらの宣言は関数内のコードのパフォーマンスには影響しません。
<!-- End -->
<!-- Start -->
However, they will reduce the overhead of calls to the function that include keyword arguments.
> ただし、キーワード引数を含む関数への呼び出しのオーバーヘッドを削減します。
<!-- End -->

<!-- Start -->
Functions with keyword arguments have near-zero overhead for call sites that pass only positional arguments.
> キーワード引数を持つ関数は、位置引数のみを渡す呼び出しサイトにはほとんどゼロのオーバーヘッドを持ちます。
<!-- End -->

<!-- Start -->
Passing dynamic lists of keyword arguments, as in `f(x; keywords...)`, can be slow and should be avoided in performance-sensitive code.
> `f(x; keywords ...)` のようにキーワード引数の動的リストを渡すのは遅く、パフォーマンス重視のコードでは避けるべきです。
<!-- End -->

<!-- Start -->
## Break functions into multiple definitions
> ## 複数の定義に関数を分割する
<!-- End -->

<!-- Start -->
Writing a function as many small definitions allows the compiler to directly call the most applicable code, or even inline it.
> 関数を多くの小さな定義として記述することで、コンパイラは最も適切なコードを直接呼び出すことができます。
<!-- End -->

<!-- Start -->
Here is an example of a "compound function" that should really be written as multiple definitions:
> 実際に複数の定義として記述されるべき「複合関数」の例を以下に示します。
<!-- End -->

```julia
function norm(A)
    if isa(A, Vector)
        return sqrt(real(dot(A,A)))
    elseif isa(A, Matrix)
        return maximum(svd(A)[2])
    else
        error("norm: invalid argument")
    end
end
```

<!-- Start -->
This can be written more concisely and efficiently as:
> これは以下のようにより簡潔かつ効率的に書くことができます。
<!-- End -->

```julia
norm(x::Vector) = sqrt(real(dot(x,x)))
norm(A::Matrix) = maximum(svd(A)[2])
```

<!-- Start -->
## Write "type-stable" functions
> ## "型安定"関数を書く
<!-- End -->

<!-- Start -->
When possible, it helps to ensure that a function always returns a value of the same type. 
> 可能な場合は、関数が常に同じ型の値を返すようにすることができます。
<!-- End -->
<!-- Start -->
Consider the following definition:
> 次の定義を考えてみましょう。
<!-- End -->

```julia
pos(x) = x < 0 ? 0 : x
```

<!-- Start -->
Although this seems innocent enough, the problem is that `0` is an integer (of type `Int`) and `x` might be of any type. 
> これは無意味に思えるかもしれませんが、問題は `0` は整数 (Int型) で、 `x` は任意の型であるということです。
<!-- End -->
<!-- Start -->
Thus, depending on the value of `x`, this function might return a value of either of two types. 
> したがって、 `x` の値によっては、この関数は2つの型のいずれかの値を返すかもしれません。
<!-- End -->
<!-- Start -->
This behavior is allowed, and may be desirable in some cases. But it can easily be fixed as follows:
> この動作は許可されており、場合によっては望ましい場合もあります。 しかし、以下のように簡単に修正することができます：
<!-- End -->

```julia
pos(x) = x < 0 ? zero(x) : x
```

<!-- Start BBB-->
There is also a [`one()`](@ref) function, and a more general [`oftype(x, y)`](@ref) function, which returns `y` converted to the type of `x`.
[`one()`](@ref) 関数と、より一般的な [`oftype(x, y)`](@ref) 関数もあります。 。
<!-- End -->

<!-- Start -->
## Avoid changing the type of a variable
> ## 変数の型の変更を避ける
<!-- End -->

<!-- Start -->
An analogous "type-stability" problem exists for variables used repeatedly within a function:
> 関数内で繰り返し使用される変数には、同様の「型安定性」問題が存在します。
<!-- End -->

```julia
function foo()
    x = 1
    for i = 1:10
        x = x/bar()
    end
    return x
end
```

<!-- Start -->
Local variable `x` starts as an integer, and after one loop iteration becomes a floating-point number (the result of [`/`](@ref) operator). 
> ローカル変数 `x` は整数で始まり、1回のループ反復後には浮動小数点数 ([`/`](@ref) 演算子の結果 )になります。
<!-- End -->
<!-- Start -->
This makes it more difficult for the compiler to optimize the body of the loop. There are several possible fixes:
> これにより、コンパイラがループの本体を最適化することがより困難になります。 いくつかの修正が可能です：
<!-- End -->
   <!-- Start -->
* Initialize `x` with `x = 1.0`
* > `x = 1.0` で `x` を初期化する
<!-- End -->
<!-- Start -->
* Declare the type of `x`: `x::Float64 = 1`
* > `x` の型を宣言します： `x::Float64 = 1`
<!-- End -->
<!-- Start -->
* Use an explicit conversion: `x = oneunit(T)`
* > 明示的な変換を使う： `x = oneunit(T)`
<!-- End -->
<!-- Start -->
* Initialize with the first loop iteration, to `x = 1/bar()`, then loop `for i = 2:10`
* > 最初のループ反復で `x = 1/bar()` に初期化し、 `for i = 2:10`をループします。
<!-- End -->

<!-- Start -->
## [Separate kernel functions (aka, function barriers)](@id kernal-functions)
> ## [別々のカーネル関数(別名、関数バリア)](@idカーネル関数)
<!-- End -->

<!-- Start -->
Many functions follow a pattern of performing some set-up work, and then running many iterations to perform a core computation. 
> 多くの関数は、いくつかのセットアップ作業を実行し、次に多くの繰り返しを実行してコア計算を実行するパターンに従います。
<!-- End -->
<!-- Start -->
Where possible, it is a good idea to put these core computations in separate functions. 
> 可能であれば、これらのコア計算を別々の関数に入れるのは良い考えです。
<!-- End -->
<!-- Start -->
For example, the following contrived function returns an array of a randomly-chosen type:
> たとえば、次のような関数は、ランダムに選択された型の配列を返します。
<!-- End -->

```@meta
DocTestSetup = quote
    srand(1234)
end
```

```jldoctest
julia> function strange_twos(n)
           a = Vector{rand(Bool) ? Int64 : Float64}(n)
           for i = 1:n
               a[i] = 2
           end
           return a
       end
strange_twos (generic function with 1 method)

julia> strange_twos(3)
3-element Array{Float64,1}:
 2.0
 2.0
 2.0
```

<!-- Start -->
This should be written as:
> これは次のように記述する必要があります。
<!-- End -->

```jldoctest
julia> function fill_twos!(a)
           for i=1:length(a)
               a[i] = 2
           end
       end
fill_twos! (generic function with 1 method)

julia> function strange_twos(n)
           a = Array{rand(Bool) ? Int64 : Float64}(n)
           fill_twos!(a)
           return a
       end
strange_twos (generic function with 1 method)

julia> strange_twos(3)
3-element Array{Float64,1}:
 2.0
 2.0
 2.0
```

<!-- Start -->
Julia's compiler specializes code for argument types at function boundaries, so in the original implementation it does not know the type of `a` during the loop (since it is chosen randomly).
> Juliaのコンパイラは関数の境界で引数型のコードを特化しているので、元の実装では、ループ中の `a` の型は(ランダムに選択されているので)知りません。
<!-- End -->
<!-- Start -->
Therefore the second version is generally faster since the inner loop can be recompiled as part of `fill_twos!` for different types of `a`.
> したがって、第2のバージョンは一般的に高速です。なぜなら、内部ループは `a` の異なるタイプに対して `fill_twos!` の一部として再コンパイルできるからです。
<!-- End -->

<!-- Start -->
The second form is also often better style and can lead to more code reuse.
> 2番目の形式は、しばしばスタイルが改善され、コードを再利用できるようになります。
<!-- End -->

<!-- Start -->
This pattern is used in several places in the standard library. 
> このパターンは、標準ライブラリのいくつかの場所で使用されます。
<!-- End -->
<!-- Start -->
For example, see `hvcat_fill` in [`abstractarray.jl`](https://github.com/JuliaLang/julia/blob/master/base/abstractarray.jl), or the [`fill!`](@ref) function, which we could have used instead of writing our own `fill_twos!`.
> たとえば、[`abstractarray.jl`](https://github.com/JuliaLang/julia/blob/master/base/abstractarray.jl) の `hvcat_fill` や [`fill!`](@ref) 関数を使用することができます。これは、独自の `fill_twos!` を書く代わりに使用することができます。
<!-- End -->

<!-- Start -->
Functions like `strange_twos` occur when dealing with data of uncertain type, for example data loaded from an input file that might contain either integers, floats, strings, or something else.
> `strange_twos` のような関数は、整数、浮動小数点数、文字列などの入力ファイルから読み込まれたデータなど、不確定な型のデータを扱うときに発生します。
<!-- End -->

<!-- Start -->
## Types with values-as-parameters
> ## 値をパラメータとする型
<!-- End -->

<!-- Start -->
Let's say you want to create an `N`-dimensional array that has size 3 along each axis. Such arrays can be created like this:
> 各軸に沿ってサイズが3のN次元配列を作成したいとしましょう。そのような配列は次のように作成できます：
<!-- End -->

```jldoctest
julia> A = fill(5.0, (3, 3))
3×3 Array{Float64,2}:
 5.0  5.0  5.0
 5.0  5.0  5.0
 5.0  5.0  5.0
```

<!-- Start -->
This approach works very well: the compiler can figure out that `A` is an `Array{Float64,2}` because it knows the type of the fill value (`5.0::Float64`) and the dimensionality (`(3, 3)::NTuple{2,Int}`).
> この方法は非常にうまくいきます：コンパイラは、 `A` が `Array {Float64,2}` であることを知ることができます。なぜなら、それはフィル値( `5.0::Float64` )のタイプと次元 ( `(3, 3)::NTuple{2,Int}` )。
<!-- End -->
<!-- Start -->
This implies that the compiler can generate very efficient code for any future usage of `A` in the same function.
> これは、コンパイラが、同じ関数内の将来の `A` の使用のための非常に効率的なコードを生成できることを意味します。
<!-- End -->

<!-- Start -->
But now let's say you want to write a function that creates a 3×3×... array in arbitrary dimensions;
> しかし、今では、任意の次元で3×3×...配列を作成する関数を記述したいとしましょう。
<!-- End -->
<!-- Start -->
you might be tempted to write a function
> あなたは関数を書こうと思うかもしれません
<!-- End -->

```jldoctest
julia> function array3(fillval, N)
           fill(fillval, ntuple(d->3, N))
       end
array3 (generic function with 1 method)

julia> array3(5.0, 2)
3×3 Array{Float64,2}:
 5.0  5.0  5.0
 5.0  5.0  5.0
 5.0  5.0  5.0
```

<!-- Start -->
This works, but (as you can verify for yourself using `@code_warntype array3(5.0, 2)`) the problem is that the output type cannot be inferred: the argument `N` is a *value* of type `Int`, and type-inference does not (and cannot) predict its value in advance. 
> これはうまくいきます( `@code_warntype array3(5.0,2)` を使って自分自身で確認できます )、出力型を推論できないという問題があります。引数 `N` は `Int` 型の *value* 型推論はその値を前もって予測しません(そしてできません)。
<!-- End -->
<!-- Start -->
This means that code using the output of this function has to be conservative, checking the type on each access of `A`; such code will be very slow.
> これは、この関数の出力を使用するコードは保守的で、 `A` の各アクセス時に型をチェックする必要があることを意味します。 そのようなコードは非常に遅くなります。
<!-- End -->

<!-- Start -->
Now, one very good way to solve such problems is by using the [function-barrier technique](@ref kernal-functions).
> このような問題を解決するには、[function-barrier technique](@ref kernal-functions) を使うのが良い方法です。
<!-- End -->
<!-- Start -->
However, in some cases you might want to eliminate the type-instability altogether.  
> しかし、場合によってはタイプの不安定性を完全に排除したいかもしれません。
<!-- End -->
<!-- Start -->
In such cases, one approach is to pass the dimensionality as a parameter, for example through `Val{T}()` (see ["Value types"](@ref)):
> そのような場合、一つのアプローチは、例えば `Val{T}()`( ["Value types"](@ref) 参照)を介して、次元としてパラメータを渡すことです：
<!-- End -->

```jldoctest
julia> function array3(fillval, ::Val{N}) where N
           fill(fillval, ntuple(d->3, Val(N)))
       end
array3 (generic function with 1 method)

julia> array3(5.0, Val(2))
3×3 Array{Float64,2}:
 5.0  5.0  5.0
 5.0  5.0  5.0
 5.0  5.0  5.0
```

<!-- Start -->
Julia has a specialized version of `ntuple` that accepts a `Val{::Int}` instance as the second parameter; by passing `N` as a type-parameter, you make its "value" known to the compiler.
> Juliaには、第2引数として `Val{::Int}` インスタンスを受け入れる特殊なバージョンの `ntuple` があります。 型パラメータとして `N` を渡すことによって、コンパイラがその "値" を知ることができます。
<!-- End -->
<!-- Start -->
Consequently, this version of `array3` allows the compiler to predict the return type.
> その結果、このバージョンの `array3` は、コンパイラが戻り型を予測することを可能にします。
<!-- End -->

<!-- Start -->
However, making use of such techniques can be surprisingly subtle. 
> しかしながら、このような技術を利用することは、驚くほど微妙なことである。
<!-- End -->
<!-- Start -->
For example, it would be of no help if you called `array3` from a function like this:
> 例えば、あなたが `array3`を以下のような関数から呼び出すと助けにならないでしょう：
<!-- End -->

```julia
function call_array3(fillval, n)
    A = array3(fillval, Val(n))
end
```

<!-- Start -->
Here, you've created the same problem all over again: the compiler can't guess what `n` is, so it doesn't know the *type* of `Val(n)`.  
> ここでは、同じ問題を繰り返し作成しました。コンパイラは `n` が何であるかを推測することができないので、 `Val(n)` の * type* を知りません。
<!-- End -->
<!-- Start -->
Attempting to use `Val`, but doing so incorrectly, can easily make performance *worse* in many situations.  (Only in situations where you're effectively combining `Val` with the function-barrier trick, to make the kernel function more efficient, should code like the above be used.)
> `Val` を使用しようとしましたが、間違って実行すると、多くの状況で簡単にパフォーマンスが悪くなります*。 ( `Val` と関数バリアのトリックを効果的に組み合わせて、カーネル関数をより効率的にする場合にのみ、上記のようなコードを使うべきです)。
<!-- End -->

<!-- Start -->
An example of correct usage of `Val` would be:
> `Val` の正しい使い方の例は：
<!-- End -->

```julia
function filter3(A::AbstractArray{T,N}) where {T,N}
    kernel = array3(1, Val(N))
    filter(A, kernel)
end
```

<!-- Start -->
In this example, `N` is passed as a parameter, so its "value" is known to the compiler.  
> この例では、 `N` がパラメータとして渡されるため、その "value" はコンパイラに認識されます。
<!-- End -->
<!-- Start -->
Essentially, `Val(T)` works only when `T` is either hard-coded/literal (`Val(3)`) or already specified in the type-domain.
> 本質的に `Val(T)` は、 `T` がハードコーディング/リテラル( `Val(3)` )であるか、型ドメインで既に指定されている場合にのみ機能します。
<!-- End -->

<!-- Start -->
## The dangers of abusing multiple dispatch (aka, more on types with values-as-parameters)
> ## 複数のディスパッチを悪用する危険性(別名、値をパラメータとする型について)
<!-- End -->

<!-- Start -->
Once one learns to appreciate multiple dispatch, there's an understandable tendency to go crazy and try to use it for everything. 
> 一度、複数のディスパッチを理解することを学ぶと、狂ったようになり、すべてのためにそれを使用しようとする理解可能な傾向があります。
<!-- End -->
<!-- Start -->
For example, you might imagine using it to store information, e.g.
> たとえば、情報を格納するために使用するとします。
<!-- End -->

```
struct Car{Make,Model}
    year::Int
    ...more fields...
end
```

<!-- Start -->
and then dispatch on objects like `Car{:Honda,:Accord}(year, args...)`.
> `Car {:Honda,:Accord}(year,args ...)` のようなオブジェクトにディスパッチします。
<!-- End -->

<!-- Start -->
This might be worthwhile when the following are true:
> 次のことが当てはまる場合、これは価値があるかもしれません。
<!-- End -->

<!-- Start -->
* You require CPU-intensive processing on each `Car`, and it becomes vastly more efficient if you know the `Make` and `Model` at compile time.
* > それぞれの `Car` でCPUを大量に処理する必要があります。コンパイル時に `Make` と `Model` を知っていれば、非常に効率的になります。
<!-- End -->
<!-- Start -->
* You have homogenous lists of the same type of `Car` to process, so that you can store them all in an `Array{Car{:Honda,:Accord},N}`.
* > 処理する同じタイプの `Car` の同種のリストを持っていますので、それらを `Array {Carr::Honda,:Accord},N}` にすべて格納することができます。
<!-- End -->

<!-- Start -->
When the latter holds, a function processing such a homogenous array can be productively specialized:
> 後者の場合、均質なアレイを処理する関数は、生産的に特殊化することができます。
<!-- End -->
<!-- Start -->
Julia knows the type of each element in advance (all objects in the container have the same concrete type), so Julia can "look up" the correct method calls when the function is being compiled (obviating the need to check at run-time) and thereby emit efficient code for processing the whole list.
> Juliaは各要素の型を事前に知っています(コンテナ内のすべてのオブジェクトは同じ具体的な型を持っています)ので、Juliaは関数がコンパイルされているときに正しいメソッド呼び出しを "ルックアップ"できます(実行時にチェックする必要はありません)リスト全体を処理するための効率的なコードを発行する。
<!-- End -->

<!-- Start -->
When these do not hold, then it's likely that you'll get no benefit; worse, the resulting "combinatorial explosion of types" will be counterproductive.  
> これらが保持されない場合、あなたは利益を得られない可能性があります。結果として生じる「タイプのコンビナトリアル爆発」は逆効果になります。
<!-- End -->
<!-- Start -->
If `items[i+1]` has a different type than `item[i]`, Julia has to look up the type at run-time, search for the appropriate method in method tables, decide (via type intersection) which one matches, determine whether it has been JIT-compiled yet (and do so if not), and then make the call. 
> `items [i+1]` が `item[i]` と異なる型を持っている場合、Juliaは実行時に型を調べ、メソッドテーブルで適切なメソッドを検索し、それがまだJITコンパイルされているかどうかを判断し(そうでなければそうする)、呼び出しを行う。
<!-- End -->
<!-- Start -->
In essence, you're asking the full type- system and JIT-compilation machinery to basically execute the equivalent of a switch statement or dictionary lookup in your own code.
> 本質的に、完全な型システムとJITコンパイル機械に、基本的にswitch文や辞書検索と同等のコードを自分のコードで実行するように要求しています。
<!-- End -->

<!-- Start -->
Some run-time benchmarks comparing (1) type dispatch, (2) dictionary lookup, and (3) a "switch" statement can be found [on the mailing list](https://groups.google.com/forum/#!msg/julia-users/jUMu9A3QKQQ/qjgVWr7vAwAJ).
> (1)型ディスパッチ、 (2)辞書ルックアップ、 (3) "switch"文を比較する実行時ベンチマークは[メーリングリストで]見つけることができます (https://groups.google.com/forum/#!msg/julia-users/jUMu9A3QKQQ/qjgVWr7vAwAJ) 。
<!-- End -->

<!-- Start -->
Perhaps even worse than the run-time impact is the compile-time impact: Julia will compile specialized functions for each different `Car{Make, Model}`; if you have hundreds or thousands of such types, then every function that accepts such an object as a parameter (from a custom `get_year` function you might write yourself, to the generic `push!` function in the standard library) will have hundreds or thousands of variants compiled for it.  
> 多分実行時の影響よりも悪いのは、コンパイル時の影響です。Juliaは、それぞれ異なる `Car{Make, Model}` のための特殊な関数をコンパイルします。そのような型が何百、何千もある場合、そのようなオブジェクトをパラメータとして受け入れるすべての関数(独自の `get_year` 関数から、標準ライブラリの汎用 `push!` 関数に自分自身を書くことができます)は、またはそれのためにコンパイルされた何千もの変種。
<!-- End -->
<!-- Start -->
Each of these increases the size of the cache of compiled code, the length of internal lists of methods, etc.  Excess enthusiasm for values-as-parameters can easily waste enormous resources.
> これらのそれぞれは、コンパイルされたコードのキャッシュのサイズ、メソッドの内部リストの長さなどを増加させます。パラメータとしての値に対する過度の熱意は、膨大なリソースを簡単に無駄にする可能性があります。
<!-- End -->

<!-- Start -->
## Access arrays in memory order, along columns
> ## 列に沿ってメモリの順序で配列にアクセスする
<!-- End -->

<!-- Start -->
Multidimensional arrays in Julia are stored in column-major order. 
> Juliaの多次元配列は、カラムメジャー順に格納されます。
<!-- End -->
<!-- Start -->
This means that arrays are stacked one column at a time. 
> つまり、配列は一度に1列ずつ積み重ねられます。
<!-- End -->
<!-- Start -->
This can be verified using the `vec` function or the syntax `[:]` as shown below (notice that the array is ordered `[1 3 2 4]`, not `[1 2 3 4]`):
> これは、以下に示すように、 `vec` 関数や構文 `[:]` を使って検証できます(配列は `[1 2 3 4]` ではなく `[1 3 2 4]` )
<!-- End -->

```jldoctest
julia> x = [1 2; 3 4]
2×2 Array{Int64,2}:
 1  2
 3  4

julia> x[:]
4-element Array{Int64,1}:
 1
 3
 2
 4
```

<!-- Start -->
This convention for ordering arrays is common in many languages like Fortran, Matlab, and R (to name a few). 
> 配列の順序付けのこの規約は、Fortran、Matlab、およびRなどの多くの言語で一般的です(いくつか例を挙げると)。
<!-- End -->
<!-- Start -->
The alternative to column-major ordering is row-major ordering, which is the convention adopted by C and Python (`numpy`) among other languages. 
> 列メジャー順序の代わりに、行優先順序があります。これは、他の言語の中でCおよびPython (`numpy`) によって採用されている規則です。
<!-- End -->
<!-- Start -->
Remembering the ordering of arrays can have significant performance effects when looping over arrays. 
> 配列の順番を覚えておくと、配列をループする際にパフォーマンスに大きな影響を与える可能性があります。
<!-- End -->
<!-- Start -->
A rule of thumb to keep in mind is that with column-major arrays, the first index changes most rapidly. 
> 経験則では、カラムメジャー配列では、最初のインデックスが最も急速に変化するということです。
<!-- End -->
<!-- Start -->
Essentially this means that looping will be faster if the inner-most loop index is the first to appear in a slice expression.
> 基本的にこれは、最も内側のループインデックスがスライス式に最初に現れる場合、ループが速くなることを意味します。
<!-- End -->

<!-- Start -->
Consider the following contrived example. 
> 次のような人為的な例を考えてみましょう。
<!-- End -->
<!-- Start -->
Imagine we wanted to write a function that accepts a [`Vector`](@ref) and returns a square [`Matrix`](@ref) with either the rows or the columns filled with copies of the input vector. 
>私たちが [`Vector`](@ref) を受け取り、入力ベクトルのコピーでいっぱいになった行または列のある正方形の [`Matrix`](@ref)を返す関数を書いたとします。
<!-- End -->
<!-- Start -->
Assume that it is not important whether rows or columns are filled with these copies (perhaps the rest of the code can be easily adapted accordingly). 
> 行または列がこれらのコピーで満たされているかどうかは重要ではないと仮定します(残りのコードはおそらくそれに応じて容易に適合させることができます)。
<!-- End -->
<!-- Start -->
We could conceivably do this in at least four ways (in addition to the recommended call to the built-in [`repmat()`](@ref)):
> 組み込みの[`repmat()`](@ref)への推奨呼び出しに加えて、少なくとも4つの方法でこれを行うことができます。
<!-- End -->

```julia
function copy_cols(x::Vector{T}) where T
    n = size(x, 1)
    out = Array{T}(n, n)
    for i = 1:n
        out[:, i] = x
    end
    out
end

function copy_rows(x::Vector{T}) where T
    n = size(x, 1)
    out = Array{T}(n, n)
    for i = 1:n
        out[i, :] = x
    end
    out
end

function copy_col_row(x::Vector{T}) where T
    n = size(x, 1)
    out = Array{T}(n, n)
    for col = 1:n, row = 1:n
        out[row, col] = x[row]
    end
    out
end

function copy_row_col(x::Vector{T}) where T
    n = size(x, 1)
    out = Array{T}(n, n)
    for row = 1:n, col = 1:n
        out[row, col] = x[col]
    end
    out
end
```

<!-- Start -->
Now we will time each of these functions using the same random `10000` by `1` input vector:
> 今度は、これらの関数のそれぞれを、 `1` 入力ベクトルと同じランダムな `10000` を使って時間を計るでしょう：
<!-- End -->

```julia-repl
julia> x = randn(10000);

julia> fmt(f) = println(rpad(string(f)*": ", 14, ' '), @elapsed f(x))

julia> map(fmt, Any[copy_cols, copy_rows, copy_col_row, copy_row_col]);
copy_cols:    0.331706323
copy_rows:    1.799009911
copy_col_row: 0.415630047
copy_row_col: 1.721531501
```

<!-- Start -->
Notice that `copy_cols` is much faster than `copy_rows`. 
> `copy_cols` は `copy_rows` よりはるかに高速です。
<!-- End -->
<!-- Start -->
This is expected because `copy_cols` respects the column-based memory layout of the `Matrix` and fills it one column at a time. 
> これは、 `copy_cols` が `Matrix` の列ベースのメモリレイアウトを尊重し、一度に一つの列に充てんするために必要です。
<!-- End -->
<!-- Start -->
Additionally, `copy_col_row` is much faster than `copy_row_col` because it follows our rule of thumb that the first element to appear in a slice expression should be coupled with the inner-most loop.
> さらに、 `copy_col_row` は `copy_row_col` よりはるかに高速です。私たちの経験則に従うと、スライス式に表示される最初の要素は最も内側のループと結合する必要があるからです。
<!-- End -->

<!-- Start -->
## Pre-allocating outputs
> ## 出力の事前割り当て
<!-- End -->

<!-- Start -->
If your function returns an `Array` or some other complex type, it may have to allocate memory.
> あなたの関数が `Array` やその他の複合型を返す場合、メモリを割り当てる必要があります。
<!-- End -->
<!-- Start -->
Unfortunately, oftentimes allocation and its converse, garbage collection, are substantial bottlenecks.
> 残念なことに、しばしば割り振りと逆にガベージコレクションは、かなりのボトルネックです。
<!-- End -->

<!-- Start -->
Sometimes you can circumvent the need to allocate memory on each function call by preallocating the output.  
> 場合によっては、出力を事前に割り当てることによって、各関数呼び出しでメモリを割り当てる必要性を回避できます。
<!-- End -->
<!-- Start -->
As a trivial example, compare
> 簡単な例として、
<!-- End -->

```julia
function xinc(x)
    return [x, x+1, x+2]
end

function loopinc()
    y = 0
    for i = 1:10^7
        ret = xinc(i)
        y += ret[2]
    end
    y
end
```

<!-- Start -->
with
> 依って
<!-- End -->

```julia
function xinc!(ret::AbstractVector{T}, x::T) where T
    ret[1] = x
    ret[2] = x+1
    ret[3] = x+2
    nothing
end

function loopinc_prealloc()
    ret = Array{Int}(3)
    y = 0
    for i = 1:10^7
        xinc!(ret, i)
        y += ret[2]
    end
    y
end
```

<!-- Start -->
Timing results:
> タイミングの結果：
<!-- End -->

```julia-repl
julia> @time loopinc()
  0.529894 seconds (40.00 M allocations: 1.490 GiB, 12.14% gc time)
50000015000000

julia> @time loopinc_prealloc()
  0.030850 seconds (6 allocations: 288 bytes)
50000015000000
```

<!-- Start -->
Preallocation has other advantages, for example by allowing the caller to control the "output" type from an algorithm.  
> 事前割り振りには、呼び出し元がアルゴリズムから「出力」タイプを制御できるようにするなど、他の利点があります。
<!-- End -->
<!-- Start -->
In the example above, we could have passed a `SubArray` rather than an [`Array`](@ref), had we so desired.
> 上の例では、私たちが望むならば、 [`Array`](@ref) ではなく `SubArray` を渡すことができました。
<!-- End -->

<!-- Start -->
Taken to its extreme, pre-allocation can make your code uglier, so performance measurements and some judgment may be required. 
> 極端な場合、事前割り当てによってコードが醜いものになる可能性があるため、パフォーマンス測定と判断が必要になることがあります。
<!-- End -->
<!-- Start -->
However, for "vectorized" (element-wise) functions, the convenient syntax `x .= f.(y)` can be used for in-place operations with fused loops and no temporary arrays (see the [dot syntax for vectorizing functions](@ref man-vectorized)).
> しかし、 "ベクトル化"(要素別)関数の場合、融合ループと一時配列を使用しないインプレース操作に便利な構文 `x .= f.(y)` を使用できます(関数をベクトル化するためのドット構文 ](@ref man-vectorized))。
<!-- End -->

<!-- Start -->
## More dots: Fuse vectorized operations
> ## その他の点：ベクター化された操作を融合する
<!-- End -->

<!-- Start -->
Julia has a special [dot syntax](@ref man-vectorized) that converts any scalar function into a "vectorized" function call, and any operator into a "vectorized" operator, with the special property that nested "dot calls" are *fusing*: they are combined at the syntax level into a single loop, without allocating temporary arrays. 
> Juliaにはスカラー関数を「ベクトル化」関数呼び出しに変換する特別な [ドット構文](@ref man-vectorized)があり、すべての演算子は「ベクトル化」演算子に入れられます。 fusing*: 一時配列を割り当てずに、構文レベルで単一のループに結合されます。
<!-- End -->
<!-- Start -->
If you use `.=` and similar assignment operators, the result can also be stored in-place in a pre-allocated array (see above).
> `.=` と同様の代入演算子を使用すると、その結果をあらかじめ割り当てられた配列のインプレースに格納することもできます(上記参照)。
<!-- End -->

<!-- Start -->
In a linear-algebra context, this means that even though operations like `vector + vector` and `vector * scalar` are defined, it can be advantageous to instead use `vector .+ vector` and `vector .* scalar` because the resulting loops can be fused with surrounding computations. 
> 線形代数コンテキストでは、これは、「ベクトル+ベクトル」および「ベクトル*スカラー」のような演算が定義されていても、代わりに「ベクトル。+ベクトル」および「ベクトル。*スカラー」を使用することが有利であることを意味する。 結果のループは周囲の計算と融合することができます。
<!-- End -->
<!-- Start -->
For example, consider the two functions:
> たとえば、次の2つの関数を考えてみましょう。
<!-- End -->

```julia
f(x) = 3x.^2 + 4x + 7x.^3

fdot(x) = @. 3x^2 + 4x + 7x^3 <!-- Start -->
# equivalent to 3 .* x.^2 .+ 4 .* x .+ 7 .* x.^3
```

<!-- Start -->
Both `f` and `fdot` compute the same thing.  However, `fdot` (defined with the help of the [`@.`](@ref @__dot__) macro) is significantly faster when applied to an array:
> `f` と `fdot` は同じことを計算します。 しかし `fdot`([`@.`]@ref @__dot__ マクロの助けを借りて定義された) は、配列に適用するとかなり速くなります：
<!-- End -->

```julia-repl
julia> x = rand(10^6);

julia> @time f(x);
  0.010986 seconds (18 allocations: 53.406 MiB, 11.45% gc time)

julia> @time fdot(x);
  0.003470 seconds (6 allocations: 7.630 MiB)

julia> @time f.(x);
  0.003297 seconds (30 allocations: 7.631 MiB)
```

<!-- Start -->
That is, `fdot(x)` is three times faster and allocates 1/7 the memory of `f(x)`, because each `*` and `+` operation in `f(x)` allocates a new temporary array and executes in a separate loop. 
> つまり、 `f(x)` の各 `*` と `+` 演算は新しい一時配列を割り当てるので、 `fdot(x)` は3倍高速で `f(x)` のメモリを1/7に割り当てます。 別のループで実行されます。
<!-- End -->
<!-- Start -->
(Of course, if you just do `f.(x)` then it is as fast as `fdot(x)` in this example, but in many contexts it is more convenient to just sprinkle some dots in your expressions rather than defining a separate function for each vectorized operation.)
> (もちろん、単に `f(x)` を実行すれば `fdot(x)` と同じくらい速くなりますが、多くの場合、いくつかのドットを式に定義するのではなく、 ベクトル化された操作ごとに別々の関数)。
<!-- End -->

<!-- Start -->
## Consider using views for slices
> ## スライスのビューの使用を検討する
<!-- End -->

<!-- Start BBB -->
In Julia, an array "slice" expression like `array[1:5, :]` creates a copy of that data (except on the left-hand side of an assignment, where `array[1:5, :] = ...` assigns in-place to that portion of `array`).
> Juliaでは、 `array [1：5, :]` のような配列のスライス式は、そのデータのコピーを作成します(代入の左側では `array [1:5, :] = ...` となります `array` のその部分にインプレースを割り当てます)。
<!-- End -->
<!-- Start -->
If you are doing many operations on the slice, this can be good for performance because it is more efficient to work with a smaller contiguous copy than it would be to index into the original array.
> スライスで多くの操作を行っている場合は、元の配列にインデックスを付けるよりも小さな連続したコピーで作業する方が効率的であるため、パフォーマンスが良い場合があります。
<!-- End -->
<!-- Start -->
On the other hand, if you are just doing a few simple operations on the slice, the cost of the allocation and copy operations can be substantial.
> 一方で、スライス上で簡単な操作をいくつか行うだけであれば、割り当てやコピー操作のコストが相当に高くなる可能性があります。
<!-- End -->

<!-- Start -->
An alternative is to create a "view" of the array, which is an array object (a `SubArray`) that actually references the data of the original array in-place, without making a copy.  
> 別の方法としては、コピーを作成せずに元の配列のデータを実際に参照する配列オブジェクト( `SubArray` )である配列の「ビュー」を作成する方法があります。
<!-- End -->
<!-- Start -->
(If you write to a view, it modifies the original array's data as well.)
> (ビューに書き込むと、元の配列のデータも同様に変更されます)。
<!-- End -->
<!-- Start -->
This can be done for individual slices by calling [`view()`](@ref), or more simply for a whole expression or block of code by putting [`@views`](@ref) in front of that expression.
> これは個々のスライスに対して、[`view()`](@ref) を呼び出すことによって、あるいは単純にその式の前に [`@views`](@ref) を置くことによって、
<!-- End -->
<!-- Start -->
For example:
> 例えば：
<!-- End -->

```julia-repl
julia> fcopy(x) = sum(x[2:end-1])

julia> @views fview(x) = sum(x[2:end-1])

julia> x = rand(10^6);

julia> @time fcopy(x);
  0.003051 seconds (7 allocations: 7.630 MB)

julia> @time fview(x);
  0.001020 seconds (6 allocations: 224 bytes)
```

<!-- Start -->
Notice both the 3× speedup and the decreased memory allocation of the `fview` version of the function.
> 関数の `fview` バージョンの3倍速化と減少したメモリ割り当ての両方に注目してください。
<!-- End -->

<!-- Start -->
## Copying data is not always bad
> ## データのコピーは必ずしも悪いとは限りません
<!-- End -->

<!-- Start -->
Arrays are stored contiguously in memory, lending themselves to CPU vectorization and fewer memory accesses due to caching. 
> 配列はメモリに連続して格納され、キャッシングによるCPUベクトル化とメモリアクセスの減少に役立ちます。
<!-- End -->
<!-- Start -->
These are the same reasons that it is recommended to access arrays in column-major order (see above). 
> これらは、配列をカラム優先配列(上記参照)にアクセスすることが推奨されるのと同じ理由です。
<!-- End -->
<!-- Start -->
Irregular access patterns and non-contiguous views can drastically slow down computations on arrays because of non-sequential memory access.
> 不規則なアクセスパターンや非連続的なビューは、非シーケンシャルなメモリアクセスのために、配列の計算を大幅に遅くする可能性があります。
<!-- End -->

<!-- Start -->
Copying irregularly-accessed data into a contiguous array before operating on it can result in a large speedup, such as in the example below. 
> 不規則にアクセスされたデータを連続アレイにコピーしてから操作すると、以下の例のように高速化する可能性があります。
<!-- End -->
<!-- Start -->
Here, a matrix and a vector are being accessed at 800,000 of their randomly-shuffled indices before being multiplied. Copying the views into plain arrays speeds the multiplication by more than a factor of 2 even with the cost of the copying operation.
> ここでは、乗算される前にランダムにシャッフルされた80万のインデックスで行列とベクトルにアクセスしています。 ビューをプレーン・アレイにコピーすると、コピー操作のコストでも倍率が2倍以上速くなります。
<!-- End -->

```julia-repl
julia> x = randn(1_000_000);

julia> inds = shuffle(1:1_000_000)[1:800000];

julia> A = randn(50, 1_000_000);

julia> xtmp = zeros(800_000);

julia> Atmp = zeros(50, 800_000);

julia> @time sum(view(A, :, inds) * view(x, inds))
  0.640320 seconds (41 allocations: 1.391 KiB)
7253.242699002263

julia> @time begin
           copy!(xtmp, view(x, inds))
           copy!(Atmp, view(A, :, inds))
           sum(Atmp * xtmp)
       end
  0.261294 seconds (41 allocations: 1.391 KiB)
7253.242699002323
```
<!-- Start -->
Provided there is enough memory for the copies, the cost of copying the view to an array is far outweighed by the speed boost from doing the matrix multiplication on a contiguous array.
> コピーに十分なメモリがあれば、アレイにビューをコピーするコストは、連続したアレイで行列乗算を行うことによるスピードブーストよりはるかに重要です。
<!-- End -->

<!-- Start -->
## Avoid string interpolation for I/O
> ## I/O の文字列補間を避ける
<!-- End -->

<!-- Start -->
When writing data to a file (or other I/O device), forming extra intermediate strings is a source of overhead.
> ファイル(または他の I/O デバイス)にデータを書き込むときに、余分な中間ストリングを形成することがオーバーヘッドの原因になります。
<!-- End -->
<!-- Start -->
Instead of:
> の代わりに：
<!-- End -->

```julia
println(file, "$a $b")
```

<!-- Start -->
use:
> これをつかいましょう：
<!-- End -->

```julia
println(file, a, " ", b)
```

<!-- Start -->
The first version of the code forms a string, then writes it to the file, while the second version writes values directly to the file. 
> コードの最初のバージョンはストリングを形成し、それをファイルに書き込み、2番目のバージョンはファイルに直接値を書き込みます。
<!-- End -->
<!-- Start -->
Also notice that in some cases string interpolation can be harder to read. Consider:
> 場合によっては、文字列補間が読みにくくなることもあります。 検討してください：
<!-- End -->

```julia
println(file, "$(f(a))$(f(b))")
```

<!-- Start -->
versus:
> 対：
<!-- End -->

```julia
println(file, f(a), f(b))
```

<!-- Start -->
## Optimize network I/O during parallel execution
> ## 並列実行中にネットワーク I/O を最適化する
<!-- End -->

<!-- Start -->
When executing a remote function in parallel:
> 並列にリモート機能を実行する場合：
<!-- End -->

```julia
responses = Vector{Any}(nworkers())
@sync begin
    for (idx, pid) in enumerate(workers())
        @async responses[idx] = remotecall_fetch(pid, foo, args...)
    end
end
```

<!-- Start -->
is faster than:
> 以下より高速です。
<!-- End -->

```julia
refs = Vector{Any}(nworkers())
for (idx, pid) in enumerate(workers())
    refs[idx] = @spawnat pid foo(args...)
end
responses = [fetch(r) for r in refs]
```

<!-- Start -->
The former results in a single network round-trip to every worker, while the latter results in two network calls - first by the [`@spawnat`](@ref) and the second due to the [`fetch`](@ref) (or even a [`wait`](@ref)). 
> 前者はすべての作業者に単一のネットワークラウンドトリップをもたらし、後者は最初に [`@spawnat`](@ref) と [`fetch`](@ref) の2つのネットワーク呼び出しをもたらします )(または [`wait`](@ref)) であってもよい。
<!-- End -->
<!-- Start -->
The [`fetch`](@ref)/[`wait`](@ref) is also being executed serially resulting in an overall poorer performance. 
> [`fetch`](@ref)/[`wait`](@ref) も連続して実行されているため、パフォーマンスは全体的に劣ります。
<!-- End -->

<!-- Start -->
## Fix deprecation warnings
> ## 廃止予定の警告を修正
<!-- End -->

<!-- Start -->
A deprecated function internally performs a lookup in order to print a relevant warning only once.
> 推奨されない関数は内部的に関連する警告を1回だけ印刷するためにルックアップを実行します。
<!-- End -->
<!-- Start -->
This extra lookup can cause a significant slowdown, so all uses of deprecated functions should be modified as suggested by the warnings.
> この余分な参照は大幅な減速を引き起こす可能性があるため、廃止された関数のすべての使用法は、警告によって示唆されるように変更する必要があります。
<!-- End -->

<!-- Start -->
## Tweaks
> ## 調整
<!-- End -->

<!-- Start -->
These are some minor points that might help in tight inner loops.
> これらは、タイトな内部ループを助けるかもしれないいくつかの小さな点です。
<!-- End -->

<!-- Start -->
* Avoid unnecessary arrays. For example, instead of [`sum([x,y,z])`](@ref) use `x+y+z`.
* > 不要な配列は避けてください。 例えば、[`sum([x,y,z])`](@ref) の代わりに `x+y+z` を使います。
<!-- End -->
<!-- Start -->
* Use [`abs2(z)`](@ref) instead of [`abs(z)^2`](@ref) for complex `z`. 
* > 複雑な `z` の場合は [`abs(z)^2`](@ref) の代わりに[`abs2(z)`](@ref)を使います。
<!-- End -->
<!-- Start -->
* In general, try to rewrite code to use [`abs2()`](@ref) instead of [`abs()`](@ref) for complex arguments.
* > 一般に複雑な引数に対しては、[`abs()`](@ref) の代わりに [`abs2()`](@ref) を使うようにコードを書き直してみてください。
<!-- End -->
<!-- Start -->
* Use [`div(x,y)`](@ref) for truncating division of integers instead of [`trunc(x/y)`](@ref), [`fld(x,y)`](@ref) instead of [`floor(x/y)`](@ref), and [`cld(x,y)`](@ref) instead of [`ceil(x/y)`](@ref).
* > [`trunc(x/y)`](@ref), [`fld(x,y)`](@ref) の代わりに整数の除算を切り捨てるには [`div(x,y)`](@ref)  [`ceil(x/y)`](@ref) の代わりに [`floor(x/y)`](@ref) と [`cld(x,y)`](@ref)
<!-- End -->

<!-- Start -->
## [Performance Annotations](@id man-performance-annotations )
> ## [パフォーマンスアノテーション](@id man-performance-annotations)
<!-- End -->

<!-- Start -->
Sometimes you can enable better optimization by promising certain program properties.
> 場合によっては、特定のプログラムプロパティを約束することによって、より良い最適化を実現することができます。
<!-- End -->

<!-- Start -->
* Use `@inbounds` to eliminate array bounds checking within expressions.
* > `@inbounds`を使うと、式の中で配列の境界チェックをなくすことができます。
<!-- End -->
<!-- Start -->
* Be certain before doing this. If the subscripts are ever out of bounds, you may suffer crashes or silent corruption.
* > これを行う前に必ず確認してください。 下付き文字が範囲外になると、クラッシュやサイレント破損が発生する可能性があります。
<!-- End -->
<!-- Start -->
* Use `@fastmath` to allow floating point optimizations that are correct for real numbers, but lead to differences for IEEE numbers. 
* > 実数に対して正しい浮動小数点最適化を許可するには `@fastmath` を使用しますが、IEEE数の違いにつながります。
<!-- End -->
<!-- Start -->
* Be careful when doing this, as this may change numerical results. 
* > 数値計算結果が変わる可能性があるので、これを行うときは注意してください。
<!-- End -->
<!-- Start -->
* This corresponds to the `-ffast-math` option of clang.
* > これはclangの `-ffast-math`オプションに相当します。
<!-- End -->
<!-- Start -->
* Write `@simd` in front of `for` loops that are amenable to vectorization. 
* > ベクトル化が容易な `for`ループの前に` @ simd`を書いてください。
<!-- End -->
<!-- Start -->
* **This feature is experimental** and could change or disappear in future versions of Julia.
* > **この機能は実験的なもの** であり、将来のバージョンのJuliaでは変更または消滅する可能性があります。
<!-- End -->

<!-- Start -->
Note: While `@simd` needs to be placed directly in front of a loop, both `@inbounds` and `@fastmath` can be applied to several statements at once, e.g. using `begin` ... `end`, or even to a whole function.
> 注： `@ simd`はループの直前に置く必要がありますが、` @inbounds`と `@fastmath`は同時に複数の文に適用できます。 `begin` ...` end`を使って、あるいは関数全体にさえも使うことができます。
<!-- End -->

<!-- Start -->
Here is an example with both `@inbounds` and `@simd` markup:
> `@inbounds`と` @ simd`の両方のマークアップを使った例です：
<!-- End -->

```julia
function inner(x, y)
    s = zero(eltype(x))
    for i=1:length(x)
        @inbounds s += x[i]*y[i]
    end
    s
end

function innersimd(x, y)
    s = zero(eltype(x))
    @simd for i=1:length(x)
        @inbounds s += x[i]*y[i]
    end
    s
end

function timeit(n, reps)
    x = rand(Float32,n)
    y = rand(Float32,n)
    s = zero(Float64)
    time = @elapsed for j in 1:reps
        s+=inner(x,y)
    end
    println("GFlop/sec        = ",2.0*n*reps/time*1E-9)
    time = @elapsed for j in 1:reps
        s+=innersimd(x,y)
    end
    println("GFlop/sec (SIMD) = ",2.0*n*reps/time*1E-9)
end

timeit(1000,1000)
```

<!-- Start -->
On a computer with a 2.4GHz Intel Core i5 processor, this produces:
> 2.4GHz Intel Core i5プロセッサ搭載のコンピュータでは、次のようになります。
<!-- End -->

```
GFlop/sec        = 1.9467069505224963
GFlop/sec (SIMD) = 17.578554163920018
```

<!-- Start -->
(`GFlop/sec` measures the performance, and larger numbers are better.) 
> ( `GFlop/sec` はパフォーマンスの基準で、大きな数字ほど良いです。)
<!-- End -->
<!-- Start -->
The range for a `@simd for` loop should be a one-dimensional range. 
`@simd for` ループの範囲は1次元の範囲でなければなりません。
<!-- End -->
<!-- Start -->
A variable used for accumulating, such as `s` in the example, is called a *reduction variable*. 
> この例で `s` のように累積に使用される変数は、*reduction変数* と呼ばれます。
<!-- End -->
<!-- Start -->
By using `@simd`, you are asserting several properties of the loop:
> `@simd`を使うことで、ループのいくつかのプロパティをアサートします：
<!-- End -->

   <!-- Start -->
* It is safe to execute iterations in arbitrary or overlapping order, with special consideration for reduction variables.
* > reduction変数を特に考慮して、任意の順序または重複する順序で反復を実行することは安全です。
<!-- End -->
   <!-- Start -->
* Floating-point operations on reduction variables can be reordered, possibly causing different results than without `@simd`.
<!-- End -->
* > reduction変数の浮動小数点演算は並べ替えることができ、`@simd`を使わない場合とは異なる結果が生じる可能性があります。
<!-- Start -->
* No iteration ever waits on another iteration to make forward progress.
<!-- End -->
* > 繰返しは、進行を進めるために別の反復を待つことはありません。

<!-- Start -->
A loop containing `break`, `continue`, or `@goto` will cause a compile-time error.
> `break`, `continue`, または `@goto` を含むループは、コンパイル時エラーを引き起こします。
<!-- End -->

<!-- Start -->
Using `@simd` merely gives the compiler license to vectorize.
> `@simd`を使うとコンパイラへ、ベクトル化の許可を単純に与えます。
<!-- End -->
<!-- Start -->
Whether it actually does so depends on the compiler. 
> 実際にそうするかどうかは、コンパイラによって決まります。
<!-- End -->
<!-- Start -->
To actually benefit from the current implementation, your loop should have the following additional properties:
>現在の実装から実際に利益を得るには、ループに次の追加プロパティが必要です。
<!-- End -->

<!-- Start -->
* The loop must be an innermost loop.
* > ループは最も内側のループでなければなりません。
<!-- End -->
<!-- Start -->
*  The loop body must be straight-line code. This is why `@inbounds` is currently needed for all array accesses.
* > ループ本体は直列コードでなければなりません。 このため、現在、すべての配列アクセスに `@inbounds`が必要です。
<!-- End -->
<!-- Start -->
* The compiler can sometimes turn short `&&`, `||`, and `?:` expressions into straight-line code, if it is safe to evaluate all operands unconditionally. 
* > コンパイラは、すべてのオペランドを無条件に評価することが安全である場合、短い`&&`、`||`、および`？：`の式を直線コードにすることがあります。
<!-- End -->
<!-- Start -->
* Consider using [`ifelse()`](@ref) instead of `?:` in the loop if it is safe to do so.
* > ループの中で `?:` の代わりに [`ifelse()`](@ref) を使うことをお勧めします。
<!-- End -->
<!-- Start -->
* Accesses must have a stride pattern and cannot be "gathers" (random-index reads) or "scatters" (random-index writes).
* > アクセスはストライドパターンを持たなければならず、「収集」(ランダム索引読取り)または「散在」(ランダム索引書込み)はできません。
<!-- End -->
   <!-- Start -->
* The stride should be unit stride.
* > ストライドはユニットストライドでなければなりません。
<!-- End -->
<!-- Start -->
* In some simple cases, for example with 2-3 arrays accessed in a loop, the LLVM auto-vectorization may kick in automatically, leading to no further speedup with `@simd`.
* > いくつかの簡単な例では、例えばループ内で2-3個の配列がアクセスされている場合、LLVMの自動ベクトル化は自動的に開始され、 `@simd`ではそれ以上のスピードアップはありません。
<!-- End -->

<!-- Start -->
Here is an example with all three kinds of markup. 
> ここでは、3種類のマークアップの例を示します。
<!-- End -->

<!-- Start -->
This program first calculates the finite difference of a one-dimensional array, and then evaluates the L2-norm of the result:
> このプログラムは、まず、1次元配列の有限差分を計算し、結果のL2ノルムを評価します。
<!-- End -->

```julia
function init!(u)
    n = length(u)
    dx = 1.0 / (n-1)
    @fastmath @inbounds @simd for i in 1:n
        u[i] = sin(2pi*dx*i)
    end
end

function deriv!(u, du)
    n = length(u)
    dx = 1.0 / (n-1)
    @fastmath @inbounds du[1] = (u[2] - u[1]) / dx
    @fastmath @inbounds @simd for i in 2:n-1
        du[i] = (u[i+1] - u[i-1]) / (2*dx)
    end
    @fastmath @inbounds du[n] = (u[n] - u[n-1]) / dx
end

function norm(u)
    n = length(u)
    T = eltype(u)
    s = zero(T)
    @fastmath @inbounds @simd for i in 1:n
        s += u[i]^2
    end
    @fastmath @inbounds return sqrt(s/n)
end

function main()
    n = 2000
    u = Array{Float64}(n)
    init!(u)
    du = similar(u)

    deriv!(u, du)
    nu = norm(du)

    @time for i in 1:10^6
        deriv!(u, du)
        nu = norm(du)
    end

    println(nu)
end

main()
```

<!-- Start -->
On a computer with a 2.7 GHz Intel Core i7 processor, this produces:
> 2.7 GHz Intel Core i7プロセッサ搭載のコンピュータでは、次のようになります。
<!-- End -->

```
$ julia wave.jl;
elapsed time: 1.207814709 seconds (0 bytes allocated)

$ julia --math-mode=ieee wave.jl;
elapsed time: 4.487083643 seconds (0 bytes allocated)
```

<!-- Start -->
Here, the option `--math-mode=ieee` disables the `@fastmath` macro, so that we can compare results.
> ここで、オプション `--math-mode = ieee` は `@fastmath` マクロを無効にし、結果を比較することができます。
<!-- End -->

<!-- Start -->
In this case, the speedup due to `@fastmath` is a factor of about 3.7. 
> この場合、 `@fastmath 'によるスピードアップは約3.7倍です。
<!-- End -->
<!-- Start -->
This is unusually large – in general, the speedup will be smaller. 
>これは異常に大きいです - 一般的に、スピードアップはもっと小いものです。
<!-- End -->
<!-- Start -->
(In this particular example, the working set of the benchmark is small enough to fit into the L1 cache of the processor, so that memory access latency does not play a role, and computing time is dominated by CPU usage. 
> (この特定の例では、ベンチマークのワーキングセットはプロセッサのL1キャッシュに収まるほど小さいため、メモリアクセスのレイテンシは影響を受けず、コンピューティング時間はCPU使用量が支配的です。
<!-- End -->
<!-- Start -->
In many real world programs this is not the case.) 
> 多くの現実世界のプログラムではそうではありません。)
<!-- End -->
<!-- Start -->
Also, in this case this optimization does not change the result – in general, the result will be slightly different. 
> また、この例の場合、この最適化によって結果が変ることはありません。一般に、結果は若干異なります。
<!-- End -->
<!-- Start -->
In some cases, especially for numerically unstable algorithms, the result can be very different.
> 場合によっては、特に数値的に不安定なアルゴリズムの場合、結果が大きく異なる場合があります。
<!-- End -->

<!-- Start -->
The annotation `@fastmath` re-arranges floating point expressions, e.g. changing the order of evaluation, or assuming that certain special cases (inf, nan) cannot occur. 
> アノテーション `@fastmath` は浮動小数点式を再配置します。評価の順序を変更したり、特定の特殊ケース(inf、nan)が発生しないと仮定しています。
<!-- End -->
<!-- Start -->
In this case (and on this particular computer), the main difference is that the expression `1 / (2*dx)` in the function `deriv` is hoisted out of the loop (i.e. calculated outside the loop), as if one had written `idx = 1 / (2*dx)`. In the loop, the expression `... / (2*dx)` then becomes `... * idx`, which is much faster to evaluate. 
> この場合(およびこの特定のコンピュータ上で)、主な相違点は、関数`deriv` の式 `1 /(2 * dx)` がループ外に持ち出される(つまり、ループ外で計算される) `idx = 1 /(2 * dx)` と書かれていました。ループの中では、式 `... /(2 * dx)` は `... * idx`になります。これは評価がはるかに高速です。
<!-- End -->
<!-- Start -->
Of course, both the actual optimization that is applied by the compiler as well as the resulting speedup depend very much on the hardware. You can examine the change in generated code by using Julia's [`code_native()`](@ref) function.
> もちろん、コンパイラによって適用される実際の最適化とそれに伴うスピードアップは、ハードウェアに大きく依存します。 Juliaの [`code_native()`](@ref) 関数を使って、生成されたコードの変更を調べることができます。
<!-- End -->

<!-- Start -->
## Treat Subnormal Numbers as Zeros
## Subnormal Numbers をゼロとして扱う
<!-- End -->

<!-- Start -->
Subnormal numbers, formerly called [denormal numbers](https://en.wikipedia.org/wiki/Denormal_number), are useful in many contexts, but incur a performance penalty on some hardware. 
> 以前の[デノーマル番号(denormal numbers)](https://en.wikipedia.org/wiki/Denormal_number)と呼ばれる異常な数値は、多くの場合に役立ちますが、一部のハードウェアではパフォーマンス上のペナルティが発生します。
<!-- End -->
<!-- Start -->
A call [`set_zero_subnormals(true)`](@ref) grants permission for floating-point operations to treat subnormal inputs or outputs as zeros, which may improve performance on some hardware. A call [`set_zero_subnormals(false)`](@ref) enforces strict IEEE behavior for subnormal numbers.
> [`set_zero_subnormals(true)`](@ref) 呼び出しは、浮動小数点演算に対して、非正規の入力または出力をゼロとして扱うための許可を与えます。これによって、ハードウェアのパフォーマンスが向上する可能性があります。 [`set_zero_subnormals(false)`](@ref) 呼び出しは、異常な数値に対して厳密なIEEE動作を強制します。
<!-- End -->

<!-- Start -->
Below is an example where subnormals noticeably impact performance on some hardware:
> 以下は、サブノールが一部のハードウェアでパフォーマンスに顕著に影響する例です。
<!-- End -->

```julia
function timestep(b::Vector{T}, a::Vector{T}, Δt::T) where T
    @assert length(a)==length(b)
    n = length(b)
    b[1] = 1                            <!-- Start -->
# Boundary condition
<!-- End -->
    for i=2:n-1
        b[i] = a[i] + (a[i-1] - T(2)*a[i] + a[i+1]) * Δt
    end
    b[n] = 0                            <!-- Start -->
# Boundary condition
<!-- End -->
end

function heatflow(a::Vector{T}, nstep::Integer) where T
    b = similar(a)
    for t=1:div(nstep,2)                <!-- Start -->
# Assume nstep is even
<!-- End -->
        timestep(b,a,T(0.1))
        timestep(a,b,T(0.1))
    end
end

heatflow(zeros(Float32,10),2)           <!-- Start -->
# Force compilation
<!-- End -->
for trial=1:6
    a = zeros(Float32,1000)
    set_zero_subnormals(iseven(trial))  <!-- Start -->
# Odd trials use strict IEEE arithmetic
<!-- End -->
    @time heatflow(a,1000)
end
```

<!-- Start -->
This example generates many subnormal numbers because the values in `a` become an exponentially decreasing curve, which slowly flattens out over time.
> この例では、 `a` の値が指数関数的に減少する曲線になり、時間の経過とともにゆっくりと平らになるため、多くの非正常な数値が生成されます。
<!-- End -->

<!-- Start -->
Treating subnormals as zeros should be used with caution, because doing so breaks some identities, such as `x-y == 0` implies `x == y`:
> サブノーマルをゼロとして扱う場合は慎重になるべきです。例えば、`x-y == 0` は `x == y`を意味するなど、いくつかのアイデンティティが壊れるからです。
<!-- End -->

```jldoctest
julia> x = 3f-38; y = 2f-38;

julia> set_zero_subnormals(true); (x - y, x == y)
(0.0f0, false)

julia> set_zero_subnormals(false); (x - y, x == y)
(1.0000001f-38, false)
```

<!-- Start -->
In some applications, an alternative to zeroing subnormal numbers is to inject a tiny bit of noise.
> いくつかのアプリケーションでは、非正常な数値をゼロにする代わりに、わずかなノイズを注入することができます。
<!-- End -->
<!-- Start -->
> For example, instead of initializing `a` with zeros, initialize it with:
たとえば、 `a` をゼロで初期化する代わりに、次のように初期化します。
<!-- End -->

```julia
a = rand(Float32,1000) * 1.f-9
```

<!-- Start -->
## [[`@code_warntype`](@ref)](@id man-code-warntype)
>## [[`@ code_warntype`](@ref)](@id man-code-warntype)
<!-- End -->

<!-- Start -->
The macro [`@code_warntype`](@ref) (or its function variant [`code_warntype()`](@ref)) can sometimes be helpful in diagnosing type-related problems. 
> マクロ [`@code_warntype`](@ref) (またはその関数型 [`code_warntype()`](@ref) ) は型の問題を診断するのに役立ちます。
<!-- End -->
<!-- Start -->
Here's an example:
> ここに例があります：
<!-- End -->

```julia
pos(x) = x < 0 ? 0 : x

function f(x)
    y = pos(x)
    sin(y*x+1)
end

julia> @code_warntype f(3.2)
Variables:
#self#::#f
  x::Float64
  y::UNION{FLOAT64,INT64}
  fy::Float64
#temp#@_5::UNION{FLOAT64,INT64}
#temp#@_6::Core.MethodInstance
#temp#@_7::Float64

Body:
  begin
      $(Expr(:inbounds, false))
# meta: location REPL[1] pos 1
# meta: location float.jl < 487
      fy::Float64 = (Core.typeassert)((Base.sitofp)(Float64,0)::Float64,Float64)::Float64
# meta: pop location
      unless (Base.or_int)((Base.lt_float)(x::Float64,fy::Float64)::Bool,(Base.and_int)((Base.and_int)((Base.eq_float)(x::Float64,fy::Float64)::Bool,(Base.lt_float)(fy::Float64,9.223372036854776e18)::Bool)::Bool,(Base.slt_int)((Base.fptosi)(Int64,fy::Float64)::Int64,0)::Bool)::Bool)::Bool goto 9
#temp#@_5::UNION{FLOAT64,INT64} = 0
      goto 11
      9:
#temp#@_5::UNION{FLOAT64,INT64} = x::Float64
      11:
# meta: pop location
      $(Expr(:inbounds, :pop))
      y::UNION{FLOAT64,INT64} = <!-- Start -->
#temp#@_5::UNION{FLOAT64,INT64} # line 3:
      unless (y::UNION{FLOAT64,INT64} isa Int64)::ANY goto 19
#temp#@_6::Core.MethodInstance = MethodInstance for *(::Int64, ::Float64)
      goto 28
      19:
      unless (y::UNION{FLOAT64,INT64} isa Float64)::ANY goto 23
#temp#@_6::Core.MethodInstance = MethodInstance for *(::Float64, ::Float64)
      goto 28
      23:
      goto 25
      25:
#temp#@_7::Float64 = (y::UNION{FLOAT64,INT64} * x::Float64)::Float64
      goto 30
      28:
#temp#@_7::Float64 = $(Expr(:invoke, :(#temp#@_6), :(Main.*), :(y), :(x)))
      30:
      return $(Expr(:invoke, MethodInstance for sin(::Float64), :(Main.sin), :((Base.add_float)(#temp#@_7,(Base.sitofp)(Float64,1)::Float64)::Float64)))
  end::Float64
```

<!-- Start -->
> Interpreting the output of [`@code_warntype`](@ref), like that of its cousins [`@code_lowered`](@ref), [`@code_typed`](@ref), [`@code_llvm`](@ref), and [`@code_native`](@ref), takes a little practice.
>[`@code_warntype`](@ref), [`@code_lowv`](@ref), [`@code_typed`](@ref), @ref), [`@code_native`](@ref) は少し練習します。
<!-- End -->
<!-- Start -->
Your code is being presented in form that has been partially digested on its way to generating compiled machine code.  
> あなたのコードは、コンパイルされたマシンコードを生成する途中で部分的に消化された形で提示されています。
<!-- End -->
<!-- Start -->
Most of the expressions are annotated by a type, indicated by the `::T` (where `T` might be [`Float64`](@ref), for example). 
> ほとんどの式は、 `::T` ( `T` は [`Float64`](@ref) など)で示される型によって注釈されます。
<!-- End -->
<!-- Start -->
The most important characteristic of [`@code_warntype`](@ref) is that non-concrete types are displayed in red; in the above example, such output is shown in all-caps.
> [`@code_warntype`](@ref) のもっとも重要な特徴は、非具体的な型が赤で表示されていることです。上記の例では、そのような出力はすべて大文字で示されています。
<!-- End -->

<!-- Start -->
The top part of the output summarizes the type information for the different variables internal to the function. You can see that `y`, one of the variables you created, is a `Union{Int64,Float64}`, due to the type-instability of `pos`.  
> 出力の上部には、関数内部のさまざまな変数の型情報がまとめられています。型の不安定性のために、あなたが作成した変数の1つである `y` が `Union {Int64、Float64}` であることが分かります。
<!-- End -->
<!-- Start -->
There is another variable, `_var4`, which you can see also has the same type.
> 別の変数 `_var4` もあります。これは同じタイプの変数もあります。
<!-- End -->

<!-- Start -->
The next lines represent the body of `f`. The lines starting with a number followed by a colon
> 次の行は `f` の本体を表します。数字で始まりコロンが続く行
<!-- End -->
<!-- Start -->
(`1:`, `2:`) are labels, and represent targets for jumps (via `goto`) in your code.  
> (`1:`,`2:`) はラベルで、コード内のジャンプ( `goto` 経由)のターゲットを表します。
<!-- End -->
<!-- Start -->
Looking at the body, you can see that `pos` has been *inlined* into `f`--everything before `2:` comes from code defined in `pos`.
> 本文を見ると、`pos` がインライン展開されて `f` になっていることが分かります。 `2:` の前のものは `pos` で定義されたコードに由来します。
<!-- End -->

<!-- Start -->
Starting at `2:`, the variable `y` is defined, and again annotated as a `Union` type.  
> `2:` で始まり、変数 `y` が定義され、`Union` 型として再び注釈されます。
<!-- End -->
<!-- Start -->
Next, we see that the compiler created the temporary variable `_var1` to hold the result of `y*x`. Because a [`Float64`](@ref) times *either* an [`Int64`](@ref) or `Float64` yields a `Float64`, all type-instability ends here. 
> 次に、コンパイラが `y * x` の結果を保持する一時変数 `_var1` を作成したことがわかります。 [`Float64`](@ref) times *either* [`Int64`](@ref) または `Float64` は `Float64` を生成するので、すべての型不安定性がここで終わります。
<!-- End -->
<!-- Start -->
The net result is that `f(x::Float64)` will not be type-unstable in its output, even if some of the intermediate computations are type-unstable.
> 正味の結果は、中間計算のいくつかが型不安定であっても、 `f(x::Float64)` はその出力に型不安定ではないということです。
<!-- End -->

<!-- Start -->
How you use this information is up to you. 
> この情報の使い方はあなた次第です。
<!-- End -->
<!-- Start -->
Obviously, it would be far and away best to fix `pos` to be type-stable: if you did so, all of the variables in `f` would be concrete, and its performance would be optimal.  
> 明らかに、 `pos` を型安定に修正することは遠くて遠いでしょう：そうした場合、 `f` のすべての変数は具体的になり、そのパフォーマンスは最適になります。
<!-- End -->
<!-- Start -->
However, there are circumstances where this kind of *ephemeral* type instability might not matter too much: for example, if `pos` is never used in isolation, the fact that `f`'s output is type-stable (for [`Float64`](@ref) inputs) will shield later code from the propagating effects of type instability.  
> しかし、この種の一時的な型の不安定性はあまり重要ではないかもしれません：例えば、 `pos` が決して孤立して使われない場合、 `f` の出力は型安定です ([`Float64`](@ref) 入力) は、後のコードを型不安定性の伝搬効果から保護します。
<!-- End -->
<!-- Start -->
This is particularly relevant in cases where fixing the type instability is difficult or impossible: for example, currently it's not possible to infer the return type of an anonymous function.  
> これは、タイプの不安定性の修正が困難または不可能な場合に特に重要です。たとえば、現在、無名関数の戻り値の型を推測することはできません。
<!-- End -->
<!-- Start -->
In such cases, the tips above (e.g., adding type annotations and/or breaking up functions) are your best tools to contain the "damage" from type instability.
> このような場合、上記のヒント(タイプアノテーションの追加や機能の分割など)は、タイプの不安定性による「ダメージ」を抑えるための最良のツールです。
<!-- End -->

<!-- Start -->
The following examples may help you interpret expressions marked as containing non-leaf types:
> 次の例は、非リーフタイプを含むとマークされた式の解釈に役立ちます。
<!-- End -->

<!-- Start -->
* Function body ending in `end::Union{T1,T2})`
* > `end::Union{T1,T2})` で終わる関数本体
<!-- End -->
<!-- Start -->
* Interpretation: function with unstable return type
* > 解釈：不安定な戻り値型の関数
<!-- End -->
<!-- Start -->
* Suggestion: make the return value type-stable, even if you have to annotate it
* > 提案：アノテーションを付ける必要がある場合でも、戻り値は型安定しているようにする
<!-- End -->
*  `f(x::T)::Union{T1,T2}`

<!-- Start -->
* Interpretation: call to a type-unstable function
* > 解釈：型不安定関数への呼び出し
<!-- End -->
<!-- Start -->
* Suggestion: fix the function, or if necessary annotate the return value
* > 提案：関数を修正するか、必要であれば戻り値に注釈を付けます
<!-- End -->
* `(top(arrayref))(A::Array{Any,1},1)::Any`

<!-- Start -->
* Interpretation: accessing elements of poorly-typed arrays
* > 解釈：型の悪い配列の要素にアクセスする
<!-- End -->
<!-- Start -->
* Suggestion: use arrays with better-defined types, or if necessary annotate the type of individual element accesses
* > 示唆：よりよく定義された型を持つ配列を使用するか、必要であれば個々の要素アクセスの型に注釈を付ける
<!-- End -->
* `(top(getfield))(A::ArrayContainer{Float64},:data)::Array{Float64,N}`

  <!-- Start -->
  * Interpretation: getting a field that is of non-leaf type. In this case, `ArrayContainer` had a field `data::Array{T}`. 
  * > 通訳：非葉型のフィールドを取得する。この場合、 `ArrayContainer`は` data :: Array {T} `フィールドを持っていました。
  <!-- End -->
  <!-- Start -->
  * But `Array` needs the dimension `N`, too, to be a concrete type.
  * > しかし、「配列」は次元「N」も必要です。具体的な型です。
  <!-- End -->
  <!-- Start -->
  * Suggestion: use concrete types like `Array{T,3}` or `Array{T,N}`, where `N` is now a parameter of `ArrayContainer`
  * > 提案： `Array {T、3}`や `Array {T、N}`のような具体的な型を使います。ここで `N`は` ArrayContainer`のパラメータです。
  <!-- End -->
