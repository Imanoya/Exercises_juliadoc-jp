<!-- Start -->

# Environment Variables

> # 環境変数

<!-- End -->
<!-- Start -->
Julia may be configured with a number of environment variables, either in the usual way of the operating system, or in a portable way from within Julia.
> ジュリアは、オペレーティングシステムの通常の方法で、またはジュリア内から移植可能な方法で、いくつかの環境変数で構成することができます。
<!-- End -->
<!-- Start -->
Suppose you want to set the environment variable `JULIA_EDITOR` to `vim`, then either type `ENV["JULIA_EDITOR"] = "vim"` for instance in the REPL to make this change on a case by case basis, or add the same to the user configuration file `.juliarc.jl` in the user's home directory to have a permanent effect. 
> 環境変数 `JULIA_EDITOR`を` vim`に設定し、REPLに `ENV [" JULIA_EDITOR "] =" vim "`と入力して、この変更をケースバイケースで行うか、 ユーザのホームディレクトリの `.juliarc.jl`ユーザ設定ファイルと同じで、永続的な効果があります。
<!-- End -->
<!-- Start -->
The current value of the same environment variable is determined by evaluating `ENV["JULIA_EDITOR"]`.
> 同じ環境変数の現在の値は、 `ENV [" JULIA_EDITOR "]`を評価することによって決定されます。
<!-- End -->

<!-- Start -->
The environment variables that Julia uses generally start with `JULIA`. 
> Juliaが使用する環境変数は、一般的に `JULIA`から始まります。
<!-- End -->
<!-- Start -->
If [`Base.versioninfo`](@ref) is called with `verbose` equal to `true`, then the output will list defined environment variables relevant for Julia, including those for which `JULIA` appears in the name.
> [`Base.versioninfo`](@ref)が `verbose` を `true` として呼び出された場合、出力にはジュリアに関係する定義済みの環境変数がリストされます。
<!-- End -->

<!-- Start -->

## File locations

ファイルの場所

<!-- End -->
<!-- Start -->

### `JULIA_HOME`

<!-- End -->
<!-- Start -->
The absolute path of the directory containing the Julia executable, which sets the global variable [`Base.JULIA_HOME`](@ref). 
> グローバル変数[`Base.JULIA_HOME`](@ ref)を設定するJulia実行可能ファイルを含むディレクトリの絶対パス。
<!-- End -->
<!-- Start -->
If `$JULIA_HOME` is not set, then Julia determines the value `Base.JULIA_HOME` at run-time.
> `$ JULIA_HOME`が設定されていない場合、Juliaは実行時に` Base.JULIA_HOME`という値を決定します。
<!-- End -->

<!-- Start -->
The executable itself is one of
> 実行可能ファイル自体は
<!-- End -->

```sh
$JULIA_HOME/julia
$JULIA_HOME/julia-debug
```

<!-- Start -->
by default.
> デフォルトでは
<!-- End -->

<!-- Start -->
The global variable `Base.DATAROOTDIR` determines a relative path from `Base.JULIA_HOME` to the data directory associated with Julia. 
> グローバル変数 `Base.DATAROOTDIR`は` Base.JULIA_HOME`からJuliaに関連するデータディレクトリまでの相対パスを決定します。
<!-- End -->
<!-- Start -->
Then the path
その後、経路
<!-- End -->

```sh
$JULIA_HOME/$DATAROOTDIR/julia/base
```

<!-- Start -->
determines the directory in which Julia initially searches for source files (via `Base.find_source_file()`).
> Juliaが最初に(Base.find_source_file()を介して)ソースファイルを検索するディレクトリを決定します。
<!-- End -->

<!-- Start -->
Likewise, the global variable `Base.SYSCONFDIR` determines a relative path to the configuration file directory. 
> 同様に、グローバル変数 `Base.SYSCONFDIR`は、設定ファイルディレクトリへの相対パスを決定します。
<!-- End -->
<!-- Start -->
Then Julia searches for a `juliarc.jl` file at
> Juliaは `juliarc.jl`ファイルを
<!-- End -->

```sh
$JULIA_HOME/$SYSCONFDIR/julia/juliarc.jl
$JULIA_HOME/../etc/julia/juliarc.jl
```

<!-- Start -->
by default (via `Base.load_juliarc()`).
> デフォルトで( `Base.load_juliarc()`を介して)。
<!-- End -->

<!-- Start -->
For example, a Linux installation with a Julia executable located at `/bin/julia`, a `DATAROOTDIR` of `../share`, and a `SYSCONFDIR` of `../etc` will have `JULIA_HOME` set to `/bin`, a source-file search path of
> たとえば、 `/ bin / julia`にあるJulia実行ファイル、` ../ share`の `DATAROOTDIR`、` ../ etc`の `SYSCONFDIR`を持つLinuxインストールでは、` JULIA_HOME`が `/ bin`はソースファイルの検索パスです。
<!-- End -->

```sh
/share/julia/base
```

<!-- Start -->
and a global configuration search path of
およびグローバル構成検索パス
<!-- End -->

```sh
/etc/julia/juliarc.jl
```

<!-- Start -->

## `JULIA_LOAD_PATH`

<!-- End -->
<!-- Start -->
A separated list of absolute paths that are to be appended to the variable [`LOAD_PATH`](@ref). 
> 変数[`LOAD_PATH`](@ ref)に追加される絶対パスの区切られたリスト。
<!-- End -->
<!-- Start -->
(In Unix-like systems, the path separator is `:`; in Windows systems, the path separator is `;`.) 
> (Unixのようなシステムでは、パス区切りは `：`です; Windowsシステムでは、パス区切りは `;`です)。
<!-- End -->
<!-- Start -->
The `LOAD_PATH` variable is where [`Base.require`](@ref) and `Base.load_in_path()` look for code; it defaults to the absolute paths
> `LOAD_PATH`変数は[` Base.require`](@ref)と `Base.load_in_path()`がコードを探す場所です。 デフォルトでは絶対パスになります
<!-- End -->

```sh
$JULIA_HOME/../local/share/julia/site/v$(VERSION.major).$(VERSION.minor)
$JULIA_HOME/../share/julia/site/v$(VERSION.major).$(VERSION.minor)
```

<!-- Start -->
so that, e.g., version 0.6 of Julia on a Linux system with a Julia executable at `/bin/julia` will have a default `LOAD_PATH` of
`/ bin / julia`にあるJulia実行可能ファイルを持つLinuxシステム上のJuliaのバージョン0.6は、デフォルトの` LOAD_PATH`を
<!-- End -->

```sh
/local/share/julia/site/v0.6
/share/julia/site/v0.6
```

<!-- Start -->

## `JULIA_PKGDIR`

<!-- End -->
<!-- Start -->
The path of the parent directory `Pkg.Dir._pkgroot()` for the version-specific Julia package repositories. 
> バージョン固有のJuliaパッケージリポジトリの親ディレクトリ `Pkg.Dir._pkgroot()`のパス。
<!-- End -->
<!-- Start -->
If the path is relative, then it is taken with respect to the working directory. 
> パスが相対パスの場合は、作業ディレクトリに対して取得されます。
<!-- End -->
<!-- Start -->
If `$JULIA_PKGDIR` is not set, then `Pkg.Dir._pkgroot()` defaults to
> `$ JULIA_PKGDIR`が設定されていない場合、` Pkg.Dir._pkgroot() `はデフォルトで
<!-- End -->

```sh
$HOME/.julia
```

<!-- Start -->
Then the repository location [`Pkg.dir`](@ref) for a given Julia version is
> 指定されたJuliaバージョンのリポジトリの場所 [`Pkg.dir`](@ref) は、
<!-- End -->

```sh
$JULIA_PKGDIR/v$(VERSION.major).$(VERSION.minor)
```

<!-- Start -->
For example, for a Linux user whose home directory is `/home/alice`, the directory containing the package repositories would by default be
> 例えば、ホームディレクトリが `/home/alice` の Linuxユーザの場合、パッケージリポジトリを含むディレクトリは、デフォルトでは
<!-- End -->

```sh
/home/alice/.julia
```

<!-- Start -->
and the package repository for version 0.6 of Julia would be
> Juliaのバージョン0.6のパッケージリポジトリは
<!-- End -->

```sh
/home/alice/.julia/v0.6
```

<!-- Start -->

### `JULIA_HISTORY`

<!-- End -->
<!-- Start -->
The absolute path `Base.REPL.find_hist_file()` of the REPL's history file. 
> REPLの履歴ファイルの絶対パス `Base.REPL.find_hist_file()`。
<!-- End -->
<!-- Start -->
If `$JULIA_HISTORY` is not set, then `Base.REPL.find_hist_file()` defaults to
> `$ JULIA_HISTORY`が設定されていない場合、` Base.REPL.find_hist_file() `はデフォルトで
<!-- End -->

```sh
$HOME/.julia_history
```

<!-- Start -->

### `JULIA_PKGRESOLVE_ACCURACY`

<!-- End -->
<!-- Start -->
A positive `Int` that determines how much time the max-sum subroutine `MaxSum.maxsum()` of the package dependency resolver [`Base.Pkg.resolve`](@ref) will devote to attempting satisfying constraints before giving up: 
> パッケージ依存リゾルバ[`Base.Pkg.resolve`](@ ref)のmax-sumサブルーチン` MaxSum.maxsum() `が、あきらめる前に充足する制約を試みるのにどれくらいの時間を費やすかを決定する正の` Int`：
<!-- End -->
<!-- Start -->
this value is by default `1`, and larger values correspond to larger amounts of time.
> この値はデフォルトでは「1」であり、大きな値はより長い時間量に対応します。
<!-- End -->

<!-- Start -->
Suppose the value of `$JULIA_PKGRESOLVE_ACCURACY` is `n`. 
> `$ JULIA_PKGRESOLVE_ACCURACY` の値が `n` であるとします。

Then

*   the number of pre-decimation iterations is `20*n`,
*   the number of iterations between decimation steps is `10*n`, and
*   at decimation steps, at most one in every `20*n` packages is decimated.

<!-- Start -->
## External applications

<!-- End -->
<!-- Start -->

### `JULIA_SHELL`

<!-- End -->
<!-- Start -->
The absolute path of the shell with which Julia should execute external commands (via `Base.repl_cmd()`). Defaults to the environment variable `$SHELL`, and falls back to `/bin/sh` if `$SHELL` is unset.
> Juliaが(Base.repl_cmd()を介して)外部コマンドを実行するシェルの絶対パス。 環境変数 `$ SHELL`にデフォルト設定され、` $ SHELL`が設定されていなければ `/ bin / sh`に戻ります。
<!-- End -->

<!-- Start -->
!!! note
<!-- End -->
<!-- Start -->
  On Windows, this environment variable is ignored, and external commands are executed directly.
  >  Windowsでは、この環境変数は無視され、外部コマンドは直接実行されます。

<!-- End -->
<!-- Start -->

# `JULIA_EDITOR`

<!-- End -->
<!-- Start -->
The editor returned by `Base.editor()` and used in, e.g., [`Base.edit`](@ref), referring to the command of the preferred editor, for instance `vim`.
> エディタは `Base.editor()`によって返され、 `` Base.edit`(@ref)などで使用され、優先エディタのコマンド、例えば `vim 'を参照します。
<!-- End -->

<!-- Start -->
`$JULIA_EDITOR` takes precedence over `$VISUAL`, which in turn takes precedence over `$EDITOR`. 
> `$JULIA_EDITOR` は `$ VISUAL` よりも優先され、 `$ EDITOR` よりも優先されます。
<!-- End -->
<!-- Start -->
If none of these environment variables is set, then the editor is taken to be `open` on Windows and OS X, or `/etc/alternatives/editor` if it exists, or `emacs` otherwise.
> これらの環境変数が1つも設定されていない場合、エディタはWindowsとOS Xでは `open` 、存在する場合は`/etc/alternatives/editor` 、それ以外の場合は `emacs` となります。
<!-- End -->

<!-- Start -->
!!! note
<!-- End -->
<!-- Start -->
   `$JULIA_EDITOR` is *not* used in the determination of the editor for [`Base.Pkg.edit`](@ref): this function checks `$VISUAL` and `$EDITOR` alone.
   > `$JULIA_EDITOR` は [`Base.Pkg.edit`](@ref) のエディタの決定に *使用されません。* この関数は `$VISUAL` と`$EDITOR` だけをチェックします。

<!-- End -->
<!-- Start -->
<!-- End -->
<!-- Start -->
 Parallelization

<!-- End -->
<!-- Start -->
<!-- End -->
<!-- Start -->
<!-- End -->
<!-- Start -->
 `JULIA_CPU_CORES`

<!-- End -->
<!-- Start -->
Overrides the global variable [`Base.Sys.CPU_CORES`](@ref), the number of logical CPU cores available.
使用可能な論理CPUコアの数であるグローバル変数[`Base.Sys.CPU_CORES`](@ ref)を上書きします。

<!-- End -->
<!-- Start -->
<!-- End -->
<!-- Start -->
<!-- End -->
<!-- Start -->
 `JULIA_WORKER_TIMEOUT`

#A [`Float64`](@ref) that sets the value of `Base.worker_timeout()` (default: `60.0`).
Base.worker_timeout()の値を設定する[`Float64`](@ ref)です(デフォルト：` 60.0`)。
#This function gives the number of seconds a worker process will wait for a master process to establish a connection before dying.
このファンクションは、作業プロセスがマスタープロセスが接続を確立してから終了するまで待機する秒数を示します。

### `JULIA_NUM_THREADS`

#An unsigned 64-bit integer (`uint64_t`) that sets the maximum number of threads available to Julia. 
Juliaが利用できるスレッドの最大数を設定する符号なし64ビット整数( `uint64_t`)。
#If `$JULIA_NUM_THREADS` exceeds the number of available physical CPU cores, then the number of threads is set to the number of cores. 
`$ JULIA_NUM_THREADS`が使用可能な物理CPUコアの数を超えた場合、スレッドの数はコアの数に設定されます。
#If `$JULIA_NUM_THREADS` is not positive or is not set, or if the number of CPU cores cannot be determined through system calls, then the number of threads is set to `1`.
`$ JULIA_NUM_THREADS`が正でないか、または設定されていない場合、またはCPUコアの数がシステムコールによって決定できない場合、スレッド数は` 1`に設定されます。

### `JULIA_THREAD_SLEEP_THRESHOLD`

#If set to a string that starts with the case-insensitive substring `"infinite"`, then spinning threads never sleep. 
大文字と小文字を区別しない部分文字列 `"無限 "で始まる文字列に設定された場合、スレッドは決してスリープしません。
#Otherwise, `$JULIA_THREAD_SLEEP_THRESHOLD` is interpreted as an unsigned 64-bit integer (`uint64_t`) and gives, in nanoseconds, the amount of time after which spinning threads should sleep.
それ以外の場合、 `$ JULIA_THREAD_SLEEP_THRESHOLD`は符号なし64ビット整数(` uint64_t`)として解釈され、回転スレッドがスリープするまでの時間がナノ秒単位で与えられます。

### `JULIA_EXCLUSIVE`

#If set to anything besides `0`, then Julia's thread policy is consistent with running on a dedicated machine: the master thread is on proc 0, and threads are affinitized. 
`0`以外の値に設定すると、Juliaのスレッドポリシーは専用マシン上での実行と一致します。マスタースレッドはproc0にあり、スレッドは親和されています。
#Otherwise, Julia lets the operating system handle thread policy.
それ以外の場合、Juliaはオペレーティングシステムにスレッドポリシーを処理させます。

## REPL formatting

#Environment variables that determine how REPL output should be formatted at the terminal. 
REPL出力を端末でどのようにフォーマットするかを決定する環境変数。
#Generally, these variables should be set to [ANSI terminal escape sequences](http://ascii-table.com/ansi-escape-sequences.php). 
一般に、これらの変数は[ANSI端末エスケープシーケンス](http://ascii-table.com/ansi-escape-sequences.php)に設定する必要があります。
#Julia provides a high-level interface with much of the same functionality: 
ジュリアは、同じ機能性の多くを備えた高水準のインタフェースを提供します。
#see the section on [Interacting With Julia](@ref).
[Juliaとの対話](@ ref)の節を参照してください。

### `JULIA_ERROR_COLOR`

#The formatting `Base.error_color()` (default: light red, `"\033[91m"`) that errors should have at the terminal.
`Base.error_color()`の書式設定(デフォルト：明るい赤、 `" \ 033 [91m "`)です。

### `JULIA_WARN_COLOR`

#The formatting `Base.warn_color()` (default: yellow, `"\033[93m"`) that warnings should have at the terminal.
`Base.warn_color()`の書式設定(デフォルト：yellow、 `" \ 033 [93m "`)。

### `JULIA_INFO_COLOR`

#The formatting `Base.info_color()` (default: cyan, `"\033[36m"`) that info should have at the terminal.
`Base.info_color()`の書式設定(デフォルト：シアン、 `" \ 033 [36m "`)情報が端末になければなりません。

### `JULIA_INPUT_COLOR`

#The formatting `Base.input_color()` (default: normal, `"\033[0m"`) that input should have at the terminal.
`Base.input_color()`の書式設定(デフォルト：normal、 `" \ 033 [0m "`)です。

### `JULIA_ANSWER_COLOR`

#The formatting `Base.answer_color()` (default: normal, `"\033[0m"`) that output should have at the terminal.
`Base.answer_color()`の書式設定(デフォルト：normal、 `" \ 033 [0m "`)です。

### `JULIA_STACKFRAME_LINEINFO_COLOR`

#The formatting `Base.stackframe_lineinfo_color()` (default: bold, `"\033[1m"`) that line info should have during a stack trace at the terminal.
`Base.stackframe_lineinfo_color()`の書式設定(デフォルト：太字、 `" \ 033 [1m "`は、行情報が端末のスタックトレース中に持つべきである。)です。

### `JULIA_STACKFRAME_FUNCTION_COLOR`

#The formatting `Base.stackframe_function_color()` (default: bold, `"\033[1m"`) that function calls should have during a stack trace at the terminal.
`Base.stackframe_function_color()`(デフォルト：太字、 `" \ 033 [1m "`))は、端末でスタックトレース中に関数呼び出しが持つべきである。

### `JULIA_STACKFRAME_FUNCTION_COLOR`

## Debugging and profiling

### `JULIA_GC_ALLOC_POOL`, `JULIA_GC_ALLOC_OTHER`, `JULIA_GC_ALLOC_PRINT`

#If set, these environment variables take strings that optionally start with the character `'r'`, followed by a string interpolation of a colon-separated list of three signed 64-bit integers (`int64_t`). 
設定されている場合、これらの環境変数はオプションで文字 `` r ''で始まり、続いて3つの符号付き64ビット整数( `int64_t`)のコロン区切りリストの文字列補間が続きます。
#This triple of integers `a:b:c` represents the arithmetic sequence `a`, `a + b`, `a + 2*b`, ... `c`.
この3つの整数a：b：cは算術シーケンスa、a + b、a + 2 * b、... cを表す。

   * 
   # If it's the `n`th time that `jl_gc_pool_alloc()` has been called, and `n` belongs to the arithmetic    sequence represented by `$JULIA_GC_ALLOC_POOL`, then garbage collection is forced.
   `jl_gc_pool_alloc()`が呼び出され、 `n`が` $ JULIA_GC_ALLOC_POOL`で表される算術シーケンスに属しているn回目の場合、ガベージコレクションは強制されます。
   * 
   # If it's the `n`th time that `maybe_collect()` has been called, and `n` belongs to the arithmetic      sequence represented by `$JULIA_GC_ALLOC_OTHER`, then garbage collection is forced.
   `maybe_collect()`が呼び出され、 `n`が` $ JULIA_GC_ALLOC_OTHER`で表される算術シーケンスに属している `n回目の場合は、ガベージコレクションが強制されます。
   * 
   # If it's the `n`th time that `jl_gc_collect()` has been called, and `n` belongs to the arithmetic sequence represented by `$JULIA_GC_ALLOC_PRINT`, then counts for the number of calls to `jl_gc_pool_alloc()` and `maybe_collect()` are printed.
   `jl_gc_collect()`が呼び出され、 `n`が` $ JULIA_GC_ALLOC_PRINT`で表される算術シーケンスに属している `n回目の場合、` jl_gc_pool_alloc() `と` maybe_collect () `が印刷されます。

   # If the value of the environment variable begins with the character `'r'`, then the interval between garbage collection events is randomized.
   環境変数の値が文字 '' r ''で始まる場合、ガベージコレクションイベントの間隔はランダム化されます。

#!!! note
!!!注意

   # These environment variables only have an effect if Julia was compiled with garbage-collection debugging (that is, if `WITH_GC_DEBUG_ENV` is set to `1` in the build configuration).
    これらの環境変数は、ジュリアがガベージコレクションデバッグ(つまり、ビルド設定で `WITH_GC_DEBUG_ENV`が` 1`に設定されている)でコンパイルされた場合にのみ有効です。

### `JULIA_GC_NO_GENERATIONAL`

#If set to anything besides `0`, then the Julia garbage collector never performs "quick sweeps" of memory.
`0`以外に設定されている場合、Juliaガベージコレクタは決してメモリの"クイックスイープ "を実行しません。

#!!! note
!!! 注意

   # This environment variable only has an effect if Julia was compiled with garbage-collection debugging (that is, if `WITH_GC_DEBUG_ENV` is set to `1` in the build configuration).
    この環境変数は、ジュリアがガベージコレクションデバッグ(つまり、ビルド設定で `WITH_GC_DEBUG_ENV`が` 1`に設定されている)でコンパイルされた場合にのみ有効です。

### `JULIA_GC_WAIT_FOR_DEBUGGER`

#If set to anything besides `0`, then the Julia garbage collector will wait for a debugger to attach instead of aborting whenever there's a critical error.
`0`以外の値に設定されている場合、Juliaガベージコレクタは、重大なエラーが発生したときにデバッガがアボートするのではなく、アタッチするのを待ちます。

#!!! note
!!! 注意

   # This environment variable only has an effect if Julia was compiled with garbage-collection debugging (that is, if `WITH_GC_DEBUG_ENV` is set to `1` in the build configuration).
    この環境変数は、ジュリアがガベージコレクションデバッグ(つまり、ビルド設定で `WITH_GC_DEBUG_ENV`が` 1`に設定されている)でコンパイルされた場合にのみ有効です。

### `ENABLE_JITPROFILING`

#If set to anything besides `0`, then the compiler will create and register an event listener for just-in-time (JIT) profiling.
`0`以外に設定された場合、コンパイラはJIT(Just-In-Time)プロファイリング用のイベントリスナーを作成して登録します。

#!!! note
!!! 注意

   # This environment variable only has an effect if Julia was compiled with JIT profiling support, using either
    この環境変数は、JuliaがJITプロファイリングをサポートするようにコンパイルされている場合、

*   
   # Intel's [VTune™ Amplifier](https://software.intel.com/en-us/intel-vtune-amplifier-xe) (`USE_INTEL_JITEVENTS` set to `1` in the build configuration), or
   インテルの[VTune™Amplifier](https://software.intel.com/en-us/intel-vtune-amplifier-xe)(ビルド構成で `USE_INTEL_JITEVENTS`を` 1`に設定)、または
*
*   
   # [OProfile](http://oprofile.sourceforge.net/news/) (`USE_OPROFILE_JITEVENTS` set to `1` in the build configuration).
    [OProfile](http://oprofile.sourceforge.net/news/)( `USE_OPROFILE_JITEVENTS`は` 1`に設定されています)

### `JULIA_LLVM_ARGS`

#Arguments to be passed to the LLVM backend.
LLVMバックエンドに渡される引数。

#!!! note
!!! 注意

   # This environment variable has an effect only if Julia was compiled with `JL_DEBUG_BUILD` set — in particular, the `julia-debug` executable is always compiled with this build variable.
    この環境変数は、Juliaが `JL_DEBUG_BUILD`を指定してコンパイルされた場合にのみ有効です。特に、` julia-debug`実行可能ファイルは常にこのビルド変数でコンパイルされます。

### `JULIA_DEBUG_LOADING`

#If set, then Julia prints detailed information about the cache in the loading process of [`Base.require`](@ref).
設定されている場合、Juliaは[`Base.require`](@ ref)の読み込みプロセスでキャッシュに関する詳細情報を表示します。

