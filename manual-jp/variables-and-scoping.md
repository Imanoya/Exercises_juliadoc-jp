# [Scope of Variables](@id scope-of-variables)

<!-- The *scope* of a variable is the region of code within which a variable is visible. -->
変数の* scope *は、変数が表示されるコードの領域です。
<!-- Variable scoping helps avoid variable naming conflicts. -->
変数のスコープは、変数の名前の競合を避けるのに役立ちます。
<!-- The concept is intuitive: two functions can both have arguments called `x` without the two `x`'s referring to the same thing. -->
この概念は直感的です.2つの関数は、同じことを指す2つの `x`がなくても、` x`という引数を持つことができます。
<!-- Similarly there are many other cases where different blocks of code can use the same name without referring to the same thing. -->
同様に、コードの異なるブロックが同じものを参照せずに同じ名前を使用できる他の多くのケースがあります。
<!-- The rules for when the same variable name does or doesn't refer to the same thing are called scope rules; this section spells them out in detail.-->
同じ変数名が同じものを参照するときとそうでないときの規則は、スコープ規則と呼ばれます。 このセクションで詳しく説明します。

<!-- Certain constructs in the language introduce *scope blocks*, which are regions of code that are eligible to be the scope of some set of variables. -->
言語の中の特定の構文は*スコープブロック*を導入しています。これはいくつかの変数セットのスコープとなるコードの領域です。
<!-- The scope of a variable cannot be an arbitrary set of source lines; instead, it will always line up with one of these blocks. -->
変数のスコープは、ソース行の任意の集合にすることはできません。 代わりに、これらのブロックのいずれかと常に整列します。
<!-- There are two main types of scopes in Julia, *global scope* and *local scope*, the latter can be nested. -->
Juliaにはグローバルスコープ*と*ローカルスコープの2つの主なスコープがあり、スコープはネストすることができます。
<!-- The constructs introducing scope blocks are:-->
スコープブロックを導入するコンストラクトは次のとおりです。


| Scope name           | block/construct introducing this kind of scope                                  |
|:-------------------------------------------------------------------------------------------------------- |
| [Global Scope](@ref) | `module`, `baremodule`, 
                         at interactive prompt (REPL)                                                     |
|:-------------------------------------------------------------------------------------------------------- |
| [Local Scope](@ref)  | [Soft Local Scope](@ref): `for`, 
                        `while`, comprehensions, try-catch-finally, `let`                       |
|:-------------------------------------------------------------------------------------------------------- |
| [Local Scope](@ref)  | [Hard Local Scope](@ref): 
                        functions (either syntax, anonymous & do-blocks), `struct`, `macro`            |
|:-------------------------------------------------------------------------------------------------------- |

<!-- Notably missing from this table are [begin blocks](@ref man-compound-expressions) and [if blocks](@ref man-conditional-evaluation), which do *not* introduce new scope blocks. -->
特にこのテーブルには、新しいスコープブロックを導入しない* [*]ブロック（@ ref man-compound-expressions）と[ifブロック（@ ref man-conditional-evaluation）]がありません。
<!-- All three types of scopes follow somewhat different rules which will be explained below as well as some extra rules for certain blocks.-->
スコープの3つのタイプはすべて、以下に説明されるように、いくつかの異なるルールに従います。

<!-- Julia uses [lexical scoping](https://en.wikipedia.org/wiki/Scope_%28computer_science%29#Lexical_scoping_vs._dynamic_scoping), meaning that a function's scope does not inherit from its caller's scope, but from the scope in which the function was defined. -->
Juliaは、[レキシカルスコープ]（https://en.wikipedia.org/wiki/Scope_%28computer_science%29#Lexical_scoping_vs._dynamic_scoping）を使用します。つまり、関数のスコープは呼び出し元のスコープから継承されませんが、 関数が定義されました。
<!-- For example, in the following code the `x` inside `foo` refers to the `x` in the global scope of its module `Bar`:-->
例えば、次のコードでは `foo`の` x`はモジュール `Bar`のグローバルスコープの` x`を参照しています：

```jldoctest moduleBar
julia> module Bar
           x = 1
           foo() = x
       end;
```

<!-- and not a `x` in the scope where `foo` is used:-->
`foo`が使用されているスコープの` x`ではなく：

```jldoctest moduleBar
julia> import .Bar

julia> x = -1;

julia> Bar.foo()
1
```

<!-- Thus *lexical scope* means that the scope of variables can be inferred from the source code alone.-->
したがって、*レキシカルスコープ*は、変数のスコープがソースコードだけから推測できることを意味します。

<!--- # Global Scope-->
##グローバルスコープ

<!-- *Each module introduces a new global scope*, separate from the global scope of all other modules; there is no all-encompassing global scope. -->
*各モジュールは、他のすべてのモジュールのグローバルスコープとは別の新しいグローバルスコープ*を導入しています。 すべての包括的なスコープは存在しません。
<!-- Modules can introduce variables of other modules into their scope through the [using or import](@ref modules) statements or through qualified access using the dot-notation, i.e. each module is a so-called *namespace*. -->
モジュールは、[usingまたはimport]（@ refモジュール）ステートメントを通して、またはドット表記法を用いた修飾されたアクセスを通じて、他のモジュールの変数をスコープに導入できます。つまり、各モジュールはいわゆる* namespace *です。
<!-- Note that variable bindings can only be changed within their global scope and not from an outside module.-->
可変バインディングは、グローバルスコープ内でのみ変更でき、外部モジュールでは変更できないことに注意してください。

```jldoctest
julia> module A
           a = 1 # a global in A's scope
       end;

julia> module B
           module C
               c = 2
           end
           b = C.c    # can access the namespace of a nested global scope
                      # through a qualified access
           import ..A # makes module A available
           d = A.a
       end;

julia> module D
           b = a # errors as D's global scope is separate from A's
       end;
ERROR: UndefVarError: a not defined

julia> module E
           import ..A # make module A available
           A.a = 2    # throws below error
       end;
ERROR: cannot assign variables in other modules
```

<!-- Note that the interactive prompt (aka REPL) is in the global scope of the module `Main`.-->
インタラクティブプロンプト（別名REPL）は、モジュール `Main`のグローバルスコープにあります。

<!--- # Local Scope-->
##ローカルスコープ

<!-- A new local scope is introduced by most code-blocks, see above table for a complete list.-->
新しいローカルスコープは、ほとんどのコードブロックによって導入されています。
<!-- A local scope *usually* inherits all the variables from its parent scope, both for reading and writing. -->
ローカルスコープ*は、親スコープのすべての変数を読み書きの両方で継承します。
<!-- There are two subtypes of local scopes, hard and soft, with slightly different rules concerning what variables are inherited. -->
ローカルスコープにはハードとソフトの2つのサブタイプがあり、どの変数が継承されているかについてはわずかに異なるルールがあります。
<!-- Unlike global scopes, local scopes are not namespaces, thus variables in an inner scope cannot be retrieved from the parent scope through some sort of qualified access.-->
グローバルスコープとは異なり、ローカルスコープは名前空間ではないため、内部スコープ内の変数は、ある種の修飾されたアクセスによって親スコープから取り出すことはできません。

<!-- The following rules and examples pertain to both hard and soft local scopes. -->
次の規則と例は、ハードローカルスコープとソフトローカルスコープの両方に関係します。
<!-- A newly introduced variable in a local scope does not back-propagate to its parent scope. -->
ローカルスコープ内に新しく導入された変数は、その親スコープにバックプロパゲーションされません。
<!-- For example, here the `z` is not introduced into the top-level scope:-->
例えば、ここで `z`はトップレベルスコープには導入されていません：

```jldoctest
julia> for i = 1:10
           z = i
       end

julia> z
ERROR: UndefVarError: z not defined
```

<!-- (Note, in this and all following examples it is assumed that their top-level is a global scope with a clean workspace, for instance a newly started REPL.)-->
（この例と以下のすべての例では、トップレベルが、新規に起動されたREPLなどのクリーンな作業領域を持つグローバルスコープであると仮定しています）。

<!-- Inside a local scope a variable can be forced to be a local variable using the `local` keyword:-->
ローカルスコープの内部では、 `local`キーワードを使用して変数を強制的にローカル変数にすることができます：

```jldoctest
julia> x = 0;

julia> for i = 1:10
           local x
           x = i + 1
       end

julia> x
0
```

<!-- Inside a local scope a new global variable can be defined using the keyword `global`:-->
ローカルスコープの中で、新しいグローバル変数はキーワード `global`を使って定義することができます：

```jldoctest
julia> for i = 1:10
           global z
           z = i
       end

julia> z
10
```

<!-- The location of both the `local` and `global` keywords within the scope block is irrelevant.-->
スコープブロック内の `local`と` global`キーワードの両方の位置は無関係です。
<!-- The following is equivalent to the last example (although stylistically worse):-->
以下は最後の例と同じです（形式的には悪化しますが）。

```jldoctest
julia> for i = 1:10
           z = i
           global z
       end

julia> z
10
```

<!-- The `local` and `global` keywords can also be applied to destructuring assignments, e.g. `local x, y = 1, 2`. -->
`local`と` global`キーワードは、構造化代入にも適用できます。 `local x、y = 1,2 'である。
<!-- In this case the keyword affects all listed variables.-->
この場合、キーワードはすべてのリストされた変数に影響します。

<!--- ## Soft Local Scope-->
###ソフトローカルスコープ

<!-- >> In a soft local scope, all variables are inherited from its parent scope unless a variable is-->
>ソフトローカルスコープでは、すべての変数が親スコープから継承されます。
<!-- >> specifically marked with the keyword `local`.-->
>キーワード `local`で特にマークされています。

<!-- Soft local scopes are introduced by for-loops, while-loops, comprehensions, try-catch-finally-blocks, and let-blocks. -->
ソフトループスコープは、for-loops、while-loops、comprehensions、try-catch-finally-block、およびlet-blocksによって導入されています。
<!-- There are some extra rules for [Let Blocks](@ref) and for [For Loops and Comprehensions](@ref).-->
[Let Blocks]（@ ref）と[For Loops and Comprehensions]（@ ref）のいくつかの特別な規則があります。

<!-- In the following example the `x` and `y` refer always to the same variables as the soft local scope inherits both read and write variables:-->
次の例では、 `x`と` y`は常に同じ変数を参照し、ソフトローカルスコープは読み書き変数を継承します。

```jldoctest
julia> x, y = 0, 1;

julia> for i = 1:10
           x = i + y + 1
       end

julia> x
12
```

<!-- Within soft scopes, the *global* keyword is never necessary, although allowed. -->
ソフトスコープ内では、* global *キーワードは必須ではありませんが、許可されています。
<!-- The only case when it would change the semantics is (currently) a syntax error:-->
セマンティクスを変更する唯一のケースは、（現在）構文エラーです。

```jldoctest
julia> let
           local j = 2
           let
               global j = 3
           end
       end
ERROR: syntax: `global j`: j is local variable in the enclosing scope
```

<!--- ## Hard Local Scope-->
###ハードローカルスコープ

<!-- Hard local scopes are introduced by function definitions (in all their forms), struct type definition blocks, and macro-definitions.-->
ハードローカルスコープは、（すべての形式で）関数定義、構造体タイプ定義ブロック、およびマクロ定義によって導入されます。

<!-- 
> In a hard local scope, all variables are inherited from its parent scope unless:
>ハードローカルスコープでは、次の場合を除き、すべての変数は親スコープから継承されます。
>
>   * an assignment would result in a modified *global* variable, or
>   * a variable is specifically marked with the keyword `local`.
> *代入によって変更された*グローバル変数*
> *変数には特に `local`というキーワードが付けられています。
-->

<!-- Thus global variables are only inherited for reading but not for writing:-->
したがって、グローバル変数は、読み込みのためだけ継承され、書き込みのためには継承されません。

```jldoctest
julia> x, y = 1, 2;

julia> function foo()
           x = 2        # assignment introduces a new local
           return x + y # y refers to the global
       end;

julia> foo()
4

julia> x
1
```

<!-- An explicit `global` is needed to assign to a global variable:-->
グローバル変数に代入するには明示的な `global`が必要です：

```jldoctest
julia> x = 1;

julia> function foobar()
           global x = 2
       end;

julia> foobar();

julia> x
2
```

<!-- Note that *nested functions* can behave differently to functions defined in the global scope as they can modify their parent scope's *local* variables:-->
*ネストされた関数*は親スコープの* local *変数を変更できるので、グローバルスコープで定義された関数とは動作が異なることに注意してください。

```jldoctest
julia> x, y = 1, 2;

julia> function baz()
           x = 2 # introduces a new local
           function bar()
               x = 10       # modifies the parent's x
               return x + y # y is global
           end
           return bar() + x # 12 + 10 (x is modified in call of bar())
       end;

julia> baz()
22

julia> x, y
(1, 2)
```

<!-- The distinction between inheriting global and local variables for assignment can lead to some slight differences between functions defined in local vs. global scopes. -->
代入のグローバル変数とローカル変数を区別することによって、ローカルスコープとグローバルスコープで定義された関数間に若干の違いが生じる可能性があります。
<!-- Consider the modification of the last example by moving `bar` to the global scope:-->
`bar`をグローバルスコープに移動することで、最後の例の変更を考えてみましょう：

```jldoctest
julia> x, y = 1, 2;

julia> function bar()
           x = 10 # local
           return x + y
       end;

julia> function quz()
           x = 2 # local
           return bar() + x # 12 + 2 (x is not modified)
       end;

julia> quz()
14

julia> x, y
(1, 2)
```

<!-- Note that above subtlety does not pertain to type and macro definitions as they can only appear at the global scope. -->
上記の微妙さは型とマクロの定義には関係しません。なぜなら型はグローバルスコープにしか現れないからです。
<!-- There are special scoping rules concerning the evaluation of default and keyword function arguments which are described in the [Function section](@ref man-functions).-->
[Function section]（@ ref man-functions）に記述されている、デフォルトおよびキーワード関数の引数の評価に関する特別なスコープ規則があります。

<!-- An assignment introducing a variable used inside a function, type or macro definition need not come before its inner usage:-->
関数、型、またはマクロ定義内で使用される変数を導入する代入は、その内部使用の前に来る必要はありません。

```jldoctest
julia> f = y -> y + a
(::#1) (generic function with 1 method)

julia> f(3)
ERROR: UndefVarError: a not defined
Stacktrace:
 [1] (::##1#2)(::Int64) at ./none:1

julia> a = 1
1

julia> f(3)
4
```

<!-- This behavior may seem slightly odd for a normal variable, but allows for named functions -- which are just normal variables holding function objects -- to be used before they are defined. -->
この動作は、通常の変数では少し奇妙に思えるかもしれませんが、関数オブジェクトを保持する通常の変数である名前付き関数を定義する前に使用することができます。
<!-- This allows functions to be defined in whatever order is intuitive and convenient, rather than forcing bottom up ordering or requiring forward declarations, as long as they are defined by the time they are actually called. -->
これにより、実際に呼び出された時点で定義されている限り、ボトムアップの順序付けや前方宣言が必要なく、直感的かつ便利な順序で関数を定義できます。
<!-- As an example, here is an inefficient, mutually recursive way to test if positive integers are even or odd:-->
たとえば、正の整数が偶数か奇数かをテストする非効率的な、相互再帰的な方法を次に示します。

```jldoctest
julia> even(n) = n == 0 ? true : odd(n-1);

julia> odd(n) = n == 0 ? false : even(n-1);

julia> even(3)
false

julia> odd(3)
true
```

<!-- Julia provides built-in, efficient functions to test for oddness and evenness called [`iseven()`](@ref) and [`isodd()`](@ref) so the above definitions should only be taken as examples.-->
Juliaは、[`iseven（）`]（@ ref）と[`isodd（）`]（@ ref）と呼ばれる奇妙さと均等性をテストするための組み込みの効率的な関数を提供しているので、上記の定義は例としてとるべきです。

<!--- ## Hard vs. Soft Local Scope-->
###ハードスコープとソフトスコープ

<!-- Blocks which introduce a soft local scope, such as loops, are generally used to manipulate the variables in their parent scope. -->
ループなどのソフトローカルスコープを導入するブロックは、通常、親スコープ内の変数を操作するために使用されます。
<!-- Thus their default is to fully access all variables in their parent scope.-->
したがって、デフォルトでは、親スコープのすべての変数に完全にアクセスします。

<!-- Conversely, the code inside blocks which introduce a hard local scope (function, type, and macro definitions) can be executed at any place in a program. -->
逆に、ハードローカルスコープ（関数、型、およびマクロ定義）を導入するブロック内のコードは、プログラムの任意の場所で実行できます。
<!-- Remotely changing the state of global variables in other modules should be done with care and thus this is an opt-in feature requiring the `global` keyword.-->
他のモジュールでグローバル変数の状態を遠隔で変更する場合は注意が必要です。したがって、これは `global`キーワードを必要とするオプトイン機能です。

<!-- The reason to allow *modifying local* variables of parent scopes in nested functions is to allow constructing [closures](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29) which have a private state, for instance the `state` variable in the following example:-->
ネストされた関数で親スコープのローカル*変数を変更できるようにする理由は、[closures]（https://en.wikipedia.org/wiki/Closure_%28computer_programming%29）を構築することができるようにするためです。 次の例では `state`変数を使用します：

```jldoctest
julia> let
           state = 0
           global counter
           counter() = state += 1
       end;

julia> counter()
1

julia> counter()
2
```

<!-- See also the closures in the examples in the next two sections.-->
次の2つのセクションの例のクロージャも参照してください。

<!--- ## Let Blocks-->
### Let Blocks

<!-- Unlike assignments to local variables, `let` statements allocate new variable bindings each time they run. -->
ローカル変数への代入とは異なり、 `let`ステートメントは実行するたびに新しい変数バインディングを割り当てます。
<!-- An assignment modifies an existing value location, and `let` creates new locations.-->
割り当ては既存の値の場所を変更し、 `let`は新しい場所を作成します。
<!-- This difference is usually not important, and is only detectable in the case of variables that outlive their scope via closures. -->
この差は通常は重要ではなく、クロージャによって範囲を超えている変数の場合にのみ検出可能です。
<!-- The `let` syntax accepts a comma-separated series of assignments and variable names:-->
`let`構文はカンマで区切られた一連の代入と変数名を受け取ります：

```jldoctest
julia> x, y, z = -1, -1, -1;

julia> let x = 1, z
           println("x: $x, y: $y") # x is local variable, y the global
           println("z: $z") # errors as z has not been assigned yet but is local
       end
x: 1, y: -1
ERROR: UndefVarError: z not defined
```

<!-- The assignments are evaluated in order, with each right-hand side evaluated in the scope before the new variable on the left-hand side has been introduced. -->
割り当ては順番に評価され、左側の新しい変数が導入される前に、各右側がスコープ内で評価されます。
<!-- Therefore it makes sense to write something like `let x = x` since the two `x` variables are distinct and have separate storage.-->
したがって、 `let x = x`のようなものを書くことは理にかなっています。なぜなら、2つの` x`変数は区別され、別々の記憶域を持っているからです。
<!-- Here is an example where the behavior of `let` is needed:-->
`let`の動作が必要な場合の例を以下に示します：

```jldoctest
julia> Fs = Array{Any}(2); i = 1;

julia> while i <= 2
           Fs[i] = ()->i
           i += 1
       end

julia> Fs[1]()
3

julia> Fs[2]()
3
```

<!-- Here we create and store two closures that return variable `i`. -->
ここでは、変数 `i`を返す2つのクロージャを作成して格納します。
<!-- However, it is always the same variable `i`, so the two closures behave identically. -->
しかし、それは常に同じ変数 `i`であるため、2つのクロージャは同じように動作します。
<!-- We can use `let` to create a new binding for `i`:-->
`let`を使って` i`の新しい束縛を作ることができます：

```jldoctest
julia> Fs = Array{Any}(2); i = 1;

julia> while i <= 2
           let i = i
               Fs[i] = ()->i
           end
           i += 1
       end

julia> Fs[1]()
1

julia> Fs[2]()
2
```

<!-- Since the `begin` construct does not introduce a new scope, it can be useful to use a zero-argument `let` to just introduce a new scope block without creating any new bindings:-->
`begin`構造体は新しいスコープを導入しないので、新しいバインディングを作成せずに新しいスコープブロックを導入するために引数のない` let`を使うと便利です：

```jldoctest
julia> let
           local x = 1
           let
               local x = 2
           end
           x
       end
1
```

<!-- Since `let` introduces a new scope block, the inner local `x` is a different variable than the outer local `x`.-->
`let`は新しいスコープブロックを導入するので、内部ローカル` x`は外部ローカル `x`とは異なる変数です。

<!--- ## For Loops and Comprehensions-->
###ループと理解のために

<!-- `for` loops and [Comprehensions](@ref) have the following behavior: any new variables introduced in their body scopes are freshly allocated for each loop iteration. -->
`for 'ループと[Comprehensions]（@ ref）は次のような動作をします。ボディスコープに導入された新しい変数は、ループの繰り返しごとに新たに割り当てられます。
<!-- This is in contrast to `while` loops which reuse the variables for all iterations. -->
これは、すべての反復で変数を再利用する `while`ループとは対照的です。
<!-- Therefore these constructs are similar to `while` loops with `let` blocks inside:-->
したがって、これらの構造体は、 `let`ブロックが内部にある` while`ループと似ています：

```jldoctest
julia> Fs = Array{Any}(2);

julia> for j = 1:2
           Fs[j] = ()->j
       end

julia> Fs[1]()
1

julia> Fs[2]()
2
```

<!-- `for` loops will reuse existing variables for its iteration variable:-->
`for`ループは反復変数に既存の変数を再利用します：

```jldoctest
julia> i = 0;

julia> for i = 1:3
       end

julia> i
3
```

<!-- However, comprehensions do not do this, and always freshly allocate their iteration variables:-->
しかし、理解はこれをおこなわず、いつも新しく反復変数を割り当てます。

```jldoctest
julia> x = 0;

julia> [ x for x = 1:3 ];

julia> x
0
```

<!--- # Constants-->
## 定数

<!-- A common use of variables is giving names to specific, unchanging values.-->
変数の一般的な使用は、特定の変更されない値に名前を付けることです。
<!-- Such variables are only assigned once.-->
そのような変数は一度だけ割り当てられます。
<!-- This intent can be conveyed to the compiler using the `const` keyword:-->
この意図は、 `const`キーワードを使用してコンパイラに伝えることができます：

```jldoctest
julia> const e  = 2.71828182845904523536;

julia> const pi = 3.14159265358979323846;
```

<!-- The `const` declaration is allowed on both global and local variables, but is especially useful for globals. -->
`const`宣言は、グローバル変数とローカル変数の両方で許可されますが、特にグローバルに便利です。
<!-- It is difficult for the compiler to optimize code involving global variables, since their values (or even their types) might change at almost any time. -->
コンパイラは、グローバル変数を含むコードを最適化することは困難です。その値（または型も）はほとんどいつでも変更される可能性があるからです。
<!-- If a global variable will not change, adding a `const` declaration solves this performance problem. -->
グローバル変数が変更されない場合、 `const`宣言を追加することでこのパフォーマンスの問題が解決されます。

<!-- Local constants are quite different. -->
ローカル定数は全く異なります。
<!-- The compiler is able to determine automatically when a local variable is constant, so local constant declarations are not necessary for performance purposes. -->
コンパイラは、ローカル変数が定数のときに自動的に判別できるため、パフォーマンス上の目的でローカル定数宣言は必要ありません。

<!-- Special top-level assignments, such as those performed by the `function` and `struct` keywords, are constant by default. -->
`function`や` struct`キーワードによって実行されるような特別なトップレベルの割り当ては、デフォルトでは定数です。

<!-- Note that `const` only affects the variable binding; the variable may be bound to a mutable object (such as an array), and that object may still be modified. -->
`const`は変数バインディングにのみ影響することに注意してください。 変数は変更可能なオブジェクト（配列など）にバインドされていても、そのオブジェクトは変更されている可能性があります。
