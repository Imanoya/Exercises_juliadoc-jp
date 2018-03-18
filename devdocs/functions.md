# Julia Functions

<!-- EN -->
This document will explain how functions, method definitions, and method tables work.
> このドキュメントでは、関数、メソッド定義、メソッドテーブルの仕組みについて説明します。

## Method Tables

<!-- EN -->
Every function in Julia is a generic function. A generic function is conceptually a single function, but consists of many definitions, or methods.
> Juliaのすべての関数は汎用関数です。 ジェネリック関数は概念的には単一の関数ですが、多くの定義またはメソッドで構成されています。
<!-- EN -->
The methods of a generic function are stored in a method table.
> ジェネリック関数のメソッドは、メソッドテーブルに格納されます。
<!-- EN -->
Method tables (type `MethodTable`) are associated with `TypeName`s. A `TypeName` describes a family of parameterized types.
> メソッドテーブル (`MethodTable` 型) は `TypeName`s に関連付けられています。 `TypeName` は、パラメータ化された型のファミリを記述します。
<!-- EN -->
For example `Complex{Float32}` and `Complex{Float64}` share the same `Complex` type name object.
> 例えば `Complex {Float32}` と `Complex {Float64}` は同じ `Complex` 型名オブジェクトを共有します。

<!-- EN -->
All objects in Julia are potentially callable, because every object has a type, which in turn has a `TypeName`.
> Juliaのすべてのオブジェクトは、すべてのオブジェクトが型を持ち、 `TypeName` を持っているため、潜在的に呼び出し可能です。

## Function calls

<!-- EN -->
Given the call `f(x,y)`, the following steps are performed: first, the method table to use is accessed as `typeof(f).name.mt`.
> 与えられた `f(x,y)` は、以下のステップが実行されます。まず、使用するメソッドテーブルは `typeof(f).name.mt` としてアクセスされます。
<!-- EN -->
Second, an argument tuple type is formed, `Tuple{typeof(f), typeof(x), typeof(y)}`.
> 第2に、引数タプル型が形成され、 'Tuple{typeof(f), typeof(x), typeof(y)}'が生成される。
<!-- EN -->
Note that the type of the function itself is the first element.
> 関数自体の型が最初の要素であることに注意してください。
<!-- EN -->
This is because the type might have parameters, and so needs to take part in dispatch.
> これは型がパラメータを持つ可能性があるため、ディスパッチに参加する必要があるためです。
<!-- EN -->
This tuple type is looked up in the method table.
> このタプル型は、メソッドテーブルで検索されます。

<!-- EN -->
This dispatch process is performed by `jl_apply_generic`, which takes two arguments: a pointer to an array of the values f, x, and y, and the number of values (in this case 3).
> このディスパッチ・プロセスは、 `jl_apply_generic` によって実行されます。この引数は、f, x, y,の値の配列へのポインタと値の数（この場合は3）の2つの引数をとります。

<!-- EN -->
Throughout the system, there are two kinds of APIs that handle functions and argument lists: those that accept the function and arguments separately, and those that accept a single argument structure.
> システム全体を通して、関数と引数リストを扱うAPIには、関数と引数を別々に受け入れるものと、単一の引数構造を受け入れるものがあります。
<!-- EN -->
In the first kind of API, the "arguments" part does *not* contain information about the function, since that is passed separately. 
> 最初の種類のAPIでは、「引数」部分は、関数についての情報を含みません。これは、それが別々に渡されるためです。
<!-- EN -->
In the second kind of API, the function is the first element of the argument structure.
> 第2の種類のAPIでは、関数は引数構造の最初の要素です。

<!-- EN -->
For example, the following function for performing a call accepts just an `args` pointer, so the first element of the args array will be the function to call:
> 例えば、呼び出しを実行する次の関数は  `args` ポインタだけを受け入れます。したがって、args 配列の最初の要素は呼び出す関数になります：

```c
jl_value_t *jl_apply(jl_value_t **args, uint32_t nargs)
```

<!-- EN -->
This entry point for the same functionality accepts the function separately, so the `args` array does not contain the function:
> 同じ機能のためのこのエントリーポイントは関数を別々に受け入れます、従って `args` 配列は関数を含みません：

```c
jl_value_t *jl_call(jl_function_t *f, jl_value_t **args, int32_t nargs);
```

## Adding methods

<!-- EN -->
Given the above dispatch process, conceptually all that is needed to add a new method is (1) a tuple type, and (2) code for the body of the method.
> 上記のディスパッチ処理を考えると、概念的には、新しいメソッドを追加するために必要なのは、(1)タプル型、(2) メソッド本体のコードです。
<!-- EN -->
`jl_method_def` implements this operation.
> `jl_method_def` はこの操作を実装します。
<!-- EN -->
`jl_first_argument_datatype` is called to extract the relevant method table from what would be the type of the first argument.
> `jl_first_argument_datatype` が呼び出されて、最初の引数の型から関連するメソッドテーブルが抽出されます。
<!-- EN -->
This is much more complicated than the corresponding procedure during dispatch, since the argument tuple type might be abstract. 
> 引数のタプル型は抽象型である可能性があるため、ディスパッチ中の対応するプロシージャよりもはるかに複雑です。
<!-- EN -->
For example, we can define:
> たとえば、次のように定義できます。

```julia
(::Union{Foo{Int},Foo{Int8}})(x) = 0
```

<!-- EN -->
which works since all possible matching methods would belong to the same method table.
> これはすべての可能なマッチングメソッドが同じメソッドテーブルに属しているので機能します。

## Creating generic functions

<!-- EN -->
Since every object is callable, nothing special is needed to create a generic function.
> すべてのオブジェクトは呼び出し可能なので、汎用関数を作成するために特別なものは必要ありません。
<!-- EN -->
Therefore `jl_new_generic_function` simply creates a new singleton (0 size) subtype of `Function` and returns its instance.
> したがって、 `jl_new_generic_function`は` Function`の新しいシングルトン（0サイズ）サブタイプを作成し、そのインスタンスを返します。
<!-- EN -->
A function can have a mnemonic "display name" which is used in debug info and when printing objects.
> 関数は、デバッグ情報やオブジェクトの印刷時に使用されるニーモニック "表示名"を持つことができます。
<!-- EN -->
For example the name of `Base.sin` is `sin`.
> 例えば、 `Base.sin`の名前は` sin`です。
<!-- EN -->
By convention, the name of the created *type* is the same as the function name, with a `#` prepended.
> 規約では、作成された* type *の名前は関数名と同じで、先頭に `＃`が付いています。
<!-- EN -->
So `typeof(sin)` is `Base.#sin`.
> したがって、 `typeof（sin）`は `Base。＃sin`です。

## Closures

<!-- EN -->
A closure is simply a callable object with field names corresponding to captured variables.
> クロージャは、キャプチャされた変数に対応するフィールド名を持つ呼び出し可能なオブジェクトです。
<!-- EN -->
For example, the following code:
> たとえば、次のコード：

```julia
function adder(x)
    return y->x+y
end
```

is lowered to (roughly):


```julia
struct ##1{T}
    x::T
end

(_::##1)(y) = _.x + y

function adder(x)
    return ##1(x)
end
```

## Constructors

<!-- EN -->
A constructor call is just a call to a type.
> コンストラクタ呼び出しは、型への呼び出しです。
<!-- EN -->
The type of most types is `DataType`, so the method table for `DataType` contains most constructor definitions.
> ほとんどの型の型は `DataType` なので、`DataType` のメソッド表にはほとんどのコンストラクタ定義が含まれています。
<!-- EN -->
One wrinkle is the fallback definition that makes all types callable via `convert`:
> 1つのしわは、 `convert` によってすべての型を呼び出し可能とするフォールバック定義です。

```julia
(::Type{T})(args...) where {T} = convert(T, args...)::T
```

<!-- EN -->
In this definition the function type is abstract, which is not normally supported.
> この定義では、関数型は抽象型であり、通常はサポートされていません。
<!-- EN -->
To make this work, all subtypes of `Type` (`Type`, `UnionAll`, `Union`, and `DataType`) currently share a method table via special arrangement.
> これを実現するために、 `Type` (`Type`, `UnionAll`, `Union`, `DataType`)のすべてのサブタイプは現在、特別な配列を介してメソッドテーブルを共有しています。

## Builtins

<!-- EN -->
The "builtin" functions, defined in the `Core` module, are:
> `Core` モジュールで定義されている"組み込み "関数は次のとおりです：

```
=== typeof sizeof <: isa typeassert throw tuple getfield setfield! fieldtype
nfields isdefined arrayref arrayset arraysize applicable invoke apply_type _apply
_expr svec
```

<!-- EN -->
These are all singleton objects whose types are subtypes of `Builtin`, which is a subtype of `Function`.
> これらはすべて、型が `Builtin` のサブタイプであり、`Function` のサブタイプであるシングルトンオブジェクトです。
<!-- EN -->
Their purpose is to expose entry points in the run time that use the "jlcall" calling convention:
> その目的は、実行時に "jlcall"呼び出し規約を使用するエントリポイントを公開することです。

```c
jl_value_t *(jl_value_t*, jl_value_t**, uint32_t)
```

<!-- EN -->
The method tables of builtins are empty. Instead, they have a single catch-all method cache entry (`Tuple{Vararg{Any}}`) whose jlcall fptr points to the correct function.
> 組み込みメソッドのメソッドテーブルは空です。 その代わりに、単一のcatch-allメソッドキャッシュエントリ（ `Tuple {Vararg {Any}}`）があり、jlcallのfptrが正しい関数を指しています。
<!-- EN -->
This is kind of a hack but works reasonably well.
> これは一種のハックですが、合理的にうまく動作します。

## Keyword arguments

<!-- EN -->
Keyword arguments work by associating a special, hidden function object with each method table that has definitions with keyword arguments.
> キーワード引数は、特別な隠し関数オブジェクトを、キーワード引数を持つ定義を持つ各メソッドテーブルに関連付けることによって機能します。
<!-- EN -->
This function is called the "keyword argument sorter" or "keyword sorter", or "kwsorter", and is stored in the `kwsorter` field of `MethodTable` objects.
> この関数は、「キーワード引数ソート」または「キーワードソート」または「kwsorter」と呼ばれ、 `MethodTable` オブジェクトの `kwsorter` フィールドに格納されます。
<!-- EN -->
Every definition in the kwsorter function has the same arguments as some definition in the normal method table, except with a single `Array` argument prepended.
> kwsorter関数のすべての定義は、通常のメソッドテーブルのいくつかの定義と同じ引数を持ちますが、単一の `Array` 引数が前に付いています。
<!-- EN -->
This array contains alternating symbols and values that represent the passed keyword arguments.
> この配列には、渡されたキーワード引数を表すシンボルと値が交互に含まれています。
<!-- EN -->
The kwsorter's job is to move keyword arguments into their canonical positions based on name, plus evaluate and substite any needed default value expressions.
> kwsorterの仕事は、名前に基づいてキーワード引数を正規の位置に移動し、必要なデフォルト値式を評価して置換することです。
<!-- EN -->
The result is a normal positional argument list, which is then passed to yet another function.
> 結果は通常の位置引数リストであり、それはさらに別の関数に渡されます。

<!-- EN -->
The easiest way to understand the process is to look at how a keyword argument method definition is lowered.
> プロセスを理解する最も簡単な方法は、キーワード引数のメソッド定義がどのように低下するかを調べることです。
<!-- EN -->
The code:
> コード：

```julia
function circle(center, radius; color = black, fill::Bool = true, options...)
    # draw
end
```

<!-- EN -->
actually produces *three* method definitions.
> *3つの*メソッド定義を実際に生成します。
<!-- EN -->
The first is a function that accepts all arguments (including keywords) as positional arguments, and includes the code for the method body.
> 1つは、すべての引数（キーワードを含む）を位置引数として受け取り、メソッド本体のコードを含む関数です。
<!-- EN -->
It has an auto-generated name:
> それは自動生成された名前を持っています：

```julia
function #circle#1(color, fill::Bool, options, circle, center, radius)
    # draw
end
```

<!-- EN -->
The second method is an ordinary definition for the original `circle` function, which handles the case where no keyword arguments are passed:
> 2番目の方法は、元の `circle` 関数の通常の定義であり、キーワード引数が渡されない場合を処理します：

```julia
function circle(center, radius)
    #circle#1(black, true, Any[], circle, center, radius)
end
```

<!-- EN -->
This simply dispatches to the first method, passing along default values.
> これは、最初のメソッドにディスパッチし、デフォルト値を渡します。
<!-- EN -->
Finally there is the kwsorter definition:
> 最後に、kwsorterの定義があります。

```
function (::Core.kwftype(typeof(circle)))(kw::Array, circle, center, radius)
    options = Any[]
    color = arg associated with :color, or black if not found
    fill = arg associated with :fill, or true if not found
    # push remaining elements of kw into options array
    #circle#1(color, fill, options, circle, center, radius)
end
```

<!-- EN -->
The front end generates code to loop over the `kw` array and pick out arguments in the right order, evaluating default expressions when an argument is not found.
> フロントエンドは、 `kw` 配列をループし、正しい順序で引数を取り出すコードを生成し、引数が見つからないときにデフォルトの式を評価します。

<!-- EN -->
The function `Core.kwftype(t)` fetches (and creates, if necessary) the field `t.name.mt.kwsorter`.
> `Core.kwftype（t）`関数は `t.name.mt.kwsorter`フィールドを取り出します（必要に応じて作成します）。

<!-- EN -->
This design has the feature that call sites that don't use keyword arguments require no special handling; everything works as if they were not part of the language at all.
> この設計では、キーワード引数を使用しないコールサイトで特別な処理は必要ありません。 あたかもそれらが言語の一部ではないかのようにすべてが機能します。
<!-- EN -->
Call sites that do use keyword arguments are dispatched directly to the called function's kwsorter.
<!-- EN -->
> キーワード引数を使用する呼び出しサイトは、呼び出された関数のkwsorterに直接ディスパッチされます。
For example the call:
<!-- EN -->
> 例えば：

```julia
circle((0,0), 1.0, color = red; other...)
```

<!-- EN -->
is lowered to:
> 次の値に下げられます。

```julia
kwfunc(circle)(Any[:color,red,other...], circle, (0,0), 1.0)
```

<!-- EN -->
The unpacking procedure represented here as `other...` actually further unpacks each *element* of `other`, expecting each one to contain two values (a symbol and a value).
> ここで `other ... 'と表現されているアンパック手順は、実際には` other`の各*要素*をさらに展開し、それぞれが2つの値（シンボルと値）を含むことを期待しています。
<!-- EN -->
`kwfunc` (also in `Core`) fetches the kwsorter for the called function.
> `kwfunc` (`Core` でも)は呼び出された関数のkwsorterをフェッチします。
<!-- EN -->
Notice that the original `circle` function is passed through, to handle closures.
> クロージャを処理するために、元の `circle` 関数が渡されていることに注意してください。

## Compiler efficiency issues

<!-- EN -->
Generating a new type for every function has potentially serious consequences for compiler resource use when combined with Julia's "specialize on all arguments by default" design.
> Juliaの「デフォルトですべての引数に特化する」設計と組み合わせると、コンパイラリソースの使用に潜在的に重大な影響を及ぼす可能性があります。
<!-- EN -->
Indeed, the initial implementation of this design suffered from much longer build and test times, higher memory use, and a system image nearly 2x larger than the baseline.
> 実際、このデザインの最初の実装では、ビルド時間とテスト時間が大幅に長くなり、メモリ使用量が増え、システムイメージはベースラインよりも2倍近く大きくなりました。
<!-- EN -->
In a naive implementation, the problem is bad enough to make the system nearly unusable.
> 素朴な実装では、システムをほとんど使えなくするほどの問題があります。
<!-- EN -->
Several significant optimizations were needed to make the design practical.
> 設計を実用的にするためには、いくつかの重要な最適化が必要でした。

<!-- EN -->
The first issue is excessive specialization of functions for different values of function-valued arguments.
> 最初の問題は、関数値の引数の値が異なるために、関数の過度の特殊化です。
<!-- EN -->
Many functions simply "pass through" an argument to somewhere else, e.g. to another function or to a storage location.
> 多くの関数は引数を他のどこかに単に "渡す"。別の機能または保管場所に移動することができます。
<!-- EN -->
Such functions do not need to be specialized for every closure that might be passed in.
> このような関数は、渡される可能性があるすべてのクロージャーに特化する必要はありません。
<!-- EN -->
Fortunately this case is easy to distinguish by simply considering whether a function *calls* one of its arguments (i.e. the argument appears in "head position" somewhere).
> 幸いにも、このケースは、関数が引数の1つを呼び出すかどうか（つまり、引数がどこかの "頭の位置"に現れるかどうか）を単純に考えることで簡単に区別できます。
<!-- EN -->
Performance-critical higher-order functions like `map` certainly call their argument function and so will still be specialized as expected.
> `map`のようなパフォーマンスに重大な高次関数は、確かに引数関数を呼び出すので、期待どおりに特殊化されます。
<!-- EN -->
This optimization is implemented by recording which arguments are called during the `analyze-variables` pass in the front end.
> この最適化は、フロントエンドの `analyze-variables`パスの間にどの引数が呼び出されるかを記録することで実装されます。
<!-- EN -->
When `cache_method` sees an argument in the `Function` type hierarchy passed to a slot declared as `Any` or `Function`, it behaves as if the `@nospecialize` annotation were applied.
> `cache_method`が` Any`や `Function`として宣言されたスロットに渡された` Function`型階層の引数を見ると `@nospecialize`アノテーションが適用されたように振る舞います。
<!-- EN -->
This heuristic seems to be extremely effective in practice.
> このヒューリスティックは実際には非常に効果的であるようです。

<!-- EN -->
The next issue concerns the structure of method cache hash tables.
> 次の問題は、メソッドキャッシュハッシュテーブルの構造に関係します。
<!-- EN -->
Empirical studies show that the vast majority of dynamically-dispatched calls involve one or two arguments.
> 実証研究によれば、動的にディスパッチされるコールの大部分には、1つまたは2つの引数が含まれています。
<!-- EN -->
In turn, many of these cases can be resolved by considering only the first argument.
> 次に、これらのケースの多くは、最初の引数だけを考慮することで解決できます。
<!-- EN -->
(Aside: proponents of single dispatch would not be surprised by this at all.
> （別に、単一の派遣の支持者はこれに全く驚かないだろう。
<!-- EN -->
However, this argument means "multiple dispatch is easy to optimize in practice", and that we should therefore use it, *not* "we should use single dispatch"!) So the method cache uses the type of the first argument as its primary key.
> しかし、この引数は "複数のディスパッチは実際に最適化するのが簡単である"ということを意味し、それゆえにそれを使用するべきである* "*単一のディスパッチを使用するべきではない！"）したがって、メソッドキャッシュは第1引数の型をプライマリキー。
<!-- EN -->
Note, however, that this corresponds to the *second* element of the tuple type for a function call (the first element being the type of the function itself).
> ただし、これは関数呼び出しのタプル型の* 2番目の要素に対応することに注意してください（最初の要素は関数自体の型です）。
<!-- EN -->
Typically, type variation in head position is extremely low -- indeed, the majority of functions belong to singleton types with no parameters.
> 典型的には、頭部位置の型の変化は非常に低く、実際、大部分の関数はパラメータを持たないシングルトン型に属します。
<!-- EN -->
However, this is not the case for constructors, where a single method table holds constructors for every type.
> ただし、コンストラクタでは、単一のメソッドテーブルにすべての型のコンストラクタが保持されます。
<!-- EN -->
Therefore the `Type` method table is special-cased to use the *first* tuple type element instead of the second.
> したがって、 `Type`メソッドテーブルは特別なケースになっていて、第2のテーブルの代わりに* first *タプルタイプの要素を使用します。

<!-- EN -->
The front end generates type declarations for all closures.
> フロントエンドはすべてのクロージャの型宣言を生成します。
<!-- EN -->
Initially, this was implemented by generating normal type declarations.
> 最初は、これは通常の型宣言を生成することによって実装されました。
<!-- EN -->
However, this produced an extremely large number of constructors, all of which were trivial (simply passing all arguments through to [`new`](@ref)).
> しかし、これは非常に多数のコンストラクタを生成しました。これらのコンストラクタはすべて些細なものでした（すべての引数を[`new`]（@ ref）に渡すだけです）。
<!-- EN -->
Since methods are partially ordered, inserting all of these methods is O(n^2), plus there are just too many of them to keep around.
> メソッドは部分的に順序付けされているので、これらのメソッドをすべて挿入するのはO（n ^ 2）です。
<!-- EN -->
This was optimized by generating `struct_type` expressions directly (bypassing default constructor generation), and using `new` directly to create closure instances.
> これは `struct_type` 式を直接生成し（デフォルトコンストラクタ生成をバイパスして）、`new` を直接使用してクロージャインスタンスを作成することによって最適化されました。
<!-- EN -->
Not the prettiest thing ever, but you do what you gotta do.
> 今までに一番美しいものではありませんが、あなたは何をしなければなりません。

<!-- EN -->
The next problem was the `@test` macro, which generated a 0-argument closure for each test case.
> 次の問題は `@test` マクロで、各テストケースに対して0引数のクロージャを生成しました。
<!-- EN -->
This is not really necessary, since each test case is simply run once in place.
> これは実際には必要ではありません。なぜなら、各テストケースは単に1回だけ実行されるからです。
<!-- EN -->
Therefore I modified `@test` to expand to a try-catch block that records the test result (true, false, or exception raised) and calls the test suite handler on it.
> そのため、テスト結果（true、false、またはexception raised）を記録し、その上にテストスイートハンドラを呼び出すtry-catchブロックに展開するように `@test` を修正しました。
 