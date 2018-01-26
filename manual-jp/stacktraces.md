<!-- Start -->

# Stack Traces
> # スタックトレース

<!-- End -->
<!-- Start -->
The `StackTraces` module provides simple stack traces that are both human readable and easy to use programmatically.
> `StackTraces`モジュールは、人が読みやすく、プログラム的に使いやすい単純なスタックトレースを提供します。
<!-- End -->
<!-- Start -->

## Viewing a stack trace

> ## スタックトレースを見る

<!-- End -->
<!-- Start -->
The primary function used to obtain a stack trace is [`stacktrace()`](@ref):
> スタックトレースを取得するために使用される主要な関数は[`stacktrace（）`]（@ ref）です：
<!-- End -->

```julia-repl
julia> stacktrace()
4-element Array{StackFrame,1}:
 eval(::Module, ::Any) at boot.jl:236
 eval_user_input(::Any, ::Base.REPL.REPLBackend) at REPL.jl:66
 macro expansion at REPL.jl:97 [inlined]
 (::Base.REPL.##1#2{Base.REPL.REPLBackend})() at event.jl:73
```

<!-- Start -->
Calling [`stacktrace()`](@ref) returns a vector of [`StackFrame`](@ref) s. 
> [`stacktrace（）`]（@ref）を呼び出すと、[`StackFrame`]（@ ref）のベクトルが返されます。
<!-- End -->
<!-- Start -->
For ease of use, the alias [`StackTrace`](@ref) can be used in place of `Vector{StackFrame}`. 
> 使い易さのため、別名[`StackTrace`]（@ ref）は` Vector {StackFrame} 'の代わりに使うことができます。
<!-- End -->
<!-- Start -->
(Examples with `[...]` indicate that output may vary depending on how the code is run.)
> （ `[...]`の例は、コードの実行方法によって出力が異なることを示しています）。
<!-- End -->

```julia-repl
julia> example() = stacktrace()
example (generic function with 1 method)

julia> example()
5-element Array{StackFrame,1}:
 example() at REPL[1]:1
 eval(::Module, ::Any) at boot.jl:236
[...]

julia> @noinline child() = stacktrace()
child (generic function with 1 method)

julia> @noinline parent() = child()
parent (generic function with 1 method)

julia> grandparent() = parent()
grandparent (generic function with 1 method)

julia> grandparent()
7-element Array{StackFrame,1}:
 child() at REPL[3]:1
 parent() at REPL[4]:1
 grandparent() at REPL[5]:1
[...]
```

<!-- Start -->
Note that when calling [`stacktrace()`](@ref) you'll typically see a frame with `eval(...) at boot.jl`. 
> [stacktrace（） `]（@ ref）を呼び出すと、通常、` eval（...）at boot.jl`というフレームが表示されることに注意してください。
<!-- End -->
<!-- Start -->
When calling [`stacktrace()`](@ref) from the REPL you'll also have a few extra frames in the stack from `REPL.jl`, usually looking something like this:
> REPLから[`stacktrace（）`]（@ref）を呼び出すときには、スタック内に `REPL.jl`からいくつか余分なフレームがあります。通常は次のようになります：
<!-- End -->

```julia-repl
julia> example() = stacktrace()
example (generic function with 1 method)

julia> example()
5-element Array{StackFrame,1}:
 example() at REPL[1]:1
 eval(::Module, ::Any) at boot.jl:236
 eval_user_input(::Any, ::Base.REPL.REPLBackend) at REPL.jl:66
 macro expansion at REPL.jl:97 [inlined]
 (::Base.REPL.##1#2{Base.REPL.REPLBackend})() at event.jl:73
```
<!-- Start -->

### Extracting useful information

> ### 有用な情報の抽出

<!-- End -->
<!-- Start -->
Each [`StackFrame`](@ref) contains the function name, file name, line number, lambda info, a flag indicating whether the frame has been inlined, a flag indicating whether it is a C function (by default C functions do not appear in the stack trace), and an integer representation of the pointer returned by [`backtrace()`](@ref):
> 各[`StackFrame`]（@ref）には、関数名、ファイル名、行番号、ラムダ情報、フレームがインライン化されているかどうかを示すフラグ、それがC関数かどうかを示すフラグ（デフォルトではC関数は スタックトレースに現れる）と、[`backtrace（）`]（@ ref）によって返されるポインタの整数表現です：
<!-- End -->

```julia-repl
julia> top_frame = stacktrace()[1]
eval(::Module, ::Any) at boot.jl:236

julia> top_frame.func
:eval

julia> top_frame.file
Symbol("./boot.jl")

julia> top_frame.line
236

julia> top_frame.linfo
Nullable{Core.MethodInstance}(MethodInstance for eval(::Module, ::Any))

julia> top_frame.inlined
false

julia> top_frame.from_c
false
```

```julia-repl
julia> top_frame.pointer
0x00007f390d152a59
```

<!-- Start -->
This makes stack trace information available programmatically for logging, error handling, and more.
> これにより、ロギング、エラー処理などのためにスタックトレース情報をプログラムで使用できるようになります。
<!-- End -->

### Error handling
###エラー処理

<!-- Start -->
While having easy access to information about the current state of the callstack can be helpful in many places, the most obvious application is in error handling and debugging.
> コールスタックの現在の状態に関する情報に簡単にアクセスできることは、多くの場面で役立ちますが、最も明白なアプリケーションはエラー処理とデバッグです。
<!-- End -->

```julia-repl
julia> @noinline bad_function() = undeclared_variable
bad_function (generic function with 1 method)

julia> @noinline example() = try
           bad_function()
       catch
           stacktrace()
       end
example (generic function with 1 method)

julia> example()
5-element Array{StackFrame,1}:
 example() at REPL[2]:4
 eval(::Module, ::Any) at boot.jl:236
[...]
```

<!-- Start -->
You may notice that in the example above the first stack frame points points at line 4, where [`stacktrace()`](@ref) is called, rather than line 2, where *bad_function* is called, and `bad_function`'s frame is missing entirely. 
>上記の例では、最初のスタックフレームが、bad_function *が呼び出される2行目ではなく[`stacktrace（）`]（@ ref）が呼び出される4行目の点を指していることに気付くかもしれません。 sのフレームが完全に欠落しています。
<!-- End -->
<!-- Start -->
This is understandable, given that [`stacktrace()`](@ref) is called from the context of the *catch*. 
> これは、[stacktrace（） `]（@ ref）が* catch *のコンテキストから呼び出されたことを考えると分かります。
<!-- End -->
<!-- Start -->
While in this example it's fairly easy to find the actual source of the error, in complex cases tracking down the source of the error becomes nontrivial.
> この例では、エラーの実際の原因を見つけるのはかなり簡単ですが、複雑なケースでは、エラーの原因を突き止めることは重要ではありません。
<!-- End -->

<!-- Start -->
This can be remedied by calling [`catch_stacktrace()`](@ref) instead of [`stacktrace()`](@ref).
> これは[stacktrace（） `]（@ ref）の代わりに[` catch_stacktrace（） `]（@ ref）を呼び出すことで解決できます。
<!-- End -->
<!-- Start -->
Instead of returning callstack information for the current context, [`catch_stacktrace()`](@ref) returns stack information for the context of the most recent exception:
> 現在のコンテキストの呼び出しスタック情報を返す代わりに、[`catch_stacktrace（）`]（@ ref）は最新の例外のコンテキストのスタック情報を返します：
<!-- End -->

```julia-repl
julia> @noinline bad_function() = undeclared_variable
bad_function (generic function with 1 method)

julia> @noinline example() = try
           bad_function()
       catch
           catch_stacktrace()
       end
example (generic function with 1 method)

julia> example()
6-element Array{StackFrame,1}:
 bad_function() at REPL[1]:1
 example() at REPL[2]:2
[...]
```

<!-- Start -->
Notice that the stack trace now indicates the appropriate line number and the missing frame.
> スタックトレースは、適切な行番号と欠落しているフレームを示していることに注意してください。
<!-- End -->

```julia-repl
julia> @noinline child() = error("Whoops!")
child (generic function with 1 method)

julia> @noinline parent() = child()
parent (generic function with 1 method)

julia> @noinline function grandparent()
           try
               parent()
           catch err
               println("ERROR: ", err.msg)
               catch_stacktrace()
           end
       end
grandparent (generic function with 1 method)

julia> grandparent()
ERROR: Whoops!
7-element Array{StackFrame,1}:
 child() at REPL[1]:1
 parent() at REPL[2]:1
 grandparent() at REPL[3]:3
[...]
```

<!-- Start -->

## Comparison with [`backtrace()`](@ref)

> ## [`backtrace（）`]との比較（@ ref）

<!-- End -->
<!-- Start -->
A call to [`backtrace()`](@ref) returns a vector of `Ptr{Void}`, which may then be passed into [`stacktrace()`](@ref) for translation:
[`backtrace（）`]（@ref）の呼び出しは `Ptr {Void}`のベクトルを返します。このベクトルは翻訳のために[`stacktrace（）`]（@ ref）に渡されます：
<!-- End -->

```julia-repl
julia> trace = backtrace()
21-element Array{Ptr{Void},1}:
 Ptr{Void} @0x00007f10049d5b2f
 Ptr{Void} @0x00007f0ffeb4d29c
 Ptr{Void} @0x00007f0ffeb4d2a9
 Ptr{Void} @0x00007f1004993fe7
 Ptr{Void} @0x00007f10049a92be
 Ptr{Void} @0x00007f10049a823a
 Ptr{Void} @0x00007f10049a9fb0
 Ptr{Void} @0x00007f10049aa718
 Ptr{Void} @0x00007f10049c0d5e
 Ptr{Void} @0x00007f10049a3286
 Ptr{Void} @0x00007f0ffe9ba3ba
 Ptr{Void} @0x00007f0ffe9ba3d0
 Ptr{Void} @0x00007f1004993fe7
 Ptr{Void} @0x00007f0ded34583d
 Ptr{Void} @0x00007f0ded345a87
 Ptr{Void} @0x00007f1004993fe7
 Ptr{Void} @0x00007f0ded34308f
 Ptr{Void} @0x00007f0ded343320
 Ptr{Void} @0x00007f1004993fe7
 Ptr{Void} @0x00007f10049aeb67
 Ptr{Void} @0x0000000000000000

julia> stacktrace(trace)
5-element Array{StackFrame,1}:
 backtrace() at error.jl:46
 eval(::Module, ::Any) at boot.jl:236
 eval_user_input(::Any, ::Base.REPL.REPLBackend) at REPL.jl:66
 macro expansion at REPL.jl:97 [inlined]
 (::Base.REPL.##1#2{Base.REPL.REPLBackend})() at event.jl:73
```

<!-- Start -->
Notice that the vector returned by [`backtrace()`](@ref) had 21 pointers, while the vector returned by [`stacktrace()`](@ref) only has 5. 
> [`backtrace（）`]（@ ref）によって返されるベクトルは21個のポインタを持ち、[`stacktrace（）`]（@ ref）によって返されるベクトルは5個しか持たないことに注意してください。
<!-- End -->
<!-- Start -->
This is because, by default, [`stacktrace()`](@ref) removes any lower-level C functions from the stack. 
> これは、デフォルトで[stacktrace（） `]（@ ref）がスタックから下位レベルのC関数を削除するためです。
<!-- End -->
<!-- Start -->
If you want to include stack frames from C calls, you can do it like this:
> Cコールのスタックフレームを含める場合は、次のようにします。
<!-- End -->

```julia-repl
julia> stacktrace(trace, true)
27-element Array{StackFrame,1}:
 jl_backtrace_from_here at stackwalk.c:103
 backtrace() at error.jl:46
 backtrace() at sys.so:?
 jl_call_method_internal at julia_internal.h:248 [inlined]
 jl_apply_generic at gf.c:2215
 do_call at interpreter.c:75
 eval at interpreter.c:215
 eval_body at interpreter.c:519
 jl_interpret_toplevel_thunk at interpreter.c:664
 jl_toplevel_eval_flex at toplevel.c:592
 jl_toplevel_eval_in at builtins.c:614
 eval(::Module, ::Any) at boot.jl:236
 eval(::Module, ::Any) at sys.so:?
 jl_call_method_internal at julia_internal.h:248 [inlined]
 jl_apply_generic at gf.c:2215
 eval_user_input(::Any, ::Base.REPL.REPLBackend) at REPL.jl:66
 ip:0x7f1c707f1846
 jl_call_method_internal at julia_internal.h:248 [inlined]
 jl_apply_generic at gf.c:2215
 macro expansion at REPL.jl:97 [inlined]
 (::Base.REPL.##1#2{Base.REPL.REPLBackend})() at event.jl:73
 ip:0x7f1c707ea1ef
 jl_call_method_internal at julia_internal.h:248 [inlined]
 jl_apply_generic at gf.c:2215
 jl_apply at julia.h:1411 [inlined]
 start_task at task.c:261
 ip:0xffffffffffffffff
```

<!-- Start -->
Individual pointers returned by [`backtrace()`](@ref) can be translated into [`StackFrame`](@ref) s by passing them into [`StackTraces.lookup()`](@ref):
> [`backtrace（）`]（@ ref）によって返された個々のポインタは、 `` StackTraces.lookup（） `]（@ ref）に渡すことで[` StackFrame`]
<!-- End -->

```julia-repl
julia> pointer = backtrace()[1];

julia> frame = StackTraces.lookup(pointer)
1-element Array{StackFrame,1}:
 jl_backtrace_from_here at stackwalk.c:103

julia> println("The top frame is from $(frame[1].func)!")
The top frame is from jl_backtrace_from_here!
```
