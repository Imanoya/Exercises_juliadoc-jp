# Using Valgrind with Julia

<!--JP-->
# Juliaで Valgrid を使用する

[Valgrind](http://valgrind.org/) is a tool for memory debugging, memory leak detection, and profiling.
This section describes things to keep in mind when using Valgrind to debug memory issues with Julia.
<!--JP-->
> [Valgrind]（http://valgrind.org/）は、メモリのデバッグ、メモリリークの検出、プロファイリングのためのツールです。
> このセクションでは、Valgrindを使用してJuliaのメモリ問題をデバッグする際に留意すべき事項について説明します。

## General considerations

<!--EN-->
By default, Valgrind assumes that there is no self modifying code in the programs it runs.
This assumption works fine in most instances but fails miserably for a just-in-time compiler like `julia`.
For this reason it is crucial to pass `--smc-check=all-non-file` to `valgrind`, else code may crash or behave unexpectedly (often in subtle ways).
<!--JP-->
> デフォルトでは、Valgrindは、実行中のプログラムに自己修正コードがないとみなします。
> この仮定はほとんどの場合うまく動作しますが、 `julia`のようなジャストインタイムコンパイラでは悲惨に失敗します。
> このため、 `--smc-check=all-non-file` を `valgrind` に渡すことが重要です。
> そうでないと、コードがクラッシュしたり予期せず（しばしば微妙に）動作する可能性があります。

<!--EN-->
In some cases, to better detect memory errors using Valgrind it can help to compile `julia` with memory pools disabled.
The compile-time flag `MEMDEBUG` disables memory pools in Julia, and `MEMDEBUG2` disables memory pools in FemtoLisp.
To build `julia` with both flags, add the following line to `Make.user`:
<!--JP-->
> 場合によっては、Valgrindを使用してメモリエラーをよりよく検出するために、メモリプールを無効にして `julia` をコンパイルするのに役立ちます。
> コンパイル時フラグ `MEMDEBUG` はJuliaのメモリプールを無効にし、 `MEMDEBUG2` はFemtoLispのメモリプールを無効にします。
> 両方のフラグで `julia`をビルドするには、` Make.user`に次の行を追加します：

```julia
CFLAGS = -DMEMDEBUG -DMEMDEBUG2
```

<!--EN-->
Another thing to note: if your program uses multiple workers processes, it is likely that you want all such worker processes to run under Valgrind, not just the parent process.
To do this, pass `--trace-children=yes` to `valgrind`.
<!--JP-->
> 別の注意点として、プログラムが複数のワーカープロセスを使用する場合、そのようなワーカープロセスをすべて親プロセスだけでなくValgrindで実行する必要がある可能性があります。
> これを行うには、 `--trace-children = yes`を` valgrind`に渡します。

## Suppressions

<!--EN-->
Valgrind will typically display spurious warnings as it runs.
To reduce the number of such warnings, it helps to provide a [suppressions file](http://valgrind.org/docs/manual/manual-core.html#manual-core.suppress) to Valgrind.
A sample suppressions file is included in the Julia source distribution at `contrib/valgrind-julia.supp`.
<!--JP-->
> Valgrindは通常、実行時に偽の警告を表示します。
> そのような警告の数を減らすには、[抑制ファイル]（http://valgrind.org/docs/manual/manual-core.html#manual-core.suppress）をValgrindに提供することが役立ちます。
> 抑制ファイルのサンプルは `contrib/valgrind-julia.supp` のJuliaソースディストリビューションに含まれています。

<!--EN-->
The suppressions file can be used from the `julia/` source directory as follows:
<!--JP-->
> 抑止ファイルは `julia/` ソースディレクトリから以下のように使用できます：

```
$ valgrind --smc-check=all-non-file --suppressions=contrib/valgrind-julia.supp ./julia progname.jl
```

Any memory errors that are displayed should either be reported as bugs or contributed as additional suppressions.
Note that some versions of Valgrind are [shipped with insufficient default suppressions](https://github.com/JuliaLang/julia/issues/8314#issuecomment-55766210), so that may be one thing to consider before submitting any bugs.
> 表示されるメモリエラーは、バグとして報告されるか、追加の抑制として寄与されるべきです。
> Valgrindのいくつかのバージョンは[不十分なデフォルト抑制を伴って出荷されています]（https://github.com/JuliaLang/julia/issues/8314#issuecomment-55766210）、バグを提出する前に考慮すべき点があることに注意してください。

## Running the Julia test suite under Valgrind

<!--EN-->
It is possible to run the entire Julia test suite under Valgrind, but it does take quite some time (typically several hours).
To do so, run the following command from the `julia/test/` directory:
<!--JP-->
> Valgrindの下でJuliaテストスイート全体を実行することは可能ですが、かなり時間がかかります（通常は数時間かかる）。
> これを行うには、 `julia/test/` ディレクトリから次のコマンドを実行します:

```
valgrind --smc-check=all-non-file --trace-children=yes --suppressions=$PWD/../contrib/valgrind-julia.supp ../julia runtests.jl all
```

<!--EN-->
If you would like to see a report of "definite" memory leaks, pass the flags `--leak-check=full --show-leak-kinds=definite` to `valgrind` as well.
<!--JP-->
> "明確な"メモリリークのレポートを見たい場合は、フラグ `--leak-check=full --show-leak-kinds=definite` を `valgrind` にも渡します。

## Caveats

<!--EN-->
Valgrind currently [does not support multiple rounding modes](https://bugs.kde.org/show_bug.cgi?id=136779),
so code that adjusts the rounding mode will behave differently when run under Valgrind.
<!--JP-->
> Valgrindは現在、複数の丸めモードをサポートしていません（https://bugs.kde.org/show_bug.cgi?id=136779）。したがって、丸めモードを調整するコードは、Valgrindで実行すると、異なる動作をします。

<!--EN-->
In general, if after setting `--smc-check=all-non-file` you find that your program behaves differently when run under Valgrind, it may help to pass `--tool=none` to `valgrind` as you investigate further.
This will enable the minimal Valgrind machinery but will also run much faster than when the full memory checker is enabled.
<!--JP-->
> 一般的に、 `--smc-check = all-non-file`を設定した場合、あなたのプログラムは異なった動作をします、Valgrindの下で実行すると、さらに調査する際に `valgrind`に `--tool=none` を渡すのに役立ちます。
> これによりValgrindの機械は最小限に抑えられますが、メモリチェッカーが有効になります。
