<!-- Start -->

# Running External Programs

># 外部プログラムの実行

<!-- End -->
<!-- Start -->
Julia borrows backtick notation for commands from the shell, Perl, and Ruby. However, in Julia, writing
> Juliaは、シェル、Perl、Rubyのコマンドのバックティック表記法を借りています。 しかし、ジュリアでは、
<!-- End -->

```jldoctest
julia> `echo hello`
`echo hello`
```

<!-- Start -->
differs in several aspects from the behavior in various shells, Perl, or Ruby:
> さまざまなシェル、Perl、またはRubyの動作とはいくつかの面で異なります。
<!-- End -->

<!-- Start -->
* Instead of immediately running the command, backticks create a `Cmd` object to represent the command. 
* > コマンドを直ちに実行する代わりに、バッククォートはコマンドを表す `Cmd`オブジェクトを作成します。
<!-- End -->
<!-- Start -->
* You can use this object to connect the command to others via pipes, run it, and read or write to it.
* > このオブジェクトを使用すると、パイプを介して他の人にコマンドを接続し、実行し、読み書きできます。
<!-- End -->
<!-- Start -->
*  When the command is run, Julia does not capture its output unless you specifically arrange for it to. 
* > コマンドが実行されると、Juliaは特別な手配をしない限り、出力をキャプチャしません。
<!-- End -->
<!-- Start -->
*  Instead, the output of the command by default goes to [`STDOUT`](@ref) as it would using `libc`'s `system` call.
* > その代わりに、コマンドの出力は `libc`の` system`呼び出しを使うのと同じように、デフォルトで[`STDOUT`]（@ref）に行きます。
<!-- End -->
<!-- Start -->
*  The command is never run with a shell. Instead, Julia parses the command syntax directly, appropriately interpolating variables and splitting on words as the shell would, respecting shell quoting syntax.
* > コマンドは決してシェルで実行されません。 代わりに、Juliaはコマンド構文を直接解析し、変数を適切に補間し、シェルのように単語を分割して、シェルの引用構文を尊重します。
<!-- End -->
<!-- Start -->
* The command is run as `julia`'s immediate child process, using `fork` and `exec` calls.
* > コマンドは、 `julia`の直接の子プロセスとして、` fork`と `exec`呼び出しを使って実行されます。
<!-- End -->

<!-- Start -->
Here's a simple example of running an external program:
> 以下に、外部プログラムを実行する簡単な例を示します。
<!-- End -->

```jldoctest
julia> mycommand = `echo hello`
`echo hello`

julia> typeof(mycommand)
Cmd

julia> run(mycommand)
hello
```

<!-- Start -->
The `hello` is the output of the `echo` command, sent to [`STDOUT`](@ref). 
> `hello`は` echo`コマンドの出力で、[`STDOUT`]（@ ref）に送られます。
<!-- End -->
<!-- Start -->
The run method itself returns `nothing`, and throws an [`ErrorException`](@ref) if the external command fails to run successfully.
> runメソッド自体は `nothing`を返し、外部コマンドが正常に実行されなかった場合は[` ErrorException`]（@ref）をスローします。
<!-- End -->

<!-- Start -->
If you want to read the output of the external command, [`readstring()`](@ref) can be used instead:
>  外部コマンドの出力を読みたい場合は、代わりに[`readstring（）`]（@ ref）を使うことができます：
<!-- End -->

```jldoctest
julia> a = readstring(`echo hello`)
"hello\n"

julia> chomp(a) == "hello"
true
```

<!-- Start -->
More generally, you can use [`open()`](@ref) to read from or write to an external command.
> より一般的には、[`open（）`]（@ ref）を使って外部コマンドを読み書きすることができます。
<!-- End -->

```jldoctest
julia> open(`less`, "w", STDOUT) do io
           for i = 1:3
               println(io, i)
           end
       end
1
2
3
```

<!-- Start -->
The program name and the individual arguments in a command can be accessed and iterated over as if the command were an array of strings:
> コマンド内のプログラム名と個々の引数は、コマンドが文字列の配列であるかのようにアクセスおよび反復することができます。
<!-- End -->

```jldoctest
julia> collect(`echo "foo bar"`)
2-element Array{String,1}:
 "echo"
 "foo bar"

julia> `echo "foo bar"`[2]
"foo bar"
```

<!-- Start -->

### [Interpolation](@id command-interpolation)

> ### [補間]（@ idコマンド補間）

<!-- End -->
<!-- Start -->
Suppose you want to do something a bit more complicated and use the name of a file in the variable `file` as an argument to a command. 
> もう少し複雑なやり方をしたいとし、 `file`変数のファイル名をコマンドの引数として使用したいとします。
<!-- End -->
<!-- Start -->
You can use `$` for interpolation much as you would in a string literal (see [Strings](@ref)):
> 文字列リテラルと同様に補間に `$`を使うことができます（[Strings]（@ ref）参照）：
<!-- End -->

```jldoctest
julia> file = "/etc/passwd"
"/etc/passwd"

julia> `sort $file`
`sort /etc/passwd`
```

<!-- Start -->
A common pitfall when running external programs via a shell is that if a file name contains characters that are special to the shell, they may cause undesirable behavior. 
> シェルを介して外部プログラムを実行するときの一般的な落とし穴は、ファイル名にシェルにとって特別な文字が含まれていると、望ましくない動作を引き起こす可能性があるということです。
<!-- End -->
<!-- Start -->
Suppose, for example, rather than `/etc/passwd`, we wanted to sort the contents of the file `/Volumes/External HD/data.csv`.
> たとえば、 `/etc/passwd` ではなく、`/Volumes/External HD/data.csv` ファイルの内容をソートしたいとします。
<!-- End -->
<!-- Start -->
Let's try it:
> 試してみよう：
<!-- End -->

```jldoctest
julia> file = "/Volumes/External HD/data.csv"
"/Volumes/External HD/data.csv"

julia> `sort $file`
`sort '/Volumes/External HD/data.csv'`
```

<!-- Start -->
How did the file name get quoted? Julia knows that `file` is meant to be interpolated as a single argument, so it quotes the word for you. 
> ファイル名はどのように引用されましたか？ Juliaは、 `file`は単一の引数として補間されることを意味するので、あなたのために単語を引用します。
<!-- End -->
<!-- Start -->
Actually, that is not quite accurate: the value of `file` is never interpreted by a shell, so there's no need for actual quoting; the quotes are inserted only for presentation to the user. 
> 実際には、それはかなり正確ではありません。 `file`の値は決してシェルによって解釈されないので、実際の引用をする必要はありません。 引用符は、ユーザーに提示するためにのみ挿入されます。
<!-- End -->
<!-- Start -->
This will even work if you interpolate a value as part of a shell word:
> これは、シェル単語の一部として値を補間する場合でも機能します：
<!-- End -->

```jldoctest
julia> path = "/Volumes/External HD"
"/Volumes/External HD"

julia> name = "data"
"data"

julia> ext = "csv"
"csv"

julia> `sort $path/$name.$ext`
`sort '/Volumes/External HD/data.csv'`
```

<!-- Start -->
As you can see, the space in the `path` variable is appropriately escaped. 
> ご覧のように、 `path`変数のスペースは適切にエスケープされます。
<!-- End -->
<!-- Start -->
But what if you *want* to interpolate multiple words? 
> しかし、複数の言葉を補間したいと思ったら？
<!-- End -->
<!-- Start -->
In that case, just use an array (or any other iterable container):
> その場合は、配列（または他の反復可能なコンテナ）を使用してください：
<!-- End -->

```jldoctest
julia> files = ["/etc/passwd","/Volumes/External HD/data.csv"]
2-element Array{String,1}:
 "/etc/passwd"
 "/Volumes/External HD/data.csv"

julia> `grep foo $files`
`grep foo /etc/passwd '/Volumes/External HD/data.csv'`
```

<!-- Start -->
If you interpolate an array as part of a shell word, Julia emulates the shell's `{a,b,c}` argument generation:
> 配列をシェルワードの一部として補間すると、Juliaはシェルの `{a、b、c}`引数の生成をエミュレートします：
<!-- End -->

```jldoctest
julia> names = ["foo","bar","baz"]
3-element Array{String,1}:
 "foo"
 "bar"
 "baz"

julia> `grep xylophone $names.txt`
`grep xylophone foo.txt bar.txt baz.txt`
```

<!-- Start -->
Moreover, if you interpolate multiple arrays into the same word, the shell's Cartesian product generation behavior is emulated:
> さらに、複数の配列を同じ単語に補間すると、シェルのデカルト積生成動作はエミュレートされます。
<!-- End -->

```jldoctest
julia> names = ["foo","bar","baz"]
3-element Array{String,1}:
 "foo"
 "bar"
 "baz"

julia> exts = ["aux","log"]
2-element Array{String,1}:
 "aux"
 "log"

julia> `rm -f $names.$exts`
`rm -f foo.aux foo.log bar.aux bar.log baz.aux baz.log`
```

<!-- Start -->
Since you can interpolate literal arrays, you can use this generative functionality without needing to create temporary array objects first:
> リテラル配列を補間することができるので、一時的な配列オブジェクトを最初に作成しなくても、この生成関数を使用できます。
<!-- End -->

```jldoctest
julia> `rm -rf $["foo","bar","baz","qux"].$["aux","log","pdf"]`
`rm -rf foo.aux foo.log foo.pdf bar.aux bar.log bar.pdf baz.aux baz.log baz.pdf qux.aux qux.log qux.pdf`
```

<!-- Start -->

### Quoting

> ### 引用する

<!-- End -->
<!-- Start -->
Inevitably, one wants to write commands that aren't quite so simple, and it becomes necessary to use quotes. 
> 必然的に、あまり単純ではないコマンドを書くことが望まれ、引用符を使用する必要があります。
<!-- End -->
<!-- Start -->
Here's a simple example of a Perl one-liner at a shell prompt:
> シェルプロンプトでPerlの1つのライナーの簡単な例を次に示します。
<!-- End -->

```
sh$ perl -le '$|=1; for (0..3) { print }'
0
1
2
3
```

<!-- Start -->
The Perl expression needs to be in single quotes for two reasons: so that spaces don't break the expression into multiple shell words, and so that uses of Perl variables like `$|` (yes, that's the name of a variable in Perl), don't cause interpolation. 
> Perl式は二つの理由から一重引用符で囲む必要があります：スペースが式を複数のシェル単語に分割しないように、そして `$ |`のようなPerl変数を使うように（そう、Perlの変数の名前です ）、補間を行わない。
<!-- End -->
<!-- Start -->
In other instances, you may want to use double quotes so that interpolation *does* occur:
> 他の例では、補間*が行われるように二重引用符を使用することができます：
<!-- End -->

```
sh$ first="A"
sh$ second="B"
sh$ perl -le '$|=1; print for @ARGV' "1: $first" "2: $second"
1: A
2: B
```

<!-- Start -->
In general, the Julia backtick syntax is carefully designed so that you can just cut-and-paste shell commands as is into backticks and they will work: the escaping, quoting, and interpolation behaviors are the same as the shell's. 
> 一般的に、Juliaバックティックの構文は慎重に設計されているので、シェルコマンドをそのままバッククォートにカット＆ペーストすることができます。エスケープ、クォート、補間の動作はシェルの動作と同じです。
<!-- End -->
<!-- Start -->
#The only difference is that the interpolation is integrated and aware of Julia's notion of what is a single string value, and what is a container for multiple values. Let's try the above two examples in Julia:
> 唯一の違いは、補間が統合されており、Juliaの単一の文字列値の概念と複数の値のコンテナの概念を認識していることです。 ジュリアで上記の2つの例を試してみましょう：
<!-- End -->

```jldoctest
julia> A = `perl -le '$|=1; for (0..3) { print }'`
`perl -le '$|=1; for (0..3) { print }'`

julia> run(A)
0
1
2
3

julia> first = "A"; second = "B";

julia> B = `perl -le 'print for @ARGV' "1: $first" "2: $second"`
`perl -le 'print for @ARGV' '1: A' '2: B'`

julia> run(B)
1: A
2: B
```

<!-- Start -->
The results are identical, and Julia's interpolation behavior mimics the shell's with some improvements due to the fact that Julia supports first-class iterable objects while most shells use strings split on spaces for this, which introduces ambiguities. 
> 結果は同一であり、Juliaの補間動作は、ジュリアが一流の反復可能オブジェクトをサポートしているために、いくつかの改良を加えてシェルを模倣しています。
<!-- End -->
<!-- Start -->
When trying to port shell commands to Julia, try cut and pasting first. 
> シェルコマンドをJuliaに移植しようとするときは、最初に切り取って貼り付けてみてください。
<!-- End -->
<!-- Start -->
Since Julia shows commands to you before running them, you can easily and safely just examine its interpretation without doing any damage.
> Juliaは実行前にコマンドを表示しているので、簡単に、そして安全に、解釈を破損することなく調べることができます。
<!-- End -->
<!-- Start -->

### Pipelines

> ### パイプライン

<!-- End -->
<!-- Start -->
Shell metacharacters, such as `|`, `&`, and `>`, need to be quoted (or escaped) inside of Julia's backticks:
> `|`, `&`, `>` のようなシェルのメタキャラクタは、Juliaのバッククォート内でクォート（またはエスケープ）する必要があります：
<!-- End -->

```jldoctest
julia> run(`echo hello '|' sort`)
hello | sort

julia> run(`echo hello \| sort`)
hello | sort
```

<!-- Start -->
This expression invokes the `echo` command with three words as arguments: `hello`, `|`, and `sort`. The result is that a single line is printed: `hello | sort`. 
> この式は `hello`、` | `、` sort`の3つの単語を引数として `echo`コマンドを呼び出します。 その結果、1行が出力されます： `hello | ソート `。
<!-- End -->
<!-- Start -->
How, then, does one construct a pipeline? Instead of using `'|'` inside of backticks, one uses [`pipeline()`](@ref):
> どのようにしてパイプラインを構築するのですか？ backticksの中で `` | '`を使うのではなく、[` pipeline（） `]（@ ref）を使います：
<!-- End -->

```jldoctest
julia> run(pipeline(`echo hello`, `sort`))
hello
```

<!-- Start -->
This pipes the output of the `echo` command to the `sort` command. 
> これは `echo`コマンドの出力を` sort`コマンドにパイプします。
<!-- End -->
<!-- Start -->
Of course, this isn't terribly interesting since there's only one line to sort, but we can certainly do much more interesting things:
> もちろん、並べ替える行が1つしかないので、これはあまり面白いことではありませんが、もっと面白いことをすることができます。
<!-- End -->

```julia-repl
julia> run(pipeline(`cut -d: -f3 /etc/passwd`, `sort -n`, `tail -n5`))
210
211
212
213
214
```

<!-- Start -->
This prints the highest five user IDs on a UNIX system. 
> これにより、UNIXシステム上で最高5つのユーザーIDが印刷されます。
<!-- End -->
<!-- Start -->
The `cut`, `sort` and `tail` commands are all spawned as immediate children of the current `julia` process, with no intervening shell process. 
> `cut`、` sort`および `tail`コマンドはすべて、現在の` julia`プロセスの直接の子として生成され、介在するシェルプロセスはありません。
<!-- End -->
<!-- Start -->
Julia itself does the work to setup pipes and connect file descriptors that is normally done by the shell. 
> Juliaはパイプをセットアップし、通常はシェルが行うファイル記述子を接続する作業を行います。
<!-- End -->
<!-- Start -->
Since Julia does this itself, it retains better control and can do some things that shells cannot.
> Juliaはこれ自体を行うので、より良い制御を保持し、シェルができないことをいくつか実行できます。
<!-- End -->

<!-- Start -->
Julia can run multiple commands in parallel:
> Juliaは複数のコマンドを並行して実行できます。
<!-- End -->

```julia-repl
julia> run(`echo hello` & `echo world`)
world
hello
```

<!-- Start -->
The order of the output here is non-deterministic because the two `echo` processes are started nearly simultaneously, and race to make the first write to the [`STDOUT`](@ref) descriptor they share with each other and the `julia` parent process. 
> ここでの出力の順序は、2つの `echo`プロセスがほぼ同時に開始され、互いが共有する `` STDOUT`（@ ref）記述子への最初の書き込みを競争し、 `julia 親プロセス。
<!-- End -->
<!-- Start -->
Julia lets you pipe the output from both of these processes to another program:
> Juliaでは、これらのプロセスの両方からの出力を別のプログラムにパイプすることができます。
<!-- End -->

```jldoctest
julia> run(pipeline(`echo world` & `echo hello`, `sort`))
hello
world
```

<!-- Start -->
In terms of UNIX plumbing, what's happening here is that a single UNIX pipe object is created and written to by both `echo` processes, and the other end of the pipe is read from by the `sort` command.
> UNIX配管の場合、ここで起こっているのは、単一のUNIXパイプオブジェクトが `echo`プロセスによって作成され、` sort`コマンドによってパイプの他端が読み取られるということです。
<!-- End -->

<!-- Start -->
IO redirection can be accomplished by passing keyword arguments stdin, stdout, and stderr to the `pipeline` function:
> IOリダイレクションは、キーワード引数stdin、stdout、およびstderrを `pipeline`関数に渡すことで実現できます：
<!-- End -->

```julia
pipeline(`do_work`, stdout=pipeline(`sort`, "out.txt"), stderr="errs.txt")
```

<!-- Start -->

#### Avoiding Deadlock in Pipelines

> #### パイプラインのデッドロックを回避する

<!-- End -->
<!-- Start -->
When reading and writing to both ends of a pipeline from a single process, it is important to avoid forcing the kernel to buffer all of the data.
> 1つのプロセスからパイプラインの両端に読み書きするときは、カーネルにすべてのデータをバッファリングさせないようにすることが重要です。
<!-- End -->

<!-- Start -->
For example, when reading all of the output from a command, call `readstring(out)`, not `wait(process)`, since the former will actively consume all of the data written by the process, whereas the latter will attempt to store the data in the kernel's buffers while waiting for a reader to be connected.
> 例えば、コマンドからのすべての出力を読むときは、 `wait（process）`ではなく `readstring（out）`を呼び出します。なぜなら、前者はプロセスによって書き込まれたすべてのデータを積極的に消費するからです。 リーダーが接続されるのを待っている間に、カーネルのバッファーにデータを保管してください。
<!-- End -->

<!-- Start -->
Another common solution is to separate the reader and writer of the pipeline into separate Tasks:
> 別の一般的な解決方法は、パイプラインのリーダーとライターを別々のタスクに分けることです。
<!-- End -->

```julia
writer = @async write(process, "data")
reader = @async do_compute(readstring(process))
wait(process)
fetch(reader)
```

<!-- Start -->

### Complex Example

> ###複雑な例

<!-- End -->
<!-- Start -->
The combination of a high-level programming language, a first-class command abstraction, and automatic setup of pipes between processes is a powerful one. 
> 高度なプログラミング言語、ファーストクラスのコマンド抽象化、プロセス間のパイプの自動セットアップの組み合わせは、強力なものです。
<!-- End -->
<!-- Start -->
To give some sense of the complex pipelines that can be created easily, here are some more sophisticated examples, with apologies for the excessive use of Perl one-liners:
> 簡単に作成できる複雑なパイプラインを理解するために、Perlの1ライナーの過度の使用についてお詫び申し上げますいくつかの洗練された例があります：
<!-- End -->

```julia-repl
julia> prefixer(prefix, sleep) = `perl -nle '$|=1; print "'$prefix' ", $_; sleep '$sleep';'`;

julia> run(pipeline(`perl -le '$|=1; for(0..9){ print; sleep 1 }'`, prefixer("A",2) & prefixer("B",2)))
A 0
B 1
A 2
B 3
A 4
B 5
A 6
B 7
A 8
B 9
```

<!-- Start -->
This is a classic example of a single producer feeding two concurrent consumers: one `perl` process generates lines with the numbers 0 through 9 on them, while two parallel processes consume that output, one prefixing lines with the letter "A", the other with the letter "B". 
> これは、1つのプロデューサが2つのコンシューマコンシューマにコンプリートする古典的な例です.1つの `perl`プロセスは0から9までの数字を持つ行を生成し、2つの並列プロセスはその出力を消費します。 文字「B」で始まる。
<!-- End -->
<!-- Start -->
Which consumer gets the first line is non-deterministic, but once that race has been won, the lines are consumed alternately by one process and then the other. 
> どちらの消費者が最初のラインを得るかは、非決定論的ですが、そのレースが勝利すると、ラインは1つのプロセスで交互に消費され、次にもう1つの消費によって交互に消費されます。
<!-- End -->
<!-- Start -->
(Setting `$|=1` in Perl causes each print statement to flush the [`STDOUT`](@ref) handle, which is necessary for this example to work. 
> （Perlで `$ | = 1`を設定すると、この例文が動作するために必要な[` STDOUT`]（@ ref）ハンドルを各print文でフラッシュします。
<!-- End -->
<!-- Start -->
Otherwise all the output is buffered and printed to the pipe at once, to be read by just one consumer process.)
> それ以外の場合は、すべての出力がバッファリングされ、一度にパイプに出力され、消費者プロセスは1つだけ読み込まれます。）
<!-- End -->

<!-- Start -->
Here is an even more complex multi-stage producer-consumer example:
> さらに複雑な複数ステージのプロデューサ - コンシューマの例を次に示します。
<!-- End -->

```julia-repl
julia> run(pipeline(`perl -le '$|=1; for(0..9){ print; sleep 1 }'`,
           prefixer("X",3) & prefixer("Y",3) & prefixer("Z",3),
           prefixer("A",2) & prefixer("B",2)))
A X 0
B Y 1
A Z 2
B X 3
A Y 4
B Z 5
A X 6
B Y 7
A Z 8
B X 9
```

<!-- Start -->
This example is similar to the previous one, except there are two stages of consumers, and the stages have different latency so they use a different number of parallel workers, to maintain saturated throughput.
> この例は、2つのステージのコンシューマがあり、ステージのレイテンシが異なり、飽和スループットを維持するために異なる数の並列ワーカーを使用する点を除いて、前の例と似ています。
<!-- End -->

<!-- Start -->
We strongly encourage you to try all these examples to see how they work.
> どのように動作するかを確認するために、これらの例をすべて試してみることを強くお勧めします。
<!-- End -->