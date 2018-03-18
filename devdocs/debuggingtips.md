# gdb debugging tips

## Displaying Julia variables

<!-- EN -->
Within `gdb`, any `jl_value_t*` object `obj` can be displayed using
> `gdb`の中で、` jl_value_t * `オブジェクト` obj`は、

```
(gdb) call jl_(obj)
```

<!-- EN -->
The object will be displayed in the `julia` session, not in the gdb session.
> オブジェクトはgdbセッションではなく `julia`セッションに表示されます。
<!-- EN -->
This is a useful way to discover the types and values of objects being manipulated by Julia's C code.
> これは、JuliaのCコードによって操作されるオブジェクトの型と値を発見するのに便利な方法です。

<!-- EN -->
Similarly, if you're debugging some of Julia's internals (e.g., `compiler.jl`), you can print `obj` using
> 同様に、Juliaの内部構造（例えば、 `compiler.jl`）の一部をデバッグする場合、` obj`を使って `

```julia
ccall(:jl_, Cvoid, (Any,), obj)
```

<!-- EN -->
This is a good way to circumvent problems that arise from the order in which julia's output streams are initialized.
> これは、juliaの出力ストリームが初期化される順序から生じる問題を回避する良い方法です。

<!-- EN -->
Julia's flisp interpreter uses `value_t` objects; these can be displayed with `call fl_print(fl_ctx, ios_stdout, obj)`.
> Juliaのflispインタプリタは `value_t`オブジェクトを使います。 これらは `call fl_print（fl_ctx、ios_stdout、obj）`で表示することができます。

## Useful Julia variables for Inspecting

<!-- EN -->
While the addresses of many variables, like singletons, can be be useful to print for many failures, there are a number of additional variables (see `julia.h` for a complete list) that are even more useful.
> シングルトンのような多くの変数のアドレスは多くの失敗のために印刷するのに便利ですが、さらに便利ないくつかの変数（完全なリストについては `julia.h`を参照）があります。

  <!-- EN -->
  * (when in `jl_apply_generic`) `mfunc` and `jl_uncompress_ast(mfunc->def, mfunc->code)` :: for figuring out a bit about the call-stack
  * > ( `jl_apply_generic`のとき) `mfunc` と `jl_uncompress_ast(mfunc->def, mfunc->code)` :: コールスタックについて少し分かります
  <!-- EN -->
  * `jl_lineno` and `jl_filename` :: for figuring out what line in a test to go start debugging from (or figure out how far into a file has been parsed)
  * > `jl_lineno` と `jl_filename`:: テストのどの行からデバッグを開始するか（またはファイルがどのくらいまで解析されたかを調べます）
  <!-- EN -->
  * `$1` :: not really a variable, but still a useful shorthand for referring to the result of the last gdb command (such as `print`)
  * > `$1` :: 実際には変数ではありませんが、最後のgdbコマンドの結果を参照するのに便利な省略形です (`print` など)
  <!-- EN -->
  * `jl_options` :: sometimes useful, since it lists all of the command line options that were successfully parsed
  * > `jl_options` :: 成功した場合、解析に成功したすべてのコマンドラインオプション
  <!-- EN -->
  * `jl_uv_stderr` :: because who doesn't like to be able to interact with stdio
  * > `jl_uv_stderr` ::標準入出力と対話するのが好ましくありません。

## Useful Julia functions for Inspecting those variables

  <!-- EN -->
  * `jl_gdblookup($rip)` :: For looking up the current function and line. (use `$eip` on i686 platforms)
  * > `jl_gdblookup（$ rip）` ::現在の関数と行を検索します。 （i686プラットフォームでは `$ eip`を使います）
  <!-- EN -->
  * `jlbacktrace()` :: For dumping the current Julia backtrace stack to stderr. Only usable after `record_backtrace()` has been called.
  * > `jlbacktrace（）` ::現在のJulia backtraceスタックをstderrにダンプします。 `record_backtrace（）`の後にのみ使用可能です。
  <!-- EN -->
  * `jl_dump_llvm_value(Value*)` :: For invoking `Value->dump()` in gdb, where it doesn't work natively.
  * > `jl_dump_llvm_value（Value *）` :: gdbで `Value-> dump（）`を呼び出すために、ネイティブでは動作しません。
    <!-- EN -->
    For example, `f->linfo->functionObject`, `f->linfo->specFunctionObject`, and `to_function(f->linfo)`.
    > たとえば、 `f->linfo->functionObject`, `f->linfo->specFunctionObject`, および `to_function(f->linfo)` のようになります。
  <!-- EN -->
  * `Type->dump()` :: only works in lldb. Note: add something like `;1` to prevent lldb from printing its prompt over the output
  * > `Type->dump()` :: lldb でのみ動作します。 注：lldb がプロンプトを出力しないように `;1` のようなものを追加してください
  <!-- EN -->
  * `jl_eval_string("expr")` :: for invoking side-effects to modify the current state or to lookup symbols
  * > `jl_eval_string("expr")` :: 副作用を呼び出して現在の状態を変更したり、シンボルを検索したりするためのもの
  <!-- EN -->
  * `jl_typeof(jl_value_t*)` :: for extracting the type tag of a Julia value (in gdb, call `macro define jl_typeof jl_typeof` first, or pick something short like `ty` for the first arg to define a shorthand)
  * > `jl_typeof(jl_value_t*)` :: Julia値の型タグを抽出するため（ gdb では、 `macro define jl_typeof jl_typeof` を最初に呼び出すか、 最初の引数が省略形を定義するために `ty` のような短いものを選んでください）

## Inserting breakpoints for inspection from gdb

In your `gdb` session, set a breakpoint in `jl_breakpoint` like so:
> `gdb`セッションで、` jl_breakpoint`にブレークポイントを設定します：

```
(gdb) break jl_breakpoint
```

Then within your Julia code, insert a call to `jl_breakpoint` by adding
> あなたのJuliaコード内で、 `jl_breakpoint`への呼び出しを

```julia
ccall(:jl_breakpoint, Cvoid, (Any,), obj)
```

where `obj` can be any variable or tuple you want to be accessible in the breakpoint.
> ここで `obj`はブレークポイントでアクセス可能な変数またはタプルです。

It's particularly helpful to back up to the `jl_apply` frame, from which you can display the arguments to a function using, e.g.,
> `jl_apply`フレームにバックアップすることは特に有用です。そこから関数への引数を表示することができます。例えば、

```
(gdb) call jl_(args[0])
```

Another useful frame is `to_function(jl_method_instance_t *li, bool cstyle)`.
> もう一つの有用なフレームは `to_function(jl_method_instance_t *li, bool cstyle)`です。
The `jl_method_instance_t*` argument is a struct with a reference to the final AST sent into the compiler.
> `jl_method_instance_t*` 引数は、コンパイラに送られる最終的なASTへの参照を持つ構造体です。
However, the AST at this point will usually be compressed; to view the AST, call `jl_uncompress_ast` and then pass the result to `jl_`:
> ただし、この時点でのASTは通常圧縮されます。 ASTを表示するには `jl_uncompress_ast`を呼び出し、その結果を` jl_`に渡します：

```
#2  0x00007ffff7928bf7 in to_function (li=0x2812060, cstyle=false) at codegen.cpp:584
584          abort();
(gdb) p jl_(jl_uncompress_ast(li, li->ast))
```

## Inserting breakpoints upon certain conditions

### Loading a particular file

Let's say the file is `sysimg.jl`:
> ファイルが `sysimg.jl` であるとします:

```
(gdb) break jl_load if strcmp(fname, "sysimg.jl")==0
```

### Calling a particular method

```
(gdb) break jl_apply_generic if strcmp((char*)(jl_symbol_name)(jl_gf_mtable(F)->name), "method_to_break")==0
```

Since this function is used for every call, you will make everything 1000x slower if you do this.
> この関数はすべての呼び出しで使用されるため、これを行うとすべてが1000倍遅くなります。

## Dealing with signals

Julia requires a few signal to function property.
> ジュリアは、信号を機能させるためにいくつかの信号を必要とします。
The profiler uses `SIGUSR2` for sampling and the garbage collector uses `SIGSEGV` for threads synchronization.
> プロファイラはサンプリングに `SIGUSR2`を使い、ガーベジコレクタはスレッドの同期に` SIGSEGV`を使います。
If you are debugging some code that uses the profiler or multiple threads, you may want to let the debugger ignore these signals since they can be triggered very often during normal operations.
> プロファイラまたは複数のスレッドを使用するコードをデバッグする場合は、通常の操作中に非常に頻繁にトリガされる可能性があるため、デバッガにこれらのシグナルを無視させることができます。
The command to do this in GDB is (replace `SIGSEGV` with `SIGUSRS` or other signals you want to ignore):
> GDBでこれを行うコマンドは（ `SIGSEGV`を` SIGUSRS`や無視したい他のシグナルに置き換えてください）：

```
(gdb) handle SIGSEGV noprint nostop pass
```

The corresponding LLDB command is (after the process is started):
> 対応するLLDBコマンドは、（プロセスの開始後）次のとおりです。

```
(lldb) pro hand -p true -s false -n false SIGSEGV
```

If you are debugging a segfault with threaded code, you can set a breakpoint on `jl_critical_error` (`sigdie_handler` should also work on Linux and BSD) in order to only catch the actual segfault rather than the GC synchronization points.
> スレッドコードを使ってsegfaultをデバッグする場合は、 `jl_critical_error`（` sigdie_handler`はLinuxとBSDでも動作するはずです）にブレークポイントを設定することで、GC同期ポイントではなく実際のsegfaultを捕捉することができます。

## Debugging during Julia's build process (bootstrap)

Errors that occur during `make` need special handling. Julia is built in two stages, constructing `sys0` and `sys.ji`.
> `make`の間に発生するエラーには特別な処理が必要です。 Juliaは `sys0` と `sys.ji` を構築する2つの段階で構築されています。
To see what commands are running at the time of failure, use `make VERBOSE=1`.
> どのコマンドが実行時に失敗しているかを見るには、 `make VERBOSE = 1`を使います。

At the time of this writing, you can debug build errors during the `sys0` phase from the `base` directory using:
> この記事を書いている時点で、 `base` ディレクトリから` sys0`フェーズでビルドエラーをデバッグできます：

```
julia/base$ gdb --args ../usr/bin/julia-debug -C native --build ../usr/lib/julia/sys0 sysimg.jl
```

You might need to delete all the files in `usr/lib/julia/` to get this to work.
> これを動作させるには、 `usr/lib/julia/` にあるすべてのファイルを削除する必要があります。

You can debug the `sys.ji` phase using:
> `sys.ji`フェーズをデバッグすることができます：

```
julia/base$ gdb --args ../usr/bin/julia-debug -C native --build ../usr/lib/julia/sys -J ../usr/lib/julia/sys0.ji sysimg.jl
```

By default, any errors will cause Julia to exit, even under gdb.
> デフォルトでは、gdbの下でもエラーがあればJuliaは終了します。
To catch an error "in the act", set a breakpoint in `jl_error` (there are several other useful spots, for specific kinds of failures, including: `jl_too_few_args`, `jl_too_many_args`, and `jl_throw`).
> "行為の中で"エラーをキャッチするには、 `jl_error`にブレークポイントを設定します（` jl_too_few_args`、 `jl_too_many_args`、` jl_throw`など）。

Once an error is caught, a useful technique is to walk up the stack and examine the function by inspecting the related call to `jl_apply`.
> エラーが捕捉されると、便利なテクニックはスタックを歩いて `jl_apply` への関連する呼び出しを調べることによって関数を調べることです。
To take a real-world example:
> 実際の例を取るには：

```
Breakpoint 1, jl_throw (e=0x7ffdf42de400) at task.c:802
802 {
(gdb) p jl_(e)
ErrorException("auto_unbox: unable to determine argument type")
$2 = void
(gdb) bt 10
#0  jl_throw (e=0x7ffdf42de400) at task.c:802
#1  0x00007ffff65412fe in jl_error (str=0x7ffde56be000 <_j_str267> "auto_unbox:
   unable to determine argument type")
   at builtins.c:39
#2  0x00007ffde56bd01a in julia_convert_16886 ()
#3  0x00007ffff6541154 in jl_apply (f=0x7ffdf367f630, args=0x7fffffffc2b0, nargs=2) at julia.h:1281
...
```

The most recent `jl_apply` is at frame #3, so we can go back there and look at the AST for the function `julia_convert_16886`.
> 最新の `jl_apply`はフレーム＃3にあるので、そこに戻って関数` julia_convert_16886`のASTを見ることができます。
This is the uniqued name for some method of `convert`.
> これは、 `convert`メソッドの一意の名前です。
`f` in this frame is a `jl_function_t*`, so we can look at the type signature, if any, from the `specTypes` field:
> このフレームの `f`は` jl_function_t * `なので、` specTypes`フィールドからタイプシグネチャがあればそれを見ることができます：

```
(gdb) f 3
#3  0x00007ffff6541154 in jl_apply (f=0x7ffdf367f630, args=0x7fffffffc2b0, nargs=2) at julia.h:1281
1281            return f->fptr((jl_value_t*)f, args, nargs);
(gdb) p f->linfo->specTypes
$4 = (jl_tupletype_t *) 0x7ffdf39b1030
(gdb) p jl_( f->linfo->specTypes )
Tuple{Type{Float32}, Float64}           # <-- type signature for julia_convert_16886
```

Then, we can look at the AST for this function:
> 次に、この関数のASTを見ることができます：

```
(gdb) p jl_( jl_uncompress_ast(f->linfo, f->linfo->ast) )
Expr(:lambda, Array{Any, 1}[:#s29, :x], Array{Any, 1}[Array{Any, 1}[], Array{Any, 1}[Array{Any, 1}[:#s29, :Any, 0], Array{Any, 1}[:x, :Any, 0]], Array{Any, 1}[], 0], Expr(:body,
Expr(:line, 90, :float.jl)::Any,
Expr(:return, Expr(:call, :box, :Float32, Expr(:call, :fptrunc, :Float32, :x)::Any)::Any)::Any)::Any)::Any
```

Finally, and perhaps most usefully, we can force the function to be recompiled in order to step through the codegen process.
> 最後に、おそらく最も有益なことに、codegenプロセスをステップ実行するために関数を強制的に再コンパイルすることができます。
To do this, clear the cached `functionObject` from the `jl_lamdbda_info_t*`:
> これを行うには、 `jl_lamdbda_info_t *`からキャッシュされた `functionObject`をクリアします：

```
(gdb) p f->linfo->functionObject
$8 = (void *) 0x1289d070
(gdb) set f->linfo->functionObject = NULL
```

Then, set a breakpoint somewhere useful (e.g. `emit_function`, `emit_expr`, `emit_call`, etc.), and run codegen:
> 次に、ブレークポイントを有用な場所（例えば、 `emit_function`、` emit_expr`、 `emit_call`など）に設定し、codegenを実行します：

```
(gdb) p jl_compile(f)
... # your breakpoint here
```

## Debugging precompilation errors

Module precompilation spawns a separate Julia process to precompile each module.
> モジュールのプリコンパイルは、各モジュールをプリコンパイルするために、別々のJuliaプロセスを生成します。
Setting a breakpoint or catching failures in a precompile worker requires attaching a debugger to the worker.
> ブレークポイントを設定するか、またはプリコンパイルされたワーカーでエラーをキャッチするには、ワーカーにデバッガを接続する必要があります。
The easiest approach is to set the debugger watch for new process launches matching a given name.
> 最も簡単な方法は、指定された名前に一致する新しいプロセスの起動のためにデバッガの監視を設定することです。
For example:
> 例えば：

```
(gdb) attach -w -n julia-debug
```

or:
> または

```
(lldb) process attach -w -n julia-debug
```

Then run a script/command to start precompilation.
> その後、スクリプト/コマンドを実行して、プリコンパイルを開始します。
As described earlier, use conditional breakpoints in the parent process to catch specific file-loading events and narrow the debugging window.
> 先に説明したように、親プロセスで条件付きブレークポイントを使用して特定のファイルロードイベントを捕捉し、デバッグウィンドウを絞り込みます。
(some operating systems may require alternative approaches, such as following each `fork` from the parent process)
> （一部のオペレーティングシステムでは、親プロセスからのそれぞれの `fork 'の後に続くような別のアプローチが必要な場合があります）

## Mozilla's Record and Replay Framework (rr)

Julia now works out of the box with [rr](http://rr-project.org/), the lightweight recording and deterministic debugging framework from Mozilla.
> Juliaは、Mozillaの軽量レコーディングと確定的なデバッグフレームワークである[rr]（http://rr-project.org/）ですぐに使えます。
This allows you to replay the trace of an execution deterministically.
> これにより、実行のトレースを確定的に再生することができます。
The replayed execution's address spaces, register contents, syscall data etc are exactly the same in every run.
> 再生された実行のアドレス空間、レジスタの内容、システムコールのデータなどはすべての実行でまったく同じです。

A recent version of rr (3.1.0 or higher) is required.
> 最近のバージョンのrr（3.1.0以上）が必要です。