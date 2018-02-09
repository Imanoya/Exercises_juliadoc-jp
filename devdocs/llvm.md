# Working with LLVM

This is not a replacement for the LLVM documentation, but a collection of tips for working on LLVM for Julia.
> これはLLVMのドキュメントの代わりではなく、JuliaのためのLLVMで作業するためのヒント集です。

## Overview of Julia to LLVM Interface

Julia dynamically links against LLVM by default.
> JuliaはデフォルトでLLVMと動的にリンクします。
Build with `USE_LLVM_SHLIB=0` to link statically.
> 静的にリンクするには `USE_LLVM_SHLIB = 0`でビルドしてください。

The code for lowering Julia AST to LLVM IR or interpreting it directly is in directory `src/`.
> Julia ASTをLLVM IRに下げたり直接解釈するためのコードは `src /`ディレクトリにあります。

| File                | Description                                                |
|:------------------- |:---------------------------------------------------------- |
| `builtins.c`        | Builtin functions                                          |
| `ccall.cpp`         | Lowering `ccall`                                           |
| `cgutils.cpp`       | Lowering utilities, notably for array and tuple accesses   |
| `codegen.cpp`       | Top-level of code generation, pass list, lowering builtins |
| `debuginfo.cpp`     | Tracks debug information for JIT code                      |
| `disasm.cpp`        | Handles native object file and JIT code diassembly         |
| `gf.c`              | Generic functions                                          |
| `intrinsics.cpp`    | Lowering intrinsics                                        |
| `llvm-simdloop.cpp` | Custom LLVM pass for `@simd`                               |
| `sys.c`             | I/O and operating system utility functions                 |

Some of the `.cpp` files form a group that compile to a single object.
> `.cpp` ファイルの中には、単一のオブジェクトにコンパイルされるグループを形成するものがあります。

The difference between an intrinsic and a builtin is that a builtin is a first class function that can be used like any other Julia function.
> 組み込み関数と組み込み関数の違いは、組み込み関数が他のどのようなJulia関数と同様に使用できるファーストクラスの関数であることです。
An intrinsic can operate only on unboxed data, and therefore its arguments must be statically typed.
> 組み込み関数はボックス化されていないデータに対してのみ動作できるため、その引数は静的に型指定する必要があります。

### Alias Analysis

Julia currently uses LLVM's [Type Based Alias Analysis](http://llvm.org/docs/LangRef.html#tbaa-metadata).
> Juliaは現在、LLVMの[Type Based Alias Analysis](http://llvm.org/docs/LangRef.html#tbaa-metadata)を使用しています。
To find the comments that document the inclusion relationships, look for `static MDNode*` in `src/codegen.cpp`.
> インクルード関係を文書化したコメントを見つけるには、 `src / codegen.cpp`で` static MDNode * `を探してください。

The `-O` option enables LLVM's [Basic Alias Analysis](http://llvm.org/docs/AliasAnalysis.html#the-basicaa-pass).
> `-O` オプションは、LLVMの[Basic Alias Analysis](http://llvm.org/docs/AliasAnalysis.html#the-basicaa-pass)を有効にします。

## Building Julia with a different version of LLVM

The default version of LLVM is specified in `deps/Versions.make`.
> LLVMのデフォルトバージョンは `deps / Versions.make`で指定されています。
You can override it by creating a file called `Make.user` in the top-level directory and adding a line to it such as:
> 最上位のディレクトリに `Make.user`という名前のファイルを作成し、次のような行を追加することで上書きすることができます：

```
LLVM_VER = 3.5.0
```

Besides the LLVM release numerals, you can also use `LLVM_VER = svn` to bulid against the latest development version of LLVM.
> LLVMのリリース番号のほかに、 `LLVM_VER = svn` を使って、最新の開発版のLLVMを検証することもできます。

## Passing options to LLVM

You can pass options to LLVM using *debug* builds of Julia. To create a debug build, run `make debug`.
> Juliaの* debug * buildsを使ってLLVMにオプションを渡すことができます。 デバッグビルドを作成するには、 `make debug` を実行します。
The resulting executable is `usr/bin/julia-debug`.
> 実行される実行ファイルは `usr/bin/julia-debug` です。
You can pass LLVM options to this executable via the environment variable `JULIA_LLVM_ARGS`.
> LLVMオプションは環境変数 `JULIA_LLVM_ARGS` を介してこの実行可能ファイルに渡すことができます。
Here are example settings using `bash` syntax:
> `bash`構文を使った設定の例を以下に示します：

  * `export JULIA_LLVM_ARGS = -print-after-all` dumps IR after each pass.
  * > `JULIA_LLVM_ARGS = -print-after-all`は、各パスの後にIRをダンプします。
  * `export JULIA_LLVM_ARGS = -debug-only=loop-vectorize` dumps LLVM `DEBUG(...)` diagnostics for loop vectorizer *if* you built Julia with `LLVM_ASSERTIONS=1`.
  * > `LLVM_ASSERTIONS = 1` でJuliaをビルドした場合、`export JULIA_LLVM_ARGS = -debug-only = loop-vectorize` はloop vectorizer *のため*のLLVM `DEBUG(...)` 診断をダンプします。
  Otherwise you will get warnings about "Unknown command line argument".
  > それ以外の場合は、 "Unknown command line argument"に関する警告が表示されます。
  Counter-intuitively, building Julia with `LLVM_DEBUG=1` is *not* enough to dump `DEBUG` diagnostics from a pass.
  > Counter直感的に、 `LLVM_DEBUG = 1` でJuliaを構築することは、 `DEBUG` 診断をパスからダンプするのに*十分*ではありません。

## Debugging LLVM transformations in isolation

On occasion, it can be useful to debug LLVM's transformations in isolation from the rest of the Julia system, e.g. because reproducing the issue inside `julia` would take too long, or because one wants to take advantage of LLVM's tooling (e.g. bugpoint).
> 場合によっては、Juliaシステムの他の部分とは別にLLVMの変換をデバッグすると便利です。 `julia`の中で問題を再現するには時間がかかりすぎるか、LLVMのツール（例えばバグポイント）を利用したいからです。
To get unoptimized IR for the entire system image, pass the `--output-unopt-bc unopt.bc` option to the system image build process, which will output the unoptimized IR to an `unopt.bc` file.
> システムイメージ全体に対して最適化されていないIRを取得するには、 `--output-unopt-bc unopt.bc`オプションをシステムイメージビルドプロセスに渡します。これにより、最適化されていないIRが` unopt.bc`ファイルに出力されます。
This file can then be passed to LLVM tools as usual.
> このファイルは通常どおりLLVMツールに渡すことができます。
`libjulia` can function as an LLVM pass plugin and can be loaded into LLVM tools, to make julia-specific passes available in this environment.
> `libjulia`はLLVMパスプラグインとして機能し、LLVMツールにロードして、この環境でジュリア固有のパスを利用できるようにします。
In addition, it exposes the `-julia` meta-pass, which runs the entire Julia pass-pipeline over the IR.
> また、Juliaパスパイプライン全体をIR上で実行する `-julia`メタパスを公開しています。
As an example, to generate a system image, one could do:
> 一例として、システムイメージを生成するには、次のようにします。

```
opt -load libjulia.so -julia -o opt.bc unopt.bc
llc -o sys.o opt.bc
cc -shared -o sys.so sys.o
```

This system image can then be loaded by `julia` as usual.
> このシステムイメージは通常どおり `julia 'によって読み込まれます。

Alternatively, you can use `--output-jit-bc jit.bc` to obtain a trace of all IR passed to the JIT.
> また、  `--output-jit-bc jit.bc`を使ってJITに渡されたすべてのIRのトレースを取得してください。
This is useful for code that cannot be run as part of the sysimg generation process (e.g. because it creates unserializable state).
> これは、sysimg生成プロセスの一部として実行できないコード（たとえば、直列化不可能な状態を作成するため）に役立ちます。
However, the resulting `jit.bc` does not include sysimage data, and can thus not be used as such.
> しかし、結果として得られる `jit.bc`にはsysimageデータが含まれていないため、そのまま使用することはできません。

It is also possible to dump an LLVM IR module for just one Julia function, using:
> Julia機能のためにLLVM IRモジュールをダンプすることも可能です：

```julia
f, T = +, Tuple{Int,Int} # Substitute your function of interest here
optimize = false
open("plus.ll", "w") do f
    println(f, Base._dump_function(f, T, false, false, false, true, :att, optimize))
end
```
These files can be processed the same way as the unoptimized sysimg IR shown above.
> これらのファイルは上記の最適化されていないsysimg IRと同じ方法で処理できます。

## Improving LLVM optimizations for Julia

Improving LLVM code generation usually involves either changing Julia lowering to be more friendly to LLVM's passes, or improving a pass.
> LLVMのコード生成を改善するには、通常、LLVMのパスよりもやさしくなるようにJuliaを変更するか、パスを改善する必要があります。

If you are planning to improve a pass, be sure to read the [LLVM developer policy](http://llvm.org/docs/DeveloperPolicy.html).
> パスを改善する予定の場合は、[LLVM開発者ポリシー](http://llvm.org/docs/DeveloperPolicy.html)を必ずお読みください。
The best strategy is to create a code example in a form where you can use LLVM's `opt` tool to study it and the pass of interest in isolation.
> 最善の戦略は、LLVMの `opt`ツールを使ってそれを勉強することができ、興味のあるパスを孤立させた形でコード例を作成することです。

1. Create an example Julia code of interest.
> 1.興味のあるジュリアコードを作成します。
2. Use `JULIA_LLVM_ARGS = -print-after-all` to dump the IR.
> 2. `JULIA_LLVM_ARGS = -print-after-all`を使ってIRをダンプします。
3. Pick out the IR at the point just before the pass of interest runs.
> 3. 関心のパスが実行される直前の時点でIRを選択します。
4. Strip the debug metadata and fix up the TBAA metadata by hand.
> 4. デバッグメタデータを取り除き、手動でTBAAメタデータを修正します。

The last step is labor intensive.  Suggestions on a better way would be appreciated.
> 最後のステップは労働集約的です。 より良い方法での提案は高く評価されます。

## The jlcall calling convention

Julia has a generic calling convention for unoptimized code, which looks somewhat as follows:
> Juliaには、最適化されていないコードの汎用呼び出し規約があります。

```c
jl_value_t *any_unoptimized_call(jl_value_t *, jl_value_t **, int);
```
where the first argument is the boxed function object, the second argument is an on-stack array of arguments and the third is the number of arguments.
> 最初の引数はボックス化された関数オブジェクトで、2番目の引数はスタック上の引数の配列で、3番目の引数は引数の数です。
Now, we could perform a straightforward lowering and emit an alloca for the argument array.
> 今度は、単純な引き下げを実行して、引数配列に対してallocaを送出します。
However, this would betray the SSA nature of the uses at the call site, making optimizations (including GC root placement), significantly harder.
> しかし、これはコールサイトでの使用のSSA性質を裏切り、最適化（GCルート配置を含む）を大幅に困難にします。
Instead, we emit it as follows:
> 代わりに、次のように発行します。

```llvm
%bitcast = bitcast @any_unoptimized_call to %jl_value_t *(*)(%jl_value_t *, %jl_value_t *)
call cc 37 %jl_value_t *%bitcast(%jl_value_t *%arg1, %jl_value_t *%arg2)
```

The special `cc 37` annotation marks the fact that this call site is really using the jlcall calling convention.
> 特殊な `cc 37`アノテーションは、このコールサイトが本当にjlcall呼び出し規約を使用しているという事実を示しています。
This allows us to retain the SSA-ness of the uses throughout the optimizer.
> これにより、オプティマイザ全体の使用状況のSSAを保持することができます。
GC root placement will later lower this call to the original C ABI.
> GCルート配置は、後で元のC ABIへのこの呼び出しを下げる。
In the code the calling convention number is represented by the `JLCALL_F_CC` constant.
> コードでは、呼び出し規約番号は `JLCALL_F_CC`定数で表されます。
In addition, there is the `JLCALL_CC` calling convention which functions similarly, but omits the first argument.
> さらに、同様に機能するが、最初の引数は省略した `JLCALL_CC`呼び出し規約があります。

## GC root placement

GC root placement is done by an LLVM pass late in the pass pipeline.
> GCルートの配置は、パスパイプラインの遅れてLLVMパスによって行われます。
Doing GC root placement this late enables LLVM to make more aggressive optimizations around code that requires GC roots, as well as allowing us to reduce the number of required GC roots and GC root store operations (since LLVM doesn't understand our GC, it wouldn't otherwise know what it is and is not allowed to do with values stored to the GC frame, so it'll conservatively do very little).
> 今度はGCルート配置を行うことで、GCルートを必要とするコードの周りでより積極的な最適化を行い、必要なGCルートとGCルートストア操作の数を減らすことができます（LLVMはGCを理解できないため、 さもなければ、GCフレームに格納されている値を使って行うことが許可されていないので、保守的に行うことはほとんどありません）。
As an example, consider an error path
> 例として、エラー・パス

```julia
if some_condition()
    #= Use some variables maybe =#
    error("An error occurred")
end
```
During constant folding, LLVM may discover that the condition is always false, and can remove the basic block.
> 定数フォールディング中、LLVMは条件が常に偽であることを検出し、基本ブロックを削除できます。
However, if GC root lowering is done early, the GC root slots used in the deleted block, as well as any values kept alive in those slots only because they were used in the error path, would be kept alive by LLVM.
> ただし、GCルートの低下が早期に行われた場合、削除されたブロックで使用されたGCルートスロットと、エラーパスで使用されたためにそれらのスロットで有効になっていた値は、LLVMによって保持されます。
By doing GC root lowering late, we give LLVM the license to do any of its usual optimizations (constant folding, dead code elimination, etc.), without having to worry (too much) about which values may or may not be GC tracked.
> 遅くGCルートを下げることによって、GCが追跡される可能性のある値とされない可能性のある値について心配することなく、通常の最適化（一定の折りたたみ、デッドコード除去など）を行うためのライセンスをLLVMに与えます。

However, in order to be able to do late GC root placement, we need to be able to identify a) which pointers are gc tracked and b) all uses of such pointers.
> しかし、GCルートの配置を遅くするために、a）どのポインタがgcであるか、b）そのようなポインタのすべての用途を識別できる必要があります。
The goal of the GC placement pass is thus simple:
> したがって、GC配置パスの目標は簡単です。

Minimize the number of needed GC roots/stores to them subject to the constraint that at every safepoint, any live GC-tracked pointer (i.e. for which there is a path after this point that contains a use of this pointer) is in some GC slot.
> 必要なGCルーツ/ストアの数を最小限に抑え、各セーフポイントで、ライブGCトラッキングされたポインタ（このポインタの使用を含むこのポイントの後のパスがある）があるGCスロットにあるという制約があります 。

### Representation

The primary difficulty is thus choosing an IR representation that allows us to identify GC-tracked pointers and their uses, even after the program has been run through the optimizer.
> したがって、プログラムがオプティマイザを介して実行された後でも、GC追跡ポインタおよびその用途を識別できるIR表現を選択することが第一の難点です。
Our design makes use of three LLVM features to achieve this:
> 私たちの設計では、これを達成するために3つのLLVM機能を使用しています。

- Custom address spaces
> - カスタムアドレススペース
- Operand Bundles
> - オペランドバンドル
- Non-integral pointers
> - 非整数ポインタ

Custom address spaces allow us to tag every point with an integer that needs to be preserved through optimizations.
> カスタムアドレススペースを使用すると、最適化によって保存する必要がある整数をすべてのポイントにタグ付けすることができます。
The compiler may not insert casts between address spaces that did not exist in the original program and it must never change the address space of a pointer on a load/store/etc operation.
> コンパイラは、元のプログラムには存在しなかったアドレス空間の間にキャストを挿入することはできず、ロード/ストア/ etc操作でポインタのアドレス空間を決して変更してはいけません。
This allows us to annotate which pointers are GC-tracked in an optimizer-resistant way.
> これにより、どのポインタがオプティマイザに耐性のない方法でGC追跡されているのか注釈することができます。
Note that metadata would not be able to achieve the same purpose.
> メタデータは同じ目的を達成できないことに注意してください。
Metadata is supposed to always be discardable without altering the semantics of the program.
> メタデータは、プログラムのセマンティクスを変更することなく常に破棄可能であると考えられている。
However, failing to identify a GC-tracked pointer alters the resulting program behavior dramatically - it'll probably crash or return wrong results.
> しかし、GC追跡されたポインタを特定できないと、結果として得られるプログラムの動作が劇的に変わります。おそらくクラッシュしたり、間違った結果を返すことになります。
We currently use three different address spaces (their numbers are defined in `src/codegen_shared.cpp`):
> 現在、3つの異なるアドレス空間を使用しています（その数は `src / codegen_shared.cpp`で定義されています）：

- GC Tracked Pointers (currently 10): These are pointers to boxed values that may be put into a GC frame.
- > GC追跡ポインタ（現在10）：これは、GCフレームに入れることができるボックス化された値へのポインタです。
  It is loosely equivalent to a `jl_value_t*` pointer on the C side.
  >  これは、C側の `jl_value_t *`ポインタに匹敵します。
  N.B. It is illegal to ever have a pointer in this address space that may not be stored to a GC slot.
  > N.B. GCスロットに格納されない可能性のあるこのアドレス空間にポインタを置くのは不正です。
- Derived Pointers (currently 11): These are pointers that are derived from some GC tracked pointer.
- > 派生ポインタ（現在11）：これらは、GC追跡ポインタから派生したポインタです。
  Uses of these pointers generate uses of the original pointer.
  > これらのポインタを使用すると、元のポインタの使用が生成されます。
  However, they need not themselves be known to the GC.
  > しかし、彼ら自身がGCに知られる必要はありません。
  The GC root placement pass MUST always find the GC tracked pointer from which this pointer is derived and use that as the pointer to root.
  > GCルート配置パスは、このポインタが導出されたGC追跡ポインタを常に見つけ、それをルートへのポインタとして使用しなければならない（MUST）。
- Callee Rooted Pointers (currently 12): This is a utility address space to express the notion of a callee rooted value.
- > 呼び出し先ルーテッドポインタ（現在12）：これは呼び出し先の根拠となる値の概念を表現するためのユーティリティーアドレス空間です。
  All values of this address space MUST be storable to a GC root (though it is possible to relax this condition in the future), but unlike the other pointers need not be rooted if passed to a call (they do still need to be rooted if they are live across another safepoint between the definition and the call).
  > このアドレス空間のすべての値は、GCルートに格納可能でなければならない（Mut）ことができなければならない（Mutはこの状態を緩和することは可能である）が、他のポインタとは異なり、コールに渡されれば根を付ける必要はない。彼らは定義と呼び出しの間の別のセーフポイントを通して生きています）。


### Invariants

The GC root placement pass makes use of several invariants, which need to be observed by the frontend and are preserved by the optimizer.
> GCルート配置パスは、いくつかの不変量を利用します。これらの不変量は、フロントエンドによって観察される必要があり、オプティマイザによって保存されます。

First, only the following address space casts are allowed:
> まず、次のアドレス空間キャストのみが許可されます。
- 0->{Tracked,Derived,CalleeRooted}: It is allowable to decay an untracked pointer to any of the others.
- > 0 - > {Tracked、Derived、CalleeRooted}：追跡されていないポインタを他のポインタに壊すことは許されます。
  However, do note that the optimizer has broad license to not root such a value.
  > ただし、オプティマイザには、そのような値に根を付けないための幅広いライセンスがあることに注意してください。
  It is never safe to have a value in address space 0 in any part of the program if it is (or is derived from) a value that requires a GC root.
  > GCルートを必要とする値である（またはそこから派生した）場合、プログラムのどの部分でもアドレス空間0に値を持つことは決して安全ではありません。
- Tracked->Derived: This is the standard decay route for interior values.
- > Tracked-> Derived：内部値の標準的な減衰経路です。
  The placement pass will look for these to identify the base pointer for any use.
  > プレースメント・パスは、使用のためにベース・ポインタを識別するためにこれらを探します。
- Tracked->CalleeRooted: Addrspace CalleeRooted serves merely as a hint that a GC root is not required.
- > Tracked-> CalleeRooted：Addrspace CalleeRootedは、GCルートが必要でないというヒントとしてのみ機能します。
  However, do note that the Derived->CalleeRooted decay is prohibited, since pointers should generally be storable to a GC slot, even in this address space.
  > しかし、ポインタは一般にこのアドレス空間であってもGCスロットに格納可能でなければならないので、Derived-> CalleeRooted減衰は禁止されていることに注意してください。

Now let us consider what constitutes a use:
> さて、使用を構成するものを考えてみましょう。
- Loads whose loaded values is in one of the address spaces
- > ロードされた値がアドレス空間の1つにあるロード
- Stores of a value in one of the address spaces to a location
- > アドレス空間の1つの値のある場所へのストア
- Stores to a pointer in one of the address spaces
- > アドレス空間の1つのポインタへの格納
- Calls for which a value in one of the address spaces is an operand
- > アドレス空間の1つの値がオペランドである呼び出し
- Calls in jlcall ABI, for which the argument array contains a value
- > 引数配列に値が入っているjlcall ABIの呼び出し
- Return instructions.
- > 返品指示。

We explicitly allow load/stores and simple calls in address spaces Tracked/Derived.
> Tracked / Derivedのアドレス空間にロード/ストアとシンプルコールを明示的に許可します。
Elements of jlcall argument arrays must always be in address space Tracked (it is required by the ABI that they are valid `jl_value_t*` pointers).
> jlcall引数配列の要素は、常にアドレス空間のTracked（ABIが有効な `jl_value_t *`ポインタである必要があります）になければなりません。
The same is true for return instructions (though note that struct return arguments are allowed to have any of the address spaces).
> 戻り命令についても同じことが当てはまります（ただし、構造体の戻り引数にはアドレス空間のいずれかがあることに注意してください）。
The only allowable use of an address space CalleeRooted pointer is to pass it to a call (which must have an appropriately typed operand).
> アドレススペースCalleeRootedポインタの唯一の許容される使用は、それを呼び出しに渡すことです（これには、適切に型付けされたオペランドが必要です）。

Further, we disallow `getelementptr` in addrspace Tracked.
> さらに、addrspace Trackedで `getelementptr`を許可しません。
This is because unless the operation is a noop, the resulting pointer will not be validly storable to a GC slot and may thus not be in this address space.
> これは、操作がnoopでない限り、結果のポインタはGCスロットに有効に格納できず、したがってこのアドレス空間に存在しない可能性があるからです。
If such a pointer is required, it should be decayed to addrspace Derived first.
> そのようなポインタが必要な場合は、最初に派生されたaddrspaceに崩壊する必要があります。

Lastly, we disallow `inttoptr`/`ptrtoint` instructions in these address spaces.
> 最後に、これらのアドレス空間で `inttoptr` /` ptrtoint`命令を禁止します。
Having these instructions would mean that some `i64` values are really GC tracked.
> これらの指示があれば、実際にGCトラッキングされている値があることを意味します。
This is problematic, because it breaks that stated requirement that we're able to identify GC-relevant pointers.
> これは、GC関連のポインタを識別できるという上記の要件を破るため、問題があります。
This invariant is accomplished using the LLVM "non-integral pointers" feature, which is new in LLVM 5.0.
> この不変量は、LLVM 5.0の新機能であるLLVMの "非整数ポインタ"機能を使用して実現されます。
It prohibits the optimizer from making optimizations that would introduce these operations.
> オプティマイザがこれらの操作を導入する最適化を行うことを禁止します。
Note we can still insert static constants at JIT time by using `inttoptr` in address space 0 and then decaying to the appropriate address space afterwards.
> アドレス空間0で `inttoptr`を使用し、その後適切なアドレス空間に減衰させることで、JIT時に静的定数を挿入することができます。

### Supporting ccall

One important aspect missing from the discussion so far is the handling of `ccall`.
> これまで議論されていなかった重要な点の1つは、 `ccall`の扱いです。
`ccall` has the peculiar feature that the location and scope of a use do not coincide.
> `ccall`は、使用の場所とスコープが一致しないという独特の特徴を持っています。
As an example consider:
> 例として、

```julia
A = randn(1024)
ccall(:foo, Cvoid, (Ptr{Float64},), A)
```
In lowering, the compiler will insert a conversion from the array to the pointer which drops the reference to the array value.
> 下降すると、コンパイラは配列から配列への参照を削除するポインタへの変換を挿入します。
However, we of course need to make sure that the array does stay alive while we're doing the `ccall`.
> しかし、もちろん、 `ccall`を実行している間は配列が生きていることを確認する必要があります。
To understand how this is done, first recall the lowering of the above code:
> これがどのように行われているかを理解するには、まず上記のコードの低下を思い出してください：

```julia
return $(Expr(:foreigncall, :(:foo), Cvoid, svec(Ptr{Float64}), :(:ccall), 1, :($(Expr(:foreigncall, :(:jl_array_ptr), Ptr{Float64}, svec(Any), :(:ccall), 1, :(A)))), :(A)))
```

The last `:(A)`, is an extra argument list inserted during lowering that informs the code generator which Julia level values need to be kept alive for the duration of this `ccall`.
> 最後の `：（A）`は、この `ccall`の持続時間の間、どのジュリアレベルの値を生かしておく必要があるかをコードジェネレータに知らせる、余分な引数リストです。
We then take this information and represent it in an "operand bundle" at the IR level.
> その後、この情報をIRレベルの「オペランドバンドル」で表現します。
An operand bundle is essentially a fake use that is attached to the call site.
> オペランドバンドルは、本質的にコールサイトに添付される偽の使用です。
At the IR level, this looks like so:
> IRレベルでは、これは次のようになります。


```llvm
call void inttoptr (i64 ... to void (double*)*)(double* %5) [ "jl_roots"(%jl_value_t addrspace(10)* %A) ]
```
The GC root placement pass will treat the `jl_roots` operand bundle as if it were a regular operand.
> GCルート配置パスは `jl_roots`オペランドバンドルを通常のオペランドであるかのように扱います。
However, as a final step, after the GC roots are inserted, it will drop the operand bundle to avoid confusing instruction selection.
> しかし、GCルートを挿入した後の最後のステップとして、命令選択の混乱を避けるためにオペランドバンドルを削除します。

### Supporting pointer_from_objref

`pointer_from_objref` is special because it requires the user to take explicit control of GC rooting.
> `pointer_from_objref`は特別です。なぜなら、ユーザーはGCのルートを明示的に制御する必要があるからです。
By our above invariants, this function is illegal, because it performs an address space cast from 10 to 0.
> 上記の不変量によって、この関数は10から0へのキャストアドレス空間を実行するため、不正です。
However, it can be useful, in certain situations, so we provide a special intrinsic:
> ただし、特定の状況では便利なので、特別な組み込み関数を用意しています。

```llvm
declared %jl_value_t *julia.pointer_from_objref(%jl_value_t addrspace(10)*)
```
which is lowered to the corresponding address space cast after GC root lowering.
> これはGCルートを下げた後にキャストされた対応するアドレス空間に引き下げられます。
Do note however that by using this intrinsic, the caller assumes all responsibility for making sure that the value in question is rooted.
> ただし、この組み込み関数を使用すると、呼び出し元は、問題の値が根づいていることを確認するすべての責任を負うことに注意してください。
Further this intrinsic is not considered a use, so the GC root placement pass will not provide a GC root for the function.
> さらに、この組み込み関数は使用とはみなされないので、GCルート配置パスは関数のGCルートを提供しません。
As a result, the external rooting must be arranged while the value is still tracked by the system. 
> その結果、システムによって値が追跡されている間に、外部ルーテイングを配置する必要があります。
I.e. it is not valid to attempt to use the result of this operation to establish a global root - the optimizer may have already dropped the value.
> この操作の結果を使用してグローバルルートを確立しようとすると、オプティマイザが値をすでにドロップしている可能性があります。

### Keeping values alive in the absence of uses

In certain cases it is necessary to keep an object alive, even though there is no compiler-visible use of said object.
> 場合によっては、前記オブジェクトのコンパイラが目に見える用途がなくても、オブジェクトを生かしておく必要がある。
This may be case for low level code that operates on the memory-representation of an object directly or code that needs to interface with C code.
> これは、オブジェクトのメモリ表現で直接動作する低レベルのコードや、Cコードとのインタフェースが必要なコードの場合があります。
In order to allow this, we provide the following intrinsics at the LLVM level:
> これを可能にするために、LLVMレベルで次の組み込み関数を提供します。

```
token @llvm.julia.gc_preserve_begin(...)
void @llvm.julia.gc_preserve_end(token)
```

(The `llvm.` in the name is required in order to be able to use the `token` type).
> （ `token`型を使うためには` llvm.`が必要です）。
The semantics of these intrinsics are as follows:
> これらの組み込み関数のセマンティクスは次のとおりです。
At any safepoint that is dominated by a `gc_preserve_begin` call, but that is not not dominated by a corresponding `gc_preserve_end` call (i.e. a call whose argument is the token returned by a `gc_preserve_begin` call), the values passed as arguments to that `gc_preserve_begin` will be kept live.
> `gc_preserve_begin`呼び出しが支配しているが、対応する` gc_preserve_end`呼び出し（すなわち、引数が `gc_preserve_begin`呼び出しによって返された呼び出しである）によって支配されていないセーフポイントでは、引数として渡された値 `gc_preserve_begin`は生き続けるでしょう。
Note that the `gc_preserve_begin` still counts as a regular use of those values, so the standard lifetime semantics will ensure that the values will be kept alive before entering the preserve region.
> `gc_preserve_begin`はそれらの値の通常の使用としてカウントされることに注意してください。したがって、標準の生涯セマンティクスは、保存領域に入る前に値が生きていることを保証します。