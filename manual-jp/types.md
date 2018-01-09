<!-- > [Types](@id man-types) -->
# [Types](@id man-types)
# [型](@id man-types)

<!-- Type systems have traditionally fallen into two quite different camps: static type systems, where every program expression must have a type computable before the execution of the program, and dynamic type systems, where nothing is known about types until run time, when the actual values manipulated by the program are available.  -->
タイプ・システムは伝統的に2つの全く異なるキャンプに分類されています。静的タイプ・システムでは、プログラムの実行前に計算可能なタイプを持つ静的タイプ・システムと、実行時までタイプについては何も知られていない動的タイプ・システムプログラムによって操作されるものがあります。
<!--- Object orientation allows some flexibility in statically typed languages by letting code be written without the precise types of values being known at compile time.  --->
オブジェクト指向を使用すると静的型付言語で、コンパイル時に値の正確な型がわからなくてもコードを書ける柔軟性が得られます。
<!-- The ability to write code that can operate on different types is called polymorphism.  -->
異なる型で動作できるコードを書く能力は、ポリモルフィズムと呼ばれます。
<!-- All code in classic dynamically typed languages is polymorphic: only by explicitly checking types, or when objects fail to support operations at run-time, are the types of any values ever restricted. -->
動的型定義された古典的な言語のコードはすべて、多型です。タイプを明示的にチェックするか、実行時にオブジェクトが操作をサポートできない場合は、値の型が制限されます。

<!-- Julia's type system is dynamic, but gains some of the advantages of static type systems by making it possible to indicate that certain values are of specific types.  -->
Juliaの型システムは動的ですが、特定の値が特定の型であることを示すことができるようにすることで、静的型システムの利点のいくつかを得ています。
<!-- This can be of great assistance in generating efficient code, but even more significantly, it allows method dispatch on the types of function arguments to be deeply integrated with the language.  -->
これは効率的なコードを生成する上で大きな助けになる可能性がありますが、さらに重要なことは、関数引数の型に対するメソッドディスパッチを言語と深く統合できることです。
<!-- Method dispatch is explored in detail in [Methods](@ref), but is rooted in the type system presented here. -->
メソッドディスパッチは、[Methods](@ref)で詳細に探究されていますが、ここに示した型システムに根ざしています。

<!-- The default behavior in Julia when types are omitted is to allow values to be of any type.  -->
型が省略されたときにJuliaのデフォルトの振る舞いは、値が任意の型になることです。
<!-- Thus, one can write many useful Julia programs without ever explicitly using types.  -->
したがって、型を明示的に使用することなく、多くの有用なJuliaプログラムを書くことができます。
<!-- When additional expressiveness is needed, however, it is easy to gradually introduce explicit type annotations into previously "untyped" code.  -->
しかし、さらなる表現力が必要な場合、以前の"型なし"コードに明示的な型注釈を徐々に導入するのは簡単です。
<!-- Doing so will typically increase both the performance and robustness of these systems, and perhaps somewhat counterintuitively, often significantly simplify them. -->
そうすることで、通常、これらのシステムのパフォーマンスと堅牢性の両方が向上し、おそらく直観に反して、しばしば大幅に簡略化されます。

#Describing Julia in the lingo of [type systems](https://en.wikipedia.org/wiki/Type_system), it is: dynamic, nominative and parametric. 
[type systems](https://en.wikipedia.org/wiki/Type_system)の用語集でJuliaを説明すると、動的、名目上、パラメトリックです。
#Generic types can be parameterized, and the hierarchical relationships between types are [explicitly declared](https://en.wikipedia.org/wiki/Nominal_type_system), rather than [implied by compatible structure](https://en.wikipedia.org/wiki/Structural_type_system).
一般的な型をパラメータ化することができ、型間の階層関係は、[互換構造体に含意](https：//en.wikipedia)ではなく[明示的に宣言](https://en.wikipedia.org/wiki/Nominal_type_system)です。 org / wiki / Structural_type_system)。
#One particularly distinctive feature of Julia's type system is that concrete types may not subtype each other: all concrete types are final and may only have abstract types as their supertypes.
Juliaの型システムの1つの特に特徴的な点は、具体的な型は互いにサブタイプを持たなくてもよいことです。すべての具体的な型は最終的なものであり、抽象型のみをそのスーパータイプとして持つことができます。
#While this might at first seem unduly restrictive, it has many beneficial consequences with surprisingly few drawbacks. 
これは最初は過度に制限的に見えるかもしれませんが、驚くべきことにいくつかの欠点を伴う多くの有益な結果があります。
#It turns out that being able to inherit behavior is much more important than being able to inherit structure, and inheriting both causes significant difficulties in traditional object-oriented languages. 
振る舞いを継承できることは構造を継承することよりも重要であり、継承することは従来のオブジェクト指向言語では重大な困難を引き起こすことが判明しました。
<!-- Other high-level aspects of Julia's type system that should be mentioned up front are: -->
Juliaのタイプシステムの他の上位レベルの側面は、以下のとおりです。

   <!--- * There is no division between object and non-object values: all values in Julia are true objects having a type that belongs to a single, fully connected type graph, all nodes of which are equally first-class as types. --->
   * Juliaのすべての値は、完全に接続された1つのタイプのグラフに属するタイプを持つ真のオブジェクトであり、すべてのノードがタイプとして同等のファーストクラスです。
   <!--- * There is no meaningful concept of a "compile-time type": the only type a value has is its actual type when the program is running.  --->
   * "コンパイル時の型"という意味の概念はありません。値が持つ唯一の型は、プログラムが実行されているときの実際の型です。
   <!--- * This is called a "run-time type" in object-oriented languages where the combination of static compilation with polymorphism makes this distinction significant. --->
   * オブジェクト指向言語では、これをスタティックコンパイルとポリモフィズムの組み合わせで区別することができる、「実行時型」と呼ばれています。
   <!--- * Only values, not variables, have types -- variables are simply names bound to values. --->
   * 変数ではなく値だけが型を持ちます - 変数は単に値に結び付けられた名前です。
   <!--- * Both abstract and concrete types can be parameterized by other types.  --->
   * 抽象型と具象型の両方を他の型でパラメータ化できます。
   <!--- * They can also be parameterized by symbols, by values of any type for which [`isbits()`](@ref) returns true (essentially, things like numbers and bools that are stored like C types or structs with no pointers to other objects), and also by tuples thereof.  --->
   * [`isbits()`](@ ref)がtrueを返す任意の型の値によってシンボルでパラメータ化することもできます(基本的に、Cの型や他のオブジェクトへのポインタを持たない構造体のように格納される数値やbool )、およびそれらのタプルによって。
   <!--- > Type parameters may be omitted when they do not need to be referenced or restricted. --->
   * 型パラメータは、参照または制限する必要がないときに省略することができます。
 
<!-- > Julia's type system is designed to be powerful and expressive, yet clear, intuitive and unobtrusive.  -->
Juliaのタイプシステムは、強力で表現力豊かに設計されています。はっきりとわかりやすく直感的で邪魔にならないように
<!-- > Many Julia programmers may never feel the need to write code that explicitly uses types.  -->
多くのJuliaプログラマは、明示的に型を使用するコードを書く必要性を決して感じないかもしれません。
<!-- > Some kinds of programming, however, become clearer, simpler, faster and more robust with declared types. -->
しかし、いくつかの種類のプログラミングは、宣言された型に対して、より明確で、より簡単に、より速く、より堅牢になります。
 
<!-- ## Type Declarations -->
## 型宣言 (Type DEclarations)

<!-- The `::` operator can be used to attach type annotations to expressions and variables in programs. -->
`::`演算子を使用して、プログラム中の式や変数に型注釈を付けることができます。
<!-- There are two primary reasons to do this: -->
これには主に2つの理由があります。

<!-- 1. As an assertion to help confirm that your program works the way you expect, -->
1. あなたのプログラムが期待どおりに動作することを確認するためのアサーションとして、
<!-- 2. To provide extra type information to the compiler, which can then improve performance in some cases -->
2. コンパイラに余分な型情報を提供することで、場合によってはパフォーマンスを向上させることができます

<!-- When appended to an expression computing a value, the `::` operator is read as "is an instance of". -->
`::`演算子は値を計算する式に追加すると、"is an instance of"と読みまます。 
<!-- It can be used anywhere to assert that the value of the expression on the left is an instance of the type on the right.  -->
左辺の式の値が右辺の型のインスタンスであることを宣言するために、どこでも使用できます。
<!--- When the type on the right is concrete, the value on the left must have that type as its implementation -- recall that all concrete types are final, so no implementation is a subtype of any other. -->
右側の型が具体的な場合、左側の値はその実装としてその型を持たなければなりません。すべての具体的な型が final であることを思い出してください。実装は他の型のサブタイプではありません。
<!--- When the type is abstract, it suffices for the value to be implemented by a concrete type that is a subtype of the abstract type. -->
型が抽象型である場合、抽象型のサブタイプである具体的な型によって値が実装されていれば十分です。
<!--- If the type assertion is not true, an exception is thrown, otherwise, the left-hand value is returned:-->
型検証(type assertion) が真でない場合は例外がスローされ、そうでない場合は左辺値が返されます。

```jldoctest
julia> (1+2)::AbstractFloat
ERROR: TypeError: typeassert: expected AbstractFloat, got Int64

julia> (1+2)::Int
3
```

<!-- This allows a type assertion to be attached to any expression in-place. -->
これにより、型検証(type assertion)を任意の式にその場でアタッチすることができます。

<!--- When appended to a variable on the left-hand side of an assignment, or as part of a `local` declaration, the `::` operator means something a bit different: it declares the variable to always have the specified type, like a type declaration in a statically-typed language such as C. -->
代入の左辺の変数に、または `local`宣言の一部として追加された場合、` :: `演算子は少し異なるものを意味します:
C言語のような静的型言語の型宣言のように、変数が常に指定された型を持つことを宣言します。
<!--- Every value assigned to the variable will be converted to the declared type using [`convert()`](@ref):-->
変数に割り当てられたすべての値は、[`convert()`](@ref) を使って宣言された型に変換されます：

```jldoctest
julia> function foo()
           x::Int8 = 100
           x
       end
foo (generic function with 1 method)

julia> foo()
100

julia> typeof(ans)
Int8
```

<!-- This feature is useful for avoiding performance "gotchas" that could occur if one of the assignments to a variable changed its type unexpectedly. -->
この機能は、変数への代入の1つが予期せずその型を変更した場合に発生する可能性のあるパフォーマンスの"落とし穴"を回避するのに便利です。

<!-- This "declaration" behavior only occurs in specific contexts: -->
この「宣言」動作は、特定のコンテキストでのみ発生します。

```julia
local x::Int8  # in a local declaration
x::Int8 = 10   # as the left-hand side of an assignment
```

<!--- and applies to the whole current scope, even before the declaration.-->
宣言の前であっても、現在のスコープ全体に適用されます。 
<!--- Currently, type declarations cannot be used in global scope, e.g. in the REPL, since Julia does not yet have constant-type globals.-->
現在、型宣言はグローバルスコープでは使用できません。 REPLでは、Juliaはまだ定型のグローバルを持っていないためです。

<!--- Declarations can also be attached to function definitions:-->
宣言は関数定義にも取り付けることができます。

```julia
function sinc(x)::Float64
    if x == 0
        return 1
    end
    return sin(pi*x)/(pi*x)
end
```

<!--- Returning from this function behaves just like an assignment to a variable with a declared type: the value is always converted to `Float64`.-->
この関数からの返り値は、宣言された型の変数への代入と同様に動作します。値は常に `Float64` に変換されます。

<!-- ## Abstract Types -->
##抽象タイプ

#Abstract types cannot be instantiated, and serve only as nodes in the type graph, thereby describing sets of related concrete types: those concrete types which are their descendants. 
抽象型はインスタンス化することはできず、型グラフ内のノードとしてのみ機能し、それによって関連する具体的な型のセット、すなわちそれらの子孫である具体的な型を記述します。
#We begin with abstract types even though they have no instantiation because they are the backbone of the type system: they form the conceptual hierarchy which makes Julia's type system more than just a collection of object implementations.
抽象型は、型システムのバックボーンであるためにインスタンス化されていないにもかかわらず、抽象型から始まります。それらはJuliaの型システムを単なるオブジェクト実装の集合体以上にする概念的階層を形成します。

#Recall that in [Integers and Floating-Point Numbers](@ref), we introduced a variety of concrete types of numeric values: [`Int8`](@ref), [`UInt8`](@ref), [`Int16`](@ref), [`UInt16`](@ref), [`Int32`](@ref), [`UInt32`](@ref), [`Int64`](@ref), [`UInt64`](@ref), [`Int128`](@ref), [`UInt128`](@ref), [`Float16`](@ref), [`Float32`](@ref), and [`Float64`](@ref). 
[整数と浮動小数点数](@ ref)では、さまざまな具体的な数値型を導入したことを思い出してください：[`Int8`](@ ref)、[` UInt8`](@ref)、[`Int16 @ ref)、[`Int32`](@ ref)、[` UInt32`](@ref)、[`Int64`](@ref)、[` UInt64`] @ ref)、[`UInt128`](@ ref)、[` Float16`](@ ref)、[`Float32`](@ ref)、および` ` Float64`](@ ref)。
<!-- Although they have different representation sizes, `Int8`, `Int16`, `Int32`, `Int64` and `Int128` all have in common that they are signed integer types.  -->
表記サイズは異なりますが、`Int8`,`Int16`,`Int32`,`Int64`,`Int128` はすべて符号付き整数型であるという共通点があります。
<!-- Likewise `UInt8`, `UInt16`, `UInt32`, `UInt64` and `UInt128` are all unsigned integer types, while `Float16`, `Float32` and `Float64` are distinct in being floating-point types rather than integers.  -->
同様に `UInt8`,`UInt16`, `UInt32`,`UInt64`,`UInt128` はすべて符号なし整数型ですが、 `Float16`,`Float32`,`Float64` は整数ではなく浮動小数点型です。
#It is common for a piece of code to make sense, for example, only if its arguments are some kind of integer,
例えば、その引数がある種の整数である場合にのみ、コードが意味をなさせるのが一般的です。
<!-- but not really depend on what particular *kind* of integer.  -->
実際には整数の *種類* には依存しません。
<!-- For example, the greatest common denominator algorithm works for all kinds of integers, but will not work for floating-point numbers.  -->
たとえば、最大の共通分母アルゴリズムはあらゆる種類の整数で動作しますが、浮動小数点数では機能しません。
<!-- Abstract types allow the construction of a hierarchy of types, providing a context into which concrete types can fit.  -->
抽象型は、型の階層の構築を可能にし、具体的な型が適合できるコンテキストを提供します。
<!-- This allows you, for example, to easily program to any type that is an integer, without restricting an algorithm to a specific type of integer. -->
これにより、アルゴリズムを特定の整数型に制限することなく、任意の整数型で簡単にプログラムすることができます。

<!-- Abstract types are declared using the `abstract type` keyword.  -->
抽象型は `abstract type` キーワードを使用して宣言されます。
<!-- The general syntaxes for declaring an abstract type are: -->
抽象型を宣言するための一般的な構文は次のとおりです。

```
abstract type «name» end
abstract type «name» <: «supertype» end
```
<!-- The `abstract type` keyword introduces a new abstract type, whose name is given by `«name»`. -->
`abstract type`キーワードは新しい抽象型を導入します。その名前は` name`によって与えられます。
<!-- This name can be optionally followed by `<:` and an already-existing type, indicating that the newly declared abstract type is a subtype of this "parent" type.-->
この名前の後にオプションで `<:`と既に存在する型が続き、新しく宣言された抽象型がこの "親"型のサブタイプであることを示します。

<!-- When no supertype is given, the default supertype is `Any` -- a predefined abstract type that all objects are instances of and all types are subtypes of. -->
スーパータイプが指定されていない場合、デフォルトのスーパータイプは `Any` であり、 -- `Any` は事前定義された抽象タイプで、すべてのオブジェクトが`Any` のインスタンスであり、すべてのタイプが `Any` のサブタイプです。
<!-- In type theory, `Any` is commonly called "top" because it is at the apex of the type graph. -->
型理論では、型グラフの頂点にあるので、`Any` は一般に `top` と呼ばれます。
<!-- Julia also has a predefined abstract "bottom" type, at the nadir of the type graph, which is written as `Union{}`. -->
Juliaには、型グラフの最下位にあらかじめ定義された抽象的な *ボトム* 型があり、これは `Union{}` として書かれています。
<!-- It is the exact opposite of `Any`: no object is an instance of `Union{}` and all types are supertypes of `Union{}`.-->
これは `Any` とまったく反対です。オブジェクトは `Union{}` のインスタンスではなく、すべての型は `Union{}` のスーパータイプです。

<!-- Let's consider some of the abstract types that make up Julia's numerical hierarchy: -->
Juliaの数値階層を構成する抽象型のいくつかを考えてみましょう。

```julia
abstract type Number end
abstract type Real     <: Number end
abstract type AbstractFloat <: Real end
abstract type Integer  <: Real end
abstract type Signed   <: Integer end
abstract type Unsigned <: Integer end
```

<!-- The [`Number`](@ref) type is a direct child type of `Any`, and [`Real`](@ref) is its child. -->
[`Number`](@ref) 型は `Any` の直接の子型であり、 [`Real`](@ref) はその子です。
<!-- In turn, `Real` has two children (it has more, but only two are shown here; we'll get to the others later): [`Integer`](@ref) and [`AbstractFloat`](@ref), separating the world into representations of integers and representations of real numbers. -->
そしては、`Real` には2つの子(もっとたくさんありますが、ここでは2つしか表示されません;後でもう一度やります)があり世界を [`Integer`](@ref) と [`AbstractFloat`](@ref) 、整数の表象と実数の表象に分けています。
<!-- Representations of real numbers include, of course, floating-point types, but also include other types, such as rationals. -->
実数の表現には、もちろん浮動小数点型が含まれますが、有理数などの他の型も含まれます。
<!-- Hence, `AbstractFloat` is a proper subtype of `Real`, including only floating-point representations of real numbers.-->
したがって、 `AbstractFloat` は実数の浮動小数点表現だけを含む適切な `Real` のサブタイプです。
<!-- Integers are further subdivided into [`Signed`](@ref) and [`Unsigned`](@ref) varieties.-->
整数はさらに [`Signed`](@ref) と [`Unsigned`](@ref) の種類に細分されます。

<!-- The `<:` operator in general means "is a subtype of", and, used in declarations like this, declares the right-hand type to be an immediate supertype of the newly declared type.  -->
一般的に `<:` 演算子は、"is a subtype of" を意味し、このような宣言で使用されると、右手型が新しく宣言された型の直近のスーパータイプになると宣言します。
<!-- It can also be used in expressions as a subtype operator which returns `true` when its left operand is a subtype of its right operand: -->
左辺オペランドが右オペランドのサブタイプであるときに `true` を返すサブタイプ演算子として式で使用することもできます：

```jldoctest
julia> Integer <: Number
true

julia> Integer <: AbstractFloat
false
```

<!-- An important use of abstract types is to provide default implementations for concrete types.  -->
抽象型の重要な用途は、具体的な型のデフォルトの実装を提供することです。
<!-- To give a simple example, consider: -->
簡単な例を挙げてみましょう。

```julia
function myplus(x,y)
    x+y
end
```

<!-- The first thing to note is that the above argument declarations are equivalent to `x::Any` and `y::Any`.  -->
最初の注意点は、上記の引数宣言は `x::Any` と `y::Any` と同等であることです。

<!-- When this function is invoked, say as `myplus(2,5)`, the dispatcher chooses the most specific method named `myplus` that matches the given arguments. -->
この関数が呼び出されると、 `myplus(2,5)` のように、ディスパッチャーは与えられた引数に一致する最も具体的なメソッド `myplus` を選択します。
<!-- (See [Methods](@ref) for more information on multiple dispatch.) -->
(マルチディスパッチの詳細については、 [メソッド](@ref) を参照してください)。

#Assuming no method more specific than the above is found, Julia next internally defines and compiles a method called `myplus` specifically for two `Int` arguments based on the generic function given above, i.e., it implicitly defines and compiles:
上記よりも具体的なメソッドが見つからないと仮定すると、ジュリアは内部的に、上記のジェネリック関数に基づいて2つの `Int` 引数に対して特に `myplus` というメソッドを内部的に定義してコンパイルします。

```julia
function myplus(x::Int,y::Int)
    x+y
end
```

<!-- and finally, it invokes this specific method. -->
最後に、この特定のメソッドを呼び出します。

<!-- Thus, abstract types allow programmers to write generic functions that can later be used as the default method by many combinations of concrete types.  -->
したがって、抽抽象型はプログラマが汎用関数を書くことを可能にし、後で具体的な型の多くの組み合わせによってデフォルトメソッドとして使用できます。
<!-- Thanks to multiple dispatch, the programmer has full control over whether the default or more specific method is used. -->
複数のディスパッチのおかげで、プログラマは、デフォルトのメソッドまたはより特定のメソッドを使用するかどうかを完全に制御できます。

<!-- An important point to note is that there is no loss in performance if the programmer relies on a function whose arguments are abstract types, because it is recompiled for each tuple of argument concrete types with which it is invoked.  -->
注目すべき重要な点は、プログラマが関数で抽象型引数に頼った場合パフォーマンスの低下がありません、なぜなら呼び出される引数型の各タプルに対して再コンパイルされるためです。
#(There may be a performance issue, however, in the case of function arguments that are containers of abstract types; see [Performance Tips](@ref man-performance-tips).)
パフォーマンス上の問題が、抽象型のコンテナである関数の引数の場合にあります:([パフォーマンスのヒント](@ref man-performance-tips) を参照してください。

<!-- ## Primitive Types -->
##プリミティブ型

#A primitive type is a concrete type whose data consists of plain old bits. Classic examples of primitive types are integers and floating-point values. 
プリミティブ型は、データが普通の古いビットで構成される具象型です。プリミティブ型の古典的な例は、整数と浮動小数点値です。
#Unlike most languages, Julia lets you declare your own primitive types, rather than providing only a fixed set of built-in ones. 
ほとんどの言語とは異なり、Juliaは固定セットの組み込み関数を提供するのではなく、独自のプリミティブ型を宣言します。
#In fact, the standard primitive types are all defined in the language itself:
実際、標準プリミティブ型はすべて言語自体で定義されています。

```julia
primitive type Float16 <: AbstractFloat 16 end
primitive type Float32 <: AbstractFloat 32 end
primitive type Float64 <: AbstractFloat 64 end

primitive type Bool <: Integer 8 end
primitive type Char 32 end

primitive type Int8    <: Signed   8 end
primitive type UInt8   <: Unsigned 8 end
primitive type Int16   <: Signed   16 end
primitive type UInt16  <: Unsigned 16 end
primitive type Int32   <: Signed   32 end
primitive type UInt32  <: Unsigned 32 end
primitive type Int64   <: Signed   64 end
primitive type UInt64  <: Unsigned 64 end
primitive type Int128  <: Signed   128 end
primitive type UInt128 <: Unsigned 128 end
```

#The general syntaxes for declaring a primitive type are:
プリミティブ型を宣言するための一般的な構文は次のとおりです。

```
primitive type «name» «bits» end
primitive type «name» <: «supertype» «bits» end
```


#The number of bits indicates how much storage the type requires and the name gives the new type a name. 
ビット数は、タイプに必要なストレージの量を示し、名前は新しいタイプに名前を付けます。
#A primitive type can optionally be declared to be a subtype of some supertype. 
プリミティブ型は、スーパータイプのサブタイプとして任意に宣言することができます。
#If a supertype is omitted, then the type defaults to having `Any` as its immediate supertype. 
スーパータイプが省略されている場合、そのタイプは、デフォルトのスーパータイプとして `Any`を持つことがデフォルトとなります。
#The declaration of [`Bool`](@ref) above therefore means that a boolean value takes eight bits to store, and has [`Integer`](@ref) as its immediate supertype. Currently, only sizes that are multiples of 8 bits are supported. 
したがって、上記の `` Bool``(@ ref)の宣言は、ブール値が格納するのに8ビットを要し、直接のスーパータイプとして[`Integer`](@ ref)を持つことを意味します。現在、8ビットの倍数であるサイズのみがサポートされています。
#Therefore, boolean values, although they really need just a single bit, cannot be declared to be any smaller than eight bits.
したがって、ブール値は実際には単なるビットである必要がありますが、8ビットより小さく宣言することはできません。

#The types [`Bool`](@ref), [`Int8`](@ref) and [`UInt8`](@ref) all have identical representations: they are eight-bit chunks of memory. 
[`Bool`](@ref),[`Int8`](@ref) と [`UInt8`](@ref) の型はすべて同じ表現をしています：それらは8ビットのメモリです。
#Since Julia's type system is nominative, however, they are not interchangeable despite having identical structure. 
しかし、Juliaのタイプシステムは名目上のものであるため、同一の構造を持っていても互換性はありません。
#A fundamental difference between them is that they have different supertypes: [`Bool`](@ref)'s direct supertype is [`Integer`](@ref), [`Int8`](@ref)'s is [`Signed`](@ref), and [`UInt8`](@ref)'s is [`Unsigned`](@ref). 
[Bool](@ ref)の直接スーパータイプは[`Integer`](@ ref)、[` Int8`](@ ref)は `` (@ ref)、[`UInt8`](@ref)は` `Unsigned`(@ ref)です。
#All other differences between [`Bool`](@ref), [`Int8`](@ref), and [`UInt8`](@ref) are matters of behavior -- the way functions are defined to act when given objects of these types as arguments. 
[Bool`](@ ref)、[`Int8`](@ref)、[` UInt8`](@ref)の間のその他の違いは、動作の問題です。これらの型のうちの1つを引数として取ります。
#This is why a nominative type system is necessary: if structure determined type, which in turn dictates behavior, then it would be impossible to make [`Bool`](@ref) behave any differently than [`Int8`](@ref) or [`UInt8`](@ref).
これは、指名型システムが必要な理由です：構造決定型が振る舞いを指示する場合、 [`Bool`](@ref) を [`Int8`](@ref) または [`UInt8`](@ref)  と異なるように動作させることは不可能です。

<!--- ## Composite Types-->
##コンポジットタイプ

#[Composite types](https://en.wikipedia.org/wiki/Composite_data_type) are called records, structs, or objects in various languages. 
[複合型](https://en.wikipedia.org/wiki/Composite_data_type)は、さまざまな言語のレコード、構造体、またはオブジェクトと呼ばれます。
#A composite type is a collection of named fields, an instance of which can be treated as a single value. 
複合型は、名前付きフィールドのコレクションであり、そのインスタンスは単一の値として扱うことができます。
#In many languages, composite types are the only kind of user-definable type, and they are by far the most commonly used user-defined type in Julia as well.
多くの言語では、コンポジット型はユーザ定義可能な唯一の型であり、Juliaでも最も一般的に使用されるユーザ定義型です。

#In mainstream object oriented languages, such as C++, Java, Python and Ruby, composite types also have named functions associated with them, and the combination is called an "object". 
C ++、Java、Python、Rubyなどの主流のオブジェクト指向言語では、複合型にもそれらに関連付けられた名前付き関数があり、その組み合わせを「オブジェクト」と呼びます。
#In purer object-oriented languages, such as Ruby or Smalltalk, all values are objects whether they are composites or not. 
RubyやSmalltalkのようなより純粋なオブジェクト指向言語では、すべての値は合成であろうとなかろうとオブジェクトです。
#In less pure object oriented languages, including C++ and Java, some values, such as integers and floating-point values, are not objects, while instances of user-defined composite types are true objects with associated methods. 
より純粋でないオブジェクト指向言語(C ++やJavaなど)では、整数や浮動小数点値などの一部の値はオブジェクトではなく、ユーザー定義コンポジット型のインスタンスは、関連するメソッドを持つ真のオブジェクトです。
#In Julia, all values are objects, but functions are not bundled with the objects they operate on. 
Juliaでは、すべての値がオブジェクトですが、関数は操作対象のオブジェクトにバンドルされていません。
#This is necessary since Julia chooses which method of a function to use by multiple dispatch, meaning that the types of *all* of a function's arguments are considered when selecting a method, rather than just the first one (see [Methods](@ref)
これは、Juliaが複数のディスパッチで使用する関数のメソッドを選択するので必要です。つまり、メソッドの選択時に、メソッドの引数の* all *型が考慮されます([メソッド](@ ref )
#for more information on methods and dispatch). Thus, it would be inappropriate for functions to "belong" to only their first argument. 
メソッドとディスパッチの詳細については、を参照してください)。したがって、関数が最初の引数だけに属していることは不適切です。
#Organizing methods into function objects rather than having named bags of methods "inside" each object ends up being a highly beneficial aspect of the language design.
各オブジェクトの「内部」に名前のついたバッグを持たせるのではなく、メソッドを関数オブジェクトに編成することは、言語設計の非常に有益な側面になります。

<!--- Composite types are introduced with the `struct` keyword followed by a block of field names, optionally annotated with types using the `::` operator:-->
コンポジット型は、`struct` キーワードとそれに続くフィールド名のブロックで導入され、オプションで `::` 演算子を使って型を注釈します：

```jldoctest footype
julia> struct Foo
           bar
           baz::Int
           qux::Float64
       end
```

<!--- Fields with no type annotation default to `Any`, and can accordingly hold any type of value.-->
型アノテーションのないフィールドは、デフォルトで `Any` になり、それに応じて任意の型の値を保持することができます。

<!--- #New objects of type `Foo` are created by applying the `Foo` type object like a function to values for its fields:-->
`Foo` 型の新しいオブジェクトは、関数のような `Foo` 型オブジェクトをそのフィールドの値に適用することによって作成されます：

```jldoctest footype
julia> foo = Foo("Hello, world.", 23, 1.5)
Foo("Hello, world.", 23, 1.5)

julia> typeof(foo)
Foo
```


<!--- When a type is applied like a function it is called a *constructor*. -->
型が関数のように適用されるときは、*コンストラクタ* と呼ばれます。
<!--- Two constructors are generated automatically (these are called *default constructors*). One accepts any arguments and calls [`convert()`](@ref) to convert them to the types of the fields, and the other accepts arguments that match the field types exactly. -->
2つのコンストラクタが自動的に生成されます(これらは *デフォルトコンストラクタ* と呼ばれます)。 1つは引数を受け取り、[`convert()`](@ref) を呼び出してフィールドの型に変換し、もう一方はフィールドの型に完全に一致する引数を受け取ります。
<!--- The reason both of these are generated is that this makes it easier to add new definitions without inadvertently replacing a default constructor.-->
これらの両方が生成される理由は、これにより、不注意にデフォルトのコンストラクタを置き換えずに新しい定義を追加することが容易になります。

<!--- Since the `bar` field is unconstrained in type, any value will do. -->
`bar` フィールドはタイプに制限されていないので、どんな値でも構いません。
<!--- However, the value for `baz` must be convertible to `Int`:-->
しかし、 `baz` の値は `Int` に変換可能でなければなりません：

```jldoctest footype
julia> Foo((), 23.5, 1)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int64}, ::Float64) at ./float.jl:680
 [2] Foo(::Tuple{}, ::Float64, ::Int64) at ./none:2
```

<!--- You may find a list of field names using the `fieldnames` function.-->
フィールド名のリストは `fieldnames` 関数を使って見つけることができます。

```jldoctest footype
julia> fieldnames(Foo)
3-element Array{Symbol,1}:
 :bar
 :baz
 :qux
```

<!--- You can access the field values of a composite object using the traditional `foo.bar` notation:-->
フィールド名のリストは `fieldnames` 関数を使って見つけることができます。

```jldoctest footype
julia> foo.bar
"Hello, world."

julia> foo.baz
23

julia> foo.qux
1.5
```

<!--- Composite objects declared with `struct` are *immutable*; they cannot be modified after construction. -->
`struct` で宣言された複合オブジェクトは *immutable* です。 構築後に変更することはできません。
<!--- This may seem odd at first, but it has several advantages:-->
これは最初は奇妙に見えるかもしれませんが、いくつかの利点があります：

   * 
   # It can be more efficient. 
    より効率的にすることができます。
   # Some structs can be packed efficiently into arrays, and in some cases the compiler is able to avoid allocating immutable objects entirely.
    いくつかの構造体を効率的に配列にパックすることができ、場合によっては、コンパイラが不変オブジェクトを完全に割り当てることを避けることができます。
   * 
   # It is not possible to violate the invariants provided by the type's constructors.
     型のコンストラクタによって提供される不変量に違反することはできません。
   * 
   # Code using immutable objects can be easier to reason about.
     不変オブジェクトを使用するコードは、推論するのが簡単になります。

#An immutable object might contain mutable objects, such as arrays, as fields. 
不変オブジェクトには、配列などの変更可能なオブジェクトがフィールドとして含まれることがあります。

#Those contained objects will remain mutable; only the fields of the immutable object itself cannot be changed to point to different objects.
含まれているオブジェクトは変更可能なままです。 不変オブジェクト自体のフィールドだけを変更して、異なるオブジェクトを指すことはできません。

#Where required, mutable composite objects can be declared with the keyword `mutable struct`, to be discussed in the next section.
必要に応じて、mutable compositeオブジェクトは `mutable struct`というキーワードで宣言できます。これについては次のセクションで説明します。

#Composite types with no fields are singletons; there can be only one instance of such types:
フィールドのない複合型はシングルトンです。 そのような型のインスタンスは1つだけです。

```jldoctest
julia> struct NoFields
       end

julia> NoFields() === NoFields()
true
```

#The `===` function confirms that the "two" constructed instances of `NoFields` are actually one and the same. 
`===`関数は、`NoFields` の "2つの" 構築されたインスタンスが実際には一つで同じものであることを確認します。

#There is much more to say about how instances of composite types are created, but that discussion depends on both [Parametric Types](@ref) and on [Methods](@ref), and is sufficiently important to be addressed in its own section: [Constructors](@ref man-constructors).
複合型のインスタンスがどのように作成されるかについてははるかに多くの話がありますが、その議論は[パラメトリック型](@ ref)と[メソッド](@ref)の両方に依存し、それ自身のセクション ：[コンストラクタ](@ref man-constructors)
#Singleton types are described in further detail [below](@ref man-singleton-types).
シングルトン型については、以下で詳しく説明します [below](@ref man-singleton-types)。

## Mutable Composite Types
##変更可能な複合型

#If a composite type is declared with `mutable struct` instead of `struct`, then instances of it can be modified:
コンポジット型が `struct` の代わりに `mutable struct` で宣言されている場合は、そのインスタンスを変更することができます：

```jldoctest bartype
julia> mutable struct Bar
           baz
           qux::Float64
       end

julia> bar = Bar("Hello", 1.5);

julia> bar.qux = 2.0
2.0

julia> bar.baz = 1//2
1//2
```

#In order to support mutation, such objects are generally allocated on the heap, and have stable memory addresses.
変異をサポートするために、そのようなオブジェクトは一般にヒープ上に割り当てられ、安定したメモリアドレスを有する。
#A mutable object is like a little container that might hold different values over time, and so can only be reliably identified with its address.
可変オブジェクトは、時間の経過とともに異なる値を保持する可能性のある小さなコンテナと似ているため、そのアドレスでのみ確実に識別できます。
#In contrast, an instance of an immutable type is associated with specific field values --- the field values alone tell you everything about the object.
対照的に、不変型のインスタンスは、特定のフィールド値に関連付けられています。フィールド値だけでは、オブジェクトに関するすべての情報が表示されます。
#In deciding whether to make a type mutable, ask whether two instances with the same field values would be considered identical, or if they might need to change independently over time. If they would be considered identical, the type should probably be immutable.
タイプを変更可能にするかどうかを決めるには、同じフィールド値を持つ2つのインスタンスが同一であるとみなされるか、または時間の経過とともに独立して変更する必要があるかどうかを尋ねます。 それらが同一であると考えられるならば、その型はおそらく不変であるべきです。

#To recap, two essential properties define immutability in Julia:
要約すると、Juliaにおける不変性を定義する2つの重要な特性：

   * 
   # An object with an immutable type is passed around (both in assignment statements and in function calls) by copying, whereas a mutable type is passed around by reference.
    不変型のオブジェクトは、コピーによって代入文と関数呼び出しの両方で渡されますが、変更可能な型は参照渡しされます。
   * 
   # It is not permitted to modify the fields of a composite immutable type.
    コンポジット不変型のフィールドを変更することはできません。

#It is instructive, particularly for readers whose background is C/C++, to consider why these two properties go hand in hand.  
C/C++ のバックグラウンドを持つ読者にとっては、なぜこれらの2つのプロパティが連携しているのかを考えるのは有益なことです。
#If they were separated, i.e., if the fields of objects passed around by copying could be modified, then it would become more difficult to reason about certain instances of generic code.  
それらが分離されている場合、すなわちコピーによって渡されたオブジェクトのフィールドが変更された場合、ジェネリックコードの特定のインスタンスについて推論することはより困難になる。
#For example, suppose `x` is a function argument of an abstract type, and suppose that the function changes a field: `x.isprocessed = true`.  
たとえば、 `x` が抽象型の関数引数であり、関数がフィールドを変更したとしましょう： `x.isprocessed = true`。
#Depending on whether `x` is passed by copying or by reference, this statement may or may not alter the actual argument in the calling routine.  
`x` がコピーによって渡されるのか、参照によって渡されるのかによって、このステートメントは呼び出し側ルーチンの実際の引数を変更することも、変更しないこともあります。
#Julia sidesteps the possibility of creating functions with unknown effects in this scenario by forbidding modification of fields of objects passed around by copying.
Juliaは、このシナリオで未知の効果を持つ関数を作成する可能性を回避するため、コピーによって渡されたオブジェクトのフィールドの変更を禁止します。

### Declared Types
##宣言された型

#The three kinds of types discussed in the previous three sections are actually all closely related.
前述の3つのセクションで説明した3種類のタイプは、実際はすべて密接に関連しています。
#They share the same key properties:
それらは同じキープロパティを共有します：

   * 
   <!-- > They are explicitly declared. -->
    それらは明示的に宣言されています。
   * 
   <!-- > They have names. -->
    彼らは名前を持っています。
   * 
   <!-- > They have explicitly declared supertypes. -->
    彼らは明示的にスーパータイプを宣言しています。
   * 
   <!-- > They may have parameters. -->
    彼らはパラメータを持つかもしれません。

#Because of these shared properties, these types are internally represented as instances of the same concept, `DataType`, which is the type of any of these types:
これらの共有プロパティのため、これらの型は内部的に同じコンセプトのインスタンスとして表され、これらの型のいずれかの型である `DataType` があります。

```jldoctest
julia> typeof(Real)
DataType

julia> typeof(Int)
DataType
```

#A `DataType` may be abstract or concrete. 
`DataType` は抽象的なものでもよいし具体的なものでもよい。

#If it is concrete, it has a specified size, storage layout, and (optionally) field names. 
具体的であれば、指定されたサイズ、ストレージレイアウト、および(オプションで)フィールド名を持ちます。
#Thus a bits type is a `DataType` with nonzero size, but no field names. A composite type is a `DataType` that has field names or is empty (zero size).
したがってビットタイプは、サイズがゼロではなくフィールド名を持たない `DataType` です。 複合型は、フィールド名を持つ、または空(ゼロサイズ)の  `DataType` です。

#Every concrete value in the system is an instance of some `DataType`.
システムのあらゆる具体的な値は、いくつかの `DataType` のインスタンスです。

### Type Unions
##型連合

#A type union is a special abstract type which includes as objects all instances of any of its argument types, constructed using the special `Union` function:
型共用体は特別な抽象型であり、特別な `Union` 関数を使用して構築された、その引数型のすべてのインスタンスをオブジェクトとして含みます：

```jldoctest
julia> IntOrString = Union{Int,AbstractString}
Union{AbstractString, Int64}

julia> 1 :: IntOrString
1

julia> "Hello!" :: IntOrString
"Hello!"

julia> 1.0 :: IntOrString
ERROR: TypeError: typeassert: expected Union{AbstractString, Int64}, got Float64
```

#The compilers for many languages have an internal union construct for reasoning about types; Julia simply exposes it to the programmer.
多くの言語のコンパイラには、型についての推論のための内部結合構造があります。 Juliaは単純にそれをプログラマに公開します。

<!-- ## Parametric Types -->
##パラメトリックタイプ

#An important and powerful feature of Julia's type system is that it is parametric: types can take parameters, so that type declarations actually introduce a whole family of new types -- one for each possible combination of parameter values. 
Juliaの型システムの重要かつ強力な機能は、それがパラメトリックであることです。型はパラメータを取ることができるので、型宣言は実際に新しい型のファミリを導入します。
#There are many languages that support some version of [generic programming](https://en.wikipedia.org/wiki/Generic_programming), wherein data structures and algorithms to manipulate them may be specified without specifying the exact types involved.
[ジェネリックプログラミング](https://en.wikipedia.org/wiki/Generic_programming)のいくつかのバージョンをサポートする多くの言語があります。これらの言語を操作するデータ構造やアルゴリズムは、正確な型を指定せずに指定できます。
#For example, some form of generic programming exists in ML, Haskell, Ada, Eiffel, C++, Java, C#, F#, and Scala, just to name a few. Some of these languages support true parametric polymorphism (e.g. ML, Haskell, Scala), while others support ad-hoc, template-based styles of generic programming (e.g. C++, Java). 
たとえば、ML、Haskell、Ada、Eiffel、C ++、Java、C＃、F＃、およびScalaには、いくつかの例を挙げて、いくつかの汎用プログラミング形式が存在します。これらの言語の中には真のパラメトリック多形性(例：ML、Haskell、Scala)をサポートするものもあれば、テンプレートベースの一般的なプログラミングスタイル(C ++、Javaなど)をサポートするものもあります。
#With so many different varieties of generic programming and parametric types in various languages, we won't even attempt to compare Julia's parametric types to other languages, but will instead focus on explaining Julia's system in its own right. 
さまざまな言語のジェネリックプログラミングとパラメトリックタイプが非常に多種多様であるため、Juliaのパラメトリックタイプを他の言語と比較しようとはしませんが、代わりにJuliaのシステムについて説明することに専念します。
#We will note, however, that because Julia is a dynamically typed language and doesn't need to make all type decisions at compile time, many traditional difficulties encountered in static parametric type systems can be relatively easily handled.
しかし、Juliaは動的に型付けされた言語であり、コンパイル時にすべての型決定を行う必要がないため、静的パラメトリック型システムで発生する多くの従来の問題は比較的簡単に処理できます。

#All declared types (the `DataType` variety) can be parameterized, with the same syntax in each case. 
すべての宣言型( `DataType`バラエティ)は、それぞれの場合に同じ構文でパラメータ化できます。
#We will discuss them in the following order: first, parametric composite types, then parametric abstract types, and finally parametric bits types.
最初に、パラメトリックコンポジット型、パラメトリック抽象型、最後にパラメトリックビット型の順に説明します。

#### Parametric Composite Types
###パラメトリックコンポジット型

#Type parameters are introduced immediately after the type name, surrounded by curly braces:
型パラメータは、中括弧で囲まれた型名の直後に挿入されます。

```jldoctest pointtype
julia> struct Point{T}
           x::T
           y::T
       end
```

#This declaration defines a new parametric type, `Point{T}`, holding two "coordinates" of type `T`. 
この宣言は、 `T`型の2つの「座標」を保持する新しいパラメトリック型、` Point {T} `を定義します。
#What, one may ask, is `T`? 
誰かが聞くかもしれないことは、 `T 'ですか？
#Well, that's precisely the point of parametric types: it can be any type at all (or a value of any bits type, actually, although here it's clearly used as a type).
さて、それはパラメトリックタイプのポイントです。どのタイプでもかまいません(実際にはビットタイプの値ですが、実際はタイプとして使用されています)。
#`Point{Float64}` is a concrete type equivalent to the type defined by replacing `T` in the definition of `Point` with [`Float64`](@ref). 
`Point {Float64}`は、 `Point`の定義で` T`を[`Float64`](@ ref)に置き換えて定義した型に相当する具体的な型です。
#Thus, this single declaration actually declares an unlimited number of types: `Point{Float64}`, `Point{AbstractString}`, `Point{Int64}`, etc. 
したがって、この1つの宣言は、実際には `Point {Float64}`、 `Point {AbstractString}`、 `Point {Int64}`など無制限の数を宣言します。
#Each of these is now a usable concrete type:
これらのそれぞれは現在、使用可能な具体的なタイプです：

```jldoctest pointtype
julia> Point{Float64}
Point{Float64}

julia> Point{AbstractString}
Point{AbstractString}
```

#The type `Point{Float64}` is a point whose coordinates are 64-bit floating-point values, while the type `Point{AbstractString}` is a "point" whose "coordinates" are string objects (see [Strings](@ref)).
タイプ `Point {Float64}` は座標が64ビットの浮動小数点値である点であり、
タイプ "Point {AbstractString}"は、 "座標"が文字列オブジェクト([Strings](@ ref)参照)である "ポイント"です。

<!-- `Point` itself is also a valid type object, containing all instances `Point{Float64}`, `Point{AbstractString}`,etc. -->
`Point` 自体も有効な型オブジェクトであり、`Point{Float64}`,`Point{AbstractString}`, など。
<!--  as subtypes: -->
サブタイプとして：

```jldoctest pointtype
julia> Point{Float64} <: Point
true

julia> Point{AbstractString} <: Point
true
```

<!-- Other types, of course, are not subtypes of it: -->
もちろん、他のタイプはそれのサブタイプではありません：

```jldoctest pointtype
julia> Float64 <: Point
false

julia> AbstractString <: Point
false
```

#Concrete `Point` types with different values of `T` are never subtypes of each other:
`T`の値が異なる具体的な` Point`型は決して相互のサブタイプではありません：

```jldoctest pointtype
julia> Point{Float64} <: Point{Int64}
false

julia> Point{Float64} <: Point{Real}
false
```

#!!! warning
!!! 警告

   # This last point is *very* important: even though `Float64 <: Real` we **DO NOT** have `Point{Float64} <: Point{Real}`.
    この最後の点は*非常に重要です： `Float64 <：Real`でも、私たちは** Point {Float64} <：Point {Real}を持っていません。

#In other words, in the parlance of type theory, Julia's type parameters are *invariant*, rather than being [covariant (or even contravariant)](https://en.wikipedia.org/wiki/Covariance_and_contravariance_%28computer_science%29). 
言い換えれば、型理論の言い方では、Juliaの型パラメータは[共変(または反変)]ではなく、不変*です(https://en.wikipedia.org/wiki/Covariance_and_contravariance_%28computer_science%29)。
#This is for practical reasons: while any instance of `Point{Float64}` may conceptually be like an instance of `Point{Real}` as well, the two types have different representations in memory:
これは実用的な理由によるものです： `Point {Float64}`のインスタンスは、概念的には `Point {Real} 'のインスタンスのようなものかもしれませんが、メモリ内で異なる表現をしています：

   * 
   # An instance of `Point{Float64}` can be represented compactly and efficiently as an immediate pair of 64-bit values;
   ＃「Point {Float64}」のインスタンスは、64ビット値の直近のペアとしてコンパクトかつ効率的に表現できます。
   * 
   # An instance of `Point{Real}` must be able to hold any pair of instances of [`Real`](@ref). 
    #Point {Real}のインスタンスは、[`Real`](@ ref)のインスタンスのペアを保持できなければなりません。
   #Since objects that are instances of `Real` can be of arbitrary size and structure, in practice an instance of `Point{Real}` must be represented as a pair of pointers to individually allocated `Real` objects.
    #Realのインスタンスであるオブジェクトは任意の大きさと構造を持つことができるので、実際にはPoint {Real}のインスタンスは個々に割り当てられた `Real`オブジェクトへのポインタの対として表現されなければなりません。


#The efficiency gained by being able to store `Point{Float64}` objects with immediate values is magnified enormously in the case of arrays: an `Array{Float64}` can be stored as a contiguous memory block of 64-bit floating-point values, whereas an `Array{Real}` must be an array of pointers to individually allocated [`Real`](@ref) objects -- which may well be [boxed](https://en.wikipedia.org/wiki/Object_type_%28object-oriented_programming%29#Boxing) 64-bit floating-point values, but also might be arbitrarily large, complex objects, which are declared to be implementations of the `Real` abstract type.
`Point {Float64}`オブジェクトを即値で格納できることによって得られる効率は、配列の場合には非常に大きくなります。 `Array {Float64}`は、64ビット浮動小数点値の連続したメモリブロックとして格納できます `Array {Real}`は個々に割り当てられた[`Real`](@ref)オブジェクトへのポインタの配列でなければなりません - (boxed)(https://en.wikipedia.org/wiki/ Object_type_％28object-oriented_programming％29＃Boxing)64ビット浮動小数点値だけでなく、 `Real`抽象型の実装であると宣言されている任意の大きさの複雑なオブジェクトでも構いません。

#Since `Point{Float64}` is not a subtype of `Point{Real}`, the following method can't be applied to arguments of type `Point{Float64}`:
`Point {Float64}`は `Point {Real}`のサブタイプではないので、 `Point {Float64}`型の引数に以下のメソッドを適用することはできません：

```julia
function norm(p::Point{Real})
    sqrt(p.x^2 + p.y^2)
end
```

#A correct way to define a method that accepts all arguments of type `Point{T}` where `T` is a subtype of [`Real`](@ref) is:
`T`が[` Real`](@ ref)のサブタイプである `Point {T}`型の引数をすべて受け入れるメソッドを定義する正しい方法は次のとおりです：

```julia
function norm(p::Point{<:Real})
    sqrt(p.x^2 + p.y^2)
end
```

#(Equivalently, one could define `function norm{T<:Real}(p::Point{T})` or `function norm(p::Point{T} where T<:Real)`; see [UnionAll Types](@ref).)
(同様に、「関数のノルム{T <：Real}(p :: Point {T})」または「関数ノルム(p :: Point {T}ここでT <：Real) (@ref)。)

#More examples will be discussed later in [Methods](@ref).
より多くの例については、後ほど[方法](@ref)で説明します。

#How does one construct a `Point` object? It is possible to define custom constructors for composite types, which will be discussed in detail in [Constructors](@ref man-constructors), but in the absence of any special constructor declarations, there are two default ways of creating new composite objects, one in which the type parameters are explicitly given and the other in which they are implied by the arguments to the object constructor.
どのようにして `Point` オブジェクトを構成しますか？ [コンストラクタ](@ref man-constructors) で詳しく説明しているコンポジット型のカスタムコンストラクタを定義することは可能ですが、特別なコンストラクタ宣言がない場合、新しいコンポジットオブジェクトを作成する2つのデフォルトの方法があります。 型パラメータが明示的に与えられ、型のパラメータがオブジェクトコンストラクタへの引数によって暗示されるもの。

<!--- Since the type `Point{Float64}` is a concrete type equivalent to `Point` declared with [`Float64`](@ref) in place of `T`, it can be applied as a constructor accordingly:-->
`Point{Float64}` 型は `T` の代わりに [`Float64`](@ref) で宣言された `Point` に相当する具体的な型ですので、それに応じてコンストラクタとして適用することができます：

```jldoctest pointtype
julia> Point{Float64}(1.0, 2.0)
Point{Float64}(1.0, 2.0)

julia> typeof(ans)
Point{Float64}
```

<!--- For the default constructor, exactly one argument must be supplied for each field:-->
デフォルトコンストラクタでは、各フィールドに1つの引数を指定する必要があります。

```jldoctest pointtype
julia> Point{Float64}(1.0)
ERROR: MethodError: Cannot `convert` an object of type Float64 to an object of type Point{Float64}
```
<!--- This may have arisen from a call to the constructor Point{Float64}(...),-->
<!--- since type constructors fall back to convert methods.-->
これは、コンストラクタ Point{Float64}(...) への呼び出しから発生した可能性があります。
型コンストラクタはメソッドを変換するために後退します。

```
Stacktrace:
 [1] Point{Float64}(::Float64) at ./sysimg.jl:102

julia> Point{Float64}(1.0,2.0,3.0)
ERROR: MethodError: no method matching Point{Float64}(::Float64, ::Float64, ::Float64)
```

<!--- Only one default constructor is generated for parametric types, since overriding it is not possible. This constructor accepts any arguments and converts them to the field types.-->
パラメトリック型に対しては、デフォルトのコンストラクタは1つだけ生成されます。オーバーライドできないためです。 このコンストラクタは引数を受け取り、フィールド型に変換します。

<!--- In many cases, it is redundant to provide the type of `Point` object one wants to construct, since the types of arguments to the constructor call already implicitly provide type information. -->
多くの場合、コンストラクタ呼び出しの引数の型はすでに暗黙的に型情報を提供しているため、構築したい `Point` オブジェクトの型を提供することは冗長です。
<!--- For that reason, you can also apply `Point` itself as a constructor, provided that the implied value of the parameter type `T` is unambiguous:-->
そのため、パラメータ型 `T` の暗黙の値が明白であれば、`Point` 自体をコンストラクタとして適用することもできます：

```jldoctest pointtype
julia> Point(1.0,2.0)
Point{Float64}(1.0, 2.0)

julia> typeof(ans)
Point{Float64}

julia> Point(1,2)
Point{Int64}(1, 2)

julia> typeof(ans)
Point{Int64}
```

<!--- In the case of `Point`, the type of `T` is unambiguously implied if and only if the two arguments to `Point` have the same type. When this isn't the case, the constructor will fail with a [`MethodError`](@ref):-->
`Point` の場合、`T` の型は、 `Point` への2つの引数が同じ型を持つ場合にのみ明白に暗示されます。 そうでない場合、コンストラクタは[`MethodError`](@ref) で失敗します：

```jldoctest pointtype
julia> Point(1,2.5)
ERROR: MethodError: no method matching Point(::Int64, ::Float64)
Closest candidates are:
  Point(::T, !Matched::T) where T at none:2
```

<!--- Constructor methods to appropriately handle such mixed cases can be defined, but that will not be discussed until later on in [Constructors](@ref man-constructors).-->
そのような混合したケースを適切に扱うコンストラクタメソッドは定義できますが、後ほど [コンストラクタ](@ref man-constructor) で議論されることはありません。

<!-- ## Parametric Abstract Types-->
###パラメトリック抽象型 ( Parametric Abstract Types )

<!--- Parametric abstract type declarations declare a collection of abstract types, in much the same way:-->
パラメトリック抽象型宣言は、ほぼ同じ方法で抽象型のコレクションを宣言します。

```jldoctest pointytype
julia> abstract type Pointy{T} end
```

<!--- With this declaration, `Pointy{T}` is a distinct abstract type for each type or integer value of `T`. -->
この宣言では、 `Pointy{T}` は、`T` の各型または整数値の異なる抽象型です。
<!--- As with parametric composite types, each such instance is a subtype of `Pointy`:-->
パラメトリック・コンポジット・タイプと同様に、そのようなインスタンスはそれぞれ、`Pointy`のサブタイプです。

```jldoctest pointytype
julia> Pointy{Int64} <: Pointy
true

julia> Pointy{1} <: Pointy
true
```

<!-- Parametric abstract types are invariant, much as parametric composite types are:-->
パラメトリックな抽象型は、パラメトリックな複合型と同じように不変です。

```jldoctest pointytype
julia> Pointy{Float64} <: Pointy{Real}
false

julia> Pointy{Real} <: Pointy{Float64}
false
```

#The notation `Pointy{<:Real}` can be used to express the Julia analogue of a *covariant* type, while `Pointy{>:Int}` the analogue of a *contravariant* type, but technically these represent *sets* of types (see [UnionAll Types](@ref)).
表記法 `Pointy{<:Real}` を使用して、IIII型のJuliaアナログを表現することができます。
一方、Pointy __は、反変種のアナログです。
しかし、技術的には型の集合を表します（[UnionAll Types]（@ ref）を参照）。

`Pointy{<:Real}` という表記法は、*共変型* 型の Julia 類似体を表現するために使用できますが、`Pointy{>:Int}`は反反例型の類似体ですが、 ([UnionAll Types](@ref)を参照してください)。

```jldoctest pointytype
julia> Pointy{Float64} <: Pointy{<:Real}
true

julia> Pointy{Real} <: Pointy{>:Int}
true
```

#Much as plain old abstract types serve to create a useful hierarchy of types over concrete types, parametric abstract types serve the same purpose with respect to parametric composite types. 
普通の古い抽象型は、具体的な型よりも型の有用な階層を作成するのに役立ちますが、パラメトリックな抽象型はパラメトリックな複合型に関して同じ目的を果たします。
#We could, for example, have declared `Point{T}` to be a subtype of `Pointy{T}` as follows:
たとえば、 `Point {T}`を `Pointy {T}`のサブタイプと宣言すると、次のようになります。

```jldoctest pointytype
julia> struct Point{T} <: Pointy{T}
           x::T
           y::T
       end
```

<!-- Given such a declaration, for each choice of `T`, we have `Point{T}` as a subtype of `Pointy{T}`: -->
そのような宣言が与えられると、`T` の各選択肢に対して `Pointy{T}` のサブタイプとして `Point{T}` があります：

```jldoctest pointytype
julia> Point{Float64} <: Pointy{Float64}
true

julia> Point{Real} <: Pointy{Real}
true

julia> Point{AbstractString} <: Pointy{AbstractString}
true
```

<!-- This relationship is also invariant: -->
この関係も不変です：

```jldoctest pointytype
julia> Point{Float64} <: Pointy{Real}
false

julia> Point{Float64} <: Pointy{<:Real}
true
```

<!-- What purpose do parametric abstract types like `Pointy` serve?-->
どのような目的に`Pointy`のようなパラメトリック抽象タイプが役立つのでしょうか？
<!-- Consider if we create a point-like implementation that only requires a single coordinate because the point is on the diagonal line *x = y*:-->
ポイントが対角線 *x = y*上にあるため、1つの座標だけを必要とするポイントライクな実装を作成した場合、を考えてみましょう、

```jldoctest pointytype
julia> struct DiagPoint{T} <: Pointy{T}
           x::T
       end
```

<!-- Now both `Point{Float64}` and `DiagPoint{Float64}` are implementations of the `Pointy{Float64}` abstraction, and similarly for every other possible choice of type `T`.  -->
現在、 `Point{Float64}` と `DiagPoint{Float64}` の両方が `Pointy{Float64}` 抽象化の実装であり、`T` 型の他の全ての可能な選択についても同様です。
<!-- This allows programming to a common interface shared by all `Pointy` objects, implemented for both `Point` and `DiagPoint`. -->
これは、`Point` と `DiagPoint` の両方に実装されたすべての `Pointy` オブジェクトによって共有される共通インタフェースへのプログラミングを可能にします。
<!-- This cannot be fully demonstrated, however, until we have introduced methods and dispatch in the next section, [Methods](@ref). -->
しかし、次のセクション [メソッド](@ref) にメソッドとディスパッチが導入されるまで、これを完全に証明することはできません。

#There are situations where it may not make sense for type parameters to range freely over all possible types. 
タイプパラメータがすべての可能なタイプにわたって自由に範囲を定めることが意味を成さない場合があります。
#In such situations, one can constrain the range of `T` like so:
このような状況では、 `T 'の範囲を次のように制限することができます：

```jldoctest realpointytype
julia> abstract type Pointy{T<:Real} end
```

#With such a declaration, it is acceptable to use any type that is a subtype of [`Real`](@ref) in place of `T`, but not types that are not subtypes of `Real`:
このような宣言では、 `T` の代わりに [`Real`](@ref) のサブタイプであるタイプは使用できますが、 `Real` のサブタイプではないタイプは使用できません。

```jldoctest realpointytype
julia> Pointy{Float64}
Pointy{Float64}

julia> Pointy{Real}
Pointy{Real}

julia> Pointy{AbstractString}
ERROR: TypeError: Pointy: in T, expected T<:Real, got Type{AbstractString}

julia> Pointy{1}
ERROR: TypeError: Pointy: in T, expected T<:Real, got Int64
```

<!-- Type parameters for parametric composite types can be restricted in the same manner: -->
パラメトリックコンポジットタイプのタイプパラメータは、同じ方法で制限できます。

```julia
struct Point{T<:Real} <: Pointy{T}
    x::T
    y::T
end
```

#To give a real-world example of how all this parametric type machinery can be useful, here is the actual definition of Julia's [`Rational`](@ref) immutable type (except that we omit the constructor here for simplicity), representing an exact ratio of integers: 
このパラメトリック型の機械がどれほど有用であるかの現実的な例を与えるために、Juliaの[`Rational`](@ ref)不変型の実際の定義があります(ここでは単純化のためにコンストラクタを省略します)。 整数の正確な比：
`
```julia
struct Rational{T<:Integer} <: Real
    num::T
    den::T
end
```

#It only makes sense to take ratios of integer values, so the parameter type `T` is restricted to being a subtype of [`Integer`](@ref), and a ratio of integers represents a value on the real number line, so any [`Rational`](@ref) is an instance of the [`Real`](@ref) abstraction.
整数型の比率を取ることは意味があるので、パラメータ型 `T 'は[` Integer`](@ ref)のサブタイプに限定され、整数の比は実数ライン上の値を表します。 任意の[`Rational`](@ ref)は[` Real`](@ ref)抽象のインスタンスです。

###タプルの型 (Tuple Types)

#Tuples are an abstraction of the arguments of a function -- without the function itself. 
タプルは関数の引数の抽象であり、関数自体はありません。
#The salient aspects of a function's arguments are their order and their types. 
関数の引数の顕著な側面は、その順序と型です。
#Therefore a tuple type is similar to a parameterized immutable type where each parameter is the type of one field. 
したがって、タプル型は、各パラメータが1つのフィールドの型であるパラメータ化された不変型に似ています。
#For example, a 2-element tuple type resembles the following immutable type:
たとえば、2要素タプル型は、次の不変型に似ています。

```julia
struct Tuple2{A,B}
    a::A
    b::B
end
```

#However, there are three key differences:
しかし、3つの重要な違いがあります。

   * 
   # Tuple types may have any number of parameters.
    タプル型は、任意の数のパラメータを持つことができます。
   * 
   # Tuple types are *covariant* in their parameters: `Tuple{Int}` is a subtype of `Tuple{Any}`. 
    タプルの型は、それらのパラメータで*共変*です： `Tuple {Int}`は `Tuple {Any} 'のサブタイプです。
   # Therefore `Tuple{Any}` is considered an abstract type, and tuple types are only concrete if their parameters are.
    したがって、 `Tuple {Any}`は抽象型とみなされ、タプル型はそのパラメータが指定されている場合にのみ具体的です。
   * 
   # Tuples do not have field names; fields are only accessed by index.
    タプルにはフィールド名はありません。 フィールドはインデックスによってのみアクセスされます。

#Tuple values are written with parentheses and commas.
タプル値は、カッコとカンマで書かれています。
#When a tuple is constructed, an appropriate tuple type is generated on demand:
タプルが構築されると、必要に応じて適切なタプル型が生成されます。

```jldoctest
julia> typeof((1,"foo",2.5))
Tuple{Int64,String,Float64}
```

<!-- Note the implications of covariance:-->
共分散の意味に注意してください。

```jldoctest
julia> Tuple{Int,AbstractString} <: Tuple{Real,Any}
true

julia> Tuple{Int,AbstractString} <: Tuple{Real,Real}
false

julia> Tuple{Int,AbstractString} <: Tuple{Real,}
false
```

#Intuitively, this corresponds to the type of a function's arguments being a subtype of the function's signature (when the signature matches).
これは直感的に、関数の引数の型(関数の署名のサブタイプ)(関数の署名が一致する場合)に対応します。

<!-- ### Vararg Tuple Types-->
###バラックタプルの型 ( Vararg Tuple Types )

#The last parameter of a tuple type can be the special type `Vararg`, which denotes any number of trailing elements:
タプル型の最後のパラメータは特殊な型 `Vararg`です。これは任意の数の後続要素を表します：

```jldoctest
julia> mytupletype = Tuple{AbstractString,Vararg{Int}}
Tuple{AbstractString,Vararg{Int64,N} where N}

julia> isa(("1",), mytupletype)
true

julia> isa(("1",1), mytupletype)
true

julia> isa(("1",1,2), mytupletype)
true

julia> isa(("1",1,2,3.0), mytupletype)
false
```

#Notice that `Vararg{T}` corresponds to zero or more elements of type `T`. 
`Vararg {T}`は、タイプが `T`の0個以上の要素に対応することに注意してください。
#Vararg tuple types are used to represent the arguments accepted by varargs methods (see [Varargs Functions](@ref)). 
varargsタプル型は、varargsメソッドで受け入れられる引数を表すために使用されます([Varargs関数](@ ref)を参照)。
#The type `Vararg{T,N}` corresponds to exactly `N` elements of type `T`.  
`Vararg {T、N} '型は` T`型の厳密に `N'個の要素に対応します。
#`NTuple{N,T}` is a convenient alias for `Tuple{Vararg{T,N}}`, i.e. a tuple type containing exactly `N` elements of type `T`.
`NTuple {N、T}`は `Tuple {Vararg {T、N}}`の便利なエイリアスです。すなわち、 `T`型の厳密に` N '個の要素を含むタプル型です。

<!-- #### [Singleton Types](@id man-singleton-types)-->
#### シングルトンタイプ [Singleton Types](@id man-singleton-types)

#There is a special kind of abstract parametric type that must be mentioned here: singleton types. 
特別な種類の抽象パラメトリック型について、ここで言及しなけれなりません：シングルトン型。
#For each type, `T`, the "singleton type" `Type{T}` is an abstract type whose only instance is the object `T`. 
`T`型のそれぞれについて、「シングルトン型」`型{T}`はインスタンスがオブジェクト`T`である抽象型です。
#Since the definition is a little difficult to parse, let's look at some examples:
定義は解析するのが少し難しいので、いくつかの例を見てみましょう：

```jldoctest
julia> isa(Float64, Type{Float64})
true

julia> isa(Real, Type{Float64})
false

julia> isa(Real, Type{Real})
true

julia> isa(Float64, Type{Real})
false
```

#In other words, [`isa(A,Type{B})`](@ref) is true if and only if `A` and `B` are the same object and that object is a type. 
言い換えれば、 [`isa(A,Type{B})`](@ref) は `A` と `B` が同じオブジェクトで、そのオブジェクトが型である場合にのみtrueです。
#Without the parameter, `Type` is simply an abstract type which has all type objects as its instances, including, of course, singleton types:
パラメータがなければ、`Type` は単なる抽象型であり、すべての型オブジェクトをインスタンスとして持ちます。もちろんシングルトンも含みます

```jldoctest
julia> isa(Type{Float64}, Type)
true

julia> isa(Float64, Type)
true

julia> isa(Real, Type)
true
```

<!-- Any object that is not a type is not an instance of `Type`: -->
型でないオブジェクトは `Type` のインスタンスではありません：

```jldoctest
julia> isa(1, Type)
false

julia> isa("foo", Type)
false
```

#Until we discuss [Parametric Methods](@ref) and [conversions](@ref conversion-and-promotion), it is difficult to explain the utility of the singleton type construct, but in short, it allows one to specialize function behavior on specific type *values*. 
[パラメトリックメソッド](@ref) と [コンバージョン](@ref conversion-and-promotion) について議論するまで、シングルトン型構造の有用性を説明するのは難しいですが、要するに、 特定のタイプ*値*。

#This is useful for writing methods (especially parametric ones) whose behavior depends on a type that is given as an explicit argument rather than implied by the type of one of its arguments.
これは、その引数の型の型ではなく、明示的な引数として与えられる型に依存する振る舞いを持つメソッド(特にパラメトリックなもの)を書くのに便利です。

#A few popular languages have singleton types, including Haskell, Scala and Ruby. 
いくつかの一般的な言語には、Haskell、Scala、Rubyなどのシングルトンタイプがあります。
#In general usage, the term "singleton type" refers to a type whose only instance is a single value. 
一般的な使用法では、「シングルトンタイプ」という用語は、インスタンスが単一の値であるタイプを指します。
#This meaning applies to Julia's singleton types, but with that caveat that only type objects have singleton types.
この意味はJuliaのシングルトンタイプに適用されますが、タイプオブジェクトだけがシングルトンタイプを持つという注意点があります。

### Parametric Primitive Types
###パラメトリックプリミティブ型

#Primitive types can also be declared parametrically. 
プリミティブ型はパラメトリックに宣言することもできます。
#For example, pointers are represented as primitive types which would be declared in Julia like this:
たとえば、ポインタはプリミティブ型として表されます。プリミティブ型は、次のようにJuliaで宣言されます。

```julia
# 32-bit system:
primitive type Ptr{T} 32 end

# 64-bit system:
primitive type Ptr{T} 64 end
```

#The slightly odd feature of these declarations as compared to typical parametric composite types, is that the type parameter `T` is not used in the definition of the type itself -- it is just an abstract tag, essentially defining an entire family of types with identical structure, differentiated only by their type parameter. 
典型的なパラメトリックコンポジット型と比較して、これらの宣言のほんの奇妙な特徴は、型パラメータ 'T'が型自体の定義に使用されていないということです。 同一の構造であり、それらの型パラメータによってのみ区別される。
#Thus, `Ptr{Float64}` and `Ptr{Int64}` are distinct types, even though they have identical representations. 
したがって、 `Ptr {Float64}`と `Ptr {Int64}`は、同じ表現をしていても、異なる型です。
#And of course, all specific pointer types are subtypes of the umbrella `Ptr` type:
なので当然、すべての特定のポインタ型は、傘型 `Ptr`型のサブタイプです：

```jldoctest
julia> Ptr{Float64} <: Ptr
true

julia> Ptr{Int64} <: Ptr
true
```

<!-- # UnionAll Types -->
## UnionAllタイプ

#We have said that a parametric type like `Ptr` acts as a supertype of all its instances (`Ptr{Int64}` etc.). 
私たちは、 `Ptr`のようなパラメトリック型はすべてのインスタンス(` Ptr {Int64} `など)のスーパータイプとして機能すると言ってきました。
#How does this work? `Ptr` itself cannot be a normal data type, since without knowing the type of the referenced data the type clearly cannot be used for memory operations.
これはどのように作動するのでしょうか？ `Ptr`自体は、参照されるデータの型を知らなくても型が明らかにメモリ操作に使用できないので、通常のデータ型であることはできません。
#The answer is that `Ptr` (or other parametric types like `Array`) is a different kind of type called a `UnionAll` type. 
答えは、 `Ptr`(あるいは` Array`のような他のパラメトリック型)は `UnionAll`型と呼ばれる異なる種類の型です。
#Such a type expresses the *iterated union* of types for all values of some parameter.
このような型は、いくつかのパラメータのすべての値に対して型の*反復された和集合*を表現します。

#`UnionAll` types are usually written using the keyword `where`. 
`UnionAll`型は、通常、キーワード` where`を使って書かれます。
#For example `Ptr` could be more accurately written as `Ptr{T} where T`, meaning all values whose type is `Ptr{T}` for some value of `T`. 
例えば、 `Ptr`は` Ptr {T}ここでT`と書くことができます。これは `T`のある値に対して` Ptr {T} 'の値を持つすべての値を意味します。
#In this context, the parameter `T` is also often called a "type variable" since it is like a variable that ranges over types.
この文脈では、パラメータ「T」は、型にまたがる変数のようなものであるため、しばしば「型変数」と呼ばれます。
#Each `where` introduces a single type variable, so these expressions are nested for types with multiple parameters, for example `Array{T,N} where N where T`.
それぞれの `where`は単一の型変数を導入しているので、これらの式は複数のパラメータを持つ型に対して入れ子になっています。例えば` Array {T、N}、NはT`です。

#The type application syntax `A{B,C}` requires `A` to be a `UnionAll` type, and first substitutes `B` for the outermost type variable in `A`.
型のアプリケーション構文 `A {B、C}`は `A`が` UnionAll`型であることを要求し、最初に `A`の最も外側の型変数に` B`を代入します。
#The result is expected to be another `UnionAll` type, into which `C` is then substituted. So `A{B,C}` is equivalent to `A{B}{C}`.
結果は別の `UnionAll`型であると予想され、` C`が代入されます。だから、 `A {B、C}`は `A {B} {C}`と同じです。
#This explains why it is possible to partially instantiate a type, as in `Array{Float64}`: the first parameter value has been fixed, but the second still ranges over all possible values.
これは `Array {Float64}`のように、なぜ型を部分的にインスタンス化できるのかを説明します。最初のパラメータ値は固定されていますが、2番目の値はすべての値の範囲です。
#Using explicit `where` syntax, any subset of parameters can be fixed. For example, the type of all 1-dimensional arrays can be written as `Array{T,1} where T`.
明示的な `where`構文を使用すると、パラメータの任意のサブセットを修正できます。たとえば、すべての1次元配列の型は、 `Array {T、1} T 'と書くことができます。

#Type variables can be restricted with subtype relations.
型変数は下限と上限の両方を持つことができます。
#`Array{T} where T<:Integer` refers to all arrays whose element type is some kind of [`Integer`](@ref).
`Array {T}ここで、T <：Integer`は要素型が何らかの[` Integer`](@ ref)であるすべての配列を参照します。
#The syntax `Array{<:Integer}` is a convenient shorthand for `Array{T} where T<:Integer`.
#Type variables can have both lower and upper bounds.
タイプ変数は、サブタイプの関係で制限することができます。
#`Array{T} where Int<:T<:Number` refers to all arrays of [`Number`](@ref)s that are able to contain `Int`s (since `T` must be at least as big as `Int`).
`Array {<：Integer} 'の構文は` T <：Integer`の `Array {T}'の便利な省略形です。
#The syntax `where T>:Int` also works to specify only the lower bound of a type variable, and `Array{>:Int}` is equivalent to `Array{T} where T>:Int`.
`Int {：T <：Number}は` Int`を含むことができる( `T`は少なくとも以下のように大きくなければならないので) Int)。
`T>：Int`の構文は型変数の下限のみを指定するためにも働き、` Array {>：Int} `は` T>：Int`の `Array {T} 'に相当します。

#Since `where` expressions nest, type variable bounds can refer to outer type variables.
`where`式が入れ子になっているので、type変数boundsは外部変数を参照できます。
#For example `Tuple{T,Array{S}} where S<:AbstractArray{T} where T<:Real` refers to 2-tuples whose first element is some [`Real`](@ref), and whose second element is an `Array` of any kind of array whose element type contains the type of the first tuple element.
ここで、T <：Realは、最初の要素が何らかの[`Real`](@ ref)であり、2番目の要素が` `Real``である2つのタプルを参照しています。` Tuple {T、Array {S}要素型に最初のタプル要素の型が含まれている任意の種類の配列の `Array`です。

#The `where` keyword itself can be nested inside a more complex declaration. 
`where`キーワード自体は、より複雑な宣言の中にネストすることができます。
#For example, consider the two types created by the following declarations:
たとえば、次の宣言で作成される2つの型を考えてみましょう。

```jldoctest
julia> const T1 = Array{Array{T,1} where T, 1}
Array{Array{T,1} where T,1}

julia> const T2 = Array{Array{T,1}, 1} where T
Array{Array{T,1},1} where T
```

#Type `T1` defines a 1-dimensional array of 1-dimensional arrays; each of the inner arrays consists of objects of the same type, but this type may vary from one inner array to the next.
タイプ 'T1'は、1次元配列の1次元配列を定義する。 各内部配列は同じタイプのオブジェクトで構成されますが、このタイプは内部配列ごとに異なる場合があります。
#On the other hand, type `T2` defines a 1-dimensional array of 1-dimensional arrays all of whose inner arrays must have the same type.  
他方、タイプ「T2」は、内部配列がすべて同じ型でなければならない1次元配列の1次元配列を定義する。
#Note that `T2` is an abstract type, e.g., `Array{Array{Int,1},1} <: T2`, whereas `T1` is a concrete type. 
「T2」は抽象型であり、例えば、「Array {Array {Int、1}、1} <：T2」であり、「T1」は具体的な型であることに留意されたい。
#As a consequence, `T1` can be constructed with a zero-argument constructor `a=T1()` but `T2` cannot.
結果として、 `T1`は引数なしのコンストラクタ` a = T1() `で構築できますが、` T2`はできません。

#There is a convenient syntax for naming such types, similar to the short form of function definition syntax:
このような型の命名に便利な構文がありますが、これは短い形式の関数定義構文と似ています。

```julia
Vector{T} = Array{T,1}
```

#This is equivalent to `const Vector = Array{T,1} where T`.
これは `const Vector = Array {T、1} where T`と等価です。
#Writing `Vector{Float64}` is equivalent to writing `Array{Float64,1}`, and the umbrella type `Vector` has as instances all `Array` objects where the second parameter -- the number of array dimensions -- is 1, regardless of what the element type is. 
`Vector {Float64} 'の記述は` Array {Float64,1} `の記述と等価であり、傘型` Vector`はインスタンスとして `Array`オブジェクト全てを持ちます。 要素の種類に関係なく、
#In languages where parametric types must always be specified in full, this is not especially helpful, but in Julia, this allows one to write just `Vector` for the abstract type including all one-dimensional dense arrays of any element type.
パラメトリック型を常に完全に指定しなければならない言語では、これは特に有用ではありませんが、Juliaでは要素型のすべての1次元高密度配列を含む抽象型に `Vector`を書くことができます。

<!-- ## Type Aliases -->
##型エイリアス

#Sometimes it is convenient to introduce a new name for an already expressible type.
すでに表現可能なタイプの新しい名前を導入すると便利なことがあります。
#This can be done with a simple assignment statement.
これは簡単な代入文で行うことができます。
#For example, `UInt` is aliased to either [`UInt32`](@ref) or [`UInt64`](@ref) as is appropriate for the size of pointers on the system:
例えば、 `UInt`はシステム上のポインタの大きさに応じて[UInt32`](@ref)または[` UInt64`](@ref)にエイリアスされます：

```julia-repl
# 32-bit system:
julia> UInt
UInt32

# 64-bit system:
julia> UInt
UInt64
```

#This is accomplished via the following code in `base/boot.jl`:
これは、 `base / boot.jl`の次のコードで行います：

```julia
if Int === Int64
    const UInt = UInt64
else
    const UInt = UInt32
end
```

#Of course, this depends on what `Int` is aliased to -- but that is predefined to be the correct type -- either [`Int32`](@ref) or [`Int64`](@ref).
もちろん、これは `Int`がどのような別名になっているかに依存しますが、正しい型になるように[Int32`](@ ref)か[Int64`](@ref)のどちらかにあらかじめ定義されています。

#(Note that unlike `Int`, `Float` does not exist as a type alias for a specific sized [`AbstractFloat`](@ref). 
( `Int`とは異なり、` Float`は特定のサイズの[`AbstractFloat`](@ ref)の型エイリアスとして存在しません。
#Unlike with integer registers, the floating point register sizes are specified by the IEEE-754 standard. 
整数レジスタとは異なり、浮動小数点レジスタのサイズはIEEE-754規格で規定されています。
#Whereas the size of `Int` reflects the size of a native pointer on that machine.)
`Int`のサイズは、そのマシン上のネイティブポインタのサイズを反映します)。

### Operations on Types
##型の操作

#Since types in Julia are themselves objects, ordinary functions can operate on them. 
Juliaの型はそれ自体がオブジェクトなので、通常の関数がその上で動作することができます。
#Some functions that are particularly useful for working with or exploring types have already been introduced, such as the `<:` operator, which indicates whether its left hand operand is a subtype of its right hand operand.
左辺のオペランドが右手オペランドのサブタイプであるかどうかを示す `<：`演算子など、型の操作や探索に特に役立つ関数が既に導入されています。

#The [`isa`](@ref) function tests if an object is of a given type and returns true or false:
[`isa`](@ref)関数は、オブジェクトが与えられた型かどうかを調べ、trueまたはfalseを返します：

```jldoctest
julia> isa(1, Int)
true

julia> isa(1, AbstractFloat)
false
```

#The [`typeof()`](@ref) function, already used throughout the manual in examples, returns the type of its argument. 
例のマニュアル全体ですでに使用されている[`typeof()`](@ ref)関数は、引数の型を返します。
#Since, as noted above, types are objects, they also have types, and we can ask what their types are:
上記のように、型はオブジェクトであるため、型もあり、その型が何であるかを尋ねることができます：

```jldoctest
julia> typeof(Rational{Int})
DataType

julia> typeof(Union{Real,Float64,Rational})
DataType

julia> typeof(Union{Real,String})
Union
```

#What if we repeat the process? What is the type of a type of a type? As it happens, types are all composite values and thus all have a type of `DataType`:
プロセスを繰り返すとどうなりますか？ タイプの型の型は何ですか？ それが起こると、型はすべて複合値なので、すべて型の型が `DataType`です。

```jldoctest
julia> typeof(DataType)
DataType

julia> typeof(Union)
DataType
```

#`DataType` is its own type.
`DataType`はそれ自身の型です。

#Another operation that applies to some types is [`supertype()`](@ref), which reveals a type's supertype. 
いくつかの型に適用されるもう1つの操作は[`supertype()`](@ ref)です。これは型のスーパータイプを示します。
#Only declared types (`DataType`) have unambiguous supertypes:
宣言された型( `DataType`)のみが明白なスーパータイプを持っています：

```jldoctest
julia> supertype(Float64)
AbstractFloat

julia> supertype(Number)
Any

julia> supertype(AbstractString)
Any

julia> supertype(Any)
Any
```

#If you apply [`supertype()`](@ref) to other type objects (or non-type objects), a [`MethodError`](@ref) is raised:
他の型オブジェクト(または非型オブジェクト)に[`supertype()`](@ref)を適用すると、[`MethodError`](@ ref)が生成されます：

```jldoctest
julia> supertype(Union{Float64,Int64})
ERROR: MethodError: no method matching supertype(::Type{Union{Float64, Int64}})
Closest candidates are:
  supertype(!Matched::DataType) at operators.jl:41
  supertype(!Matched::UnionAll) at operators.jl:46
```

### Custom pretty-printing
##カスタムプリント

#Often, one wants to customize how instances of a type are displayed.  
多くの場合、型のインスタンスがどのように表示されるかをカスタマイズする必要があります。
#This is accomplished by overloading the [`show()`](@ref) function.  
これは、[`show()`](@ ref)関数のオーバーロードによって実現されます。
#For example, suppose we define a type to represent complex numbers in polar form:
たとえば、極座標形式で複素数を表す型を定義するとします。

```jldoctest polartype
julia> struct Polar{T<:Real} <: Number
           r::T
           Θ::T
       end

julia> Polar(r::Real,Θ::Real) = Polar(promote(r,Θ)...)
Polar
```

#Here, we've added a custom constructor function so that it can take arguments of different [`Real`](@ref) types and promote them to a common type (see [Constructors](@ref man-constructors) and [Conversion and Promotion](@ref conversion-and-promotion)).
ここでは、さまざまな[`Real`](@ ref)型の引数をとり、それらを共通の型に変換するカスタムコンストラクタ関数を追加しました([コンストラクタ](@ ref man-constructors)と[Conversion とプロモーション](@ refの変換とプロモーション))。

#(Of course, we would have to define lots of other methods, too, to make it act like a [`Number`](@ref), e.g. `+`, `*`, `one`, `zero`, promotion rules and so on.) 
(もちろん、 `+`、 `*`、 `one`、` zero`、promotionなど、 `` Number``(@ ref)のように動作させるために、他にもたくさんのメソッドを定義する必要があります ルールなど)。
#By default, instances of this type display rather simply, with information about the type name and the field values, as e.g. `Polar{Float64}(3.0,4.0)`.
デフォルトでは、このタイプのインスタンスは、タイプ名およびフィールド値に関する情報を、例えば、 `Polar {Float64}(3.0,4.0)`です。

#If we want it to display instead as `3.0 * exp(4.0im)`, we would define the following method to print the object to a given output object `io` (representing a file, terminal, buffer, etcetera; see [Networking and Streams](@ref)):
代わりに `3.0 * exp(4.0im)`で表示したい場合は、オブジェクトを出力オブジェクト `io`(ファイル、端末、バッファなどを表現する;ネットワーキング とストリーム](@ ref))：

```jldoctest polartype
julia> Base.show(io::IO, z::Polar) = print(io, z.r, " * exp(", z.Θ, "im)")
```
#More fine-grained control over display of `Polar` objects is possible. 
`Polar`オブジェクトの表示より細かい制御が可能です。
#In particular, sometimes one wants both a verbose multi-line printing format, used for displaying a single object in the REPL and other interactive environments, and also a more compact single-line format used for [`print()`](@ref) or for displaying the object as part of another object (e.g. in an array). 
特に、REPLや他のインタラクティブな環境で単一のオブジェクトを表示するために使用される冗長な複数行の印刷形式と、[print() `](@ ref )、またはオブジェクトを別のオブジェクトの一部として(たとえば配列内に)表示するために使用します。
#Although by default the `show(io, z)` function is called in both cases, you can define a *different* multi-line format for displaying an object by overloading a three-argument form of `show` that takes the `text/plain` MIME type as its second argument (see [Multimedia I/O](@ref)), for example:
デフォルトでは `show(io、z)`関数がどちらの場合でも呼び出されますが、 `show 'のオーバーロードによってオブジェクトを表示するための* different *複数行形式を定義することができます。 / plain "MIMEタイプを2番目の引数として使用します([Multimedia I / O](@ ref)参照)。

```jldoctest polartype
julia> Base.show(io::IO, ::MIME"text/plain", z::Polar{T}) where{T} =
           print(io, "Polar{$T} complex number:\n   ", z)
```

#(Note that `print(..., z)` here will call the 2-argument `show(io, z)` method.) This results in:
(ここで `print(...、z)`は2引数の `show(io、z)`メソッドを呼び出すことに注意してください)。

```jldoctest polartype
julia> Polar(3, 4.0)
Polar{Float64} complex number:
   3.0 * exp(4.0im)

julia> [Polar(3, 4.0), Polar(4.0,5.3)]
2-element Array{Polar{Float64},1}:
 3.0 * exp(4.0im)
 4.0 * exp(5.3im)
```

#where the single-line `show(io, z)` form is still used for an array of `Polar` values.   
`Polar`値の配列には一行の` show(io、z) `形式が使われています。
#Technically, the REPL calls `display(z)` to display the result of executing a line, which defaults to `show(STDOUT, MIME("text/plain"), z)`, which in turn defaults to `show(STDOUT, z)`, but you should *not* define new [`display()`](@ref) methods unless you are defining a new multimedia display handler (see [Multimedia I/O](@ref)).
技術的には、REPLは `show(STDOUT、MIME(" text / plain ")、z)`のデフォルトを `` show(STDOUT 新しいマルチメディアディスプレイハンドラ([Multimedia I / O](@ ref)を参照)を定義していない限り、新しい[`display()`](@ ref)メソッドを定義しないでください。


#Moreover, you can also define `show` methods for other MIME types in order to enable richer display (HTML, images, etcetera) of objects in environments that support this (e.g. IJulia).   
さらに、これをサポートする環境(例えば、IJulia)でオブジェクトのより豊かな表示(HTML、画像など)を可能にするために、他のMIMEタイプに対して `show`メソッドを定義することもできます。
#For example, we can define formatted HTML display of `Polar` objects, with superscripts and italics, via:
たとえば、次のようにして、上付き文字と斜体を使用して、「Polar」オブジェクトの書式付きHTML表示を定義できます。

```jldoctest polartype
julia> Base.show(io::IO, ::MIME"text/html", z::Polar{T}) where {T} =
           println(io, "<code>Polar{$T}</code> complex number: ",
                   z.r, " <i>e</i><sup>", z.Θ, " <i>i</i></sup>")
```

#A `Polar` object will then display automatically using HTML 0uin an environment that supports HTML display, but you can call `show` manually to get HTML output if you want:
`Polar`オブジェクトは、HTML表示をサポートする環境でHTML 0uinを使用して自動的に表示されますが、必要に応じて` show`を手動で呼び出してHTML出力を得ることができます：

```jldoctest polartype
julia> show(STDOUT, "text/html", Polar(3.0,4.0))
<code>Polar{Float64}</code> complex number: 3.0 <i>e</i><sup>4.0 <i>i</i></sup>
```

```@raw html
<p>An HTML renderer would display this as: <code>Polar{Float64}</code> complex number: 3.0 <i>e</i><sup>4.0 <i>i</i></sup></p>
```

<!-- # "Value types" -->
## "値のタイプ"

#In Julia, you can't dispatch on a *value* such as `true` or `false`. However, you can dispatch on parametric types, and Julia allows you to include "plain bits" values (Types, Symbols, Integers, floating-point numbers, tuples, etc.) as type parameters.  
Juliaでは、 `true`や` false`のような* value *にはディスパッチできません。 ただし、パラメトリック型でディスパッチすることはできますが、Juliaでは、型パラメータとして「プレーンビット」の値(型、シンボル、整数、浮動小数点数、タプルなど)を含めることができます。
#A common example is the dimensionality parameter in `Array{T,N}`, where `T` is a type (e.g., [`Float64`](@ref)) but `N` is just an `Int`.
一般的な例は、 `T {{Float64`}(@ ref))型であるが、` N`は単なる `Int`である` Array {T、N} `の次元数パラメータです。

#You can create your own custom types that take values as parameters, and use them to control dispatch of custom types. 
値をパラメータとして使用する独自のカスタム型を作成し、それらを使用してカスタム型のディスパッチを制御することができます。
#By way of illustration of this idea, let's introduce a parametric type, `Val{x}`, and a constructor `Val(x) = Val{x}()`, which serves as a customary way to exploit this technique for cases where you don't need a more elaborate hierarchy.
この考え方を説明するために、パラメータ型「Val {x}」とコンストラクタ「Val(x)= Val {x}()」を導入しましょう。 あなたはもっと精巧な階層を必要としません。

`Val`は次のように定義されます：
#`Val` is defined as:

```jldoctest valtype
julia> struct Val{x}
       end
Base.@pure Val(x) = Val{x}()
```

#There is no more to the implementation of `Val` than this.  
これよりも `Val`の実装はもうありません。
#Some functions in Julia's standard library accept `Val` instances as arguments, and you can also use it to write your own functions.
Juliaの標準ライブラリのいくつかの関数は `Val`インスタンスを引数として受け取ります。また、それを使って独自の関数を書くこともできます。
# For example:
  例えば：

```jldoctest valtype
julia> firstlast(::Val{true}) = "First"
firstlast (generic function with 1 method)

julia> firstlast(::Val{false}) = "Last"
firstlast (generic function with 2 methods)

julia> firstlast(Val(true))
"First"

julia> firstlast(Val(false))
"Last"
```

#For consistency across Julia, the call site should always pass a `Val`*instance* rather than using a *type*, i.e., use `foo(Val(:bar))` rather than `foo(Val{:bar})`.
Juliaの一貫性を保つために、コールサイトは、 `foo(Val {：bar})ではなく` foo(Val(：bar)) `を使用するのではなく、* type * `。

#It's worth noting that it's extremely easy to mis-use parametric "value" types, including `Val`; in unfavorable cases, you can easily end up making the performance of your code much *worse*. 
`Val`を含むパラメトリックな" value "型を誤って使用するのは非常に簡単だということは注目に値します。不利なケースでは、あなたのコードのパフォーマンスをもっと悪くすることは簡単に終わる可能性があります*。
#In particular, you would never want to write actual code as illustrated above.  For more information about the proper (and improper) uses of `Val`, please read the more extensive discussion in [the performance tips](@ref man-performance-tips).
特に、上記のように実際のコードを記述することは決してありません。 `Val`の適切な(そして不適切な)使い方の詳細については、[パフォーマンスのヒント](@ ref man-performance-tips)のより広範な議論を読んでください。

## [Nullable Types: Representing Missing Values](@id man-nullable-types)
## [Nullable型：欠損値を表す](@ idの人がNULL可能な型)

#In many settings, you need to interact with a value of type `T` that may or may not exist. 
多くの設定では、存在するかもしれないし、存在しないかもしれない型 `T`の値と対話する必要があります。
#To handle these settings, Julia provides a parametric type called [`Nullable{T}`](@ref), which can be thought of as a specialized container type that can contain either zero or one values. 
これらの設定を処理するために、Juliaは[Nullable {T} `(@ ref)というパラメトリック型を提供します。これは、0または1の値を含む特殊なコンテナ型と考えることができます。
#`Nullable{T}` provides a minimal interface designed to ensure that interactions with missing values are safe. 
`Nullable {T}`は、欠損値とのやりとりが安全であることを保証するための最小限のインターフェースを提供します。
#At present, the interface consists of several possible interactions:
現在、インタフェースはいくつかの相互作用から成り立っています。

  * 
  # Construct a `Nullable` object.
  `Nullable`オブジェクトを構築します。
  * 
  # Check if a `Nullable` object has a missing value.
   `Nullable`オブジェクトに欠損値があるかどうかを確認してください。
  * 
  # Access the value of a `Nullable` object with a guarantee that a [`NullException`](@ref) will be thrown if the object's value is missing.
   `Nullable`オブジェクトの値にアクセスすると、オブジェクトの値がない場合には` `NullException`(@ ref)がスローされることが保証されます。
  * 
  # Access the value of a `Nullable` object with a guarantee that a default value of type `T` will be returned if the object's value is missing.
   `Nullable`オブジェクトの値にアクセスするには、オブジェクトの値がない場合に` T`型のデフォルト値が返されることを保証します。
  * 
  # Perform an operation on the value (if it exists) of a `Nullable` object, getting a `Nullable` result. The result will be missing if the original value was missing.
   `Nullable`オブジェクトの値(存在する場合)に操作を実行し、` Nullable`結果を取得します。 元の値がない場合、結果は失われます。
  * 
  # Performing a test on the value (if it exists) of a `Nullable` object, getting a result that is missing if either the `Nullable` itself was missing, or the test failed.
   `Nullable`オブジェクトの値(存在する場合)のテストを行い、` Nullable`自体が存在しないか、テストが失敗した場合に欠けている結果を取得します。
  * 
  # Perform general operations on single `Nullable` objects, propagating the missing data.
   単一の `Nullable`オブジェクトに対して一般的な操作を行い、欠落したデータを伝播します。

### Constructing [`Nullable`](@ref) objects
### [`Nullable`](@ ref)オブジェクトの作成

#To construct an object representing a missing value of type `T`, use the `Nullable{T}()` function:
`T`型の欠損値を表すオブジェクトを構築するには、` Nullable {T}() `関数を使います：

```jldoctest
julia> x1 = Nullable{Int64}()
Nullable{Int64}()

julia> x2 = Nullable{Float64}()
Nullable{Float64}()

julia> x3 = Nullable{Vector{Int64}}()
Nullable{Array{Int64,1}}()
```

#To construct an object representing a non-missing value of type `T`, use the `Nullable(x::T)` function:
`T`型の欠損値を表現するオブジェクトを構築するには、` Nullable(x :: T) `関数を使います：

```jldoctest
julia> x1 = Nullable(1)
Nullable{Int64}(1)

julia> x2 = Nullable(1.0)
Nullable{Float64}(1.0)

julia> x3 = Nullable([1, 2, 3])
Nullable{Array{Int64,1}}([1, 2, 3])
```

#Note the core distinction between these two ways of constructing a `Nullable` object: in one style, you provide a type, `T`, as a function parameter; in the other style, you provide a single value of type `T` as an argument.
`Nullable`オブジェクトを構築するこれらの2つの方法の中核的な違いに注目してください。一つのスタイルでは、関数型として` T`型を提供します。 他のスタイルでは、引数として `T`型の単一の値を指定します。

### Checking if a `Nullable` object has a value
### `Nullable`オブジェクトに値があるかどうかを調べる

#You can check if a `Nullable` object has any value using [`isnull()`](@ref):
[`isnull()`](@ ref)を使って `Nullable`オブジェクトに値があるかどうかを調べることができます：

```jldoctest
julia> isnull(Nullable{Float64}())
true

julia> isnull(Nullable(0.0))
false
```

### Safely accessing the value of a `Nullable` object
###安全に `Nullable`オブジェクトの値にアクセスする

#You can safely access the value of a `Nullable` object using [`get()`](@ref):
`Nullable`オブジェクトの値へ[`get()`](@ ref) を使って安全にアクセスできます：

```jldoctest
julia> get(Nullable{Float64}())
ERROR: NullException()
Stacktrace:
 [1] get(::Nullable{Float64}) at ./nullable.jl:92

julia> get(Nullable(1.0))
1.0
```

#If the value is not present, as it would be for `Nullable{Float64}`, a [`NullException`](@ref) error will be thrown. 
値が存在しない場合、`Nullable {Float64}`のように、[`NullException`](@ref) エラーがスローされます。
#The error-throwing nature of the `get()` function ensures that any attempt to access a missing value immediately fails.
`get()`関数のエラースローの性質は、欠損値にアクセスしようとする試みが直ちに失敗することを保証します。

#In cases for which a reasonable default value exists that could be used when a `Nullable` object's value turns out to be missing, you can provide this default value as a second argument to `get()`:
`Nullable`オブジェクトの値が欠落していることが判明したときに使用できる合理的なデフォルト値が存在する場合、このデフォルト値を` get() `の第2引数として指定できます：

```jldoctest
julia> get(Nullable{Float64}(), 0.0)
0.0

julia> get(Nullable(1.0), 0.0)
1.0
```

!!! tip
   # Make sure the type of the default value passed to `get()` and that of the `Nullable` object match to avoid type instability, which could hurt performance. 
   型の不安定さを避けるために、 `get()`と `Nullable`オブジェクトのデフォルト値の型が一致していることを確認してください。
   # Use [`convert()`](@ref) manually if needed.
   必要に応じて手動で[`convert()`](@ ref)を手動で使用してください。

### Performing operations on `Nullable` objects
### `` Nullable``オブジェクトに対する操作の実行

#`Nullable` objects represent values that are possibly missing, and it is possible to write all code using these objects by first testing to see if the value is missing with [`isnull()`](@ref), and then doing an appropriate action. 
`Nullable`オブジェクトは失われている可能性のある値を表し、最初にテストすることでこれらのオブジェクトを使ってすべてのコードを書くことができます。値が[`isnull()`](@ref) で見つからない場合は、適切な処置を行ってください。
#However, there are some common use cases where the code could be more concise or clear by using a higher-order function.
しかし、高次関数を使用してコードをより簡潔または明確にすることができる一般的な使用例がいくつかあります。

#The [`map`](@ref) function takes as arguments a function `f` and a `Nullable` value `x`. It produces a `Nullable`:
[`map`](@ ref)関数は、引数として関数` f`と `Nullable`値` x`をとります。 `Nullable`を生成します：

   # - If `x` is a missing value, then it produces a missing value;
   - `x`が欠損値であれば、欠損値を生成します。
   # - If `x` has a value, then it produces a `Nullable` containing `f(get(x))` as value.
   - `x`が値を持つ場合、値として` f(get(x)) `を含む` Nullable`を生成します。

#This is useful for performing simple operations on values that might be missing if the desired behaviour is to simply propagate the missing values forward.
これは、欠落している値を単純に伝播することが望ましい場合に、欠落している可能性のある値に対して単純な操作を実行する場合に便利です。

#The [`filter`](@ref) function takes as arguments a predicate function `p` (that is, a function returning a boolean) and a `Nullable` value `x`.
[`filter`](@ref)関数は、述語関数` p`(すなわちブール値を返す関数)と `Nullable`値` x`を引数としてとります。
#It produces a `Nullable` value:
それは `Nullable`値を生成します：

   #- If `x` is a missing value, then it produces a missing value;
   - `x`が欠損値であれば、欠損値を生成します。
   #- If `p(get(x))` is true, then it produces the original value `x`;
   - `p(get(x))`が真であれば、元の値 `x`を生成します。
   #- If `p(get(x))` is false, then it produces a missing value.
   - `p(get(x))`が偽なら、欠損値を生成する。

#In this way, `filter` can be thought of as selecting only allowable values, and converting non-allowable values to missing values.
このように、`filter`は許容値のみを選択し、許容できない値を欠損値に変換するものと考えることができます。

#While `map` and `filter` are useful in specific cases, by far the most useful higher-order function is [`broadcast`](@ref), which can handle a wide variety of cases, including making existing operations work and propagate `Nullable`s. 
特定のケースでは `map`と` filter`が便利ですが、最も有用な高階関数は[`broadcast`](@ref)です。これは既存の操作を含む幅広いケースを扱うこや、`Nullable`を伝播することができます。
#An example will motivate the need for `broadcast`. 
一例が「放送」の必要性を促すだろう。
#Suppose we have a function that computes the greater of two real roots of a quadratic equation, using the quadratic formula:
2次方程式の2つの実数のうち大きい方を計算する関数があるとします。

```jldoctest nullableroot
julia> root(a::Real, b::Real, c::Real) = (-b + √(b^2 - 4a*c)) / 2a
root (generic function with 1 method)
```

#We may verify that the result of `root(1, -9, 20)` is `5.0`, as we expect,
`root(1、-9、20)`の結果が `5.0`であることを検証するかもしれませんが、

#since `5.0` is the greater of two real roots of the quadratic equation.
「5.0」は2次方程式の2つの実根のうち大きい方であるからである。

#Suppose now that we want to find the greatest real root of a quadratic equations where the coefficients might be missing values. 
係数が欠損している可能性がある2次方程式の最大の実数根を見つけたいと思ったとします。
#Having missing values in datasets is a common occurrence in real-world data, and so it is important to be able to deal with them. 
データセットに欠損値があることは、現実のデータに共通する事象であり、対処することが重要です。
#But we cannot find the roots of an equation if we do not know all the coefficients. The best solution to this will depend on the particular use case; perhaps we should throw an error. 
しかし、すべての係数を知らなければ、方程式の根を見つけることはできません。 これに対する最良のソリューションは、特定のユースケースに依存します。 恐らくエラーを投げるべきです。
#However, for this example, we will assume that the best solution is to propagate the missing values forward; that is, if any input is missing, we simply produce a missing output.
ただし、この例では、欠落している値を前方に伝播することが最善の解決策であると仮定します。 つまり、入力が欠落している場合は、単に欠落した出力が生成されます。

#The `broadcast()` function makes this task easy; we can simply pass the `root` function we wrote to `broadcast`:
`broadcast()`関数はこの作業を容易にします。 私たちが書き込んだ `ルート`関数を単に `broadcast`に渡すことができます：

```jldoctest nullableroot
julia> broadcast(root, Nullable(1), Nullable(-9), Nullable(20))
Nullable{Float64}(5.0)

julia> broadcast(root, Nullable(1), Nullable{Int}(), Nullable{Int}())
Nullable{Float64}()

julia> broadcast(root, Nullable{Int}(), Nullable(-9), Nullable(20))
Nullable{Float64}()
```

<!--- If one or more of the inputs is missing, then the output of `broadcast()` will be missing. --->
1つまたは複数の入力がない場合、 `broadcast()`の出力が欠落します。

<!--- There exists special syntactic sugar for the `broadcast()` function using a dot notation: --->
ドット表記法を使う `broadcast()`関数のための特別な糖衣構文が存在します：

```jldoctest nullableroot
julia> root.(Nullable(1), Nullable(-9), Nullable(20))
Nullable{Float64}(5.0)
```

<!--- In particular, the regular arithmetic operators can be `broadcast()` conveniently using `.`-prefixed operators: --->
特に、通常の算術演算子は接頭演算子`.`を使って` broadcast() `にすることができます：

```jldoctest
julia> Nullable(2) ./ Nullable(3) .+ Nullable(1.0)
Nullable{Float64}(1.66667)
```
