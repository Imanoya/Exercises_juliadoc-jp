# Parallel Computing
#並列コンピューティング

#Most modern computers possess more than one CPU, and several computers can be combined together in a cluster. 
最新のコンピュータのほとんどは複数のCPUを搭載しており、複数のコンピュータを1つのクラスタにまとめることができます。
#Harnessing the power of these multiple CPUs allows many computations to be completed more quickly. 
これらの複数のCPUの能力を活用することで、より多くの計算をより迅速に完了することができます。
#There are two major factors that influence performance: the speed of the CPUs themselves, and the speed of their access to memory. 
パフォーマンスに影響を与える主な要因は2つあります.1つはCPU自体の速度、もう1つはメモリへのアクセス速度です。
#In a cluster, it's fairly obvious that a given CPU will have fastest access to the RAM within the same computer (node). 
クラスタでは、特定のCPUが同じコンピュータ（ノード）内のRAMに最速でアクセスすることはかなり明らかです。
#Perhaps more surprisingly, similar issues are relevant on a typical multicore laptop, due to differences in the speed of main memory and the [cache](https://www.akkadia.org/drepper/cpumemory.pdf). 
おそらくもっと驚くべきことに、メインメモリと[キャッシュ]（https://www.akkadia.org/drepper/cpumemory.pdf）の速度の違いにより、同様の問題が典型的なマルチコアラップトップに関係していることでしょう。
#Consequently, a good multiprocessing environment should allow control over the "ownership" of a chunk of memory by a particular CPU.
したがって、優れたマルチプロセッシング環境では、特定のCPUによるメモリチャンクの「所有権」を制御できます。
#Julia provides a multiprocessing environment based on message passing to allow programs to run on multiple processes in separate memory domains at once.
Juliaは、プログラムを別々のメモリドメイン内の複数のプロセスで同時に実行できるように、メッセージの受け渡しに基づいたマルチプロセッシング環境を提供します。

#Julia's implementation of message passing is different from other environments such as MPI [^1].
Juliaのメッセージパッシングの実装は、MPI [^ 1]などの他の環境とは異なります。
#Communication in Julia is generally "one-sided", meaning that the programmer needs to explicitly manage only one process in a two-process operation. 
Juliaでのコミュニケーションは一般的に「片面」です。つまり、プログラマは2プロセス操作で1つのプロセスのみを明示的に管理する必要があります。
#Furthermore, these operations typically do not look like "message send" and "message receive" but rather resemble higher-level operations like calls to user functions.
さらに、これらの操作は、通常、「メッセージ送信」および「メッセージ受信」のようには見えませんが、ユーザー機能への呼び出しなどの高レベル操作に似ています。

#Parallel programming in Julia is built on two primitives: *remote references* and *remote calls*.
Juliaの並列プログラミングは、*リモート参照*と*リモート呼び出し*の2つのプリミティブで構築されています。
#A remote reference is an object that can be used from any process to refer to an object stored on a particular process. 
リモート参照は、特定のプロセスに格納されているオブジェクトを参照するために、どのプロセスからでも使用できるオブジェクトです。
#A remote call is a request by one process to call a certain function on certain arguments on another (possibly the same) process.
リモート呼び出しとは、あるプロセスが別の（おそらくは同じ）プロセス上の特定の引数で特定の関数を呼び出すための要求のことです。

#Remote references come in two flavors: [`Future`](@ref) and [`RemoteChannel`](@ref).
リモート参照には、[`Future`]（@ ref）と` `RemoteChannel`（@ref）という2つの味があります。

#A remote call returns a [`Future`](@ref) to its result. 
リモート呼び出しは、その結果に[`Future`]（@ ref）を返します。
#Remote calls return immediately; the process that made the call proceeds to its next operation while the remote call happens somewhere else.
リモート呼び出しはすぐに戻ります。リモート呼び出しが別の場所で発生している間に、呼び出しを行ったプロセスは次の操作に進みます。
#You can wait for a remote call to finish by calling [`wait()`](@ref) on the returned [`Future`](@ref), and you can obtain the full value of the result using [`fetch()`](@ref).
返された[`Future`]（@ ref）に対して[` wait（） `]（@ ref）を呼び出すことによって、リモート呼び出しが終了するのを待つことができます。そして、` `fetch ） `]（@ ref）。

#On the other hand, [`RemoteChannel`](@ref) s are rewritable. 
一方、[`RemoteChannel`]（@ ref）は書き換え可能です。
#For example, multiple processes can co-ordinate their processing by referencing the same remote `Channel`.
例えば、複数のプロセスは、同じリモート「チャネル」を参照することによって、それらの処理を調整することができる。

#Each process has an associated identifier. 
各プロセスには、関連する識別子があります。
#The process providing the interactive Julia prompt always has an `id` equal to 1. 
対話式Juliaプロンプトを提供するプロセスは、常に1に等しい「id」を有する。
#The processes used by default for parallel operations are referred to as "workers". When there is only one process, process 1 is considered a worker. 
パラレル操作でデフォルトで使用されるプロセスは、「ワーカー」と呼ばれます。プロセスが1つだけの場合、プロセス1は作業者と見なされます。
#Otherwise, workers are considered to be all processes other than process 1.
それ以外の場合、ワーカーはプロセス1以外のすべてのプロセスとみなされます。

#Let's try this out. 
これを試してみましょう。
#Starting with `julia -p n` provides `n` worker processes on the local machine. 
`julia -p n`はローカルマシン上で` n`ワーカープロセスを提供します。
#Generally it makes sense for `n` to equal the number of CPU cores on the machine.
一般に、 `n 'はマシン上のCPUコアの数に等しいことが理にかなっています。

```julia
$ ./julia -p 2

julia> r = remotecall(rand, 2, 2, 2)
Future(2, 1, 4, Nullable{Any}())

julia> s = @spawnat 2 1 .+ fetch(r)
Future(2, 1, 5, Nullable{Any}())

julia> fetch(s)
2×2 Array{Float64,2}:
 1.18526  1.50912
 1.16296  1.60607
```

#The first argument to [`remotecall()`](@ref) is the function to call. 
[`remotecall（）`]（@ ref）の最初の引数は呼び出す関数です。
#Most parallel programming in Julia does not reference specific processes or the number of processes available, but [`remotecall()`](@ref) is considered a low-level interface providing finer control. 
Juliaのほとんどの並列プログラミングは、特定のプロセスや利用可能なプロセスの数を参照しませんが、[`remotecall（）`]（@ ref）は、より細かい制御を提供する低レベルのインタフェースとみなされます。
#The second argument to [`remotecall()`](@ref) is the `id` of the process that will do the work, and the remaining arguments will be passed to the function being called.
[`remotecall（）`]（@ ref）の第二引数は、作業を行うプロセスの `id`であり、残りの引数は呼び出される関数に渡されます。

#As you can see, in the first line we asked process 2 to construct a 2-by-2 random matrix, and in the second line we asked it to add 1 to it. The result of both calculations is available in the two futures, `r` and `s`. 
ご覧のように、最初の行では、プロセス2に2行2列のランダムな行列を作成するよう依頼し、2行目で2行に1を追加するように指示しました。両方の計算の結果は、2つの先物、「r」および「s」で利用可能である。
#The [`@spawnat`](@ref) macro evaluates the expression in the second argument on the process specified by the first argument.
[`@ spawnat`]（@ ref）マクロは、第1引数で指定されたプロセスの第2引数の式を評価します。

#Occasionally you might want a remotely-computed value immediately. 
場合によっては、すぐにリモートで計算された値が必要な場合があります。
#This typically happens when you read from a remote object to obtain data needed by the next local operation. 
これは、通常、リモートオブジェクトから読み込んで次のローカル操作で必要なデータを取得するときに発生します。
#The function [`remotecall_fetch()`](@ref) exists for this purpose. 
この目的のために[`remotecall_fetch（）`]（@ ref）関数が存在します。
#It is equivalent to `fetch(remotecall(...))` but is more efficient.
これは `fetch（remotecall（...））`と同じですが、より効率的です。

```julia-repl
julia> remotecall_fetch(getindex, 2, r, 1, 1)
0.18526337335308085
```

#Remember that [`getindex(r,1,1)`](@ref) is [equivalent](@ref man-array-indexing) to `r[1,1]`, so this call fetches the first element of the future `r`.
`` getindex（r、1,1） `]（@ ref）は` r [1,1] `と[equivalent]（@ ref man-array-indexing）であることに注意してください。 将来の `r`。

#The syntax of [`remotecall()`](@ref) is not especially convenient. 
[`remotecall（）`]（@ ref）の構文は特に便利ではありません。
#The macro [`@spawn`](@ref) makes things easier. 
マクロ[`@ spawn`]（@ ref）は物事を簡単にします。
#It operates on an expression rather than a function, and picks where to do the operation for you:
関数ではなく式で動作し、操作を行う場所を指定します。

```julia-repl
julia> r = @spawn rand(2,2)
Future(2, 1, 4, Nullable{Any}())

julia> s = @spawn 1 .+ fetch(r)
Future(3, 1, 5, Nullable{Any}())

julia> fetch(s)
2×2 Array{Float64,2}:
 1.38854  1.9098
 1.20939  1.57158
```

#Note that we used `1 .+ fetch(r)` instead of `1 .+ r`. 
`1。+ r`の代わりに` 1。+ fetch（r） `を使ったことに注意してください。
#This is because we do not know where the code will run, so in general a [`fetch()`](@ref) might be required to move `r` to the process doing the addition. 
これはコードがどこで実行されるのかわからないためです。一般に、 `r`を追加するプロセスに` [fetch（） `]（@ ref）が必要な場合があります。
#In this case, [`@spawn`](@ref) is smart enough to perform the computation on the process that owns `r`, so the [`fetch()`](@ref) will be a no-op (no work is done).
この場合、[`@ spawn`]（@ ref）は` r`を所有するプロセスの計算を実行するほどスマートなので、[`fetch（）`]（@ ref）はノーオペレーション 作業は行われません）。

#(It is worth noting that [`@spawn`](@ref) is not built-in but defined in Julia as a [macro](@ref man-macros).
（[`@ spawn`]（@ ref）は組み込みではなく、[macro]（@ ref man-macros）としてJuliaで定義されていることに注意してください。
#It is possible to define your own such constructs.)
そのような構造を独自に定義することも可能です。）

#An important thing to remember is that, once fetched, a [`Future`](@ref) will cache its value locally. 
重要なことは、一度フェッチされると、[`Future`]（@ ref）がその値をローカルにキャッシュするということです。
#Further [`fetch()`](@ref) calls do not entail a network hop. 
さらに、[`fetch（）`]（@ ref）呼び出しはネットワークホップを伴わない。
#Once all referencing [`Future`](@ref)s have fetched, the remote stored value is deleted.
[`Future`]（@ ref）を参照しているすべてのものが取得されると、リモートの格納された値が削除されます。

## Code Availability and Loading Packages
##コードの可用性とロードパッケージ

#Your code must be available on any process that runs it. 
あなたのコードはそれを実行するどのプロセスでも利用可能でなければなりません。
#For example, type the following into the Julia prompt:
たとえば、Juliaプロンプトに次のように入力します。

```julia-repl
julia> function rand2(dims...)
           return 2*rand(dims...)
       end

julia> rand2(2,2)
2×2 Array{Float64,2}:
 0.153756  0.368514
 1.15119   0.918912

julia> fetch(@spawn rand2(2,2))
ERROR: RemoteException(2, CapturedException(UndefVarError(Symbol("#rand2"))
[...]
```

#Process 1 knew about the function `rand2`, but process 2 did not.
プロセス1は関数 `rand2`を知っていましたが、プロセス2は関数を認識しませんでした。

#Most commonly you'll be loading code from files or packages, and you have a considerable amount of flexibility in controlling which processes load code. 
最も一般的には、ファイルやパッケージからコードをロードし、コードをロードするプロセスを柔軟に制御できます。
#Consider a file, `DummyModule.jl`, containing the following code:
次のコードを含むファイル `DummyModule.jl`を考えてみましょう：

```julia
module DummyModule

export MyType, f

mutable struct MyType
    a::Int
end

f(x) = x^2+1

println("loaded")

end
```

#Starting Julia with `julia -p 2`, you can use this to verify the following:
Juliaを `julia -p 2`で起動すると、これを使って次のことを確認できます：

    *
   # `include("DummyModule.jl")` loads the file on just a single process (whichever one executes the statement).
   `include（" DummyModule.jl "）`は、単一のプロセス（文を実行するもの）でファイルを読み込みます。
    *
   # `using DummyModule` causes the module to be loaded on all processes; however, the module is brought into scope only on the one executing the statement.
   `DummyModuleを使用すると`モジュールは全てのプロセスにロードされます。 しかし、モジュールは、そのステートメントを実行しているスコープ内でのみスコープに入れられます。
    *
   # As long as `DummyModule` is loaded on process 2, commands like
   `DummyModule`がプロセス2にロードされている限り、

    ```julia
    rr = RemoteChannel(2)
    put!(rr, MyType(7))
    ```

   # allow you to store an object of type `MyType` on process 2 even if `DummyModule` is not in scope on process 2.
   プロセス2上で `DummyModule`がスコープ内になくても、プロセス2に` MyType`型のオブジェクトを格納することができます。

#You can force a command to run on all processes using the [`@everywhere`](@ref) macro. 
[`@ everywhere`]（@ ref）マクロを使って、すべてのプロセスに対してコマンドを実行させることができます。
#For example, `@everywhere` can also be used to directly define a function on all processes:
たとえば、 `@ everywhere`はすべてのプロセスで直接関数を定義するためにも使用できます：

```julia-repl
julia> @everywhere id = myid()

julia> remotecall_fetch(()->id, 2)
2
```

#A file can also be preloaded on multiple processes at startup, and a driver script can be used to drive the computation:
起動時にファイルを複数のプロセスにプリロードすることもでき、ドライバスクリプトを使用して計算を駆動することもできます。

```
julia -p <n> -L file1.jl -L file2.jl driver.jl
```

#The Julia process running the driver script in the example above has an `id` equal to 1, just like a process providing an interactive prompt.
上の例のドライバスクリプトを実行しているJuliaプロセスは、対話型のプロンプトを提供するプロセスのように、1に等しいidを持っています。

#The base Julia installation has in-built support for two types of clusters:
基本的なJuliaのインストールでは、2種類のクラスタのサポートが組み込まれています。

   *  
   #    A local cluster specified with the `-p` option as shown above.
        上記の `-p`オプションで指定されたローカルクラスタ。

   * 
       A cluster spanning machines using the `--machinefile` option. 
        `--machinefile`オプションを使ってマシンを張るクラスタ。
       This uses a passwordless `ssh` login to start Julia worker processes (from the same path as the current host) on the specified machines.
        これはパスワードなしの `ssh`ログインを使用して、指定されたマシン上のJuliaワーカープロセス（現在のホストと同じパスから）を開始します。

#Functions [`addprocs()`](@ref), [`rmprocs()`](@ref), [`workers()`](@ref), and others are available as a programmatic means of adding, removing and querying the processes in a cluster.
プログラムの追加、削除、および削除の手段として、関数[`addprocs（）`]（@ ref）、[`rmprocs（）`]（@ ref）、[`workers（）`]（@ ref）クラスタ内のプロセスを照会する。

#Note that workers do not run a `.juliarc.jl` startup script, nor do they synchronize their global state (such as global variables, new method definitions, and loaded modules) with any of the other running processes.
ワーカーは、 `.juliarc.jl`起動スクリプトを実行したり、グローバル変数（グローバル変数、新しいメソッド定義、ロードされたモジュールなど）を他の実行中のプロセスと同期させないことに注意してください。

#Other types of clusters can be supported by writing your own custom `ClusterManager`, as described below in the [ClusterManagers](@ref) section.
以下の[ClusterManagers]（@ ref）セクションで説明するように、独自のカスタムClusterManagerを作成することで、他のタイプのクラスタをサポートすることができます。

## Data Movement
##データ移動

#Sending messages and moving data constitute most of the overhead in a parallel program. 
メッセージの送信とデータの移動は、並列プログラムのオーバーヘッドの大半を占めます。
#Reducing the number of messages and the amount of data sent is critical to achieving performance and scalability.
パフォーマンスとスケーラビリティを達成するには、メッセージ数と送信データ量を削減することが不可欠です。
#To this end, it is important to understand the data movement performed by Julia's various parallel programming constructs.
このためには、Juliaのさまざまな並列プログラミング構造によって実行されるデータ移動を理解することが重要です。

#[`fetch()`](@ref) can be considered an explicit data movement operation, since it directly asks that an object be moved to the local machine. 
[`fetch（）`]（@ ref）は、オブジェクトがローカルマシンに移動することを直接要求するので、明示的なデータ移動操作と見なすことができます。
#[`@spawn`](@ref) (and a few related constructs) also moves data, but this is not as obvious, hence it can be called an implicit data movement operation. 
[`@ spawn`]（@ref）（およびいくつかの関連する構造体）もデータを移動しますが、これは明白ではないので、暗黙のデータ移動操作と呼ばれることがあります。
#Consider these two approaches to constructing and squaring a random matrix:
ランダム行列の構築と二乗の2つのアプローチを考えてみましょう。

#Method 1:
方法1：

```julia-repl
julia> A = rand(1000,1000);

julia> Bref = @spawn A^2;

[...]

julia> fetch(Bref);
```

Method 2:

```julia-repl
julia> Bref = @spawn rand(1000,1000)^2;

[...]

julia> fetch(Bref);
```

#The difference seems trivial, but in fact is quite significant due to the behavior of [`@spawn`](@ref).
違いはほんのわずかですが、実際は[`@ spawn`]（@ ref）の動作のためにかなり重要です。
#In the first method, a random matrix is constructed locally, then sent to another process where it is squared. 
第1の方法では、ランダムマトリクスが局所的に構築され、次にそれが二乗された別のプロセスに送られる。
#In the second method, a random matrix is both constructed and squared on another process. 
第2の方法では、ランダムマトリクスが構築され、別のプロセスで2乗される。
#Therefore the second method sends much less data than the first.
したがって、第2の方法は、第1の方法よりもはるかに少ないデータを送信する。

#In this toy example, the two methods are easy to distinguish and choose from. However, in a real program designing data movement might require more thought and likely some measurement. 
このおもちゃの例では、2つの方法は区別して選択するのが簡単です。しかし、実際のプログラムでは、データの動きを設計するには、より多くの考えが必要であり、ある程度の測定が必要になることがあります
#For example, if the first process needs matrix `A` then the first method might be better. 
例えば、最初のプロセスが行列Aを必要とする場合、最初のメソッドがより良いかもしれません。
#Or, if computing `A` is expensive and only the current process has it, then moving it to another process might be unavoidable. 
あるいは、 `A`を計算するのが高価で、現在のプロセスだけがそれを持っているならば、それを別のプロセスに移動することは避けられないでしょう。
#Or, if the current process has very little to do between the [`@spawn`](@ref) and `fetch(Bref)`, it might be better to eliminate the parallelism altogether. 
現在のプロセスが[`@spawn`]（@ref）と` fetch（Bref） `の間に非常に少ない場合、並列性を完全に排除するほうが良いかもしれません。
#Or imagine `rand(1000,1000)` is replaced with a more expensive operation. 
または `rand（1000,1000）`がより高価な操作に置き換えられたとします。
#Then it might make sense to add another [`@spawn`](@ref) statement just for this step.
それでは、このステップのためだけに別の[`@ spawn`]（@ ref）文を追加すると意味があります。

## Global variables
＃グローバル変数
#Expressions executed remotely via `@spawn`, or closures specified for remote execution using `remotecall` may refer to global variables. 
`@spawn`を介してリモートで実行される式、または` remotecall`を使用してリモート実行のために指定されたクロージャは、グローバル変数を参照します。
#Global bindings under module `Main` are treated a little differently compared to global bindings in other modules. 
`Main`モジュールのグローバルバインディングは、他のモジュールのグローバルバインディングとは少し異なります。
#Consider the following code snippet:
次のコードスニペットを考えてみましょう。

```julia-repl
A = rand(10,10)
remotecall_fetch(()->norm(A), 2)
```

#In this case [`norm`](@ref) is a function that takes 2D array as a parameter, and MUST be defined in the remote process.  
この場合、[`norm`]（@ ref）は、2D配列をパラメータとする関数であり、リモートプロセスで定義しなければなりません。
#You could use any function other than `norm` as long as it is defined in the remote process and accepts the appropriate parameter.
リモートプロセスで定義され、適切なパラメータを受け入れる限り、 `norm`以外の関数を使うことができます。

#Note that `A` is a global variable defined in the local workspace. Worker 2 does not have a variable called `A` under `Main`. 
`A`はローカルワークスペースで定義されたグローバル変数です。 Worker 2には、「Main」の下に「A」という変数がありません。
#The act of shipping the closure `()->norm(A)` to worker 2 results in `Main.A` being defined on 2. 
クロージャー `（） - > norm（A）`をワーカー2に発送する行為は、 `Main.A`が2で定義される結果になります。
#`Main.A` continues to exist on worker 2 even after the call `remotecall_fetch` returns. 
`main.A`は、` remotecall_fetch`呼び出しが復帰した後でさえ、作業者2に存在し続けます。
#Remote calls with embedded global references (under `Main` module only) manage globals as follows:
`Main`モジュールのみの下に）グローバル参照を埋め込んだリモート呼び出しは、以下のようにグローバルを管理します：

- 
   # New global bindings are created on destination workers if they are referenced as part of a remote call.
    新しいグローバルバインディングは、リモート呼び出しの一部として参照されている場合は、宛先ワーカーで作成されます。

- 
   # Global constants are declared as constants on remote nodes too.
     グローバル定数はリモートノードでも定数として宣言されています。

- 
   # Globals are re-sent to a destination worker only in the context of a remote call, and then only if its value has changed. 
    グローバルコールは、リモート呼び出しのコンテキストでのみ宛先ワーカーに再送信され、その値が変更された場合にのみ送信されます。
   # Also, the cluster does not synchronize global bindings across nodes. 
     また、クラスタは、ノード間のグローバルバインドを同期しません。
   # For example:
     例えば：

  ```julia
  A = rand(10,10)
  remotecall_fetch(()->norm(A), 2) # worker 2
  A = rand(10,10)
  remotecall_fetch(()->norm(A), 3) # worker 3
  A = nothing
  ```

  # Executing the above snippet results in `Main.A` on worker 2 having a different value from `Main.A` on worker 3, while the value of `Main.A` on node 1 is set to `nothing`.
  上記のスニペットを実行すると、作業者2の「Main.A」は作業者3の「Main.A」とは異なる値になり、ノード1の「Main.A」の値は「nothing」に設定されます。

#As you may have realized, while memory associated with globals may be collected when they are reassigned on the master, no such action is taken on the workers as the bindings continue to be valid.
ご存じのように、グローバルに関連付けられたメモリは、マスタで再割り当てされたときに収集されることがありますが、バインディングが引き続き有効であるため、ワーカーにはそのような動作は行われません。
#[`clear!`](@ref) can be used to manually reassign specific globals on remote nodes to `nothing` once they are no longer required. 
[`clear！`]（@ ref）は、リモートノード上の特定のグローバルをもう一度必要でなくなると、何もしないように手動で再割り当てするために使用できます。
#This will release any memory associated with them as part of a regular garbage collection cycle.
これにより、通常のガベージコレクションサイクルの一環として、関連付けられたメモリが解放されます。

#Thus programs should be careful referencing globals in remote calls. In fact, it is preferable to avoid them altogether if possible. 
したがって、プログラムはリモート呼び出しでグローバルを参照するように注意する必要があります。 事実、可能ならばそれらを完全に避けることが望ましい。
#If you must reference globals, consider using `let` blocks to localize global variables.
グローバルを参照する必要がある場合は、 `let`ブロックを使ってグローバル変数をローカライズすることを検討してください。

#For example:
例えば：

```julia-repl
julia> A = rand(10,10);

julia> remotecall_fetch(()->A, 2);

julia> B = rand(10,10);

julia> let B = B
           remotecall_fetch(()->B, 2)
       end;

julia> @spawnat 2 whos();

julia>  From worker 2:                               A    800 bytes  10×10 Array{Float64,2}
        From worker 2:                            Base               Module
        From worker 2:                            Core               Module
        From worker 2:                            Main               Module
```

# As can be seen, global variable `A` is defined on worker 2, but `B` is captured as a local variable and hence a binding for `B` does not exist on worker 2.
ワーク変数2にはグローバル変数Aが定義されているが、ローカル変数としてはBが取り込まれており、従業員2にはBの束縛は存在しないことがわかる。

## Parallel Map and Loops
##パラレルマップとループ

# Fortunately, many useful parallel computations do not require data movement. 
幸いにも、多くの有用な並列計算では、データの移動は必要ありません。
# A common example is a Monte Carlo simulation, where multiple processes can handle independent simulation trials simultaneously. 
一般的な例は、複数のプロセスが独立したシミュレーション試行を同時に処理できるモンテカルロシミュレーションです。
# We can use [`@spawn`](@ref) to flip coins on two processes. 
私たちは[`@ spawn`]（@ ref）を使って2つのプロセスでコインを反転させることができます。
# First, write the following function in `count_heads.jl`:
まず、 `count_heads.jl`に次の関数を書いてください：

```julia
function count_heads(n)
    c::Int = 0
    for i = 1:n
        c += rand(Bool)
    end
    c
end
```

#The function `count_heads` simply adds together `n` random bits. 
`count_heads`関数は単に` n`個のランダムビットを加算します。
#Here is how we can perform some trials on two machines, and add together the results:
2台のマシンでいくつかの試行を実行し、結果を一緒に追加する方法は次のとおりです。

```julia-repl
julia> @everywhere include("count_heads.jl")

julia> a = @spawn count_heads(100000000)
Future(2, 1, 6, Nullable{Any}())

julia> b = @spawn count_heads(100000000)
Future(3, 1, 7, Nullable{Any}())

julia> fetch(a)+fetch(b)
100001564
```

#This example demonstrates a powerful and often-used parallel programming pattern. 
この例は、強力でよく使われる並列プログラミングパターンを示しています。
#Many iterations run independently over several processes, and then their results are combined using some function.
多くの反復はいくつかのプロセスで独立して実行され、その結果はある機能を使用して結合されます。
#The combination process is called a *reduction*, since it is generally tensor-rank-reducing: a vector of numbers is reduced to a single number, or a matrix is reduced to a single row or column, etc. 
結合プロセスは、一般にテンソルランク削減であるため、* reduce *と呼ばれます。数字のベクトルが単一の数に減らされたり、行列が単一の行や列に縮小されたりします。
#In code, this typically looks like the pattern `x = f(x,v[i])`, where `x` is the accumulator, `f` is the reduction function, and the `v[i]` are the elements being reduced. 
コードでは、これは通常、 `x = f（x、v [i]）`のように見えます。ここで `x`はアキュムレータ、` f`はリダクション関数、 `v [i]`は要素 削減される。
#It is desirable for `f` to be associative, so that it does not matter what order the operations are performed in.
`f`は連想型であることが望ましいので、操作の実行順序は関係ありません。

#Notice that our use of this pattern with `count_heads` can be generalized. 
このパターンを `count_heads`で使用することは一般化できることに注意してください。
#We used two explicit [`@spawn`](@ref) statements, which limits the parallelism to two processes. 
2つの明示的な[`@ spawn`]（@ ref）文を使用して、並列性を2つのプロセスに限定しました。
#To run on any number of processes, we can use a *parallel for loop*, which can be written in Julia using [`@parallel`](@ref) like this:
任意の数のプロセスを実行するために、私たちは* for parallel for loop *を使うことができます。これはJuliaで[`@ parallel`]（@ ref）を使って以下のように書くことができます：

```julia
nheads = @parallel (+) for i = 1:200000000
    Int(rand(Bool))
end
```

#This construct implements the pattern of assigning iterations to multiple processes, and combining them with a specified reduction (in this case `(+)`). 
この構造体は、反復を複数のプロセスに割り当て、指定された縮小（この場合は `（+）`）と組み合わせるというパターンを実装します。
#The result of each iteration is taken as the value of the last expression inside the loop. 
各反復の結果は、ループ内の最後の式の値と見なされます。
#The whole parallel loop expression itself evaluates to the final answer.
すべての並列ループ式自体が最終的な解答に評価されます。

#Note that although parallel for loops look like serial for loops, their behavior is dramatically different. 
並列forループは直列forループのように見えますが、その動作は劇的に異なります。
#In particular, the iterations do not happen in a specified order, and writes to variables or arrays will not be globally visible since iterations run on different processes. 
特に、反復は指定された順序で行われるわけではなく、反復は異なるプロセスで実行されるため、変数や配列への書き込みはグローバルには表示されません。
#Any variables used inside the parallel loop will be copied and broadcast to each process.
並列ループの内部で使用される変数はすべてコピーされ、各プロセスにブロードキャストされます。

#For example, the following code will not work as intended:
たとえば、次のコードは意図したとおりに動作しません。

```julia
a = zeros(100000)
@parallel for i = 1:100000
    a[i] = i
end
```

#This code will not initialize all of `a`, since each process will have a separate copy of it.
このコードは `a`の全てを初期化しません。なぜなら、各プロセスには別々のコピーがあるからです。
#Parallel for loops like these must be avoided. Fortunately, [Shared Arrays](@ref man-shared-arrays) can be used to get around this limitation:
これらのような並列ループは避けなければなりません。 幸いにも、[Shared Arrays]（@ ref man-shared-arrays）を使ってこの制限を回避することができます：

```julia
a = SharedArray{Float64}(10)
@parallel for i = 1:10
    a[i] = i
end
```

#Using "outside" variables in parallel loops is perfectly reasonable if the variables are read-only:
並列ループで "外部"変数を使用するのは、変数が読み取り専用の場合は完全に合理的です。

```julia
a = randn(1000)
@parallel (+) for i = 1:100000
    f(a[rand(1:end)])
end
```

#Here each iteration applies `f` to a randomly-chosen sample from a vector `a` shared by all processes.
ここで、各反復は、すべてのプロセスによって共有されるベクトル「a」からランダムに選択されたサンプルに「f」を適用する。

#As you could see, the reduction operator can be omitted if it is not needed. 
ご覧のように、必要がない場合は省略することができます。
#In that case, the loop executes asynchronously, i.e. it spawns independent tasks on all available workers and returns an array of [`Future`](@ref) immediately without waiting for completion. 
この場合、ループは非同期的に実行されます。つまり、使用可能なすべてのワーカーで独立したタスクが生成され、完了を待たずに直ちに[`Future`]（@ ref）の配列が返されます。
#The caller can wait for the [`Future`](@ref) completions at a later point by calling [`fetch()`](@ref) on them, or wait for completion at the end of the loop by prefixing it with [`@sync`](@ref), like `@sync @parallel for`.
呼び出し側は、[`fetch（）`]（@ ref）を呼び出すことによって、後で[`Future``（@ ref）の補完を待つことも、ループの最後に補完を待つこともできます @sync @parallel for`のように、[`@ sync`]（@ ref）

#In some cases no reduction operator is needed, and we merely wish to apply a function to all integers in some range (or, more generally, to all elements in some collection). 
場合によっては、減算演算子は必要なく、ある範囲内のすべての整数（またはより一般的には、あるコレクション内のすべての要素）に関数を適用したいだけです。
#This is another useful operation called *parallel map*, implemented in Julia as the [`pmap()`](@ref) function. 
これは、Juliaで[`pmap（）`]（@ ref）関数として実装された* parallel map *と呼ばれるもう1つの便利な操作です。
#For example, we could compute the singular values of several large random matrices in parallel as follows:
例えば、いくつかの大きなランダム行列の特異値を以下のように並列に計算することができます。

```julia-repl
julia> M = Matrix{Float64}[rand(1000,1000) for i = 1:10];

julia> pmap(svd, M);
```

#Julia's [`pmap()`](@ref) is designed for the case where each function call does a large amount of work. 
Juliaの[`pmap（）`]（@ ref）は、各関数呼び出しが大量の作業を行う場合に設計されています。
#In contrast, `@parallel for` can handle situations where each iteration is tiny, perhaps merely summing two numbers. 
対照的に、 `@parallel for`は各反復が小さい状況を扱うことができます。おそらく2つの数値を加算するだけです。
#Only worker processes are used by both [`pmap()`](@ref) and `@parallel for` for the parallel computation. 
並列計算では、[`pmap（）`]（@ ref）と `@parallel for`の両方でワーカープロセスだけが使用されます。
#In case of `@parallel for`, the final reduction is done on the calling process.
`@parallel for`の場合、最終的な削減は呼び出しプロセスで行われます。

## Synchronization With Remote References
##リモート参照との同期

## Scheduling
##スケジューリング

#Julia's parallel programming platform uses [Tasks (aka Coroutines)](@ref man-tasks) to switch among multiple computations. 
Juliaの並列プログラミングプラットフォームは、[Tasks（aka Coroutines）]（@ ref man-tasks）を使用して複数の計算を切り替えます。
#Whenever code performs a communication operation like [`fetch()`](@ref) or [`wait()`](@ref), the current task is suspended and a scheduler picks another task to run. 
コードが[`fetch（）`]（@ ref）や[`wait（）`]（@ ref）のような通信操作を行うと、現在のタスクは中断され、スケジューラは実行する別のタスクを選択します。
#A task is restarted when the event it is waiting for completes.

#For many problems, it is not necessary to think about tasks directly. 
多くの問題では、タスクについて直接考える必要はありません。
#However, they can be used to wait for multiple events at the same time, which provides for *dynamic scheduling*. 
ただし、同時に複数のイベントを待つために使用することができ、*動的スケジューリング*を提供します。
#In dynamic scheduling, a program decides what to compute or where to compute it based on when other jobs finish. 
動的スケジューリングでは、プログラムは、他のジョブがいつ終了するかに基づいて、計算対象や計算場所を決定します。
#This is needed for unpredictable or unbalanced workloads, where we want to assign more work to processes only when they finish their current tasks.
これは、現在のタスクが完了したときにのみプロセスに多くの作業を割り当てたい、予測不能または不均衡な作業負荷に必要です。

#As an example, consider computing the singular values of matrices of different sizes:

例として、異なるサイズの行列の特異値を計算することを検討してください。

```julia-repl
julia> M = Matrix{Float64}[rand(800,800), rand(600,600), rand(800,800), rand(600,600)];

julia> pmap(svd, M);
```

#If one process handles both 800×800 matrices and another handles both 600×600 matrices, we will not get as much scalability as we could. 
1つのプロセスが800×800の行列を処理し、別のプロセスが両方とも600×600の行列を処理する場合、我々は可能な限りスケーラビリティを得ることはできません。
#The solution is to make a local task to "feed" work to each process when it completes its current task. 
解決方法は、現在のタスクが完了したときに各プロセスに作業を「フィード」するローカルタスクを作成することです。
#For example, consider a simple [`pmap()`](@ref) implementation:
例えば、単純な[`pmap（）`]（@ ref）の実装を考えてみましょう：

```julia
function pmap(f, lst)
    np = nprocs()  # determine the number of processes available
    n = length(lst)
    results = Vector{Any}(n)
    i = 1
    # function to produce the next work item from the queue.
    # in this case it's just an index.
    nextidx() = (idx=i; i+=1; idx)
    @sync begin
        for p=1:np
            if p != myid() || np == 1
                @async begin
                    while true
                        idx = nextidx()
                        if idx > n
                            break
                        end
                        results[idx] = remotecall_fetch(f, p, lst[idx])
                    end
                end
            end
        end
    end
    results
end
```

#[`@async`](@ref) is similar to [`@spawn`](@ref), but only runs tasks on the local process. 
[`@ async`]（@ref）は[` @spawn`]（@ ref）と似ていますが、ローカルプロセス上でのみタスクを実行します。
#We use it to create a "feeder" task for each process. 
私たちは、それを使って各プロセスの「フィーダー」タスクを作成します。
#Each task picks the next index that needs to be computed, then waits for its process to finish, then repeats until we run out of indexes. 
各タスクは、計算が必要な次のインデックスを選択し、そのプロセスが完了するのを待ってから、インデックスがなくなるまで繰り返します。
#Note that the feeder tasks do not begin to execute until the main task reaches the end of the [`@sync`](@ref) block, at which point it surrenders control and waits for all the local tasks to complete before returning from the function. 
フィーダタスクは、メインタスクが[`@ sync`]（@ ref）ブロックの終わりに達するまで実行されないことに注意してください。この時点で、制御は取り消され、すべてのローカルタスクが完了するのを待ちます。関数。
#The feeder tasks are able to share state via `nextidx()` because they all run on the same process. 
フィーダタスクは、すべて同じプロセス上で実行されるため、 `nextidx（） 'を介して状態を共有することができます。
#No locking is required, since the threads are scheduled cooperatively and not preemptively. 
スレッドは協調的にスケジュールされ、優先的にスケジュールされないため、ロックは必要ありません。
#This means context switches only occur at well-defined points: in this case, when [`remotecall_fetch()`](@ref) is called.
これはコンテキストスイッチが明確なポイントでのみ発生することを意味します。この場合、[`remotecall_fetch（）`]（@ ref）が呼び出されたときです。

## Channels
##チャンネル

#The section on [`Task`](@ref)s in [Control Flow](@ref) discussed the execution of multiple functions in a co-operative manner. 
[Control Flow]（@ ref）の[`Task`]（@ ref）のセクションは、複数の関数の実行を協調的に議論しました。
#[`Channel`](@ref)s can be quite useful to pass data between running tasks, particularly those involving I/O operations.
[`Channel`]（@ ref）は、特にI / O操作を含む実行中のタスク間でデータを渡すのに非常に便利です。

#Examples of operations involving I/O include reading/writing to files, accessing web services, executing external programs, etc. 
I / Oを含む操作の例には、ファイルの読み書き、Webサービスへのアクセス、外部プログラムの実行などがあります。
#In all these cases, overall execution time can be improved if other tasks can be run while a file is being read, or while waiting for an external service/program to complete.
これらのすべてのケースでは、ファイルが読み込まれている間、または外部のサービス/プログラムが完了するのを待っている間に他のタスクを実行できる場合、全体の実行時間を改善できます。

#A channel can be visualized as a pipe, i.e., it has a write end and read end.
チャネルは、パイプとして視覚化することができ、すなわち、書き込み終了と読み出し終了を有する。

   *  
   #     Multiple writers in different tasks can write to the same channel concurrently via [`put!()`](@ref) calls.
        異なるタスクの複数のライターは、[`put！（）`]（@ ref）呼び出しを介して同じチャネルに同時に書き込むことができます。
   * 
   #     Multiple readers in different tasks can read data concurrently via [`take!()`](@ref) calls.
         異なるタスクの複数の読者は、[`take！（）`]（@ ref）呼び出しによって同時にデータを読み取ることができます。
   * 
   #     As an example:
         例として：

    ```julia
    # Given Channels c1 and c2,
    c1 = Channel(32)
    c2 = Channel(32)

    # and a function `foo()` which reads items from from c1, processes the item read
    # and writes a result to c2,
    function foo()
        while true
            data = take!(c1)
            [...]               # process data
            put!(c2, result)    # write out result
        end
    end

    # we can schedule `n` instances of `foo()` to be active concurrently.
    for _ in 1:n
        @schedule foo()
    end
    ```
   * 
  # Channels are created via the `Channel{T}(sz)` constructor. 
    チャネルは `Channel {T}（sz）`コンストラクタを介して生成されます。
  # The channel will only hold objects of type `T`. 
    チャネルは `T`型のオブジェクトだけを保持します。
  # If the type is not specified, the channel can hold objects of any type. 
    型が指定されていない場合、チャネルは任意の型のオブジェクトを保持できます。
  # `sz` refers to the maximum number of elements that can be held in the channel at any time. 
    `sz`はいつでもチャネルに保持できる要素の最大数を指します。
  # For example, `Channel(32)` creates a channel that can hold a maximum of 32 objects of any type. 
    たとえば、 `Channel（32）`は任意のタイプのオブジェクトを最大32個保持できるチャネルを作成します。
  # A `Channel{MyType}(64)` can hold up to 64 objects of `MyType` at any time.
    `Channel {MyType}（64）`は `MyType`のオブジェクトをいつでも最大64個まで保持できます。
   * 
  # If a [`Channel`](@ref) is empty, readers (on a [`take!()`](@ref) call) will block until data is available. 
    [`Channel`]（@ref）が空の場合、（` `take！（）` '（@ ref）呼び出しで）データが利用可能になるまでブロックされます。
   * 
  # If a [`Channel`](@ref) is full, writers (on a [`put!()`](@ref) call) will block until space becomes available.
   [`Channel`]（@ ref）がいっぱいになると、（` put！（） `（@ ref）呼び出しのライターはスペースが利用可能になるまでブロックします。
   * 
  # [`isready()`](@ref) tests for the presence of any object in the channel, while [`wait()`](@ref) waits for an object to become available.
   [`isready（）`]（@ ref）はチャネル内のオブジェクトの存在をテストし、[`wait（）`]（@ ref）はオブジェクトが利用可能になるのを待ちます。
   * 
  # A [`Channel`](@ref) is in an open state initially. 
    [`Channel`]（@ ref）は最初はオープン状態です。
  # This means that it can be read from and written to freely via [`take!()`](@ref) and [`put!()`](@ref) calls. [`close()`](@ref) closes a [`Channel`](@ref).
   つまり、[`take！（）`]（@ ref）と[`put！（）`]（@ ref）呼び出しを使って自由に読み書きできます。 [`close（）`]（@ ref）は[`Channel`]（@ ref）を閉じます。
  # On a closed [`Channel`](@ref), [`put!()`](@ref) will fail. For example:
   閉じた[`Channel`]（@ ref）では、` `put！（）`]（@ ref）は失敗します。例えば：

```julia-repl
julia> c = Channel(2);

julia> put!(c, 1) # `put!` on an open channel succeeds
1

julia> close(c);

julia> put!(c, 2) # `put!` on a closed channel throws an exception.
ERROR: InvalidStateException("Channel is closed.",:closed)
[...]
```

   * 
   #[`take!()`](@ref) and [`fetch()`](@ref) (which retrieves but does not remove the value) on a closed channel successfully return any existing values until it is emptied. 
   閉じたチャンネルの[`take！（）`]（@ref）と[`fetch（）`]（@ref）は空になるまで正常に値を返します。
   #Continuing the above example:
   上記の例を続けます：

```julia-repl
julia> fetch(c) # Any number of `fetch` calls succeed.
1

julia> fetch(c)
1

julia> take!(c) # The first `take!` removes the value.
1

julia> take!(c) # No more data available on a closed channel.
ERROR: InvalidStateException("Channel is closed.",:closed)
[...]
```

#A `Channel` can be used as an iterable object in a `for` loop, in which case the loop runs as long as the `Channel` has data or is open. 
`Channel`は` for`ループ内の反復可能オブジェクトとして使用することができます。この場合、 `Channel`がデータを持つか開いている限りループが実行されます。
#The loop variable takes on all values added to the `Channel`. The `for` loop is terminated once the `Channel` is closed and emptied.
ループ変数は、 `Channel`に追加されたすべての値をとります。 `for`ループは` Channel`が閉じて空になると終了します。

#For example, the following would cause the `for` loop to wait for more data:
例えば、次のようにすると、 `for`ループはより多くのデータを待つでしょう：

```julia-repl
julia> c = Channel{Int}(10);

julia> foreach(i->put!(c, i), 1:3) # add a few entries

julia> data = [i for i in c]
```

#while this will return after reading all data:
これはすべてのデータを読み取った後に戻ります：

```julia-repl
julia> c = Channel{Int}(10);

julia> foreach(i->put!(c, i), 1:3); # add a few entries

julia> close(c);                    # `for` loops can exit

julia> data = [i for i in c]
3-element Array{Int64,1}:
 1
 2
 3
```

#Consider a simple example using channels for inter-task communication. 
タスク間通信にチャネルを使用する簡単な例を考えてみましょう。
#We start 4 tasks to process data from a single `jobs` channel. Jobs, identified by an id (`job_id`), are written to the channel.
1つの `jobs`チャンネルからデータを処理するための4つのタスクを開始します。 id（ `job_id`）で識別されるジョブがチャネルに書き込まれます。
#Each task in this simulation reads a `job_id`, waits for a random amout of time and writes back a tuple of `job_id` and the simulated time to the results channel. 
このシミュレーションの各タスクは `job_id`を読み取り、ランダムな時間を待って` job_id`のタプルとシミュレートされた時間を結果チャンネルに書き戻します。
#Finally all the `results` are printed out.
最後にすべての結果が出力されます。

```julia-repl
julia> const jobs = Channel{Int}(32);

julia> const results = Channel{Tuple}(32);

julia> function do_work()
           for job_id in jobs
               exec_time = rand()
               sleep(exec_time)                # simulates elapsed time doing actual work
                                               # typically performed externally.
               put!(results, (job_id, exec_time))
           end
       end;

julia> function make_jobs(n)
           for i in 1:n
               put!(jobs, i)
           end
       end;

julia> n = 12;

julia> @schedule make_jobs(n); # feed the jobs channel with "n" jobs

julia> for i in 1:4 # start 4 tasks to process requests in parallel
           @schedule do_work()
       end

julia> @elapsed while n > 0 # print out results
           job_id, exec_time = take!(results)
           println("$job_id finished in $(round(exec_time,2)) seconds")
           n = n - 1
       end
4 finished in 0.22 seconds
3 finished in 0.45 seconds
1 finished in 0.5 seconds
7 finished in 0.14 seconds
2 finished in 0.78 seconds
5 finished in 0.9 seconds
9 finished in 0.36 seconds
6 finished in 0.87 seconds
8 finished in 0.79 seconds
10 finished in 0.64 seconds
12 finished in 0.5 seconds
11 finished in 0.97 seconds
0.029772311
```

#The current version of Julia multiplexes all tasks onto a single OS thread. 
現在のバージョンのJuliaは、すべてのタスクを単一のOSスレッドに多重化します。
#Thus, while tasks involving I/O operations benefit from parallel execution, compute bound tasks are effectively executed sequentially on a single OS thread. 
したがって、I / O操作を含むタスクはパラレル実行の恩恵を受けますが、バインドされたタスクは単一のOSスレッドで効率的に順次実行されます。
#Future versions of Julia may support scheduling of tasks on multiple threads, in which case compute bound tasks will see benefits of parallel execution too.
Juliaの将来のバージョンでは、複数のスレッドのタスクのスケジューリングがサポートされています。この場合、バインドされたタスクの計算には並列実行のメリットもあります。

## Remote References and AbstractChannels
##リモート参照と抽象チャンネル

#Remote references always refer to an implementation of an `AbstractChannel`.
リモート参照は、常に `AbstractChannel`の実装を参照します。

#A concrete implementation of an `AbstractChannel` (like `Channel`), is required to implement [`put!()`](@ref), [`take!()`](@ref), [`fetch()`](@ref), [`isready()`](@ref) and [`wait()`](@ref).
[put！（） `]（@ ref）、[` take！（） `]（@ref）、[` fetch（）]を実装するには `AbstractChannel`（` Channel`のような） @]、[`isready（）`]（@ ref）、[`wait（）`]（@ ref）のいずれかを指定します。
#The remote object referred to by a [`Future`](@ref) is stored in a `Channel{Any}(1)`, i.e., a `Channel` of size 1 capable of holding objects of `Any` type.
[`Future`]（@ref）によって参照されるリモートオブジェクトは` Any {1} `、すなわち` Any`型のオブジェクトを保持できるサイズ1の `Channel`に格納されます。

#[`RemoteChannel`](@ref), which is rewritable, can point to any type and size of channels, or any other implementation of an `AbstractChannel`.
書き換え可能な[`RemoteChannel`]（@ref）は、任意のタイプとサイズのチャンネル、または` AbstractChannel`の他の実装を指すことができます。

#The constructor `RemoteChannel(f::Function, pid)()` allows us to construct references to channels holding more than one value of a specific type. 
コンストラクタ `RemoteChannel（f :: Function、pid）（）`は、特定の型の複数の値を保持するチャネルへの参照を構築することを可能にします。
#`f()` is a function executed on `pid` and it must return an `AbstractChannel`.
`f（）`は `pid`で実行される関数で、` AbstractChannel`を返さなければなりません。

#For example, `RemoteChannel(()->Channel{Int}(10), pid)`, will return a reference to a channel of type `Int` and size 10. 
たとえば、 `RemoteChannel（（） - > Channel {Int}（10）、pid）`は、 `Int`型とサイズ10のチャネルへの参照を返します。
#The channel exists on worker `pid`.
チャネルは作業者のpidに存在します。

#Methods [`put!()`](@ref), [`take!()`](@ref), [`fetch()`](@ref), [`isready()`](@ref) and [`wait()`](@ref) on a [`RemoteChannel`](@ref) are proxied onto the backing store on the remote process.
（@ ref）、[`is！（）`]（@ ref）、 `` fetch（） ``（@ ref）、[`` isready（） `] [`RemoteChannel`]（@ ref）の[` wait（） `]（@ref）は、リモートプロセスのバッキングストアにプロキシされます。

#[`RemoteChannel`](@ref) can thus be used to refer to user implemented `AbstractChannel` objects.
[`RemoteChannel`]（@ ref）は、ユーザが実装した` AbstractChannel`オブジェクトを参照するために使用できます。
#A simple example of this is provided in `examples/dictchannel.jl` which uses a dictionary as its remote store.
これの簡単な例は `examples / dictchannel.jl`で提供されています。これは辞書をリモートストアとして使用しています。

## Channels and RemoteChannels
##チャネルとリモートチャネル

   * 
   # A [`Channel`](@ref) is local to a process. 
    [`Channel`]（@ ref）はプロセスにとってローカルです。
   # Worker 2 cannot directly refer to a `Channel` on worker 3 and vice-versa. 
    作業者2は、作業者3の「チャネル」を直接参照することはできず、その逆も可能である。
   # A [`RemoteChannel`](@ref), however, can put and take values across workers.
    しかし、[`` RemoteChannel`]（@ ref）は、ワーカー間で値を設定したり取り込んだりすることができます。
   * 
   # A [`RemoteChannel`](@ref) can be thought of as a *handle* to a `Channel`.
   [`RemoteChannel`]（@ ref）は` Channel`への*ハンドル*と考えることができます。
   * 
   # The process id, `pid`, associated with a [`RemoteChannel`](@ref) identifies the process where the backing store, i.e., the backing `Channel` exists.
   [`RemoteChannel`]（@ ref）に関連付けられたプロセスid、` pid`は、バッキングストア、つまりバッキング `Channel`が存在するプロセスを識別します。
   * 
   # Any process with a reference to a [`RemoteChannel`](@ref) can put and take items from the channel.
   [`RemoteChannel`]（@ ref）への参照を持つプロセスはすべてチャンネルから項目を出し入れできます。
   # Data is automatically sent to (or retrieved from) the process a [`RemoteChannel`](@ref) is associated with.
    データは、[`RemoteChannel`]（@ ref）が関連付けられているプロセスに自動的に送信（または取得）されます。
   * 
   # Serializing  a `Channel` also serializes any data present in the channel. 
    `Channel`をシリアライズすると、そのチャンネルに存在するデータもシリアライズされます。
   # Deserializing it therefore effectively makes a copy of the original object.
    したがって、それを逆シリアル化すると、効果的に元のオブジェクトのコピーが作成されます。
   * 
   # On the other hand, serializing a [`RemoteChannel`](@ref) only involves the serialization of an identifier that identifies the location and instance of `Channel` referred to by the handle. 
   一方、[`RemoteChannel`]（@ref）のシリアライズは、ハンドルによって参照される` Channel`の位置とインスタンスを識別する識別子のシリアライズのみを含みます。
   # A deserialized [`RemoteChannel`](@ref) object (on any worker), therefore also points to the same backing store as the original.
   逆シリアル化された[`RemoteChannel`]（@ref）オブジェクトは（ワーカー上で）、元のバッキングストアと同じバッキングストアを指しています。

#The channels example from above can be modified for interprocess communication, as shown below.
上記のチャネルの例は、以下に示すように、プロセス間通信用に変更できます。

#We start 4 workers to process a single `jobs` remote channel. 
1つの `jobs`リモートチャネルを処理するために4人の作業者を開始します。
#Jobs, identified by an id (`job_id`), are written to the channel. 
id（ `job_id`）で識別されるジョブがチャネルに書き込まれます。
#Each remotely executing task in this simulation reads a `job_id`, waits for a random amount of time and writes back a tuple of `job_id`, time taken and its own `pid` to the results channel. 
このシミュレーションでは、リモートで実行されている各タスクは `job_id`を読み取り、ランダムな時間だけ待機し、` job_id`のタプル、取得した時間と自身の `pid`を結果チャンネルに書き戻します。
#Finally all the `results` are printed out on the master process.
最後に、すべての結果がマスタープロセスに出力されます。

```julia-repl
julia> addprocs(4); # add worker processes

julia> const jobs = RemoteChannel(()->Channel{Int}(32));

julia> const results = RemoteChannel(()->Channel{Tuple}(32));

julia> @everywhere function do_work(jobs, results) # define work function everywhere
           while true
               job_id = take!(jobs)
               exec_time = rand()
               sleep(exec_time) # simulates elapsed time doing actual work
               put!(results, (job_id, exec_time, myid()))
           end
       end

julia> function make_jobs(n)
           for i in 1:n
               put!(jobs, i)
           end
       end;

julia> n = 12;

julia> @schedule make_jobs(n); # feed the jobs channel with "n" jobs

julia> for p in workers() # start tasks on the workers to process requests in parallel
           remote_do(do_work, p, jobs, results)
       end

julia> @elapsed while n > 0 # print out results
           job_id, exec_time, where = take!(results)
           println("$job_id finished in $(round(exec_time,2)) seconds on worker $where")
           n = n - 1
       end
1 finished in 0.18 seconds on worker 4
2 finished in 0.26 seconds on worker 5
6 finished in 0.12 seconds on worker 4
7 finished in 0.18 seconds on worker 4
5 finished in 0.35 seconds on worker 5
4 finished in 0.68 seconds on worker 2
3 finished in 0.73 seconds on worker 3
11 finished in 0.01 seconds on worker 3
12 finished in 0.02 seconds on worker 3
9 finished in 0.26 seconds on worker 5
8 finished in 0.57 seconds on worker 4
10 finished in 0.58 seconds on worker 2
0.055971741
```

## Remote References and Distributed Garbage Collection
##リモート参照と分散ガベージコレクション

#Objects referred to by remote references can be freed only when *all* held references in the cluster are deleted.
リモート参照で参照されるオブジェクトは、クラスタ内の*すべての保持参照が削除された場合にのみ解放できます。

#The node where the value is stored keeps track of which of the workers have a reference to it.
値が格納されているノードは、どの従業員に参照があるかを追跡します。
#Every time a [`RemoteChannel`](@ref) or a (unfetched) [`Future`](@ref) is serialized to a worker, the node pointed to by the reference is notified. 
[`RemoteChannel`]（@ref）または（未取得）[` Future`]（@ref）がワーカーにシリアライズされるたびに、参照によって指し示されるノードが通知されます。
#And every time a [`RemoteChannel`](@ref) or a (unfetched) [`Future`](@ref) is garbage collected locally, the node owning the value is again notified.
[`RemoteChannel`]（@ref）または（未取得）[` Future`]（@ ref）がローカルでガベージコレクトされるたびに、その値を所有するノードに再び通知されます。

#The notifications are done via sending of "tracking" messages--an "add reference" message when a reference is serialized to a different process and a "delete reference" message when a reference is locally garbage collected.
通知は、トラッキングメッセージを送信することによって行われます。参照が別のプロセスにシリアル化されたときは「参照の追加」メッセージ、参照がローカルでガベージコレクションされるときは「参照の削除」メッセージです。

#Since [`Future`](@ref)s are write-once and cached locally, the act of [`fetch()`](@ref)ing a [`Future`](@ref) also updates reference tracking information on the node owning the value.
`` Future``（@ ref）はライトワンスでローカルにキャッシュされているので、[`Future`]（@ ref）を実行する[` fetch（） `]（@ref）そのノードはその値を所有しています。

#The node which owns the value frees it once all references to it are cleared.
値を所有しているノードは、そのノードへのすべての参照がクリアされるとその値を解放します。

#With [`Future`](@ref)s, serializing an already fetched [`Future`](@ref) to a different node also sends the value since the original remote store may have collected the value by this time.
`` Future``（@ ref）では、すでにフェッチされた[`Future`]（@ ref）を別のノードにシリアライズすると、元のリモートストアがこの時間までに値を収集した可能性があるため、値を送ります。

#It is important to note that *when* an object is locally garbage collected depends on the size of the object and the current memory pressure in the system.
オブジェクトがローカルでガベージコレクトされたとき*は、オブジェクトのサイズとシステム内の現在のメモリ圧に依存することに注意することが重要です。

#In case of remote references, the size of the local reference object is quite small, while the value stored on the remote node may be quite large. 
リモート参照の場合、ローカル参照オブジェクトのサイズは非常に小さいのに対して、リモートノードに格納された値はかなり大きい場合があります。
#Since the local object may not be collected immediately, it is a good practice to explicitly call [`finalize()`](@ref) on local instances of a [`RemoteChannel`](@ref), or on unfetched [`Future`](@ref)s. 
ローカルオブジェクトはすぐには収集されない可能性があるので、[`RemoteChannel`]（@ ref）のローカルインスタンス、または未完成の[` Future `]（@ ref）s。
#Since calling [`fetch()`](@ref) on a [`Future`](@ref) also removes its reference from the remote store, this is not required on fetched [`Future`](@ref)s. 
[`Future`]（@ref）で[` fetch（） `]（@ref）を呼び出すと、リモートストアから参照が削除されるので、フェッチされた[` Future`]（@ ref）では不要です。
#Explicitly calling [`finalize()`](@ref) results in an immediate message sent to the remote node to go ahead and remove its reference to the value.
明示的に[`finalize（）`]（@ ref）を呼び出すと、リモートノードに送信された即時メッセージが返され、その値への参照が削除されます。

#Once finalized, a reference becomes invalid and cannot be used in any further calls.
ファイナライズが完了すると、参照は無効になり、その後の呼び出しでは使用できなくなります。

## [Shared Arrays](@id man-shared-arrays)
## [共有配列]（@ idのman-shared-arrays）

#Shared Arrays use system shared memory to map the same array across many processes. 
共有アレイは、システム共有メモリを使用して、同じアレイを多くのプロセスにマップします。
#While there are some similarities to a [`DArray`](https://github.com/JuliaParallel/DistributedArrays.jl), the behavior of a [`SharedArray`](@ref) is quite different. 
[`DArray`]（https://github.com/JuliaParallel/DistributedArrays.jl）にはいくつかの類似点がありますが、[` SharedArray`]（@ ref）の動作は全く異なります。
#In a [`DArray`](https://github.com/JuliaParallel/DistributedArrays.jl), each process has local access to just a chunk of the data, and no two processes share the same chunk; in contrast, in a [`SharedArray`](@ref) each "participating" process has access to the entire array.  
[`DArray`]（https://github.com/JuliaParallel/DistributedArrays.jl）では、各プロセスはデータのちょうどへのローカルアクセスを持ち、2つのプロセスが同じチャンクを共有することはありません。対照的に、[`SharedArray`]（@ref）では、各"参加 "プロセスは配列全体にアクセスできます。
#A [`SharedArray`](@ref) is a good choice when you want to have a large amount of data jointly accessible to two or more processes on the same machine.
[`SharedArray`]（@ ref）は、同じマシン上の2つ以上のプロセスに大量のデータを共同でアクセスさせたい場合に適しています。

#[`SharedArray`](@ref) indexing (assignment and accessing values) works just as with regular arrays, and is efficient because the underlying memory is available to the local process. 
[`SharedArray`]（@ref）インデックス付け（代入とアクセスの値）は、通常の配列と同様に動作し、ローカルのプロセスで使用できるメモリがあるため効率的です。
#Therefore, most algorithms work naturally on [`SharedArray`](@ref)s, albeit in single-process mode. 
したがって、ほとんどのアルゴリズムはシングルプロセスモードではあるが、[SharedArray`]（@ ref）で自然に動作します。
#In cases where an algorithm insists on an [`Array`](@ref) input, the underlying array can be retrieved from a [`SharedArray`](@ref) by calling [`sdata()`](@ref). 
アルゴリズムが[`Array`]（@ ref）入力を主張する場合、[` sdata（） `]（@ ref）を呼び出すことによって、基礎配列を[` SharedArray`]（@ ref）から取り出すことができます。
#For other `AbstractArray` types, [`sdata()`](@ref) just returns the object itself, so it's safe to use [`sdata()`](@ref) on any `Array`-type object.
他の `AbstractArray`型の場合、[` sdata（） `]（@ ref）はオブジェクト自体を返すだけなので、` `Array`-typeオブジェクトに対して[` sdata（） `]（@ref）を使うことは安全です。

#The constructor for a shared array is of the form:
共有配列のコンストラクタの形式は次のとおりです。

```julia
SharedArray{T,N}(dims::NTuple; init=false, pids=Int[])
```

#which creates an `N`-dimensional shared array of a bits type `T` and size `dims` across the processes specified by `pids`. 
これは、 `pids 'で指定されたプロセスの間でビットタイプ` T`とサイズ `dims`の` N`次元共有配列を作成します。
#Unlike distributed arrays, a shared array is accessible only from those participating workers specified by the `pids` named argument (and the creating process too, if it is on the same host).
分散配列とは異なり、共有配列は、 `pids`という名前の引数で指定された参加ワーカー（および同じホスト上にある場合は作成プロセス）からのみアクセスできます。

#If an `init` function, of signature `initfn(S::SharedArray)`, is specified, it is called on all the participating workers. 
シグネチャ `initfn（S :: SharedArray）`の `init`関数が指定されている場合、それはすべての参加するワーカーに対して呼び出されます。
#You can specify that each worker runs the `init` function on a distinct portion of the array, thereby parallelizing initialization.
各ワーカーが配列の別個の部分で `init`関数を実行するように指定して、初期化を並列化することができます。

#Here's a brief example:
以下に簡単な例を示します。

```julia-repl
julia> addprocs(3)
3-element Array{Int64,1}:
 2
 3
 4

julia> S = SharedArray{Int,2}((3,4), init = S -> S[Base.localindexes(S)] = myid())
3×4 SharedArray{Int64,2}:
 2  2  3  4
 2  3  3  4
 2  3  4  4

julia> S[3,2] = 7
7

julia> S
3×4 SharedArray{Int64,2}:
 2  2  3  4
 2  3  3  4
 2  7  4  4
```

#[`Base.localindexes()`](@ref) provides disjoint one-dimensional ranges of indexes, and is sometimes convenient for splitting up tasks among processes. 
[`Base.localindexes（）`]（@ref）は、一意の1次元インデックス範囲を提供し、時にはプロセス間でタスクを分割するのに便利です。
#You can, of course, divide the work any way you wish:
もちろん、作品をあなたが望むように分けることができます：

```julia-repl
julia> S = SharedArray{Int,2}((3,4), init = S -> S[indexpids(S):length(procs(S)):length(S)] = myid())
3×4 SharedArray{Int64,2}:
 2  2  2  2
 3  3  3  3
 4  4  4  4
```

#Since all processes have access to the underlying data, you do have to be careful not to set up conflicts. 
すべてのプロセスが基礎となるデータにアクセスできるため、競合を設定しないように注意する必要があります。
#For example:
例えば：

```julia
@sync begin
    for p in procs(S)
        @async begin
            remotecall_wait(fill!, p, S, p)
        end
    end
end
```

#would result in undefined behavior. Because each process fills the *entire* array with its own `pid`, whichever process is the last to execute (for any particular element of `S`) will have its `pid` retained.
定義されていない動作になります。 各プロセスは*全体の配列を独自の `pid`で満たすので、最後に実行されるプロセス（` S`の特定の要素に対して）は `pid`を保持します。

#As a more extended and complex example, consider running the following "kernel" in parallel:
より拡張された複雑な例として、以下の「カーネル」を並行して実行することを検討してください。

```julia
q[i,j,t+1] = q[i,j,t] + u[i,j,t]
```

#In this case, if we try to split up the work using a one-dimensional index, we are likely to run into trouble: if `q[i,j,t]` is near the end of the block assigned to one worker and `q[i,j,t+1]` is near the beginning of the block assigned to another, it's very likely that `q[i,j,t]` will not be ready at the time it's needed for computing `q[i,j,t+1]`. 
この場合、1次元のインデックスを使用して作業を分割しようとすると、問題が発生する可能性が高くなります。q [i、j、t]が1人の作業者に割り当てられたブロックの終わりに近く、 `q [i、j、t + 1]`が別のブロックに割り当てられたブロックの先頭に近い場合、 `q [i、j、t] 'が` qを計算するために必要な時には、 [i、j、t + 1] `となる。
#In such cases, one is better off chunking the array manually. 
そのような場合は、アレイを手動でチャンクするほうがよいでしょう。
#Let's split along the second dimension. 
2番目の次元に沿って分割しましょう。
#Define a function that returns the `(irange, jrange)` indexes assigned to this worker:
このワーカーに割り当てられた `（irange、jrange）`インデックスを返す関数を定義します：

```julia-repl
julia> @everywhere function myrange(q::SharedArray)
           idx = indexpids(q)
           if idx == 0 # This worker is not assigned a piece
               return 1:0, 1:0
           end
           nchunks = length(procs(q))
           splits = [round(Int, s) for s in linspace(0,size(q,2),nchunks+1)]
           1:size(q,1), splits[idx]+1:splits[idx+1]
       end
```

#Next, define the kernel:
次に、カーネルを定義します。

```julia-repl
julia> @everywhere function advection_chunk!(q, u, irange, jrange, trange)
           @show (irange, jrange, trange)  # display so we can see what's happening
           for t in trange, j in jrange, i in irange
               q[i,j,t+1] = q[i,j,t] + u[i,j,t]
           end
           q
       end
```

#We also define a convenience wrapper for a `SharedArray` implementation
`SharedArray`実装のための便利なラッパーも定義します

```julia-repl
julia> @everywhere advection_shared_chunk!(q, u) =
           advection_chunk!(q, u, myrange(q)..., 1:size(q,3)-1)
```

#Now let's compare three different versions, one that runs in a single process:
ここでは、1つのプロセスで実行される3つの異なるバージョンを比較してみましょう。

```julia-repl
julia> advection_serial!(q, u) = advection_chunk!(q, u, 1:size(q,1), 1:size(q,2), 1:size(q,3)-1);
```

#one that uses [`@parallel`](@ref):
[`@ parallel`]（@ ref）を使うもの：

```julia-repl
julia> function advection_parallel!(q, u)
           for t = 1:size(q,3)-1
               @sync @parallel for j = 1:size(q,2)
                   for i = 1:size(q,1)
                       q[i,j,t+1]= q[i,j,t] + u[i,j,t]
                   end
               end
           end
           q
       end;
```

#and one that delegates in chunks:
チャンクでデリゲートするもの：

```julia-repl
julia> function advection_shared!(q, u)
           @sync begin
               for p in procs(q)
                   @async remotecall_wait(advection_shared_chunk!, p, q, u)
               end
           end
           q
       end;
```

#If we create `SharedArray`s and time these functions, we get the following results (with `julia -p 4`):
`SharedArray`を作成してこれらの関数を実行すると、次の結果が得られます（` julia -p 4`）。

```julia-repl
julia> q = SharedArray{Float64,3}((500,500,500));

julia> u = SharedArray{Float64,3}((500,500,500));
```

#Run the functions once to JIT-compile and [`@time`](@ref) them on the second run:
関数を一度JITコンパイルして実行し、2回目の実行で関数を[`@ time`]（@ ref）します：

```julia-repl
julia> @time advection_serial!(q, u);
(irange,jrange,trange) = (1:500,1:500,1:499)
 830.220 milliseconds (216 allocations: 13820 bytes)

julia> @time advection_parallel!(q, u);
   2.495 seconds      (3999 k allocations: 289 MB, 2.09% gc time)

julia> @time advection_shared!(q,u);
        From worker 2:       (irange,jrange,trange) = (1:500,1:125,1:499)
        From worker 4:       (irange,jrange,trange) = (1:500,251:375,1:499)
        From worker 3:       (irange,jrange,trange) = (1:500,126:250,1:499)
        From worker 5:       (irange,jrange,trange) = (1:500,376:500,1:499)
 238.119 milliseconds (2264 allocations: 169 KB)
```

#The biggest advantage of `advection_shared!` is that it minimizes traffic among the workers, allowing each to compute for an extended time on the assigned piece.
`advection_shared！ 'の最大の利点は、ワーカー間のトラフィックを最小限に抑え、割り当てられた部分の長時間の計算を可能にすることです。

### Shared Arrays and Distributed Garbage Collection
##共有配列と分散ガベージコレクション

#Like remote references, shared arrays are also dependent on garbage collection on the creating node to release references from all participating workers. 
リモート参照と同様に、共有配列も、作成ノード上のガベージコレクションに依存して、参加しているすべてのワーカーからの参照を解放します。
#Code which creates many short lived shared array objects would benefit from explicitly finalizing these objects as soon as possible.
多くの短命の共有配列オブジェクトを作成するコードは、できるだけ早くこれらのオブジェクトを明示的にファイナライズすることで利益を得ます。
#This results in both memory and file handles mapping the shared segment being released sooner.
これにより、共有セグメントをマッピングするメモリとファイルハンドルの両方がより早く解放されます。

### ClusterManagers
## ClusterManagers

#The launching, management and networking of Julia processes into a logical cluster is done via cluster managers. 
ジュリアプロセスの論理クラスタへの立ち上げ、管理、およびネットワーキングは、クラスタマネージャを介して行われます。
#A `ClusterManager` is responsible for
`ClusterManager`は、
   * 
   # launching worker processes in a cluster environment
   クラスタ環境でのワーカープロセスの起動
   * 
   # managing events during the lifetime of each worker
   各ワーカの生涯のイベント管理
   * 
   # optionally, providing data transport
   オプションで、データ転送を提供する

#A Julia cluster has the following characteristics:
Juliaクラスタの特徴は次のとおりです。

   * 
   # The initial Julia process, also called the `master`, is special and has an `id` of 1.
    `master`とも呼ばれる最初のJuliaプロセスは特別で、` id`が1です。
   * 
   # Only the `master` process can add or remove worker processes.
    `master`プロセスだけがワーカープロセスを追加または削除できます。
   * 
   # All processes can directly communicate with each other.
    すべてのプロセスが互いに直接通信できます。

#Connections between workers (using the in-built TCP/IP transport) is established in the following manner:
作業者間の接続（組み込みのTCP / IP転送を使用）は、次の方法で確立されます。

   * 
   # [`addprocs()`](@ref) is called on the master process with a `ClusterManager` object.
   [`addprocs（）`]（@ ref）は `ClusterManager`オブジェクトでマスタプロセス上で呼び出されます。
   * 
   # [`addprocs()`](@ref) calls the appropriate [`launch()`](@ref) method which spawns required number of worker processes on appropriate machines.
   [`addprocs（）`]（@ref）は適切なマシン上で必要な数のワーカープロセスを生成する適切な[`launch（）`]（@ ref）メソッドを呼び出します。
   * 
   # Each worker starts listening on a free port and writes out its host and port information to [`STDOUT`](@ref).
   各ワーカーは空いているポートでリッスンを開始し、そのホストとポート情報を[`STDOUT`]（@ ref）に書き出します。
   * 
   # The cluster manager captures the [`STDOUT`](@ref) of each worker and makes it available to the master process.
   クラスタマネージャは、各作業者の[`STDOUT`]（@ ref）を取得し、それをマスタプロセスが利用できるようにします。
   * 
   # The master process parses this information and sets up TCP/IP connections to each worker.
   マスタープロセスはこの情報を解析し、各ワーカーにTCP/IP接続を設定します。
   * 
   # Every worker is also notified of other workers in the cluster.
   すべてのワーカーにはクラスタ内の他のワーカーも通知されます。
   * 
   # Each worker connects to all workers whose `id` is less than the worker's own `id`.
   各作業者は、「id」が自分の「id」よりも小さいすべての作業者に接続します。
   * 
   # In this way a mesh network is established, wherein every worker is directly connected with every other worker.
   このようにして、すべての作業者が他のすべての作業者と直接接続されたメッシュネットワークが確立される。

#While the default transport layer uses plain `TCPSocket`, it is possible for a Julia cluster to provide its own transport.
デフォルトのトランスポート層は単純な `TCPSocket`を使用しますが、Juliaクラスタは独自のトランスポートを提供することができます。

#Julia provides two in-built cluster managers:
Juliaには、2つの組み込みクラスタマネージャがあります。
   * 
   # `LocalManager`, used when [`addprocs()`](@ref) or [`addprocs(np::Integer)`](@ref) are called
   [`addprocs（）`]（@ ref）または[`addprocs（np :: Integer）`]（@ ref）が呼び出されたときに使われる `LocalManager`
   * 
   # `SSHManager`, used when [`addprocs(hostnames::Array)`](@ref) is called with a list of hostnames
    `` addprocs（hostnames :: Array） `]（@ ref）がホスト名のリストと共に呼び出されたときに使われる` SSHManager`

#`LocalManager` is used to launch additional workers on the same host, thereby leveraging multi-core and multi-processor hardware.
LocalManagerは、同じホスト上で追加のワーカーを起動するために使用され、マルチコアとマルチプロセッサのハードウェアを活用します。

#Thus, a minimal cluster manager would need to:
したがって、最小限のクラスタ・マネージャは、以下を行う必要があります。
   * 
   # be a subtype of the abstract `ClusterManager`
    抽象クラス「ClusterManager」のサブタイプにする
   * 
   # implement [`launch()`](@ref), a method responsible for launching new workers
    新しいワーカーを起動する方法である[`launch（）`]（@ ref）を実装する
   * 
   # implement [`manage()`](@ref), which is called at various events during a worker's lifetime (for example, sending an interrupt signal)
    作業者の生存期間中にさまざまなイベントで呼び出される（例えば、割り込み信号を送信する）[manage（） `]（@ ref）

#[`addprocs(manager::FooManager)`](@ref addprocs) requires `FooManager` to implement:
[`addprocs（manager :: FooManager）`]（@ ref addprocs）では `FooManager`を実装する必要があります：

```julia
function launch(manager::FooManager, params::Dict, launched::Array, c::Condition)
    [...]
end

function manage(manager::FooManager, id::Integer, config::WorkerConfig, op::Symbol)
    [...]
end
```

#As an example let us see how the `LocalManager`, the manager responsible for starting workers on the same host, is implemented:
例として、同じホスト上でワーカーを開始する責任を負うマネージャーLocalManagerがどのように実装されているかを見てみましょう。

```julia
struct LocalManager <: ClusterManager
    np::Integer
end

function launch(manager::LocalManager, params::Dict, launched::Array, c::Condition)
    [...]
end

function manage(manager::LocalManager, id::Integer, config::WorkerConfig, op::Symbol)
    [...]
end
```

#The [`launch()`](@ref) method takes the following arguments:
[`launch（）`]（@ ref）メソッドは以下の引数をとります：

  * 
    `manager::ClusterManager`: the cluster manager that [`addprocs()`](@ref) is called with
    `manager :: ClusterManager`：[` addprocs（） `]（@ ref）が呼び出されたクラスタマネージャ
  * 
    `params::Dict`: all the keyword arguments passed to [`addprocs()`](@ref)
    `params :: Dict`：[addprocs（）`]（@ ref）に渡されたすべてのキーワード引数
  * 
    `launched::Array`: the array to append one or more `WorkerConfig` objects to
    `launch :: Array`：1つ以上の` WorkerConfig`オブジェクトを追加する配列
  * 
    `c::Condition`: the condition variable to be notified as and when workers are launched
    `c :: Condition`：ワーカーの起動時に通知される条件変数

#The [`launch()`](@ref) method is called asynchronously in a separate task. 
[`launch（）`]（@ ref）メソッドは別のタスクで非同期に呼び出されます。
#The termination of this task signals that all requested workers have been launched. 
このタスクが終了すると、要求されたすべてのワーカーが起動したことが通知されます。
#Hence the [`launch()`](@ref) function MUST exit as soon as all the requested workers have been launched.
したがって、要求されたすべてのワーカーが起動するとすぐに、[`launch（）`]（@ ref）関数を終了しなければなりません。

#Newly launched workers are connected to each other and the master process in an all-to-all manner.
新たに立ち上げられた労働者は、お互いに、そしてマスタープロセスは、すべての方法で結ばれています。
#Specifying the command line argument `--worker[=<cookie>]` results in the launched processes initializing themselves as workers and connections being set up via TCP/IP sockets.
コマンドライン引数 `--worker [= <cookie>]`を指定すると、起動されたプロセスはTCP / IPソケットを介して設定されたワーカーと接続として初期化されます。

#All workers in a cluster share the same [cookie](#cluster-cookie) as the master. 
クラスタ内のすべてのワーカーは、マスターと同じ[cookie]（＃cluster-cookie）を共有します。
#When the cookie is unspecified, i.e, with the `--worker` option, the worker tries to read it from its standard input.
Cookieが指定されていない場合、つまり `--worker`オプションを指定すると、ワーカーは標準入力からCookieを読み込もうとします。
#`LocalManager` and `SSHManager` both pass the cookie to newly launched workers via their standard inputs.
`LocalManager`と` SSHManager`は、両方とも標準入力を介して新しく起動したワーカーにクッキーを渡します。

#By default a worker will listen on a free port at the address returned by a call to `getipaddr()`.
デフォルトでは、ワーカーは `getipaddr（）`の呼び出しによって返されたアドレスの空いているポートを待ち受けます。
#A specific address to listen on may be specified by optional argument `--bind-to bind_addr[:port]`.
listenする特定のアドレスはオプションの引数 `--bind-to bind_addr [：port]`で指定することができます。
#This is useful for multi-homed hosts.
これは、マルチホームホストに便利です。

#As an example of a non-TCP/IP transport, an implementation may choose to use MPI, in which case `--worker` must NOT be specified. 
非TCP / IPトランスポートの例として、実装はMPIの使用を選択することができます。この場合、 `--worker`を指定してはいけません。
#Instead, newly launched workers should call `init_worker(cookie)` before using any of the parallel constructs.
代わりに、新しく起動されたワーカーは、並列構造のいずれかを使用する前に `init_worker（cookie）`を呼び出す必要があります。

#For every worker launched, the [`launch()`](@ref) method must add a `WorkerConfig` object (with appropriate fields initialized) to `launched`
すべてのワーカーが起動すると、[`launch（）`]（@ ref）メソッドは ``起動された ``にWorkerConfig`オブジェクトを（適切なフィールドを初期化して）追加する必要があります。

```julia
mutable struct WorkerConfig
    # Common fields relevant to all cluster managers
    io::Nullable{IO}
    host::Nullable{AbstractString}
    port::Nullable{Integer}

    # Used when launching additional workers at a host
    count::Nullable{Union{Int, Symbol}}
    exename::Nullable{AbstractString}
    exeflags::Nullable{Cmd}

    # External cluster managers can use this to store information at a per-worker level
    # Can be a dict if multiple fields need to be stored.
    userdata::Nullable{Any}

    # SSHManager / SSH tunnel connections to workers
    tunnel::Nullable{Bool}
    bind_addr::Nullable{AbstractString}
    sshflags::Nullable{Cmd}
    max_parallel::Nullable{Integer}

    connect_at::Nullable{Any}

    [...]
end
```

#Most of the fields in `WorkerConfig` are used by the inbuilt managers. 
`WorkerConfig`のフィールドのほとんどは、ビルドされたマネージャによって使用されます。
#Custom cluster managers would typically specify only `io` or `host` / `port`:
カスタムクラスタマネージャは通常、 `io`や` host` / `port`だけを指定します：

   * 
   #If `io` is specified, it is used to read host/port information. 
    `io`を指定すると、ホスト/ポート情報の読み込みに使用されます。
   #A Julia worker prints out its bind address and port at startup. 
    Juliaワーカーは起動時にバインドアドレスとポートを表示します。
   #This allows Julia workers to listen on any free port available instead of requiring worker ports to be configured manually.
    これにより、Juliaワーカーは、ワーカーポートを手動で設定する必要はなく、使用可能な空きポートで待機することができます。
   * 
   #If `io` is not specified, `host` and `port` are used to connect.
    `io`を指定しないと、` host`と `port`が接続に使用されます。
   * 
   #`count`, `exename` and `exeflags` are relevant for launching additional workers from a worker.
   `count`、` exename`と `exeflags`は、追加のワーカーをワーカーから起動するのに関係します。
   #For example, a cluster manager may launch a single worker per node, and use that to launch additional workers.
   たとえば、クラスタマネージャはノードごとに1人のワーカーを起動し、それを使用して追加のワーカーを起動することができます。

       * 
       #`count` with an integer value `n` will launch a total of `n` workers.
        整数値 `n`で` count`を実行すると `n`個の作業者が起動します。
       * 
       #`count` with a value of `:auto` will launch as many workers as the number of cores on that machine.
        `：auto 'の値を持つ` count`は、そのマシン上のコアの数だけ多くのワーカを起動します。
       * 
       #`exename` is the name of the `julia` executable including the full path.
        `exename`はフルパスを含む` julia`実行可能ファイルの名前です。
       * 
       #`exeflags` should be set to the required command line arguments for new workers.
        `exeflags`は、新しいワーカーのために必要なコマンドライン引数に設定する必要があります。
   * 
   #`tunnel`, `bind_addr`, `sshflags` and `max_parallel` are used when a ssh tunnel is required to connect to the workers from the master process.
   `tunnel`、` bind_addr`、 `sshflags`、` max_parallel`は、マスタプロセスからワーカーに接続するためにsshトンネルが必要な場合に使用されます。
   * 
   #`userdata` is provided for custom cluster managers to store their own worker-specific information.
   `userdata`はカスタムクラスタ管理者が独自のワーカー固有情報を保存するために用意されています。

#`manage(manager::FooManager, id::Integer, config::WorkerConfig, op::Symbol)` is called at different times during the worker's lifetime with appropriate `op` values:
`manage（manager :: FooManager、id :: Integer、config :: WorkerConfig、op :: Symbol）`はワーカーの生存期間中に適切な `op`値で呼び出されます：
   * 
   #with `:register`/`:deregister` when a worker is added / removed from the Julia worker pool.
   *
    Juliaワーカープールにワーカーが追加/削除されたときに `：register` /`：deregister`を実行します。
   * 
   #with `:interrupt` when `interrupt(workers)` is called. The `ClusterManager` should signal the appropriate worker with an interrupt signal.
   `interrupt（workers）`が呼び出されたときに `：interrupt`と呼ばれます。 `ClusterManager`は、適切な作業者に割り込み信号を知らせます。
   * 
   #with `:finalize` for cleanup purposes.
    `：finalize`をクリーンアップの目的で使用します。

## Cluster Managers with Custom Transports
##カスタムトランスポートを持つクラスタマネージャ

#Replacing the default TCP/IP all-to-all socket connections with a custom transport layer is a little more involved. 
デフォルトのTCP / IP all-to-allソケット接続をカスタムトランスポートレイヤーに置き換えることはもう少し複雑です。
#Each Julia process has as many communication tasks as the workers it is connected to. 
各ジュリアプロセスには、接続先の従業員と同数の通信タスクがあります。
#For example, consider a Julia cluster of 32 processes in an all-to-all mesh network:
たとえば、all-to-allメッシュ・ネットワークで32プロセスのジュリア・クラスタを考えてみましょう。

   * 
   #Each Julia process thus has 31 communication tasks.
    したがって、各ジュリアプロセスには31の通信タスクがあります。
   * 
   #Each task handles all incoming messages from a single remote worker in a message-processing loop.
    各タスクは、メッセージ処理ループ内の単一のリモートワーカーからのすべての着信メッセージを処理します。
   * 
   #The message-processing loop waits on an `IO` object (for example, a `TCPSocket` in the default implementation), reads an entire message, processes it and waits for the next one.
   メッセージ処理ループは、 `IO`オブジェクト（例えば、デフォルトの実装では` TCPSocket`）を待ち、メッセージ全体を読み込み、それを処理して次のメッセージを待ちます。
   * 
   #Sending messages to a process is done directly from any Julia task--not just communication tasks--again, via the appropriate `IO` object.
   プロセスへのメッセージの送信は、通信タスクだけでなく、適切な `IO`オブジェクトを介して、ジュリアタスクから直接実行されます。

#Replacing the default transport requires the new implementation to set up connections to remote workers and to provide appropriate `IO` objects that the message-processing loops can wait on.
デフォルトのトランスポートを置き換えるには、新しい実装がリモートワーカーへの接続をセットアップし、メッセージ処理ループが待機できる適切な `IO`オブジェクトを提供する必要があります。
#The manager-specific callbacks to be implemented are:
実装されるマネージャ固有のコールバックは次のとおりです。

```julia
connect(manager::FooManager, pid::Integer, config::WorkerConfig)
kill(manager::FooManager, pid::Int, config::WorkerConfig)
```

#The default implementation (which uses TCP/IP sockets) is implemented as `connect(manager::ClusterManager, pid::Integer, config::WorkerConfig)`.
デフォルトの実装（TCP / IPソケットを使用する）は `connect（manager :: ClusterManager、pid :: Integer、config :: WorkerConfig）`として実装されています。

#`connect` should return a pair of `IO` objects, one for reading data sent from worker `pid`, and the other to write data that needs to be sent to worker `pid`. 
`connect`はワーカー` pid`から送られたデータを読み込むためのものと、ワーカー `pid`に送られる必要のあるデータを書き込むためのものです。
#Custom cluster managers can use an in-memory `BufferStream` as the plumbing to proxy data between the custom, possibly non-`IO` transport and Julia's in-built parallel infrastructure.
カスタム・クラスタ・マネージャは、カスタムの、おそらくは「IO」でないトランスポートとJuliaの組み込み並列インフラストラクチャとの間でデータをプロキシするための配管として、メモリ内の「BufferStream」を使用することができる。

#A `BufferStream` is an in-memory `IOBuffer` which behaves like an `IO`--it is a stream which can be handled asynchronously.
`BufferStream`は` IO`のように振る舞うインメモリ `IOBuffer`です。これは非同期に扱うことができるストリームです。

#Folder `examples/clustermanager/0mq` contains an example of using ZeroMQ to connect Julia workers in a star topology with a 0MQ broker in the middle. 
`examples / clustermanager / 0mq`フォルダには、ZeroMQを使用してスタートポロジ内のJuliaワーカーと0MQブローカを途中で接続する例が含まれています。
#Note: The Julia processes are still all *logically* connected to each other--any worker can message any other worker directly without any awareness of 0MQ being used as the transport layer.
注：Juliaプロセスは、すべて論理的に*接続されています.0MQがトランスポート層として使用されていることを意識することなく、他のワーカーに直接メッセージを送ることができます。

#When using custom transports:
カスタムトランスポートを使用する場合：

   * 
   #Julia workers must NOT be started with `--worker`. 
    Juliaワーカーは `--worker`で始めるべきではありません。
   #Starting with `--worker` will result in the newly launched workers defaulting to the TCP/IP socket transport implementation.
    `--worker`で始まると、新しく起動されたワーカーはTCP / IPソケット転送実装にデフォルト設定されます。
   * 
   #For every incoming logical connection with a worker, `Base.process_messages(rd::IO, wr::IO)()` must be called. 
    ワーカーとの着信論理接続ごとに、 `Base.process_messages（rd :: IO、wr :: IO）（）`を呼び出さなければなりません。
   #This launches a new task that handles reading and writing of messages from/to the worker represented by the `IO` objects.
    これは、 `IO`オブジェクトによって表される作業者から/へのメッセージの読み書きを処理する新しいタスクを起動します。
   * 
   #`init_worker(cookie, manager::FooManager)` MUST be called as part of worker process initialization.
    `init_worker（cookie、manager :: FooManager）`はワーカープロセス初期化の一部として呼び出さなければなりません。
   * 
   #Field `connect_at::Any` in `WorkerConfig` can be set by the cluster manager when [`launch()`](@ref) is called. 
   [`launch（）`]（@ ref）が呼び出されると、 `WorkerConfig`の` connect_at :: Any`フィールドはクラスタマネージャによって設定されます。
   #The value of this field is passed in in all [`connect()`](@ref) callbacks. 
   このフィールドの値はすべての[`connect（）`]（@ ref）コールバックで渡されます。
   #Typically, it carries information on *how to connect* to a worker. 
    典型的には、*作業者に*接続する方法に関する情報を持ちます。
   #For example, the TCP/IP socket transport uses this field to specify the `(host, port)` tuple at which to connect to a worker.
   たとえば、TCP / IPソケット転送では、このフィールドを使用してワーカーに接続するタプル（ホスト、ポート）を指定します。

#`kill(manager, pid, config)` is called to remove a worker from the cluster. 
`kill（manager、pid、config）`は、クラスタからワーカーを削除するために呼び出されます。
#On the master process, the corresponding `IO` objects must be closed by the implementation to ensure proper cleanup.
マスタープロセスでは、適切なクリーンアップを確実にするために、対応する `IO`オブジェクトを実装によって閉じなければなりません。
#The default implementation simply executes an `exit()` call on the specified remote worker.
デフォルトの実装は、指定されたリモートワーカーで `exit（）`コールを実行するだけです。

#`examples/clustermanager/simple` is an example that shows a simple implementation using UNIX domain sockets for cluster setup.
`examples / clustermanager / simple`は、クラスタセットアップのためにUNIXドメインソケットを使用する単純な実装を示す例です。

## Network Requirements for LocalManager and SSHManager
## LocalManagerとSSHManagerのネットワーク要件

#Julia clusters are designed to be executed on already secured environments on infrastructure such as local laptops, departmental clusters, or even the cloud. 
Juliaクラスタは、ローカルのラップトップ、部門クラスタ、クラウドなどのインフラストラクチャ上の既に保護された環境で実行されるように設計されています。
#This section covers network security requirements for the inbuilt `LocalManager` and `SSHManager`:
このセクションでは、組み込みの `LocalManager`と` SSHManager`のネットワークセキュリティ要件について説明します：

   * 
   #The master process does not listen on any port. It only connects out to the workers.
   マスタープロセスはどのポートでもリッスンしません。それは労働者にしかつながりません。
   * 
   #Each worker binds to only one of the local interfaces and listens on an ephemeral port number assigned by the OS.
   各ワーカーは、ローカルインタフェースの1つのみにバインドし、OSによって割り当てられた一時的なポート番号をリッスンします。
   * 
   #`LocalManager`, used by `addprocs(N)`, by default binds only to the loopback interface. 
   `addprocs（N）`によって使用される `LocalManager`は、デフォルトではループバックインタフェースにのみバインドされます。
   #This means that workers started later on remote hosts (or by anyone with malicious intentions) are unable to connect to the cluster. 
   これは、後でリモートホスト（または悪意のある人物）によって開始されたワーカーがクラスタに接続できないことを意味します。
   #An `addprocs(4)` followed by an `addprocs(["remote_host"])` will fail.
   `addprocs（4）`に続けて `addprocs（[" remote_host "]）`が失敗します。
   #Some users may need to create a cluster comprising their local system and a few remote systems.
   一部のユーザーは、ローカルシステムといくつかのリモートシステムを含むクラスタを作成する必要があります。
   #This can be done by explicitly requesting `LocalManager` to bind to an external network interface via the `restrict` keyword argument: `addprocs(4; restrict=false)`.
   これは、 `restrict`キーワード引数を介して外部ネットワークインターフェースにバインドする` LocalManager`を明示的に要求することによって行うことができます： `addprocs（4; restrict = false）`。
   * 
   #`SSHManager`, used by `addprocs(list_of_remote_hosts)`, launches workers on remote hosts via SSH.
   `addprocs（list_of_remote_hosts）`によって使用される `SSHManager`は、SSH経由でリモートホスト上のワーカーを起動します。
   #By default SSH is only used to launch Julia workers. 
   デフォルトでは、SSHはJuliaワーカーの起動にのみ使用されます。
   #Subsequent master-worker and worker-worker connections use plain, unencrypted TCP/IP sockets. 
   後続のマスタワーカーとワーカー/ワーカの接続では、暗号化されていない単純なTCP / IPソケットが使用されます。
   #The remote hosts must have passwordless login enabled.
   リモートホストでパスワードなしログインが有効になっている必要があります。
   #Additional SSH flags or credentials may be specified via keyword argument `sshflags`.
   追加のSSHフラグや信用証明書は、キーワード引数 `sshflags`で指定することができます。
   * 
   #`addprocs(list_of_remote_hosts; tunnel=true, sshflags=<ssh keys and other flags>)` is useful when we wish to use SSH connections for master-worker too. 
   `addprocs（list_of_remote_hosts; tunnel = true、sshflags = <ssh keysと他のフラグ>）は、マスターワーカーにもSSH接続を使いたいときに便利です。
   #A typical scenario for this is a local laptop running the Julia REPL (i.e., the master) with the rest of the cluster on the cloud, say on Amazon EC2. 
   これに対する典型的なシナリオは、Julia REPL（すなわち、マスター）をクラウド上のクラスタの他の部分、例えばAmazon EC2上で実行しているローカルラップトップです。
   #In this case only port 22 needs to be opened at the remote cluster coupled with SSH client authenticated via public key infrastructure (PKI). 
   この場合、公開鍵インフラストラクチャ（PKI）を介して認証されたSSHクライアントと結合したリモートクラスタでは、ポート22のみを開く必要があります。
   #Authentication credentials can be supplied via `sshflags`, for example ```sshflags=`-e <keyfile>` ```.
   認証資格証明は、 `` `sshflags =` -e <keyfile> `` ``のような `` sshflags ''を通して供給することができます。

   * 
   #In an all-to-all topology (the default), all workers connect to each other via plain TCP sockets.
   オールトゥオールトポロジ（デフォルト）では、すべてのワーカーが単純なTCPソケットを介して相互に接続します。
   #The security policy on the cluster nodes must thus ensure free connectivity between workers for the ephemeral port range (varies by OS).
   したがって、クラスタノードのセキュリティポリシーは、エフェメラルポート範囲（OSによって異なる）のためのワーカー間の自由な接続性を保証する必要があります。
   * 
   #Securing and encrypting all worker-worker traffic (via SSH) or encrypting individual messages can be done via a custom ClusterManager.
   SSHによるすべてのワーカー・ワーカー・トラフィックの保護と暗号化、または個々のメッセージの暗号化は、カスタムClusterManagerを使用して実行できます。

## Cluster Cookie
##クラスタクッキー

#All processes in a cluster share the same cookie which, by default, is a randomly generated string on the master process:
クラスタ内のすべてのプロセスは、同じクッキーを共有します。これは、デフォルトでマスタプロセス上でランダムに生成された文字列です。

   * 
   #[`Base.cluster_cookie()`](@ref) returns the cookie, while `Base.cluster_cookie(cookie)()` sets it and returns the new cookie.
   [`Base.cluster_cookie（）`]（@ref）はクッキーを返し、 `Base.cluster_cookie（cookie）（）`はそれを設定して新しいクッキーを返します。
   * 
   #All connections are authenticated on both sides to ensure that only workers started by the master are allowed to connect to each other.
   すべての接続が両側で認証され、マスタによって開始されたワーカーのみが互いに接続できるようになります。
   * 
   #The cookie may be passed to the workers at startup via argument `--worker=<cookie>`. 
   クッキーは、起動時に `--worker = <cookie> '引数を介してワーカーに渡されます。
   #If argument `--worker` is specified without the cookie, the worker tries to read the cookie from its standard input (STDIN). 
   クッキーなしで引数 `--worker`が指定された場合、ワーカーは標準入力（STDIN）からクッキーを読み込もうとします。
   #The STDIN is closed immediately after the cookie is retrieved.
   STDINは、クッキーが取得された直後に閉じられます。
   * 
   #ClusterManagers can retrieve the cookie on the master by calling [`Base.cluster_cookie()`](@ref).
   ClusterManagersは[`Base.cluster_cookie（）`]（@ ref）を呼び出してマスター上のクッキーを取得できます。
   #Cluster managers not using the default TCP/IP transport (and hence not specifying `--worker`) must call `init_worker(cookie, manager)` with the same cookie as on the master.
   デフォルトTCP / IP転送を使用しない（したがって `--worker`を指定しない）クラスタ管理者は、マスターと同じクッキーで` init_worker（cookie、manager） `を呼び出さなければなりません。

#Note that environments requiring higher levels of security can implement this via a custom `ClusterManager`.
より高いレベルのセキュリティを必要とする環境は、カスタム `ClusterManager`を介してこれを実装できることに注意してください。
#For example, cookies can be pre-shared and hence not specified as a startup argument.
たとえば、クッキーはあらかじめ共有することができ、したがって起動引数として指定されません。

## Specifying Network Topology (Experimental)
##ネットワークトポロジの指定（実験的）

#The keyword argument `topology` passed to `addprocs` is used to specify how the workers must be connected to each other:
`addprocs`に渡されるキーワード引数` topology`は、ワーカー同士の接続方法を指定するために使用されます。

   * 
   #`:all_to_all`, the default: all workers are connected to each other.
     `：all_to_all`、デフォルト：すべてのワーカーが互いに接続されています。
   * 
   #`:master_slave`: only the driver process, i.e. `pid` 1, has connections to the workers.
     `：master_slave`：ドライバプロセス、つまり` pid` 1のみがワーカーに接続しています。
   * 
   #`:custom`: the `launch` method of the cluster manager specifies the connection topology via the fields `ident` and `connect_idents` in `WorkerConfig`. 
   `：custom`：クラスタマネージャの` launch`メソッドは `WorkerConfig`の` ident`と `connect_idents`フィールドを使って接続トポロジを指定します。
   #A worker with a cluster-manager-provided identity `ident` will connect to all workers specified in `connect_idents`.
   クラスタマネージャが提供するID「ident」を持つワーカーは、 `connect_ident`で指定されたすべてのワーカーに接続します。

#Currently, sending a message between unconnected workers results in an error. 
現在、未接続のワーカー間でメッセージを送信するとエラーが発生します。
#This behaviour, as with the functionality and interface, should be considered experimental in nature and may change in future releases.
この動作は、機能やインターフェイスの場合と同様に、実際には実験的なものと見なされ、将来のリリースで変更される可能性があります。

## Multi-Threading (Experimental)
##マルチスレッド（実験的）

#In addition to tasks, remote calls, and remote references, Julia from `v0.5` forwards will natively support multi-threading. 
タスク、リモートコール、およびリモート参照に加えて、 `v0.5`フォワードからのJuliaは、ネイティブにマルチスレッドをサポートします。
#Note that this section is experimental and the interfaces may change in the future.
このセクションは実験的なものであり、将来インターフェイスが変更される可能性があることに注意してください。

### Setup
### セットアップ

#By default, Julia starts up with a single thread of execution. 
デフォルトでは、Juliaは単一の実行スレッドで起動します。
#This can be verified by using the command [`Threads.nthreads()`](@ref):
これは[`Threads.nthreads（）`]（@ ref）コマンドを使って確認することができます：

```julia-repl
julia> Threads.nthreads()
1
```

#The number of threads Julia starts up with is controlled by an environment variable called `JULIA_NUM_THREADS`. 
Juliaが起動するスレッドの数は、 `JULIA_NUM_THREADS`という環境変数によって制御されます。
#Now, let's start up Julia with 4 threads:
今、4つのスレッドでJuliaを起動しましょう：

```bash
export JULIA_NUM_THREADS=4
```

#(The above command works on bourne shells on Linux and OSX. 
（上記のコマンドは、LinuxおよびOSXのbourneシェルで動作します。
#Note that if you're using a C shell on these platforms, you should use the keyword `set` instead of `export`. 
これらのプラットフォームでCシェルを使用している場合は、 `export`の代わりにキーワード` set`を使うべきです。
#If you're on Windows, start up the command line in the location of `julia.exe` and use `set` instead of `export`.)
Windowsの場合は、 `julia.exe`の場所でコマンドラインを起動し、` export`の代わりに `set`を使います。

#Let's verify there are 4 threads at our disposal.
私たちが自由に使えるスレッドが4つあることを確認しましょう。

```julia-repl
julia> Threads.nthreads()
4
```

#But we are currently on the master thread. To check, we use the command [`Threads.threadid()`](@ref)
しかし、我々は現在、マスタースレッドにあります。 確認するには、コマンド `` Threads.threadid（） `]（@ ref）を使用します。

```julia-repl
julia> Threads.threadid()
1
```

### The `@threads` Macro
### `@ threads`マクロ


#Let's work a simple example using our native threads. 
ネイティブスレッドを使用した簡単な例を試してみましょう。
#Let us create an array of zeros:
一連のゼロを作成しましょう：

```jldoctest
julia> a = zeros(10)
10-element Array{Float64,1}:
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
```

#Let us operate on this array simultaneously using 4 threads. 
この配列を同時に4つのスレッドで操作してみましょう。
#We'll have each thread write its thread ID into each location.
各スレッドはスレッドIDを各場所に書き込ませます。

#Julia supports parallel loops using the [`Threads.@threads`](@ref) macro. 
Juliaは[`Threads。@ threads`]（@ ref）マクロを使って並列ループをサポートしています。
#This macro is affixed in front of a `for` loop to indicate to Julia that the loop is a multi-threaded region:
このマクロは、ループがマルチスレッド領域であることをJuliaに示す `for`ループの前に付いています：

```julia-repl
julia> Threads.@threads for i = 1:10
           a[i] = Threads.threadid()
       end
```

#The iteration space is split amongst the threads, after which each thread writes its thread ID to its assigned locations:
反復空間はスレッド間で分割され、各スレッドは割り当てられた位置にスレッドIDを書き込みます。

```julia-repl
julia> a
10-element Array{Float64,1}:
 1.0
 1.0
 1.0
 2.0
 2.0
 2.0
 3.0
 3.0
 4.0
 4.0
```

#Note that [`Threads.@threads`](@ref) does not have an optional reduction parameter like [`@parallel`](@ref).
[`Threads。@ threads`]（@ref）には[` @ parallel`]（@ ref）のような省略可能な縮小パラメータはありません。

## @threadcall (Experimental)
## @threadcall（実験的）

#All I/O tasks, timers, REPL commands, etc are multiplexed onto a single OS thread via an event loop. A patched version of libuv ([http://docs.libuv.org/en/v1.x/](http://docs.libuv.org/en/v1.x/)) provides this functionality. 
すべてのI / Oタスク、タイマー、REPLコマンドなどは、イベントループを介して単一のOSスレッドに多重化されます。 libuv（[http://docs.libuv.org/en/v1.x/](http://docs.libuv.org/ja/v1.x/）]のパッチ版では、この機能が提供されています。
#Yield points provide for co-operatively scheduling multiple tasks onto the same OS thread. 
歩留まりポイントは、複数のタスクを同じOSスレッドに協調的にスケジューリングする機能を提供します。
#I/O tasks and timers yield implicitly while waiting for the event to occur. 
I / Oタスクとタイマーは、イベントが発生するのを待つ間に暗黙的に発生します。
#Calling [`yield()`](@ref) explicitly allows for other tasks to be scheduled.
[`yield（）`]（@ ref）を明示的に呼び出すと、ほかのタスクをスケジュールすることができます。

#Thus, a task executing a [`ccall`](@ref) effectively prevents the Julia scheduler from executing any other tasks till the call returns. 
したがって、[`ccall`]（@ ref）を実行するタスクは、Juliaスケジューラがコールが戻るまで他のタスクを実行することを効果的に防ぎます。
#This is true for all calls into external libraries. 
これは、外部ライブラリへのすべての呼び出しに当てはまります。
#Exceptions are calls into custom C code that call back into Julia (which may then yield) or C code that calls `jl_yield()` (C equivalent of [`yield()`](@ref)).
例外は、Julia（それはyieldすることがあります）または `jl_yield（）`を呼び出すCコード（Cの[`yield（）`]（@ ref））をコールするカスタムCコードへの呼び出しです。

#Note that while Julia code runs on a single thread (by default), libraries used by Julia may launch their own internal threads. 
Juliaコードは単一スレッド（デフォルト）で実行されますが、Juliaが使用するライブラリは独自の内部スレッドを起動する可能性があります。
#For example, the BLAS library may start as many threads as there are cores on a machine.
たとえば、BLASライブラリは、マシン上にコアがあるものと同じ数のスレッドを開始することができます。

#The `@threadcall` macro addresses scenarios where we do not want a `ccall` to block the main Julia event loop. 
`@ threadcall`マクロは` ccall`がメインのJuliaイベントループをブロックしないようにするシナリオを扱います。
#It schedules a C function for execution in a separate thread. 
別のスレッドで実行するためのC関数をスケジュールします。
#A threadpool with a default size of 4 is used for this. The size of the threadpool is controlled via environment variable `UV_THREADPOOL_SIZE`. 
これには、デフォルトサイズ4のスレッドプールが使用されます。スレッドプールのサイズは、環境変数 `UV_THREADPOOL_SIZE`によって制御されます。
#While waiting for a free thread, and during function execution once a thread is available, the requesting task (on the main Julia event loop) yields to other tasks. 
フリーのスレッドを待つ間、そしてスレッドが利用可能になった後に関数が実行されている間は、（メインのJuliaイベントループ上の）要求タスクは他のタスクに帰着します。
#Note that `@threadcall` does not return till the execution is complete. 
`@ threadcall`は実行が完了するまで戻りません。
#From a user point of view, it is therefore a blocking call like other Julia APIs.
ユーザーの視点から見ると、それは他のジュリアAPIと同様にブロッキングコールです。

#It is very important that the called function does not call back into Julia.
呼び出された関数がJuliaにコールバックしないことは非常に重要です。

#`@threadcall` may be removed/changed in future versions of Julia.
`@ threadcall`はJuliaの将来のバージョンで削除/変更されるかもしれません。

[^1]:
   #In this context, MPI refers to the MPI-1 standard. 
   この文脈において、MPIはMPI-1標準を指す。
   #Beginning with MPI-2, the MPI standards committee introduced a new set of communication mechanisms, collectively referred to as Remote Memory Access (RMA). 
    MPI標準委員会は、MPI-2から、リモートメモリアクセス（RMA）と総称される新しい通信メカニズムを導入しました。
   #The motivation for adding RMA to the MPI standard was to facilitate one-sided communication patterns. 
    MPI標準にRMAを追加する動機は、片面通信パターンを容易にすることでした。
   #For additional information on the latest MPI standard, see [http://mpi-forum.org/docs](http://mpi-forum.org/docs/). 
    最新のMPI標準に関する追加情報については、[http://mpi-forum.org/docs](http://mpi-forum.org/docs/）を参照してください。
