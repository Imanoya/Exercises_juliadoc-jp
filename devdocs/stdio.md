# printf() and stdio in the Julia runtime

## Libuv wrappers for stdio

<!-- EN -->
`julia.h` defines [libuv](http://docs.libuv.org) wrappers for the `stdio.h` streams:
> `julia.h`は` stdio.h`ストリーム用の[libuv]（http://docs.libuv.org）ラッパーを定義しています：

```c
uv_stream_t *JL_STDIN;
uv_stream_t *JL_STDOUT;
uv_stream_t *JL_STDERR;
```

<!-- EN -->
... and corresponding output functions:
> ...と対応する出力関数：

```c
int jl_printf(uv_stream_t *s, const char *format, ...);
int jl_vprintf(uv_stream_t *s, const char *format, va_list args);
```

<!-- EN -->
These `printf` functions are used by the `.c` files in the `src/` and `ui/` directories wherever stdio is needed to ensure that output buffering is handled in a unified way.
> これらの `printf`関数は、`src/`と `ui/` ディレクトリの `.c` ファイルによって使用され、出力バッファリングが統一された方法で確実に処理されるようにするために必要です。

<!-- EN -->
In special cases, like signal handlers, where the full libuv infrastructure is too heavy, `jl_safe_printf()` can be used to [`write(2)`](@ref) directly to `STDERR_FILENO`:
> 特別な場合、完全なlibuvインフラストラクチャが非常に重いシグナルハンドラのように、 `jl_safe_printf()`は `STDERR_FILENO` に直接 [` write(2)`](@ref)することができます：

```c
void jl_safe_printf(const char *str, ...);
```

## Interface between JL_STD* and Julia code

<!-- EN -->
[`Base.STDIN`](@ref), [`Base.STDOUT`](@ref) and [`Base.STDERR`](@ref) are bound to the `JL_STD*` libuv streams defined in the runtime.
> [`Base.STDIN`](@ref)、 [`Base.STDOUT`](@ref) および [`Base.STDERR`](@ref) は、ランタイムで定義された `JL_STD*` libuvストリームにバインドされています。

<!-- EN -->
Julia's `__init__()` function (in `base/sysimg.jl`) calls `reinit_stdio()` (in `base/stream.jl`) to create Julia objects for [`Base.STDIN`](@ref), [`Base.STDOUT`](@ref) and [`Base.STDERR`](@ref).
> Juliaの `__init __()` 関数 (`base/sysimg.jl`) は、[`Base.STDIN`](@ref) のJuliaオブジェクトを作成するために `reinit_stdio()` ( `base/stream.jl`) [`Base.STDOUT`](@ref) と [`Base.STDERR`](@ref)です。

<!-- EN -->
`reinit_stdio()` uses [`ccall`](@ref) to retrieve pointers to `JL_STD*` and calls `jl_uv_handle_type()` to inspect the type of each stream.
> `reinit_stdio（）`は `` ccall``（@ ref）を使って `JL_STD *`へのポインタを取得し、 `jl_uv_handle_type（）`を呼び出して各ストリームの型を検査します。
<!-- EN -->
It then creates a Julia `Base.IOStream`, `Base.TTY` or `Base.PipeEndpoint` object to represent each stream, e.g.:
> 次に、各ストリームを表すジュリア `Base.IOStream`、` Base.TTY`または `Base.PipeEndpoint`オブジェクトを作成します。

```
$ julia -e 'println(typeof((STDIN, STDOUT, STDERR)))'
Tuple{Base.TTY,Base.TTY,Base.TTY}

$ julia -e 'println(typeof((STDIN, STDOUT, STDERR)))' < /dev/null 2>/dev/null
Tuple{IOStream,Base.TTY,IOStream}

$ echo hello | julia -e 'println(typeof((STDIN, STDOUT, STDERR)))' | cat
Tuple{Base.PipeEndpoint,Base.PipeEndpoint,Base.TTY}
```

<!-- EN -->
The [`Base.read`](@ref) and [`Base.write`](@ref) methods for these streams use [`ccall`](@ref) to call libuv wrappers in `src/jl_uv.c`, e.g.:
> これらのストリームの[`Base.read`]（@ ref）メソッドと` `Base.write`（@ref）メソッドは` `src / jl_uv.c`のlibuvラッパーを呼び出すために[` ccall`]（@ref）を使います。 例えば：

```
stream.jl: function write(s::IO, p::Ptr, nb::Integer)
               -> ccall(:jl_uv_write, ...)
  jl_uv.c:          -> int jl_uv_write(uv_stream_t *stream, ...)
                        -> uv_write(uvw, stream, buf, ...)
```

## printf() during initialization

<!-- EN -->
The libuv streams relied upon by `jl_printf()` etc., are not available until midway through initialization of the runtime (see `init.c`, `init_stdio()`).
> `jl_printf（）`などに依存するlibuvストリームは、ランタイムの初期化の途中で利用可能です（ `init.c`、` init_stdio（） `参照）。
<!-- EN -->
Error messages or warnings that need to be printed before this are routed to the standard C library `fwrite()` function by the following mechanism:
> それ以前に出力する必要があるエラーメッセージや警告は、次のメカニズムによって標準のCライブラリ `fwrite（）`関数に送られます：

<!-- EN -->
In `sys.c`, the `JL_STD*` stream pointers are statically initialized to integer constants: `STD*_FILENO (0, 1 and 2)`.
> `sys.c`では、` JL_STD * `ストリームポインタは静的に整数定数に初期化されます：` STD * _FILENO（0,1,2） `。
<!-- EN -->
In `jl_uv.c` the `jl_uv_puts()` function checks its `uv_stream_t* stream` argument and calls `fwrite()` if stream is set to `STDOUT_FILENO` or `STDERR_FILENO`.
> `jl_uv_put.c（）`関数はstreamが `STDOUT_FILENO`または` STDERR_FILENO`に設定されている場合、 `uv_stream_t * stream`引数をチェックし、` fwrite（） `を呼び出します。

<!-- EN -->
This allows for uniform use of `jl_printf()` throughout the runtime regardless of whether or not any particular piece of code is reachable before initialization is complete.
> これにより、初期化が完了する前に特定のコードに到達可能かどうかにかかわらず、ランタイム全体で `jl_printf（）`を一様に使用することができます。

## Legacy `ios.c` library

<!-- EN -->
The `src/support/ios.c` library is inherited from [femtolisp](https://github.com/JeffBezanson/femtolisp).
> `src / support / ios.c`ライブラリは[femtolisp](https://github.com/JeffBezanson/femtolisp)から継承されています。
<!-- EN -->
It provides cross-platform buffered file IO and in-memory temporary buffers.
> これは、クロスプラットフォームのバッファ付きファイルIOとインメモリの一時バッファを提供します。

<!-- EN -->
`ios.c` is still used by:
> `ios.c`はまだ使用されています：

  * `src/flisp/*.c`
  * `src/dump.c` – for serialization file IO and for memory buffers.
  * `src/staticdata.c` – for serialization file IO and for memory buffers.
  * `base/iostream.jl` – for file IO (see `base/fs.jl` for libuv equivalent).

<!-- EN -->
Use of `ios.c` in these modules is mostly self-contained and separated from the libuv I/O system.
> これらのモジュールでの `ios.c 'の使用は、ほとんどが自己完結型であり、libuv入出力システムから分離されています。
<!-- EN -->
However, there is [one place](https://github.com/JuliaLang/julia/blob/master/src/flisp/print.c#L654) where femtolisp calls through to `jl_printf()` with a legacy `ios_t` stream.
> しかし、femtolispが従来の `ios_t` ストリームを使って `jl_printf()` を呼び出す[1つの場所](https://github.com/JuliaLang/julia/blob/master/src/flisp/print.c#L654) があります 。

<!-- EN -->
There is a hack in `ios.h` that makes the `ios_t.bm` field line up with the `uv_stream_t.type` and ensures that the values used for `ios_t.bm` to not overlap with valid `UV_HANDLE_TYPE` values.
> `ios.h` には、 `ios_t.bm` フィールドが  `uv_stream_t.type` と一直線上に並んでいて、 `ios_t.bm` が有効な  `UV_HANDLE_TYPE` 値と重ならないようにするためのハックがあります。
<!-- EN -->
This allows `uv_stream_t` pointers to point to `ios_t` streams.
> これにより、 `uv_stream_t` ポインタが `ios_t` ストリームを指し示すことができます。

<!-- EN -->
This is needed because `jl_printf()` caller `jl_static_show()` is passed an `ios_t` stream by femtolisp's `fl_print()` function.
> これは、 `jl_printf()` 呼び出し側 `jl_static_show()` がfemtolispの `fl_print()` 関数によって `ios_t` ストリームを渡されるために必要です。
<!-- EN -->
Julia's `jl_uv_puts()` function has special handling for this:
> Juliaの `jl_uv_puts()` 関数はこれを特別に扱います：

```c
if (stream->type > UV_HANDLE_TYPE_MAX) {
    return ios_write((ios_t*)stream, str, n);
}
```
