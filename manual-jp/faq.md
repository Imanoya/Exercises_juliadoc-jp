# Frequently Asked Questions

<!-- ## Sessions and the REPL-->
##セッションとREPL

<!-- ### How do I delete an object in memory?-->
###メモリ内のオブジェクトを削除するには？

#Julia does not have an analog of MATLAB's `clear` function; once a name is defined in a Julia session (technically, in module `Main`), it is always present.
Juliaには、MATLABの `clear`関数の類推がありません。 Juliaセッション（技術的には `Main`モジュール）で名前が定義されると、それは常に存在します。

<!-- If memory usage is your concern, you can always replace objects with ones that consume less memory.-->
メモリ使用量が懸念される場合は、オブジェクトを常にメモリを消費するオブジェクトに置き換えることができます。
<!-- For example, if `A` is a gigabyte-sized array that you no longer need, you can free the memory with `A = 0`.  -->
たとえば、 `A`が不要なギガバイトサイズの配列である場合、` A = 0`でメモリを解放することができます。
<!-- The memory will be released the next time the garbage collector runs; you can force this to happen with [`gc()`](@ref).-->
ガベージコレクタが次に実行されるときにメモリが解放されます。 これを強制的に[`gc（）`]（@ ref）で行うことができます。

<!-- ### How can I modify the declaration of a type in my session?-->
###どのように私のセッションで型の宣言を変更できますか？

#Perhaps you've defined a type and then realize you need to add a new field.  
おそらくタイプを定義した後、新しいフィールドを追加する必要があると認識しています。
#If you try this at the REPL, you get the error:
REPLでこれを試してみると、エラーが出ます：

```
ERROR: invalid redefinition of constant MyType
```

#Types in module `Main` cannot be redefined.
`Main`モジュールの型を再定義することはできません。

#While this can be inconvenient when you are developing new code, there's an excellent workaround.
これは新しいコードを開発する際には不便ですが、優れた回避策があります。
#Modules can be replaced by redefining them, and so if you wrap all your new code inside a module you can redefine types and constants.  
モジュールはそれらを再定義することで置き換えることができます。したがって、モジュール内に新しいコードをすべてラップすると、型と定数を再定義できます。
#You can't import the type names into `Main` and then expect to be able to redefine them there, but you can use the module name to resolve the scope.  
タイプ名を `Main`にインポートすることはできませんし、そこでそれらを再定義できると期待しますが、モジュール名を使用してスコープを解決することができます。
#In other words, while developing you might use a workflow something like this:
つまり、開発中に次のようなワークフローを使用することができます。

```julia
include("mynewcode.jl")              # this defines a module MyModule
obj1 = MyModule.ObjConstructor(a, b)
obj2 = MyModule.somefunction(obj1)
# Got an error. Change something in "mynewcode.jl"
include("mynewcode.jl")              # reload the module
obj1 = MyModule.ObjConstructor(a, b) # old objects are no longer valid, must reconstruct
obj2 = MyModule.somefunction(obj1)   # this time it worked!
obj3 = MyModule.someotherfunction(obj2, c)
...
```

### Functions
### 機能

###私は関数に引数xを渡し、関数内でそれを修正しましたが、外側では、


### I passed an argument `x` to a function, modified it inside that function, but on the outside,
#the variable `x` is still unchanged. 
変数 `x`はまだ変更されていません。
#Why?
どうして？

#Suppose you call a function like this:
次のような関数を呼び出すとします：

```jldoctest
julia> x = 10
10

julia> function change_value!(y)
           y = 17
       end
change_value! (generic function with 1 method)

julia> change_value!(x)
17

julia> x # x is unchanged!
10
```

#In Julia, the binding of a variable `x` cannot be changed by passing `x` as an argument to a function.
Juliaでは、関数に引数として `x`を渡すことによって、変数` x`の束縛を変更することはできません。
#When calling `change_value!(x)` in the above example, `y` is a newly created variable, bound initially to the value of `x`, i.e. `10`; then `y` is rebound to the constant `17`, while the variable `x` of the outer scope is left untouched.
上の例で `change_value！（x）`を呼び出すと、 `y`は新しく作成された変数で、最初は` x`の値、つまり `10`に束縛されています。 `y`は定数` 17`にリバウンドし、外側スコープの変数 `x`はそのままになります。

#But here is a thing you should pay attention to: suppose `x` is bound to an object of type `Array` (or any other *mutable* type). 
しかし、ここで注意すべきことがあります： `x`が` Array`型（または他の*可変型*）のオブジェクトに束縛されているとします。
#From within the function, you cannot "unbind" `x` from this Array, but you can change its content. 
この関数の中から、この配列から `x`をアンバインドすることはできませんが、その内容を変更することはできます。
#For example:
例えば：

```jldoctest
julia> x = [1,2,3]
3-element Array{Int64,1}:
 1
 2
 3

julia> function change_array!(A)
           A[1] = 5
       end
change_array! (generic function with 1 method)

julia> change_array!(x)
5

julia> x
3-element Array{Int64,1}:
 5
 2
 3
```

#Here we created a function `change_array!()`, that assigns `5` to the first element of the passed array (bound to `x` at the call site, and bound to `A` within the function). 
ここでは、 `change_array！（）`関数を作成しました。この関数は、渡された配列の最初の要素（呼び出し側で `x`に束縛され、関数内で` A`に束縛されています）に `5`を割り当てます。
#Notice that, after the function call, `x` is still bound to the same array, but the content of that array changed: the variables `A` and `x` were distinct bindings refering to the same mutable `Array` object.
関数呼び出しの後、 `x`はまだ同じ配列にバインドされていますが、その配列の内容は変更されています。変数` A`と `x`は同じ変更可能な` Array`オブジェクトを参照する別個のバインディングでした。

#### Can I use `using` or `import` inside a function?
###関数内で `using`または` import`を使うことはできますか？

#No, you are not allowed to have a `using` or `import` statement inside a function.  
いいえ、関数内で `using`または` import`文を使用することはできません。
#If you want to import a module but only use its symbols inside a specific function or set of functions, you
モジュールをインポートするが、そのシンボルを特定の関数または関数のセット内でのみ使用する場合は、
#have two options:
2つのオプションがあります：

#>1. Use `import`:
1. `import`を使用してください：

   ```julia
   import Foo
   function bar(...)
       # ... refer to Foo symbols via Foo.baz ...
   end
   ```

   # This loads the module `Foo` and defines a variable `Foo` that refers to the module, but does not import any of the other symbols from the module into the current namespace.  
   これはモジュール `Foo`をロードし、そのモジュールを参照する変数` Foo`を定義しますが、他のシンボルをモジュールから現在の名前空間にインポートしません。
   # You refer to the `Foo` symbols by their qualified names `Foo.bar` etc.
   `Foo`シンボルは、修飾名` Foo.bar`などで参照します。
#2. Wrap your function in a module:
2. モジュール内で関数をラップします。

   ```julia
   module Bar
   export bar
   using Foo
   function bar(...)
       # ... refer to Foo.baz as simply baz ....
   end
   end
   using Bar
   ```

   # This imports all the symbols from `Foo`, but only inside the module `Bar`.
   これは全てのシンボルを `Foo`からインポートしますが、モジュール` Bar`の中でのみインポートします。

### What does the `...` operator do?
### `...`演算子は何をしますか？

### The two uses of the `...` operator: slurping and splatting
### `...`演算子の2つの使い方：スラピングとスプラット

#Many newcomers to Julia find the use of `...` operator confusing. Part of what makes the `...` operator confusing is that it means two different things depending on context.
Juliaへの新入社員の多くは、「...」演算子の使用が混乱していることを発見しました。 `...`演算子を混乱させる原因の1つは、コンテキストに応じて2つの異なることを意味するということです。

### `...` combines many arguments into one argument in function definitions
### `...`は多くの引数を関数定義の一つの引数に結合します

#In the context of function definitions, the `...` operator is used to combine many different arguments into a single argument. 
関数定義の文脈では、 `...`演算子は、多くの異なる引数を単一の引数に結合するために使用されます。
#This use of `...` for combining many different arguments into a single argument is called slurping:
多くの異なる引数を単一の引数に結合するためのこの `... 'の使用は、スラピングと呼ばれます：

```jldoctest
julia> function printargs(args...)
           @printf("%s\n", typeof(args))
           for (i, arg) in enumerate(args)
               @printf("Arg %d = %s\n", i, arg)
           end
       end
printargs (generic function with 1 method)

julia> printargs(1, 2, 3)
Tuple{Int64,Int64,Int64}
Arg 1 = 1
Arg 2 = 2
Arg 3 = 3
```

#If Julia were a language that made more liberal use of ASCII characters, the slurping operator might have been written as `<-...` instead of `...`.
JuliaがASCII文字をより自由に使用する言語であった場合、スラッピング演算子は `...`ではなく `<-... 'として記述されている可能性があります。

### `...` splits one argument into many different arguments in function calls
### `...`は一つの引数を関数呼び出しの多くの異なる引数に分割します

#In contrast to the use of the `...` operator to denote slurping many different arguments into one argument when defining a function, the `...` operator is also used to cause a single function argument to be split apart into many different arguments when used in the context of a function call. 
`... '演算子を使用して関数を定義するときに、多くの異なる引数を1つの引数にスラッピングするのとは対照的に、` ... `演算子を使用して、単一の関数引数を多くの異なる 関数呼び出しのコンテキストで使用された場合の引数
#This use of `...` is called splatting:
この `...`の使用はスプラットと呼ばれます：

```jldoctest
julia> function threeargs(a, b, c)
           @printf("a = %s::%s\n", a, typeof(a))
           @printf("b = %s::%s\n", b, typeof(b))
           @printf("c = %s::%s\n", c, typeof(c))
       end
threeargs (generic function with 1 method)

julia> vec = [1, 2, 3]
3-element Array{Int64,1}:
 1
 2
 3

julia> threeargs(vec...)
a = 1::Int64
b = 2::Int64
c = 3::Int64
```

#If Julia were a language that made more liberal use of ASCII characters, the splatting operator might have been written as `...->` instead of `...`.
JuliaがASCII文字をより自由に使用する言語だった場合、スプラット演算子は `...-`の代わりに `...-` `と書かれているかもしれません。

## Types, type declarations, and constructors
##型、型宣言、およびコンストラクタ

### [What does "type-stable" mean?](@id man-type-stability)
### [「型安定」とは何ですか？]（@ id型の型安定性）

#It means that the type of the output is predictable from the types of the inputs.  
これは、出力のタイプが入力のタイプから予測可能であることを意味します。
#In particular, it means that the type of the output cannot vary depending on the *values* of the inputs. 
特に、入力の* value *に応じて出力のタイプを変えることはできません。
#The following code is *not* type-stable:
次のコードはタイプ安定ではありません：

```jldoctest
julia> function unstable(flag::Bool)
           if flag
               return 1
           else
               return 1.0
           end
       end
unstable (generic function with 1 method)
```

#It returns either an `Int` or a [`Float64`](@ref) depending on the value of its argument. 
引数の値に応じて、 `Int`か` Float64`（@ ref）のいずれかを返します。

#Since Julia can't predict the return type of this function at compile-time, any computation that uses it will have to guard against both types possibly occurring, making generation of fast machine code difficult.
Juliaはコンパイル時にこの関数の戻り値の型を予測することができないため、それを使用するすべての計算では、両方の型が発生するのを防ぐ必要があり、高速のマシンコードの生成が困難になります。

### [Why does Julia give a `DomainError` for certain seemingly-sensible operations?](@id faq-domain-errors)
### [なぜジュリアは特定の一見賢明な操作のために `DomainError`を出すのですか？]（@ id faq-domain-errors）

#Certain operations make mathematical sense but result in errors:
特定の操作は数学的には意味がありますが、エラーが発生します。

```jldoctest
julia> sqrt(-2.0)
ERROR: DomainError:
sqrt will only return a complex result if called with a complex argument. Try sqrt(complex(x)).
Stacktrace:
 [1] sqrt(::Float64) at ./math.jl:438

julia> 2^-5
ERROR: DomainError:
Cannot raise an integer x to a negative power -n.
Make x a float by adding a zero decimal (e.g. 2.0^-n instead of 2^-n), or write 1/x^n, float(x)^-n, or (x//1)^-n.
Stacktrace:
 [1] power_by_squaring(::Int64, ::Int64) at ./intfuncs.jl:173
 [2] literal_pow(::Base.#^, ::Int64, ::Type{Val{-5}}) at ./intfuncs.jl:208
```

#This behavior is an inconvenient consequence of the requirement for type-stability.  
この挙動は、型安定性の要求の不都合な結果である。
#In the case of [`sqrt()`](@ref), most users want `sqrt(2.0)` to give a real number, and would be unhappy if
[`sqrt（）`]（@ ref）の場合、ほとんどのユーザは `sqrt（2.0）`に実数を与えることを望み、
#it produced the complex number `1.4142135623730951 + 0.0im`.  
それは複素数「1.4142135623730951 + 0.0im」を生成した。
#One could write the [`sqrt()`](@ref) function to switch to a complex-valued output only when passed a negative number (which is what [`sqrt()`](@ref) does in some other languages), but then the result would not be [type-stable](@ref man-type-stability) and the [`sqrt()`](@ref) function would have poor performance.
（sqrt（） `）（@ ref）関数は、他の言語では` `sqrt（）`]（@ ref）と同じように、負の数値を渡したときにのみ複素数値の出力に切り替えることができます （@ ref man-type-stability）と[`sqrt（）`]（@ref）関数のパフォーマンスが悪くなります。

#In these and other cases, you can get the result you want by choosing an *input type* that conveys your willingness to accept an *output type* in which the result can be represented:
これらの場合と他の場合では、結果を表すことができる*出力タイプ*を受け入れる意思を伝える*入力タイプ*を選択することで、必要な結果を得ることができます：

```jldoctest
julia> sqrt(-2.0+0im)
0.0 + 1.4142135623730951im

julia> 2.0^-5
0.03125
```

#### Why does Julia use native machine integer arithmetic?
###ジュリアはなぜネイティブマシンの整数計算を使用していますか？

#Julia uses machine arithmetic for integer computations. 
Juliaは整数計算に機械計算を使用します。
#This means that the range of `Int` values is bounded and wraps around at either end so that adding, subtracting and multiplying integers can overflow or underflow, leading to some results that can be unsettling at first:
これは、Intの値の範囲が制限されており、加算、減算、および整数の乗算がオーバーフローまたはアンダーフローするため、最初は不安定になることがあるいくつかの結果につながることを意味します。

```jldoctest
julia> typemax(Int)
9223372036854775807

julia> ans+1
-9223372036854775808

julia> -ans
-9223372036854775808

julia> 2*ans
0
```

#Clearly, this is far from the way mathematical integers behave, and you might think it less than ideal for a high-level programming language to expose this to the user. 
明らかに、これは数学的整数の振る舞いとはほど遠く、高レベルのプログラミング言語ではこれをユーザーに公開するのが理想的ではないと考えるかもしれません。
#For numerical work where efficiency and transparency are at a premium, however, the alternatives are worse.
しかし、効率と透明性が重視される数値作業では、代替案が悪化しています。

#One alternative to consider would be to check each integer operation for overflow and promote results to bigger integer types such as [`Int128`](@ref) or [`BigInt`](@ref) in the case of overflow.
考慮すべき1つの代替案は、オーバーフローの場合の各整数演算をチェックし、オーバーフローの場合に[[Int128`]（@ ref）または[`BigInt`]（@ref）のようなより大きな整数型に結果を昇格させることです。
#Unfortunately, this introduces major overhead on every integer operation (think incrementing a loop counter) – it requires emitting code to perform run-time overflow checks after arithmetic instructions and branches to handle potential overflows. 
残念ながら、これはすべての整数演算に大きなオーバーヘッドをもたらします（ループカウンタをインクリメントすると思います）。算術命令と分岐後に実行時オーバーフローチェックを実行して潜在的なオーバーフローを処理するコードが必要です。
#Worse still, this would cause every computation involving integers to be type-unstable. 
さらに悪いことに、これは整数を含むすべての計算が型不安​​定になる原因になります。
#As we mentioned above, [type-stability is crucial](@ref man-type-stability) for effective generation of efficient code. 
上記のように、効率的なコードを効果的に生成するためには、[型安定性が重要です]（@ ref man-type-stability）。
#If you can't count on the results of integer operations being integers, it's impossible to generate fast, simple code the way C and Fortran compilers do.
整数演算の結果を整数にすることができない場合は、CコンパイラやFortranコンパイラのように高速で簡単なコードを生成することは不可能です。

#A variation on this approach, which avoids the appearance of type instability is to merge the `Int` and [`BigInt`](@ref) types into a single hybrid integer type, that internally changes representation when a result no longer fits into the size of a machine integer. 
型の不安定性の出現を避けるこのアプローチの変形は、 `Int`型と` BigInt`型（@ref）型を1つのハイブリッド型にマージし、結果がもはやマシン整数のサイズ。
#While this superficially avoids type-instability at the level of Julia code, it just sweeps the problem under the rug by foisting all of the same difficulties onto the C code implementing this hybrid integer type. 
これは表面的にはJuliaコードのレベルでタイプの不安定性を避けていますが、この混成整数型を実装するCコードに同じ困難を全面的に押しつけるだけで問題を掃除するだけです。
#This approach *can* be made to work and can even be made quite fast in many cases, but has several drawbacks.
このアプローチは仕事にすることができ、多くの場合かなり速くすることもできますが、いくつかの欠点があります。
#One problem is that the in-memory representation of integers and arrays of integers no longer match the natural representation used by C, Fortran and other languages with native machine integers.
1つの問題は、整数と整数の配列のメモリ内表現が、C、Fortranおよび他の言語で使用されている自然表現ともはやネイティブの機械整数と一致しないことです。
#Thus, to interoperate with those languages, we would ultimately need to introduce native integer types anyway. 
したがって、それらの言語と相互運用するためには、最終的にはネイティブの整数型を導入する必要があります。
#Any unbounded representation of integers cannot have a fixed number of bits, and thus cannot be stored inline in an array with fixed-size slots – large integer values will always require separate heap-allocated storage. 
整数の無制限表現は固定ビット数を持つことができないため、固定サイズのスロットを持つ配列にインラインで格納することはできません。大きな整数値は常に別々のヒープ割り当てストレージを必要とします。
#And of course, no matter how clever a hybrid integer implementation one uses, there are always performance traps – situations where performance degrades unexpectedly. 
もちろん、ハイブリッド整数の実装がどれほど巧妙であっても、パフォーマンスが予期せず低下する状況が常に存在します。
#Complex representation, lack of interoperability with C and Fortran, the inability to represent integer arrays without additional heap storage, and unpredictable performance characteristics make even the cleverest hybrid integer implementations a poor choice for high-performance numerical work.
複雑な表現、CおよびFortranとの相互運用性の欠如、追加のヒープストレージなしで整数配列を表現できないこと、予測できないパフォーマンス特性により、ハイブリッドな数値作業のための最も賢明なハイブリッド整数実装さえも悪い選択肢になります。

#An alternative to using hybrid integers or promoting to BigInts is to use saturating integer arithmetic, where adding to the largest integer value leaves it unchanged and likewise for subtracting from the smallest integer value. 
ハイブリッド整数の使用やBigIntsへの昇格の代わりに、飽和整数演算を使用することができます。ここでは、最大の整数値を加算するとその値は変更されず、同様に最小の整数値から減算されます。
#This is precisely what Matlab™ does:
これはまさにMatlab™の機能です。

```
>> int64(9223372036854775807)

ans =

  9223372036854775807

>> int64(9223372036854775807) + 1

ans =

  9223372036854775807

>> int64(-9223372036854775808)

ans =

 -9223372036854775808

>> int64(-9223372036854775808) - 1

ans =

 -9223372036854775808
```

#At first blush, this seems reasonable enough since 9223372036854775807 is much closer to 9223372036854775808 than -9223372036854775808 is and integers are still represented with a fixed size in a natural way that is compatible with C and Fortran. 
これは、9223372036854775807が-9223372036854775808 isよりもはるかに9223372036854775808に近く、整数はCとFortranと互換性のある自然な方法で固定サイズで表されているので、最初は赤面目になります。
#Saturated integer arithmetic, however, is deeply problematic. 
しかし、飽和整数演算は深刻な問題です。
#The first and most obvious issue is that this is not the way machine integer arithmetic works, so implementing saturated operations requires emitting instructions after each machine integer operation to check for underflow or overflow and replace the result with [`typemin(Int)`](@ref) or [`typemax(Int)`](@ref) as appropriate. 
最初の最も顕著な問題は、これが機械整数演算の仕方ではないことです。飽和演算を実装するには、アンダーフローまたはオーバーフローをチェックして結果を[`typemin（Int）`]で置き換えるために、 @ref）または[`typemax（Int）`]（@ ref）を適切に使用してください。
#This alone expands each integer operation from a single, fast instruction into half a dozen instructions, probably including branches. 
これだけで、各整数演算が単一の高速命令から半ダースの命令（おそらく分岐を含む）に拡張されます。
#Ouch. 
痛い
#But it gets worse – saturating integer arithmetic isn't associative. 
しかし、それは悪化します。飽和している整数演算は連想ではありません。
#Consider this Matlab computation:
このMatlabの計算を考えてみましょう：

```
>> n = int64(2)^62
4611686018427387904

>> n + (n - 1)
9223372036854775807

>> (n + n) - 1
9223372036854775806
```

#This makes it hard to write many basic integer algorithms since a lot of common techniques depend on the fact that machine addition with overflow *is* associative. 
これは、多くの基本的な整数アルゴリズムを書くのが難しくなります。なぜなら、多くの一般的なテクニックは、オーバーフロー*を伴うマシンの追加が*連想であるという事実に依存しているからです。
#Consider finding the midpoint between integer values `lo` and `hi` in Julia using the expression `(lo + hi) >>> 1`:
Juliaで `（lo + hi）>>> 1`という式を使って整数値loとhiの中点を見つけることを考えてみましょう。

```jldoctest
julia> n = 2^62
4611686018427387904

julia> (n + 2n) >>> 1
6917529027641081856
```

#See? No problem. 
見る？ 問題ない。
#That's the correct midpoint between 2^62 and 2^63, despite the fact that `n + 2n` is -4611686018427387904. 
`n + 2n`が-4611686018427387904であるにもかかわらず、それは2 ^ 62と2 ^ 63の間の正しい中間点です。
#Now try it in Matlab:
Matlabで試してみましょう：

```
>> (n + 2*n)/2

ans =

  4611686018427387904
```

#Oops. 
おっとっと。
#Adding a `>>>` operator to Matlab wouldn't help, because saturation that occurs when adding `n` and `2n` has already destroyed the information necessary to compute the correct midpoint.
`n 'と` 2n'を追加するときに発生する彩度が正しい中間点を計算するのに必要な情報を既に破壊しているので、Matlabに `>>>`演算子を追加することは役に立たないでしょう。

#Not only is lack of associativity unfortunate for programmers who cannot rely it for techniques like this, but it also defeats almost anything compilers might want to do to optimize integer arithmetic. 
このようなテクニックに頼ることができないプログラマにとっては、連想の欠如は残念であるばかりでなく、コンパイラが整数算術を最適化するためにやりたいことがほとんどないことにもなります。
#For example, since Julia integers use normal machine integer arithmetic, LLVM is free to aggressively optimize simple little functions like `f(k) = 5k-1`. 
例えば、Julia整数は通常の機械整数演算を使用するので、LLVMは `f（k）= 5k-1`のような簡単な関数を積極的に最適化することができます。
#The machine code for this function is just this:
この関数のマシンコードはこれだけです：

```julia-repl
julia> code_native(f, Tuple{Int})
  .text
Filename: none
  pushq %rbp
  movq  %rsp, %rbp
Source line: 1
  leaq  -1(%rdi,%rdi,4), %rax
  popq  %rbp
  retq
  nopl  (%rax,%rax)
```

#The actual body of the function is a single `leaq` instruction, which computes the integer multiply and add at once. 
関数の実際の本体は、単一の `leaq`命令で、整数の乗算と加算を一度に計算します。
#This is even more beneficial when `f` gets inlined into another function:
これは、 `f`が別の関数にインライン展開されるときにさらに有益です：

```julia-repl
julia> function g(k, n)
           for i = 1:n
               k = f(k)
           end
           return k
       end
g (generic function with 1 methods)

julia> code_native(g, Tuple{Int,Int})
  .text
Filename: none
  pushq %rbp
  movq  %rsp, %rbp
Source line: 2
  testq %rsi, %rsi
  jle L26
  nopl  (%rax)
Source line: 3
L16:
  leaq  -1(%rdi,%rdi,4), %rdi
Source line: 2
  decq  %rsi
  jne L16
Source line: 5
L26:
  movq  %rdi, %rax
  popq  %rbp
  retq
  nop
```

#Since the call to `f` gets inlined, the loop body ends up being just a single `leaq` instruction.
`f`の呼び出しがインラインになるので、ループ本体は単なる` leaq`命令になります。
#Next, consider what happens if we make the number of loop iterations fixed:
次に、ループ反復回数を固定するとどうなるかを考えてみましょう。

```julia-repl
julia> function g(k)
           for i = 1:10
               k = f(k)
           end
           return k
       end
g (generic function with 2 methods)

julia> code_native(g,(Int,))
  .text
Filename: none
  pushq %rbp
  movq  %rsp, %rbp
Source line: 3
  imulq $9765625, %rdi, %rax    # imm = 0x9502F9
  addq  $-2441406, %rax         # imm = 0xFFDABF42
Source line: 5
  popq  %rbp
  retq
  nopw  %cs:(%rax,%rax)
```

#Because the compiler knows that integer addition and multiplication are associative and that multiplication distributes over addition – neither of which is true of saturating arithmetic – it can optimize the entire loop down to just a multiply and an add. 
コンパイラは整数の加算と乗算が連想的であることを知っているため、乗算は加算を超えて分散します。どちらも飽和演算には当てはまりません。ループ全体を乗算と加算に最適化できます。
#Saturated arithmetic completely defeats this kind of optimization since associativity and distributivity can fail at each loop iteration, causing different outcomes depending on which iteration the failure occurs in. 
飽和演算は、連想性と分散性が各ループ反復で失敗する可能性があるため、この種の最適化を完全に打ち消し、どの反復が発生するかによって異なる結果を引き起こします。
#The compiler can unroll the loop, but it cannot algebraically reduce multiple operations into fewer equivalent operations.
コンパイラはループを展開できますが、複数の演算を代数的に同等の演算に減らすことはできません。

#The most reasonable alternative to having integer arithmetic silently overflow is to do checked arithmetic everywhere, raising errors when adds, subtracts, and multiplies overflow, producing values that are not value-correct. 
整数演算を静かにオーバーフローさせる最も妥当な方法は、チェックされた算術をどこでも実行し、オーバーフローを加算、減算、および乗算するときにエラーを発生させ、値が正しくない値を生成することです。
#In this [blog post](http://danluu.com/integer-overflow/), Dan Luu analyzes this and finds that rather than the trivial cost that this approach should in theory have, it ends up having a substantial cost due to compilers (LLVM and GCC) not gracefully optimizing around the added overflow checks. 
このブログ記事（http://danluu.com/integer-overflow/）ではDan Luuがこれを分析して、理論的にはこのアプローチに必要な些細なコストではなく、コンパイラ（LLVMとGCC）は、追加されたオーバーフローチェックをうまく最適化しません。
#If this improves in the future, we could consider defaulting to checked integer arithmetic in Julia, but for now, we have to live with the possibility of overflow.
これが将来的に改善されれば、Juliaでのチェック整数計算へのデフォルト設定を検討することができますが、今はオーバーフローの可能性を生かす必要があります。

### What are the possible causes of an `UndefVarError` during remote execution?
###リモート実行中の `UndefVarError`の原因は何ですか？

#As the error states, an immediate cause of an `UndefVarError` on a remote node is that a binding by that name does not exist. 
エラー状態として、リモートノード上の `UndefVarError`の直接の原因は、その名前によるバインディングが存在しないことです。
#Let us explore some of the possible causes.
考えられる原因のいくつかを探そう。

```julia-repl
julia> module Foo
           foo() = remotecall_fetch(x->x, 2, "Hello")
       end

julia> Foo.foo()
ERROR: On worker 2:
UndefVarError: Foo not defined
[...]
```

#The closure `x->x` carries a reference to `Foo`, and since `Foo` is unavailable on node 2, an `UndefVarError` is thrown.
閉包 `x-> x`は` Foo`への参照を持ち、 `Foo`はノード2で利用できないので` UndefVarError`がスローされます。

#Globals under modules other than `Main` are not serialized by value to the remote node. 
`Main`以外のモジュールのグローバルは、値によってリモートノードにシリアル化されません。
#Only a reference is sent. 
参照のみが送信されます。
#Functions which create global bindings (except under `Main`) may cause an `UndefVarError` to be thrown later.
グローバルバインディング（ `Main`を除く）を作成する関数は、後で` UndefVarError`をスローする可能性があります。

```julia-repl
julia> @everywhere module Foo
           function foo()
               global gvar = "Hello"
               remotecall_fetch(()->gvar, 2)
           end
       end

julia> Foo.foo()
ERROR: On worker 2:
UndefVarError: gvar not defined
[...]
```

#In the above example, `@everywhere module Foo` defined `Foo` on all nodes. 
上記の例では、 `@everywhere module Foo`はすべてのノードで` Foo`を定義していました。
#However the call to `Foo.foo()` created a new global binding `gvar` on the local node, but this was not found on node 2 resulting in an `UndefVarError` error.
しかし、 `Foo.foo（）`を呼び出すと、ローカルノードに新しいグローバルバインディング `gvar`が作成されましたが、これはノード2では見つかりませんでした。その結果、` UndefVarError`エラーが発生しました。

#Note that this does not apply to globals created under module `Main`. 
これはモジュール `Main`の下で作成されたグローバルには当てはまりません。
#Globals under module `Main` are serialized and new bindings created under `Main` on the remote node.
`Main`モジュールのグローバルは直列化され、リモートノードの` Main`の下に新しいバインディングが作成されます。

```julia-repl
julia> gvar_self = "Node1"
"Node1"

julia> remotecall_fetch(()->gvar_self, 2)
"Node1"

julia> remotecall_fetch(whos, 2)
	From worker 2:	                          Base  41762 KB     Module
	From worker 2:	                          Core  27337 KB     Module
	From worker 2:	                           Foo   2477 bytes  Module
	From worker 2:	                          Main  46191 KB     Module
	From worker 2:	                     gvar_self     13 bytes  String
```

#This does not apply to `function` or `type` declarations. 
これは `function`宣言や` type`宣言には当てはまりません。
#However, anonymous functions bound to global variables are serialized as can be seen below.
ただし、以下に示すように、グローバル変数にバインドされた無名関数が直列化されています。

```julia-repl
julia> bar() = 1
bar (generic function with 1 method)

julia> remotecall_fetch(bar, 2)
ERROR: On worker 2:
UndefVarError: #bar not defined
[...]

julia> anon_bar  = ()->1
(::#21) (generic function with 1 method)

julia> remotecall_fetch(anon_bar, 2)
1
```

## Packages and Modules
##パッケージとモジュール


### What is the difference between "using" and "importall"?
### "using"と "importall"の違いは何ですか？

#There is only one difference, and on the surface (syntax-wise) it may seem very minor. 
違いは1つだけであり、表面上（構文上）にはそれは非常に小さいように見えるかもしれません。
#The difference between `using` and `importall` is that with `using` you need to say `function Foo.bar(..` to extend module Foo's function bar with a new method, but with `importall` or `import Foo.bar`,
`using`と` importall`の違いは、 `using`を使うと` function Foo.bar（.. `は新しいメソッドでモジュールFooの関数バーを拡張するが、` importall`や `Foo import 'を使う必要があるということです。バー、
#you only need to say `function bar(...` and it automatically extends module Foo's function bar.
`function bar（...`）と言うだけでモジュールFooの関数バーが自動的に拡張されます。

#If you use `importall`, then `function Foo.bar(...` and `function bar(...` become equivalent.
`importall`を使うと、` function Foo.bar（... `と` function bar（... `は等価になります。
#If you use `using`, then they are different.
`using`を使うと、それらは異なっています。

#The reason this is important enough to have been given separate syntax is that you don't want to accidentally extend a function that you didn't know existed, because that could easily cause a bug. 
これが別個の構文を与えられているのに十分重要である理由は、あなたが誤って存在していなかった関数を誤って拡張したくないということです。
#This is most likely to happen with a method that takes a common type like a string or integer, because both you and the other module could define a method to handle such a common type. 
これは、あなたと他のモジュールの両方がそのような共通型を扱うメソッドを定義できるので、文字列や整数のような共通の型を取るメソッドで起こりそうです。
#If you use `importall`, then you'll replace the other module's implementation of `bar(s::AbstractString)` with your new implementation, which could easily do something completely different (and break all/many future usages of the other functions in module Foo that depend on calling bar).
`importall`を使うと、他のモジュールの` bar（s :: AbstractString） `の実装をあなたの新しい実装に置き換えます。これは簡単に何かを完全に変えることができます（そして、コールバーに依存するモジュールFoo）。

## Nothingness and missing values
##無駄と欠損値

### How does "null" or "nothingness" work in Julia?
### Juliaでは "null"または "nothingness"はどのように機能しますか？

#Unlike many languages (for example, C and Java), Julia does not have a "null" value. 
多くの言語（CやJavaなど）とは異なり、Juliaには「null」という値はありません。
#When a reference (variable, object field, or array element) is uninitialized, accessing it will immediately throw an error. 
参照（変数、オブジェクトフィールド、または配列要素）が初期化されていない場合、それにアクセスするとすぐにエラーがスローされます。
#This situation can be detected using the `isdefined` function.
この状況は `isdefined`関数を使って検出できます。

#Some functions are used only for their side effects, and do not need to return a value. 
一部の関数は副作用のためにのみ使用され、値を返す必要はありません。
#In these cases, the convention is to return the value `nothing`, which is just a singleton object of type `Void`. 
このような場合、慣習では `nothing 'という値を返します。これは` Void`型の単なる単なるオブジェクトです。
#This is an ordinary type with no fields; there is nothing special about it except for this convention, and that the REPL does not print anything for it. 
これはフィールドを持たない普通の型です。この条約を除いて特別なことは何もなく、REPLは何も印刷しません。
#Some language constructs that would not otherwise have a value also yield `nothing`, for example `if false; end`.
そうでなければ値を持たない言語構造体も、 `false 'のように` nothing`を返します。終わり。

#For situations where a value exists only sometimes (for example, missing statistical data), it is best to use the `Nullable{T}` type, which allows specifying the type of a missing value.
時々だけ値が存在する状況（例えば、統計データが欠落している）の場合、 `Nullable {T}`型を使うのが最善です。これは欠損値の型を指定することを可能にします。

#The empty tuple (`()`) is another form of nothingness. 
空のタプル（ `（）`）はもう一つの無形の形式です。
#But, it should not really be thought of as nothing but rather a tuple of zero values.
しかし、それは実際にゼロ値のタプルであると考えられるべきではありません。

#In code written for Julia prior to version 0.4 you may occasionally see `None`, which is quite different. 
バージョン0.4より前のJulia用に書かれたコードでは、 `None`と表示されることがあります。これはまったく異なっています。
#It is the empty (or "bottom") type, a type with no values and no subtypes (except itself).
これは空の（または「ボトム」）型で、値を持たず、サブタイプもありません（それ自体を除く）。
#This is now written as `Union{}` (an empty union type). 
これは `Union {}`（空の共用体型）として書かれています。
#You will generally not need to use this type.
一般に、このタイプは使用する必要はありません。

## Memory
##メモリ

### Why does `x += y` allocate memory when `x` and `y` are arrays?
### `x`と` y`が配列のときに `x + = y`がメモリを割り当てるのはなぜですか？

#In Julia, `x += y` gets replaced during parsing by `x = x + y`. 
Juliaでは、 `x = x + y`によって解析中に` x + = y`が置き換えられます。
#For arrays, this has the consequence that, rather than storing the result in the same location in memory as `x`, it allocates a new array to store the result.
配列の場合、結果は `x`と同じ位置に結果を格納するのではなく、新しい配列を割り当てて結果を格納するという結果になります。

#While this behavior might surprise some, the choice is deliberate. 
この動作は驚くかもしれませんが、その選択は意図的です。
#The main reason is the presence of immutable objects within Julia, which cannot change their value once created.  
主な理由は、ジュリア内に不変のオブジェクトが存在することです。ジュリアは一度作成されると値を変更できません。
#Indeed, a number is an immutable object; the statements `x = 5; x += 1` do not modify the meaning of `5`, they modify the value bound to `x`. 
実際、数値は不変のオブジェクトです。 ステートメント `x = 5; x + = 1`は `5`の意味を変更せず、` x`に束縛された値を変更します。
#For an immutable, the only way to change the value is to reassign it.
不変の場合、値を変更する唯一の方法は、それを再割り当てすることです。

#To amplify a bit further, consider the following function:
さらに少し増幅するには、次の関数を考えてみましょう：

```julia
function power_by_squaring(x, n::Int)
    ispow2(n) || error("This implementation only works for powers of 2")
    while n >= 2
        x *= x
        n >>= 1
    end
    x
end
```

#After a call like `x = 5; y = power_by_squaring(x, 4)`, you would get the expected result: `x == 5 && y == 625`.
`x = 5;のような呼び出しの後。 y = power_by_squaring（x、4） `とすると、期待される結果が得られます：` x == 5 && y == 625`。
 #However, now suppose that `*=`, when used with matrices, instead mutated the left hand side.
 しかし、今では `* =`が行列と一緒に使用されたときに、左辺を変更したと仮定します。
 #There would be two problems:
 2つの問題があります。
  *
   # For general square matrices, `A = A*B` cannot be implemented without temporary storage: `A[1,1]` gets computed and stored on the left hand side before you're done using it on the right hand side.
    一般的な正方行列の場合、 `A = A * B`は一時記憶なしには実装できません：` A [1,1] `が計算され、右側で使用される前に左辺に格納されます。
  *
   # Suppose you were willing to allocate a temporary for the computation (which would eliminate most of the point of making `*=` work in-place); if you took advantage of the mutability of `x`, then this function would behave differently for mutable vs. immutable inputs. 
   計算のために一時的なものを割り当てようとしているとしましょう（これは、 `* =`の作業の大半を取り除くでしょう）。 `x`の変更可能性を利用した場合、この関数は変更可能な入力と変更不能な入力に対して動作が異なります。
   # In particular, for immutable `x`, after the call you'd have (in general) `y != x`, but for mutable `x` you'd have `y == x`.
    特に、不変の `x`の場合は、呼び出しの後に（一般的に）` y！= x`を持ちますが、変更可能な `x`では` y == x`を持ちます。

#Because supporting generic programming is deemed more important than potential performance optimizations that can be achieved by other means (e.g., using explicit loops), operators like `+=` and `*=` work by rebinding new values.
ジェネリックプログラミングをサポートすることは、他の手段（例えば、明示的なループを使用すること）によって達成され得る潜在的なパフォーマンスの最適化よりも重要であると考えられるため、新しい値を再バインドすることによって、 `+ =`や `* =

## Asynchronous IO and concurrent synchronous writes
##非同期IOと並行同期書き込み

### Why do concurrent writes to the same stream result in inter-mixed output?
###なぜ同じストリームへの並行書き込みが相互混合出力になるのですか？

#While the streaming I/O API is synchronous, the underlying implementation is fully asynchronous.
ストリーミングI / O APIは同期型ですが、基本的な実装は完全に非同期です。

#Consider the printed output from the following:
以下の印刷出力を考えてみましょう。

```jldoctest
julia> @sync for i in 1:3
           @async write(STDOUT, string(i), " Foo ", " Bar ")
       end
123 Foo  Foo  Foo  Bar  Bar  Bar
```

#This is happening because, while the `write` call is synchronous, the writing of each argument yields to other tasks while waiting for that part of the I/O to complete.
これは、 `write`呼び出しが同期的である間に、I / Oのその部分が完了するのを待っている間に、各引数の書き込みが他のタスクに生じるためです。

#`print` and `println` "lock" the stream during a call.
`print`と` println`は呼び出し中にストリームを "ロック"します。
#Consequently changing `write` to `println` in the above example results in:
したがって、上の例で `println`に` write`を変更すると次のようになります：

```jldoctest
julia> @sync for i in 1:3
           @async println(STDOUT, string(i), " Foo ", " Bar ")
       end
1 Foo  Bar
2 Foo  Bar
3 Foo  Bar
```

#You can lock your writes with a `ReentrantLock` like this:
あなたは `ReentrantLock`で以下のように書き込みをロックすることができます：

```jldoctest
julia> l = ReentrantLock()
ReentrantLock(Nullable{Task}(), Condition(Any[]), 0)

julia> @sync for i in 1:3
           @async begin
               lock(l)
               try
                   write(STDOUT, string(i), " Foo ", " Bar ")
               finally
                   unlock(l)
               end
           end
       end
1 Foo  Bar 2 Foo  Bar 3 Foo  Bar
```

### Julia Releases

#### Do I want to use a release, beta, or nightly version of Julia?
### Juliaのリリース版、ベータ版、または夜間版を使用したいですか？

#You may prefer the release version of Julia if you are looking for a stable code base. 
安定したコードベースを探している場合は、Juliaのリリース版が好きかもしれません。
#Releases generally occur every 6 months, giving you a stable platform for writing code.
リリースは一般的に6ヵ月ごとに行われ、コードを書くための安定したプラットフォームを提供します。

#You may prefer the beta version of Julia if you don't mind being slightly behind the latest bugfixes and changes, but find the slightly faster rate of changes more appealing. 
あなたが最新のバグ修正や変更の後ろに少しでも気にしないならば、Juliaのベータ版を好むかもしれませんが、少し速い変化の割合がより魅力的であることがわかります。
#Additionally, these binaries are tested before they are published to ensure they are fully functional.
さらに、これらのバイナリは、それらが完全に機能していることを確認するために公開前にテストされます。

#You may prefer the nightly version of Julia if you want to take advantage of the latest updates to the language, and don't mind if the version available today occasionally doesn't actually work.
最新のアップデートを利用したい場合は、夜間バージョンのJuliaを使用することをお勧めします。また、今日入手可能なバージョンが実際に動作しない場合は気にしないでください。

#Finally, you may also consider building Julia from source for yourself. 
最後に、ジュリアを自分でソースから構築することも考えられます。
#This option is mainly for those individuals who are comfortable at the command line, or interested in learning. 
このオプションは、主にコマンドラインに慣れていたり、学習に興味を持っている人のためのものです。
#If this describes you, you may also be interested in reading our [guidelines for contributing](https://github.com/JuliaLang/julia/blob/master/CONTRIBUTING.md).
これがあなたに当てはまる場合は、[寄稿のためのガイドライン]（https://github.com/JuliaLang/julia/blob/master/CONTRIBUTING.md）をご覧ください。

#Links to each of these download types can be found on the download page at [https://julialang.org/downloads/](https://julialang.org/downloads/). 
これらのダウンロードタイプへのリンクは、[https://julialang.org/downloads/](https://julialang.org/downloads/）]のダウンロードページにあります。
#Note that not all versions of Julia are available for all platforms.
Juliaのすべてのバージョンがすべてのプラットフォームで使用できるわけではないことに注意してください。

<!-- ## When are deprecated functions removed?-->
###廃止された関数はいつ削除されますか？

<!-- Deprecated functions are removed after the subsequent release. -->
廃止された関数は、次のリリース後に削除されます。
<!-- For example, functions marked as deprecated in the 0.1 release will not be available starting with the 0.2 release.-->
たとえば、0.1リリースで廃止予定とマークされた機能は、0.2リリースからは使用できません。