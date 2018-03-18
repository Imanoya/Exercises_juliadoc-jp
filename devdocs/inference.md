# Inference

## How inference works

<!-- EN -->
[Type inference](https://en.wikipedia.org/wiki/Type_inference) refers to the process of deducing the types of later values from the types of input values.
> [型推論]（https://en.wikipedia.org/wiki/Type_inference）は、入力値の型から後の型の型を推測するプロセスを指します。
<!-- EN -->
Julia's approach to inference has been described in blog posts ([1](https://juliacomputing.com/blog/2016/04/04/inference-convergence.html), [2](https://juliacomputing.com/blog/2017/05/15/inference-converage2.html)).
> Juliaの推論へのアプローチは、ブログ投稿（[1]（https://juliacomputing.com/blog/2016/04/04/inference-convergence.html）、[2]（https://juliacomputing.com/） blog / 2017/05/15 / inference-converage2.html））。

## Debugging compiler.jl

<!-- EN -->
You can start a Julia session, edit `compiler/*.jl` (for example to insert `print` statements), and then replace `Core.Compiler` in your running session by navigating to `base/compiler` and executing `include("compiler.jl")`.
> Juliaセッションを開始し、 `print /` .jl`を編集して（例えば `print`文を挿入する）、実行中のセッションで` core.Compiler`を `base / compiler`に移動して` include （ "compiler.jl"） `を実行します。
<!-- EN -->
This trick typically leads to much faster development than if you rebuild Julia for each change.
> このトリックは、通常、変更ごとにJuliaを再構築する場合よりもはるかに高速な開発につながります。

<!-- EN -->
A convenient entry point into inference is `typeinf_code`. Here's a demo running inference on `convert(Int, UInt(1))`:
> 推論の便利な入り口は `typeinf_code`です。 以下は `convert(Int、UInt(1))` の推論を実行するデモです：

```julia
# Get the method
atypes = Tuple{Type{Int}, UInt}  # argument types
mths = methods(convert, atypes)  # worth checking that there is only one
m = first(mths)

# Create variables needed to call `typeinf_code`
params = Core.Compiler.Params(typemax(UInt))  # parameter is the world age,
                                                        #   typemax(UInt) -> most recent
sparams = Core.svec()      # this particular method doesn't have type-parameters
optimize = true            # run all inference optimizations
cached = false             # force inference to happen (do not use cached results)
Core.Compiler.typeinf_code(m, atypes, sparams, optimize, cached, params)
```

<!-- EN -->
If your debugging adventures require a `MethodInstance`, you can look it up by calling `Core.Compiler.code_for_method` using many of the variables above.
> デバッグの冒険に `MethodInstance` が必要な場合は、上記の変数の多くを使って `Core.Compiler.code_for_method` を呼び出すことで調べることができます。
<!-- EN -->
A `CodeInfo` object may be obtained with
> `CodeInfo`オブジェクトは

```julia
# Returns the CodeInfo object for `convert(Int, ::UInt)`:
ci = (@code_typed convert(Int, UInt(1)))[1]
```

## The inlining algorithm (inline_worthy)

<!-- EN -->
Much of the hardest work for inlining runs in `inlining_pass`.
> インライン化のための最も難しい作業の多くは `inlining_pass` で実行されます。
<!-- EN -->
However, if your question is "why didn't my function inline?" then you will most likely be interested in `isinlineable` and its primary callee, `inline_worthy`.
> しかし、あなたの質問が「なぜ私の機能をインラインにしていないのですか？あなたは `isinlineable` とその主な呼び出し先 `inline_worthy`に興味を持ちそうです。
<!-- EN -->
`isinlineable` handles a number of special cases (e.g., critical functions like `next` and `done`, incorporating a bonus for functions that return tuples, etc.).
> `isinlineable` は、いくつかの特殊なケース（例えば、`next` や `done` のような重要な関数、タプルを返す関数のためのボーナスの組み込みなど）を処理します。
<!-- EN -->
The main decision-making happens in `inline_worthy`, which returns `true` if the function should be inlined.
> 主な意思決定は `inline_worthy` で行われ、関数がインライン化されるべきであれば `true` を返します。

<!-- EN -->
`inline_worthy` implements a cost-model, where "cheap" functions get inlined; more specifically, we inline functions if their anticipated run-time is not large compared to the time it would take to [issue a call](https://en.wikipedia.org/wiki/Calling_convention) to them if they were not inlined.
> `inline_worthy`はコストモデルを実装します。ここでは、"安い "関数がインライン化されます。より具体的には、インライン関数が、インライン化されていない場合に実行する時間（呼び出しを行う時間）（https://en.wikipedia.org/wiki/Calling_convention）に比べて大きくない場合、インライン関数をインライン化します。
<!-- EN -->
The cost-model is extremely simple and ignores many important details: for example, all `for` loops are analyzed as if they will be executed once, and the cost of an `if...else...end` includes the summed cost of all branches.
> コストモデルは非常に単純であり、多くの重要な詳細は無視されます。たとえば、すべてのforループが1回実行されるかのように分析され、if ... else ... endのコストには、すべての支店のコスト。
<!-- EN -->
It's also worth acknowledging that we currently lack a suite of functions suitable for testing how well the cost model predicts the actual run-time cost, although [BaseBenchmarks](https://github.com/JuliaCI/BaseBenchmarks.jl) provides a great deal of indirect information about the successes and failures of any modification to the inlining algorithm.
> また、[BaseBenchmarks]（https://github.com/JuliaCI/BaseBenchmarks.jl）では、コストモデルが実際の実行時コストをどれくらいうまく予測できるかをテストするのに適した一連の関数がないことを認識することもできますインライン化アルゴリズムの変更の成功と失敗に関する間接的な情報の処理。

<!-- EN -->
The foundation of the cost-model is a lookup table, implemented in `add_tfunc` and its callers, that assigns an estimated cost (measured in CPU cycles) to each of Julia's intrinsic functions.
> コストモデルの基礎は、（CPUサイクルで測定された）推定コストをJuliaの各組み込み関数に割り当てる `add_tfunc`とその呼び出し側で実装されたルックアップテーブルです。
<!-- EN -->
These costs are based on [standard ranges for common architectures](http://ithare.com/wp-content/uploads/part101_infographics_v08.png) (see [Agner Fog's analysis](http://www.agner.org/optimize/instruction_tables.pdf) for more detail).
> これらのコストは[一般的なアーキテクチャの標準範囲]（http://ithare.com/wp-content/uploads/part101_infographics_v08.png）に基づいています（[Agner Fogの分析]（http://www.agner.org/optimizeを参照） /instruction_tables.pdf）を参照してください）。

<!-- EN -->
We supplement this low-level lookup table with a number of special cases.
> この低レベルルックアップテーブルには、多くの特別なケースが追加されています。
<!-- EN -->
For example, an `:invoke` expression (a call for which all input and output types were inferred in advance) is assigned a fixed cost (currently 20 cycles).
> たとえば、 `:invoke` 式（すべての入力と出力の型があらかじめ推論された呼び出し）には固定コスト（現在20サイクル）が割り当てられます。
<!-- EN -->
In contrast, a `:call` expression, for functions other than intrinsics/builtins, indicates that the call will require dynamic dispatch, in which case we assign a cost set by `Params.inline_nonleaf_penalty` (currently set at 1000).
> これとは対照的に、組み込み関数/組み込み関数以外の関数の `:call` 式は、呼び出しに動的ディスパッチが必要であることを示します。この場合、Params.inline_nonleaf_penalty（現在は1000に設定）で設定されたコストを割り当てます。
<!-- EN -->
Note that this is not a "first-principles" estimate of the raw cost of dynamic dispatch, but a mere heuristic indicating that dynamic dispatch is extremely expensive.
> これは、動的ディスパッチの生のコストの「第一原理」推定ではなく、ダイナミックディスパッチが非常に高価であることを示すヒューリスティックなものに過ぎないことに注意してください。

<!-- EN -->
Each statement gets analyzed for its total cost in a function called `statement_cost`.
> 各ステートメントは、 `statement_cost` という関数で総コストを分析されます。
<!-- EN -->
You can run this yourself by following this example:
> 次の例を実行すると、これを自分で実行できます。

```julia
params = Core.Compiler.Params(typemax(UInt))
# Get the CodeInfo object
ci = (@code_typed fill(3, (5, 5)))[1]  # we'll try this on the code for `fill(3, (5, 5))`
# Calculate cost of each statement
cost(stmt) = Core.Compiler.statement_cost(stmt, ci, Base, params)
cst = map(cost, ci.code)
```

<!-- EN -->
The output is a `Vector{Int}` holding the estimated cost of each statement in `ci.code`.
> 出力は `ci.code`の各ステートメントの推定コストを保持する` Vector {Int} 'です。
<!-- EN -->
Note that `ci` includes the consequences of inlining callees, and consequently the costs do too.
> `ci`には呼び出し先をインライン化する結果が含まれていることに注意してください。その結果、コストも高くなります。