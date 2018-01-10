
<!---  Control Flow -->
# 制御フロー

<!--- Julia provides a variety of control flow constructs: -->
Juliaは、さまざまな制御フロー構成を提供しています。

  * [Compound Expressions](@ref man-compound-expressions): `begin` and `(;)`.
  * [Conditional Evaluation](@ref man-conditional-evaluation): `if`-`elseif`-`else` and `?:` (ternary operator).
  * [Short-Circuit Evaluation](@ref): `&&`, `||` and chained comparisons.
  * [Repeated Evaluation: Loops](@ref man-loops): `while` and `for`.
  * [Exception Handling](@ref): `try`-`catch`, [`error()`](@ref) and [`throw()`](@ref).
  * [Tasks (aka Coroutines)](@ref man-tasks): [`yieldto()`](@ref).

<!--- The first five control flow mechanisms are standard to high-level programming languages.  -->
最初の5つの制御フローメカニズムは、高水準プログラミング言語の標準です。
<!--- [`Task`](@ref) are not so standard: they provide non-local control flow, making it possible to switch between temporarily-suspended computations.  -->
[`Task`](@ref)はあまり標準的ではありません。ローカルではない制御フローを提供し、一時的に中断された計算を切り替えることができます。
<!--- This is a powerful construct: both exception handling and cooperative multitasking are implemented in Julia using tasks.  -->
これは強力な構成です：例外処理と協調マルチタスクの両方がタスクを使用してJuliaで実装されています。
<!--- Everyday programming requires no direct usage of tasks, but certain problems can be solved much more easily by using tasks. -->
毎日のプログラミングでは、タスクを直接使用する必要はありませんが、特定の問題はタスクを使用するとはるかに簡単に解決できます。

<!--- # [Compound Expressions](@id man-compound-expressions) -->
## [化合物式](@id man-compound-expressions)

<!--- Sometimes it is convenient to have a single expression which evaluates several subexpressions in order, returning the value of the last subexpression as its value.  -->
時々、いくつかの部分式を順番に評価し、最後の部分式の値をその値として返す単一の式を持つと便利です。
<!--- There are two Julia constructs that accomplish this: `begin` blocks and `(;)` chains.  -->
これを実現する2つのJulia構造体があります： `begin`ブロックと`(;) `鎖。
<!--- The value of both compound expression constructs is that of the last subexpression.  -->
両方の複合式構成の値は、最後の部分式の値です。
<!--- Here's an example of a `begin` block: -->
`begin`ブロックの例です：

```jldoctest
julia> z = begin
           x = 1
           y = 2
           x + y
       end
3
```

<!--- Since these are fairly small, simple expressions, they could easily be placed onto a single line, which is where the `(;)` chain syntax comes in handy: -->
これらはかなり小さく、簡単な式なので、 `(;)`チェーン構文が便利な単一の行に簡単に置くことができます：

```jldoctest
julia> z = (x = 1; y = 2; x + y)
3
```

<!--- This syntax is particularly useful with the terse single-line function definition form introduced in [Functions](@ref).  -->
この構文は、[Functions](@ref)で導入された簡潔な単一行の関数定義形式で特に便利です。
<!--- Although it is typical, there is no requirement that `begin` blocks be multiline or that `(;)` chains be single-line: -->
典型的ではあるが、 `begin`ブロックが複数行であるか、`(;) `チェーンが単一行である必要はありません：

```jldoctest
julia> begin x = 1; y = 2; x + y end
3

julia> (x = 1;
        y = 2;
        x + y)
3
```

<!--- ## [Conditional Evaluation](@id man-conditional-evaluation) -->
## [条件付き評価](@id man-conditional-evaluation)

<!--- Conditional evaluation allows portions of code to be evaluated or not evaluated depending on the value of a boolean expression.  -->
条件付き評価では、ブール式の値に応じてコードの一部を評価したり評価したりすることができません。
<!--- Here is the anatomy of the `if`-`elseif`-`else` conditional syntax: -->
`if`-`elseif`-` else`条件文の構造を以下に示します。

```julia
if x < y
    println("x is less than y")
elseif x > y
    println("x is greater than y")
else
    println("x is equal to y")
end
```

<!-- If the condition expression `x < y` is `true`, then the corresponding block is evaluated; otherwise the condition expression `x > y` is evaluated, and if it is `true`, the corresponding block is evaluated; if neither expression is true, the `else` block is evaluated. -->
条件式 `x < y`が` true`ならば、対応するブロックが評価されます。 それ以外の場合は条件式 `x> y`が評価され、` true`の場合は対応するブロックが評価されます。 どちらの式も真でない場合、 `else`ブロックが評価されます。
<!--- Here it is in action: -->
ここでそれは行動している：

```jldoctest
julia> function test(x, y)
           if x < y
               println("x is less than y")
           elseif x > y
               println("x is greater than y")
           else
               println("x is equal to y")
           end
       end
test (generic function with 1 method)

julia> test(1, 2)
x is less than y

julia> test(2, 1)
x is greater than y

julia> test(1, 1)
x is equal to y
```

<!--- The `elseif` and `else` blocks are optional, and as many `elseif` blocks as desired can be used. -->
`elseif`と` else`ブロックはオプションで、必要に応じて多くの `elseif`ブロックを使用できます。
<!--- The condition expressions in the `if`-`elseif`-`else` construct are evaluated until the first one evaluates to `true`, after which the associated block is evaluated, and no further condition expressions or blocks are evaluated. -->
`if`-`elseif`-`else`構造体の条件式は、最初のものが`true`と評価されるまで評価され、その後関連するブロックが評価され、それ以上の条件式やブロックは評価されません。

<!--- `if` blocks are "leaky", i.e. they do not introduce a local scope.  -->
`if 'ブロックは「漏れ」、すなわちローカルスコープを導入しない。
<!--- This means that new variables defined inside the `if` clauses can be used after the `if` block, even if they weren't defined before.  -->
つまり、 `if`節の中で定義された新しい変数は、前に定義されていなくても、` if`ブロックの後に使用できます。
<!--- So, we could have defined the `test` function above as -->
したがって、上記の `test`関数を以下のように定義することができました。

```jldoctest
julia> function test(x,y)
           if x < y
               relation = "less than"
           elseif x == y
               relation = "equal to"
           else
               relation = "greater than"
           end
           println("x is ", relation, " y.")
       end
test (generic function with 1 method)

julia> test(2, 1)
x is greater than y.
```

<!--- The variable `relation` is declared inside the `if` block, but used outside.  -->
変数 `relation`は` if`ブロックの中で宣言されていますが、外側で使われます。
<!--- However, when depending on this behavior, make sure all possible code paths define a value for the variable.  -->
ただし、この動作に応じて、可能なすべてのコードパスが変数の値を定義していることを確認してください。
<!--- The following change to the above function results in a runtime error -->
上記の関数を次のように変更すると、ランタイムエラーが発生します

```jldoctest
julia> function test(x,y)
           if x < y
               relation = "less than"
           elseif x == y
               relation = "equal to"
           end
           println("x is ", relation, " y.")
       end
test (generic function with 1 method)

julia> test(1,2)
x is less than y.

julia> test(2,1)
ERROR: UndefVarError: relation not defined
Stacktrace:
 [1] test(::Int64, ::Int64) at ./none:7
```

<!--- `if` blocks also return a value, which may seem unintuitive to users coming from many other languages. -->
`if`ブロックも値を返します。これは、他の多くの言語から来ているユーザには直感的ではないように思えるかもしれません。
<!--- This value is simply the return value of the last executed statement in the branch that was chosen, so -->
この値は、単に選択されたブランチ内の最後に実行されたステートメントの戻り値です。

```jldoctest
julia> x = 3
3

julia> if x > 0
           "positive!"
       else
           "negative..."
       end
"positive!"
```

<!--- Note that very short conditional statements (one-liners) are frequently expressed using Short-Circuit Evaluation in Julia, as outlined in the next section. -->
非常に短い条件文(1ライナー)は、次のセクションで説明するように、JuliaのShort-Circuit Evaluationを使用して頻繁に表現されることに注意してください。

<!--- Unlike C, MATLAB, Perl, Python, and Ruby -- but like Java, and a few other stricter, typed languages -->
C、MATLAB、Perl、Python、Rubyとは異なりますが、Javaやいくつかのより厳しい型付き言語
<!--- -- it is an error if the value of a conditional expression is anything but `true` or `false`: -->
条件式の値が `true`または` false`以外のものであればエラーとなります：

```jldoctest
julia> if 1
           println("true")
       end
ERROR: TypeError: non-boolean (Int64) used in boolean context
```

<!--- This error indicates that the conditional was of the wrong type: [`Int64`](@ref) rather than the required [`Bool`](@ref). -->
このエラーは、必要条件[Bool`](@ref)ではなく、条件が間違った型[`Int64`](@ref)であったことを示します。

#The so-called "ternary operator", `?:`, is closely related to the `if`-`elseif`-`else` syntax, but is used where a conditional choice between single expression values is required, as opposed to conditional execution of longer blocks of code. 
いわゆる "三項演算子"、 `?:`は、 `if`-`elseif`-`else`構文と密接に関係しますが、条件式とは対照的に、単一の式の値の間の条件付き選択が必要な場合に使用されます より長いコードブロックの実行。
<!--- It gets its name from being the only operator in most languages taking three operands: -->
それは3つのオペランドを取るほとんどの言語で唯一の演算子であることからその名前が得られます：

```julia
a ? b : c
```

#The expression `a`, before the `?`, is a condition expression, and the ternary operation evaluates the expression `b`, before the `:`, if the condition `a` is `true` or the expression `c`, after the `:`, if it is `false`. 
`？`の前にある式 `a`は条件式であり、` a`が `true`ならば`： `の前に` b`を評価し、 `c`の場合は` b`を評価します。 `：`の後に `` false 'ならば、
#Note that the spaces around `?` and `:` are mandatory: an expression like `a?b:c` is not a valid ternary expression (but a newline is acceptable after both the `?` and the `:`).
`a?b:c`のような式は有効な三項式ではありません(ただし、改行は`?`と`:`の後ろで使えます)。

<!-- The easiest way to understand this behavior is to see an example.  -->
この動作を理解する最も簡単な方法は、例を見ることです。
<!-- In the previous example, the `println` call is shared by all three branches: the only real choice is which literal string to print.  -->
前の例では、 `println`コールは、すべての3つの分岐によって共有されます:唯一の本当の選択は、印刷する文字列リテラルです。
<!-- This could be written more concisely using the ternary operator.  -->
これは三項演算子を使ってより簡潔に書くことができます。
<!-- For the sake of clarity, let's try a two-way version first: -->
わかりやすくするために、最初に双方向バージョンを試してみましょう：

```jldoctest
julia> x = 1; y = 2;

julia> println(x < y ? "less than" : "not less than")
less than

julia> x = 1; y = 0;

julia> println(x < y ? "less than" : "not less than")
not less than
```

#If the expression `x < y` is true, the entire ternary operator expression evaluates to the string `"less than"` and otherwise it evaluates to the string `"not less than"`. 
`x < y`の式が真の場合、3項演算子の式は文字列`より小さい`に評価され、それ以外の場合は文字列`未満でない`と評価されます。
<!-- The original three-way example requires chaining multiple uses of the ternary operator together: -->
オリジナルの3ウェイの例では、3項演算子を複数組み合わせて使用する必要があります。

```jldoctest
julia> test(x, y) = println(x < y ? "x is less than y"    :
                            x > y ? "x is greater than y" : "x is equal to y")
test (generic function with 1 method)

julia> test(1, 2)
x is less than y

julia> test(2, 1)
x is greater than y

julia> test(1, 1)
x is equal to y
```

#To facilitate chaining, the operator associates from right to left.
連鎖を容易にするために、演算子は右から左に関連付けます。

#It is significant that like `if`-`elseif`-`else`, the expressions before and after the `:` are only evaluated if the condition expression evaluates to `true` or `false`, respectively:
`if`-`elseif`-` else`のように、 `：`の前後の式は、条件式が `true`または` false`と評価された場合にのみ評価されます。

```jldoctest
julia> v(x) = (println(x); x)
v (generic function with 1 method)

julia> 1 < 2 ? v("yes") : v("no")
yes
"yes"

julia> 1 > 2 ? v("yes") : v("no")
no
"no"
```

## Short-Circuit Evaluation
## 短絡評価

#Short-circuit evaluation is quite similar to conditional evaluation. 
短絡評価は条件付き評価と非常によく似ています。
#The behavior is found in most imperative programming languages having the `&&` and `||` boolean operators: in a series of boolean expressions connected by these operators, only the minimum number of expressions are evaluated as are necessary to determine the final boolean value of the entire chain. 
この振る舞いは、 `&&`と `||`ブール演算子を持つほとんどの命令型プログラミング言語で見られます。 これらの演算子で連結された一連のブール式では、チェーン全体の最終的なブール値を決定するのに必要な最小数の式しか評価されません。
<!-- Explicitly, this means that: -->
明示的には、これは次のことを意味します。

   <!-- * In the expression `a && b`, the subexpression `b` is only evaluated if `a` evaluates to `true`. -->
   * `a && b 'という表現では、` a`が `true`と評価された場合にのみ、部分式` b`が評価されます。
   # * In the expression `a || b`, the subexpression `b` is only evaluated if `a` evaluates to `false`.
   * 式 `a || `サブ式`b`は、`a`が`false`と評価された場合にのみ評価される。

#The reasoning is that `a && b` must be `false` if `a` is `false`, regardless of the value of `b`, and likewise, the value of `a || b` must be true if `a` is `true`, regardless of the value of `b`. 
その理由は、 `a && b`は` b`の値にかかわらず `a`が` false`ならば `false`でなければならず、同様に` a || `b`の値にかかわらず` a`が `true`なら` b`は真でなければなりません。
<!-- Both `&&` and `||` associate to the right, but `&&` has higher precedence than `||` does.  -->
`&&`と `||`はどちらも右に関連しますが、 `&&`よりも優先順位が高くなります。
<!-- It's easy to experiment with this behavior: -->
この動作を実験するのは簡単です：

```jldoctest tandf
julia> t(x) = (println(x); true)
t (generic function with 1 method)

julia> f(x) = (println(x); false)
f (generic function with 1 method)

julia> t(1) && t(2)
1
2
true

julia> t(1) && f(2)
1
2
false

julia> f(1) && t(2)
1
false

julia> f(1) && f(2)
1
false

julia> t(1) || t(2)
1
true

julia> t(1) || f(2)
1
true

julia> f(1) || t(2)
1
2
true

julia> f(1) || f(2)
1
2
false
```

<!-- You can easily experiment in the same way with the associativity and precedence of various combinations of `&&` and `||` operators. -->
`&&`と `||`演算子のさまざまな組み合わせの連想性と優先順位を使って、同じ方法で簡単に実験することができます。

<!-- This behavior is frequently used in Julia to form an alternative to very short `if` statements. -->
非常に短い `if`文の代わりに、この動作がJuliaで頻繁に使用されます。
<!-- Instead of `if <cond> <statement> end`, one can write `<cond> && <statement>` (which could be read as: <cond> *and then* <statement>).  -->
`if <cond> <statement> end`の代わりに` <cond> && <statement> `と書くことができます。 (これは：<cond> *であれば* <statement>と読めます)。
<!-- Similarly, instead of `if ! <cond> <statement> end`, one can write `<cond> || <statement>` (which could be read as: <cond> *or else* <statement>). -->
同様に、`if! <cond> <statement> end`の代りに、 `<cond> || <statement> `と書けます。(これは<cond> *でなければ* <statement>と読むことができます)。

<!-- For example, a recursive factorial routine could be defined like this: -->
たとえば、再帰的階乗ルーチンは次のように定義できます。

```jldoctest
julia> function fact(n::Int)
           n >= 0 || error("n must be non-negative")
           n == 0 && return 1
           n * fact(n-1)
       end
fact (generic function with 1 method)

julia> fact(5)
120

julia> fact(0)
1

julia> fact(-1)
ERROR: n must be non-negative
Stacktrace:
 [1] fact(::Int64) at ./none:2
```

#Boolean operations *without* short-circuit evaluation can be done with the bitwise boolean operators introduced in [Mathematical Operations and Elementary Functions](@ref): `&` and `|`. 
*数学的演算と基本関数(@ref)： `＆`と `|`で導入されたビット単位のブール演算子で短絡評価なしのブール演算*を行うことができます。
<!-- These are normal functions, which happen to support infix operator syntax, but always evaluate their arguments: -->
これらは通常の関数であり、中置演算子の構文をサポートしますが、常に引数を評価します。

```jldoctest tandf
julia> f(1) & t(2)
1
2
false

julia> t(1) | t(2)
1
2
true
```

#Just like condition expressions used in `if`, `elseif` or the ternary operator, the operands of `&&` or `||` must be boolean values (`true` or `false`). 
`if`、`elseif`または三項演算子で使用される条件式と同様に、 `&&`または `||`のオペランドはブール値(`true`または`false`)でなければなりません。
<!-- Using a non-boolean value anywhere except for the last entry in a conditional chain is an error: -->
条件付きチェーンの最後のエントリを除くどこでも非ブール値を使用するとエラーになります。

```jldoctest
julia> 1 && true
ERROR: TypeError: non-boolean (Int64) used in boolean context
```

<!-- On the other hand, any type of expression can be used at the end of a conditional chain.  -->
一方、条件付きチェーンの終わりには、あらゆるタイプの式を使用できます。
<!-- It will be evaluated and returned depending on the preceding conditionals: -->
前の条件に応じて評価され、返されます。

```jldoctest
julia> true && (x = (1, 2, 3))
(1, 2, 3)

julia> false && (x = (1, 2, 3))
false
```

<!-- ## [Repeated Evaluation: Loops](@id man-loops) -->
## [反復評価：ループ](@ idのman-loops)

<!-- There are two constructs for repeated evaluation of expressions: the `while` loop and the `for` loop.  -->
式を繰り返し評価するには、`while`ループと`for`ループの2つの構文があります。
<!-- Here is an example of a `while` loop: -->
`while`ループの例を以下に示します：

```jldoctest
julia> i = 1;

julia> while i <= 5
           println(i)
           i += 1
       end
1
2
3
4
5
```

#The `while` loop evaluates the condition expression (`i <= 5` in this case), and as long it remains `true`, keeps also evaluating the body of the `while` loop. 
`while`ループは条件式(この場合は` i <= 5`)を評価し、それが `true`のままであれば` while`ループの本体も評価します。
#If the condition expression is `false` when the `while` loop is first reached, the body is never evaluated.
`while`ループに最初に到達したときに条件式が` false`の場合、ボディは評価されません。

#The `for` loop makes common repeated evaluation idioms easier to write. 
`for`ループは共通の繰り返し評価慣用句を書くのを容易にします。
#Since counting up and down like the above `while` loop does is so common, it can be expressed more concisely with a `for` loop:
上記の `while`ループのようにカウントダウンとカウントダウンが非常に一般的なので、` for`ループでより簡潔に表現することができます：

```jldoctest
julia> for i = 1:5
           println(i)
       end
1
2
3
4
5
```

<!-- Here the `1:5` is a `Range` object, representing the sequence of numbers 1, 2, 3, 4, 5.  -->
ここで、`1:5`は数字1,2,3,4,5のシーケンスを表す`Range`オブジェクトです。
<!-- The `for` loop iterates through these values, assigning each one in turn to the variable `i`.  -->
`for`ループはこれらの値を反復し、それぞれを変数`i`に割り当てます。
<!-- One rather important distinction between the previous `while` loop form and the `for` loop form is the scope during which the variable is visible.  -->
以前の `while`ループ形式と`for`ループ形式との間の重要な違いの1つは、変数が可視であるスコープです。
<!-- If the variable `i` has not been introduced in an other scope, in the `for` loop form, it is visible only inside of the `for` loop, and not afterwards. -->
変数 `i`が他のスコープに導入されていない場合、`for`ループ形式では、`for`ループの内部でのみ表示され、その後では表示されません。
<!-- You'll either need a new interactive session instance or a different variable name to test this: -->
これをテストするには、新しい対話型セッションインスタンスまたは別の変数名が必要です。

```jldoctest
julia> for j = 1:5
           println(j)
       end
1
2
3
4
5

julia> j
ERROR: UndefVarError: j not defined
```

#See [Scope of Variables](@ref scope-of-variables) for a detailed explanation of variable scope and how it works in Julia.
Variable Scopeの詳細と動作のJuliaでの詳細については、[Scope of Variables](@ref scope-of-variables)を参照してください。

<!-- In general, the `for` loop construct can iterate over any container.  -->
一般的に、 `for`ループ構造体はどのコンテナに対しても反復処理が可能です。
<!-- In these cases, the alternative (but fully equivalent) keyword `in` or `∈` is typically used instead of `=`, since it makes the code read more clearly: -->
このような場合、`=`の代わりに `in`や`ε`の代わりに(完全に等価な)キーワードが使われます。

```jldoctest
julia> for i in [1,4,0]
           println(i)
       end
1
4
0

julia> for s ∈ ["foo","bar","baz"]
           println(s)
       end
foo
bar
baz
```

<!-- Various types of iterable containers will be introduced and discussed in later sections of the manual (see, e.g., [Multi-dimensional Arrays](@ref man-multi-dim-arrays)). -->
さまざまなタイプの反復可能なコンテナについては、マニュアルの後のセクションで紹介します(例：[Mult-dimensional Arrays](@ref man-multi-dim-arrays)参照)。

#It is sometimes convenient to terminate the repetition of a `while` before the test condition is falsified or stop iterating in a `for` loop before the end of the iterable object is reached.
反復可能なオブジェクトの終わりに達する前に、テスト条件が改ざんされる前に `while`の繰り返しを終了するか、` for`ループで反復を停止するのが便利なことがあります。
<!-- This can be accomplished with the `break` keyword: -->
これは `break`キーワードで実現できます：

```jldoctest
julia> i = 1;

julia> while true
           println(i)
           if i >= 5
               break
           end
           i += 1
       end
1
2
3
4
5

julia> for i = 1:1000
           println(i)
           if i >= 5
               break
           end
       end
1
2
3
4
5
```

<!-- Without the `break` keyword, the above `while` loop would never terminate on its own, and the `for` loop would iterate up to 1000.  -->
`break`キーワードがなければ、上記の` while`ループは決してそれ自身で終了することはなく、 `for`ループは1000まで繰り返されます。
<!-- These loops are both exited early by using `break`. -->
これらのループは、どちらも早期に `break`を使用して終了します。

<!-- In other circumstances, it is handy to be able to stop an iteration and move on to the next one immediately.  -->
他の状況では、繰り返しを停止してすぐに次のものに進むことができれば便利です。
<!-- The `continue` keyword accomplishes this: -->
`continue`キーワードはこれを達成します：

```jldoctest
julia> for i = 1:10
           if i % 3 != 0
               continue
           end
           println(i)
       end
3
6
9
```

#This is a somewhat contrived example since we could produce the same behavior more clearly by negating the condition and placing the `println` call inside the `if` block. 
条件を否定し、 `if`ブロックの中に` println`呼び出しを置くことで、より明確に同じ振る舞いを作り出すことができるので、これはやや工夫した例です。
#In realistic usage there is more code to be evaluated after the `continue`, and often there are multiple points from which one calls `continue`.
現実的な使い方では、 `continue`の後に評価されるコードが多くあり、しばしば` continue`を呼び出す複数のポイントがあります。

<!-- Multiple nested `for` loops can be combined into a single outer loop, forming the cartesian product of its iterables:-->
複数のネストされた `for`ループを1つの外側ループに結合して、iterablesのデカルト積を形成することができます。

```jldoctest
julia> for i = 1:2, j = 3:4
           println((i, j))
       end
(1, 3)
(1, 4)
(2, 3)
(2, 4)
```

<!-- A `break` statement inside such a loop exits the entire nest of loops, not just the inner one.-->
そのようなループ内の `break`文は、内側のループだけではなく、ループのネスト全体を終了します。

<!-- # Exception Handling -->
## 例外処理

<!-- When an unexpected condition occurs, a function may be unable to return a reasonable value to its caller. -->
予期しない状態が発生した場合、関数は呼び出し側に妥当な値を返すことができない可能性があります。
#In such cases, it may be best for the exceptional condition to either terminate the program, printing a diagnostic error message, or if the programmer has provided code to handle such exceptional circumstances, allow that code to take the appropriate action.
そのような場合、例外条件ではプログラムを終了させるか、診断エラーメッセージを出力するか、またはプログラマが例外的な状況を処理するコードを提供している場合は、そのコードが適切な処置をとるようにしてください。

<!-- ### Built-in `Exception`s -->
### 組み込み`例外`

<!-- `Exception`s are thrown when an unexpected condition has occurred. -->
予期しない条件が発生したときに `Exception`がスローされます。
<!-- The built-in `Exception`s listed below all interrupt the normal flow of control.-->
以下に挙げるビルトインの「例外」はすべて、制御の通常の流れを中断します。

| `Exception`                   |
|:----------------------------- |
| [`ArgumentError`](@ref)       |
| [`BoundsError`](@ref)         |
| `CompositeException`          |
| [`DivideError`](@ref)         |
| [`DomainError`](@ref)         |
| [`EOFError`](@ref)            |
| [`ErrorException`](@ref)      |
| [`InexactError`](@ref)        |
| [`InitError`](@ref)           |
| [`InterruptException`](@ref)  |
| `InvalidStateException`       |
| [`KeyError`](@ref)            |
| [`LoadError`](@ref)           |
| [`OutOfMemoryError`](@ref)    |
| [`ReadOnlyMemoryError`](@ref) |
| [`RemoteException`](@ref)     |
| [`MethodError`](@ref)         |
| [`OverflowError`](@ref)       |
| [`ParseError`](@ref)          |
| [`SystemError`](@ref)         |
| [`TypeError`](@ref)           |
| [`UndefRefError`](@ref)       |
| [`UndefVarError`](@ref)       |
| `UnicodeError`                |

<!-- For example, the [`sqrt()`](@ref) function throws a [`DomainError`](@ref) if applied to a negative real value:-->
例えば、[`sqrt()`](@ref) 関数は、負の実数値に適用されると[DomainError`](@ref)をスローします：

```jldoctest
julia> sqrt(-1)
ERROR: DomainError:
sqrt will only return a complex result if called with a complex argument. 
Try sqrt(complex(x)).
Stacktrace:
 [1] sqrt(::Int64) at ./math.jl:447
```

<!-- You may define your own exceptions in the following way:-->
独自の例外を次のように定義することができます。

```jldoctest
julia> struct MyCustomException <: Exception end
```

<!-- ## The [`throw()`](@ref) function-->
### [`throw()`]](@ref)関数

<!-- Exceptions can be created explicitly with [`throw()`](@ref). -->
例外は[`throw()`](@ref)で明示的に生成できます。
<!-- For example, a function defined only for nonnegative numbers could be written to [`throw()`](@ref) a [`DomainError`](@ref) if the argument is negative:-->
例えば、負でない数値に対してのみ定義された関数は、引数が負の場合、[`throw()`](@ref)a [`DomainError`](@ref)

```jldoctest
julia> f(x) = x>=0 ? exp(-x) : throw(DomainError())
f (generic function with 1 method)

julia> f(1)
0.36787944117144233

julia> f(-1)
ERROR: DomainError:
Stacktrace:
 [1] f(::Int64) at ./none:1
```

<!-- Note that [`DomainError`](@ref) without parentheses is not an exception, but a type of exception.-->
括弧のない[`DomainError`](@ref)は例外ではなく、例外の一種であることに注意してください。
<!-- It needs to be called to obtain an `Exception` object:-->
`Exception`オブジェクトを取得するには、これを呼び出す必要があります：

```jldoctest
julia> typeof(DomainError()) <: Exception
true

julia> typeof(DomainError) <: Exception
false
```

<!-- Additionally, some exception types take one or more arguments that are used for error reporting:-->
さらに、一部の例外タイプでは、エラー報告に使用される1つ以上の引数が使用されます。

```jldoctest
julia> throw(UndefVarError(:x))
ERROR: UndefVarError: x not defined
```

<!-- This mechanism can be implemented easily by custom exception types following the way [`UndefVarError`](@ref) is written:-->
このメカニズムは、[`UndefVarError`](@ref)が書き込まれる方法に従ったカスタム例外の型によって簡単に実装できます：

```jldoctest
julia> struct MyUndefVarError <: Exception
           var::Symbol
       end

julia> Base.showerror(io::IO, e::MyUndefVarError) = print(io, e.var, " not defined")
```

!!! note
!!! 注意




   # When writing an error message, it is preferred to make the first word lowercase. 
     エラーメッセージを書くときは、最初の単語を小文字にすることをお勧めします。
   # For example,
     例えば、
   # `size(A) == size(B) || throw(DimensionMismatch("size of A not equal to size of B"))`
     `size(A)== size(B)|| throw(DimensionMismatch( "Bのサイズと等しくないサイズ")) `

   # is preferred over
     より優先する

   `size(A) == size(B) || throw(DimensionMismatch("Size of A not equal to size of B"))`.

   # However, sometimes it makes sense to keep the uppercase first letter, for instance if an argument to a function is a capital letter: `size(A,1) == size(B,2) || throw(DimensionMismatch("A has first dimension..."))`.
    しかし、関数の引数が大文字である場合など、大文字の最初の文字をそのまま使うのは時には意味があります： `size(A、1)== size(B、2)|| throw(DimensionMismatch( "Aは最初の次元を持つ...")) `。

### Errors
### エラー

#The [`error()`](@ref) function is used to produce an [`ErrorException`](@ref) that interrupts the normal flow of control.
[`error()`](@ref)関数は、通常の制御フローを中断する[`ErrorException`](@ref)を生成するために使用されます。

<!-- Suppose we want to stop execution immediately if the square root of a negative number is taken.-->
負の数の平方根が取られた場合、直ちに実行を停止したいとします。
<!-- To do this, we can define a fussy version of the [`sqrt()`](@ref) function that raises an error if its argument is negative:-->
これを行うには、引数が負の場合にエラーを発生させる[`sqrt()`](@ref)関数のぎっしりしたバージョンを定義することができます：

```jldoctest fussy_sqrt
julia> fussy_sqrt(x) = x >= 0 ? sqrt(x) : error("negative x not allowed")
fussy_sqrt (generic function with 1 method)

julia> fussy_sqrt(2)
1.4142135623730951

julia> fussy_sqrt(-1)
ERROR: negative x not allowed
Stacktrace:
 [1] fussy_sqrt(::Int64) at ./none:1
```

<!-- If `fussy_sqrt` is called with a negative value from another function, instead of trying to continue execution of the calling function, it returns immediately, displaying the error message in the interactive session:-->
`fussy_sqrt`が別の関数から負の値で呼び出された場合、呼び出し元の関数の実行を継続しようとするのではなく、直ちに戻り、エラーメッセージを対話セッションに表示します：

```jldoctest fussy_sqrt
julia> function verbose_fussy_sqrt(x)
           println("before fussy_sqrt")
           r = fussy_sqrt(x)
           println("after fussy_sqrt")
           return r
       end
verbose_fussy_sqrt (generic function with 1 method)

julia> verbose_fussy_sqrt(2)
before fussy_sqrt
after fussy_sqrt
1.4142135623730951

julia> verbose_fussy_sqrt(-1)
before fussy_sqrt
ERROR: negative x not allowed
Stacktrace:
 [1] fussy_sqrt at ./none:1 [inlined]
 [2] verbose_fussy_sqrt(::Int64) at ./none:3
```

<!-- ## Warnings and informational messages-->
## 警告と情報メッセージ

<!-- Julia also provides other functions that write messages to the standard error I/O, but do not throw any `Exception`s and hence do not interrupt execution:-->
Juliaは、標準エラーI/Oにメッセージを書き込む他の関数も提供していますが、 `Exception`をスローしないため、実行を中断しません。

```jldoctest
julia> info("Hi"); 1+1
INFO: Hi
2

julia> warn("Hi"); 1+1
WARNING: Hi
2

julia> error("Hi"); 1+1
ERROR: Hi
Stacktrace:
 [1] error(::String) at ./error.jl:21
```

### The `try/catch` statement
### `try/catch` 文

<!-- The `try/catch` statement allows for `Exception`s to be tested for. -->
`try / catch`文は、` Exception`がテストされることを可能にします。
<!-- For example, a customized square root function can be written to automatically call either the real or complex square root method on demand using `Exception`s :-->
例えば、カスタマイズされた平方根関数は、 `Exception`sを使ってオンデマンドで実数または複素数平方根法を自動的に呼び出すように書くことができます：

```jldoctest
julia> f(x) = try
           sqrt(x)
       catch
           sqrt(complex(x, 0))
       end
f (generic function with 1 method)

julia> f(1)
1.0

julia> f(-1)
0.0 + 1.0im
```

<!-- It is important to note that in real code computing this function, one would compare `x` to zero instead of catching an exception. -->
この関数を計算する実際のコードでは、例外をキャッチするのではなく、`x`をゼロと比較することに注意することが重要です。
<!-- The exception is much slower than simply comparing and branching.-->
例外は単純に比較して分岐するよりもずっと遅いです。

<!-- `try/catch` statements also allow the `Exception` to be saved in a variable. -->
`try/catch`ステートメントはまた、`Exception`が変数に保存されるようにします。
#In this contrived example, the following example calculates the square root of the second element of `x` if `x` is indexable, otherwise assumes `x` is a real number and returns its square root:
このような例では、次の例では `x`がインデックス可能な場合は` x`の2番目の要素の平方根を計算し、それ以外の場合は `x`を実数とし、平方根を返します。

```jldoctest
julia> sqrt_second(x) = try
           sqrt(x[2])
       catch y
           if isa(y, DomainError)
               sqrt(complex(x[2], 0))
           elseif isa(y, BoundsError)
               sqrt(x)
           end
       end
sqrt_second (generic function with 1 method)

julia> sqrt_second([1 4])
2.0

julia> sqrt_second([1 -4])
0.0 + 2.0im

julia> sqrt_second(9)
3.0

julia> sqrt_second(-9)
ERROR: DomainError:
Stacktrace:
 [1] sqrt_second(::Int64) at ./none:7
```

#Note that the symbol following `catch` will always be interpreted as a name for the exception, so care is needed when writing `try/catch` expressions on a single line. 
`catch`の後のシンボルは常に例外の名前として解釈されるので、` try/catch`式を1行に書くときは注意が必要です。
#The following code will *not* work to return the value of `x` in case of an error:
次のコードは、エラーが発生した場合に `x`の値を返すように*働かないでしょう：

```julia
try bad() catch x end
```

Instead, use a semicolon or insert a line break after `catch`:

```julia
try bad() catch; x end

try bad()
catch
    x
end
```

<!-- The `catch` clause is not strictly necessary; when omitted, the default return value is `nothing`.-->
`catch`節は厳密には必要ではありません。 省略された場合、デフォルトの戻り値は `nothing`です。

```jldoctest
julia> try error() end # Returns nothing
```

<!-- The power of the `try/catch` construct lies in the ability to unwind a deeply nested computation immediately to a much higher level in the stack of calling functions. -->
`try/catch`構造の力は、深く入れ子にされた計算を直ちに呼び出し関数のスタック内のより高いレベルに巻き戻す能力にあります。
#There are situations where no error has occurred, but the ability to unwind the stack and pass a value to a higher level is desirable. 
エラーが発生しない状況がありますが、スタックを巻き戻してより高いレベルに値を渡す機能が望ましいです。
#Julia provides the [`rethrow()`](@ref), [`backtrace()`](@ref) and [`catch_backtrace()`](@ref) functions for more advanced error handling.
Juliaは、より高度なエラー処理のために、[`rethrow()`](@ref)、[`backtrace()`](@ref)、[`catch_backtrace()`](@ref)関数を提供しています。

<!-- ## `finally` Clauses-->
## `finally`節

<!-- In code that performs state changes or uses resources like files, there is typically clean-up work (such as closing files) that needs to be done when the code is finished. -->
状態変更を実行するコードやファイルのようなリソースを使用するコードでは、コードが終了したときに行う必要のあるクリーンアップ作業(ファイルのクローズなど)が一般的です。
<!-- Exceptions potentially complicate this task, since they can cause a block of code to exit before reaching its normal end. -->
例外は、コードのブロックが正常終了する前に終了する可能性があるため、このタスクを複雑にする可能性があります。
<!-- The `finally` keyword provides a way to run some code when a given block of code exits, regardless of how it exits.-->
`finally`キーワードは、特定のコードブロックがどのように終了するかにかかわらず、そのコードブロックが終了したときにいくつかのコードを実行する方法を提供します。

<!-- For example, here is how we can guarantee that an opened file is closed:-->
たとえば、開いているファイルが閉じられることを保証する方法は次のとおりです。

```julia
f = open("file")
try
    # operate on file f
finally
    close(f)
end
```

#When control leaves the `try` block (for example due to a `return`, or just finishing normally), `close(f)` will be executed. 
制御が `try`ブロックを離れると(例えば` return`または正常終了)、 `close(f)`が実行されます。
#If the `try` block exits due to an exception, the exception will continue propagating. 
`try`ブロックが例外のために終了した場合、例外は伝播を続けます。
#A `catch` block may be combined with `try` and `finally` as well. 
`catch`ブロックは` try`と `finally`と組み合わせることもできます。
#In this case the `finally` block will run after `catch` has handled the error.
この場合、 `finally`ブロックは` catch`がエラーを処理した後に実行されます。

## [Tasks (aka Coroutines)](@id man-tasks)
## [タスク(別名コルーチン)](@ id man-tasks)

#Tasks are a control flow feature that allows computations to be suspended and resumed in a flexible manner. 
タスクは、柔軟に計算を中断して再開できるようにする制御フロー機能です。
#This feature is sometimes called by other names, such as symmetric coroutines, lightweight threads, cooperative multitasking, or one-shot continuations.
この機能は、対称コルーチン、軽量スレッド、協調マルチタスキング、ワンショット継続など、他の名前で呼ばれることがあります。

#When a piece of computing work (in practice, executing a particular function) is designated as a [`Task`](@ref), it becomes possible to interrupt it by switching to another [`Task`](@ref).
コンピューティングワーク(実際には特定の機能を実行する)が[`Task`](@ref)として指定されると、別の[Task`](@ref)に切り替えることで中断することが可能になります。
#The original [`Task`](@ref) can later be resumed, at which point it will pick up right where it left off. 
元の[`Task`](@ref)は後で再開することができます。その時点で、中断したところからすぐにピックアップします。
#At first, this may seem similar to a function call. 
最初は、これは関数呼び出しと同じように見えるかもしれません。
#However there are two key differences. 
しかし、2つの重要な違いがあります。
#First, switching tasks does not use any space, so any number of task switches can occur without consuming the call stack. 
まず、タスクの切り替えはスペースを使用しないため、コールスタックを消費せずに任意の数のタスクスイッチを実行できます。
#Second, switching among tasks can occur in any order, unlike function calls, where the called function must finish executing before control returns to the calling function.
第2に、関数呼び出しとは異なり、タスク間の切り替えは任意の順序で行うことができ、呼び出された関数は、制御が呼び出し関数に戻る前に実行を終了する必要があります。

<!-- This kind of control flow can make it much easier to solve certain problems. -->
この種の制御フローにより、特定の問題を簡単に解決できるようになります。
<!-- In some problems, the various pieces of required work are not naturally related by function calls; there is no obvious "caller" or "callee" among the jobs that need to be done. -->
いくつかの問題では、必要な作業のさまざまな部分は、関数呼び出しによって自然に関連するものではありません。実行する必要があるジョブの中に明白な「呼び出し元」または「呼び出し先」はありません。
<!-- An example is the producer-consumer problem, where one complex procedure is generating values and another complex procedure is consuming them. -->
例えば、ある複雑なプロシージャが値を生成し、別の複雑なプロシージャがそれらを消費している、プロデューサ - コンシューマの問題です。
<!-- The consumer cannot simply call a producer function to get a value, because the producer may have more values to generate and so might not yet be ready to return. -->
消費者は単にプロデューサ関数を呼び出して値を得ることはできません。なぜなら、プロデューサは生成する値が多く、返す準備がまだできていない可能性があるからです。
<!-- With tasks, the producer and consumer can both run as long as they need to, passing values back and forth as necessary.-->
タスクを使用すると、プロデューサとコンシューマは、必要に応じて前後に値を渡しながら、必要なだけ実行することができます。

<!-- Julia provides a [`Channel`](@ref) mechanism for solving this problem.-->
Juliaは、この問題を解決するための[`Channel`](@ref)メカニズムを提供しています。
<!-- A [`Channel`](@ref) is a waitable first-in first-out queue which can have multiple tasks reading from and writing to it.-->
[`Channel`](@ref)は待ち行列である先入れ先出し待ち行列であり、複数のタスクを読み書きすることができます。

<!-- Let's define a producer task, which produces values via the [`put!`](@ref) call.-->
プロデューサタスクを定義して、[`put！`](@ref)呼び出しによって値を生成しましょう。
<!-- To consume values, we need to schedule the producer to run in a new task. -->
値を使用するには、新しいタスクで実行するようにプロデューサをスケジュールする必要があります。
#A special [`Channel`](@ref) constructor which accepts a 1-arg function as an argument can be used to run a task bound to a channel.
1-arg関数を引数として受け入れる特殊な[`Channel`](@ref)コンストラクタを使用して、チャネルにバインドされたタスクを実行することができます。
#We can then [`take!()`](@ref) values repeatedly from the channel object:
次に、チャンネルオブジェクトから値を繰り返し取ることができます：

```jldoctest producer
julia> function producer(c::Channel)
           put!(c, "start")
           for n=1:4
               put!(c, 2n)
           end
           put!(c, "stop")
       end;

julia> chnl = Channel(producer);

julia> take!(chnl)
"start"

julia> take!(chnl)
2

julia> take!(chnl)
4

julia> take!(chnl)
6

julia> take!(chnl)
8

julia> take!(chnl)
"stop"
```

One way to think of this behavior is that `producer` was able to return multiple times. 
Between calls to [`put!()`](@ref), the producer's execution is suspended and the consumer has control.

The returned [`Channel`](@ref) can be used as an iterable object in a `for` loop, in which case the loop variable takes on all the produced values. 
The loop is terminated when the channel is closed.

```jldoctest producer
julia> for x in Channel(producer)
           println(x)
       end
start
2
4
6
8
stop
```

Note that we did not have to explicitly close the channel in the producer. 
This is because the act of binding a [`Channel`](@ref) to a [`Task()`](@ref) associates the open lifetime of a channel with that of the bound task. 
The channel object is closed automatically when the task terminates. 
Multiple channels can be bound to a task, and vice-versa.

While the [`Task()`](@ref) constructor expects a 0-argument function, the [`Channel()`](@ref) method which creates a channel bound task expects a function that accepts a single argument of type [`Channel`](@ref). 
A common pattern is for the producer to be parameterized, in which case a partial function application is needed to create a 0 or 1 argument [anonymous function](@ref man-anonymous-functions).

For [`Task()`](@ref) objects this can be done either directly or by use of a convenience macro:

```julia
function mytask(myarg)
    ...
end

taskHdl = Task(() -> mytask(7))
# or, equivalently
taskHdl = @task mytask(7)
```

#To orchestrate more advanced work distribution patterns, [`bind()`](@ref) and [`schedule()`](@ref) can be used in conjunction with [`Task()`](@ref) and [`Channel()`](@ref) constructors to explicitly link a set of channels with a set of producer/consumer tasks.
より高度な作業の分布パターンを編成するために,[`bind()`](@ref)と[`schedule()`](@ref)は、`_Task`と` _Channel() `コンストラクタと一緒に使用して、一連のチャンネルを一連のプロデューサ/コンシューマタスクに明示的にリンクすることができます。

<!-- Note that currently Julia tasks are not scheduled to run on separate CPU cores.-->
注意、現在ジュリアタスクは別々のCPUコアで実行するようにスケジュールされていません。
<!-- True kernel threads are discussed under the topic of [Parallel Computing](@ref).-->
真のカーネルスレッドについては、[Parallel Computing](@ref)のトピックで説明します。

<!-- ## Core task operations-->
## コアタスクの操作

<!-- Let us explore the low level construct [`yieldto()`](@ref) to underestand how task switching works.-->
低レベルの構文[`yieldto()`](@ref)を調べて、タスク切り替えの仕組みを調べてみましょう。
#`yieldto(task,value)` suspends the current task, switches to the specified `task`, and causes that task's last [`yieldto()`](@ref) call to return the specified `value`. 
`yieldto(task、value)`は現在のタスクを中断し、指定された `task`に切り替え、そのタスクの最後の[yieldto()`](@ref)呼び出しが指定された `value`を返すようにします。
#Notice that [`yieldto()`](@ref) is the only operation required to use task-style control flow; instead of calling and returning we are always just switching to a different task. 
タスクスタイルの制御フローを使用するには、[`yieldto()`](@ref)だけが必要であることに注意してください。コールして戻るのではなく、常に別のタスクに切り替えるだけです。
#This is why this feature is also called "symmetric coroutines"; each task is switched to and from using the same mechanism.
このため、この機能は「対称コルーチン」とも呼ばれます。各タスクは同じメカニズムを使用して切り替えられます。

#[`yieldto()`](@ref) is powerful, but most uses of tasks do not invoke it directly. 
[`yieldto()`](@ref)は強力ですが、タスクの大半は直接呼び出すことはありません。
#Consider why this might be. 
なぜこれがあるのか​​考えてみてください。
#If you switch away from the current task, you will probably want to switch back to it at some point, but knowing when to switch back, and knowing which task has the responsibility of switching back, can require considerable coordination. 
現在のタスクから離れた場合は、いつでも元に戻すことができますが、いつスイッチバックするかを知っていて、どのタスクがスイッチバックの責任を持っているかを知るにはかなりの調整が必要です。
#For example, [`put!()`](@ref) and [`take!()`](@ref) are blocking operations, which, when used in the context of channels maintain state to remember who the consumers are. 
たとえば、[`put！()`](@ref)と[`take！()`](@ref)は、チャネルのコンテキストで使用されるときに、消費者が誰であるかを記憶する状態を維持するブロック操作です。
#Not needing to manually keep track of the consuming task is what makes [`put!()`](@ref) easier to use than the low-level [`yieldto()`](@ref).
消費するタスクを手動で追跡する必要がないので、低レベルの[`yieldto()`](@ref)よりも[`put！()`](@ref)を使いやすくなります。

#In addition to [`yieldto()`](@ref), a few other basic functions are needed to use tasks effectively.
[`yieldto()`](@ref)に加えて、いくつかの基本的な関数がタスクを効果的に使うために必要です。

<!---
  * [`current_task()`](@ref) gets a reference to the currently-running task.
  * [`istaskdone()`](@ref) queries whether a task has exited.
  * [`istaskstarted()`](@ref) queries whether a task has run yet.
  * [`task_local_storage()`](@ref) manipulates a key-value store specific to the current task.
---->
   * [`current_task()`](@ref)は、現在実行中のタスクへの参照を取得します。
   * [`istaskdone()`](@ref)は、タスクが終了したかどうかを問い合わせます。
   * [`istaskstarted()`](@ref)は、タスクがまだ実行されているかどうかを問い合わせます。
   * [`task_local_storage()`](@ref)は、現在のタスクに固有のキー値ストアを操作します。

<!-- ## Tasks and events -->
## タスクとイベント

<!-- Most task switches occur as a result of waiting for events such as I/O requests, and are performed by a scheduler included in the standard library.  -->
ほとんどのタスクスイッチは、I/O要求などのイベントを待機した結果として発生し、標準ライブラリに含まれるスケジューラによって実行されます。
<!-- The scheduler maintains a queue of runnable tasks, and executes an event loop that restarts tasks based on external events such as message arrival. -->
スケジューラは、実行可能なタスクのキューを維持し、メッセージ到着などの外部イベントに基づいてタスクを再開するイベントループを実行します。

<!-- The basic function for waiting for an event is [`wait()`](@ref).  -->
イベントを待つための基本的な機能は[`wait()`](@ref)です。
<!-- Several objects implement [`wait()`](@ref); for example, given a `Process` object, [`wait()`](@ref) will wait for it to exit.  -->
いくつかのオブジェクトは[`wait()`](@ref)を実装しています。例えば、 `Process`オブジェクトが与えられた場合、[wait()`](@ref)は終了するのを待ちます。
<!-- [`wait()`](@ref) is often implicit; for example, a [`wait()`](@ref) can happen inside a call to [`read()`](@ref) to wait for data to be available. -->
[`wait()`](@ref) はしばしば暗黙的です。例えば、[`wait()`](@ref)は[`read()`](@ref)の呼び出しの中でデータが利用可能になるのを待つことができます。

#In all of these cases, [`wait()`](@ref) ultimately operates on a [`Condition`](@ref) object, which is in charge of queueing and restarting tasks. 
これらのすべての場合、[`wait()`](@ref) は最終的にタスクのキューイングと再起動を担当する[Condition`](@ref)オブジェクトで動作します。
#When a task calls [`wait()`](@ref) on a [`Condition`](@ref), the task is marked as non-runnable, added to the condition's queue, and switches to the scheduler.
タスクが[`Condition`](@ref)で[` wait() `](@ref)を呼び出すと、タスクは実行不可能としてマークされ、条件キューに追加され、スケジューラに切り替わります。
<!-- The scheduler will then pick another task to run, or block waiting for external events. -->
スケジューラは、実行する別のタスクを選択するか、外部イベントの待機をブロックします。
<!-- If all goes well, eventually an event handler will call [`notify()`](@ref) on the condition, which causes tasks waiting for that condition to become runnable again. -->
すべてがうまくいけば、最終的にはイベントハンドラがその条件に対して[`notify()`](@ref)を呼び出し、その条件を待っているタスクを再度実行可能にします。

<!-- A task created explicitly by calling [`Task`](@ref) is initially not known to the scheduler. -->
[`Task`](@ref)を呼び出して明示的に作成されたタスクは、最初はスケジューラには知られていません。
<!-- This allows you to manage tasks manually using [`yieldto()`](@ref) if you wish. -->
これにより、必要に応じて[`yieldto()`](@ref)を使って手動でタスクを管理することができます。
<!-- However, when such a task waits for an event, it still gets restarted automatically when the event happens, as you would expect. -->
しかし、そのようなタスクがイベントを待っているときは、イベントが発生したときに自動的に再起動されます。
<!-- It is also possible to make the scheduler run a task whenever it can, without necessarily waiting for any events. -->
また、必ずしもイベントを待つことなく、できる限りスケジューラにタスクを実行させることも可能です。
#This is done by calling [`schedule()`](@ref), or using the [`@schedule`](@ref) or [`@async`](@ref) macros (see [Parallel Computing](@ref) for more details).
これは、[`schedule()`](@ref)を呼び出すか、[`@ schedule`](@ref)または[` @ async`](@ref)マクロを使用して行われます([Parallel Computing] ref)を参照してください)。

<!-- ## Task states -->
## タスクの状態

<!-- Tasks have a `state` field that describes their execution status.  -->
タスクには、実行状態を記述する `state`フィールドがあります。
<!-- A [`Task`](@ref) `state` is one of the following symbols: -->
[`Task`](@ref)`state`は以下のシンボルの一つです：

<!--
| Symbol      | Meaning                                            |
|:----------- |:-------------------------------------------------- |
| `:runnable` | Currently running, or available to be switched to  |
| `:waiting`  | Blocked waiting for a specific event               |
| `:queued`   | In the scheduler's run queue about to be restarted |
| `:done`     | Successfully finished executing                    |
| `:failed`   | Finished with an uncaught exception                |
-->

| Symbol      | Meaning                                            |
|:----------- |:-------------------------------------------------- |
| `:runnable` |現在実行中、または切り替え可能                        | 
| `:waiting`  |特定のイベントを待ってブロックされました               |
| `:queued`   |スケジューラの実行キューが再起動されるタイミング       |
| `:done`     |実行が正常に終了しました                             |
| `:failed`   |未知の例外で終了しました                             |
