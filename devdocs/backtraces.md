# Reporting and analyzing crashes (segfaults)

So you managed to break Julia.  Congratulations!
> だからあなたはジュリアを壊すことができた。 おめでとう！
Collected here are some general procedures you can undergo for common symptoms encountered when something goes awry.
> ここには何かが間違っているときに遭遇する一般的な症状のために受けることができるいくつかの一般的な手順が集められています。
Including the information from these debugging steps can greatly help the maintainers when tracking down a segfault or trying to figure out why your script is running slower than expected.
> これらのデバッグステップからの情報を含めると、セグメンテーションを追跡したり、スクリプトが予想よりも遅く実行されている理由を突き止める際に、メンテナが大いに役立ちます。

If you've been directed to this page, find the symptom that best matches what you're experiencing and follow the instructions to generate the debugging information requested.
> このページに移動した場合は、発生している症状に最もよく合った症状を見つけ、指示に従って要求されたデバッグ情報を生成してください。
Table of symptoms:
> 症状の表：

  * [Segfaults during bootstrap (`sysimg.jl`)](@ref)
  > * [ブートストラップ中のSegfault（ `sysimg.jl`）]（@ref）
  * [Segfaults when running a script](@ref)
  > * [スクリプト実行時のエラー]（@ref）
  * [Errors during Julia startup](@ref)
  > * [ジュリア起動時のエラー]（@ref）

## [Version/Environment info](@id dev-version-info)

No matter the error, we will always need to know what version of Julia you are running.
> エラーが発生しても、実行しているJuliaのバージョンを常に把握する必要があります。
When Julia first starts up, a header is printed out with a version number and date.
> Juliaが最初に起動すると、ヘッダーにバージョン番号と日付が印刷されます。
Please also include the output of `versioninfo()` in any report you create:
> 作成したレポートに `versioninfo（）` の出力も含めてください：

```@repl
versioninfo()
```

## Segfaults during bootstrap (`sysimg.jl`)

Segfaults toward the end of the `make` process of building Julia are a common symptom of something going wrong while Julia is preparsing the corpus of code in the `base/` folder.
> Juliaが `base/` フォルダ内のコードのコーパスを準備している間に、Juliaを構築する `make` プロセスの終わりに向かうSegfaultsは、間違ったことの共通の症状です。
Many factors can contribute toward this process dying unexpectedly, however it is as often as not due to an error in the C-code portion of Julia, and as such must typically be debugged with a debug build inside of `gdb`.
> 多くの要素が予期せず死んでしまうこのプロセスに寄与する可能性がありますが、JuliaのCコード部分でエラーが発生することはよくありません。通常は `gdb` 内部のデバッグビルドでデバッグする必要があります。
Explicitly:
> 明示的に：

Create a debug build of Julia:
> Juliaのデバッグビルドを作成します。

```
$ cd <julia_root>
$ make debug
```

Note that this process will likely fail with the same error as a normal `make` incantation, however this will create a debug executable that will offer `gdb` the debugging symbols needed to get accurate backtraces.
> このプロセスはおそらく通常の `make` コマンドと同じエラーで失敗するでしょうが、正確なバックトレースを得るために必要なデバッグシンボルを `gdb` で提供するデバッグ実行可能ファイルを作成します。
Next, manually run the bootstrap process inside of `gdb`:
> 次に、 `gdb` の内部で手動でブートストラッププロセスを実行します：

```
$ cd base/
$ gdb -x ../contrib/debug_bootstrap.gdb
```

This will start `gdb`, attempt to run the bootstrap process using the debug build of Julia, and print out a backtrace if (when) it segfaults.
> これは `gdb` を起動し、Juliaのデバッグビルドを使用してブートストラッププロセスを実行しようとし、segfaultsの場合はバックトレースを出力します。
You may need to hit `<enter>` a few times to get the full backtrace.
> フルバックトレースを取得するには、 `<enter>` を数回押してください。
Create a [gist](https://gist.github.com) with the backtrace, the [version info](@ref dev-version-info), and any other pertinent information you can think of and open a new [issue](https://github.com/JuliaLang/julia/issues?q=is%3Aopen) on Github with a link to the gist.
> バックトレース、[バージョン情報](@ref dev-version-info)、その他の関連情報を使って[gist](https://gist.github.com)を作成し、新しい[issue ](https://github.com/JuliaLang/julia/issues?q=is%3Aopen)をクリックすると、要点へのリンクが表示されます。


## Segfaults when running a script

The procedure is very similar to [Segfaults during bootstrap (`sysimg.jl`)](@ref).
> この手順は [ブートストラップ中の Segfaults( `sysimg.jl`)](@ref) と非常によく似ています。
Create a debug build of Julia, and run your script inside of a debugged Julia process:
> Juliaのデバッグビルドを作成し、デバッグされたJuliaプロセスの内部でスクリプトを実行します。


```
$ cd <julia_root>
$ make debug
$ gdb --args usr/bin/julia-debug <path_to_your_script>
```

Note that `gdb` will sit there, waiting for instructions.
> `gdb` がそこに座って指示を待つことに注意してください。
Type `r` to run the process, and `bt` to generate a backtrace once it segfaults:
> プロセスを実行するために `r` と入力し、一度 segfaults を実行するとバックトレースを生成するには `bt` を入力します：

```
(gdb) r
Starting program: /home/sabae/src/julia/usr/bin/julia-debug ./test.jl
...
(gdb) bt
```

Create a [gist](https://gist.github.com) with the backtrace, the [version info](@ref dev-version-info), and any other pertinent information you can think of and open a new [issue](https://github.com/JuliaLang/julia/issues?q=is%3Aopen) on Github with a link to the gist.
> バックトレース、[バージョン情報](@ref dev-version-info)、その他の関連情報を使って [gist](https://gist.github.com) を作成し、新しい[issue](https://github.com/JuliaLang/julia/issues?q=is%3Aopen) をクリックすると、要点へのリンクが表示されます。

## Errors during Julia startup

Occasionally errors occur during Julia's startup process (especially when using binary distributions, as opposed to compiling from source) such as the following:
> Juliaの起動プロセス中にエラーが発生することがあります（特にソースからコンパイルするのではなくバイナリディストリビューションを使用する場合）。

```julia
$ julia
exec: error -5
```

These errors typically indicate something is not getting loaded properly very early on in the bootup phase, and our best bet in determining what's going wrong is to use external tools to audit the disk activity of the `julia` process:
> これらのエラーは通常、ブートアップ段階で非常に早くロードされないことを示しています。何がうまくいかないのかを判断する最も良い方法は、外部ツールを使用して `julia` プロセスのディスクアクティビティを監査することです。

  * On Linux, use `strace`:

    ```
    $ strace julia
    ```
  * On OSX, use `dtruss`:

    ```
    $ dtruss -f julia
    ```

Create a [gist](https://gist.github.com) with the `strace`/ `dtruss` ouput, the [version info](@ref dev-version-info), and any other pertinent information and open a new [issue](https://github.com/JuliaLang/julia/issues?q=is%3Aopen) on Github with a link to the gist.
> `strace`/ `dtruss` 出力、[version info]（@ref dev-version-info）、その他の関連情報を使って[gist](https://gist.github.com) を作成し、 要点へのリンクを持つ Github の新しい [問題](https://github.com/JuliaLang/julia/issues?q=is%3Aopen)

## Glossary

A few terms have been used as shorthand in this guide:
> このガイドでは、いくつかの用語を簡略化して使用しています。

  * `<julia_root>` refers to the root directory of the Julia source tree; e.g. it should contain folders such as `base`, `deps`, `src`, `test`, etc.....
  * `<julia_root>` はジュリアソースツリーのルートディレクトリを参照します。 例えば `base`, `deps`, `src`, `test` などのフォルダを含んでいなければなりません .....
