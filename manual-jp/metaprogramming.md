<!--- > Metaprogramming-->
# メタプログラミング

<!--- The strongest legacy of Lisp in the Julia language is its metaprogramming support. -->
Julia言語におけるLispの最も強力な遺産は、メタプログラミングのサポートです。
<!--- Like Lisp, Julia represents its own code as a data structure of the language itself. -->
Lispと同様に、Juliaは言語自体のデータ構造として独自のコードを表します。
<!--- Since code is represented by objects that can be created and manipulated from within the language, it is possible for a program to transform and generate its own code. -->
コードは、言語内で作成および操作できるオブジェクトによって表されるため、プログラムが独自のコードを変換して生成することは可能です。
#This allows sophisticated code generation without extra build steps, and also allows true Lisp-style macros operating at the level of [abstract syntax trees](https://en.wikipedia.org/wiki/Abstract_syntax_tree).
これにより、追加のビルドステップを必要とせずに洗練されたコード生成が可能になります。また、[抽象構文木](https://en.wikipedia.org/wiki/Abstract_syntax_tree)のレベルで動作する真のLispスタイルマクロも使用できます。
#In contrast, preprocessor "macro" systems, like that of C and C++, perform textual manipulation and substitution before any actual parsing or interpretation occurs. 
対照的に、CやC ++のプリプロセッサ「マクロ」システムは、実際の解析や解釈が行われる前にテキストの操作と置換を行います。
#Because all data types and code in Julia are represented by Julia data structures, powerful [reflection](https://en.wikipedia.org/wiki/Reflection_%28computer_programming%29) capabilities are available to explore the internals of a program and its types just like any other data.
Juliaのすべてのデータ型とコードはJuliaのデータ構造で表現されているので、強力な[reflection](https://en.wikipedia.org/wiki/Reflection_%28computer_programming%29)の機能を使用して、プログラムの内部構造を調べることができます。他のデータと同様にタイプします。

## Program representation
##プログラム表現

Every Julia program starts life as a string:
<!--- すべてのJuliaプログラムは、文字列としてその生涯を始めます：-->

```jldoctest prog
julia> prog = "1 + 1"
"1 + 1"
```

<!--- **What happens next?**-->
**次は何が起こる？**

<!--- The next step is to [parse](https://en.wikipedia.org/wiki/Parsing#Computer_languages) each string into an object called an expression, represented by the Julia type `Expr`:-->
次のステップは、各文字列をJulia型 `Expr`で表される式と呼ばれるオブジェクトに[parse](https://en.wikipedia.org/wiki/Parsing#Computer_languages)することです。

```jldoctest prog
julia> ex1 = parse(prog)
:(1 + 1)

julia> typeof(ex1)
Expr
```

<!--- `Expr` objects contain three parts:-->
`Expr`オブジェクトには3つの部分があります：

   <!--- > a `Symbol` identifying the kind of expression.-->
   表現の種類を識別する `Symbol`。
   <!--- > A symbol is an [interned string](https://en.wikipedia.org/wiki/String_interning) identifier (more discussion below).-->
    記号は[interned string](https://en.wikipedia.org/wiki/String_interning)の識別子です(以下の説明を参照)。

```jldoctest prog
julia> ex1.head
:call
```

   * the expression arguments, which may be symbols, other expressions, or literal values:
   expression引数(シンボル、その他の式、またはリテラル値)：

```jldoctest prog
julia> ex1.args
3-element Array{Any,1}:
  :+
 1
 1
```

   <!--- * finally, the expression result type, which may be annotated by the user or inferred by the compiler (and may be ignored completely for the purposes of this chapter):-->
   * 最後に、ユーザーが注釈を付けるか、コンパイラによって推論される式の結果の型(この章では完全に無視されるかもしれません)：

```jldoctest prog
julia> ex1.typ
Any
```

#Expressions may also be constructed directly in [prefix notation](https://en.wikipedia.org/wiki/Polish_notation):
式は[接頭辞記法](https://en.wikipedia.org/wiki/Polish_notation)に直接組み立てることもできます：

```jldoctest prog
julia> ex2 = Expr(:call, :+, 1, 1)
:(1 + 1)
```

#The two expressions constructed above – by parsing and by direct construction – are equivalent:
上記で構築された2つの式は、構文解析と直接構築によって同等です。

```jldoctest prog
julia> ex1 == ex2
true
```

**The key point here is that Julia code is internally represented as a data structure that is accessible from the language itself.**
**ここで重要な点は、ジュリアコードが言語自体からアクセス可能なデータ構造として内部的に表現されていることです。**

#The [`dump()`](@ref) function provides indented and annotated display of `Expr` objects:
[`dump()`](@ ref)関数は、 `Expr`オブジェクトのインデント付きおよび注釈付き表示を提供します：

```jldoctest prog
julia> dump(ex2)
Expr
  head: Symbol call
  args: Array{Any}((3,))
    1: Symbol +
    2: Int64 1
    3: Int64 1
  typ: Any
```

#`Expr` objects may also be nested:
`Expr`オブジェクトはネストすることもできます：

```jldoctest ex3
julia> ex3 = parse("(4 + 4) / 2")
:((4 + 4) / 2)
```

#Another way to view expressions is with Meta.show_sexpr, which displays the [S-expression](https://en.wikipedia.org/wiki/S-expression) form of a given `Expr`, which may look very familiar to users of Lisp. 
式を見るもう一つの方法はMeta.show_sexprであり、これは与えられた `Expr`の[S-expression](https://en.wikipedia.org/wiki/S-expression)形式を表示します。 Lispのユーザー
#Here's an example illustrating the display on a nested `Expr`:
ネストした `Expr`の表示例を以下に示します：

```jldoctest ex3
julia> Meta.show_sexpr(ex3)
(:call, :/, (:call, :+, 4, 4), 2)
```

### Symbols

#The `:` character has two syntactic purposes in Julia. 
`：`文字はJuliaに2つの構文上の目的を持っています。
#The first form creates a [`Symbol`](@ref), an [interned string](https://en.wikipedia.org/wiki/String_interning) used as one building-block of expressions:
最初のフォームは、式の1つのビルディングブロックとして使用される[`Symbol`](@ ref)、[interned string](https://en.wikipedia.org/wiki/String_interning)を作成します。

```jldoctest
julia> :foo
:foo

julia> typeof(ans)
Symbol
```

#The [`Symbol`](@ref) constructor takes any number of arguments and creates a new symbol by concatenating their string representations together:
[`Symbol`](@ ref)コンストラクタは任意の数の引数をとり、それらの文字列表現を連結して新しいシンボルを作成します：

```jldoctest
julia> :foo == Symbol("foo")
true

julia> Symbol("func",10)
:func10

julia> Symbol(:var,'_',"sym")
:var_sym
```

#In the context of an expression, symbols are used to indicate access to variables; when an expression is evaluated, a symbol is replaced with the value bound to that symbol in the appropriate [scope](@ref scope-of-variables).
式のコンテキストでは、シンボルは変数へのアクセスを示すために使用されます。 式が評価されるとき、シンボルは適切な[スコープ](@ refスコープオブ変数)でそのシンボルにバインドされた値に置き換えられます。

#Sometimes extra parentheses around the argument to `:` are needed to avoid ambiguity in parsing.:
解析時にあいまいさを避けるために、 `：`への引数の周りに余分なカッコが必要になることがあります。

```jldoctest
julia> :(:)
:(:)

julia> :(::)
:(::)
```

<!--- # Expressions and evaluation-->
##表現と評価

<!--- ## Quoting-->
###引用する

<!--- The second syntactic purpose of the `:` character is to create expression objects without using the explicit `Expr` constructor. -->
`：`文字の2番目の構文的な目的は、明示的な `` Expr``コンストラクタを使わずに式オブジェクトを作成することです。
<!--- This is referred to as *quoting*. -->
これは* quoting *と呼ばれます。
<!--- The `:` character, followed by paired parentheses around a single statement of Julia code, produces an `Expr` object based on the enclosed code. -->
Juliaコードの1つの文のまわりの `：`文字の後にペアのかっこが続き、囲まれたコードに基づいて `Expr`オブジェクトが生成されます。
<!--- Here is example of the short form used to quote an arithmetic expression:-->
次に、算術式を引用するために使用される短い形式の例を示します。

```jldoctest
julia> ex = :(a+b*c+1)
:(a + b * c + 1)

julia> typeof(ex)
Expr
```

<!--- (to view the structure of this expression, try `ex.head` and `ex.args`, or use [`dump()`](@ref) as above)-->
(この式の構造を見るには、 `ex.head`と` ex.args`を試してみるか、上のように[`dump()`](@ref)を使ってください)

<!--- Note that equivalent expressions may be constructed using [`parse()`](@ref) or the direct `Expr` form:-->
同等の式は、[`parse()`](@ ref)や直接の `` Expr``形式を使って構築することができます：

```jldoctest
julia>      :(a + b*c + 1)  ==
       parse("a + b*c + 1") ==
       Expr(:call, :+, :a, Expr(:call, :*, :b, :c), 1)
true
```

<!--- Expressions provided by the parser generally only have symbols, other expressions, and literal values as their args, whereas expressions constructed by Julia code can have arbitrary run-time values without literal forms as args. -->
パーサーが提供する式は、通常、シンボル、その他の式、およびリテラル値をargとして持つだけですが、Juliaコードで構築された式は、argsとしてリテラル形式のない任意の実行時値を持つことができます。
<!--- In this specific example, `+` and `a` are symbols, `*(b,c)` is a subexpression, and `1` is a literal 64-bit signed integer.-->
この特定の例では、 `+`と `a`はシンボルであり、` *(b、c)`は部分式であり、` 1`は64ビットの符号付き整数です。

#There is a second syntactic form of quoting for multiple expressions: blocks of code enclosed in `quote ... end`. 
複数の式の引用の第2の構文形式があります： `quote ... end`で囲まれたコードのブロック。
#Note that this form introduces `QuoteNode` elements to the expression tree, which must be considered when directly manipulating an expression tree generated from `quote` blocks. 
このフォームは `QuoteNode`要素を式ツリーに導入します。これは` quote`ブロックから生成された式ツリーを直接操作するときに考慮する必要があります。
#For other purposes, `:( ... )` and `quote .. end` blocks are treated identically.
他の目的のために、 `:( ...)`と `quote .. end`ブロックは同じように扱われます。

```jldoctest
julia> ex = quote
           x = 1
           y = 2
           x + y
       end
quote
    #= none:2 =#
    x = 1
    #= none:3 =#
    y = 2
    #= none:4 =#
    x + y
end

julia> typeof(ex)
Expr
```

### Interpolation
###補間

<!--- Direct construction of `Expr` objects with value arguments is powerful, but `Expr` constructors can be tedious compared to "normal" Julia syntax. -->
値引数を持つ `Expr`オブジェクトの直接構築は強力ですが、`Expr`コンストラクタは "通常の" Julia構文に比べて面倒です。
<!--- As an alternative, Julia allows "splicing" or interpolation of literals or expressions into quoted expressions. -->
代わりに、Juliaはリテラルまたは式の "スプライシング"または補間を引用式に入れることができます。
<!--- Interpolation is indicated by the `$` prefix.-->
補間は `$`プレフィックスで示されます。

<!--- In this example, the literal value of `a` is interpolated:-->
この例では、`a`のリテラル値が補間されます：

```jldoctest interp1
julia> a = 1;

julia> ex = :($a + b)
:(1 + b)
```

<!--- Interpolating into an unquoted expression is not supported and will cause a compile-time error:-->
引用符で囲まれていない式に補間することはサポートされていないため、コンパイル時にエラーが発生します。

```jlcodtest interp1
julia> $a + b
ERROR: unsupported or misplaced expression $
 ...
```

<!--- In this example, the tuple `(1,2,3)` is interpolated as an expression into a conditional test:-->
この例では、タプル `(1,2,3)`が条件式テストの式として補間されます。

```jldoctest interp1
julia> ex = :(a in $:((1,2,3)) )
:(a in (1, 2, 3))
```

<!--- Interpolating symbols into a nested expression requires enclosing each symbol in an enclosing quote block:-->
シンボルをネストされた式に補間するには、各シンボルを囲むクォートブロックに囲む必要があります。

```julia-repl
julia> :( :a in $( :(:a + :b) ) )
                   ^^^^^^^^^^
                   quoted inner expression
```

#The use of `$` for expression interpolation is intentionally reminiscent of [string interpolation](@ref string-interpolation) and [command interpolation](@ref command-interpolation). 
式の補間に `$ 'を使用すると、意図的に[文字列補間](@ ref文字列補間)と[コマンド補間](@ refコマンド補間)が思い出されます。
#Expression interpolation allows convenient, readable programmatic construction of complex Julia expressions.
式の補間により、複雑なジュリア式の便利で読み易いプログラムによる構築が可能になります。

<!--- ## [`eval()`](@ref) and effects-->
### [`eval()`](@ref)とエフェクト

<!--- Given an expression object, one can cause Julia to evaluate (execute) it at global scope using [`eval()`](@ref):-->
式オブジェクトがあれば、Juliaは[`eval()`](@ref)を使ってグローバルスコープで評価することができます：

```jldoctest interp1
julia> :(1 + 2)
:(1 + 2)

julia> eval(ans)
3

julia> ex = :(a + b)
:(a + b)

julia> eval(ex)
ERROR: UndefVarError: b not defined
[...]

julia> a = 1; b = 2;

julia> eval(ex)
3
```

<!--- Every [module](@ref modules) has its own [`eval()`](@ref) function that evaluates expressions in its global scope. -->
すべての[module](@ref modules)には、グローバルスコープで式を評価する独自の[`eval()`](@ref)関数があります。
<!--- Expressions passed to [`eval()`](@ref) are not limited to returning values -- they can also have side-effects that alter the state of the enclosing module's environment:-->
[`eval()`](@ ref)に渡される式は、値を返すことに限定されず、内部モジュールの環境の状態を変更する副作用を持つこともできます：

```jldoctest
julia> ex = :(x = 1)
:(x = 1)

julia> x
ERROR: UndefVarError: x not defined

julia> eval(ex)
1

julia> x
1
```

<!--- Here, the evaluation of an expression object causes a value to be assigned to the global variable `x`.-->
ここで、表現オブジェクトの評価により、グローバル変数「x」に値が割り当てられる。

<!--- Since expressions are just `Expr` objects which can be constructed programmatically and then evaluated, it is possible to dynamically generate arbitrary code which can then be run using [`eval()`](@ref).-->
式はプログラムで構築して評価できる `Expr`オブジェクトなので、[` eval() `](@ ref)を使って実行できる任意のコードを動的に生成することができます。
<!--- Here is a simple example:-->
簡単な例を次に示します。

```julia-repl
julia> a = 1;

julia> ex = Expr(:call, :+, a, :b)
:(1 + b)

julia> a = 0; b = 2;

julia> eval(ex)
3
```

#The value of `a` is used to construct the expression `ex` which applies the `+` function to the value 1 and the variable `b`. 
`a`の値は` + `関数を値1と変数` b`に適用する式exを作成するために使われます。
#Note the important distinction between the way `a` and `b` are used:
`a`と` b`の使い方の重要な違いに注意してください：

   * 
   # The value of the *variable*`a` at expression construction time is used as an immediate value in the expression. 
   式作成時の*変数* aの値は式の即値として使用されます。
   # Thus, the value of `a` when the expression is evaluated no longer matters: the value in the expression is already `1`, independent of whatever the value of `a` might be.
    したがって、式が評価されるときの `a`の値は重要でなくなります。式の値は、` a`の値とは無関係に、すでに `1`です。
   * 
   # On the other hand, the *symbol*`:b` is used in the expression construction, so the value of the variable `b` at that time is irrelevant -- `:b` is just a symbol and the variable `b` need not even be defined. 
    一方、*記号* `：b`は式の作成に使われるので、その時の変数` b`の値は無関係です - `：b`は単なる記号であり、変数` b` 定義する必要はありません。
   # At expression evaluation time, however, the value of the symbol `:b` is resolved by looking up the value of the variable `b`.
    しかし、式の評価時には、記号「：b」の値は変数「b」の値を参照することによって解決される。

### Functions on `Expr`essions
### `Expr`セッションの関数


#As hinted above, one extremely useful feature of Julia is the capability to generate and manipulate Julia code within Julia itself. 
上記のように、Juliaの非常に有用な機能の1つは、Julia自体の中でJuliaコードを生成し操作する機能です。
#We have already seen one example of a function returning `Expr` objects: the [`parse()`](@ref) function, which takes a string of Julia code and returns the corresponding `Expr`. 
私たちは既に `Expr`オブジェクトを返す関数の一例を見てきました：[` parse() `](@ ref)関数は、Juliaコードの文字列を受け取り、対応する` Expr`を返します。
#A function can also take one or more `Expr` objects as arguments, and return another `Expr`. 
関数は、1つ以上の `Expr`オブジェクトを引数としてとり、別の` Expr`を返すこともできます。
#Here is a simple, motivating example:
ここに、簡単な動機付けの例があります：

```jldoctest
julia> function math_expr(op, op1, op2)
           expr = Expr(:call, op, op1, op2)
           return expr
       end
math_expr (generic function with 1 method)

julia>  ex = math_expr(:+, 1, Expr(:call, :*, 4, 5))
:(1 + 4 * 5)

julia> eval(ex)
21
```

#As another example, here is a function that doubles any numeric argument, but leaves expressions alone:
別の例として、ここでは任意の数値引数を2倍するが、式だけを残す関数がある：

```jldoctest
julia> function make_expr2(op, opr1, opr2)
           opr1f, opr2f = map(x -> isa(x, Number) ? 2*x : x, (opr1, opr2))
           retexpr = Expr(:call, op, opr1f, opr2f)
           return retexpr
       end
make_expr2 (generic function with 1 method)

julia> make_expr2(:+, 1, 2)
:(2 + 4)

julia> ex = make_expr2(:+, 1, Expr(:call, :*, 5, 8))
:(2 + 5 * 8)

julia> eval(ex)
42
```

## [Macros](@id man-macros)
## [Macros](@ id man-macros)

#Macros provide a method to include generated code in the final body of a program. 
マクロは、生成されたコードをプログラムの最後の本体に含める方法を提供します。
#A macro maps a tuple of arguments to a returned *expression*, and the resulting expression is compiled directly rather than requiring a runtime [`eval()`](@ref) call. 
マクロは1組の引数を返された*式*にマップし、結果の式は実行時の[`eval()`](@ ref)呼び出しを必要とせず直接コンパイルされます。
#Macro arguments may include expressions, literal values, and symbols.
マクロ引数には、式、リテラル値、およびシンボルを含めることができます。

### Basics
###基本

#Here is an extraordinarily simple macro:
これは非常に簡単なマクロです：

```jldoctest sayhello
julia> macro sayhello()
           return :( println("Hello, world!") )
       end
@sayhello (macro with 1 method)
```

#Macros have a dedicated character in Julia's syntax: the `@` (at-sign), followed by the unique name declared in a `macro NAME ... end` block. 
マクロは、Juliaの構文では `@ '(at-sign)の後に` macro NAME ... end`ブロックで宣言された一意の名前が付いた専用文字を持っています。
#In this example, the compiler will replace all instances of `@sayhello` with:
この例では、コンパイラは `@ sayhello`のすべてのインスタンスを次のように置き換えます：

```julia
:( println("Hello, world!") )
```

#When `@sayhello` is entered in the REPL, the expression executes immediately, thus we only see the evaluation result:
`@ sayhello`がREPLに入力されると、式は直ちに実行されるので、評価結果のみが表示されます。

```jldoctest sayhello
julia> @sayhello()
Hello, world!
```

#Now, consider a slightly more complex macro:
さて、もう少し複雑なマクロを考えてみましょう。

```jldoctest sayhello2
julia> macro sayhello(name)
           return :( println("Hello, ", $name) )
       end
@sayhello (macro with 1 method)
```

#This macro takes one argument: `name`. When `@sayhello` is encountered, the quoted expression is *expanded* to interpolate the value of the argument into the final expression:
このマクロは `name`という引数をとります。 `@ sayhello`に遭遇すると、引数の値を最終式に補間するために引用された式が* expanded *されます：

```jldoctest sayhello2
julia> @sayhello("human")
Hello, human
```

#We can view the quoted return expression using the function [`macroexpand()`](@ref) (**important note:** this is an extremely useful tool for debugging macros):
引用符で囲まれたリターン式は、関数 `[macroexpand()`](@ ref)を使って見ることができます(**重要な注記：**これはマクロをデバッグするのに非常に便利なツールです)：

```jldoctest sayhello2
julia> ex = macroexpand( :(@sayhello("human")) )
:((println)("Hello, ", "human"))

julia> typeof(ex)
Expr
```

#We can see that the `"human"` literal has been interpolated into the expression.
`"human"`リテラルが式に補間されていることがわかります。

#There also exists a macro [`@macroexpand`](@ref) that is perhaps a bit more convenient than the `macroexpand` function:
おそらく `macroexpand`関数よりも少し便利なマクロ[` @ macroexpand`](@ ref)も存在します：


```jldoctest sayhello2
julia> @macroexpand @sayhello "human"
:((println)("Hello, ", "human"))
```

### Hold up: why macros?
###ホールドアップ：なぜマクロ？


#We have already seen a function `f(::Expr...) -> Expr` in a previous section. 
前の節では関数 `f(:: Expr ...) - > Expr`をすでに見てきました。
#In fact, [`macroexpand()`](@ref) is also such a function. So, why do macros exist?
実際、[`macroexpand()`](@ ref)もそのような関数です。 では、マクロはなぜ存在するのですか？

#Macros are necessary because they execute when code is parsed, therefore, macros allow the programmer to generate and include fragments of customized code *before* the full program is run. 
マクロは、コードが解析されたときに実行されるため、マクロが必要です。そのため、マクロを使用すると、プログラム全体が実行される前にカスタマイズされたコードの断片を生成して含めることができます。

マクロは必要です、コードが解析されたときに実行されるからです, したがって、プログラマがカスタマイズされたコードの断片を生成し完全なプログラムが実行される,*前に*含めるすることを可能にします。
#To illustrate the difference, consider the following example:
違いを説明するために、次の例を考えてみましょう。

```jldoctest whymacros
julia> macro twostep(arg)
           println("I execute at parse time. The argument is: ", arg)
           return :(println("I execute at runtime. The argument is: ", $arg))
       end
@twostep (macro with 1 method)

julia> ex = macroexpand( :(@twostep :(1, 2, 3)) );
I execute at parse time. The argument is: $(Expr(:quote, :((1, 2, 3))))
```

#The first call to [`println()`](@ref) is executed when [`macroexpand()`](@ref) is called. 
[`println()`](@ ref)への最初の呼び出しは[`macroexpand()`](@ ref)が呼び出されたときに実行されます。
#The resulting expression contains *only* the second `println`:
結果の式には2番目の `println`のみが含まれます：

```jldoctest whymacros
julia> typeof(ex)
Expr

julia> ex
:((println)("I execute at runtime. The argument is: ", $(Expr(:copyast, :($(QuoteNode(:((1, 2, 3)))))))))

julia> eval(ex)
I execute at runtime. The argument is: (1, 2, 3)
```

### Macro invocation
###マクロ呼び出し

#Macros are invoked with the following general syntax:
マクロは、次の一般的な構文で呼び出されます。

```julia
@name expr1 expr2 ...
@name(expr1, expr2, ...)
```

#Note the distinguishing `@` before the macro name and the lack of commas between the argument expressions in the first form, and the lack of whitespace after `@name` in the second form. 
最初の形式の引数式と `@ name`の後ろの空白文字の間には、マクロ名の前に` @ `とコンマがないことが区別されます。
#The two styles should not be mixed. 
2つのスタイルを混在させないでください。
#For example, the following syntax is different from the examples above; it passes the tuple `(expr1, expr2, ...)` as one argument to the macro:
たとえば、次の構文は上記の例とは異なります。 1つの引数としてタプル `(expr1、expr2、...)`をマクロに渡します。

```julia
@name (expr1, expr2, ...)
```

#It is important to emphasize that macros receive their arguments as expressions, literals, or symbols. 
マクロは、引数が式、リテラル、またはシンボルとして受け取られることを強調することが重要です。
#One way to explore macro arguments is to call the [`show()`](@ref) function within the macro body:
マクロ引数を調べる1つの方法は、マクロ本体内の[`show()`](@ ref)関数を呼び出すことです：

```jldoctest
julia> macro showarg(x)
           show(x)
           # ... remainder of macro, returning an expression
       end
@showarg (macro with 1 method)

julia> @showarg(a)
:a

julia> @showarg(1+1)
:(1 + 1)

julia> @showarg(println("Yo!"))
:(println("Yo!"))
```

#In addition to the given argument list, every macro is passed extra arguments named `__source__` and `__module__`.
与えられた引数リストに加えて、すべてのマクロには、 `__source__`と` __module__`という余分な引数が渡されます。

#The argument `__source__` provides information (in the form of a `LineNumberNode` object) about the parser location of the `@` sign from the macro invocation.
引数 `__source__`は、マクロ呼び出しからの` @ `記号のパーサの位置に関する情報(` LineNumberNode`オブジェクトの形式)を提供します。
#This allows macros to include better error diagnostic information, and is commonly used by logging, string-parser macros, and docs, for example, as well as to implement the `@__LINE__`, `@__FILE__`, and `@__DIR__` macros.
これにより、マクロにはより良いエラー診断情報を含めることができ、例えば、 `@__ LINE__`、` @__ FILE__`、 `@__DIR__`マクロを実装するだけでなく、ロギング、文字列解析マクロ、 。

#The location information can be accessed by referencing `__source__.line` and `__source__.file`:
位置情報は `__source __。line`と` __source __。file`を参照することでアクセスできます：k

```jldoctest
julia> macro __LOCATION__(); return QuoteNode(__source__); end
@__LOCATION__ (macro with 1 method)

julia> dump(
            @__LOCATION__(
       ))
LineNumberNode
  line: Int64 2
  file: Symbol none
```

#The argument `__module__` provides information (in the form of a `Module` object) about the expansion context of the macro invocation.
引数 `__module__`は、マクロ呼び出しの展開コンテキストに関する情報(` Module`オブジェクトの形式)を提供します。
#This allows macros to look up contextual information, such as existing bindings, or to insert the value as an extra argument to a runtime function call doing self-reflection in the current module.
これにより、マクロは既存のバインディングなどのコンテキスト情報を検索したり、現在のモジュールで自己反映しているランタイム関数呼び出しに追加の引数として値を挿入したりすることができます。

### Building an advanced macro
###高度なマクロを構築する

#Here is a simplified definition of Julia's `@assert` macro:
Juliaの `@ assert`マクロの単純化された定義は次のとおりです：

```jldoctest building
julia> macro assert(ex)
           return :( $ex ? nothing : throw(AssertionError($(string(ex)))) )
       end
@assert (macro with 1 method)
```

#This macro can be used like this:
このマクロは次のように使用できます：

```jldoctest building
julia> @assert 1 == 1.0

julia> @assert 1 == 0
ERROR: AssertionError: 1 == 0
```

#In place of the written syntax, the macro call is expanded at parse time to its returned result.
書かれた構文の代わりに、マクロ呼び出しは解析時に返された結果に展開されます。
#This is equivalent to writing:
これは以下のように書くことと同じです：

```julia
1 == 1.0 ? nothing : throw(AssertionError("1 == 1.0"))
1 == 0 ? nothing : throw(AssertionError("1 == 0"))
```

#That is, in the first call, the expression `:(1 == 1.0)` is spliced into the test condition slot, while the value of `string(:(1 == 1.0))` is spliced into the assertion message slot. 
つまり、最初の呼び出しでは、文字列(:( 1 == 1.0))の値がアサーションメッセージスロットにスプライスされている間に、最初の呼び出しで、式： `(1 == 1.0)`がテスト条件スロットにスプライスされます。
#The entire expression, thus constructed, is placed into the syntax tree where the `@assert` macro call occurs.
このようにして構築された式全体は、 `@ assert`マクロ呼び出しが発生するシンタックスツリーに置かれます。
#Then at execution time, if the test expression evaluates to true, then `nothing` is returned, whereas if the test is false, an error is raised indicating the asserted expression that was false.
そして、実行時に、テスト式が真と評価された場合には「何も」が返され、テストが偽であれば、アサートされた偽の式を示すエラーが発生します。
#Notice that it would not be possible to write this as a function, since only the *value* of the condition is available and it would be impossible to display the expression that computed it in the error message.
条件の*値*のみが使用可能であり、エラーメッセージでそれを計算した式を表示することは不可能なので、これを関数として書くことはできないことに注意してください。

#The actual definition of `@assert` in the standard library is more complicated. 
標準ライブラリの `@ assert`の実際の定義はもっと複雑です。
#It allows the user to optionally specify their own error message, instead of just printing the failed expression.
失敗した式を印刷するのではなく、ユーザーがオプションで独自のエラーメッセージを指定することができます。
#Just like in functions with a variable number of arguments, this is specified with an ellipses following the last argument:
可変数の引数を持つ関数と同様に、これは最後の引数に続く省略記号で指定されます。

```jldoctest assert2
julia> macro assert(ex, msgs...)
           msg_body = isempty(msgs) ? ex : msgs[1]
           msg = string(msg_body)
           return :($ex ? nothing : throw(AssertionError($msg)))
       end
@assert (macro with 1 method)
```

#Now `@assert` has two modes of operation, depending upon the number of arguments it receives!
今すぐ `@ assert`には受け取る引数の数に応じて2つの操作モードがあります！
#If there's only one argument, the tuple of expressions captured by `msgs` will be empty and it will behave the same as the simpler definition above. 
引数が1つだけの場合、 `msgs`で取り込まれた式のタプルは空になり、上のより単純な定義と同じように動作します。
#But now if the user specifies a second argument, it is printed in the message body instead of the failing expression. You can inspect the result of a macro expansion with the aptly named [`macroexpand()`](@ref) function:
しかし、ユーザーが2番目の引数を指定すると、失敗した式の代わりにメッセージ本文に出力されます。 適切な名前の[`macroexpand()`](@ ref)関数を使って、マクロ展開の結果を調べることができます：

```jldoctest assert2
julia> macroexpand(:(@assert a == b))
:(if a == b
        nothing
    else
        (throw)((AssertionError)("a == b"))
    end)

julia> macroexpand(:(@assert a==b "a should equal b!"))
:(if a == b
        nothing
    else
        (throw)((AssertionError)("a should equal b!"))
    end)
```

#There is yet another case that the actual `@assert` macro handles: what if, in addition to printing "a should equal b," we wanted to print their values? One might naively try to use string interpolation in the custom message, e.g., `@assert a==b "a ($a) should equal b ($b)!"`, but this won't work as expected with the above macro. 
実際の `@ assert`マクロは次のような場合もあります：もし" aはbと等しくなければならない "という印字に加えて、その値を出力したいのですか？ `@assert a == b"($ a)はb($ b)と等しくなければなりません！ ""しかし、これは期待通りに動作しません マクロ。
#Can you see why? Recall from [string interpolation](@ref string-interpolation) that an interpolated string is rewritten to a call to [`string()`](@ref). Compare:
なぜ見えますか？ 補間された文字列が[`string()`](@ ref)の呼び出しに書き直されることを思い出してください[@文字列補間](@ ref文字列補間) 比較：

```jldoctest
julia> typeof(:("a should equal b"))
String

julia> typeof(:("a ($a) should equal b ($b)!"))
Expr

julia> dump(:("a ($a) should equal b ($b)!"))
Expr
  head: Symbol string
  args: Array{Any}((5,))
    1: String "a ("
    2: Symbol a
    3: String ") should equal b ("
    4: Symbol b
    5: String ")!"
  typ: Any
```

#So now instead of getting a plain string in `msg_body`, the macro is receiving a full expression that will need to be evaluated in order to display as expected. 
だから、 `msg_body`にプレーンな文字列を得るのではなく、マクロは期待どおりに表示するために評価が必要な完全な式を受け取っています。
#This can be spliced directly into the returned expression as an argument to the [`string()`](@ref) call; see [`error.jl`](https://github.com/JuliaLang/julia/blob/master/base/error.jl) for the complete implementation.
これは[`string()`](@ ref)呼び出しの引数として、返された式に直接スプライスすることができます。 完全な実装については、[`error.jl`](https://github.com/JuliaLang/julia/blob/master/base/error.jl)を参照してください。

#The `@assert` macro makes great use of splicing into quoted expressions to simplify the manipulation of expressions inside the macro body.
`@ assert`マクロは、引用された式へのスプライシングを大いに利用して、マクロ本体内部の式の操作を簡単にします。

### Hygiene
###衛生

#An issue that arises in more complex macros is that of [hygiene](https://en.wikipedia.org/wiki/Hygienic_macro).
より複雑なマクロで発生する問題は、[衛生](https://en.wikipedia.org/wiki/Hygienic_macro)の問題です。
#In short, macros must ensure that the variables they introduce in their returned expressions do not accidentally clash with existing variables in the surrounding code they expand into. 
要するに、マクロは、返される式で導入される変数が、周囲のコードの既存の変数と突然衝突することを確実にしなければなりません。
#Conversely, the expressions that are passed into a macro as arguments are often *expected* to evaluate in the context of the surrounding code, interacting with and modifying the existing variables. 
逆に、引数としてマクロに渡される式は、しばしば周囲のコードのコンテキストで評価され、既存の変数と相互作用し、変更することが期待されます。
#Another concern arises from the fact that a macro may be called in a different module from where it was defined. 
別の問題は、定義された場所とは異なるモジュールでマクロが呼び出されるという事実から発生します。
#In this case we need to ensure that all global variables are resolved to the correct module. 
この場合、すべてのグローバル変数が正しいモジュールに解決されるようにする必要があります。
#Julia already has a major advantage over languages with textual macro expansion (like C) in that it only needs to consider the returned expression. 
Juliaは、返された式だけを考慮する必要があるという点で、(Cのような)テキストマクロ展開の言語よりも大きな利点を既に持っています。
#All the other variables (such as `msg` in `@assert` above) follow the [normal scoping block behavior](@ref scope-of-variables).
他のすべての変数(上記の `@ assert`の` msg`など)は、[通常のスコープブロックの振る舞い](@ refスコープオブ変数)に従います。

#To demonstrate these issues, let us consider writing a `@time` macro that takes an expression as its argument, records the time, evaluates the expression, records the time again, prints the difference between the before and after times, and then has the value of the expression as its final value. The macro might look like this:
これらの問題を示すために、式を引数としてとり、時間を記録し、式を評価し、時間を再度記録し、前後の差を出力し、その後に時間を記録する `@time`マクロを作成することを考えてみましょう。その最終値としての式の値。マクロは次のようになります。

```julia
macro time(ex)
    return quote
        local t0 = time()
        local val = $ex
        local t1 = time()
        println("elapsed time: ", t1-t0, " seconds")
        val
    end
end
```
#Here, we want `t0`, `t1`, and `val` to be private temporary variables, and we want `time` to refer to the [`time()`](@ref) function in the standard library, not to any `time` variable the user might have (the same applies to `println`). 
ここでは、 `t0`、` t1`、 `val`をプライベートな一時変数にして、` time`が標準ライブラリの[`time()`](@ref)関数を参照するようにします。 ( `println`にも同じことが言えます)。
#Imagine the problems that could occur if the user expression `ex` also contained assignments to a variable called `t0`, or defined its own `time` variable. 
ユーザエクスプレッション `ex`が` t0`という変数への代入を含んでいたり、それ自身の `time`変数を定義していた場合に起こりうる問題を想像してください。
#We might get errors, or mysteriously incorrect behavior.
私たちは間違いや不思議な誤動作をするかもしれません。

#Julia's macro expander solves these problems in the following way. 
Juliaのマクロエクスパンダは、次のようにしてこれらの問題を解決します。
#First, variables within a macro result are classified as either local or global. 
まず、マクロ結果内の変数は、ローカルまたはグローバルのいずれかに分類されます。
#A variable is considered local if it is assigned to (and not declared global), declared local, or used as a function argument name. 
変数は、変数が代入されている(そしてグローバル宣言されていない)、ローカル宣言されている、または関数の引数名として使用されている場合、ローカルと見なされます。
#Otherwise, it is considered global. 
それ以外の場合は、グローバルとみなされます。
#Local variables are then renamed to be unique (using the [`gensym()`](@ref) function, which generates new symbols), and global variables are resolved within the macro definition environment. 
ローカル変数は、新しいシンボルを生成する[`gensym()`](@ref)関数を使用して一意になるように名前が変更され、グローバル変数はマクロ定義環境内で解決されます。
#Therefore both of the above concerns are handled; the macro's locals will not conflict with any user variables, and `time` and `println` will refer to the standard library definitions.
したがって、上記の懸念事項の両方が処理されます。マクロのローカル変数はユーザー変数と競合しません。また、 `time`と` println`は標準ライブラリ定義を参照します。

#One problem remains however.
しかし、1つの問題が残っている。
#Consider the following use of this macro:
このマクロの次の使用を考えてみましょう。

```julia
module MyModule
import Base.@time

time() = ... # compute something

@time time()
end
```

#Here the user expression `ex` is a call to `time`, but not the same `time` function that the macro uses. 
ここでは、ユーザー式 `ex`は` time`の呼び出しですが、マクロが使用する `time`関数とは異なります。
#It clearly refers to `MyModule.time`. Therefore we must arrange for the code in `ex` to be resolved in the macro call environment. 
これは明らかに `MyModule.time`を参照しています。 したがって、マクロ呼び出し環境で解決するには、exのコードを手配しなければなりません。
#This is done by "escaping" the expression with [`esc()`](@ref):
これは、式を[`esc()`](@ ref)で "エスケープ"することによって行われます：

```julia
macro time(ex)
    ...
    local val = $(esc(ex))
    ...
end
```

#An expression wrapped in this manner is left alone by the macro expander and simply pasted into the output verbatim. 
このようにしてラップされた式は、マクロエクスパンダによってそのまま残され、そのまま出力に貼り付けられます。
#Therefore it will be resolved in the macro call environment.
したがって、マクロ呼び出し環境で解決されます。

#This escaping mechanism can be used to "violate" hygiene when necessary, in order to introduce or manipulate user variables.
このエスケープ・メカニズムは、ユーザー変数を導入または操作するために、必要に応じて衛生状態を「違反する」ために使用できます。
#For example, the following macro sets `x` to zero in the call environment:
たとえば、次のマクロは呼び出し環境で `x`を0に設定します。

```jldoctest
julia> macro zerox()
           return esc(:(x = 0))
       end
@zerox (macro with 1 method)

julia> function foo()
           x = 1
           @zerox
           return x # is zero
       end
foo (generic function with 1 method)

julia> foo()
0
```

#This kind of manipulation of variables should be used judiciously, but is occasionally quite handy.
このような変数の操作は、慎重に使用する必要がありますが、時には非常に便利です。

#Getting the hygiene rules correct can be a formidable challenge.
正しい衛生規則を取得することは、挑戦になる可能性があります。
#Before using a macro, you might want to consider whether a function closure would be sufficient. 
マクロを使用する前に、関数クロージャーで十分かどうかを検討することをお勧めします。
#Another useful strategy is to defer as much work as possible to runtime. 
もう1つの有用な戦略は、可能な限り多くの作業をランタイムに任せることです。
#For example, many macros simply wrap their arguments in a QuoteNode or other similar Expr. 
例えば、多くのマクロはQuoteNodeや他の同様のExprに引数を単にラップします。
#Some examples of this include `@task body` which simply returns `schedule(Task(() -> $body))`, and `@eval expr`, which simply returns `eval(QuoteNode(expr))`.
`schedule(Task(() - > $ body))`と単に `eval(QuoteNode(expr))`を返す `@eval expr`を返す` @task body`があります。

#To demonstrate, we might rewrite the `@time` example above as:
例を示すために、上の `@ time`の例を次のように書き直すかもしれません：

```julia
macro time(expr)
    return :(timeit(() -> $(esc(expr))))
end
function timeit(f)
    t0 = time()
    val = f()
    t1 = time()
    println("elapsed time: ", t1-t0, " seconds")
    return val
end
```

#However, we don't do this for a good reason: wrapping the `expr` in a new scope block (the anonymous function) also slightly changes the meaning of the expression (the scope of any variables in it), while we want `@time` to be usable with minimum impact on the wrapped code.
しかし、正当な理由でこれを行うわけではありません。新しいスコープブロック(無名関数)に `expr`をラップすると、式の意味(その中の変数のスコープ)も少し変更されますが、 ラップされたコードへの影響を最小限に抑えて使用できるようになります。

## Code Generation
##コード生成

#When a significant amount of repetitive boilerplate code is required, it is common to generate it programmatically to avoid redundancy. 
かなりの量の繰り返し定型コードが必要な場合は、冗長性を避けるためにプログラムで生成するのが一般的です。
#In most languages, this requires an extra build step, and a separate program to generate the repetitive code. 
ほとんどの言語では、追加のビルドステップと、繰り返しコードを生成する別のプログラムが必要です。
#In Julia, expression interpolation and [`eval()`](@ref) allow such code generation to take place in the normal course of program execution.
Juliaでは、式の補間と[`eval()`](@ ref)は、プログラムの実行の通常の過程でそのようなコード生成が行われるようにします。
#For example, the following code defines a series of operators on three arguments in terms of their 2-argument forms:
たとえば、次のコードでは、2つの引数の形式で3つの引数に一連の演算子を定義しています。

```julia
for op = (:+, :*, :&, :|, :$)
    eval(quote
        ($op)(a,b,c) = ($op)(($op)(a,b),c)
    end)
end
```

#In this manner, Julia acts as its own [preprocessor](https://en.wikipedia.org/wiki/Preprocessor), and allows code generation from inside the language. 
このように、Juliaは独自の[プリプロセッサ](https://en.wikipedia.org/wiki/Preprocessor)として機能し、言語内からのコード生成を可能にします。
#The above code could be written slightly more tersely using the `:` prefix quoting form:
上記のコードは、 `：`プレフィックスクォートフォームを使用してやや簡潔に書くことができます：

```julia
for op = (:+, :*, :&, :|, :$)
    eval(:(($op)(a,b,c) = ($op)(($op)(a,b),c)))
end
```

#This sort of in-language code generation, however, using the `eval(quote(...))` pattern, is common enough that Julia comes with a macro to abbreviate this pattern:
しかし、 `eval(quote(...)) 'パターンを使用したこの種の言語内コード生成は、Juliaにこのパターンを省略するためのマクロが付いているほど一般的です。

```julia
for op = (:+, :*, :&, :|, :$)
    @eval ($op)(a,b,c) = ($op)(($op)(a,b),c)
end
```

#The [`@eval`](@ref) macro rewrites this call to be precisely equivalent to the above longer versions.
[`@ eval`](@ ref)マクロは、上記のより長いバージョンと正確に等価であるようにこの呼び出しを書き換えます。
#For longer blocks of generated code, the expression argument given to [`@eval`](@ref) can be a block:
生成されたコードのより長いブロックの場合、[`@ eval`](@ ref)に与えられた式の引数はブロックになります：

```julia
@eval begin
    # multiple lines
end
```

## Non-Standard String Literals
##非標準文字列リテラル

#Recall from [Strings](@ref non-standard-string-literals) that string literals prefixed by an identifier are called non-standard string literals, and can have different semantics than un-prefixed string literals. 
識別子が前に付いた文字列リテラルは非標準文字列リテラルと呼ばれ、接頭辞のない文字列リテラルとは異なるセマンティクスを持つことができる[[Strings](@ ref非標準文字列リテラル)を思い出してください。
#For example:
例えば：

   # * `r"^\s*(?:#|$)"` produces a regular expression object rather than a string
   `r" ^ \ s *(？：＃| $) "`は文字列ではなく正規表現オブジェクトを生成します
   # * `b"DATA\xff\u2200"` is a byte array literal for `[68,65,84,65,255,226,136,128]`.
   `b" DATA \ xff \ u2200 "は、[68,65,84,65,255,226,136,128]のバイト配列リテラルです。

#Perhaps surprisingly, these behaviors are not hard-coded into the Julia parser or compiler. 
おそらく驚くべきことに、これらの振る舞いはJuliaパーサまたはコンパイラにハードコードされていません。
#Instead, they are custom behaviors provided by a general mechanism that anyone can use: prefixed string literals are parsed as calls to specially-named macros. 
代わりに、誰でも使用できる一般的なメカニズムによって提供されるカスタムビヘイビアです。接頭辞付きの文字列リテラルは、特別な名前のマクロへの呼び出しとして解析されます。
#For example, the regular expression macro is just the following:
たとえば、正規表現マクロは次のようになります。

```julia
macro r_str(p)
    Regex(p)
end
```

#That's all. 
それで全部です。
#This macro says that the literal contents of the string literal `r"^\s*(?:#|$)"` should be passed to the `@r_str` macro and the result of that expansion should be placed in the syntax tree where the string literal occurs. 
このマクロは、文字列リテラル `r" ^ \ s *(？：＃| $) "のリテラルの内容は` @ r_str`マクロに渡されなければならず、その展開の結果は構文木 ここで文字列リテラルが発生します。
#In other words, the expression `r"^\s*(?:#|$)"` is equivalent to placing the following object directly into the syntax tree:
言い換えれば、式 "r" ^ \ s *(？：＃| $) "`は、以下のオブジェクトを構文木に直接配置するのと同じです：

```julia
Regex("^\\s*(?:#|\$)")
```

#Not only is the string literal form shorter and far more convenient, but it is also more efficient:
リテラル文字列の形式は短くてはるかに便利なだけでなく、より効率的です。
#since the regular expression is compiled and the `Regex` object is actually created *when the code is compiled*, the compilation occurs only once, rather than every time the code is executed. 
正規表現がコンパイルされ、コードがコンパイルされたときに `Regex`オブジェクトが実際に作成される*ため、コンパイルはコードが実行されるたびにではなく1回だけ実行されます。
#Consider if the regular expression occurs in a loop:
正規表現がループ内で発生するかどうかを検討してください。

```julia
for line = lines
    m = match(r"^\s*(?:#|$)", line)
    if m === nothing
        # non-comment
    else
        # comment
    end
end
```

#Since the regular expression `r"^\s*(?:#|$)"` is compiled and inserted into the syntax tree when this code is parsed, the expression is only compiled once instead of each time the loop is executed.
このコードが解析されるとき、正規表現 `r" ^ \ s *(？：＃| $) "が構文木にコンパイルされて挿入されるので、式はループが実行されるたびにではなく、一度だけコンパイルされます。
#In order to accomplish this without macros, one would have to write this loop like this:
これをマクロなしで実現するには、次のようにこのループを書く必要があります：

```julia
re = Regex("^\\s*(?:#|\$)")
for line = lines
    m = match(re, line)
    if m === nothing
        # non-comment
    else
        # comment
    end
end
```

#Moreover, if the compiler could not determine that the regex object was constant over all loops, certain optimizations might not be possible, making this version still less efficient than the more convenient literal form above. 
さらに、コンパイラが正規表現オブジェクトがすべてのループにわたって一定であると判断できない場合、特定の最適化が可能でない可能性があり、このバージョンは上記のより便利なリテラルフォームよりも効率が悪くなります。
#Of course, there are still situations where the non-literal form is more convenient: if one needs to interpolate a variable into the regular expression, one must take this more verbose approach; in cases where the regular expression pattern itself is dynamic, potentially changing upon each loop iteration, a new regular expression object must be constructed on each iteration. 
もちろん、非リテラル形式がより便利な状況がまだあります。変数を正規表現に補間する必要がある場合は、このより冗長なアプローチをとる必要があります。正規表現パターン自体が動的で、ループの繰り返しごとに変更される可能性がある場合は、新しい正規表現オブジェクトを各反復で構築する必要があります。
#In the vast majority of use cases, however, regular expressions are not constructed based on run-time data. 
ただし、大部分のユースケースでは、実行時データに基づいて正規表現は構築されません。
#In this majority of cases, the ability to write regular expressions as compile-time values is invaluable.
この大部分のケースでは、コンパイル時の値として正規表現を書く能力は非常に貴重です。

#Like non-standard string literals, non-standard command literals exist using a prefixed variant of the command literal syntax. The command literal ```custom`literal` ``` is parsed as `@custom_cmd "literal"`.
非標準文字列リテラルと同様に、非標準のコマンドリテラルは、コマンドリテラル構文の接頭辞付きの変形を使用して存在します。コマンドリテラル `` `custom`literal``` ``は `@custom_cmd"リテラル "`として解析されます。
#Julia itself does not contain any non-standard command literals, but packages can make use of this syntax. 
Julia自体には標準以外のコマンドリテラルは含まれていませんが、パッケージはこの構文を使用できます。
#Aside from the different syntax and the `_cmd` suffix instead of the `_str` suffix, non-standard command literals behave exactly like non-standard string literals.
`_str`接尾辞の代わりに異なる構文と` _cmd`接尾辞を除いて、非標準のコマンドリテラルは、標準でない文字列リテラルとまったく同じように動作します。

#In the event that two modules provide non-standard string or command literals with the same name, it is possible to qualify the string or command literal with a module name. 
2つのモジュールが同じ名前の非標準文字列またはコマンドリテラルを提供する場合は、文字列またはコマンドリテラルをモジュール名で修飾することができます。
#For instance, if both `Foo` and `Bar` provide non-standard string literal `@x_str`, then one can write `Foo.x"literal"` or `Bar.x"literal"` to disambiguate between the two.
たとえば、 `Foo`と` Bar`の両方が非標準の文字列リテラル `@ x_str`を提供する場合、` Foo.x "リテラル"または `Bar.x"リテラル "を記述して、両者を明確にすることができます。

#The mechanism for user-defined string literals is deeply, profoundly powerful. Not only are Julia's non-standard literals implemented using it, but also the command literal syntax (``` `echo "Hello, $person"` ```) is implemented with the following innocuous-looking macro:
ユーザー定義の文字列リテラルのメカニズムは深く、深く強力です。 Juliaの非標準リテラルは、それを使用して実装されているだけでなく、コマンドリテラル構文( `` `echo" Hello、$ person "` `` `)も次の無害な外観のマクロ​​で実装されています：

```julia
macro cmd(str)
    :(cmd_gen($(shell_parse(str)[1])))
end
```

#Of course, a large amount of complexity is hidden in the functions used in this macro definition, but they are just functions, written entirely in Julia. 
もちろん、このマクロ定義で使用されている関数には複雑さが大量に隠されていますが、それらは単なる関数であり、Juliaで完全に書かれています。
#You can read their source and see precisely what they do -- and all they do is construct expression objects to be inserted into your program's syntax tree.
ソースを読んで、彼らが何をするのかを正確に見ることができます。プログラムの構文ツリーに挿入する式オブジェクトを作成するだけです。

## Generated functions
##生成された関数

#A very special macro is `@generated`, which allows you to define so-called *generated functions*.
非常に特殊なマクロは `@ generated`です。これはいわゆる*生成関数*を定義することを可能にします。
#These have the capability to generate specialized code depending on the types of their arguments with more flexibility and/or less code than what can be achieved with multiple dispatch. 
これらは、複数のディスパッチで達成できるものよりも柔軟性が高く、コードの数が少ない、引数の種類に応じて特殊なコードを生成する機能を備えています。
#While macros work with expressions at parsing-time and cannot access the types of their inputs, a generated function gets expanded at a time when the types of the arguments are known, but the function is not yet compiled.
マクロは構文解析時に式を処理し、入力の型にアクセスすることはできませんが、引数の型がわかっているにも関わらず生成された関数は展開されますが、関数はまだコンパイルされていません。

#Instead of performing some calculation or action, a generated function declaration returns a quoted expression which then forms the body for the method corresponding to the types of the arguments.
呼び出されると、ボディ式が最初に評価され、コンパイルされ、返された式がコンパイルされて実行されます。
#When called, the body expression is first evaluated and compiled, then the returned expression is compiled and run. 
いくつかの計算またはアクションを実行する代わりに、生成された関数宣言は引用された式を返し、次に引数の型に対応するメソッドの本体を形成します。
#To make this efficient, the result is often cached. 
これを効率的にするために、結果はしばしばキャッシュされます。
#And to make this inferable, only a limited subset of the language is usable. 
これを推測できるようにするには、言語の限られたサブセットしか使用できません。
#Thus, generated functions provide a flexible framework to move work from run-time to compile-time, at the expense of greater restrictions on the allowable constructs.
このように、生成された関数は、実行可能な時間からコンパイル時に作業を移すための柔軟なフレームワークを提供します。

#When defining generated functions, there are four main differences to ordinary functions:
生成された関数を定義するとき、通常の関数との主な違いは4つあります。

1. 
   # You annotate the function declaration with the `@generated` macro. This adds some information to the AST that lets the compiler know that this is a generated function.
    関数宣言に `@ generated`マクロを付けてアノテーションを付けます。 これにより、コンパイラに生成された関数であることを知らせる情報がASTに追加されます。
2. 
   # In the body of the generated function you only have access to the *types* of the arguments – not their values – and any function that was defined *before* the definition of the generated function.
    生成された関数の本体では、その値ではなく引数の* types *と、生成された関数の定義の前に*定義された関数のみにアクセスできます。
3. 
   # Instead of calculating something or performing some action, you return a *quoted expression* which, when evaluated, does what you want.
    何かを計算するか、何らかのアクションを実行する代わりに、*評価されたときにあなたが望むことをする*引用された式を返します。
4. 
   # Generated functions must not *mutate* or *observe* any non-constant global state (including, for example, IO, locks, non-local dictionaries, or using `method_exists`). 
    生成された関数は、非定数グローバル状態(IO、ロック、非ローカル辞書、 `method_exists`を含む)を変更*したり、*観察したりしてはいけません。
   # This means they can only read global constants, and cannot have any side effects. 
     つまり、グローバル定数のみを読み取ることができ、副作用はありません。
   # In other words, they must be completely pure. 
     換言すれば、それらは完全に純粋でなければならない。
   # Due to an implementation limitation, this also means that they currently cannot define a closure or untyped generator.
     実装上の制約から、これは現在クロージャーまたは型なしのジェネレーターを定義できないことを意味します。

#It's easiest to illustrate this with an example. We can declare a generated function `foo` as
例を使ってこれを説明するのが最も簡単です。 生成された関数 `foo`を

```jldoctest generated
julia> @generated function foo(x)
           Core.println(x)
           return :(x * x)
       end
foo (generic function with 1 method)
```

#Note that the body returns a quoted expression, namely `:(x * x)`, rather than just the value of `x * x`.
本文は `x * x`の値ではなく、引用された式、すなわち`：(x * x) `を返すことに注意してください。

#From the caller's perspective, they are very similar to regular functions; in fact, you don't have to know if you're calling a regular or generated function - the syntax and result of the call is just the same. 
呼び出し元の観点から見ると、それらは通常の関数と非常によく似ています。 実際には、正規関数や生成関数を呼び出すかどうかを知る必要はありません。呼び出しの構文と結果はまったく同じです。
#Let's see how `foo` behaves:
`foo`の動作を見てみましょう：

```jldoctest generated
julia> x = foo(2); # note: output is from println() statement in the body
Int64

julia> x           # now we print x
4

julia> y = foo("bar");
String

julia> y
"barbar"
```

#So, we see that in the body of the generated function, `x` is the *type* of the passed argument, and the value returned by the generated function, is the result of evaluating the quoted expression we returned from the definition, now with the *value* of `x`.
したがって、生成された関数の本体では、 `x`は渡された引数の* type *であり、生成された関数によって返される値は、定義から返された引用式を評価した結果です `x`の* value *と比較します。

#What happens if we evaluate `foo` again with a type that we have already used?
すでに使用している型の `foo`を再度評価するとどうなりますか？

```jldoctest generated
julia> foo(4)
16
```
#Note that there is no printout of [`Int64`](@ref). 
[`Int64`](@ ref)の出力はありません。
#We can see that the body of the generated function was only executed once here, for the specific set of argument types, and the result was cached.
生成された関数の本体は、特定の引数型のセットに対してここで一度だけ実行され、その結果がキャッシュされていることがわかります。
#After that, for this example, the expression returned from the generated function on the first invocation was re-used as the method body. 
その後、この例では、最初の呼び出しで生成された関数から戻された式がメソッド本体として再利用されました。
#However, the actual caching behavior is an implementation-defined performance optimization, so it is invalid to depend too closely on this behavior.
ただし、実際のキャッシュ動作は実装定義のパフォーマンス最適化であるため、この動作にあまり依存しないことは無効です。

#The number of times a generated function is generated *might* be only once, but it *might* also be more often, or appear to not happen at all. 
生成された関数が生成される回数は、たった1回かもしれませんが、より頻繁に起こるかもしれないし、まったく起こらないかもしれません。
#As a consequence, you should *never* write a generated function with side effects - when, and how often, the side effects occur is undefined. 
結果として、副作用を伴う生成関数を書くことは決して*しないでください。副作用が発生する頻度と頻度は定義されていません。
#(This is true for macros too - and just like for macros, the use of [`eval()`](@ref) in a generated function is a sign that you're doing something the wrong way.) 
(これはマクロにも当てはまります。マクロの場合と同じように、生成された関数で[`eval()`](@ ref)を使うのは間違ったやり方をしているという印です)。
#However, unlike macros, the runtime system cannot correctly handle a call to [`eval()`](@ref), so it is disallowed.
しかし、マクロとは異なり、ランタイムシステムは[`eval()`](@ ref)の呼び出しを正しく扱うことができないため、許可されません。

#It is also important to see how `@generated` functions interact with method redefinition. 
`@ generated`関数がメソッドの再定義とどのように相互作用するかを知ることも重要です。
#Following the principle that a correct `@generated` function must not observe any mutable state or cause any mutation of global state, we see the following behavior.
正しい `@ generated`関数が可変状態を観察してはならない、またはグローバル状態の任意の突然変異を引き起こさないという原則に従って、我々は以下の振る舞いを見る。
#Observe that the generated function *cannot* call any method that was not defined prior to the *definition* of the generated function itself.
生成された関数*は、生成された関数自体の*定義*より前に定義されていないメソッドを呼び出すことはできないことに注意してください。

#Initially `f(x)` has one definition
最初に`f(x)`には1つの定義があります

```jldoctest redefinition
julia> f(x) = "original definition";
```

#Define other operations that use `f(x)`:
`f(x)`を使う他の操作を定義する：

```jldoctest redefinition
julia> g(x) = f(x);

julia> @generated gen1(x) = f(x);

julia> @generated gen2(x) = :(f(x));
```

We now add some new definitions for `f(x)`:
`f(x)`の新しい定義を追加します：

```jldoctest redefinition
julia> f(x::Int) = "definition for Int";

julia> f(x::Type{Int}) = "definition for Type{Int}";
```

#and compare how these results differ:
これらの結果がどのように異なるかを比較する：

```jldoctest redefinition
julia> f(1)
"definition for Int"

julia> g(1)
"definition for Int"

julia> gen1(1)
"original definition"

julia> gen2(1)
"definition for Int"
```

#Each method of a generated function has its own view of defined functions:
生成された関数の各メソッドには、定義された関数の独自のビューがあります。

```jldoctest redefinition
julia> @generated gen1(x::Real) = f(x);

julia> gen1(1)
"definition for Type{Int}"
```

#The example generated function `foo` above did not do anything a normal function `foo(x) = x * x` could not do (except printing the type on the first invocation, and incurring higher overhead).
上の例の生成された関数 `foo`は、通常の関数` foo(x)= x * x`は実行できませんでした(最初の呼び出しで型を出力することとオーバーヘッドが高くなります)。
#However, the power of a generated function lies in its ability to compute different quoted expressions depending on the types passed to it:
しかし、生成される関数の能力は、渡される型に応じて異なる引用式を計算する能力にあります。

```jldoctest
julia> @generated function bar(x)
           if x <: Integer
               return :(x ^ 2)
           else
               return :(x)
           end
       end
bar (generic function with 1 method)

julia> bar(4)
16

julia> bar("baz")
"baz"
```

#(although of course this contrived example would be more easily implemented using multiple dispatch...)
(もちろん、この人工的な例は、複数のディスパッチを使ってより簡単に実装できます...)

#Abusing this will corrupt the runtime system and cause undefined behavior:
これを酷使すると、ランタイムシステムが破壊され、未定義の動作が発生します。

```jldoctest
julia> @generated function baz(x)
           if rand() < .9
               return :(x^2)
           else
               return :("boo!")
           end
       end
baz (generic function with 1 method)
```

#Since the body of the generated function is non-deterministic, its behavior, *and the behavior of all subsequent code* is undefined.
生成された関数の本体は非決定論的なので、その振る舞い、*およびすべての後続のコード*の動作は未定義です。

#*Don't copy these examples!*
*これらの例をコピーしないでください！*

#These examples are hopefully helpful to illustrate how generated functions work, both in the definition end and at the call site; however, *don't copy them*, for the following reasons:
これらの例は、定義の最後とコールサイトの両方で、生成された関数がどのように機能するかを説明するうえで有益です。 ただし、次の理由により*コピーしない*

  * 
   # the `foo` function has side-effects (the call to `Core.println`), and it is undefined exactly when, how often or how many times these side-effects will occur
    `foo`関数は副作用(` Core.println`への呼び出し)を持っています。これらの副作用がいつ、何回、何回起こるかは正確には定義されていません
  * 
   # the `bar` function solves a problem that is better solved with multiple dispatch - defining `bar(x) = x` and `bar(x::Integer) = x ^ 2` will do the same thing, but it is both simpler and faster.
   `bar`関数は複数のディスパッチでよりよく解決される問題を解決します。 `bar(x)= x`と` bar(x :: Integer)= x ^ 2`を定義すると同じことが起こります。 しかし、それはより簡単で高速です。
  * 
   # the `baz` function is pathologically insane
   「バズ」機能は病理学的には狂っている

#Note that the set of operations that should not be attempted in a generated function is unbounded, and the runtime system can currently only detect a subset of the invalid operations. 
生成された関数で試行すべきではない操作の組は無制限であり、ランタイムシステムは現在無効な操作の部分集合しか検出できないことに注意してください。
#There are many other operations that will simply corrupt the runtime system without notification, usually in subtle ways not obviously connected to the bad definition. 
ランタイムシステムを通知なしで単に破損させる他の多くの操作がありますが、通常は微妙な方法で悪い定義に接続されていません。
#Because the function generator is run during inference, it must respect all of the limitations of that code.
関数ジェネレータは推論の間に実行されるため、そのコードのすべての制限を守る必要があります。

#Some operations that should not be attempted include:
試行してはいけない操作には、次のものがあります。
1. 
   # Caching of native pointers.
    ネイティブポインタのキャッシング
2. 
   # Interacting with the contents or methods of Core.Inference in any way.
Core.Inferenceの内容またはメソッドとのやり取り
3. 
   # Observing any mutable state.
    変更可能な状態を観察する。

   #  * Inference on the generated function may be run at *any* time, including while your code is attempting to observe or mutate this state.
   * 生成された関数の推論は、あなたのコードがこの状態を観察または突然変異させようとしている間も含めて、いつでも実行することができます。
4. 
   # Taking any locks: C code you call out to may use locks internally, (for example, it is not problematic to call `malloc`, even though most implementations require locks internally) but don't attempt to hold or acquire any while executing Julia code.
   任意のロックを取る：あなたが呼び出すCコードは内部的にロックを使用するかもしれません。しかし、ジュリアコードを実行している間は何も保持したり取得したりしないでください。(例えば、ほとんどの実装が内部的にロックを必要とするにもかかわらず、`malloc`を呼び出すのは問題ありません)
5. 
   #Calling any function that is defined after the body of the generated function. This condition is relaxed for incrementally-loaded precompiled modules to allow calling any function in the module.
    生成された関数の本体の後に定義された関数を呼び出します。 この状態は、モジュール内の任意の関数を呼び出すことができるように、段階的にロードされたプリコンパイルされたモジュールのために緩和されます。

#Alright, now that we have a better understanding of how generated functions work, let's use them to build some more advanced (and valid) functionality...
さて、生成された関数がどのように機能するかをより深く理解したので、それらを使ってより高度な(そして有効な)機能を構築しましょう...

### An advanced example
###先進の例

#Julia's base library has a [`sub2ind()`](@ref) function to calculate a linear index into an n-dimensional array, based on a set of n multilinear indices - in other words, to calculate the index `i` that can be used to index into an array `A` using `A[i]`, instead of `A[x,y,z,...]`. 
Juliaの基底ライブラリには、n個の複数の線形インデックスの集合に基づいて線形インデックスを計算するための[`sub2ind()`](@ref)関数があります。 A [x、y、z、...]の代わりに `A [i]`を使って配列 `A 'にインデックスを付けるために使うことができます。
#One possible implementation is the following:
1つの可能な実装は次のとおりです。

```jldoctest sub2ind
julia> function sub2ind_loop(dims::NTuple{N}, I::Integer...) where N
           ind = I[N] - 1
           for i = N-1:-1:1
               ind = I[i]-1 + dims[i]*ind
           end
           return ind + 1
       end
sub2ind_loop (generic function with 1 method)

julia> sub2ind_loop((3, 5), 1, 2)
4
```

#The same thing can be done using recursion:
再帰を使って同じことができます：

```jldoctest
julia> sub2ind_rec(dims::Tuple{}) = 1;

julia> sub2ind_rec(dims::Tuple{}, i1::Integer, I::Integer...) =
           i1 == 1 ? sub2ind_rec(dims, I...) : throw(BoundsError());

julia> sub2ind_rec(dims::Tuple{Integer, Vararg{Integer}}, i1::Integer) = i1;

julia> sub2ind_rec(dims::Tuple{Integer, Vararg{Integer}}, i1::Integer, I::Integer...) =
           i1 + dims[1] * (sub2ind_rec(Base.tail(dims), I...) - 1);

julia> sub2ind_rec((3, 5), 1, 2)
4
```

#Both these implementations, although different, do essentially the same thing: a runtime loop over the dimensions of the array, collecting the offset in each dimension into the final index.
これらの実装は、どちらも基本的には同じですが、配列の次元にわたる実行時ループで、各次元のオフセットを最終索引に収集します。

#However, all the information we need for the loop is embedded in the type information of the arguments.
しかし、ループに必要なすべての情報は、引数の型情報に埋め込まれています。
#Thus, we can utilize generated functions to move the iteration to compile-time; in compiler parlance, we use generated functions to manually unroll the loop. 
したがって、生成された関数を利用して反復をコンパイル時に移動することができます。 コンパイラ用語では、生成された関数を使用してループを手動で展開します。
#The body becomes almost identical, but instead of calculating the linear index, we build up an *expression* that calculates the index:
ボディはほぼ同じになりますが、線形インデックスを計算する代わりに、インデックスを計算する*式*を構築します。

```jldoctest sub2ind_gen
julia> @generated function sub2ind_gen(dims::NTuple{N}, I::Integer...) where N
           ex = :(I[$N] - 1)
           for i = (N - 1):-1:1
               ex = :(I[$i] - 1 + dims[$i] * $ex)
           end
           return :($ex + 1)
       end
sub2ind_gen (generic function with 1 method)

julia> sub2ind_gen((3, 5), 1, 2)
4
```

**What code will this generate?**
**これはどのようなコードを生成しますか？**

#An easy way to find out is to extract the body into another (regular) function:
簡単な方法は、本体を別の(通常の)関数に抽出することです。

```jldoctest sub2ind_gen2
julia> @generated function sub2ind_gen(dims::NTuple{N}, I::Integer...) where N
           return sub2ind_gen_impl(dims, I...)
       end
sub2ind_gen (generic function with 1 method)

julia> function sub2ind_gen_impl(dims::Type{T}, I...) where T <: NTuple{N,Any} where N
           length(I) == N || return :(error("partial indexing is unsupported"))
           ex = :(I[$N] - 1)
           for i = (N - 1):-1:1
               ex = :(I[$i] - 1 + dims[$i] * $ex)
           end
           return :($ex + 1)
       end
sub2ind_gen_impl (generic function with 1 method)
```

#We can now execute `sub2ind_gen_impl` and examine the expression it returns:
`sub2ind_gen_impl`を実行し、それが返す式を調べることができます：

```jldoctest sub2ind_gen2
julia> sub2ind_gen_impl(Tuple{Int,Int}, Int, Int)
:(((I[1] - 1) + dims[1] * (I[2] - 1)) + 1)
```

#So, the method body that will be used here doesn't include a loop at all - just indexing into the two tuples, multiplication and addition/subtraction. 
したがって、ここで使用されるメソッド本体には、ループはまったく含まれていません.2つのタプルにインデックスを付け、乗算と加算/減算するだけです。
#All the looping is performed compile-time, and we avoid looping during execution entirely. 
すべてのループはコンパイル時に実行され、実行中にループすることはありません。
#Thus, we only loop *once per type*, in this case once per `N` (except in edge cases where the function is generated more than once - see disclaimer above).
したがって、タイプ*ごとに1回、この場合は「N」ごとに1回だけループします(ただし、関数が複数回生成されるエッジの場合を除き - 上記の免責条項を参照してください)。
