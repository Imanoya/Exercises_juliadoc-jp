<!-- Start-->

# Variables

> # 変数

<!--End -->

<!-- Start -->
A variable, in Julia, is a name associated (or bound) to a value. 
>Juliaの変数は、値に関連付けられた（またはバインドされた）名前です。
<!--End -->
<!-- Start -->
It's useful when you want to store a value (that you obtained after some math, for example) for later use.
>後で使用するために値（たとえば、数学の後に得た値）を保存する場合に便利です。 
<!-- Start -->
For example:
>例えば：
<!--End -->

```julia-repl
# Assign the value 10 to the variable x
julia> x = 10
10

# Doing math with x's value
julia> x + 1
11

# Reassign x's value
julia> x = 1 + 1
2

<!-- Start -->
You can assign values of other types, like strings of text
> 文字列のような他の型の値を割り当てることができます
<!--End -->

julia> x = "Hello World!"
"Hello World!"
```

<!-- Start -->
Julia provides an extremely flexible system for naming variables. 
> Juliaは変数の命名に非常に柔軟なシステムを提供します。
<!--End -->
<!-- Start -->
Variable names are case-sensitive, and have no semantic meaning (that is, the language will not treat variables differently based on their names).
> 変数名では大文字と小文字が区別され、文脈的な意味はありません（つまり、名前に基づいて変数が異なる扱いをされることはありません）。
<!--End -->

```jldoctest
julia> x = 1.0
1.0

julia> y = -3
-3

julia> Z = "My string"
"My string"

julia> customary_phrase = "Hello world!"
"Hello world!"

julia> UniversalDeclarationOfHumanRightsStart = "人人生而自由，在尊严和权利上一律平等。"
"人人生而自由，在尊严和权利上一律平等。"
```

Unicode names (in UTF-8 encoding) are allowed:

```jldoctest
julia> δ = 0.00001
1.0e-5

julia> 안녕하세요 = "Hello"
"Hello"
```

<!-- Start -->
In the Julia REPL and several other Julia editing environments, you can type many Unicode math symbols by typing the backslashed LaTeX symbol name followed by tab. 
> Julia REPLや他のいくつかのJulia編集環境では、バックスラッシュされたLaTeXシンボル名の後ろにタブを入力して、多くのUnicode数学記号を入力できます。
<!--End -->
<!-- Start -->
For example, the variable name `δ` can be entered by typing `\delta`-*tab*, or even `α̂₂` by `\alpha`-*tab*-`\hat`- *tab*-`\_2`-*tab*.
> 例えば、変数名 `δ`は` \ delta`- * tab *、 `\ alpha`- * tab * -` \ hat`- * tab * -` \ _2で`α2`と入力することで入力できます ` - * tab *。
<!--End -->
<!-- Start -->
(If you find a symbol somewhere, e.g. in someone else's code, that you don't know how to type, the REPL help will tell you: just type `?` and then paste the symbol.)
> （たとえば、他の人のコードのようなシンボルが見つかった場合は、入力方法がわからないので、REPLのヘルプで「？」と入力してシンボルを貼り付けるだけです）。
<!--End -->

<!-- Start -->
Julia will even let you redefine built-in constants and functions if needed:
> Juliaは、必要に応じて組み込みの定数や関数を再定義することもできます：
<!--End -->

```jldoctest
julia> pi
π = 3.1415926535897...

julia> pi = 3
WARNING: imported binding for pi overwritten in module Main
3

julia> pi
3

julia> sqrt(100)
10.0

julia> sqrt = 4
WARNING: imported binding for sqrt overwritten in module Main
4
```

<!-- Start -->
However, this is obviously not recommended to avoid potential confusion.
> しかし、潜在的な混乱を避けるために、これは明らかに推奨されていません。
<!--End -->

<!-- Start -->

## Allowed Variable Names

> ## 許可された変数名

<!--End -->

<!-- Start -->
Variable names must begin with a letter (A-Z or a-z), underscore, or a subset of Unicode code points greater than 00A0; in particular, [Unicode character categories](http://www.fileformat.info/info/unicode/category/index.htm) Lu/Ll/Lt/Lm/Lo/Nl (letters), Sc/So (currency and other symbols), and a few other letter-like characters (e.g. a subset of the Sm math symbols) are allowed. 
> 変数名は、文字（A-Zまたはa-z）、アンダースコア、または00A0より大きいUnicodeコードポイントのサブセットで始まる必要があります。 Lu / Ll / Lt / Lm / Lo / Nl（文字）、Sc / So（通貨と文字列）他の記号）、およびいくつかの他の文字のような文字（たとえば、Smの数学記号のサブセット）が許可されます。
<!--End -->
<!-- Start -->
Subsequent characters may also include ! and digits (0-9 and other characters in categories Nd/No), as well as other Unicode code points: diacritics and other modifying marks (categories Mn/Mc/Me/Sk), some punctuation connectors (category Pc), primes, and a few other characters.
> 後続の文字には！ （Mn / Mc / Me / Skカテゴリ）、いくつかの句読点コネクタ（カテゴリPc）、素数、およびその他のユニコードコードポイントなどの数字（0〜9およびカテゴリNd / Noのその他の文字）その他いくつかの文字が含まれています。
<!--End -->

<!-- Start -->
Operators like `+` are also valid identifiers, but are parsed specially. 
> 演算子`+`などは、有効な識別子ですが特別に解析されます。
<!--End -->
<!-- Start -->
In some contexts, operators can be used just like variables; for example `(+)` refers to the addition function, and `(+) = f` will reassign it. 
<!--End -->
> 状況によっては、演算子を変数のように使用することもできます。例えば ​​`(+)` は加算関数を指し、`(+)= f` はそれを再割り当てします。
<!-- Start -->
Most of the Unicode infix operators (in category Sm), such as `⊕`, are parsed as infix operators and are available for user-defined methods.
> `⊕`といったほとんどのUnicode中置演算子記号（カテゴリSm）は、は中置演算子として解析され、ユーザー定義のメソッドで利用できます。
<!--End -->
<!-- Start -->
(e.g. you can use `const ⊗ = kron` to define `⊗` as an infix Kronecker product).
> （例えば、`const ⊗ = kron`と定義し`⊗`をクローニッカー積演算子として使うことができます）。
<!--End -->

<!-- Start -->
The only explicitly disallowed names for variables are the names of built-in statements:
> 明示的に許可されていない変数の名前は、組み込みステートメントの名前だけです。
<!--End -->

```julia-repl
julia> else = false
ERROR: syntax: unexpected "else"

julia> try = "No"
ERROR: syntax: unexpected "="
```

<!-- Start -->
<!--- Some Unicode characters are considered to be equivalent in identifiers.
一部のUnicode文字は識別子で同等と見なされます。
<!--End -->
<!-- Start -->
<!--- Different ways of entering Unicode combining characters (e.g., accents) are treated as equivalent (specifically, Julia identifiers are NFC-normalized).
Unicode結合文字（アクセントなど）を入力するさまざまな方法は同等です（具体的には、Julia識別子はNFCで正規化されています）。
<!--End -->
<!-- Start -->
<!--- The Unicode characters `ɛ` (U+025B: Latin small letter open e) and `µ` (U+00B5: micro sign) are treated as equivalent to the corresponding Greek letters, because the former are easily accessible via some input methods.
前者はいくつかの入力方法で簡単にアクセスできるため、Unicode文字「ɛ」（U + 025B：ラテン小文字e）と「μ」（U + 00B5：微小記号）は、対応するギリシャ文字と同等として扱われます。
<!--End -->

<!-- Start -->
## Stylistic Conventions

> ## 文体慣習
<!--End -->

<!-- Start -->
<!--- While Julia imposes few restrictions on valid names, it has become useful to adopt the following conventions:
Juliaは有効な名前にほとんど制限を課しませんが、次のような慣習を採用すると便利です。
<!--End -->
<!-- Start -->

* Names of variables are in lower case.
* > 変数の名前は小文字です。

<!--End -->
<!-- Start -->

* Word separation can be indicated by underscores (`'_'`), but use of underscores is discouraged unless the name would be hard to read otherwise.
* > 単語の区切りはアンダースコア (`'_'`) で示すことができますが、読みにくいものでなければアンダースコアを使用することはお勧めしません。

<!--End -->
<!-- Start -->

* Names of `Type`s and `Module`s begin with a capital letter and word separation is shown with upper camel case instead of underscores.
* > `Type`s と `Module`s の名前は大文字で始まり、単語の区切りはアンダースコアの代わりに `upper-camel-case` で表示します。 

<!--End -->
<!-- Start -->

* Names of `function`s and `macro`s are in lower case, without underscores.
* > `関数` と `マクロ` の名前は、アンダースコアなしで小文字です。

<!--End -->
<!-- Start -->

* Functions that write to their arguments have names that end in `!`. 
* > それらの引数に書き込む関数の名前は `!` で終わります。

<!--End -->

* These are sometimes called "mutating" or "in-place" functions because they are intended to produce changes in their arguments after the function is called, not just return a value.
* > これらは、関数を呼び出した後に値を返すだけでなく、引数に変更を加えることを意図しているため、"突然変異" または "インプレース" 関数と呼ばれることもあります。

<!--End -->

<!-- Start -->
For more information about stylistic conventions, see the [Style Guide](@ref).
> 文法規則の詳細については、[スタイルガイド](@ref) を参照してください。
<!--End -->
