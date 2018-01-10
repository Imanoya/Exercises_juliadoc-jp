
<!-- # Documentation -->
＃ ドキュメンテーション

<!-- Julia enables package developers and users to document functions, types and other objects easily via a built-in documentation system since Julia 0.4. -->
ジュリアは、パッケージ開発者やユーザーがJulia 0.4以降の組み込みドキュメントシステムを使用して、関数、型およびその他のオブジェクトを簡単に文書化することを可能にします。

#The basic syntax is very simple: any string appearing at the top-level right before an object (function, macro, type or instance) will be interpreted as documenting it (these are called *docstrings*).
基本的な構文は非常に単純です。オブジェクト（関数、マクロ、タイプまたはインスタンス）がドキュメント化する前に最上位に現れる文字列（* docstrings *と呼ばれます）
#Here is a very simple example:
ここには非常に簡単な例があります：

```julia
"Tell whether there are too foo items in the array."
foo(xs::Array) = ...
```

#Documentation is interpreted as [Markdown](https://en.wikipedia.org/wiki/Markdown), so you can use indentation and code fences to delimit code examples from text. 
ドキュメントは[Markdown]（https://en.wikipedia.org/wiki/Markdown）として解釈されるため、インデントとコードフェンスを使用してテキストのコード例を区切ります。
#Technically, any object can be associated with any other as metadata; Markdown happens to be the default, but one can construct other string macros and pass them to the `@doc` macro just as well.
技術的には、どのオブジェクトも他のメタデータと関連付けることができます。 Markdownはデフォルトで行われますが、他の文字列マクロを作成して `@ doc`マクロに渡すこともできます。

<!-- Here is a more complex example, still using Markdown: -->
Markdownを使用して、より複雑な例を次に示します。

````julia
"""
    bar(x[, y])

<!-- Compute the Bar index between `x` and `y`. If `y` is missing, compute the Bar index between all pairs of columns of `x`. -->
`x`と` y`の間でBarインデックスを計算します。 `y`が指定されていない場合は、` x`のすべての列のペア間でBarインデックスを計算します。

# Examples
```julia-repl
julia> bar([1, 2], [1, 2])
1
```
"""
function bar(x, y) ...
````

<!-- As in the example above, we recommend following some simple conventions when writing documentation: -->
上記の例のように、ドキュメントを書くときの簡単な慣習に従うことをお勧めします：

1. 
   # Always show the signature of a function at the top of the documentation, with a four-space indent so that it is printed as Julia code.
    文書の上部には、関数の署名を常に表示し、4桁のインデントを付けてJuliaコードとして出力します。

   # This can be identical to the signature present in the Julia code (like `mean(x::AbstractArray)`), or a simplified form. 
    これはジュリアコード（ `mean（x :: AbstractArray）`のような）、または簡略化された形式のシグネチャと同じです。
   # Optional arguments should be represented with their default values (i.e. `f(x, y=1)`) when possible, following the actual Julia syntax. 
    可能な場合は、実際のJulia構文に従い、任意の引数をデフォルト値（つまり `f（x、y = 1）`）で表す必要があります。
   # Optional arguments which do not have a default value should be put in brackets (i.e. `f(x[, y])` and `f(x[, y[, z]])`). 
    デフォルトの値を持たない任意の引数は角括弧（ `` f [x [、y]） `と` f（x [、y [、z]]）で囲む必要があります。
   # An alternative solution is to use several lines: one without optional arguments, the other(s) with them. 
     別の解決方法は、いくつかの行を使用することです：オプション引数を持たない行と、それ以外の行を使用する行です。
   # This solution can also be used to document several related methods of a given function. 
    このソリューションは、特定の関数のいくつかの関連するメソッドを文書化するためにも使用できます。
   # When a function accepts many keyword arguments, only include a `<keyword arguments>` placeholder in the signature (i.e. `f(x; <keyword arguments>)`), and give the complete list under an `# Arguments` section (see point 4 below).
    関数が多くのキーワード引数を受け入れる場合、 `<keyword arguments>`プレースホルダ（すなわち `f（x; <keyword arguments>）`）を含めるだけで、 `＃Arguments`セクション 以下のポイント4）。
2. 
   # Include a single one-line sentence describing what the function does or what the object represents after the simplified signature block. 
    簡略化された署名ブロックの後に、関数が何を表すか、またはオブジェクトが表すものを記述する単一の1行の文を含めます。
   # If needed, provide more details in a second paragraph, after a blank line.
    必要に応じて、空白行の後ろの2番目の段落に詳細を入力します。

   # The one-line sentence should use the imperative form ("Do this", "Return that") instead of the third person (do not write "Returns the length...") when documenting functions. 
    関数を文書化するときに、3行目の代わりに必須の形式（「これを実行」、「返す」）を使用する必要があります（「長さを返します...」）。
   # It should end with a period. If the meaning of a function cannot be summarized easily, splitting it into separate composable parts could be beneficial (this should not be taken as an absolute requirement for every single case though).
    期間が終了するはずです。 関数の意味を簡単に要約することができない場合、それを別々の構成可能な部分に分割することは有益です（ただし、単一のケースごとに絶対的な要件とはみなされません）。
3. 
   # Do not repeat yourself.
    繰り返さないでください。

   # Since the function name is given by the signature, there is no need to start the documentation with "The function `bar`...": go straight to the point. 
    関数名は署名によって与えられているので、「関数 `bar` ...」でドキュメントを開始する必要はありません。ポイントにまっすぐ進みます。
   # Similarly, if the signature specifies the types of the arguments, mentioning them in the description is redundant.
    同様に、シグネチャが引数の型を指定している場合、その記述でそれらを記述することは冗長です。
4. 
   # Only provide an argument list when really necessary.
    本当に必要なときにのみ引数リストを提供してください。

   # For simple functions, it is often clearer to mention the role of the arguments directly in the description of the function's purpose. 
    単純な関数の場合、関数の目的の記述の中で引数の役割を直接言及することがしばしば明らかです。
   # An argument list would only repeat information already provided elsewhere. 
    引数リストは既に他の場所で提供されている情報を繰り返すだけです。
   # However, providing an argument list can be a good idea for complex functions with many arguments (in particular keyword arguments). 
    しかし、引数リストを提供することは、多くの引数（特にキーワード引数）を持つ複雑な関数に対しては良い考えです。
   # In that case, insert it after the general description of the function, under an `# Arguments` header, with one `-` bullet for each argument.
    その場合は、関数の一般的な説明の後に、 `＃Arguments`ヘッダーの下に挿入し、各引数に1つの` -`記号を付けます。
   # The list should mention the types and default values (if any) of the arguments:
    リストには、引数の型とデフォルト値（存在する場合）が記載されている必要があります。

   ```julia
   """
   ...
   # Arguments
   - `n::Integer`: the number of elements to compute.
   - `dim::Integer=1`: the dimensions along which to perform the computation.
   ...
   """
   ```
5. 
   # Include any code examples in an `# Examples` section.
   `＃Examples`セクションにコード例を含めてください。

   # Examples should, whenever possible, be written as *doctests*. 
   例は可能な限り、* doctests *と書くべきです。
   # A *doctest* is a fenced code block (see [Code blocks](@ref)) starting with ````` ```jldoctest````` and contains any number of `julia>` prompts together with inputs and expected outputs that mimic the Julia REPL.
   A * doctest *は、 `` `` `` `` `` `` jldoctest``````で始まる分離コードブロック（[コードブロック]（@ref）参照）であり、いくつかの `julia>`プロンプトが入力と Julia REPLを模倣する期待される出力。

   # For example in the following docstring a variable `a` is defined and the expected result, as printed in a Julia REPL, appears afterwards:
   たとえば、次のdocstringでは、変数 `a`が定義され、Julia REPLに出力されている期待される結果がその後に表示されます。

   ````julia
   """
   Some nice documentation here.

   # Examples

   ```jldoctest
   julia> a = [1 2; 3 4]
   2×2 Array{Int64,2}:
    1  2
    3  4
   ```
   """
   ````

<!-- > !!! warning -->
!!!警告
   # Calling `rand` and other RNG-related functions should be avoided in doctests since they will not produce consistent outputs during different Julia sessions. 
    `rand`と他のRNG関連関数を呼び出すことは、異なるJuliaセッション中に一貫した出力を生成しないので、doctestでは避けるべきです。
   # If you would like to show some random number generation related functionality, one option is to explicitly construct and seed your own [`MersenneTwister`](@ref) (or other pseudorandom number generator) and pass it to the functions you are doctesting.
   乱数生成関連の機能をいくつか表示したい場合は、独自の[`MersenneTwister`]（@ ref）（または他の擬似乱数生成器）を明示的に構築してシードし、それをあなたが作っている関数に渡すことです。

   # Operating system word size ([`Int32`](@ref) or [`Int64`](@ref)) as well as path separator differences (`/` or `\`) will also affect the reproducibility of some doctests.
   オペレーティングシステムのワードサイズ（[`Int32`]（@ref）または[` Int64`]（@ref））とパス区切りの違い（ `/`または `\`）もいくつかのdoctestの再現性に影響します。

   # Note that whitespace in your doctest is significant! The doctest will fail if you misalign the output of pretty-printing an array, for example.
   doctestの空白が重要であることに注意してください。例えば、pretty-printing配列の出力の位置をずらすとdoctestが失敗します。

   # You can then run `make -C doc doctest` to run all the doctests in the Julia Manual, which will ensure that your example works.
   Julia Manualの `do-do docest`を実行すると、すべてのdoctestを実行することができます。これにより、あなたの例が確実に動作するようになります。

   # Examples that are untestable should be written within fenced code blocks starting with ````` ```julia````` so that they are highlighted correctly in the generated documentation.
   テストできない例は、 `` `` `` `julia``````で始まるfencedコードブロック内に、生成された文書で正しく強調表示されるように記述する必要があります。

# !!! tip
  !!!先端
       Wherever possible examples should be **self-contained** and **runnable** so that readers are able to try them out without having to include any dependencies.
       可能な限りの例は、**自己完結型**と**実行可能**でなければなりません。そうすれば、読者は依存関係を含まなくてもそれらを試すことができます。
6. 
   # Use backticks to identify code and equations.
   コードと方程式を識別するためにバッククォートを使用します。

   # Julia identifiers and code excerpts should always appear between backticks ``` ` ``` to enable highlighting. 
   ジュリア識別子とコードの抜粋は、強調表示を有効にするバッククォートの間に必ず現れます。
   # Equations in the LaTeX syntax can be inserted between double backticks ``` `` ```. 
   LaTeX構文の方程式は、ダブルバッククォート `` `` `` ``の間に挿入できます。
   # Use Unicode characters rather than their LaTeX escape sequence, i.e. ``` ``α = 1`` ``` rather than ``` ``\\alpha = 1`` ```.
   LaTeXのエスケープシーケンスではなく、 ``` `` \\ alpha = 1`` ``` ではなく `` ``α= 1`` `` `のUnicode文字を使用してください。
7. 
   # Place the starting and ending `"""` characters on lines by themselves.
   最初と最後の `` ""文字をそれ自身で行に配置します。

   # That is, write
    つまり、次のように書く：

   ```julia
   """
   ...

   ...
   """
   f(x, y) = ...
   ```

   rather than:

   ```julia
   """...

   ..."""
   f(x, y) = ...
   ```

   # This makes it more clear where docstrings start and end.
   これにより、ドキュメントストリングの開始と終了の場所が明確になります。
8. 
   # Respect the line length limit used in the surrounding code.
   周囲のコードで使用される行の長さの制限を尊重します。

   # Docstrings are edited using the same tools as code. 
   コード化と同じツールを使用してコードを編集します。
   # Therefore, the same conventions should apply. 
   したがって、同じ規則が適用されるはずです。
   # It it advised to add line breaks after 92 characters.
   それは92文字の後に改行を加えることを勧めました。

<!-- ## Accessing Documentation -->
### ドキュメントへのアクセス

<!-- Documentation can be accessed at the REPL or in [IJulia](https://github.com/JuliaLang/IJulia.jl) by typing `?` followed by the name of a function or macro, and pressing `Enter`.  -->
ドキュメンテーションは、REPL または [IJulia](https://github.com/JuliaLang/IJulia.jl) で、 `?` と続けて関数またはマクロの名前を入力し、Enterキーを押してアクセスできます。
<!-- For example, -->
例えば、

```julia
?cos
?@time
?r""
```

<!-- will bring up docs for the relevant function, macro or string macro respectively.  -->
それぞれ関連する関数、マクロ、または文字列マクロのドキュメントを表示します。
#In [Juno](http://junolab.org) using `Ctrl-J, Ctrl-D` will bring up documentation for the object under the cursor.
`Ctrl-J` を使用している [Juno](http://junolab.org) では、`Ctrl-D` を押すと、オブジェクトのドキュメントがカーソルの下に表示されます。

<!-- ## Functions & Methods -->
## 関数とメソッド

<!-- Functions in Julia may have multiple implementations, known as methods.  -->
Juliaの関数には、メソッドと呼ばれる複数の実装があります。
#While it's good practice for generic functions to have a single purpose, Julia allows methods to be documented individually if necessary. 
ジェネリック関数が単一の目的を持つのがよい習慣ですが、Juliaは必要に応じてメソッドを個別に文書化することができます。
#In general, only the most generic method should be documented, or even the function itself (i.e. the object created without any methods by `function bar end`). 
一般的に、最も一般的な方法のみを文書化するか、または関数自体（すなわち、「関数バー終了」によってメソッドを一切使用しないで作成されたオブジェクト）さえも記述する必要があります。
#Specific methods should only be documented if their behaviour differs from the more generic ones. 
具体的な方法は、その動作がより一般的なものと異なる場合にのみ文書化されるべきである。
#In any case, they should not repeat the information provided elsewhere. For example:
いずれにしても、他の場所で提供されている情報を繰り返すべきではありません。 例えば：

```julia
"""
    *(x, y, z...)

Multiplication operator. `x * y * z *...` calls this function with multiple
arguments, i.e. `*(x, y, z...)`.
"""
function *(x, y, z...)
    # ... [implementation sold separately] ...
end

"""
    *(x::AbstractString, y::AbstractString, z::AbstractString...)

When applied to strings, concatenates them.
"""
function *(x::AbstractString, y::AbstractString, z::AbstractString...)
    # ... [insert secret sauce here] ...
end

help?> *
search: * .*

  *(x, y, z...)

  Multiplication operator. x * y * z *... calls this function with multiple
  arguments, i.e. *(x,y,z...).

  *(x::AbstractString, y::AbstractString, z::AbstractString...)

  When applied to strings, concatenates them.
```

#When retrieving documentation for a generic function, the metadata for each method is concatenated with the `catdoc` function, which can of course be overridden for custom types.
ジェネリック関数のドキュメンテーションを取得するとき、各メソッドのメタデータは `catdoc`関数と連結されますが、これはもちろんカスタムタイプに対してオーバーライドすることもできます。

<!-- ## Advanced Usage -->
## 高度な使い方

<!-- The `@doc` macro associates its first argument with its second in a per-module dictionary called `META`.  -->
`@doc` マクロは `META` と呼ばれるモジュールごとの辞書で第一引数と第二引数を関連付けます。
#By default, documentation is expected to be written in Markdown, and the `doc""` string macro simply creates an object representing the Markdown content. 
デフォルトでは、ドキュメンテーションはMarkdownで記述され、 `doc""` 文字列マクロは単に Markdown コンテンツを表すオブジェクトを作成します。
#In the future it is likely to do more advanced things such as allowing for relative image or link paths.
将来的には、相対的な画像やリンクのパスを許可するなど、より高度な処理を行う可能性があります。

#When used for retrieving documentation, the `@doc` macro (or equally, the `doc` function) will search all `META` dictionaries for metadata relevant to the given object and return it. 
`@doc` マクロ（あるいは `doc` 関数）は、ドキュメントの検索に使用すると、指定されたオブジェクトに関連するメタデータのすべての `META` 辞書を検索し、それを返します。
<!-- The returned object (some Markdown content, for example) will by default display itself intelligently.  -->
返されたオブジェクト（例えば、Markdown の内容など）は、デフォルトでそれ自身をインテリジェントに表示します。
#This design also makes it easy to use the doc system in a programmatic way; for example, to re-use documentation between different versions of a function:
また、この設計により、プログラマチックな方法で doc システムを使いやすくなります。 たとえば、関数の異なるバージョン間でドキュメントを再利用するには：

```julia
@doc "..." foo!
@doc (@doc foo!) foo
```

#Or for use with Julia's metaprogramming functionality:
Juliaのメタプログラミング機能と併用する場合：

```julia
for (f, op) in ((:add, :+), (:subtract, :-), (:multiply, :*), (:divide, :/))
    @eval begin
        $f(a,b) = $op(a,b)
    end
end
@doc "`add(a,b)` adds `a` and `b` together" add
@doc "`subtract(a,b)` subtracts `b` from `a`" subtract
```

#Documentation written in non-toplevel blocks, such as `begin`, `if`, `for`, and `let`, is added to the documentation system as blocks are evaluated. 
`begin`、` if`、for、および `let`のような非トップレベルブロックで書かれたドキュメントは、ブロックが評価されるときにドキュメンテーションシステムに追加されます。
#For example:
例えば：

```julia
if VERSION > v"0.5"
    "..."
    f(x) = x
end
```

#will add documentation to `f(x)` when the condition is `true`. Note that even if `f(x)` goes out of scope at the end of the block, its documentation will remain.
条件が `true`のときに` f（x） `に文書を追加します。 `f（x）`がブロックの最後に範囲外になっても、その文書は残っていることに注意してください。

<!-- ### Dynamic documentation -->
### 動的ドキュメント

#Sometimes the appropriate documentation for an instance of a type depends on the field values of that instance, rather than just on the type itself. 
場合によっては、型のインスタンスの適切なドキュメントは、型自体ではなく、そのインスタンスのフィールド値によって異なります。
#In these cases, you can add a method to `Docs.getdoc` for your custom type that returns the documentation on a per-instance basis. 
このような場合には、ドキュメンテーションをインスタンス単位で返すカスタム型のメソッドを `Docs.getdoc` に追加することができます。
<!-- For instance, -->
例えば、

```julia
struct MyType
    value::String
end

Docs.getdoc(t::MyType) = "Documentation for MyType with value $(t.value)"

x = MyType("x")
y = MyType("y")
```

#`?x` will display "Documentation for MyType with value x" while `?y` will display "Documentation for MyType with value y".
`？x`は` `xの値を持つMy Typeのドキュメンテーションを表示し、` `y``は` `yの値を持つMy Typeのドキュメンテーションを表示します。

<!-- ## Syntax Guide -->
## 構文ガイド

#A comprehensive overview of all documentable Julia syntax.
すべての文書化可能なJulia構文の包括的な概要。

#In the following examples `"..."` is used to illustrate an arbitrary docstring which may be one of the follow four variants and contain arbitrary text:
次の例では、 `` ... "`は、以下の4つの変種のうちの1つで、任意のテキストを含む任意のdocstringを示すために使用されています。

```julia
"..."

doc"..."

"""
...
"""

doc"""
...
"""
```

#`@doc_str` should only be used when the docstring contains `$` or `\` characters that should not be parsed by Julia such as LaTeX syntax or Julia source code examples containing interpolation.
`@ doc_str`は、docstringがLaTeX構文や補間を含むJuliaソースコードの例のように、Juliaによって解析されるべきでない` $ `または` \ `文字を含む場合にのみ使用されるべきです。

<!-- ## Functions and Methods -->
## 関数とメソッド

```julia
"..."
function f end

"..."
f
```

#Adds docstring `"..."` to `Function``f`. The first version is the preferred syntax, however both are equivalent.
docstring `"..."``を `Function``f`に追加します。 最初のバージョンが優先する構文ですが、どちらも同等です。

```julia
"..."
f(x) = x

"..."
function f(x)
    x
end

"..."
f(x)
```

#Adds docstring `"..."` to `Method``f(::Any)`.
docstring `` ... ... ``を `Method``f（:: Any）`に追加します。

```julia
"..."
f(x, y = 1) = x + y
```

#Adds docstring `"..."` to two `Method`s, namely `f(::Any)` and `f(::Any, ::Any)`.
docstring `` ... ... ``を `f（:: Any）`と `f（:: Any、:: Any）`の2つの `Method`sに追加します。

### Macros

```julia
"..."
macro m(x) end
```

#Adds docstring `"..."` to the `@m(::Any)` macro definition.
`@m（:: Any）`マクロ定義にdocstring `` ... ... ``を追加します。

```julia
"..."
:(@m)
```

#Adds docstring `"..."` to the macro named `@m`.
`@ m`というマクロにdocstring` `... ...` `を追加します。

### Types

```
"..."
abstract type T1 end

"..."
mutable struct T2
    ...
end

"..."
struct T3
    ...
end
```

#Adds the docstring `"..."` to types `T1`, `T2`, and `T3`.
ドキュメントストリング `` ... "を` `T1```、` `T2``、` `T3``型に追加します。

```julia
"..."
struct T
    "x"
    x
    "y"
    y
end
```

#Adds docstring `"..."` to type `T`, `"x"` to field `T.x` and `"y"` to field `T.y`. Also applicable to `mutable struct` types.
docstring `` ... '`を` T`、 `` x "` `T.x`と` `y" `をフィールド` T.y`に追加します。 `mutable struct`型にも適用可能です。

### Modules

```julia
"..."
module M end

module M

"..."
M

end
```

#Adds docstring `"..."` to the `Module``M`. Adding the docstring above the `Module` is the preferred syntax, however both are equivalent.
`` M``にdocstring `` ... ... `を追加します。 `Module`の上にdocstringを追加するのが好ましい構文ですが、どちらも同等です。

```julia
"..."
baremodule M
# ...
end

baremodule M

import Base: @doc

"..."
f(x) = x

end
```

#Documenting a `baremodule` by placing a docstring above the expression automatically imports `@doc` into the module. 
ドキュメントストリングを式の上に置いて `baremodule`を文書化すると自動的に` @ doc`がモジュールにインポートされます。
#These imports must be done manually when the module expression is not documented. Empty `baremodule`s cannot be documented.
これらのインポートは、モジュール式が文書化されていない場合は手動で行う必要があります。 空の `baremodule`sは文書化できません。

### Global Variables

```julia
"..."
const a = 1

"..."
b = 2

"..."
global c = 3
```

#Adds docstring `"..."` to the `Binding`s `a`, `b`, and `c`.
`Binding`sの` a`、 `b`、` c`にdocstring `` ... ... ``を追加します。

#`Binding`s are used to store a reference to a particular `Symbol` in a `Module` without storing the referenced value itself.
`Binding`は、参照された値自体を格納せずに、` Module`内の特定の `Symbol`への参照を格納するために使用されます。

#!!! note
!!! 注意
   # When a `const` definition is only used to define an alias of another definition, such as is the case with the function `div` and its alias `÷` in `Base`, do not document the alias and instead document the actual function.
   `const`定義が別の定義のエイリアスを定義するためだけに使用される場合（例えば、` div`関数と `Base`のエイリアス`÷ `の場合）、エイリアスを文書化せず、代わりに実際の関数 。

   # If the alias is documented and not the real definition then the docsystem (`?` mode) will not return the docstring attached to the alias when the real definition is searched for.
   エイリアスが文書化されていて実際の定義ではない場合、実際の定義が検索されるとき、docsystem（ `？` mode）はエイリアスに付加されたdocstringを返しません。

   # For example you should write
     たとえば、

    ```julia
    "..."
    f(x) = x + 1
    const alias = f
    ```

    rather than

    ```julia
    f(x) = x + 1
    "..."
    const alias = f
    ```

```julia
"..."
sym
```

#Adds docstring `"..."` to the value associated with `sym`. Users should prefer documenting `sym` at it's definition.
docstring `` ... ... ``を `sym`に関連付けられた値に追加します。 ユーザは `sym`の定義を文書化することを好むべきです。

### Multiple Objects
### 複数のオブジェクト

```julia
"..."
a, b
```

#Adds docstring `"..."` to `a` and `b` each of which should be a documentable expression. 
docstring `` ... ... ``をそれぞれ文書化可能な式である `a`と` b`に追加します。
#This syntax is equivalent to
この構文は、

```julia
"..."
a

"..."
b
```

#Any number of expressions many be documented together in this way. This syntax can be useful when two functions are related, such as non-mutating and mutating versions `f` and `f!`.
このようにして、多くの表現が多数記述されています。 この構文は、バージョン `f`と` f！ `を変更したり変更したりするなど、2つの関数が関連している場合に便利です。

### Macro-generated code
### マクロ生成コード

```julia
"..."
@m expression
```

<!-- Adds docstring `"..."` to expression generated by expanding `@m expression`. -->
`@m式`を展開して生成した式にdocstring `` ... ... ``を追加します。
<!-- This allows for expressions decorated with `@inline`, `@noinline`, `@generated`, or any other macro to be documented in the same way as undecorated expressions. -->
これにより、 `@inline`、` @noinline`、 `@ generated`、または他のマクロで装飾された式は、装飾されていない式と同じ方法で文書化できます。

<!-- Macro authors should take note that only macros that generate a single expression will automatically support docstrings. -->
マクロ作成者は、単一の式を生成するマクロだけが自動的にドキュメントストリングをサポートすることに注意する必要があります。
<!-- If a macro returns a block containing multiple subexpressions then the subexpression that should be documented must be marked using the [`@__doc__`](@ref Core.@__doc__) macro. -->
マクロが複数の部分式を含むブロックを返す場合、文書化すべき部分式は[@ __ doc__]（@ ref Core。@ __ doc__）マクロを使ってマークする必要があります。

<!-- The `@enum` macro makes use of `@__doc__` to allow for documenting `Enum`s. -->
`@ enum`マクロは` @__ doc__`を利用して `Enum`を文書化します。
<!-- Examining it's definition should serve as an example of how to use `@__doc__` correctly. -->
定義を調べることは、 `@__ doc__`を正しく使う方法の例として役立つはずです。

```@docs
Core.@__doc__
```

<!-- ## Markdown syntax -->
## マークダウン構文

#The following markdown syntax is supported in Julia.
Juliaでは、次のマークダウン構文がサポートされています。

<!-- ### Inline elements -->
### インライン要素

#Here "inline" refers to elements that can be found within blocks of text, i.e. paragraphs. 
ここで、「インライン」とは、テキストのブロック、すなわち段落内で見つかることができる要素を指す。
#These include the following elements.
これらには、以下の要素が含まれます。

### Bold

#Surround words with two asterisks, `**`, to display the enclosed text in boldface.
囲まれたテキストを太字で表示するには、2つのアスタリスク「**」で囲む単語を囲みます。

```
A paragraph containing a **bold** word.
```

<!-- #### Italics -->
### イタリック体

#Surround words with one asterisk, `*`, to display the enclosed text in italics.
1つのアスタリスク（ `*`）で囲まれた単語を囲み、囲まれたテキストをイタリック体で表示します。

```
A paragraph containing an *emphasised* word.
```

<!-- #### Literals -->
#### リテラル

#Surround text that should be displayed exactly as written with single backticks, ``` ` ``` .
単一のバッククォート `` `` `` `で書かれたとおりに表示されるサラウンドテキスト。

```
A paragraph containing a `literal` word.
```

#Literals should be used when writing text that refers to names of variables, functions, or other parts of a Julia program.
Juliaプログラムの変数、関数、またはその他の部分の名前を参照するテキストを記述するときには、リテラルを使用する必要があります。

#!!! tip
!!! 先端
   # To include a backtick character within literal text use three backticks rather than one to enclose the text.
    リテラルテキスト内にバックティック文字を含めるには、テキストを囲むバッククィックを使用するのではなく、3つのバッククィックを使用します。

    ```
    A paragraph containing a ``` `backtick` character ```.
    ```

   # By extension any odd number of backticks may be used to enclose a lesser number of backticks.
   拡張によって、任意の数のバックティックを使用してより少ない数のバックティックを囲むことができる。

#### ``\LaTeX``

<!-- 
Surround text that should be displayed as mathematics using ``\LaTeX`` syntax with double backticks, ``` `` ``` .
 -->
二重backtick、 `` `` `` ``で `` \ LaTeX``の構文を使って数学的に表示されるべきサラウンドテキスト。

```
A paragraph containing some ``\LaTeX`` markup.
```

<!-- 
!!! tip
    As with literals in the previous section, if literal backticks need to be written within double backticks use an even number greater than two. 
    Note that if a single literal backtick needs to be included within ``\LaTeX`` markup then two enclosing backticks is sufficient.
-->

!!! 先端
    前のセクションのリテラルと同様に、二重バッククォート内にリテラルバッククォートを記述する必要がある場合は、2より大きい偶数を使用します。
    単一のリテラルバックティックを `` \ LaTeX``マークアップに含める必要がある場合は、2つの囲みバッククイックで十分です。



<!-- #### Links -->
#### リンク

<!-- Links to either external or internal addresses can be written using the following syntax, where the text enclosed in square brackets, `[ ]`, is the name of the link and the text enclosed in parentheses, `( )`, is the URL. -->
外部アドレスまたは内部アドレスへのリンクは、次の構文を使用して記述することができます。ここで、角括弧[]で囲まれたテキストはリンクの名前で、括弧で囲まれたテキストは `（）`です。

<!-- 
```
A paragraph containing a link to [Julia](http://www.julialang.org).
```
-->
```
[Julia]（http://www.julialang.org）へのリンクを含むパラグラフ。
```

<!-- It's also possible to add cross-references to other documented functions/methods/variables within the Julia documentation itself. -->
また、Juliaのドキュメント自体に、他のドキュメント化された関数/メソッド/変数への相互参照を追加することもできます。
<!-- For example: -->
例えば：

<!-- ```julia
    eigvals!(A,[irange,][vl,][vu]) -> values

Same as [`eigvals`](@ref), but saves space by overwriting the input `A`, instead of creating a copy.

```
-->

```julia
    eigvals!(A,[irange,][vl,][vu]) -> values

[`eigvals`]（@ ref）と同じですが、コピーを作成する代わりに入力` A`を上書きしてスペースを節約します。

```

<!-- This will create a link in the generated docs to the `eigvals` documentation (which has more information about what this function actually does). -->
これは、生成されたドキュメント内の `eigvals`ドキュメントへのリンクを作成します（この関数が実際に何をするかについての詳細があります）。
<!-- It's good to include cross references to mutating/non-mutating versions of a function, or to highlight a difference between two similar-seeming functions. -->
関数の突然変異/非突然変異バージョンへの相互参照を組み込むことや、類似した2つの関数の違いを強調することは良いことです。

<!-- 
!!! note
    The above cross referencing is *not* a Markdown feature, and relies on [Documenter.jl](https://github.com/JuliaDocs/Documenter.jl), which is used to build base Julia's documentation.
-->
!!! 注意
   上記の相互参照はMarkdown機能ではなく、基本的なJuliaのドキュメントを構築するために使用される[Documenter.jl]（https://github.com/JuliaDocs/Documenter.jl）に依存しています。

<!-- ### Footnote references -->
#### 脚注

<!-- Named and numbered footnote references can be written using the following syntax. -->
名前付きおよび番号付きの脚注の参照は、次の構文を使用して記述することができます。
<!-- A footnote name must be a single alphanumeric word containing no punctuation. -->
脚注名は、句読点を含まない単一の英数字の単語でなければなりません。

```
A paragraph containing a numbered footnote [^1] and a named one [^named].
```

番号付きの脚注[^ 1]と名前付きの[^名前付き]を含む段落。

<!-- 
!!! note
    The text associated with a footnote can be written anywhere within the same page as the footnote reference. 
    The syntax used to define the footnote text is discussed in the [Footnotes](@ref) section below.
-->
!!! 注意
     脚注に関連付けられたテキストは、脚注の参照と同じページ内のどこにでも書くことができます。
     脚注テキストの定義に使用する構文については、下記の[脚注]（@ ref）セクションで説明しています。

<!-- ## Toplevel elements -->
### トップレベル要素

<!-- The following elements can be written either at the "toplevel" of a document or within another "toplevel" element. -->
次の要素は、文書の "トップレベル"または別の "トップレベル"要素のいずれかに書き込むことができます。

<!-- ### Paragraphs -->
### 段落

<!-- A paragraph is a block of plain text, possibly containing any number of inline elements defined in the [Inline elements](@ref) section above, with one or more blank lines above and below it. -->
パラグラフは、上記の[Inline elements]（@ ref）セクションで定義された任意の数のインライン要素を含み、その上と下に空白行が1つ以上あるプレーンテキストのブロックです。

<!-- >
```
This is a paragraph.

And this is *another* one containing some emphasised text.
A new line, but still part of the same paragraph.
```
-->
```
これは段落です。

そして、これはいくつかの強調されたテキストを含む*別のものです。
新しい行ですが、それでも同じ段落の一部です。
```

<!-- ### Headers -->
#### ヘッダー

<!-- A document can be split up into different sections using headers. -->
ドキュメントは、ヘッダーを使用して異なるセクションに分割できます。
<!-- Headers use the following syntax: -->
ヘッダーは次の構文を使用します。

```julia
# Level One
## Level Two
### Level Three
#### Level Four
##### Level Five
###### Level Six
```

<!-- A header line can contain any inline syntax in the same way as a paragraph can. -->
ヘッダー行には、段落缶と同じ方法でインライン構文を含めることができます。

<!-- 
!!! tip
    Try to avoid using too many levels of header within a single document. 
    A heavily nested document may be indicative of a need to restructure it or split it into several pages covering separate topics.
-->
 
 !!! 先端
    1つのドキュメント内でヘッダーのレベルが多すぎるのを避けてください。
    頻繁にネストされたドキュメントは、それを再構成する必要があるか、別のトピックをカバーするいくつかのページに分割する必要があることを示している可能性があります。

<!-- ### Code blocks -->
### コードブロック

<!-- Source code can be displayed as a literal block using an indent of four spaces as shown in the following example. -->
ソースコードは、次の例に示すように4つの空白のインデントを使用してリテラルブロックとして表示できます。

```
This is a paragraph.

    function func(x)
        # ...
    end

Another paragraph.
```

<!-- Additionally, code blocks can be enclosed using triple backticks with an optional "language" to specify how a block of code should be highlighted. -->
さらに、コードブロックをどのように強調表示するかを指定するオプションの「言語」を含むトリプルバッククォートを使用して、コードブロックを囲むことができます。

````
A code block without a "language":

```
function func(x)
    # ...
end
```

# and another one with the "language" specified as `julia`:
「言語」が「ジュリア」と指定されている別のもの：

```julia
function func(x)
    # ...
end
```
````

<!-- 
!!! note
    "Fenced" code blocks, as shown in the last example, should be prefered over indented code blocks since there is no way to specify what language an indented code block is written in.
-->
!!! 注意
    インデントされたコードブロックが書き込まれる言語を指定する方法がないため、最後の例に示すように、 "フェンス"コードブロックをインデントされたコードブロックより優先する必要があります。

<!-- ### Block quotes -->
### ブロック引用符

> Text from external sources, such as quotations from books or websites, can be quoted using `>` characters prepended to each line of the quote as follows.

書籍やウェブサイトからの引用などの外部ソースからのテキストは、次のように引用符の各行の先頭に `> '文字を使用して引用することができます。

```
Here's a quote:

> Julia is a high-level, high-performance dynamic programming language for
> technical computing, with syntax that is familiar to users of other
> technical computing environments.
```

<!-- original -->
> Note that a single space must appear after the `>` character on each line. Quoted blocks may themselves contain other toplevel or inline elements.
<!-- original -->

各行で `>` 文字の後ろにスペースが1つなければならないことに注意してください。 クォートされたブロック自体に、他のトップレベル要素またはインライン要素を含めることができます。

<!-- ### Images -->
#### 画像

<!-- The syntax for images is similar to the link syntax mentioned above. -->
イメージの構文は、上記のリンク構文と似ています。
<!-- Prepending a `!` character to a link will display an image from the specified URL rather than a link to it. -->
`！ '文字をリンクに先行させると、リンクではなく指定されたURLからのイメージが表示されます。

```julia
![alternative text](link/to/image.png)
```

<!-- ### Lists -->
### リスト

<!-- Unordered lists can be written by prepending each item in a list with either `*`, `+`, or `-`. -->
順序付けられていないリストは、各項目を `*`、 `+`、または `--`のいずれかでリストに追加することで記述できます。

```
A list of items:

  * item one
  * item two
  * item three
```

<!-- Note the two spaces before each `*` and the single space after each one. -->
各 `*`の前の2つのスペースと、それぞれの後に1つのスペースがあることに注意してください。

<!-- Lists can contain other nested toplevel elements such as lists, code blocks, or quoteblocks. -->
リストには、リスト、コードブロック、引用ブロックなどの他のネストされたトップレベル要素を含めることができます。
<!-- A blank line should be left between each list item when including any toplevel elements within a list. -->
リスト内に最上位の要素を含めるときは、各リスト項目の間に空白行を残す必要があります。

```
Another list:

  * item one

  * item two

    ```
    f(x) = x
    ```

  * And a sublist:

      + sub-item one
      + sub-item two
```

<!-- 
!!! note
    The contents of each item in the list must line up with the first line of the item. 
    In the above example the fenced code block must be indented by four spaces to align with the `i` in `item two`.
-->
!!! note
     リスト内の各項目の内容は、項目の最初の行と一直線に並ばなければなりません。
     上記の例では、分離コードブロックは、「項目2」の「i」と整列するために4つのスペースでインデントされなければなりません。

<!-- Ordered lists are written by replacing the "bullet" character, either `*`, `+`, or `-`, with a positive integer followed by either `.` or `)`. -->
順序付きリストは、 "*"、 "+"、または " - "のいずれかの "箇条書き"文字を、正の整数とそれに続く `.`または` ``で置き換えることによって作成されます。

```
Two ordered lists:

 1. item one
 2. item two
 3. item three

 5) item five
 6) item six
 7) item seven
```

<!-- An ordered list may start from a number other than one, as in the second list of the above example, where it is numbered from five. -->
順序付きリストは、上記の例の2番目のリストのように、1以外の番号から開始することができます。ここでは、5から番号が付けられています。
<!-- As with unordered lists, ordered lists can contain nested toplevel elements. -->
順序付けされていないリストと同様に、順序付けされたリストにはネストされたトップレベルの要素を含めることができます。

<!-- ### Display equations -->
### 方程式を表示する

<!-- Large ``\LaTeX`` equations that do not fit inline within a paragraph may be written as display equations using a fenced code block with the "language" `math` as in the example below. -->
パラグラフ内でインラインに収まらない大きな `` \ LaTeX``方程式は、以下の例のように "language" `math`を持つfencedコードブロックを使って表示方程式として書くことができます。

````julia
```math
f(a) = \frac{1}{2\pi}\int_{0}^{2\pi} (\alpha+R\cos(\theta))d\theta
```
````

<!-- ### Footnotes -->
### 脚注

<!-- This syntax is paired with the inline syntax for [Footnote references](@ref). -->
この構文は、[脚注の参照]（@ ref）のインライン構文と対になっています。
<!-- Make sure to read that section as well. -->
そのセクションも必ず読んでください。

<!-- Footnote text is defined using the following syntax, which is similar to footnote reference syntax, aside from the `:` character that is appended to the footnote label. -->
脚注テキストは、次の構文を使用して定義されます。これは脚注の参照構文に似ていますが、脚注ラベルに追加される `：`文字以外の構文です。

```
[^1]: Numbered footnote text.

[^note]:

    Named footnote text containing several toplevel elements.

      * item one
      * item two
      * item three

    ```julia
    function func(x)
        # ...
    end
    ```
```

!!! note
   <!-- No checks are done during parsing to make sure that all footnote references have matching footnotes. -->
    すべての脚注の参照に一致する脚注があることを確認するために、解析中にチェックは行われません。

<!-- ### Horizontal rules -->
### 水平ルール

<!-- The equivalent of an `<hr>` HTML tag can be written using the following syntax: -->
`<hr>` HTMLタグに相当するものは、次の構文を使って書くことができます：

```
Text above the line.

---

And text below the line.
```

<!-- ### Tables -->
### テーブル

<!-- Basic tables can be written using the syntax described below. -->
基本テーブルは、以下に説明する構文を使用して記述することができます。
<!-- Note that markdown tables have limited features and cannot contain nested toplevel elements unlike other elements discussed above – only inline elements are allowed. -->
マークダウンテーブルは機能が限定されており、上で説明した他の要素とは異なり、ネストされたトップレベル要素を含むことはできません。インライン要素のみが許可されます。
<!-- Tables must always contain a header row with column names. -->
テーブルには常に列名のヘッダー行が含まれている必要があります。
<!-- Cells cannot span multiple rows or columns of the table. -->
セルは、表の複数の行または列にまたがることはできません。

```
| Column One | Column Two | Column Three |
|:---------- | ---------- |:------------:|
| Row `1`    | Column `2` |              |
| *Row* 2    | **Row** 2  | Column ``3`` |
```

!!! note
    As illustrated in the above example each column of `|` characters must be aligned vertically.

    A `:` character on either end of a column's header separator (the row containing `-` characters) specifies whether the row is left-aligned, right-aligned, or (when `:` appears on both ends) center-aligned. 
    Providing no `:` characters will default to right-aligning the column.

<!-- ### Admonitions -->
### 警告

<!-- Specially formatted blocks with titles such as "Notes", "Warning", or "Tips" are known as admonitions and are used when some part of a document needs special attention. -->
"Notes"、 "Warning"、または "Tips"などのタイトルを持つ特別なフォーマットのブロックは、警告として知られており、文書の一部に特別な注意が必要な場合に使用されます。
<!-- They can be defined using the following `!!!` syntax: -->
それらは次の `!!!`構文を使って定義することができます：

```
!!! note

    This is the content of the note.

!!! warning "Beware!"

    And this is another one.

    This warning admonition has a custom title: `"Beware!"`.
```

<!-- Admonitions, like most other toplevel elements, can contain other toplevel elements. -->
他の最上位要素と同様に、他のトップレベル要素も含むことができます。
<!-- When no title text, specified after the admonition type in double quotes, is included then the title used will be the type of the block, i.e. `"Note"` in the case of the `note` admonition. -->
二重引用符で囲んで指定された型の後に指定されたタイトルテキストが含まれていない場合、使用されるタイトルはブロックのタイプ、つまり `` note` 'の場合は `` Note "`になります。

<!-- ## Markdown Syntax Extensions -->
## マークダウン構文拡張

<!-- Julia's markdown supports interpolation in a very similar way to basic string literals, with the difference that it will store the object itself in the Markdown tree (as opposed to converting it to a string). -->
Juliaのマークダウンは、基本的な文字列リテラルと非常によく似た方法で補間をサポートしています。オブジェクト自体をMarkdownツリーに格納するという違いがあります（文字列に変換するのではなく）。
<!-- When the Markdown content is rendered the usual `show` methods will be called, and these can be overridden as usual. -->
Markdownのコンテンツがレンダリングされると、通常の `show` メソッドが呼び出され、これらは通常通りオーバーライドできます。
<!-- This design allows the Markdown to be extended with arbitrarily complex features (such as references) without cluttering the basic syntax. -->
この設計により、Markdownを任意の複雑なフィーチャ（参照など）で拡張することができ、基本構文が乱雑になりません。

<!-- In principle, the Markdown parser itself can also be arbitrarily extended by packages, or an entirely custom flavour of Markdown can be used, but this should generally be unnecessary. -->
原則として、Markdownパーサ自体は、パッケージによって任意に拡張することもできますし、Markdownの完全なカスタムフレーバを使用することもできますが、これは一般的には不要です。
