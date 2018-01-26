<!-- Start -->

# Networking and Streams

<!-- End -->
<!-- Start -->
Julia provides a rich interface to deal with streaming I/O objects such as terminals, pipes and TCP sockets. 
> Juliaは、端末、パイプ、TCPソケットなどのストリーミングI / Oオブジェクトを処理する豊富なインターフェイスを提供します。
<!-- End -->
<!-- Start -->
This interface, though asynchronous at the system level, is presented in a synchronous manner to the programmer and it is usually unnecessary to think about the underlying asynchronous operation. 
> このインタフェースは、システムレベルでは非同期ですが、プログラマに同期として提供されるので、通常は非同期操作について考える必要はありません。
<!-- End -->
<!-- Start -->
This is achieved by making heavy use of Julia cooperative threading ([coroutine](@ref man-tasks)) functionality.
> これは、Julia協調スレッド ([coroutine](@ref man-tasks)) の機能を大量に使用することで実現されます。
<!-- End -->
<!-- Start -->

## Basic Stream I/O

> ## 基本的な I/O Stream 

<!-- End -->
<!-- Start -->
All Julia streams expose at least a [`read()`](@ref) and a [`write()`](@ref) method, taking the stream as their first argument, e.g.:
> すべてのJuliaストリームは、最初の引数としてストリームを取り、少なくとも [`read()`](@ref) メソッドと [`write()`](@ref) メソッドを公開します。
<!-- End -->

```julia-repl
julia> write(STDOUT,"Hello World");  # suppress return value 11 with ;
Hello World
julia> read(STDIN,Char)

'\n': ASCII/Unicode U+000a (category Cc: Other, control)
```

<!-- Start -->
Note that [`write()`](@ref) returns 11, the number of bytes (in `"Hello World"`) written to [`STDOUT`](@ref), but this return value is suppressed with the `;`.
> [`write()`](@ref) は、[`STDOUT`](@ref) に書き込まれたバイト数 ( `Hello World` 中のバイト数 ) 11を返しますが、戻り値は `;` で抑制されます。
<!-- End -->

<!-- Start -->
Here Enter was pressed again so that Julia would read the newline. Now, as you can see from this example, [`write()`](@ref) takes the data to write as its second argument, while [`read()`](@ref) takes the type of the data to be read as the second argument.
> ここで、Enterが再度押されてJuliaが改行を読むようになりました。 この例からわかるように、 [`write()`](@ref) はデータを第2引数として取り込み、 [`read()`](@ref) はデータの型を 2番目の引数として読み取れます。
<!-- End -->

<!-- Start -->
For example, to read a simple byte array, we could do:
> この例のようにして、単純なバイト配列を read することができます。
<!-- End -->

```julia-repl
julia> x = zeros(UInt8, 4)
4-element Array{UInt8,1}:
 0x00
 0x00
 0x00
 0x00

julia> read!(STDIN, x)
abcd
4-element Array{UInt8,1}:
 0x61
 0x62
 0x63
 0x64
```

<!-- Start -->
However, since this is slightly cumbersome, there are several convenience methods provided. 
> しかし、これはやや面倒なので、いくつかの便利な方法があります。
For example, we could have written the above as:
> 例えば、私たちは次のように書くことができます：
<!-- End -->

```julia-repl
julia> read(STDIN,4)
abcd
4-element Array{UInt8,1}:
 0x61
 0x62
 0x63
 0x64
```

<!-- Start -->
or if we had wanted to read the entire line instead:
> 他に、行全体を読みたいと思った場合は、
<!-- End -->

```julia-repl
julia> readline(STDIN)
abcd
"abcd"
```

<!-- Start -->
Note that depending on your terminal settings, your TTY may be line buffered and might thus require an additional enter before the data is sent to Julia.
> 端末の設定によっては、TTYがラインバッファリングされている可能性があります。そのため、データがJuliaに送信される前に追加の入力が必要になる場合があります。
<!-- End -->

<!-- Start -->
To read every line from [`STDIN`](@ref) you can use [`eachline()`](@ref):
> [`STDIN`](@ref) からすべての行を読むには、[`eachline()`](@ref) を使うことができます：
<!-- End -->

```julia
for line in eachline(STDIN)
    print("Found $line")
end
```

<!-- Start -->
or [`read()`](@ref) if you wanted to read by character instead:
> またはキャラクタで読みたい場合は [`read()`](@ref)
<!-- End -->

```julia
while !eof(STDIN)
    x = read(STDIN, Char)
    println("Found: $x")
end
```

<!-- Start -->

## Text I/O

> ## テキスト I/O

<!-- End -->
<!-- Start -->
Note that the [`write()`](@ref) method mentioned above operates on binary streams. 
> 前述の [`write()`](@ref) メソッドはバイナリストリームで動作することに注意してください。
<!-- End -->
<!-- Start -->
In particular, values do not get converted to any canonical text representation but are written out as is:
> 特に、値は正規のテキスト表現に変換されるのではなく、次のように書き出されます。
<!-- End -->

```jldoctest
julia> write(STDOUT,0x61);  # suppress return value 1 with ;
a
```

<!-- Start -->
Note that `a` is written to [`STDOUT`](@ref) by the [`write()`](@ref) function and that the returned value is `1` (since `0x61` is one byte).
> [`write()`](@ref) 関数によって [`STDOUT`](@ref) に `a` が書き込まれ、`0x61` が1バイトであるので返される値は `1` であることに注意してください。
<!-- End -->

<!-- Start -->
For text I/O, use the [`print()`](@ref) or [`show()`](@ref) methods, depending on your needs (see the standard library reference for a detailed discussion of the difference between the two):
> テキスト I/O の場合は、必要に応じて [`print()`](@ref) または [`show()`](@ref) メソッドを使用します（ふたつの違いについて詳細は、標準ライブラリリファレンスを参照してください ）:
<!-- End -->

```jldoctest
julia> print(STDOUT, 0x61)
97
```

<!-- Start -->

## IO Output Contextual Properties

> ## IO出力コンテキストのプロパティ

<!-- End -->
<!-- Start -->
Sometimes IO output can benefit from the ability to pass contextual information into show methods.
> IO出力は、コンテキスト情報をshowメソッドに渡す機能から恩恵を受けることができます。
<!-- End -->
<!-- Start -->
The [`IOContext`](@ref) object provides this framework for associating arbitrary metadata with an IO object.
> [`IOContext`](@ref) オブジェクトは、任意のメタデータをIOオブジェクトに関連付けるためのフレームワークを提供します。
<!-- End -->
<!-- Start -->
For example, [`showcompact`](@ref) adds a hinting parameter to the IO object that the invoked show method should print a shorter output (if applicable).
> たとえば、[`showcompact`](@ref) は、呼び出されたshowメソッドがより短い出力をする必要がある場合、ヒントパラメータをIOオブジェクトに追加します（該当する場合）。
<!-- End -->
<!-- Start -->

## Working with Files

> ## ファイルの操作

<!-- End -->
<!-- Start -->
Like many other environments, Julia has an [`open()`](@ref) function, which takes a filename and returns an `IOStream` object that you can use to read and write things from the file. 
> 他の多くの環境と同様に、Juliaはファイル名をとり、ファイルからの読み書きに使用できる `IOStream` オブジェクトを返す [`open()`](@ref) 関数を持っています。
<!-- End -->
<!-- Start -->
For example if we have a file, `hello.txt`, whose contents are `Hello, World!`:
> 例えば、ファイル `hello.txt` があり、内容が `Hello、World！` の場合：
<!-- End -->

```julia-repl
julia> f = open("hello.txt")
IOStream(<file hello.txt>)

julia> readlines(f)
1-element Array{String,1}:
 "Hello, World!"
```

<!-- Start -->
If you want to write to a file, you can open it with the write (`"w"`) flag:
> ファイルに書き込む場合は、write (`"w"`) フラグを使ってファイルを開くことができます：
<!-- End -->

```julia-repl
julia> f = open("hello.txt","w")
IOStream(<file hello.txt>)

julia> write(f,"Hello again.")
12
```

<!-- Start -->
If you examine the contents of `hello.txt` at this point, you will notice that it is empty; nothing has actually been written to disk yet. 
> この時点で `hello.txt` の内容を調べると、それは空であることがわかります。 実際にディスクには何も書き込まれていません。
<!-- End -->
<!-- Start -->
This is because the `IOStream` must be closed before the write is actually flushed to disk:
> これは、書き込みが実際にディスクにフラッシュされる前に `IOStream` を閉じる必要があるからです：
<!-- End -->

```julia-repl
julia> close(f)
```

<!-- Start -->
Examining `hello.txt` again will show its contents have been changed.
> `hello.txt` をもう一度調べると、その内容が変更されたことが示されます。
<!-- End -->

<!-- Start -->
Opening a file, doing something to its contents, and closing it again is a very common pattern.
> ファイルを開き、内容に何かをしてからもう一度閉じると、非常に一般的なパターンです。
<!-- End -->
<!-- Start -->
To make this easier, there exists another invocation of [`open()`](@ref) which takes a function as its first argument and filename as its second, opens the file, calls the function with the file as an argument, and then closes it again. For example, given a function:
> これをより簡単にするために、最初の引数として関数をとり、ファイル名を2番目のものとして取り上げ、ファイルを開き、ファイルを引数として関数を呼び出す、[`open()`](@ref) それをもう一度閉じます。 例えば、与えられた関数：
<!-- End -->

```julia
function read_and_capitalize(f::IOStream)
    return uppercase(readstring(f))
end
```

<!-- Start -->
You can call:
> 呼び出せます。
<!-- End -->

```julia-repl
julia> open(read_and_capitalize, "hello.txt")
"HELLO AGAIN."
```

<!-- Start -->
to open `hello.txt`, call `read_and_capitalize on it`, close `hello.txt` and return the capitalized contents.
> `hello.txt` を開き、 `read_and_capitalize on it` を呼び出し、 `hello.txt` を閉じて大文字の内容を返します。
<!-- End -->

<!-- Start -->
To avoid even having to define a named function, you can use the `do` syntax, which creates an anonymous function on the fly:
> 名前付き関数を定義しなくても済むようにするには、その場で無名関数を作成する `do` 構文を使うことができます：
<!-- End -->

```julia-repl
julia> open("hello.txt") do f
           uppercase(readstring(f))
       end
"HELLO AGAIN."
```

<!-- Start -->

## A simple TCP example

> ## 単純なTCPの例

<!-- End -->
<!-- Start -->
Let's jump right in with a simple example involving TCP sockets. Let's first create a simple server:
> TCPソケットを使った簡単な例を見てみましょう。 簡単なサーバーを作成しましょう：
<!-- End -->

```julia-repl
julia> @async begin
           server = listen(2000)
           while true
               sock = accept(server)
               println("Hello World\n")
           end
       end
Task (runnable) @0x00007fd31dc11ae0
```

<!-- Start -->
To those familiar with the Unix socket API, the method names will feel familiar, though their usage is somewhat simpler than the raw Unix socket API. 
> UnixソケットAPIに精通している方には、使い慣れていると思われますが、その使用法は未処理のUnixソケットAPIよりもやや簡単です。
<!-- End -->
<!-- Start -->
The first call to [`listen()`](@ref) will create a server waiting for incoming connections on the specified port (2000) in this case. 
> [`listen())`](@ref) の最初の呼び出しは、この場合、指定されたポート（2000）で着信接続を待っているサーバーを作成します。
<!-- End -->
<!-- Start -->
The same function may also be used to create various other kinds of servers:
> 同じ機能を使用して、他のさまざまな種類のサーバーを作成することもできます。
<!-- End -->

```julia-repl
julia> listen(2000) # Listens on localhost:2000 (IPv4)
Base.TCPServer(active)

julia> listen(ip"127.0.0.1",2000) # Equivalent to the first
Base.TCPServer(active)

julia> listen(ip"::1",2000) # Listens on localhost:2000 (IPv6)
Base.TCPServer(active)

julia> listen(IPv4(0),2001) # Listens on port 2001 on all IPv4 interfaces
Base.TCPServer(active)

julia> listen(IPv6(0),2001) # Listens on port 2001 on all IPv6 interfaces
Base.TCPServer(active)

julia> listen("testsocket") # Listens on a UNIX domain socket
Base.PipeServer(active)

julia> listen("\\\\.\\pipe\\testsocket") # Listens on a Windows named pipe
Base.PipeServer(active)
```

<!-- Start -->
Note that the return type of the last invocation is different. 
> 最後の呼び出しの戻り値の型が異なることに注意してください。
<!-- End -->
<!-- Start -->
This is because this server does not listen on TCP, but rather on a named pipe (Windows) or UNIX domain socket. 
> これは、このサーバーがTCPではなく、名前付きパイプ（Windows）またはUNIXドメインソケットでリッスンするためです。
<!-- End -->
<!-- Start -->
Also note that Windows named pipe format has to be a specific pattern such that the name prefix (`\\.\pipe\`) uniquely identifies the [file type](https://msdn.microsoft.com/en- us/library/windows/desktop/aa365783(v=vs.85).aspx). 
> また、Windowsの名前付きパイプフォーマットは、名前接頭辞 (`\\.\pipe\`) が [ファイルタイプ](https://msdn.microsoft.com/en- us/library/windows/desktop/aa365783(v=vs.85).aspx) を一意に識別するような特定のパターンでなければならないことにも注意してください。
<!-- End -->
<!-- Start -->
The difference between TCP and named pipes or UNIX domain sockets is subtle and has to do with the [`accept()`](@ref) and [`connect()`](@ref) methods. 
> TCPと名前付きパイプまたはUNIXドメインソケットの違いは微妙で、[`accept()`](@ref) と [`connect()`](@ref) メソッドと関係があります。
<!-- End -->
<!-- Start -->
The [`accept()`](@ref) method retrieves a connection to the client that is connecting on the server we just created, while the [`connect()`](@ref) function connects to a server using the specified method. 
> [`accept()`](@ref) メソッドは、作成したサーバで接続しているクライアントへの接続を取得し、[`connect()`](@ref) 関数は指定されたメソッドを使用してサーバに接続します。 
<!-- End -->
<!-- Start -->
The [`connect()`](@ref) function takes the same arguments as [`listen()`](@ref), so, assuming the environment (i.e. host, cwd, etc.) is the same you should be able to pass the same arguments to [`connect()`](@ref) as you did to listen to establish the connection. 
> [`connect()`](@ref) 関数は [`listen()`](@ref) と同じ引数を取るので、環境（host、cwdなど）が同じであると仮定すると、同じ引数を [`connect()`](@ref) を呼び出して接続を確立します。
<!-- End -->
<!-- Start -->
So let's try that out (after having created the server above):
> だから、それを試してみましょう（上記のサーバーを作成した後）:
<!-- End -->

```julia-repl
julia> connect(2000)
TCPSocket(open, 0 bytes waiting)

julia> Hello World
```

<!-- Start -->
As expected we saw "Hello World" printed. 
> 予想どおり、 "Hello World" が印刷されていました。
<!-- End -->
<!-- Start -->
So, let's actually analyze what happened behind the scenes. 
> 実際に何が起こったのかを実際に分析しましょう。
<!-- End -->
<!-- Start -->
When we called [`connect()`](@ref), we connect to the server we had just created. 
> 私たちが [`connect()`](@ref) を呼び出すと、私たちはちょうど作成したサーバに接続します。
<!-- End -->
<!-- Start -->
Meanwhile, the accept function returns a server-side connection to the newly created socket and prints "Hello World" to indicate that the connection was successful.
> その間、accept関数は新しく作成されたソケットへのサーバー側接続を返し、接続が成功したことを示すために "Hello World" を出力します。
<!-- End -->

<!-- Start -->
A great strength of Julia is that since the API is exposed synchronously even though the I/O is actually happening asynchronously, we didn't have to worry callbacks or even making sure that the server gets to run.
> Juliaの大きな強みは、 I/O が実際に非同期的に発生していても、API が同期的に公開されているため、コールバックを心配する必要もなく、サーバーが実行されることを確実にする必要もありません。 
<!-- End -->
<!-- Start -->
When we called [`connect()`](@ref) the current task waited for the connection to be established and only continued executing after that was done.
> 私たちが [`connect()`](@ref) を呼び出すと、現在のタスクは接続が確立されるのを待っていました。
<!-- End -->
<!-- Start -->
In this pause, the server task resumed execution (because a connection request was now available), accepted the connection, printed the message and waited for the next client.
> この一時停止中にサーバータスクは実行を再開し（接続要求が利用可能になったため）、接続を受け入れ、メッセージを出力し、次のクライアントを待機します。
<!-- End -->
<!-- Start -->
Reading and writing works in the same way.
> 読み書きは同じように動作します。
<!-- End -->
<!-- Start -->
To see this, consider the following simple echo server:
> これを確認するには、次の単純なエコーサーバーを検討してください。
<!-- End -->

```julia-repl
julia> @async begin
           server = listen(2001)
           while true
               sock = accept(server)
               @async while isopen(sock)
                   write(sock,readline(sock))
               end
           end
       end
Task (runnable) @0x00007fd31dc12e60

julia> clientside = connect(2001)
TCPSocket(RawFD(28) open, 0 bytes waiting)

julia> @async while true
           write(STDOUT,readline(clientside))
       end
Task (runnable) @0x00007fd31dc11870

julia> println(clientside,"Hello World from the Echo Server")
Hello World from the Echo Server
```

<!-- Start -->
As with other streams, use [`close()`](@ref) to disconnect the socket:
> 他のストリームと同様に、 [`close()`](@ref) を使ってソケットを切断してください：
<!-- End -->

```julia-repl
julia> close(clientside)
```

<!-- Start -->

## Resolving IP Addresses

> ## IPアドレスの解決

<!-- End -->
<!-- Start -->
One of the [`connect()`](@ref) methods that does not follow the [`listen()`](@ref) methods is `connect(host::String,port)`, which will attempt to connect to the host given by the `host` parameter on the port given by the port parameter. It allows you to do things like:
> [`listen()`](@ref) メソッドに従わない [`connect()`](@ref) メソッドの一つは portパラメータで指定されたポート上の `host`パラメータによって与えられたホストに接続を試みる `connect(host::String,port)` です。
次のようなことをすることができます：
<!-- End -->

```julia-repl
julia> connect("google.com",80)
TCPSocket(RawFD(30) open, 0 bytes waiting)
```

<!-- Start -->
At the base of this functionality is [`getaddrinfo()`](@ref), which will do the appropriate address resolution:
> この機能の基盤には、適切なアドレス解決を行う [`getaddrinfo()`](@ref) があります：
<!-- End -->

```julia-repl
julia> getaddrinfo("google.com")
ip"74.125.226.225"
```
