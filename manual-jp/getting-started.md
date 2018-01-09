
<!-- > Getting Started -->
＃ 入門

<!-- Julia installation is straightforward, whether using precompiled binaries or compiling from source. -->
ジュリアのインストールは、プリコンパイルされたバイナリを使用する場合でも、ソースからコンパイルする場合も、簡単です。
<!-- Download and install Julia by following the instructions at [https://julialang.org/downloads/](https://julialang.org/downloads/). -->
ジュリアをダウンロードしてインストールするには、[https://julialang.org/downloads/](https://julialang.org/downloads/)]の指示に従ってください。

<!-- The easiest way to learn and experiment with Julia is by starting an interactive session (also known as a read-eval-print loop or "repl") by double-clicking the Julia executable or running `julia` from the command line: -->
Juliaを学んだり体験する簡単な方法は、Juliaの実行可能ファイルをダブルクリックするか、コマンドラインから `julia`を実行し、対話型セッション(read-eval-printループまたは"repl"とも呼ばれます)を開始することです。

```
$ julia
               _
   _       _ _(_)_     |  A fresh approach to technical computing
  (_)     | (_) (_)    |  Documentation: https://docs.julialang.org
   _ _   _| |_  __ _   |  Type "?help" for help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 0.5.0-dev+2440 (2016-02-01 02:22 UTC)
 _/ |\__'_|_|_|\__'_|  |  Commit 2bb94d6 (11 days old master)
|__/                   |  x86_64-apple-darwin13.1.0

julia> 1 + 2
3

julia> ans
3
```

<!-- To exit the interactive session, type `^D` -- the control key together with the `d` key or type `quit()`.  -->
対話セッションを終了するには、`^D` -Ctrlキーと`d`キーを一緒に入力するか、`quit()`と打ちます。
<!-- When run in interactive mode, `julia` displays a banner and prompts the user for input.  -->
対話モードで実行すると、 `julia`はバナーを表示し、ユーザーに入力を促します。
<!-- Once the user has entered a complete expression, such as `1 + 2`, and hits enter, the interactive session evaluates the expression and shows its value.  -->
ユーザーが `1 + 2`のような完全な式を入力してenterを押すと、対話セッションは式を評価し、その値を表示します。
<!-- If an expression is entered into an interactive session with a trailing semicolon, its value is not shown.  -->
式が後続のセミコロンとの対話セッションに入力された場合、その値は表示されません。
<!-- The variable `ans` is bound to the value of the last evaluated expression whether it is shown or not.  -->
変数 `ans`は、最後に評価された式の値にバインドされます。
<!-- The `ans` variable is only bound in interactive sessions, not when Julia code is run in other ways. -->
`ans`変数はインタラクティブセッションでのみバインドされ、Juliaコードが他の方法で実行されるときにはバインドされません。

<!-- To evaluate expressions written in a source file `file.jl`, write `include("file.jl")`. -->
ソースファイル `file.jl`に書かれた式を評価するには、`include("file.jl")`と書いてください。

<!-- To run code in a file non-interactively, you can give it as the first argument to the `julia` command: -->
非対話的にファイル内のコードを実行するには、 `julia`コマンドの最初の引数として与えることができます：

```
$ julia script.jl arg1 arg2...
```

<!-- As the example implies, the following command-line arguments to `julia` are taken as command-line arguments to the program `script.jl`, passed in the global constant `ARGS`.  -->
例が示すように、`julia`に対する以下のコマンドライン引数は、プログラム`script.jl`に対するコマンドライン引数として取られ、グローバル定数 `ARGS`に渡されます。
<!-- The name of the script itself is passed in as the global `PROGRAM_FILE`.  -->
スクリプト自体の名前はグローバル変数`PROGRAM_FILE`として渡されます。
<!-- Note that `ARGS` is also set when script code is given using the `-e` option on the command line (see the `julia` help output below) but `PROGRAM_FILE` will be empty.  -->
コマンドラインで `-e`オプションを使ってスクリプトコードを与えると(`julia`ヘルプの出力を参照)、`PROGRAM_FILE`は空になります。
<!-- For example, to just print the arguments given to a script, you could do this: -->
たとえば、スクリプトに与えられた引数を表示するには、次のようにします。

```
$ julia -e 'println(PROGRAM_FILE); for x in ARGS; println(x); end' foo bar

foo
bar
```

<!-- Or you could put that code into a script and run it: -->
または、そのコードをスクリプトに入れて実行することもできます。

```
$ echo 'println(PROGRAM_FILE); for x in ARGS; println(x); end' > script.jl
$ julia script.jl foo bar
script.jl
foo
bar
```

<!-- The `--` delimiter can be used to separate command-line args to the scriptfile from args to Julia: -->
`--`デリミタを使ってコマンドライン引数をスクリプトファイルとargiaからJuliaに分けることができます：

```
$ julia --color=yes -O -- foo.jl arg1 arg2..
```

<!-- Julia can be started in parallel mode with either the `-p` or the `--machinefile` options.  -->
`-p`または` --machinefile`オプションのいずれかを使い、Juliaをパラレルモードで起動することができます。
<!-- `-p n` will launch an additional `n` worker processes, while `--machinefile file` will launch a worker for each line in file `file`.  -->
`-p n`はワーカープロセスを`n`追加して起動し、`--machinefile file`はファイル` file`の各行に対してワーカを起動します。
<!-- The machines defined in `file` must be accessible via a passwordless `ssh` login, with Julia installed at the same location as the current host.  -->
`file`で定義されたマシンは、現在のホストと同じ場所にJuliaがインストールされた、パスワードなしの`ssh`ログインを介してアクセス可能でなければなりません。
<!-- Each machine definition takes the form `[count*][user@]host[:port] [bind_addr[:port]]` .  -->
各マシン定義は `[count*][user@]host[:port][bind_addr[:port]]`の形式を取ります。
<!-- `user` defaults to current user, `port` to the standard ssh port.  -->
`user`のデフォルトは現在のユーザ、`port`は標準のsshポートです。
<!-- `count` is the number of workers to spawn on the node, and defaults to 1.  -->
`count`はノードに生成するワーカーの数であり、デフォルトは1です。
<!-- The optional `bind-to bind_addr[:port]` specifies the ip-address and port that other workers should use to connect to this worker. -->
オプションの `bind-to bind_addr[:port]`は、他のワーカーがこのワーカーに接続するために使用するIPアドレスとポートを指定します。

<!-- If you have code that you want executed whenever Julia is run, you can put it in `~/.juliarc.jl`: -->
Juliaを実行するたびに実行するコードがあれば、`~/.juliarc.jl`に置くことができます：

```
$ echo 'println("Greetings! 你好! 안녕하세요?")' > ~/.juliarc.jl
$ julia
Greetings! 你好! 안녕하세요?

...
```

<!-- There are various ways to run Julia code and provide options, similar to those available for the `perl` and `ruby` programs: -->
`perl`や`ruby`と同じように、オプションを与えてJuliaコードを実行する方法はいろいろあります：

```
julia [switches] -- [programfile] [args...]
 -v, --version             Display version information
 -h, --help                Print this message

 -J, --sysimage <file>     Start up with the given system image file
 --precompiled={yes|no}    Use precompiled code from system image if available
 --compilecache={yes|no}   Enable/disable incremental precompilation of modules
 -H, --home <dir>          Set location of `julia` executable
 --startup-file={yes|no}   Load ~/.juliarc.jl
 --handle-signals={yes|no} Enable or disable Julia's default signal handlers

 -e, --eval <expr>         Evaluate <expr>
 -E, --print <expr>        Evaluate and show <expr>
 -L, --load <file>         Load <file> immediately on all processors

 -p, --procs {N|auto}      Integer value N launches N additional local worker processes
                           "auto" launches as many workers as the number of local cores
 --machinefile <file>      Run processes on hosts listed in <file>

 -i                        Interactive mode; REPL runs and isinteractive() is true
 -q, --quiet               Quiet startup (no banner)
 --color={yes|no}          Enable or disable color text
 --history-file={yes|no}   Load or save history

 --compile={yes|no|all|min}Enable or disable JIT compiler, or request exhaustive compilation
 -C, --cpu-target <target> Limit usage of cpu features up to <target>
 -O, --optimize={0,1,2,3}  Set the optimization level (default is 2 if unspecified or 3 if specified as -O)
 -g, -g <level>            Enable / Set the level of debug info generation (default is 1 if unspecified or 2 if specified as -g)
 --inline={yes|no}         Control whether inlining is permitted (overrides functions declared as @inline)
 --check-bounds={yes|no}   Emit bounds checks always or never (ignoring declarations)
 --math-mode={ieee,fast}   Disallow or enable unsafe floating point optimizations (overrides @fastmath declaration)

 --depwarn={yes|no|error}  Enable or disable syntax and method deprecation warnings ("error" turns warnings into errors)

 --output-o name           Generate an object file (including system image data)
 --output-ji name          Generate a system image data file (.ji)
 --output-bc name          Generate LLVM bitcode (.bc)
 --output-incremental=no   Generate an incremental output file (rather than complete)

 --code-coverage={none|user|all}, --code-coverage
                           Count executions of source lines (omitting setting is equivalent to "user")
 --track-allocation={none|user|all}, --track-allocation
                           Count bytes allocated by each source line
```

<!-- # Resources -->
##リソース

<!-- In addition to this manual, there are various other resources that may help new users get started with Julia: -->
このマニュアルに加えて、新規ユーザーがJuliaを使い始めるのに役立つさまざまなリソースがあります。

  * [Julia and IJulia cheatsheet](http://math.mit.edu/~stevenj/Julia-cheatsheet.pdf)
  * [Learn Julia in a few minutes](https://learnxinyminutes.com/docs/julia/)
  * [Learn Julia the Hard Way](https://github.com/chrisvoncsefalvay/learn-julia-the-hard-way)
  * [Julia by Example](http://samuelcolvin.github.io/JuliaByExample/)
  * [Hands-on Julia](https://github.com/dpsanders/hands_on_julia)
  * [Tutorial for Homer Reid's numerical analysis class](http://homerreid.dyndns.org/teaching/18.330/JuliaProgramming.shtml)
  * [An introductory presentation](https://raw.githubusercontent.com/ViralBShah/julia-presentations/master/Fifth-Elephant-2013/Fifth-Elephant-2013.pdf)
  * [Videos from the Julia tutorial at MIT](https://julialang.org/blog/2013/03/julia-tutorial-MIT)
  * [YouTube videos from the JuliaCons](https://www.youtube.com/user/JuliaLanguage/playlists)
