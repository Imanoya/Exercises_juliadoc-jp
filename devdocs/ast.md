# Julia ASTs

<!-- EN -->
Julia has two representations of code. First there is a surface syntax AST returned by the parser (e.g. the [`parse`](@ref) function), and manipulated by macros.
<!-- JP -->
>Juliaには2つのコードがあります。 最初に、パーサによって返された表面構文AST(例えば、[`parse`](@ ref)関数)があり、マクロによって操作されます。
<!-- EN -->
It is a structured representation of code as it is written, constructed by `julia-parser.scm` from a character stream.
<!-- JP -->
>これは、文字ストリームから `julia-parser.scm`によって作成されたコードの構造化された表現です。
<!-- EN -->
Next there is a lowered form, or IR (intermediate representation), which is used by type inference and code generation.
<!-- JP -->
>次に、タイプ推論とコード生成によって使用される、lower形式、または IR(中間表現)があります。
<!-- EN -->
In the lowered form there are fewer types of nodes, all macros are expanded, and all control flow is converted to explicit branches and sequences of statements.
<!-- JP -->
>lower形式 では、ノードの種類が少なくなり、すべてのマクロが展開され、すべての制御フローが明示的な分岐とステートメントのシーケンスに変換されます。
<!-- EN -->
The lowered form is constructed by `julia-syntax.scm`.
<!-- JP -->
>lower形式 は `julia-syntax.scm` によって構築されます。

<!-- EN -->
First we will focus on the lowered form, since it is more important to the compiler.
<!-- JP -->
>最初に、lower形式 に焦点を当てます、ここがコンパイラーにとりとても重要です。
<!-- EN -->
It is also less obvious to the human, since it results from a significant rearrangement of the input syntax.
<!-- JP -->
>人間にとって、入力構文の大幅な再構成の結果であるため判りやすくはありません。


## Lowered form

<!-- EN -->
The following data types exist in lowered form:
<!-- JP -->
>次のデータ型は、lower形式 で存在します。

  * `Expr`

    <!-- EN -->
    Has a node type indicated by the `head` field, and an `args` field which is a `Vector{Any}` of subexpressions.
    <!-- JP -->
    > `head`フィールドで指定されたノード型と、部分式の `Vector{Any}` である `args` フィールドを持っています。

  * `Slot`

    <!-- EN -->
    Identifies arguments and local variables by consecutive numbering.
    <!-- JP -->
    > 引数とローカル変数を連続した番号で識別します。
    <!-- EN -->
    `Slot` is an abstract type with subtypes `SlotNumber` and `TypedSlot`.
    <!-- JP -->
    > `Slot` はサブタイプ `SlotNumber` と `TypedSlot` を持つ抽象型です。
    <!-- EN -->
    Both types have an integer-valued `id` field giving the slot index.
    <!-- JP -->
    > どちらの型も、スロットインデックスを与える整数値の `id`フィールドを持っています。
    <!-- EN -->
    Most slots have the same type at all uses, and so are represented with `SlotNumber`.
    <!-- JP -->
    > ほとんどのスロットはすべての用途で同じタイプを持ち、したがって `SlotNumber`で表されます。
    <!-- EN -->
    The types of these slots are found in the `slottypes` field of their `MethodInstance` object.
    <!-- JP -->
    > `MethodInstance` オブジェクトの `slottypes` フィールドにあります。
    <!-- EN -->
    Slots that require per-use type annotations are represented with `TypedSlot`, which has a `typ` field.
    <!-- JP -->
    > per-use型の注釈を必要とするスロットは `TypedSlot`で表され、` typ`フィールドを持ちます。

  * `CodeInfo`

    <!-- EN -->
    Wraps the IR of a method.
    <!-- JP -->
    > メソッドのIRをラップします。

  * `LineNumberNode`

    <!-- EN -->
    Contains a single number, specifying the line number the next statement came from.
    <!-- JP -->
    > 次のステートメントが来た行番号を指定して、単一の数値を含みます。

  * `LabelNode`

    <!-- EN -->
    Branch target, a consecutively-numbered integer starting at 0.
    <!-- JP -->
    > 分岐先のターゲット.0から始まる連続番号の整数。

  * `GotoNode`

    <!-- EN -->
    Unconditional branch.
    <!-- JP -->
    > 無条件分岐。

  * `QuoteNode`

    <!-- EN -->
    Wraps an arbitrary value to reference as data.
    <!-- JP -->
    > データとして参照する任意の値をラップします。
    <!-- EN -->
    For example, the function `f() = :a` contains a `QuoteNode` whose `value` field is the symbol `a`, in order to return the symbol itself instead of evaluating it.
    <!-- JP -->
    > 例えば、関数 `f()=：a`は` value`フィールドがシンボル `a`である` QuoteNode`を含んでいます。シンボル自体を評価するのではなく、それを返すためです。

  * `GlobalRef`

    <!-- EN -->
    Refers to global variable `name` in module `mod`.
    <!-- JP -->
    > モジュール `mod`のグローバル変数` name`を参照してください。

  * `SSAValue`

    <!-- EN -->
    Refers to a consecutively-numbered (starting at 0) static single assignment (SSA) variable inserted by the compiler.
    <!-- JP -->
    > コンパイラーによって挿入された連続した番号のスタティック・シングル・アサインメント(SSA)変数を指します。

  * `NewvarNode`

    <!-- EN -->
    Marks a point where a variable is created. This has the effect of resetting a variable to undefined.
    <!-- JP -->
    > 変数が作成されるポイントをマークします。これは、変数を未定義にリセットする効果があります。


### Expr types

<!-- EN -->
These symbols appear in the `head` field of `Expr`s in lowered form.
<!-- JP -->
> これらのシンボルは `head` フィールドの `Expr` に lower形式 で現れます。

  * `call`

    <!-- EN -->
    Function call (dynamic dispatch). `args[1]` is the function to call, `args[2:end]` are the arguments.
    <!-- JP -->
    > 関数呼び出し(動的ディスパッチ)。 `args[1]` は呼び出す関数であり、 `args [2:end]` は引数です。

  * `invoke`

    <!-- EN -->
    Function call (static dispatch).
    <!-- JP -->
    > 関数呼び出し(静的ディスパッチ)。
    <!-- EN -->
    `args[1]` is the MethodInstance to call, `args[2:end]` are the arguments (including the function that is being called, at `args[2]`).
    <!-- JP -->
    > `args[1]` は MethodInstance to call であり、 `args [2:end]` は引数( `args[2]` で呼び出されている関数を含む) です。

  * `static_parameter`

    <!-- EN -->
    Reference a static parameter by index.
    <!-- JP -->
    > 静的パラメータをインデックスで参照します。

  * `gotoifnot`

    <!-- EN -->
    Conditional branch. If `args[1]` is false, goes to label identified in `args[2]`.
    <!-- JP -->
    > 条件分岐。 `args [1]`がfalseの場合、 `args [2]`で指定されたラベルに行きます。

  * `=`

    <!-- EN -->
    Assignment.
    <!-- JP -->
    > 割り当て。

 * `method`

    <!-- EN -->
    Adds a method to a generic function and assigns the result if necessary.
    <!-- JP -->
    > ジェネリック関数にメソッドを追加し、必要に応じて結果を割り当てます。

    <!-- EN -->
    Has a 1-argument form and a 4-argument form.
    <!-- JP -->
    > 1引数形式と4引数形式があります。
    <!-- EN -->
    The 1-argument form arises from the syntax `function foo end`.
    <!-- JP -->
    > 1引数形式は `function foo end`という構文から生まれます。
    <!-- EN -->
    In the 1-argument form, the argument is a symbol.
    <!-- JP -->
    > 1引数形式では、引数はシンボルです。
    <!-- EN -->
    If this symbol already names a function in the current scope, nothing happens.
    <!-- JP -->
    > このシンボルが現在のスコープ内の関数の名前を既に持っている場合、何も起こりません。
    <!-- EN -->
    If the symbol is undefined, a new function is created and assigned to the identifier specified by the symbol.
    <!-- JP -->
    > シンボルが未定義の場合、新しい関数が作成され、シンボルで指定された識別子に割り当てられます。
    <!-- EN -->
    If the symbol is defined but names a non-function, an error is raised. The definition of "names a function" is that the binding is constant, and refers to an object of singleton type.
    <!-- JP -->
    > シンボルが定義されていても、関数の名前でない場合、エラーが発生します。 "関数の名前"の定義は、バインディングが一定であり、シングルトン型のオブジェクトを参照することです。
    <!-- EN -->
    The rationale for this is that an instance of a singleton type uniquely identifies the type to add the method to.
    <!-- JP -->
    > これの根拠は、シングルトンタイプのインスタンスがメソッドを追加するタイプを一意に識別することです。
    <!-- EN -->
    When the type has fields, it wouldn't be clear whether the method was being added to the instance or its type.
    <!-- JP -->
    > 型にフィールドがある場合、そのメソッドがインスタンスに追加されているのか、その型に追加されたのかは明らかではありません。

    <!-- EN -->
    The 4-argument form has the following arguments:
    <!-- JP -->
    > 4引数形式の引数は次のとおりです。

      * `args[1]`

        <!-- EN -->
        A function name, or `false` if unknown.
        <!-- JP -->
        > 関数名。未知の場合は `false`。
        <!-- EN -->
        If a symbol, then the expression first behaves like the 1-argument form above.
        <!-- JP -->
        > シンボルの場合、式は最初に上記の1引数の形式のように動作します。
        <!-- EN -->
        This argument is ignored from then on.
        <!-- JP -->
        > その後、この引数は無視されます。
        <!-- EN -->
        When this is `false`, it means a method is being added strictly by type, `(::T)(x) = x`.
        <!-- JP -->
        > これが `false`の場合、メソッドが`(:: T)(x)= x`型で厳密に追加されていることを意味します。

      * `args[2]`

        <!-- EN -->
        A `SimpleVector` of argument type data.
        <!-- JP -->
        > 引数型データの `SimpleVector`。
        <!-- EN -->
        `args[2][1]` is a `SimpleVector` of the argument types, and `args[2][2]` is a `SimpleVector` of type variables corresponding to the method's static parameters.
        <!-- JP -->
        > `args [2] [1]` は引数型の `SimpleVector`であり、 `args [2][2]` はメソッドの静的パラメータに対応する型変数の `SimpleVector` です。

      * `args[3]`

        <!-- EN -->
        A `CodeInfo` of the method itself.
        <!-- JP -->
        > メソッド自体の `CodeInfo`。
        <!-- EN -->
        For "out of scope" method definitions (adding a method to a function that also has methods defined in different scopes) this is an expression that evaluates to a `:lambda` expression.
        <!-- EN -->
        <!-- JP -->
        > "範囲外" のメソッド定義(異なるスコープで定義されたメソッドを持つ関数にメソッドを追加する)の場合、これは `:lambda` 式に評価される式です。

      * `args[4]`

        <!-- EN -->
        `true` or `false`, identifying whether the method is staged (`@generated function`).
        <!-- JP -->
        > メソッドがステージングされているかどうかを識別する `true` または `false` を返します( `@generated function`)。

  * `const`

    <!-- EN -->
    Declares a (global) variable as constant.
    <!-- JP -->
    > (グローバル)変数を定数として宣言します。
  * `null`

    <!-- EN -->
    Has no arguments; simply yields the value `nothing`.
    <!-- JP -->
    > 引数はありません。単純に値「nothing」を生成します。

  * `new`

    <!-- EN -->
    Allocates a new struct-like object.
    <!-- JP -->
    > 新しい構造体のようなオブジェクトを割り当てます。
    <!-- EN -->
    First argument is the type. The [`new`](@ref) pseudo-function is lowered to this, and the type is always inserted by the compiler.
    <!-- JP -->
    > 最初の引数は型です。 [`new`](@ref)擬似関数はこれより低くなり、型は常にコンパイラによって挿入されます。
    <!-- EN -->
    This is very much an internal-only feature, and does no checking.
    <!-- JP -->
    > これは内部のみの機能であり、チェックはしません。
    <!-- EN -->
    Evaluating arbitrary `new` expressions can easily segfault.
    <!-- JP -->
    > 任意の `new` 式を評価することで、簡単に分離することができます。

  * `return`

    <!-- EN -->
    Returns its argument as the value of the enclosing function.
    <!-- JP -->
    > 引数を、囲み関数の値として返します。

  * `the_exception`

    <!-- EN -->
    Yields the caught exception inside a `catch` block.
    <!-- JP -->
    > キャッチされた例外を `catch`ブロックの中で生成します。
    <!-- EN -->
    This is the value of the run time system variable `jl_exception_in_transit`.
    <!-- JP -->
    > これは実行時システム変数 `jl_exception_in_transit`の値です。

  * `enter`

    <!-- EN -->
    Enters an exception handler (`setjmp`).
    <!-- JP -->
    > 例外ハンドラ( `setjmp`)を入力します。
    <!-- EN -->
    `args[1]` is the label of the catch block to jump to on error.
    <!-- JP -->
    > `args [1]`はエラー時にジャンプするcatchブロックのラベルです。

  * `leave`

    <!-- EN -->
    Pop exception handlers.
    <!-- JP -->
    > ポップ例外ハンドラ。
    <!-- EN -->
    `args[1]` is the number of handlers to pop.
    <!-- JP -->
    > `args [1]` はポップするハンドラの数です。

  * `inbounds`

    <!-- EN -->
    Controls turning bounds checks on or off.
    <!-- JP -->
    > 境界チェックをオンまたはオフにするコントロール。
    <!-- EN -->
    A stack is maintained; if the first argument of this expression is true or false (`true` means bounds checks are disabled), it is pushed onto the stack.
    <!-- JP -->
    > スタックは維持されます。この式の最初の引数が真または偽の場合( `true` は境界チェックが無効であることを意味します)、スタックにプッシュされます。
    <!-- EN -->
    If the first argument is `:pop`, the stack is popped.
    <!-- JP -->
    > 最初の引数が `:pop` の場合、スタックはポップされます。

  * `boundscheck`

    <!-- EN -->
    Has the value `false` if inlined into a section of code marked with `@inbounds`, otherwise has the value `true`.
    <!-- JP -->
    > `@inbounds` でマークされたコードのセクションにインライン化されている場合は `false` を返し、それ以外の場合は `true` を返します。
    <!-- JP -->
    > `false` を返す場合は、インライン化されているコードのセクションに `@inbounds` でマークされた場合で 、それ以外は `true` を返します。

  * `copyast`

    <!-- EN -->
    Part of the implementation of quasi-quote.
    <!-- JP -->
    > 準引用の実装の一部。
    <!-- EN -->
    The argument is a surface syntax AST that is simply copied recursively and returned at run time.
    <!-- JP -->
    > 引数はサーフェス構文ASTで、単純に再帰的にコピーされ、実行時に返されます。

  * `meta`

    <!-- EN -->
    Metadata. `args[1]` is typically a symbol specifying the kind of metadata, and the rest of the arguments are free-form.
    <!-- JP -->
    > メタデータ。 `args [1]`は通常、メタデータの種類を指定するシンボルであり、残りの引数は自由形式です。
    <!-- EN -->
    The following kinds of metadata are commonly used:
    <!-- JP -->
    > 次の種類のメタデータが一般的に使用されます。

      <!-- EN -->
      * `:inline` and `:noinline`: Inlining hints.
      <!-- JP -->
      > * `：inline`と`：noinline`：インラインヒント。

      <!-- EN -->
      * `:push_loc`: enters a sequence of statements from a specified source location.
      <!-- JP -->
      * > `：push_loc`：指定されたソースの場所から一連のステートメントを入力します。

        <!-- EN -->
        * `args[2]` specifies a filename, as a symbol.
        <!-- JP -->
        > * `args [2]`はシンボルとしてファイル名を指定します。
        <!-- EN -->
        * `args[3]` optionally specifies the name of an (inlined) function that originally contained the code.
        <!-- JP -->
        > * `args [3]`は、コードが元々含んでいた(インライン化された)関数の名前をオプションで指定します。

      <!-- EN -->
      * `:pop_loc`: returns to the source location before the matching `:push_loc`.
      <!-- JP -->
      * > `：pop_loc`：一致する`：push_loc`の前のソース位置に戻ります。

          <!-- EN -->
          * `args[2]::Int` (optional) specifies the number of `push_loc` to pop
          <!-- JP -->
          > * `args [2] :: Int`(オプション)はpopするための` push_loc`の数を指定します


### Method

<!-- EN -->
A unique'd container describing the shared metadata for a single method.
<!-- JP -->
> 単一のメソッドの共有メタデータを記述するユニークなコンテナ。

  * `name`, `module`, `file`, `line`, `sig`

    <!-- EN -->
    Metadata to uniquely identify the method for the computer and the human.
    <!-- JP -->
    > コンピュータと人間のためのメソッドを一意に識別するメタデータ。

  * `ambig`

    <!-- EN -->
    Cache of other methods that may be ambiguous with this one.
    <!-- JP -->
    > このメソッドとあいまいな他のメソッドのキャッシュ。

  * `specializations`

    <!-- EN -->
    Cache of all MethodInstance ever created for this Method, used to ensure uniqueness.
    <!-- JP -->
    > このメソッド用に作成されたすべてのMethodInstanceのキャッシュ。一意性を保証するために使用されます。
    <!-- EN -->
    Uniqueness is required for efficiency, especially for incremental precompile and tracking of method invalidation.
    <!-- JP -->
    > 特にメソッドの無効化のインクリメンタル・プリコンパイルとトラッキングでは、効率のために一意性が必要です。

  * `source`

    <!-- EN -->
    The original source code (usually compressed).
    <!-- JP -->
    > オリジナルのソースコード(通常は圧縮されている)。

  * `roots`

    <!-- EN -->
    Pointers to non-AST things that have been interpolated into the AST, required by compression of the AST, type-inference, or the generation of native code.
    <!-- JP -->
    > ASTの補間、型推論、またはネイティブコードの生成によって必要とされる、ASTに補間された非ASTのものへのポインタ。

  * `nargs`, `isva`, `called`, `isstaged`, `pure`

    <!-- EN -->
    Descriptive bit-fields for the source code of this Method.
    <!-- JP -->
    > このメソッドのソースコードのための記述的なビットフィールド。

  * `min_world` / `max_world`

    <!-- EN -->
    The range of world ages for which this method is visible to dispatch.
    <!-- JP -->
    > このメソッドをディスパッチするために表示できる世界の世代の範囲。


### MethodInstance

<!-- EN -->
A unique'd container describing a single callable signature for a Method.
<!-- JP -->
> メソッドの単一の呼び出し可能な署名を記述するユニークなコンテナ。
<!-- EN -->
See especially [Proper maintenance and care of multi-threading locks](@ref) for important details on how to modify these fields safely.
<!-- JP -->
> これらのフィールドを安全に変更する方法の重要な詳細については、特に[マルチスレッドロックの適切な保守と管理](@ ref)を参照してください。

  * `specTypes`

    <!-- EN -->
    The primary key for this MethodInstance. Uniqueness is guaranteed through a `def.specializations` lookup.
    <!-- JP -->
    > このMethodInstanceの主キー一意性は `def.specializations`ルックアップによって保証されます。

  * `def`

    <!-- EN -->
    The `Method` that this function describes a specialization of.
    <!-- JP -->
    > この関数が特殊化を記述する `Method`。
    <!-- EN -->
    Or a `Module`, if this is a top-level Lambda expanded in Module, and which is not part of a Method.
    <!-- JP -->
    > モジュール内で展開されたトップレベルのラムダで、メソッドの一部でない場合は `Module`を返します。

  * `sparam_vals`

    <!-- EN -->
    The values of the static parameters in `specTypes` indexed by `def.sparam_syms`.
    <!-- JP -->
    > `def.sparam_syms`によってインデックス付けされた` specTypes`の静的パラメータの値です。
    <!-- EN -->
    For the `MethodInstance` at `Method.unspecialized`, this is the empty `SimpleVector`.
    <!-- JP -->
    > `Method.unspecialized`の` MethodInstance`の場合、これは空の `SimpleVector`です。
    <!-- EN -->
    But for a runtime `MethodInstance` from the `MethodTable` cache, this will always be defined and indexable.
    <!-- JP -->
    > しかし、 `MethodTable`キャッシュからの実行時` MethodInstance`の場合、これは常に定義され、インデックス可能です。

  * `rettype`

    <!-- EN -->
    The inferred return type for the `specFunctionObject` field, which (in most cases) is also the computed return type for the function in general.
    <!-- JP -->
    > `specFunctionObject`フィールドの推定された戻り値の型です。関数の一般的な戻り値の型です。

  * `inferred`

    <!-- EN -->
    May contain a cache of the inferred source for this function, or other information about the inference result such as a constant return value may be put here (if `jlcall_api == 2`), or it could be set to `nothing` to just indicate `rettype` is inferred.
    <!-- JP -->
    > この関数の推測されたソースのキャッシュを含むか、または定数戻り値のような推論結果に関するその他の情報がここに置かれるかもしれません( `jlcall_api == 2` ならば)。 `rettype` が推論されることを示します。

  * `ftpr`

    <!-- EN -->
    The generic jlcall entry point.
    <!-- JP -->
    > ジェネリックjlcallエントリポイント。

  * `jlcall_api`

    <!-- EN -->
    The ABI to use when calling `fptr`.
    <!-- JP -->
    > `fptr`を呼び出すときに使うABI。
    <!-- EN -->
    Some significant ones include:
    <!-- JP -->
    > 重要なものには次のものがあります：

      <!-- EN -->
      * 0 - Not compiled yet
      <!-- JP -->
      > * 0 - まだコンパイルされていません
      <!-- EN -->
      * 1 - JL_CALLABLE `jl_value_t *(*)(jl_function_t *f, jl_value_t *args[nargs], uint32_t nargs)`
      <!-- JP -->
      > * 1 - JL_CALLABLE `jl_value_t *(*)(jl_function_t * f、jl_value_t * args [nargs]、uint32_t nargs)`
      <!-- EN -->
      * 2 - Constant (value stored in `inferred`)
      <!-- JP -->
      > * 2 - 定数( `推論 'に格納された値)
      <!-- EN -->
      * 3 - With Static-parameters forwarded `jl_value_t *(*)(jl_svec_t *sparams, jl_function_t *f, jl_value_t *args[nargs], uint32_t nargs)`
      <!-- JP -->
      > * 3 - 静的パラメータが転送されたとき `jl_value_t *(*)(jl_svec_t * sparams、jl_function_t * f、jl_value_t * args [nargs],uint32_t nargs)`
      <!-- EN -->
      * 4 - Run in interpreter `jl_value_t *(*)(jl_method_instance_t *meth, jl_function_t *f, jl_value_t *args[nargs], uint32_t nargs)`
      <!-- JP -->
      > * 4 - インタプリタ `jl_value_t *(*)(jl_method_instance_t * meth、jl_function_t * f、jl_value_t * args [nargs]、uint32_t nargs)` で実行します。

  * `min_world` / `max_world`

    <!-- EN -->
    The range of world ages for which this method instance is valid to be called.
    <!-- JP -->
    > このメソッドインスタンスが呼び出されるのに有効なワールド年代の範囲。


### CodeInfo

A temporary container for holding lowered source code.

  * `code`

    An `Any` array of statements

  * `slotnames`

    An array of symbols giving the name of each slot (argument or local variable).

  * `slottypes`

    An array of types for the slots.

  * `slotflags`

    A `UInt8` array of slot properties, represented as bit flags:

      * 2  - assigned (only false if there are *no* assignment statements with this var on the left)
      * 8  - const (currently unused for local variables)
      * 16 - statically assigned once
      * 32 - might be used before assigned. This flag is only valid after type inference.

  * `ssavaluetypes`

    Either an array or an `Int`.

    If an `Int`, it gives the number of compiler-inserted temporary locations in the function.
    If an array, specifies a type for each location.

Boolean properties:

  * `inferred`

    Whether this has been produced by type inference.

  * `inlineable`

    Whether this should be inlined.

  * `propagate_inbounds`

    Whether this should should propagate `@inbounds` when inlined for the purpose of eliding `@boundscheck` blocks.

  * `pure`

    Whether this is known to be a pure function of its arguments, without respect to the state of the method caches or other mutable global state.


## Surface syntax AST

<!-- EN -->
Front end ASTs consist almost entirely of `Expr`s and atoms (e.g. symbols, numbers).
<!-- JP -->
> フロントエンドのASTは、ほとんどが `Expr`とアトム(シンボル、数字など)で構成されています。
<!-- EN -->
There is generally a different expression head for each visually distinct syntactic form.
<!-- JP -->
> 一般的には、視覚的に異なる構文的フォームごとに異なる表現ヘッドが存在する。
<!-- EN -->
Examples will be given in s-expression syntax.
<!-- JP -->
> 例はs-expression構文で与えられます。
<!-- EN -->
Each parenthesized list corresponds to an Expr, where the first element is the head.
<!-- JP -->
> 各括弧で囲まれたリストはExprに対応しています。最初の要素は先頭です。
<!-- EN -->
For example `(call f x)` corresponds to `Expr(:call, :f, :x)` in Julia.
<!-- JP -->
> 例えば `(call f x)` はJuliaの  `Expr(:call,;f,:x)` に対応します。

### Calls

| Input            | AST                                |
|:---------------- |:---------------------------------- |
| `f(x)`           | `(call f x)`                       |
| `f(x, y=1, z=2)` | `(call f x (kw y 1) (kw z 2))`     |
| `f(x; y=1)`      | `(call f (parameters (kw y 1)) x)` |
| `f(x...)`        | `(call f (... x))`                 |

`do` syntax:

```julia
f(x) do a,b
    body
end
```

parses as `(do (call f x) (-> (tuple a b) (block body)))`.

### Operators

<!-- EN -->
Most uses of operators are just function calls, so they are parsed with the head `call`.
<!-- JP -->
> 演算子の大部分の使用は単に関数呼び出しであるため、 `call`という頭で解析されます。
<!-- EN -->
However some operators are special forms (not necessarily function calls), and in those cases the operator itself is the expression head.
<!-- JP -->
> しかし、一部の演算子は特別な形式(必ずしも関数呼び出しである必要はない)であり、その場合演算子自体が式の頭です。
<!-- EN -->
In julia-parser.scm these are referred to as "syntactic operators".
<!-- JP -->
> julia-parser.scmでは、これらを「構文演算子」と呼びます。
<!-- EN -->
Some operators (`+` and `*`) use N-ary parsing; chained calls are parsed as a single N-argument call. Finally, chains of comparisons have their own special expression structure.
<!-- JP -->
> 一部の演算子( `+`と `*`)では、N進解析を使用します。 連鎖された呼び出しは単一のN引数呼び出しとして解析されます。 最後に、比較の連鎖には独自の特殊な表現構造があります。

| Input       | AST                       |
|:----------- |:------------------------- |
| `x+y`       | `(call + x y)`            |
| `a+b+c+d`   | `(call + a b c d)`        |
| `2x`        | `(call * 2 x)`            |
| `a&&b`      | `(&& a b)`                |
| `x += 1`    | `(+= x 1)`                |
| `a ? 1 : 2` | `(if a 1 2)`              |
| `a:b`       | `(: a b)`                 |
| `a:b:c`     | `(: a b c)`               |
| `a,b`       | `(tuple a b)`             |
| `a==b`      | `(call == a b)`           |
| `1<i<=n`    | `(comparison 1 < i <= n)` |
| `a.b`       | `(. a (quote b))`         |
| `a.(b)`     | `(. a b)`                 |

### Bracketed forms

| Input                    | AST                                  |
|:------------------------ |:------------------------------------ |
| `a[i]`                   | `(ref a i)`                          |
| `t[i;j]`                 | `(typed_vcat t i j)`                 |
| `t[i j]`                 | `(typed_hcat t i j)`                 |
| `t[a b; c d]`            | `(typed_vcat t (row a b) (row c d))` |
| `a{b}`                   | `(curly a b)`                        |
| `a{b;c}`                 | `(curly a (parameters c) b)`         |
| `[x]`                    | `(vect x)`                           |
| `[x,y]`                  | `(vect x y)`                         |
| `[x;y]`                  | `(vcat x y)`                         |
| `[x y]`                  | `(hcat x y)`                         |
| `[x y; z t]`             | `(vcat (row x y) (row z t))`         |
| `[x for y in z, a in b]` | `(comprehension x (= y z) (= a b))`  |
| `T[x for y in z]`        | `(typed_comprehension T x (= y z))`  |
| `(a, b, c)`              | `(tuple a b c)`                      |
| `(a; b; c)`              | `(block a (block b c))`              |

### Macros

| Input         | AST                                          |
|:------------- |:-------------------------------------------- |
| `@m x y`      | `(macrocall @m (line) x y)`                  |
| `Base.@m x y` | `(macrocall (. Base (quote @m)) (line) x y)` |
| `@Base.m x y` | `(macrocall (. Base (quote @m)) (line) x y)` |

### Strings

| Input           | AST                                 |
|:--------------- |:----------------------------------- |
| `"a"`           | `"a"`                               |
| `x"y"`          | `(macrocall @x_str (line) "y")`     |
| `x"y"z`         | `(macrocall @x_str (line) "y" "z")` |
| `"x = $x"`      | `(string "x = " x)`                 |
| ``` `a b c` ``` | `(macrocall @cmd (line) "a b c")`   |

Doc string syntax:

```julia
"some docs"
f(x) = x
```

parses as `(macrocall (|.| Core '@doc) (line) "some docs" (= (call f x) (block x)))`.

### Imports and such

| Input               | AST                                          |
|:------------------- |:-------------------------------------------- |
| `import a`          | `(import (. a))`                             |
| `import a.b.c`      | `(import (. a b c))`                         |
| `import ...a`       | `(import (. . . . a))`                       |
| `import a.b, c.d`   | `(import (. a b) (. c d))`                   |
| `import Base: x`    | `(import (: (. Base) (. x)))`                |
| `import Base: x, y` | `(import (: (. Base) (. x) (. y)))`          |
| `export a, b`       | `(export a b)`                               |

### Numbers

<!-- EN -->
Julia supports more number types than many scheme implementations, so not all numbers are represented directly as scheme numbers in the AST.
<!-- JP -->
> Juliaは、多くのスキーマの実装よりも多くの数値タイプをサポートしているため、すべての数値がASTのスキーム番号として直接表されるわけではありません。

| Input                   | AST                                                     |
|:----------------------- |:------------------------------------------------------- |
| `11111111111111111111`  | `(macrocall @int128_str (null) "11111111111111111111")` |
| `0xfffffffffffffffff`   | `(macrocall @uint128_str (null) "0xfffffffffffffffff")` |
| `1111...many digits...` | `(macrocall @big_str (null) "1111....")`                |

### Block forms

<!-- EN -->
A block of statements is parsed as `(block stmt1 stmt2 ...)`.
<!-- JP -->
> 文のブロックは `(block stmt1 stmt2 ...)`として解析されます。

<!-- EN -->
If statement:
<!-- JP -->
> ifステートメント:

```julia
if a
    b
elseif c
    d
else
    e
end
```

<!-- EN -->
parses as:
<!-- JP -->
> 次のように解析します。

```
(if a (block (line 2) b)
    (elseif (block (line 3) c) (block (line 4) d)
            (block (line 5 e))))
```

<!-- EN -->
A `while` loop parses as `(while condition body)`.
<!-- JP -->
> `while`ループは`(while while condition body) `として解析されます。

<!-- EN -->
A `for` loop parses as `(for (= var iter) body)`.
<!-- JP -->
> `for`ループは`(for(= var iter)body) `として解析されます。
<!-- EN -->
If there is more than one iteration specification, they are parsed as a block: `(for (block (= v1 iter1) (= v2 iter2)) body)`.
<!-- JP -->
> 複数の反復仕様がある場合、それらはブロックとして解析されます： `(for(block(= v1 iter1)(= v2 iter2))body)`。

<!-- EN -->
`break` and `continue` are parsed as 0-argument expressions `(break)` and `(continue)`.
<!-- JP -->
> `break`と` continue`は0引数の式 `(break)`と `(continue)`として解析されます。

<!-- EN -->
`let` is parsed as `(let (= var val) body)` or `(let (block (= var1 val1) (= var2 val2) ...) body)`, like `for` loops.
<!-- JP -->
> `let`は` for(let(= var val)body) `または`(let(ブロック(= var1 val1)(= var2 val2)...)body) `として解析されます。

<!-- EN -->
A basic function definition is parsed as `(function (call f x) body)`.
<!-- JP -->
> 基本的な関数定義は `(function(call f x)body)`として解析されます。
<!-- EN -->
A more complex example:
<!-- JP -->
> より複雑な例：

```julia
function f(x::T; k = 1) where T
    return x+1
end
```

<!-- EN -->
parses as:
<!-- JP -->
> 次のように解析します。

```
(function (where (call f (parameters (kw k 1))
                       (:: x T))
                 T)
          (block (line 2) (return (call + x 1))))
```

<!-- EN -->
Type definition:
<!-- JP -->
> タイプ定義：

```julia
mutable struct Foo{T<:S}
    x::T
end
```

<!-- EN -->
parses as:
<!-- JP -->
> 次のように解析します。

```
(struct true (curly Foo (<: T S))
        (block (line 2) (:: x T)))
```

<!-- EN -->
The first argument is a boolean telling whether the type is mutable.
<!-- JP -->
> 最初の引数は、型が変更可能かどうかを示すブール値です。

<!-- EN -->
`try` blocks parse as `(try try_block var catch_block finally_block)`.
<!-- JP -->
> `try`ブロックは`(try try_block var catch_block finally_blockを試してください) `として解析します。
<!-- EN -->
If no variable is present after `catch`, `var` is `#f`. If there is no `finally` clause, then the last argument is not present.
<!-- JP -->
> `catch`の後に変数がない場合、` var`は `#f`です。 `finally`節がなければ、最後の引数は存在しません。

### Quote expressions

<!-- EN -->
Julia source syntax forms for code quoting (`quote` and `:( )`) support interpolation with `$`.
<!-- JP -->
> コード引用のためのJuliaソース構文形式( `quote` と `:()` ) は `$` での補間をサポートしています。
<!-- EN -->
In Lisp terminology, this means they are actually "backquote" or "quasiquote" forms.
<!-- JP -->
> Lispの用語では、これは実際には "backquote"または "quasiquote"形式であることを意味します。
<!-- EN -->
Internally, there is also a need for code quoting without interpolation.
<!-- JP -->
> 内部的には、補間のないコード引用も必要です。
<!-- EN -->
In Julia's scheme code, non-interpolating quote is represented with the expression head `inert`.
<!-- JP -->
> Juliaの方式コードでは、非補間式の見積もりは、表現ヘッド `inert` で表されます。

<!-- EN -->
`inert` expressions are converted to Julia `QuoteNode` objects.
<!-- JP -->
> `inert` 式はJulia `QuoteNode` オブジェクトに変換されます。
<!-- EN -->
These objects wrap a single value of any type, and when evaluated simply return that value.
<!-- JP -->
> これらのオブジェクトは任意の型の単一の値をラップし、評価されるとその値を単に返します。

<!-- EN -->
A `quote` expression whose argument is an atom also gets converted to a `QuoteNode`.
<!-- JP -->
> 引数がアトムである `quote`式も` QuoteNode`に変換されます。

### Line numbers

<!-- EN -->
Source location information is represented as `(line line_num file_name)` where the third component is optional (and omitted when the current line number, but not file name, changes).
<!-- JP -->
> ソースの位置情報は `(line line_num file_name)`として表され、第3の要素は任意である(現在の行番号がファイル名ではなく省略された場合は省略される)。

<!-- EN -->
These expressions are represented as `LineNumberNode`s in Julia.
<!-- JP -->
> これらの式は、Juliaの `LineNumberNode`として表されます。
