<!-- Start-->

## [Functions](@id man-functions)

> ## [関数](@ id man-functions)

<!-- End -->
<!-- Start-->
In Julia, a function is an object that maps a tuple of argument values to a return value.
> Juliaでは、関数は、引数値のタプルを戻り値にマップするオブジェクトです。
<!-- End -->
<!-- Start-->
Julia functions are not pure mathematical functions, in the sense that functions can alter and be affected by the global state of the program.
> ジュリア関数は純粋な数学関数ではなく、関数がプログラムの全体的な状態によって変更され、影響を受けることがあります。
<!-- End -->
<!-- Start-->
The basic syntax for defining functions in Julia is:
> Juliaで関数を定義するための基本的な構文は次のとおりです。
<!-- End -->

```jldoctest
julia> function f(x,y)
           x + y
       end
f (generic function with 1 method)
```

<!-- Start-->
There is a second, more terse syntax for defining a function in Julia.
> Juliaで関数を定義するための第2のより簡潔な構文があります。
<!-- End -->
<!-- Start-->
The traditional function declaration syntax demonstrated above is equivalent to the following compact "assignment form":
> 上に示した伝統的な関数宣言構文は、次のコンパクトな「代入形式」と同等です。
<!-- End -->

```jldoctest fofxy
julia> f(x,y) = x + y
f (generic function with 1 method)
```

<!-- Start -->
In the assignment form, the body of the function must be a single expression, although it can be a compound expression (see [Compound Expressions](@ref man-compound-expressions)). 
> 代入式では、関数の本体は単一の式でなければなりませんが、複合式( [Compound Expressions](@ref man-compound-expressions)を参照してください)。
<!-- End -->
<!-- Start-->
Short, simple function definitions are common in Julia.
> Juliaでは、簡潔で単純な関数定義が一般的です。
<!-- End -->
<!-- Start-->
The short function syntax is accordingly quite idiomatic, considerably reducing both typing and visual noise.
> したがって、短い関数構文はかなり慣用的であり、タイピングと視覚的ノイズの両方をかなり低減します。
<!-- End -->

<!-- Start-->
A function is called using the traditional parenthesis syntax:
> 関数は、従来のかっこの構文を使用して呼び出されます。
<!-- End -->

```jldoctest fofxy
julia> f(2,3)
5
```

<!-- Start-->
Without parentheses, the expression `f` refers to the function object, and can be passed around like any value:
> 括弧なしでは、式 `f`は関数オブジェクトを参照し、任意の値のように渡すことができます：
<!-- End -->

```jldoctest fofxy
julia> g = f;

julia> g(2,3)
5
```

<!-- Start-->
As with variables, Unicode can also be used for function names:
> 変数と同様に、関数名にUnicodeを使用することもできます。
<!-- End -->

```jldoctest
julia> ∑(x,y) = x + y
∑ (generic function with 1 method)

julia> ∑(2, 3)
5
```

<!-- Start-->

## Argument Passing Behavior

> ## 引数渡し動作

<!-- End -->
<!-- Start-->
Julia function arguments follow a convention sometimes called "pass-by-sharing", which means that values are not copied when they are passed to functions.
> Julia関数の引数は、 "pass-by-sharing"と呼ばれることもあります。これは、関数に渡されたときに値がコピーされないことを意味します。
<!-- End -->
<!-- Start -->
Function arguments themselves act as new variable *bindings* (new locations that can refer to values), but the values they refer to are identical to the passed values. 
> 関数の引数自体は、新しい変数*の束縛*(値を参照できる新しい場所)として機能しますが、それらが参照する値は渡された値と同じです。
<!-- End -->
<!-- Start -->
Modifications to mutable values (such as `Array`s) made within a function will be visible to the caller. 
> 関数内で行われた変更可能な値( `Array`sなど)に対する変更は、呼び出し側に見えるようになります。
<!-- End -->
<!-- Start -->
This is the same behavior found in Scheme, most Lisps, Python, Ruby and Perl, among other dynamic languages.
> これは、他の動的言語の中でScheme、ほとんどのLisps、Python、Ruby、Perlで見られるのと同じ動作です。
<!-- End -->
<!-- Start -->

## The `return` Keyword

> ## `return`キーワード

<!-- End -->
<!-- Start -->
The value returned by a function is the value of the last expression evaluated, which, by default, is the last expression in the body of the function definition. 
> 関数によって返される値は、評価された最後の式の値です。デフォルトでは、関数定義の本体の最後の式です。
<!-- End -->
<!-- Start -->
In the example function, `f`, from the previous section this is the value of the expression `x + y`. 
> 例の関数 `f`では、前のセクションから、これは式` x + y`の値です。
<!-- End -->
<!-- Start -->
As in C and most other imperative or functional languages, the `return` keyword causes a function to return immediately, providing an expression whose value is returned:
> Cや他の命令型言語や関数型言語のように、 `return`キーワードは、関数が直ちに戻り、値が返される式を提供します：
<!-- End -->

```julia
function g(x,y)
    return x * y
    x + y
end
```

<!-- Start -->
Since function definitions can be entered into interactive sessions, it is easy to compare these definitions:
> 関数定義はインタラクティブなセッションに入力できるので、これらの定義を比較するのは簡単です：
<!-- End -->

```jldoctest
julia> f(x,y) = x + y
f (generic function with 1 method)

julia> function g(x,y)
           return x * y
           x + y
       end
g (generic function with 1 method)

julia> f(2,3)
5

julia> g(2,3)
6
```

<!-- Start -->
Of course, in a purely linear function body like `g`, the usage of `return` is pointless since the expression `x + y` is never evaluated and we could simply make `x * y` the last expression in the function and omit the `return`. 
> もちろん、 `g`のような純粋に線形の関数体では、` x + y`という式は決して評価されないので、 `return`の使い方は無意味です。単に` x * y`を関数の最後の式にすることができます。 `return`を省略します。
<!-- End -->
<!-- Start -->
In conjunction with other control flow, however, `return` is of real use. 
> しかし、他の制御フローと併せて、「リターン」は実際に使用されています。
<!-- End -->
<!-- Start -->
Here, for example, is a function that computes the hypotenuse length of a right triangle with sides of length `x` and `y`, avoiding overflow:
> ここでは、例えば、長さがxとyの辺を持つ直角三角形の斜辺の長さを計算し、オーバーフローを回避する関数を次に示します。
<!-- End -->

```jldoctest
julia> function hypot(x,y)
           x = abs(x)
           y = abs(y)
           if x > y
               r = y/x
               return x*sqrt(1+r*r)
           end
           if y == 0
               return zero(x)
           end
           r = x/y
           return y*sqrt(1+r*r)
       end
hypot (generic function with 1 method)

julia> hypot(3, 4)
5.0
```

<!-- Start -->
There are three possible points of return from this function, returning the values of three different expressions, depending on the values of `x` and `y`. 
> この関数から3つの可能な戻り点があり、 `x`と` y`の値に応じて3つの異なる式の値を返します。
<!-- End -->
<!-- Start -->
The `return` on the last line could be omitted since it is the last expression.
> 最後の行の `return`は最後の式なので省略することができます。
<!-- End -->
<!-- Start -->

## Operators Are Functions

> ## 演算子は関数です

<!-- End -->
<!-- Start -->
In Julia, most operators are just functions with support for special syntax. 
> Juliaでは、ほとんどの演算子は特別な構文をサポートする関数です。
<!-- End -->
<!-- Start -->
(The exceptions are operators with special evaluation semantics like `&&` and `||`. 
> (例外は、 `&&`や `||`のような特別な評価のセマンティクスを持つ演算子です。
<!-- End -->
<!-- Start -->
These operators cannot be functions since [Short-Circuit Evaluation](@ref) requires that their operands are not evaluated before evaluation of the operator.) 
> [Short-Circuit Evaluation](@ref)ではオペレータの評価前にオペランドが評価されないことが必要なため、これらの演算子は関数ではありません。
<!-- End -->
<!-- Start -->
Accordingly, you can also apply them using parenthesized argument lists, just as you would any other function:
> したがって、他の関数と同様に、カッコで囲まれた引数リストを使用してそれらを適用することもできます。
<!-- End -->

```jldoctest
julia> 1 + 2 + 3
6

julia> +(1,2,3)
6
```

<!-- Start-->
The infix form is exactly equivalent to the function application form -- in fact the former is parsed to produce the function call internally.
> 挿入形式は、関数のアプリケーション形式とまったく同じです。実際は、前者は内部的に関数呼び出しを生成するために解析されます。
<!-- End -->
<!-- Start-->
This also means that you can assign and pass around operators such as [`+()`](@ref) and [`*()`](@ref) just like you would with other function values:
> これは、他の関数値と同じように、[`+()`](@ref)や[`*()`](@ref)などの演算子を代入して渡すこともできることを意味します：
<!-- End -->

```jldoctest
julia> f = +;

julia> f(1,2,3)
6
```

<!-- Start-->
Under the name `f`, the function does not support infix notation, however.
> `f`という名前では、関数は中置記法をサポートしていません。
<!-- End -->
<!-- Start-->

## Operators With Special Names

> ## 特殊名を持つ演算子

<!-- End -->
<!-- Start-->
A few special expressions correspond to calls to functions with non-obvious names.
> いくつかの特別な式は、明白でない名前の関数への呼び出しに対応しています。
<!-- End -->
<!-- Start-->
These are:
> これらは：
<!-- End -->

| Expression        | Calls                  |
|:----------------- |:---------------------- |
| `[A B C ...]`     | [`hcat()`](@ref)       |
| `[A; B; C; ...]`  | [`vcat()`](@ref)       |
| `[A B; C D; ...]` | [`hvcat()`](@ref)      |
| `A'`              | [`ctranspose()`](@ref) |
| `A.'`             | [`transpose()`](@ref)  |
| `1:n`             | [`colon()`](@ref)      |
| `A[i]`            | [`getindex()`](@ref)   |
| `A[i]=x`          | [`setindex!()`](@ref)  |

<!-- Start-->

## [Anonymous Functions](@id man-anonymous-functions)

> ## [匿名関数](@id の匿名関数)

<!-- End -->
<!-- Start-->
Functions in Julia are [first-class objects](https://en.wikipedia.org/wiki/First-class_citizen):
> Juliaの関数は[first-class object](https://en.wikipedia.org/wiki/First-class_citizen)です：
<!-- End -->
<!-- Start-->
they can be assigned to variables, and called using the standard function call syntax from the variable they have been assigned to.
> 変数に代入することができ、代入された変数から標準の関数呼び出し構文を使用して呼び出すことができます。
<!-- End -->
<!-- Start-->
They can be used as arguments, and they can be returned as values.
> 引数として使用でき、値として返すことができます。
<!-- End -->
<!-- Start-->
They can also be created anonymously, without being given a name, using either of these syntaxes:
> これらの構文のいずれかを使用して、名前を付けずに匿名で作成することもできます。
<!-- End -->

```jldoctest
julia> x -> x^2 + 2x - 1
(::#1) (generic function with 1 method)

julia> function (x)
           x^2 + 2x - 1
       end
(::#3) (generic function with 1 method)
```

<!-- Start-->
This creates a function taking one argument `x` and returning the value of the polynomial `x^2 + 2x - 1` at that value.
> これは、1つの引数 `x`をとり、その値で多項式` x ^ 2 + 2x - 1`の値を返す関数を作成します。
<!-- End -->
<!-- Start-->
Notice that the result is a generic function, but with a compiler-generated name based on consecutive numbering.
> 結果は汎用関数ですが、連続した番号付けに基づいたコンパイラ生成の名前が付いています。
<!-- End -->

<!-- Start-->
The primary use for anonymous functions is passing them to functions which take other functions as arguments.
> 匿名関数の主な用途は、他の関数を引数とする関数に渡すことです。
<!-- End -->
<!-- Start-->
A classic example is [`map()`](@ref), which applies a function to each value of an array and returns a new array containing the resulting values:
> 古典的な例は、配列の各値に関数を適用し、結果の値を含む新しい配列を返す[`map()`](@ref)です：
<!-- End -->

```jldoctest
julia> map(round, [1.2,3.5,1.7])
3-element Array{Float64,1}:
 1.0
 4.0
 2.0
```

<!-- Start-->
This is fine if a named function effecting the transform already exists to pass as the first argument to [`map()`](@ref).
> 変換を行う名前付き関数がすでに存在していて、最初の引数として[`map()`](@ref)に渡しても問題ありません。
<!-- End -->
<!-- Start-->
Often, however, a ready-to-use, named function does not exist.
> ただし、すぐに使用できる名前付き関数は存在しないことがよくあります。
<!-- End -->
<!-- Start-->
In these situations, the anonymous function construct allows easy creation of a single-use function object without needing a name:
> このような状況では、無名関数構造を使用すると、名前を必要とせずに使いやすい関数オブジェクトを簡単に作成できます。
<!-- End -->

```jldoctest
julia> map(x -> x^2 + 2x - 1, [1,3,-1])
3-element Array{Int64,1}:
  2
 14
 -2
```

<!-- Start-->
An anonymous function accepting multiple arguments can be written using the syntax `(x,y,z)->2x+y-z`.
> `(x,y,z)->2x+y-z`の構文を使って、複数の引数を受け入れる無名関数を書くことができます。
<!-- End -->
<!-- Start-->
A zero-argument anonymous function is written as `()->3`.
> 引数のない無名関数は `()->3`と書かれています。
<!-- End -->
<!-- Start-->
The idea of a function with no arguments may seem strange, but is useful for "delaying" a computation.
> 引数のない関数の考え方は奇妙に見えるかもしれませんが、計算を「遅らせる」ためには便利です。
<!-- End -->
<!-- Start-->
In this usage, a block of code is wrapped in a zero-argument function, which is later invoked by calling it as `f()`.
> この使用法では、コードのブロックはゼロ引数の関数にラップされ、後で `f()`として呼び出されます。
<!-- End -->
<!-- Start-->

## Multiple Return Values

> ## 複数の戻り値

<!-- End -->
<!-- Start-->
In Julia, one returns a tuple of values to simulate returning multiple values.
> Juliaでは、複数の値を返すようにシミュレーションするために値のタプルを返します。
<!-- End -->
<!-- Start-->
However, tuples can be created and destructured without needing parentheses, thereby providing an illusion that multiple values are being returned, rather than a single tuple value.
> しかし、タプルは、括弧を必要とせずに作成および破棄することができ、それによって単一のタプル値ではなく、複数の値が返されるという錯覚を提供します。
<!-- End -->
<!-- Start-->
For example, the following function returns a pair of values:
> たとえば、次の関数は値のペアを返します。
<!-- End -->

```jldoctest foofunc
julia> function foo(a,b)
           a+b, a*b
       end
foo (generic function with 1 method)
```

<!-- Start-->
If you call it in an interactive session without assigning the return value anywhere, you will see the tuple returned:
> 戻り値をどこにも割り当てないでインタラクティブなセッションで呼び出すと、返されたタプルが表示されます：
<!-- End -->

```jldoctest foofunc
julia> foo(2,3)
(5, 6)
```

<!-- Start-->
A typical usage of such a pair of return values, however, extracts each value into a variable.
> しかし、このような1組の戻り値の典型的な使用法は、各値を変数に抽出します。
<!-- End -->
<!-- Start-->
Julia supports simple tuple "destructuring" that facilitates this:
> Juliaは、これを容易にする単純なタプル「構造解除」をサポートしています。
<!-- End -->

```jldoctest foofunc
julia> x, y = foo(2,3)
(5, 6)

julia> x
5

julia> y
6
```

<!-- Start-->
You can also return multiple values via an explicit usage of the `return` keyword:
> また、`return`キーワードを明示的に使用して複数の値を返すこともできます：
<!-- End -->

```julia
function foo(a,b)
    return a+b, a*b
end
```

<!-- Start-->
This has the exact same effect as the previous definition of `foo`.
> これは 事前に定義した`foo`と全く同じ効果を持ちます。
<!-- End -->
<!-- Start-->

## Varargs Functions

> ## 可変数関数

<!-- End -->
<!-- Start-->
It is often convenient to be able to write functions taking an arbitrary number of arguments.
> 任意の数の引数を取って関数を書くことができると便利なことがよくあります。
<!-- End -->
<!-- Start-->
Such functions are traditionally known as "varargs" functions, which is short for "variable number of arguments".
> そのような関数は伝統的に "varargs"関数と呼ばれ、 "可変数の引数"の略です。
<!-- End -->
<!-- Start-->
You can define a varargs function by following the last argument with an ellipsis:
> 最後の引数の後に省略記号をつけて、varargs関数を定義することができます：
<!-- End -->

```jldoctest barfunc
julia> bar(a,b,x...) = (a,b,x)
bar (generic function with 1 method)
```

<!-- Start-->
The variables `a` and `b` are bound to the first two argument values as usual, and the variable `x` is bound to an iterable collection of the zero or more values passed to `bar` after its first two arguments:
> 変数 `a`と` b`は通常のように最初の2つの引数値に束縛され、変数 `x`は最初の2つの引数の後で` bar`に渡される0以上の値の繰り返し可能な集合に束縛されます：
<!-- End -->

```jldoctest barfunc
julia> bar(1,2)
(1, 2, ())

julia> bar(1,2,3)
(1, 2, (3,))

julia> bar(1, 2, 3, 4)
(1, 2, (3, 4))

julia> bar(1,2,3,4,5,6)
(1, 2, (3, 4, 5, 6))
```

<!-- Start-->
In all these cases, `x` is bound to a tuple of the trailing values passed to `bar`.
> これらのすべての場合、 `x`は` bar`に渡された末尾の値のタプルに束縛されます。
<!-- End -->

<!-- Start-->
It is possible to constrain the number of values passed as a variable argument; this will be discussed later in [Parametrically-constrained Varargs methods](@ref).
> 渡される値の数を可変引数として制限することは可能です。 これについては、後で[パラメトリックに制約されたVarargsメソッド](@ref)で議論します。
<!-- End -->

<!-- Start-->
On the flip side, it is often handy to "splice" the values contained in an iterable collection into a function call as individual arguments.
> 反面、iterableコレクションに含まれる値を個々の引数として関数呼び出しに「スプライス」することは、しばしば便利です。
<!-- End -->
<!-- Start-->
To do this, one also uses `...` but in the function call instead:
> これを行うために、関数呼び出しの代わりに `...`も使います：
<!-- End -->

```jldoctest barfunc
julia> x = (3, 4)
(3, 4)

julia> bar(1,2,x...)
(1, 2, (3, 4))
```

<!-- Start-->
In this case a tuple of values is spliced into a varargs call precisely where the variable number of arguments go.
> この場合、値のタプルは、引数の可変数がどこに行くか正確にvarargsコールに継承されます。
<!-- End -->
<!-- Start-->
This need not be the case, however:
> しかし、そうである必要はありません：
<!-- End -->

```jldoctest barfunc
julia> x = (2, 3, 4)
(2, 3, 4)

julia> bar(1,x...)
(1, 2, (3, 4))

julia> x = (1, 2, 3, 4)
(1, 2, 3, 4)

julia> bar(x...)
(1, 2, (3, 4))
```

<!-- Start-->
Furthermore, the iterable object spliced into a function call need not be a tuple:
> さらに、関数呼び出しに継承された反復可能オブジェクトはタプルである必要はありません。
<!-- End -->

```jldoctest barfunc
julia> x = [3,4]
2-element Array{Int64,1}:
 3
 4

julia> bar(1,2,x...)
(1, 2, (3, 4))

julia> x = [1,2,3,4]
4-element Array{Int64,1}:
 1
 2
 3
 4

julia> bar(x...)
(1, 2, (3, 4))
```

<!-- Start-->
Also, the function that arguments are spliced into need not be a varargs function (although it often is):
> また、引数がスプライスされる関数は、varargs関数である必要はありません(しばしばですが)。
<!-- End -->

```jldoctest
julia> baz(a,b) = a + b;

julia> args = [1,2]
2-element Array{Int64,1}:
 1
 2

julia> baz(args...)
3

julia> args = [1,2,3]
3-element Array{Int64,1}:
 1
 2
 3

julia> baz(args...)
ERROR: MethodError: no method matching baz(::Int64, ::Int64, ::Int64)
Closest candidates are:
  baz(::Any, ::Any) at none:1
```

<!-- Start-->
As you can see, if the wrong number of elements are in the spliced container, then the function call will fail, just as it would if too many arguments were given explicitly.
> ご覧のように、スプライスされたコンテナに間違った数の要素があると、明示的に引数が多すぎる場合と同じように、関数呼び出しは失敗します。
<!-- End -->
<!-- Start-->

## Optional Arguments

> ## オプションの引数

<!-- End -->
<!-- Start -->
In many cases, function arguments have sensible default values and therefore might not need to be passed explicitly in every call. 
> 多くの場合、関数の引数には分かりやすいデフォルト値があるため、すべての呼び出しで明示的に渡す必要はありません。
<!-- End -->
<!-- Start -->
For example, the library function [`parse(T, num, base)`](@ref) interprets a string as a number in some base. 
> 例えば、ライブラリ関数[`parse(T、num、base)`](@ref)は文字列をある基底の数字として解釈します。
<!-- End -->
<!-- Start -->
The `base` argument defaults to `10`. 
> `base`引数のデフォルトは` 10`です。
<!-- End -->
<!-- Start -->
This behavior can be expressed concisely as:
> この振る舞いは、以下のように簡潔に表現できます。
<!-- End -->

```julia
function parse(T, num, base=10)
    ###
end
```

<!-- Start -->
With this definition, the function can be called with either two or three arguments, and `10` is automatically passed when a third argument is not specified:
> この定義では、関数は2つまたは3つの引数で呼び出すことができ、第3の引数が指定されていないときには自動的に `10 'が渡されます。
<!-- End -->

```jldoctest
julia> parse(Int,"12",10)
12

julia> parse(Int,"12",3)
5

julia> parse(Int,"12")
12
```

<!-- Start -->
Optional arguments are actually just a convenient syntax for writing multiple method definitions with different numbers of arguments (see [Note on Optional and keyword Arguments](@ref)).
> オプションの引数は実際には、引数の数が異なる複数のメソッド定義を記述するための便利な構文です([オプションとキーワード引数についての注意](@ref)を参照してください)。
<!-- End -->
<!-- Start -->

### Keyword Arguments

> ### キーワードの引数

<!-- End -->
<!-- Start -->
Some functions need a large number of arguments, or have a large number of behaviors. 
> 関数によっては、多数の引数が必要なものや、多数の振る舞いを持つものがあります。
<!-- End -->
<!-- Start -->
Remembering how to call such functions can be difficult. 
> そのような関数を呼び出す方法を覚えておくのは難しいことです。
<!-- End -->
<!-- Start -->
Keyword arguments can make these complex interfaces easier to use and extend by allowing arguments to be identified by name instead of only by position.
> キーワード引数を使用すると、引数だけを位置ではなく名前で識別できるため、これらの複雑なインタフェースを使いやすく拡張することができます。
<!-- End -->

<!-- Start-->
For example, consider a function `plot` that plots a line.
> 例えば、線をプロットする関数 `plot`を考えてみましょう。
<!-- End -->
<!-- Start-->
This function might have many options, for controlling line style, width, color, and so on.
> この関数には、線のスタイル、幅、色などを制御するための多くのオプションがあります。
<!-- End -->
<!-- Start-->
If it accepts keyword arguments, a possible call might look like `plot(x, y, width=2)`, where we have chosen to specify only line width.
> キーワード引数を受け入れるならば、 `plot(x、y、width = 2)`のように見えるかもしれません。
<!-- End -->
<!-- Start-->
Notice that this serves two purposes.
> これは2つの目的に役立つことに注意してください。
<!-- End -->
<!-- Start-->
The call is easier to read, since we can label an argument with its meaning.
> 引数の意味にそのラベルを付けることができるので、呼び出しは読みやすくなります。
<!-- End -->
<!-- Start-->
It also becomes possible to pass any subset of a large number of arguments, in any order.
> また、多数の引数の任意の部分集合を任意の順序で渡すことも可能になります。
<!-- End -->

<!-- Start-->
Functions with keyword arguments are defined using a semicolon in the signature:
> キーワード引数を持つ関数は、シグネチャのセミコロンを使用して定義されます。
<!-- End -->

```julia
function plot(x, y; style="solid", width=1, color="black")
    ###
end
```

<!-- Start-->
When the function is called, the semicolon is optional: one can either call `plot(x, y, width=2)` or `plot(x, y; width=2)`, but the former style is more common.
> 関数が呼び出されると、 `plot(x、y、width = 2)`または `plot(x、y; width = 2)`を呼び出すことができますが、前者のスタイルがより一般的です。
<!-- End -->
<!-- Start-->
An explicit semicolon is required only for passing varargs or computed keywords as described below.
> 明示的なセミコロンは、以下で説明するように、varargsまたは計算キーワードを渡すためにのみ必要です。
<!-- End -->
<!-- Start-->
Keyword argument default values are evaluated only when necessary (when a corresponding keyword argument is not passed), and in left-to-right order.
> キーワード引数のデフォルト値は、必要な場合(対応するキーワード引数が渡されない場合)にのみ、左から右の順に評価されます。
<!-- End -->
<!-- Start-->
Therefore default expressions may refer to prior keyword arguments.
> したがって、デフォルト式は、キーワードの前の引数を参照することがあります。
<!-- End -->

<!-- Start-->
The types of keyword arguments can be made explicit as follows:
> キーワード引数のタイプは、次のように明示的に指定できます。
<!-- End -->

```julia
function f(;x::Int=1)
    ###
end
```

<!-- Start-->
Extra keyword arguments can be collected using `...`, as in varargs functions:
> 余分なキーワード引数は、varargs関数のように、 `...`を使って収集することができます：
<!-- End -->

```julia
function f(x; y=0, kwargs...)
    ###
end
```

<!-- Start-->
Inside `f`, `kwargs` will be a collection of `(key,value)` tuples, where each `key` is a symbol.
> `f`の中で、` kwargs`は `(キー、値)`タプルの集合で、各キーはシンボルです。
<!-- End -->

<!-- Start-->
Such collections can be passed as keyword arguments using a semicolon in a call, e.g. `f(x, z=1; kwargs...)`.
> そのようなコレクションは、コールのセミコロンを使用してキーワード引数として渡すことができます。 `f(x、z = 1; kwargs ...)`である。
<!-- End -->
<!-- Start-->
Dictionaries can also be used for this purpose.
> 辞書もこの目的のために使用することができます。
<!-- End -->

<!-- Start -->
One can also pass `(key,value)` tuples, or any iterable expression (such as a `=>` pair) that can be assigned to such a tuple, explicitly after a semicolon. 
> このようなタプルにセミコロンの後に明示的に割り当てることができる `(key、value)`タプル、または反復可能な式( `=>`ペアなど)を渡すこともできます。
<!-- End -->
<!-- Start -->
For example, `plot(x, y; (:width,2))` and `plot(x, y; :width => 2)` are equivalent to `plot(x, y, width=2)`.
> 例えば `plot(x、y;(：width、2))`と `plot(x、y;：width => 2)`は `plot(x、y、width = 2)`に相当します。
<!-- End -->
<!-- Start -->
This is useful in situations where the keyword name is computed at runtime.
> これは、実行時にキーワード名が計算される場合に便利です。
<!-- End -->

<!-- Start -->
The nature of keyword arguments makes it possible to specify the same argument more than once.
> キーワード引数の性質により、同じ引数を複数回指定することができます。
<!-- End -->
<!-- Start -->
For example, in the call `plot(x, y; options..., width=2)` it is possible that the `options` structure also contains a value for `width`. 
> たとえば、 `plot(x、y; options ...、width = 2)という呼び出しで` options`構造体に `width`の値も含まれている可能性があります。
<!-- End -->
<!-- Start -->
In such a case the rightmost occurrence takes precedence; in this example, `width` is certain to have the value `2`.
> そのような場合、右端の出現が優先されます。 この例では、 `width`は値` 2`を持っています。
<!-- End -->
<!-- Start -->

## Evaluation Scope of Default Values

> ## デフォルト値の評価範囲

<!-- End -->
<!-- Start -->
Optional and keyword arguments differ slightly in how their default values are evaluated. 
> オプションとキーワードの引数は、デフォルト値の評価方法が若干異なります。
<!-- End -->
<!-- Start -->
When optional argument default expressions are evaluated, only *previous* arguments are in scope. 
> オプションの引数デフォルト式が評価されるときは、* previous *引数だけがスコープ内にあります。
<!-- End -->
<!-- Start -->
In contrast, *all* the arguments are in scope when keyword arguments default expressions are evaluated.
> 対照的に、キーワード引数のデフォルト式が評価されるときは、* all *引数が有効範囲内にあります。
<!-- End -->
<!-- Start -->
For example, given this definition:
> たとえば、次の定義があるとします。
<!-- End -->

```julia
function f(x, a=b, b=1)
    ###
end
```

<!-- Start-->
the `b` in `a=b` refers to a `b` in an outer scope, not the subsequent argument `b`.
> `a = b`の`b`は、外側のスコープ内の`b`を指し、後続の引数`b`を指しません。
<!-- End -->
<!-- Start -->
However, if `a` and `b` were keyword arguments instead, then both would be created in the same scope and the `b` in `a=b` would refer to the subsequent argument `b` (shadowing any `b` in an outer scope), which would result in an undefined variable error (since the default expressions are evaluated left-to-right, and `b` has not been assigned yet).
> しかし、 `a`と` b`がキーワード引数であれば、両方とも同じスコープで作成され、 `a = b`の` b`は後続の引数 `b`を参照します (デフォルトの式は左から右に評価され、 `b`はまだ割り当てられていないので)未定義の変数エラーが発生します。
<!-- End -->
<!-- Start-->

## Do-Block Syntax for Function Arguments

> ## 関数引数のためのDo-Block構文

<!-- End -->
<!-- Start -->
Passing functions as arguments to other functions is a powerful technique, but the syntax for it is not always convenient. 
> 他の関数への引数として関数を渡すことは強力な手法ですが、その構文は必ずしも便利ではありません。
<!-- End -->
<!-- Start -->
Such calls are especially awkward to write when the function argument requires multiple lines. 
> このような呼び出しは、関数の引数に複数の行が必要なときに書くのが特に厄介です。
<!-- End -->
<!-- Start -->
As an example, consider calling [`map()`](@ref) on a function with several cases:
> 例として、いくつかのケースを持つ関数に対して[`map()`](@ref)を呼び出すことを考えてみましょう：
<!-- End -->

```julia
map(x->begin
           if x < 0 && iseven(x)
               return 0
           elseif x == 0
               return 1
           else
               return x
           end
       end,
    [A, B, C])
```

<!-- Start-->
Julia provides a reserved word `do` for rewriting this code more clearly:
> Juliaは、このコードをより明確に書き換えるための予約語 `do`を提供しています。
<!-- End -->

```julia
map([A, B, C]) do x
    if x < 0 && iseven(x)
        return 0
    elseif x == 0
        return 1
    else
        return x
    end
end
```

<!-- Start-->
The `do x` syntax creates an anonymous function with argument `x` and passes it as the first argument to [`map()`](@ref).
> `do x`構文は、引数`x`を持つ無名関数を作成し、最初の引数として[`map()`](@ref)へ渡します。
<!-- End -->
<!-- Start -->
Similarly, `do a,b` would create a two-argument anonymous function, and a plain `do` would declare that what follows is an anonymous function of the form `() -> ...`.
> 同様に、 `do a、b`は2つの引数無名関数を作成し、` do`は `` - > ... ``の形式の無名関数であると宣言します。
<!-- End -->

<!-- Start -->
How these arguments are initialized depends on the "outer" function; here, [`map()`](@ref) will sequentially set `x` to `A`, `B`, `C`, calling the anonymous function on each, just as would happen in the syntax `map(func, [A, B, C])`.
> これらの引数がどのように初期化されるかは、「外側」関数に依存します。 ここで `[map()`](@ref)は `map`(func)のように順次` x`を `A`、` B`、 `C`に設定し、 、[A、B、C]) `とする。
<!-- End -->

<!-- Start -->
This syntax makes it easier to use functions to effectively extend the language, since calls look like normal code blocks.
> この構文は、呼び出しが通常のコードブロックのように見えるため、関数を使用して効果的に言語を拡張することを容易にします。
<!-- End -->
<!-- Start -->
There are many possible uses quite different from [`map()`](@ref), such as managing system state.
> [`map()`](@ref)とはかなり異なる可能性があります。たとえば、システムの状態を管理するなどです。
<!-- End -->
<!-- Start -->
For example, there is a version of [`open()`](@ref) that runs code ensuring that the opened file is eventually closed:
> たとえば、オープンされたファイルが最終的に閉じられることを保証するコードを実行する[`open()`](@ref)のバージョンがあります：
<!-- End -->

```julia
open("outfile", "w") do io
    write(io, data)
end
```

<!-- Start -->
This is accomplished by the following definition:
> これは、次の定義によって実現されます。
<!-- End -->

```julia
function open(f::Function, args...)
    io = open(args...)
    try
        f(io)
    finally
        close(io)
    end
end
```

<!-- Start -->
Here, [`open()`](@ref) first opens the file for writing and then passes the resulting output stream to the anonymous function you defined in the `do ... end` block.
> ここで、[`open()`]](@ref)は、まずファイルを書き込み用に開き、結果の出力ストリームを `do ... end`ブロックで定義した無名関数に渡します。
<!-- End -->
<!-- Start -->
After your function exits, [`open()`](@ref) will make sure that the stream is properly closed, regardless of whether your function exited normally or threw an exception.
> 関数が終了した後、関数が正常に終了したか例外をスローしたかにかかわらず、[`open()`](@ref)はストリームが適切に閉じられたことを確認します。
<!-- End -->
<!-- Start -->
(The `try/finally` construct will be described in [Control Flow](@ref).)
> (`try/finally`構造は[Control Flow](@ref) で説明します)。
<!-- End -->

<!-- Start -->
With the `do` block syntax, it helps to check the documentation or implementation to know how the arguments of the user function are initialized.
> `do`ブロック構文では、ドキュメンテーションや実装をチェックして、ユーザー関数の引数がどのように初期化されるかを知ることができます。
<!-- End -->

<!-- Start -->

## [Dot Syntax for Vectorizing Functions](@id man-vectorized)

> ## [関数をベクトル化するためのドット構文](@ id man-vectorized)

<!-- End -->
<!-- Start -->
In technical-computing languages, it is common to have "vectorized" versions of functions, which simply apply a given function `f(x)` to each element of an array `A` to yield a new array via `f(A)`. 
> テクニカルコンピューティング言語では、関数f(A)を介して新しい配列を生成するために、配列 `A`の各要素に与えられた関数` f(x) `を単に適用する"ベクトル化 " `。
<!-- End -->
<!-- Start -->
This kind of syntax is convenient for data processing, but in other languages vectorization is also often required for performance: if loops are slow, the "vectorized" version of a function can call fast library code written in a low-level language. 
> この種の構文はデータ処理には便利ですが、他の言語では、パフォーマンスにはベクトル化がしばしば必要です。ループが遅い場合、関数の「ベクトル化」バージョンは低レベル言語で書かれた高速ライブラリコードを呼び出すことができます。
<!-- End -->
<!-- Start -->
In Julia, vectorized functions are *not* required for performance, and indeed it is often beneficial to write your own loops (see [Performance Tips](@ref man-performance-tips)), but they can still be convenient. 
> Juliaでは、ベクトル化された関数はパフォーマンスに必要ではなく、実際には独自のループを書くことが有益です(@ref man-performance-tipsを参照)。
<!-- End -->
<!-- Start -->
Therefore, *any* Julia function `f` can be applied elementwise to any array (or other collection) with the syntax `f.(A)`.
> したがって、*任意の* Julia関数 `f`は、構文`f(A)`で任意の配列(または他のコレクション)に要素ごとに適用できます。
<!-- End -->
<!-- Start -->
For example `sin` can be applied to all elements in the vector `A`, like so:
> 例えば、`sin`はベクトル` A`の全ての要素に適用できます：
<!-- End -->

```jldoctest
julia> A = [1.0, 2.0, 3.0]
3-element Array{Float64,1}:
 1.0
 2.0
 3.0

julia> sin.(A)
3-element Array{Float64,1}:
 0.841471
 0.909297
 0.14112
```

<!-- Start -->
Of course, you can omit the dot if you write a specialized "vector" method of `f`, e.g. via `f(A::AbstractArray) = map(f, A)`, and this is just as efficient as `f.(A)`.
> もちろん、 `f 'の特殊な"ベクトル "メソッドを書くと、ドットを省略することができます。 `f(A :: AbstractArray)= map(f、A)`を介して `f(A)`と同様に効率的です。
<!-- End -->
<!-- Start -->
But that approach requires you to decide in advance which functions you want to vectorize.
> しかし、このアプローチでは、どの関数をベクトル化するかを事前に決定する必要があります。
<!-- End -->

<!-- Start -->
More generally, `f.(args...)` is actually equivalent to `broadcast(f, args...)`, which allows you to operate on multiple arrays (even of different shapes), or a mix of arrays and scalars (see [Broadcasting](@ref)). 
> より一般的には、 `f(args ...)`は実際に `broadcast(f、args ...)`と同じです。これは複数の配列(異なる形のものでも)や配列と スカラー([Broadcasting](@ref)を参照してください)。
<!-- End -->
<!-- Start -->
For example, if you have `f(x,y) = 3x + 4y`, then `f.(pi,A)` will return a new array consisting of `f(pi,a)` for each `a` in `A`, and `f.(vector1,vector2)` will return a new vector consisting of `f(vector1[i],vector2[i])` for each index `i` (throwing an exception if the vectors have different length).
> 例えば `f(pi、A)`は `f(x、y)= 3x + 4y`ならば、` a 'の `f(pi、a)`からなる新しい配列を返します。 `A`と` f(vector1、vector2) `は各インデックス` i`に対して `f(vector1 [i]、vector2 [i])`で構成される新しいベクトルを返します(ベクトルが異なる場合 長さ)。
<!-- End -->

```jldoctest
julia> f(x,y) = 3x + 4y;

julia> A = [1.0, 2.0, 3.0];

julia> B = [4.0, 5.0, 6.0];

julia> f.(pi, A)
3-element Array{Float64,1}:
 13.4248
 17.4248
 21.4248

julia> f.(A, B)
3-element Array{Float64,1}:
 19.0
 26.0
 33.0
```

<!-- Start -->
Moreover, *nested* `f.(args...)` calls are *fused* into a single `broadcast` loop.
> さらに、* nested * `f。(args ...)`呼び出しは* fused *で単一の `broadcast`ループになります。
<!-- End -->
<!-- Start -->
For example, `sin.(cos.(X))` is equivalent to `broadcast(x -> sin(cos(x)), X)`, similar to `[sin(cos(x)) for x in X]`: there is only a single loop over `X`, and a single array is allocated for the result. 
> 例えば、 `sin(cos(X))`は `Xのxに対してsin [cos(x)]と同様の` broadcast(x - > sin(cos(x))、X) ] `：` X 'の上にループが1つだけあり、結果のために1つの配列が割り当てられます。
<!-- End -->
<!-- Start -->
[In contrast, `sin(cos(X))` in a typical "vectorized" language would first allocate one temporary array for `tmp=cos(X)`, and then compute `sin(tmp)` in a separate loop, allocating a second array.] 
> [典型的な "ベクトル化された"言語の `sin(cos(X))`は、 `tmp = cos(X)`に対して一時的な配列を1つ割り当ててから別のループで `sin(tmp)`を計算し、 2番目の配列を割り当てます。]
<!-- End -->
<!-- Start -->
This loop fusion is not a compiler optimization that may or may not occur, it is a *syntactic guarantee* whenever nested `f.(args...)` calls are encountered. 
> このループフュージョンはコンパイラの最適化ではなく、ネストされた `f(args ...)`コールが出現するたびに構文上の保証*となります。
<!-- End -->
<!-- Start -->
Technically, the fusion stops as soon as a "non-dot" function call is encountered; for example, in `sin.(sort(cos.(X)))` the `sin` and `cos` loops cannot be merged because of the intervening `sort` function.
> 技術的には、「ドットではない」関数呼び出しに遭遇すると直ちに融合が停止します。例えば、 `sin(sort(cos。(X)))`で `sin 'と` cos`ループは、介在する `sort`関数のためにマージできません。
<!-- End -->

<!-- Start -->
Finally, the maximum efficiency is typically achieved when the output array of a vectorized operation is *pre-allocated*, so that repeated calls do not allocate new arrays over and over again for the results (see [Pre-allocating outputs](@ref)). 
> 最後に、最大効率は、ベクトル化された演算の出力配列が*事前割り付け*されているときに一般的に達成されるので、繰り返し呼び出しが結果に対して何度も新しい配列を割り当てることはありません([Pre-allocating outputs] ))。
<!-- End -->
<!-- Start -->
A convenient syntax for this is `X .= ...`, which is equivalent to `broadcast!(identity, X, ...)` except that, as above, the `broadcast!` loop is fused with any nested "dot" calls. 
> このための便利な構文は `broadcast！(identity、X、...)`に相当する `X。= ...`であり、上記のように `broadcast！`ループはネストされた "ドット "を呼び出す。
<!-- End -->
<!-- Start -->
For example, `X .= sin.(Y)` is equivalent to `broadcast!(sin, X, Y)`, overwriting `X` with `sin.(Y)` in-place. 
> たとえば `X。= sin。(Y)`は `broadcast！(sin、X、Y)`に相当し、 `sin`(Y)`で `X`を上書きします。
<!-- End -->
<!-- Start -->
If the left-hand side is an array-indexing expression, e.g. `X[2:end] .= sin.(Y)`, then it translates to `broadcast!` on a `view`, e.g. `broadcast!(sin, view(X, 2:endof(X)), Y)`, so that the left-hand side is updated in-place.
> 左辺が配列インデックス式である場合、例えば、次のようになります。 `X [2：end]。= sin。(Y)`ならば、 `view`の` broadcast！ 'に変換されます。 (X、2：endof(X))、Y) `となるので、左側がインプレースで更新されます。
<!-- End -->

<!-- Start -->
Since adding dots to many operations and function calls in an expression can be tedious and lead to code that is difficult to read, the macro [`@.`](@ref @__dot__) is provided to convert *every* function call, operation, and assignment in an expression into the "dotted" version.
> 式中の多くの演算や関数呼び出しにドットを追加するのは面倒なので、読みにくいコードになることがあるので、マクロの `` @ .`(@ref @ __dot__)はすべての関数呼び出し、演算、「点在」バージョンへの式の代入が含まれます。
<!-- End -->

```jldoctest
julia> Y = [1.0, 2.0, 3.0, 4.0];

julia> X = similar(Y); # pre-allocate output array

julia> @. X = sin(cos(Y)) # equivalent to X .= sin.(cos.(Y))
4-element Array{Float64,1}:
  0.514395
 -0.404239
 -0.836022
 -0.608083
```

<!-- Start -->
Binary (or unary) operators like `.+` are handled with the same mechanism:
> `。+`のようなバイナリ(または単項)演算子は同じメカニズムで処理されます：
<!-- End -->
<!-- Start -->
they are equivalent to `broadcast` calls and are fused with other nested "dot" calls.
> それらは `broadcast`呼び出しに相当し、他のネストされた" dot "呼び出しと融合します。
<!-- End -->
<!-- Start -->
`X .+= Y` etcetera is equivalent to `X .= X .+ Y` and results in a fused in-place assignment;
> `X。+ = Y`等は` X。= X。+ Y`と等価であり、融合されたインプレースアサインとなります。
<!-- End -->
<!-- Start -->
see also [dot operators](@ref man-dot-operators).
> [ドット演算子](@ref man-dot-operators)も参照してください。
<!-- End -->
<!-- Start -->

### Further Reading

> ### 参考文献

<!-- End -->
<!-- Start -->
We should mention here that this is far from a complete picture of defining functions.
> ここでは、これは定義関数の完全な描写からは程遠いものです。
<!-- End -->
<!-- Start -->
Julia has a sophisticated type system and allows multiple dispatch on argument types.
> Juliaは洗練された型システムを持ち、引数型の多重ディスパッチが可能です。
<!-- End -->
<!-- Start -->
None of the examples given here provide any type annotations on their arguments, meaning that they are applicable to all types of arguments.
> ここで与えられた例のどれも、それらの引数にすべての型の引数に適用可能であることを意味する引数の型注釈を提供しません。
<!-- End -->
<!-- Start -->
The type system is described in [Types](@ref man-types) and defining a function in terms of methods chosen by multiple dispatch on run-time argument types is described in [Methods](@ref).
> 型システムは[Types](@ref man-types)に記述されており、実行時引数型の複数のディスパッチによって選択されたメソッドの関数を[Method](@ref)で定義しています。
<!-- End -->