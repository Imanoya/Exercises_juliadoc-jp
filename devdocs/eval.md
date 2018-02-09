# Eval of Julia code

One of the hardest parts about learning how the Julia Language runs code is learning how all of the pieces work together to execute a block of code.
> Julia Languageがどのようにコードを実行するかを学ぶ上で最も難しい部分の1つは、すべての部分がどのように連携してコードブロックを実行するかを学習することです。

Each chunk of code typically makes a trip through many steps with potentially unfamiliar names, such as (in no particular order): flisp, AST, C++, LLVM, `eval`, `typeinf`, `macroexpand`, sysimg (or system image), bootstrapping, compile, parse, execute, JIT, interpret, box, unbox, intrinsic function, and primitive function, before turning into the desired result (hopefully).
> flisp、AST、C ++、LLVM、 `eval`、` typeinf`、 `macroexpand`、sysimg（またはシステムイメージ）のような、潜在的によく知られていない名前を持つ多くのステップを、 ）、ブートストラップ、コンパイル、パース、実行、JIT、解釈、ボックス、アンボックス、組み込み関数、およびプリミティブ関数を使用します。

!!! sidebar "Definitions"
      * REPL

        REPL stands for Read-Eval-Print Loop. It's just what we call the command line environment for short.
        > REPLはRead-Eval-Printループの略です。 単にコマンドライン環境と呼んでいます。
      * AST

        Abstract Syntax Tree The AST is the digital representation of the code structure.
        In this form the code has been tokenized for meaning so that it is more suitable for manipulation and execution.
        > 抽象構文木ASTは、コード構造のデジタル表現です。
        > この形式では、操作と実行により適したコードになっています。

## Julia Execution

The 10,000 foot view of the whole process is as follows:

1. The user starts `julia`.
> 1. ユーザはジュリアを開始する。
2. The C function `main()` from `ui/repl.c` gets called.
> 2. `ui / repl.c`のC関数` main（） `が呼び出されます。
   This function processes the command line arguments, filling in the `jl_options` struct and setting the variable `ARGS`.
   > この関数は、コマンドライン引数を処理し、 `jl_options`構造体を埋め込み、変数` ARGS`を設定します。
   It then initializes Julia (by calling [`julia_init` in `task.c`](https://github.com/JuliaLang/julia/blob/master/src/task.c), which may load a previously compiled [sysimg](@ref dev-sysimg)).
   > それからJuliaを初期化します（ `task.c`の` `julia_init`（https://github.com/JuliaLang/julia/blob/master/src/task.c）を呼び出します）。これは以前にコンパイルされた[sysimg ]（@ ref dev-sysimg））。
   Finally, it passes off control to Julia by calling [`Base._start()`](https://github.com/JuliaLang/julia/blob/master/base/client.jl).
   > 最後に、[`Base._start（）`]（https://github.com/JuliaLang/julia/blob/master/base/client.jl）を呼び出すことで、Juliaの制御をパスします。
3. When `_start()` takes over control, the subsequent sequence of commands depends on the command line arguments given.
> 3. `_start（） 'が制御を引き継ぐとき、後続のコマンドのシーケンスは与えられたコマンドライン引数に依存します。
   For example, if a filename was supplied, it will proceed to execute that file.
   > たとえば、ファイル名が指定されていれば、そのファイルを実行します。
   Otherwise, it will start an interactive REPL.
   > それ以外の場合は、対話型REPLを開始します。
4. Skipping the details about how the REPL interacts with the user, let's just say the program ends up with a block of code that it wants to run.
> 4. REPLがユーザーとどのように対話するかについての詳細をスキップして、プログラムが実行したいコードのブロックで終わったとしましょう。
5. If the block of code to run is in a file, [`jl_load(char *filename)`](https://github.com/JuliaLang/julia/blob/master/src/toplevel.c) gets invoked to load the file and [parse](@ref dev-parsing) it.
> 5. 実行するコードブロックがファイル内にある場合、[`jl_load（char * filename）`]（https://github.com/JuliaLang/julia/blob/master/src/toplevel.c）が呼び出されますファイルを読み込み、[parse]（@ ref dev-parsing）してください。
   Each fragment of code is then passed to `eval` to execute.
   > 次に、コードの各フラグメントは `eval`に渡されて実行されます。
6. Each fragment of code (or AST), is handed off to [`eval()`](@ref) to turn into results.
> 6. コード（またはAST）の各フラグメントは、結果に変換するために[`eval()`](@ref)に渡されます。
7. [`eval()`](@ref) takes each code fragment and tries to run it in [`jl_toplevel_eval_flex()`](https://github.com/JuliaLang/julia/blob/master/src/toplevel.c).
> 7. [`eval（）`]（@ ref）は各コードフラグメントを取り、[`jl_toplevel_eval_flex（）`]で実行しようとします（https://github.com/JuliaLang/julia/blob/master/src/toplevel .c）。
8. `jl_toplevel_eval_flex()` decides whether the code is a "toplevel" action (such as `using` or `module`), which would be invalid inside a function.
> 8. `jl_toplevel_eval_flex（）`はコードが `toplevel 'アクション（` using`や `module`など）かどうかを決定します。これは関数内では無効です。
   If so, it passes off the code to the toplevel interpreter.
   > そうであれば、コードから最上位インタプリタに渡されます。
9. `jl_toplevel_eval_flex()` then [expands](@ref dev-macro-expansion) the code to eliminate any macros and to "lower" the AST to make it simpler to execute.
> 9. `jl_toplevel_eval_flex（）`を実行すると、マクロが削除され、ASTが実行されるようにASTを「下げる」ためにコードが展開されます（@ ref dev-macro-expansion）。
10. `jl_toplevel_eval_flex()` then uses some simple heuristics to decide whether to JIT compiler the AST or to interpret it directly.
> 10.  `jl_toplevel_eval_flex()` は、単純な発見的手法を使用してASTをコンパイルするかどうかを決定したり、ASTを直接解釈するかどうかを決定します。
11. The bulk of the work to interpret code is handled by [`eval` in `interpreter.c`](https://github.com/JuliaLang/julia/blob/master/src/interpreter.c).
> 11. コードを解釈する作業の大半は、 `interpreter.c` の `eval`(https://github.com/JuliaLang/julia/blob/master/src/interpreter.c)によって処理されます。
12. If instead, the code is compiled, the bulk of the work is handled by `codegen.cpp`.
> 12. 代わりに、コードがコンパイルされていれば、大部分の作業は `codegen.cpp`によって処理されます。
    Whenever a Julia function is called for the first time with a given set of argument types, [type inference](@ref dev-type-inference) will be run on that function.
   > 与えられた引数型のセットで初めてJulia関数が呼び出されると、その関数に対して[型推論]（@ ref dev-type-in​​ference）が実行されます。
    This information is used by the [codegen](@ref dev-codegen) step to generate faster code.
   > この情報は、より速いコードを生成するために[codegen]（@ref dev-codegen）ステップで使用されます。
13. Eventually, the user quits the REPL, or the end of the program is reached, and the `_start()` method returns.
> 13. 最終的に、ユーザがREPLを終了するか、プログラムの終わりに達し、 `_start（）`メソッドが返ります。
14. Just before exiting, `main()` calls [`jl_atexit_hook(exit_code)`](https://github.com/JuliaLang/julia/blob/master/src/init.c).
> 14. 終了する直前に、 `main（）`は[`jl_atexit_hook（exit_code）`]（https://github.com/JuliaLang/julia/blob/master/src/init.c）を呼び出します。
    This calls `Base._atexit()` (which calls any functions registered to [`atexit()`](@ref) inside Julia).
    > Juliaの中で[atexit（） `]（@ ref）に登録されている関数を呼び出す` Base._atexit（） `を呼び出します。
    Then it calls [`jl_gc_run_all_finalizers()`](https://github.com/JuliaLang/julia/blob/master/src/gc.c).
    > 次に、[`jl_gc_run_all_finalizers（）`]（https://github.com/JuliaLang/julia/blob/master/src/gc.c）を呼び出します。
    Finally, it gracefully cleans up all `libuv` handles and waits for them to flush and close.
    > 最後に、すべての `libuv`ハンドルを正常にクリーンアップし、それらがフラッシュして閉じるのを待ちます。


## [Parsing](@id dev-parsing)

The Julia parser is a small lisp program written in femtolisp, the source-code for which is distributed inside Julia in [src/flisp](https://github.com/JuliaLang/julia/tree/master/src/flisp).
> Juliaパーサーは、symtolispで書かれた小さなlispプログラムで、ソースコードは[src/flisp]（https://github.com/JuliaLang/julia/tree/master/src/flisp）のJulia内に配布されています。

The interface functions for this are primarily defined in [`jlfrontend.scm`](https://github.com/JuliaLang/julia/blob/master/src/jlfrontend.scm).
> このためのインタフェース関数は、主に[`jlfrontend.scm`]（https://github.com/JuliaLang/julia/blob/master/src/jlfrontend.scm）で定義されています。
The code in [`ast.c`](https://github.com/JuliaLang/julia/blob/master/src/ast.c) handles this handoff on the Julia side.
> [`ast.c`]（https://github.com/JuliaLang/julia/blob/master/src/ast.c）のコードはJulia側でこのハンドオフを処理します。

The other relevant files at this stage are [`julia-parser.scm`](https://github.com/JuliaLang/julia/blob/master/src/julia-parser.scm), which handles tokenizing Julia code and turning it into an AST, and [`julia-syntax.scm`](https://github.com/JuliaLang/julia/blob/master/src/julia-syntax.scm), which handles transforming complex AST representations into simpler, "lowered" AST representations which are more suitable for analysis and execution.
> この段階で関連する他のファイルは、ジュリアコードをトークン化して回転させる[`julia-parser.scm`](https://github.com/JuliaLang/julia/blob/master/src/julia-parser.scm)です。 それをASTに変換し、[`julia-syntax.scm`](https://github.com/JuliaLang/julia/blob/master/src/julia-syntax.scm)を使用して複雑なAST表現を単純化し、 分析および実行に適したAST表現を「低下させた」。

## [Macro Expansion](@id dev-macro-expansion)

When [`eval()`](@ref) encounters a macro, it expands that AST node before attempting to evaluate the expression.
> [`eval（）`]（@ ref）がマクロに出会うと、式を評価しようとする前にそのASTノードを展開します。
Macro expansion involves a handoff from [`eval()`](@ref) (in Julia), to the parser function `jl_macroexpand()` (written in `flisp`) to the Julia macro itself (written in - what else - Julia) via `fl_invoke_julia_macro()`, and back.
> マクロ展開には、Juliaマクロ自身へのパーサー関数 `jl_macroexpand（）`（ `` flisp``で書かれています）への、 `` eval（） `（@ ref） ジュリア）から `fl_invoke_julia_macro（）`を経由して戻ることができます。

Typically, macro expansion is invoked as a first step during a call to [`Meta.lower()`](@ref)/`jl_expand()`, although it can also be invoked directly by a call to [`macroexpand()`](@ref)/`jl_macroexpand()`.
> 通常、[`Meta.lower()`](@ref)/`jl_expand()` の呼び出し中にマクロ展開が最初のステップとして呼び出されますが、[`macroexpand()`](@ref)/ `jl_macroexpand()`を実行します。

## [Type Inference](@id dev-type-inference)

Type inference is implemented in Julia by [`typeinf()` in `compiler/typeinf.jl`](https://github.com/JuliaLang/julia/blob/master/base/compiler/typeinf.jl).
> 型推論はJuliaで [`compiler/typeinf.jl`](`https://github.com/JuliaLang/julia/blob/master/base/compiler/typeinf.jl') の [`typeinf()`]によって実装されています。
Type inference is the process of examining a Julia function and determining bounds for the types of each of its variables, as well as bounds on the type of the return value from the function.
> 型推論は、Julia関数を調べて、各変数の型の境界と、関数からの戻り値の型の境界を決定するプロセスです。
This enables many future optimizations, such as unboxing of known immutable values, and compile-time hoisting of various run-time operations such as computing field offsets and function pointers.
> これにより、既知の不変値のアンボックス化や、フィールドオフセットや関数ポインタなどのさまざまな実行時操作のコンパイル時のホイスト処理など、多くの将来の最適化が可能になります。
Type inference may also include other steps such as constant propagation and inlining.
> 型推論には、定常伝播やインライン展開などの他のステップも含まれます。

!!! sidebar "More Definitions"
> !!!サイドバー "その他の定義"
   * JIT

     Just-In-Time Compilation The process of generating native-machine code into memory right when it is needed.
     > Just-In-Time Compilationネイティブマシンコードを必要に応じてメモリに生成するプロセス。
   * LLVM

     Low-Level Virtual Machine (a compiler) The Julia JIT compiler is a program/library called libLLVM.
     > 低レベル仮想マシン（コンパイラ）Julia JITコンパイラはlibLLVMというプログラム/ライブラリです。
     Codegen in Julia refers both to the process of taking a Julia AST and turning it into LLVM instructions, and the process of LLVM optimizing that and turning it into native assembly instructions.
     > JuliaのCodegenは、Julia ASTを使用してLLVM命令に変換するプロセスと、それを最適化してネイティブアセンブリ命令に変換するLLVMプロセスの両方を指します。
   * C++

     The programming language that LLVM is implemented in, which means that codegen is also implemented in this language.
     > LLVMが実装されているプログラミング言語。つまり、codegenもこの言語で実装されています。
     The rest of Julia's library is implemented in C, in part because its smaller feature set makes it more usable as a cross-language interface layer.
     > Juliaのライブラリの残りの部分はC言語で実装されています。これは、フィーチャセットのサイズが小さくなるほどクロスランゲージインターフェイスレイヤとして使いやすくなるからです。
   * box

     This term is used to describe the process of taking a value and allocating a wrapper around the data that is tracked by the garbage collector (gc) and is tagged with the object's type.
     > この用語は、値を取得し、ガベージコレクタ（gc）によって追跡され、オブジェクトの型でタグ付けされたデータの周りにラッパーを割り当てるプロセスを記述するために使用されます。
   * unbox

     The reverse of boxing a value.
     > ボクシングの逆の値。
     This operation enables more efficient manipulation of data when the type of that data is fully known at compile-time (through type inference).
     > この操作により、データの型がコンパイル時に（型の推論によって）完全にわかっている場合に、データのより効率的な操作が可能になります。
   * generic function

     A Julia function composed of multiple "methods" that are selected for dynamic dispatch based on the argument type-signature
     > 引数type-signatureに基づいて動的ディスパッチ用に選択された複数の「メソッド」で構成されるJulia関数
   * anonymous function or "method"

     A Julia function without a name and without type-dispatch capabilities
     > 名前なしでタイプディスパッチ機能を持たないJulia関数
   * primitive function

     A function implemented in C but exposed in Julia as a named function "method" (albeit without generic function dispatch capabilities, similar to a anonymous function)
     > Cで実装されているが、名前付き関数 "メソッド"としてJuliaに公開されている関数です（ただし、無名関数に似た汎用関数ディスパッチ機能はありません）
   * intrinsic function

     A low-level operation exposed as a function in Julia.
     > Juliaの関数として公開された低レベルの操作。
     These pseudo-functions implement operations on raw bits such as add and sign extend that cannot be expressed directly in any other way.
     > これらの擬似関数は、他の方法で直接表現することができない加算や符号拡張などの生のビットに対して演算を実装します。
     Since they operate on bits directly, they must be compiled into a function and surrounded by a call to `Core.Intrinsics.box(T, ...)` to reassign type information to the value.
     > 彼らはビットで直接操作するので、関数にコンパイルし、型情報を値に再割当てするために `Core.Intrinsics.box（T、...） 'の呼び出しで囲む必要があります。


## [JIT Code Generation](@id dev-codegen)

The JIT environment is initialized by an early call to [`jl_init_codegen` in `codegen.cpp`](https://github.com/JuliaLang/julia/blob/master/src/codegen.cpp).
> JIT環境は初期の `codegen.cpp` の `jl_init_codegen` (https://github.com/JuliaLang/julia/blob/master/src/codegen.cpp) の初期呼び出しによって初期化されます。

On demand, a Julia method is converted into a native function by the function `emit_function(jl_method_instance_t*)`.
> 要求に応じて、Juliaメソッドは関数emit_function(jl_method_instance_t*)によってネイティブ関数に変換されます。
(note, when using the MCJIT (in LLVM v3.4+), each function must be JIT into a new module.) 
> (MCJIT(LLVM v3.4 +で)を使用する場合、各機能は新しいモジュールにJITする必要があります)。
This function recursively calls `emit_expr()` until the entire function has been emitted.
> この関数は、関数全体が送出されるまで再帰的に `emit_expr()`を呼び出します。

Much of the remaining bulk of this file is devoted to various manual optimizations of specific code patterns.
> このファイルの残りの部分の多くは、特定のコードパターンのさまざまな手動最適化に費やされています。
For example, `emit_known_call()` knows how to inline many of the primitive functions (defined in [`builtins.c`](https://github.com/JuliaLang/julia/blob/master/src/builtins.c)) for various combinations of argument types.
> たとえば、 `emit_known_call()`はプリミティブ関数の多くをインライン化する方法を知っています([builtins.c`](https://github.com/JuliaLang/julia/blob/master/src/builtins.c)で定義されています)) を引数の型のさまざまな組み合わせに対して使用します。

Other parts of codegen are handled by various helper files:
> codegenの他の部分は、さまざまなヘルパーファイルによって処理されます。

  * [`debuginfo.cpp`](https://github.com/JuliaLang/julia/blob/master/src/debuginfo.cpp)

    Handles backtraces for JIT functions
    > JIT関数のバックトレースを処理する
  * [`ccall.cpp`](https://github.com/JuliaLang/julia/blob/master/src/ccall.cpp)

    Handles the ccall and llvmcall FFI, along with various `abi_*.cpp` files
    > さまざまな `abi _ *。cpp`ファイルとともに、ccallおよびllvmcall FFIを処理します。
  * [`intrinsics.cpp`](https://github.com/JuliaLang/julia/blob/master/src/intrinsics.cpp)

    Handles the emission of various low-level intrinsic functions
    > さまざまな低レベルの組み込み関数の放出を処理します。

!!! sidebar "Bootstrapping"
> !!!サイドバー "ブートストラップ"
   The process of creating a new system image is called "bootstrapping".
   > 新しいシステムイメージを作成するプロセスは、「ブートストラップ」と呼ばれます。

   The etymology of this word comes from the phrase "pulling oneself up by the bootstraps", and refers to the idea of starting from a very limited set of available functions and definitions and ending with the creation of a full-featured environment.
   > この言葉の語源は、「ブートストラップによって自分自身を引き上げる」という言葉から来ており、利用可能な機能と定義のごく限られたセットから始まり、フル機能の環境の作成で終わるという考え方を指しています。

## [System Image](@id dev-sysimg)

The system image is a precompiled archive of a set of Julia files.
> システムイメージは、ジュリアファイルのセットのあらかじめコンパイルされたアーカイブです。
The `sys.ji` file distributed with Julia is one such system image, generated by executing the file [`sysimg.jl`](https://github.com/JuliaLang/julia/blob/master/base/sysimg.jl), and serializing the resulting environment (including Types, Functions, Modules, and all other defined values) into a file.
> Juliaと一緒に配布される `sys.ji`ファイルは、` `sysimg.jl`（https://github.com/JuliaLang/julia/blob/master/base/sysimg.jl）ファイルを実行することによって生成されたシステムイメージの1つです。 ）、結果の環境（型、関数、モジュール、およびその他の定義された値を含む）をファイルにシリアライズします。
Therefore, it contains a frozen version of the `Main`, `Core`, and `Base` modules (and whatever else was in the environment at the end of bootstrapping).
> したがって、 `Main`、` Core`、 `Base`モジュールの凍結版（およびブートストラップの終わりに環境にあったもの）が含まれています。
This serializer/deserializer is implemented by [`jl_save_system_image`/`jl_restore_system_image` in `staticdata.c`](https://github.com/JuliaLang/julia/blob/master/src/staticdata.c).
> このシリアライザ/デシリアライザは、 `staticdata.c`の` `jl_save_system_image` /` jl_restore_system_image`（https://github.com/JuliaLang/julia/blob/master/src/staticdata.c）で実装されています。

If there is no sysimg file (`jl_options.image_file == NULL`), this also implies that `--build` was given on the command line, so the final result should be a new sysimg file.
> sysimgファイルがない場合（ `jl_options.image_file == NULL`）、これはコマンドラインで` --build`が指定されたことを意味するので、最終的な結果は新しいsysimgファイルでなければなりません。
During Julia initialization, minimal `Core` and `Main` modules are created.
> Juliaの初期化中に、最小の `Core`モジュールと` Main`モジュールが作成されます。
Then a file named `boot.jl` is evaluated from the current directory.
> その後、 `boot.jl`という名前のファイルが現在のディレクトリから評価されます。
Julia then evaluates any file given as a command line argument until it reaches the end.
> Juliaは、最後に達するまでコマンドライン引数として与えられたファイルを評価します。
Finally, it saves the resulting environment to a "sysimg" file for use as a starting point for a future Julia run.
> 最後に、結果の環境を "sysimg"ファイルに保存して、将来のJulia実行の出発点として使用します。
