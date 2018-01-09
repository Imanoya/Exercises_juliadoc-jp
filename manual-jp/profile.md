## Profiling
＃プロファイリング

#The `Profile` module provides tools to help developers improve the performance of their code. 
`Profile`モジュールは、開発者がコードのパフォーマンスを向上させるためのツールを提供します。
#When used, it takes measurements on running code, and produces output that helps you understand how much time is spent on individual line(s). 
使用すると、実行中のコードの測定が行われ、個々の行に費やされた時間を理解するのに役立つ出力が生成されます。
#The most common usage is to identify "bottlenecks" as targets for optimization.
最も一般的な使用方法は、最適化の目標として「ボトルネック」を特定することです。

##`Profile` implements what is known as a "sampling" or [statistical profiler](https://en.wikipedia.org/wiki/Profiling_(computer_programming)).
＃ `Profile`は「サンプリング」又は[統計的プロファイラ（https://en.wikipedia.org/wiki/Profiling_（computer_programming））として知られるものを実装しています。
#`Profile` implements what is known as a "sampling" or statistical profiler.
`Profile`は「サンプリング」または統計プロファイラとして知られているものを実装します。

# It works by periodically taking a backtrace during the execution of any task. 
 これは、タスクの実行中に定期的にバックトレースを取ることによって機能します。
#Each backtrace captures the currently-running function and line number, plus the complete chain of function calls that led to this line, and hence is a "snapshot" of the current state of execution.
各バックトレースは、現在実行中の関数と行番号、およびこの行につながった関数呼び出しの完全な連鎖をキャプチャします。したがって、現在の実行状態の「スナップショット」です。

#If much of your run time is spent executing a particular line of code, this line will show up frequently in the set of all backtraces. 
実行時間の多くが特定のコード行を実行するのに費やされた場合、この行はすべてのバックトレースのセットに頻繁に表示されます。
#In other words, the "cost" of a given line--or really, the cost of the sequence of function calls up to and including this line--is proportional to how often it appears in the set of all backtraces.
言い換えれば、ある行の "コスト" - 実際には、この行までの関数呼び出しのシーケンスのコストは、すべてのバックトレースの集合に現れる頻度に比例します。

#A sampling profiler does not provide complete line-by-line coverage, because the backtraces occur at intervals (by default, 1 ms on Unix systems and 10 ms on Windows, although the actual scheduling is subject to operating system load). 
（実際のスケジューリングは、オペレーティングシステムのロードの対象であるが、デフォルトでUnixシステム上の1つのMS Windowsでは10ミリ秒）バックトレースが間隔で発生するので、サンプリングプロファイラは、完全なラインバイラインカバレッジを提供しません。
#Moreover, as discussed further below, because samples are collected at a sparse subset of all execution points, the data collected by a sampling profiler is subject to statistical noise.
さらに、以下でさらに説明するように、サンプルはすべての実行ポイントの疎なサブセットで収集されるので、サンプリングプロファイラによって収集されたデータは統計的ノイズの影響を受ける。

#Despite these limitations, sampling profilers have substantial strengths:
これらの制限にもかかわらず、サンプリングプロファイラには大きな強みがあります。

  * 
   # You do not have to make any modifications to your code to take timing measurements (in contrast to the alternative [instrumenting profiler](https://github.com/timholy/IProfile.jl)).
    あなたはタイミング測定（代替とは対照的に（https://github.com/timholy/IProfile.jl）[プロファイラをインストルメント]）を取るためにあなたのコードに変更を加える必要はありません。
  * 
   # You do not have to make any modifications to your code to take timing measurements (in contrast to the alternative [instrumenting profiler].
    ＃タイミング計測を行うためにコードを変更する必要はありません（代わりの[計測プロファイラ]とは対照的です）。
  * 
   # It can profile into Julia's core code and even (optionally) into C and Fortran libraries.
    Juliaのコアコードにプロファイルすることができ、（オプションで）CおよびFortranライブラリにプロファイルすることもできます。
  * 
   # By running "infrequently" there is very little performance overhead; while profiling, your code can run at nearly native speed.
    「まれに」実行すると、パフォーマンス上のオーバーヘッドはほとんどありません。 プロファイリング中に、あなたのコードはほぼネイティブな速度で動くことができます。

#For these reasons, it's recommended that you try using the built-in sampling profiler before considering any alternatives.
これらの理由から、代替案を検討する前に組み込みのサンプリングプロファイラを使用することをお勧めします。

### Basic usage
##基本的な使い方

#Let's work with a simple test case:
簡単なテストケースで作業しましょう：

```julia-repl
julia> function myfunc()
           A = rand(200, 200, 400)
           maximum(A)
       end
```

#It's a good idea to first run the code you intend to profile at least once (unless you want to profile Julia's JIT-compiler):
JuliaのJITコンパイラのプロファイルを作成する場合を除き、プロファイルを作成するコードを少なくとも1回は実行することをお勧めします。

```julia-repl
julia> myfunc() # run once to force compilation
```

Now we're ready to profile this function:

```julia-repl
julia> @profile myfunc()
```

#To see the profiling results, there is a [graphical browser](https://github.com/timholy/ProfileView.jl) available, but here we'll use the text-based display that comes with the standard library:
プロファイリング結果を見るには、[グラフィカルブラウザ]（https://github.com/timholy/ProfileView.jl）がありますが、ここでは標準ライブラリに付属のテキストベースのディスプレイを使用します：
#To see the profiling results, there is a [graphical browser] available, but here we'll use the text-based display that comes with the standard library:
プロファイリング結果を見るには[グラフィカルブラウザ]がありますが、ここでは標準ライブラリに付属のテキストベースのディスプレイを使用します：

```julia-repl
julia> Profile.print()
80 ./event.jl:73; (::Base.REPL.##1#2{Base.REPL.REPLBackend})()
 80 ./REPL.jl:97; macro expansion
  80 ./REPL.jl:66; eval_user_input(::Any, ::Base.REPL.REPLBackend)
   80 ./boot.jl:235; eval(::Module, ::Any)
    80 ./<missing>:?; anonymous
     80 ./profile.jl:23; macro expansion
      52 ./REPL[1]:2; myfunc()
       38 ./random.jl:431; rand!(::MersenneTwister, ::Array{Float64,3}, ::Int64, ::Type{B...
        38 ./dSFMT.jl:84; dsfmt_fill_array_close_open!(::Base.dSFMT.DSFMT_state, ::Ptr{F...
       14 ./random.jl:278; rand
        14 ./random.jl:277; rand
         14 ./random.jl:366; rand
          14 ./random.jl:369; rand
      28 ./REPL[1]:3; myfunc()
       28 ./reduce.jl:270; _mapreduce(::Base.#identity, ::Base.#scalarmax, ::IndexLinear,...
        3  ./reduce.jl:426; mapreduce_impl(::Base.#identity, ::Base.#scalarmax, ::Array{F...
        25 ./reduce.jl:428; mapreduce_impl(::Base.#identity, ::Base.#scalarmax, ::Array{F...
```

#Each line of this display represents a particular spot (line number) in the code. 
このディスプレイの各行は、コード内の特定の地点（行番号）を表します。
#Indentation is used to indicate the nested sequence of function calls, with more-indented lines being deeper in the sequence of calls. 
インデントは、関数呼び出しのネストしたシーケンスを示すために使用されます。インデントされた行は、呼び出しシーケンス内で深くなります。
#In each line, the first "field" is the number of backtraces (samples) taken *at this line or in any functions executed by this line*.
各行において、最初の「フィールド」は、この行またはこの行によって実行される関数で*取られたバックトレース（サンプル）の数です。

#The second field is the file name and line number and the third field is the function name.
2番目のフィールドはファイル名と行番号で、3番目のフィールドは関数名です。
#Note that the specific line numbers may change as Julia's code changes; if you want to follow along, it's best to run this example yourself.
Juliaのコードが変更されると、特定の行番号が変更されることに注意してください。あなたが一緒に従っていたい場合は、この例を自分で実行するのが最善です。

#In this example, we can see that the top level function called is in the file `event.jl`. 
この例では、呼び出された最上位レベルの関数がファイル `event.jl`にあることがわかります。
#This is the function that runs the REPL when you launch Julia. 
これはJuliaを起動するときにREPLを実行する機能です。
#If you examine line 97 of `REPL.jl`, you'll see this is where the function `eval_user_input()` is called. 
`REPL.jl`の行97を調べると、これは` eval_user_input（） `という関数がどこにあるのか分かります。
#This is the function that evaluates what you type at the REPL, and since we're working interactively these functions were invoked when we entered `@profile myfunc()`. 
これはREPLで入力したものを評価する関数であり、対話的に作業しているので、これらの関数は `@profile myfunc（）`を入力したときに呼び出されます。
#The next line reflects actions taken in the [`@profile`](@ref) macro.
次の行は[`@ profile`]（@ ref）マクロで行われたアクションを反映しています。

#The first line shows that 80 backtraces were taken at line 73 of `event.jl`, but it's not that this line was "expensive" on its own: the third line reveals that all 80 of these backtraces were actually triggered inside its call to `eval_user_input`, and so on. 
最初の行は80行のバックトレースが `event.jl`の73行目で取られたことを示していますが、この行はそれ自体では"高価 "ではありません：3行目はこれらのバックトレースの80すべてが実際に`eval_user_input`などがあります。
#To find out which operations are actually taking the time, we need to look deeper in the call chain.
どの操作が実際に時間を費やしているかを知るためには、コールチェーンをより深く見る必要があります。

#The first "important" line in this output is this one:
この出力の最初の「重要な」行は、次のとおりです。

```
52 ./REPL[1]:2; myfunc()
```

#`REPL` refers to the fact that we defined `myfunc` in the REPL, rather than putting it in a file;
`REPL`とは、ファイルに入れるのではなく、REPLに` myfunc`を定義したことです。
#if we had used a file, this would show the file name. 
ファイルを使用していた場合は、ファイル名が表示されます。
#The `[1]` shows that the function `myfunc` was the first expression evaluated in this REPL session.
`[1]`は、関数 `myfunc`がこのREPLセッションで評価される最初の式であることを示しています。
#Line 2 of `myfunc()` contains the call to `rand`, and there were 52 (out of 80) backtraces that occurred 
`myfunc（）`の2行目に `rand`への呼び出しが含まれていて、発生したバックトレースは52個（うち80個）でした
#at this line. 
この行で
#Below that, you can see a call to `dsfmt_fill_array_close_open!` inside `dSFMT.jl`.
その下に、 `dSFMT.jl`の中に` dsfmt_fill_array_close_open！ 'の呼び出しを見ることができます。

#A little further down, you see:
もう少し下を見てください：

```
28 ./REPL[1]:3; myfunc()
```

Line 3 of `myfunc` contains the call to `maximum`, and there were 28 (out of 80) backtraces taken here. 
Below that, you can see the specific places in `base/reduce.jl` that carry out the time-consuming operations in the `maximum` function for this type of input data.

Overall, we can tentatively conclude that generating the random numbers is approximately twice as expensive as finding the maximum element. 
We could increase our confidence in this result by collecting more samples:

```julia-repl
julia> @profile (for i = 1:100; myfunc(); end)

julia> Profile.print()
[....]
 3821 ./REPL[1]:2; myfunc()
  3511 ./random.jl:431; rand!(::MersenneTwister, ::Array{Float64,3}, ::Int64, ::Type...
   3511 ./dSFMT.jl:84; dsfmt_fill_array_close_open!(::Base.dSFMT.DSFMT_state, ::Ptr...
  310  ./random.jl:278; rand
   [....]
 2893 ./REPL[1]:3; myfunc()
  2893 ./reduce.jl:270; _mapreduce(::Base.#identity, ::Base.#scalarmax, ::IndexLinea...
   [....]
```

In general, if you have `N` samples collected at a line, you can expect an uncertainty on the order of `sqrt(N)` (barring other sources of noise, like how busy the computer is with other tasks).
The major exception to this rule is garbage collection, which runs infrequently but tends to be quite expensive.
#(Since Julia's garbage collector is written in C, such events can be detected using the `C=true` output mode described below, or by using [ProfileView.jl](https://github.com/timholy/ProfileView.jl).)
(Since Julia's garbage collector is written in C, such events can be detected using the `C=true` output mode described below, or by using [ProfileView.jl].)

This illustrates the default "tree" dump; an alternative is the "flat" dump, which accumulates counts independent of their nesting:

```julia-repl
julia> Profile.print(format=:flat)
 Count File          Line Function
  6714 ./<missing>     -1 anonymous
  6714 ./REPL.jl       66 eval_user_input(::Any, ::Base.REPL.REPLBackend)
  6714 ./REPL.jl       97 macro expansion
  3821 ./REPL[1]        2 myfunc()
  2893 ./REPL[1]        3 myfunc()
  6714 ./REPL[7]        1 macro expansion
  6714 ./boot.jl      235 eval(::Module, ::Any)
  3511 ./dSFMT.jl      84 dsfmt_fill_array_close_open!(::Base.dSFMT.DSFMT_s...
  6714 ./event.jl      73 (::Base.REPL.##1#2{Base.REPL.REPLBackend})()
  6714 ./profile.jl    23 macro expansion
  3511 ./random.jl    431 rand!(::MersenneTwister, ::Array{Float64,3}, ::In...
   310 ./random.jl    277 rand
   310 ./random.jl    278 rand
   310 ./random.jl    366 rand
   310 ./random.jl    369 rand
  2893 ./reduce.jl    270 _mapreduce(::Base.#identity, ::Base.#scalarmax, :...
     5 ./reduce.jl    420 mapreduce_impl(::Base.#identity, ::Base.#scalarma...
   253 ./reduce.jl    426 mapreduce_impl(::Base.#identity, ::Base.#scalarma...
  2592 ./reduce.jl    428 mapreduce_impl(::Base.#identity, ::Base.#scalarma...
    43 ./reduce.jl    429 mapreduce_impl(::Base.#identity, ::Base.#scalarma...
```

#If your code has recursion, one potentially-confusing point is that a line in a "child" function can accumulate more counts than there are total backtraces. 
コードに再帰がある場合、潜在的に分かりにくい点の1つは、「子」関数の行が、合計バックトレースより多くの数を累積できることです。
#Consider the following function definitions:
以下の関数定義を考えてみましょう。

```julia
dumbsum(n::Integer) = n == 1 ? 1 : 1 + dumbsum(n-1)
dumbsum3() = dumbsum(3)
```

#If you were to profile `dumbsum3`, and a backtrace was taken while it was executing `dumbsum(1)`, the backtrace would look like this:
`dumbsum3`をプロファイリングし、` dumbsum（1） `を実行しているときにバックトレースを取った場合、バックトレースは次のようになります。

```julia
dumbsum3
    dumbsum(3)
        dumbsum(2)
            dumbsum(1)
```

#Consequently, this child function gets 3 counts, even though the parent only gets one. 
したがって、この子関数は、親が1つだけを取得しても、3つのカウントを取得します。
#The "tree" representation makes this much clearer, and for this reason (among others) is probably the most useful way to view the results.
"ツリー"表現は、これをはるかに明確にします。この理由のために、おそらく結果を見るのに最も有用な方法でしょう。

### Accumulation and clearing
##積算と決済

##Results from [`@profile`](@ref) accumulate in a buffer; if you run multiple pieces of code under
＃Results from [`@ profile`]（@ref）はバッファに蓄積されます。 複数のコードを実行している場合
##[`@profile`](@ref), then [`Profile.print()`](@ref) will show you the combined results. 
＃[`@ profile`]（@ ref）、[` Profile.print（） `]（@ ref）は結合された結果を表示します。
#Results from [`@profile`] accumulate in a buffer; if you run multiple pieces of code under [`@profile`], then [`Profile.print()`] will show you the combined results. 
[`@ profile`]の結果はバッファに蓄積されます。 [`@ profile`]の下で複数のコードを実行すると、[` Profile.print（） `]は結合された結果を表示します。

##This can be very useful, but sometimes you want to start fresh; you can do so with [`Profile.clear()`](@ref).
＃これは非常に便利ですが、時には新鮮なものにしたいこともあります。 [`Profile.clear（）`]（@ ref）を使って行うことができます。
#This can be very useful, but sometimes you want to start fresh; you can do so with [`Profile.clear()`].
これは非常に便利ですが、時には新鮮なものから始めたいと思うこともあります。 [`Profile.clear（）`]で行うことができます。

### Options for controlling the display of profile results
##プロファイル結果の表示を制御するオプション

##[`Profile.print()`](@ref) has more options than we've described so far. Let's see the full declaration:
＃[`Profile.print（）`]（@ ref）にはこれまで以上に多くのオプションがあります。 完全な宣言を見てみましょう：
#[`Profile.print()`] has more options than we've described so far. 
[`Profile.print（）`]にはこれまで説明したより多くのオプションがあります。
#Let's see the full declaration:
完全な宣言を見てみましょう：

```julia
function print(io::IO = STDOUT, data = fetch(); kwargs...)
```

Let's first discuss the two positional arguments, and later the keyword arguments:

   * 
   # `io` -- Allows you to save the results to a buffer, e.g. a file, but the default is to print to `STDOUT` (the console).
   `io` - 結果をバッファに保存することができます。 ファイルですが、デフォルトでは `STDOUT`（コンソール）に出力されます。
    *
   * 
   # `data` -- Contains the data you want to analyze; by default that is obtained from [`Profile.fetch()`](@ref), which pulls out the backtraces from a pre-allocated buffer. For example, if you want to profile the profiler, you could say:
   `data` - 分析したいデータが入っています。 デフォルトでは、[`Profile.fetch（）`]（@ ref）から得られます。これは、事前に割り当てられたバッファからバックトレースを引き出します。 たとえば、プロファイラのプロファイルを作成する場合は、次のように指定します。

    ```julia
    data = copy(Profile.fetch())
    Profile.clear()
    @profile Profile.print(STDOUT, data) # Prints the previous results
    Profile.print()                      # Prints results from Profile.print()
    ```

The keyword arguments can be any combination of:

   * 
   #`format` -- Introduced above, determines whether backtraces are printed with (default, `:tree`) or without (`:flat`) indentation indicating tree structure.
    `format` - 上に紹介したように、バックトレースがツリー構造を示すインデント表示（デフォルト、`：tree`）または表示なし（ `：flat`）で印刷されるかどうかを決定します。
   * 
   # `C` -- If `true`, backtraces from C and Fortran code are shown (normally they are excluded).
    `C` - ` true 'の場合、CとFortranコードのバックトレースが表示されます（通常は除外されます）。
   # Try running the introductory example with `Profile.print(C = true)`.
    導入例を `Profile.print（C = true）`で実行してみてください。
   # This can be extremely helpful in deciding whether it's Julia code or C code that is causing a bottleneck; setting `C = true` also improves the interpretability of the nesting, at the cost of longer profile dumps.
    これは、ボトルネックの原因となっているJuliaコードかCコードかを判断するのに非常に役立ちます。 `C = true`を設定すると、より長いプロファイルダンプを犠牲にして、ネストの解釈可能性が向上します。
   * 
   # `combine` -- Some lines of code contain multiple operations; for example, `s += A[i]` contains both an array reference (`A[i]`) and a sum operation. 
    `combine` - コードの行には複数の操作が含まれています。 例えば ​​`s + = A [i]`は配列参照（ `A [i]`）と合計演算の両方を含んでいます。
   # These correspond to different lines in the generated machine code, and hence there may be two or more different addresses captured during backtraces on this line.
    これらは、生成されたマシンコードの異なる行に対応しているため、この行のバックトレース中に2つ以上の異なるアドレスが取り込まれることがあります。
   # `combine = true` lumps them together, and is probably what you typically want, but you can generate an output separately for each unique instruction pointer with `combine = false`.
    `combine = true`はそれらを一緒に塊にします。あなたが通常望むものですが、` combine = false`を使ってそれぞれの一意の命令ポインタのために別々に出力を生成することができます。
   * 
   # `maxdepth` -- Limits frames at a depth higher than `maxdepth` in the `:tree` format.
   `maxdepth` - `：tree`フォーマットの `maxdepth`より深いフレームを制限します。
   * 
   # `sortedby` -- Controls the order in `:flat` format. `:filefuncline` (default) sorts by the source line, whereas `:count` sorts in order of number of collected samples.
   `sortedby` - `：flat`フォーマットでの順序を制御します。 `：filefuncline`（デフォルト）はソース行でソートしますが、`：count`は収集されたサンプル数の順にソートします。
   * 
   # `noisefloor` -- Limits frames that are below the heuristic noise floor of the sample (only applies to format `:tree`).
   `noisefloor` - サンプルのヒューリスティックなノイズフロアを下回るフレームを制限します（`：tree`のフォーマットにのみ適用されます）。
   # A suggested value to try for this is 2.0 (the default is 0). This parameter hides samples for which `n <= noisefloor * √N`, where `n` is the number of samples on this line, and `N` is the number of samples for the callee.
   これを試すための推奨値は2.0です（デフォルトは0）。このパラメータは、 `n <= noisefloor *√N 'のサンプルを隠します。ここで` n`はこの行のサンプル数、 `N`は呼び出し先のサンプル数です。
   * 
   #`mincount` -- Limits frames with less than `mincount` occurrences.
    `mincount` - ` mincount`オカレンス未満のフレームを制限します。

#File/function names are sometimes truncated (with `...`), and indentation is truncated with a `+n` at the beginning, where `n` is the number of extra spaces that would have been inserted, had there been room.
ファイル/関数名は時々切り捨てられ（ `...`で）、字下げは最初に `+ n 'で切り捨てられ、` n`は挿入された余分なスペースの数です。 。
#If you want a complete profile of deeply-nested code, often a good idea is to save to a file using a wide `displaysize` in an [`IOContext`](@ref):
深くネストされたコードの完全なプロファイルが必要な場合は、[IOContext`]（@ ref）に広い `displaysize`を使ってファイルに保存することをお勧めします。

```julia
open("/tmp/prof.txt", "w") do s
    Profile.print(IOContext(s, :displaysize => (24, 500)))
end
```

### Configuration
##設定

##[`@profile`](@ref) just accumulates backtraces, and the analysis happens when you call [`Profile.print()`](@ref).
＃[`@ profile`]（@ref）はバックトレースを累積し、[` Profile.print（） `]（@ ref）を呼び出すと解析が行われます。
#[`@profile`] just accumulates backtraces, and the analysis happens when you call [`Profile.print()`].
[`@ profile`]はバックトレースを累積し、[` Profile.print（） `]を呼び出すと解析が行われます。
#For a long-running computation, it's entirely possible that the pre-allocated buffer for storing backtraces will be filled. 
長期間の計算では、バックトレースを格納するために事前に割り当てられたバッファがいっぱいになる可能性があります。
#If that happens, the backtraces stop but your computation continues.
その場合、バックトレースは停止しますが、計算は続行されます。
#As a consequence, you may miss some important profiling data (you will get a warning when that happens).
結果として、重要なプロファイリングデータが欠落する可能性があります（警告が表示されます）。

#You can obtain and configure the relevant parameters this way:
このように関連するパラメータを取得して設定することができます。

```julia
Profile.init() # returns the current settings
Profile.init(n = 10^7, delay = 0.01)
```

#`n` is the total number of instruction pointers you can store, with a default value of `10^6`.
`n`はストア可能な命令ポインタの総数で、デフォルト値は` 10 ^ 6 'です。
#If your typical backtrace is 20 instruction pointers, then you can collect 50000 backtraces, which suggests a statistical uncertainty of less than 1%.
典型的なバックトレースが20命令ポインターであれば、50000回のバックトレースを収集することができ、1％未満の統計的不確実性を示唆しています。
#This may be good enough for most applications.
これは、ほとんどのアプリケーションで十分です。

#Consequently, you are more likely to need to modify `delay`, expressed in seconds, which sets the amount of time that Julia gets between snapshots to perform the requested computations. 
その結果、ジュリアが要求された計算を実行するためにスナップショットの間に取得する時間を設定する秒単位で表現される「遅延」を変更する必要があります。
#A very long-running job might not need frequent backtraces. 
非常に長時間の仕事は頻繁なバックトレースを必要としないかもしれません。
#The default setting is `delay = 0.001`.
デフォルト設定は `delay = 0.001`です。
#Of course, you can decrease the delay as well as increase it; however, the overhead of profiling grows once the delay becomes similar to the amount of time needed to take a backtrace (~30 microseconds on the author's laptop).
もちろん、遅延を減らしたり、遅延を増やしたりすることもできます。 しかし、遅れがバックトレースを取るのに必要な時間（著者のラップトップで〜30マイクロ秒）に似ていれば、プロファイリングのオーバーヘッドが大きくなります。

## Memory allocation analysis
＃メモリ割り当ての分析

#One of the most common techniques to improve performance is to reduce memory allocation.
パフォーマンスを向上させる最も一般的な手法の1つは、メモリ割り当てを減らすことです。
#The total amount of allocation can be measured with [`@time`](@ref) and [`@allocated`](@ref), and specific lines triggering allocation can often be inferred from profiling via the cost of garbage collection that these lines incur. 
割り当ての合計量は[ `@のtime`]（@ REF）で測定し、[` @ allocated`]（@ REF）、および割り当てをトリガする特定の株は、多くの場合、これらのことは、ガベージコレクションのコストを介してプロファイリングから推論することができることができますラインが発生する。

#However, sometimes it is more efficient to directly measure the amount of memory allocated by each line of code.
ただし、コードの各行によって割り当てられるメモリの量を直接測定する方が効率的な場合もあります。

#To measure allocation line-by-line, start Julia with the `--track-allocation=<setting>` command-line option, for which you can choose `none` (the default, do not measure allocation), `user` (measure memory allocation everywhere except Julia's core code), or `all` (measure memory allocation at each line of Julia code).
`、user`を、割り当てライン・バイ・ラインを測定します（割り当てを測定しないデフォルト、）` NONE`を選択できるため、 `--track割り当て= <設定>`コマンドラインオプション、とジュリアを開始するには（Juliaのコアコードを除いてどこでもメモリ割り当てを測定）、または「すべて」（ジュリアコードの各行でのメモリ割り当てを測定）
#Allocation gets measured for each line of compiled code.
割り当ては、コンパイルされたコードの各行ごとに測定されます。
#When you quit Julia, the cumulative results are written to text files with `.mem` appended after the file name, residing in the same directory as the source file. 
Juliaを終了すると、累積結果はソースファイルと同じディレクトリにあるファイル名の後に `.mem`が追加されたテキストファイルに書き込まれます。
#Each line lists the total number of bytes allocated. 
各行には、割り当てられた合計バイト数が表示されます。
#The [`Coverage` package](https://github.com/JuliaCI/Coverage.jl) contains some elementary analysis tools, for example to sort the lines in order of number of bytes allocated.
[ `Coverage`パッケージ（https://github.com/JuliaCI/Coverage.jl）が割り当てられたバイト数の順に行をソートするために、例えば、いくつかの元素分析ツールを含んでいます。

#In interpreting the results, there are a few important details. 
結果を解釈する際には、いくつかの重要な詳細があります。
#Under the `user` setting, the first line of any function directly called from the REPL will exhibit allocation due to events that happen in the REPL code itself. 
`user`設定の下では、REPLから直接呼び出される関数の最初の行は、REPLコード自体で発生するイベントのために割り当てを示します。
#More significantly, JIT-compilation also adds to allocation counts, because much of Julia's compiler is written in Julia (and compilation usually requires memory allocation). 
Juliaのコンパイラの多くはJuliaで書かれており（通常はコンパイルにはメモリ割り当てが必要です）、JITコンパイルでは割り当てカウントが増えます。
#The recommended procedure is to force compilation by executing all the commands you want to analyze, then call [`Profile.clear_malloc_data()`](@ref) to reset all allocation counters.
推奨される手順は、分析するすべてのコマンドを実行してコンパイルを強制的に実行し、[`Profile.clear_malloc_data（）`]（@ ref）を呼び出してすべての割り当てカウンタをリセットすることです。
# Finally, execute the desired commands and quit Julia to trigger the generation of the `.mem` files.
最後に、必要なコマンドを実行し、Juliaを終了して `.mem`ファイルの生成をトリガーします。
