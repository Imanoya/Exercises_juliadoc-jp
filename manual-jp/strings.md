<!--- > [Strings](@id man-strings) -->
# [文字列](@id man-strings)


<!--- Strings are finite sequences of characters.  -->
Stringsは有限の文字列です。
<!--- Of course, the real trouble comes when one asks what a character is.  -->
もちろん、実際の問題は、キャラクターが何であるかを尋ねるときに来ます。
<!--- The characters that English speakers are familiar with are the letters `A`, `B`, `C`, etc., together with numerals and common punctuation symbols.  -->
英語の話者がよく知っている文字は、数字と共通の句読記号とともに、文字`A`、`B`、`C`などです。
<!--- These characters are standardized together with a mapping to integer values between 0 and 127 by the [ASCII](https://en.wikipedia.org/wiki/ASCII) standard.  -->
これらの文字は、[ASCII](https://en.wikipedia.org/wiki/ASCII)標準で0〜127の整数値へのマッピングと共に標準化されています。
<!--- There are, of course, many other characters used in non-English languages, including variants of the ASCII characters with accents and other modifications, related scripts such as Cyrillic and Greek, and scripts completely unrelated to ASCII and English, including Arabic, Chinese, Hebrew, Hindi, Japanese, and Korean.  -->
もちろん、英語以外の言語では、アクセントやその他の変更を加えたASCII文字の変形、キリル文字やギリシャ語などの関連スクリプト、アラビア語、中国語、韓国語などのASCIIと英語とは完全に無関係のスクリプトなど、ヘブライ語、ヒンディー語、日本語、韓国語です。
<!--- The [Unicode](https://en.wikipedia.org/wiki/Unicode) standard tackles the complexities of what exactly a character is, and is generally accepted as the definitive standard addressing this problem.  -->
[Unicode](https://en.wikipedia.org/wiki/Unicode)標準は、正確に文字が何であるかの複雑さに取り組んでおり、この問題に対処する決定的な標準として一般に受け入れられています。
<!--- Depending on your needs, you can either ignore these complexities entirely and just pretend that only ASCII characters exist, or you can write code that can handle any of the characters or encodings that one may encounter when handling non-ASCII text.  -->
必要に応じて、これらの複雑さを完全に無視して、ASCII文字だけが存在するかのように見せたり、非ASCIIテキストを扱うときに遭遇する可能性のある文字やエンコーディングを処理できるコードを書くことができます。
<!--- Julia makes dealing with plain ASCII text simple and efficient, and handling Unicode is as simple and efficient as possible.  -->
Juliaは簡単なASCIIテキストを扱うことをシンプルで効率的に行い、Unicodeの処理は可能な限り単純で効率的です。
<!--- In particular, you can write C-style string code to process ASCII strings, and they will work as expected, both in terms of performance and semantics.  -->
特に、ASCII文字列を処理するCスタイルの文字列コードを記述することができます。これらは、パフォーマンスとセマンティクスの両方で期待どおりに動作します。
<!--- If such code encounters non-ASCII text, it will gracefully fail with a clear error message, rather than silently introducing corrupt results.  -->
このようなコードで非ASCIIテキストが検出された場合、破損した結果を静かに挿入するのではなく、明確なエラーメッセージで正常に処理されます。
<!--- When this happens, modifying the code to handle non-ASCII data is straightforward. -->
この場合、非ASCIIデータを処理するコードを変更するのは簡単です。

<!--- There are a few noteworthy high-level features about Julia's strings: -->
Juliaの文字列に関する注目すべき高度な機能がいくつかあります。

   <!--- * The built-in concrete type used for strings (and string literals) in Julia is [`String`](@ref). -->
   * Juliaの文字列(および文字列リテラル)に使用される組み込みの具象型は[`String`](@ref)です。
    <!--- * This supports the full range of [Unicode](https://en.wikipedia.org/wiki/Unicode) characters via the [UTF-8](https://en.wikipedia.org/wiki/UTF-8) encoding.  -->
   * これは、[UTF-8](https://en.wikipedia.org/wiki/UTF-8)エンコーディングを介し[UNICODE](https://en.wikipedia.org/wiki/Unicode)文字の完全な範囲をサポート。
    (A [`transcode()`](@ref) function is provided to convert to/from other Unicode encodings.)
    他のUnicodeエンコーディングとの間で変換を行うためには、A [`transcode()`](@ref)関数が提供されています。
   * 
#    # All string types are subtypes of the abstract type `AbstractString`, and external packages define additional `AbstractString` subtypes (e.g. for other encodings).  
    すべての文字列型は抽象型 `AbstractString`のサブタイプであり、外部のパッケージは、(例えば、他のエンコーディングのための)追加の` AbstractString`サブタイプを定義します。
#    If you define a function expecting a string argument, you should declare the type as `AbstractString` in order to accept any string type.
    文字列引数を期待する関数を定義する場合は、文字列型を受け入れるために型を `AbstractString`として宣言する必要があります。
   * 
#   Like C and Java, but unlike most dynamic languages, Julia has a first-class type representing a single character, called `Char`. 
    CやJavaのような、ほとんどの動的言語とは異なり、ジュリアは `Char`と呼ばれる単一の文字を表すファーストクラスのタイプがあります。
#   This is just a special kind of 32-bit primitive type whose numeric value represents a Unicode code point.
    これは単なる特殊な種類の32ビットプリミティブ型であり、その数値はUnicodeコードポイントを表します。
   * 
#   As in Java, strings are immutable: the value of an `AbstractString` object cannot be changed.
    Javaの場合と同様に、文字列は不変です： `AbstractString`オブジェクトの値は変更できません。
#   To construct a different string value, you construct a new string from parts of other strings.
    異なる文字列値を作成するには、他の文字列の一部から新しい文字列を作成します。
   * 
#    Conceptually, a string is a *partial function* from indices to characters: for some index values, no character value is returned, and instead an exception is thrown. 
    概念的には、文字列がインデックスから文字に* *一部の機能である：いくつかのインデックス値のために、何の文字の値が返されず、代わりに例外がスローされます。
    This allows for efficient indexing into strings by the byte index of an encoded representation rather than by a character index, which cannot be implemented both efficiently and simply for variable-width encodings of Unicode strings.
    これは、符号化された表現のバイト・インデックスではなく、Unicode文字列の可変幅エンコーディングの両方を効率的かつ簡単に実現することができない文字インデックス、によって文字列に効率的な索引付けを可能にします。

## [Characters](@id man-characters)

#A `Char` value represents a single character: it is just a 32-bit primitive type with a special literal representation and appropriate arithmetic behaviors, whose numeric value is interpreted as a [Unicode code point](https://en.wikipedia.org/wiki/Code_point). 
`Char`値は単一の文字を表します：特殊なリテラル表現と適切な算術的振る舞いを持つ単なる32ビットのプリミティブ型です。その数値は[Unicode code point](https://en.wikipedia.org/wiki/Code_point)として解釈されます。
#Here is how `Char` values are input and shown:
`Char`値がどのように入力されて表示されるかは次のとおりです：

```jldoctest
julia> 'x'
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)

julia> typeof(ans)
Char
```

<!--- You can convert a `Char` to its integer value, i.e. code point, easily: -->
`Char`をその整数値、すなわちコードポイントに簡単に変換することができます：

```jldoctest
julia> Int('x')
120

julia> typeof(ans)
Int64
```

<!--- On 32-bit architectures, [`typeof(ans)`](@ref) will be [`Int32`](@ref).  -->
32ビットアーキテクチャでは、[`typeof(and)`](@ref)は[`Int32`](@ref)になります。
<!--- You can convert an integer value back to a `Char` just as easily: -->
整数値を簡単に `Char`に変換することができます：

```jldoctest
julia> Char(120)
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)
```

<!--- Not all integer values are valid Unicode code points, but for performance, the `Char()` conversion does not check that every character value is valid.  -->
すべての整数値が有効なUnicodeコードポイントであるわけではありませんが、パフォーマンスのために、 `Char()`変換はすべての文字値が有効であるかどうかをチェックしません。
<!--- If you want to check that each converted value is a valid code point, use the [`isvalid()`](@ref) function: -->
変換された各値が有効なコードポイントであることを確認するには、[`isvalid()`](@ref)関数を使用します：

```jldoctest
julia> Char(0x110000)
'\U110000': Unicode U+110000 (category Cn: Other, not assigned)

julia> isvalid(Char, 0x110000)
false
```

<!--- As of this writing, the valid Unicode code points are `U+00` through `U+d7ff` and `U+e000` through `U+10ffff`.  -->
この執筆時点で、有効なUnicodeコードポイントは、`U+00`から`U+d7ff`、`U+e000`から`U+10ffff`までです。
#These have not all been assigned intelligible meanings yet, nor are they necessarily interpretable by applications, but all of these values are considered to be valid Unicode characters.
これらはすべて理解可能な意味がまだ割り当てられておらず、必ずしもアプリケーションによって解釈可能ではありませんが、これらの値はすべて有効なUnicode文字と見なされます。
<!--- You can input any Unicode character in single quotes using `\u` followed by up to four hexadecimal digits or `\U` followed by up to eight hexadecimal digits (the longest valid value only requires six): -->
`\u`とそれに続く4つの16進数字、`\U`の後ろに8つまでの16進数字(最長の有効な値は6つだけ必要です)を使用して、任意のUnicode文字を一重引用符で入力できます。

```jldoctest
julia> '\u0'
'\0': ASCII/Unicode U+0000 (category Cc: Other, control)

julia> '\u78'
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)

julia> '\u2200'
'∀': Unicode U+2200 (category Sm: Symbol, math)

julia> '\U10ffff'
'\U10ffff': Unicode U+10ffff (category Cn: Other, not assigned)
```

<!--- Julia uses your system's locale and language settings to determine which characters can be printed as-is and which must be output using the generic, escaped `\u` or `\U` input forms.  -->
Juliaはあなたのシステムのロケールと言語設定を使って、どんな文字がそのまま現れているのか、そしてエスケープされた `\u`や` \U`という入力フォームを使って出力する必要があるのかを判断します。
<!--- In addition to these Unicode escape forms, all of [C's traditional escaped input forms](https://en.wikipedia.org/wiki/C_syntax#Backslash_escapes) can also be used: -->
これらのUnicodeエスケープフォームに加えて、[Cの伝統的なエスケープ入力フォーム](https://en.wikipedia.org/wiki/C_syntax#Backslash_escapes)も使用できます：

```jldoctest
julia> Int('\0')
0

julia> Int('\t')
9

julia> Int('\n')
10

julia> Int('\e')
27

julia> Int('\x7f')
127

julia> Int('\177')
127

julia> Int('\xff')
255
```

<!--- You can do comparisons and a limited amount of arithmetic with `Char` values: -->
`Char`の値で比較と限られた量の算術演算を行うことができます：

```jldoctest
julia> 'A' < 'a'
true

julia> 'A' <= 'a' <= 'Z'
false

julia> 'A' <= 'X' <= 'Z'
true

julia> 'x' - 'a'
23

julia> 'A' + 1
'B': ASCII/Unicode U+0042 (category Lu: Letter, uppercase)
```
<!--- # String Basics -->
##文字列の基本

<!--- String literals are delimited by double quotes or triple double quotes: -->
文字列リテラルは、二重引用符または三重二重引用符で区切られます。

```jldoctest helloworldstring
julia> str = "Hello, world.\n"
"Hello, world.\n"

julia> """Contains "quote" characters"""
"Contains \"quote\" characters"
```

<!--- If you want to extract a character from a string, you index into it: -->
文字列から文字を抽出する場合は、その文字列にインデックスを付けます。

```jldoctest helloworldstring
julia> str[1]
'H': ASCII/Unicode U+0048 (category Lu: Letter, uppercase)

julia> str[6]
',': ASCII/Unicode U+002c (category Po: Punctuation, other)

julia> str[end]
'\n': ASCII/Unicode U+000a (category Cc: Other, control)
```

<!--- All indexing in Julia is 1-based: the first element of any integer-indexed object is found at index 1.  -->
Juliaでのすべての索引付けは1から初まります。整数索引オブジェクトの最初の要素は索引1にあります。
<!--- (As we will see below, this does not necessarily mean that the last element is found at index `n`, where `n` is the length of the string.) -->
(以下で説明するように、必ずしも最後の要素がインデックス `n`にあることを意味するわけではありません。ここで`n`は文字列の長さです。)

<!--- In any indexing expression, the keyword `end` can be used as a shorthand for the last index (computed by [`endof(str)`](@ref)).  -->
どのインデックス式でも、キーワード `end`は最後のインデックス([endof(str)`](@ref)で算出されます)の短縮形として使用できます。
<!--- You can perform arithmetic and other operations with `end`, just like a normal value: -->
通常の値と同様に、算術演算やその他の演算を `end`で実行することができます：

```jldoctest helloworldstring
julia> str[end-1]
'.': ASCII/Unicode U+002e (category Po: Punctuation, other)

julia> str[end÷2]
' ': ASCII/Unicode U+0020 (category Zs: Separator, space)
```

<!--- Using an index less than 1 or greater than `end` raises an error: -->
1より小さいまたは `end`より大きいインデックスを使用すると、エラーが発生します。

```jldoctest helloworldstring
julia> str[0]
ERROR: BoundsError: attempt to access "Hello, world.\n"
  at index [0]
[...]

julia> str[end+1]
ERROR: BoundsError: attempt to access "Hello, world.\n"
  at index [15]
[...]
```

<!--- You can also extract a substring using range indexing: -->
範囲の索引付けを使用して部分文字列を抽出することもできます。

```jldoctest helloworldstring
julia> str[4:9]
"lo, wo"
```

<!--- Notice that the expressions `str[k]` and `str[k:k]` do not give the same result: -->
`str [k]`と `str [k：k]`という式は同じ結果を与えないことに注意してください:

```jldoctest helloworldstring
julia> str[6]
',': ASCII/Unicode U+002c (category Po: Punctuation, other)

julia> str[6:6]
","
```

<!--- The former is a single character value of type `Char`, while the latter is a string value that happens to contain only a single character.  -->
前者は `Char`型の単一文字値であり、後者は単一の文字のみを含む文字列値です。
<!--- In Julia these are very different things. -->
ジュリアでは、これらは非常に異なるものです。

<!--- # Unicode and UTF-8 -->
## UnicodeとUTF-8

<!--- Julia fully supports Unicode characters and strings.  -->
Juliaは、Unicode文字と文字列を完全にサポートしています。
<!--- As [discussed above](@ref man-characters), in character literals, Unicode code points can be represented using Unicode `\u` and `\U` escape sequences, as well as all the standard C escape sequences.  -->
UnicodeコードポイントはUnicodeの `\u`と`\U`エスケープシーケンスとすべての標準Cエスケープシーケンスを使って表すことができます(@ref man-characters)。
<!--- These can likewise be used to write string literals: -->
これらも同様に文字列リテラルを書くために使うことができます：

```jldoctest unicodestring
julia> s = "\u2200 x \u2203 y"
"∀ x ∃ y"
```

<!--- Whether these Unicode characters are displayed as escapes or shown as special characters depends on your terminal's locale settings and its support for Unicode.  -->
これらのUnicode文字がエスケープとして表示されるか、特殊文字として表示されるかは、端末のロケール設定とUnicodeのサポートによって異なります。
<!--- String literals are encoded using the UTF-8 encoding.  -->
文字列リテラルは、UTF-8エンコーディングを使用してエンコードされます。
<!--- UTF-8 is a variable-width encoding, meaning that not all characters are encoded in the same number of bytes.  -->
UTF-8は可変幅のエンコーディングです。つまり、すべての文字が同じバイト数でエンコードされているわけではありません。
<!--- In UTF-8, ASCII characters -- i.e. those with code points less than 0x80 (128) -- are encoded as they are in ASCII, using a single byte, while code points 0x80 and above are encoded using multiple bytes -- up to four per character.  -->
UTF-8では、ASCII文字(つまり、0x80(128)未満のコードポイントを持つもの)は、1バイトを使用してASCII形式でエンコードされ、0x80以上のコードポイントは複数のバイトを使用してエンコードされます 1文字につき4つ。
<!--- This means that not every byte index into a UTF-8 string is necessarily a valid index for a character.  -->
つまり、UTF-8文字列の各バイトインデックスが必ずしも文字の有効なインデックスであるとは限りません。
<!--- If you index into a string at such an invalid byte index, an error is thrown: -->
このような無効なバイトインデックスで文字列にインデックスを付けると、エラーがスローされます。

```jldoctest unicodestring
julia> s[1]
'∀': Unicode U+2200 (category Sm: Symbol, math)

julia> s[2]
ERROR: UnicodeError: invalid character index
[...]

julia> s[3]
ERROR: UnicodeError: invalid character index
[...]

julia> s[4]
' ': ASCII/Unicode U+0020 (category Zs: Separator, space)
```

<!--- In this case, the character `∀` is a three-byte character, so the indices 2 and 3 are invalid and the next character's index is 4; this next valid index can be computed by [`nextind(s,1)`](@ref), and the next index after that by `nextind(s,4)` and so on. -->
この場合、文字 `∀`は3バイトの文字なので、インデックス2と3は無効で、次の文字のインデックスは4です。 この次の有効なインデックスは、 `nextind(s、1)`](@ref)とそれに続く `nextind(s4)`などの次のインデックスによって計算できます。

<!--- Because of variable-length encodings, the number of characters in a string (given by [`length(s)`](@ref)) is not always the same as the last index.  -->
可変長エンコーディングのため、文字列([`length(s)`](@ref)で与えられる)の文字数は最後のインデックスと必ずしも同じではありません。
<!--- If you iterate through the indices 1 through [`endof(s)`](@ref) and index into `s`, the sequence of characters returned when errors aren't thrown is the sequence of characters comprising the string `s`.  -->
インデックス1〜[`endof(s)`](@ref)を繰り返し、`s`にインデックスを付けると、エラーがスローされないときに返される文字のシーケンスは、文字列`s`を含む文字のシーケンスです。
<!--- Thus we have the identity that `length(s) <= endof(s)`, since each character in a string must have its own index.  -->
したがって、文字列の各文字には独自のインデックスが必要であるため、 `length(s)<= endof(s) 'というアイデンティティーがあります。
<!--- The following is an inefficient and verbose way to iterate through the characters of `s`: -->
以下は、`s`の文字を反復する非効率的で冗長な方法です：

```jldoctest unicodestring
julia> for i = 1:endof(s)
           try
               println(s[i])
           catch
               # ignore the index error
           end
       end
∀

x

∃

y
```

<!--- The blank lines actually have spaces on them.  -->
空行には実際にスペースがあります。
<!--- Fortunately, the above awkward idiom is unnecessary for iterating through the characters in a string, since you can just use the string as an iterable object, no exception handling required: -->
幸いにも、上記の厄介なイディオムは文字列内の文字を反復処理するためには不要です。 文字列を反復可能なオブジェクトとして使用するだけで、例外処理は必要ありません。

```jldoctest unicodestring
julia> for c in s
           println(c)
       end
∀

x

∃

y
```

<!--- Julia uses the UTF-8 encoding by default, and support for new encodings can be added by packages. -->
JuliaはデフォルトでUTF-8エンコーディングを使用し、新しいエンコーディングのサポートはパッケージで追加できます。
<!--- For example, the [LegacyStrings.jl](https://github.com/JuliaArchive/LegacyStrings.jl) package implements `UTF16String` and `UTF32String` types.  -->
例えば、[LegacyStrings.jl](https://github.com/JuliaArchive/LegacyStrings.jl)パッケージは `UTF16String`と` UTF32String`タイプを実装しています。
<!--- Additional discussion of other encodings and how to implement support for them is beyond the scope of this document for the time being.  -->
他のエンコーディングについての追加の議論、およびそれらのサポートを実装する方法については、このドキュメントの対象外です。
<!--- For further discussion of UTF-8 encoding issues, see the section below on [byte array literals](@ref man-byte-array-literals). -->
UTF-8エンコーディングの問題の詳細については、以下の[バイト配列リテラル](@ref man-byte-array-literals)のセクションを参照してください。
<!--- The [`transcode()`](@ref) function is provided to convert data between the various UTF-xx encodings, primarily for working with external data and libraries. -->
主に外部データやライブラリを扱うために、さまざまなUTF-xxエンコーディング間でデータを変換するための[`transcode()`](@ref)関数が提供されています。

<!--- ## Concatenation -->
##連結

<!--- One of the most common and useful string operations is concatenation: -->
最も一般的で便利な文字列操作の1つは連結です。


```jldoctest stringconcat
julia> greet = "Hello"
"Hello"

julia> whom = "world"
"world"

julia> string(greet, ", ", whom, ".\n")
"Hello, world.\n"
```

<!--- Julia also provides `*` for string concatenation: -->
Juliaは文字列の連結に `*`も提供しています：

```jldoctest stringconcat
julia> greet * ", " * whom * ".\n"
"Hello, world.\n"
```

<!--- While `*` may seem like a surprising choice to users of languages that provide `+` for string concatenation, this use of `*` has precedent in mathematics, particularly in abstract algebra. -->
`*`は文字列連結のために `+`を提供する言語のユーザにとって驚くべき選択のように思えるかもしれません, `* 'のこの使用法は、数学、特に抽象代数において先例があります。

<!--- In mathematics, `+` usually denotes a *commutative* operation, where the order of the operands does not matter.  -->
数学では、`+`は通常、*可換*操作を表します。ここでは、オペランドの順序は関係ありません。
<!--- An example of this is matrix addition, where `A + B == B + A` for any matrices `A` and `B` that have the same shape.  -->
これの一例は、同じ形状を有する任意の行列`A`+`B`に対する行列加算であり、ここで、`A + B == B + A`です。
<!--- In contrast, `*` typically denotes a *noncommutative* operation, where the order of the operands *does* matter.  -->
対照的に`*`は通常、オペランドの順序が重要な *非可換* 演算を表します。
<!--- An example of this is matrix multiplication, where in general `A * B != B * A`.  -->
行列の乗算はこれの一例であり、一般に「A * B！= B * A」です。
<!--- As with matrix multiplication, string concatenation is noncommutative: `greet * whom != whom * greet`.  -->
行列の乗算と同様に、文字列の連結は非可換です。 `greet * whom != whom * greet`。
<!--- As such, `*` is a more natural choice for an infix string concatenation operator, consistent with common mathematical use. -->
そのため、 `*`は一般的な数学的使用と一致して、インフィックス文字列連結演算子にとってより自然な選択です。

#More precisely, the set of all finite-length strings *S* together with the string concatenation operator `*` forms a [free monoid](https://en.wikipedia.org/wiki/Free_monoid) (*S*, `*`). 
より正確には、すべての有限長の文字列のセットは* `` *文字列連結演算子 `と共に* S [自由モノイド(https://en.wikipedia.org/wiki/Free_monoid)(*S*,す* `)。
<!--- The identity element of this set is the empty string, `""`.  -->
`""`このセットのidentity要素は、空の文字列 です。
<!--- Whenever a free monoid is not commutative, the operation is typically represented as `\cdot`, `*`, or a similar symbol, rather than `+`, which as stated usually implies commutativity. -->
自由モノイドが可換でないときはいつでも、操作は一般的に述べたように、通常は可換性を意味し、むしろ `+`より `\cdot`、`*`、または類似の記号、として表されます。

<!--- # [Interpolation](@id string-interpolation) -->
## [補間](@ id文字列補間)


<!--- Constructing strings using concatenation can become a bit cumbersome, however.  -->
ただし、連結を使用して文字列を作成するのは面倒です。
<!--- To reduce the need for these verbose calls to [`string()`](@ref) or repeated multiplications, Julia allows interpolation into string literals using `$`, as in Perl: -->
Juliaは、[`string()`](@ref)へのこれらの冗長呼び出しの必要性を減らすために、Perlのように `$`を使って文字列リテラルに補間することができます。

```jldoctest stringconcat
julia> "$greet, $whom.\n"
"Hello, world.\n"
```

<!--- This is more readable and convenient and equivalent to the above string concatenation -- the system rewrites this apparent single string literal into a concatenation of string literals with variables. -->
これは、読みやすく便利で、上記の文字列連結と同等です。システムは、この明らかな単一文字列リテラルを、文字列リテラルと変数との連結に書き換えます。

<!--- The shortest complete expression after the `$` is taken as the expression whose value is to be interpolated into the string.  -->
`$`の後の最短の完全な式は、その値が文字列に補間される式とみなされます。
<!--- Thus, you can interpolate any expression into a string using parentheses: -->
したがって、カッコを使用して任意の式を文字列に補間できます。

```jldoctest
julia> "1 + 2 = $(1 + 2)"
"1 + 2 = 3"
```

<!--- Both concatenation and string interpolation call [`string()`](@ref) to convert objects into string form.  -->
連結と文字列補間の両方は、オブジェクトを文字列形式に変換するために[`string()`](@ref)を呼び出します。
<!--- Most non-`AbstractString` objects are converted to strings closely corresponding to how they are entered as literal expressions: -->
ほとんどの非 `AbstractString`オブジェクトは、リテラル式としてどのように入力されるかに密接に対応する文字列に変換されます：

```jldoctest
julia> v = [1,2,3]
3-element Array{Int64,1}:
 1
 2
 3

julia> "v: $v"
"v: [1, 2, 3]"
```

#[`string()`](@ref) is the identity for `AbstractString` and `Char` values, so these are interpolated into strings as themselves, unquoted and unescaped:
[`string()`](@ref)は `Abstract String`と` Char`の値のアイデンティティですので、引用符で囲まれていないものとアンエスケープされていないものとして文字列に補間されます：

```jldoctest
julia> c = 'x'
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)

julia> "hi, $c"
"hi, x"
```

#To include a literal `$` in a string literal, escape it with a backslash:
リテラル `$`を文字列リテラルに含めるには、バックスラッシュでエスケープします。

```jldoctest
julia> print("I have \$100 in my account.\n")
I have $100 in my account.
```

<!--- # Triple-Quoted String Literals -->
##トリプルクォート文字列リテラル

<!--- When strings are created using triple-quotes (`"""..."""`) they have some special behavior that can be useful for creating longer blocks of text.  -->
文字列が三重引用符( `"""..."""`)を使用して作成されると、長いテキストブロックを作成するのに便利な特殊な動作があります。
<!--- First, if the opening `"""` is followed by a newline, the newline is stripped from the resulting string. -->
まず、 `` "" `の後に改行が続く場合、改行は結果の文字列から取り除かれます。

```julia
"""hello"""
```

is equivalent to

```julia
"""
hello"""
```

but

```julia
"""

hello"""
```

<!--- will contain a literal newline at the beginning.  -->
最初にリテラル改行を含みます。
<!--- Trailing whitespace is left unaltered.  -->
末尾の空白は変更されません。
<!--- They can contain `"` symbols without escaping.  -->
それらはエスケープしないで `"`のシンボルを含むことができます。
<!--- Triple-quoted strings are also dedented to the level of the least-indented line.  -->
三重引用符で囲まれた文字列は、最もインデントのない行のレベルにも割り当てられます。
<!--- This is useful for defining strings within code that is indented.  -->
インデントされたコード内の文字列を定義するのに便利です。
<!--- For example: -->
例えば：

```jldoctest
julia> str = """
           Hello,
           world.
         """
"  Hello,\n  world.\n"
```

<!--- In this case the final (empty) line before the closing `"""` sets the indentation level. -->
この場合、閉じる前の最後の(空の)行はインデントレベルを設定します。

#Note that line breaks in literal strings, whether single- or triple-quoted, result in a newline (LF) character `\n` in the string, even if your editor uses a carriage return `\r` (CR) or CRLF combination to end lines. 
一重引用符か三重引用符かどうかにかかわらず、文字列の改行は、エディタがキャリッジリターンの `\r`(CR)またはCRLFの組み合わせを使用していても、改行文字(\n)行を終了します。
#To include a CR in a string, use an explicit escape `\r`; for example, you can enter the literal string `"a CRLF line ending\r\n"`.
文字列にCRを含めるには、明示的なエスケープ `\r`を使います。 たとえば、リテラル文字列 `\"を入力することができます。

## Common Operations
##一般的な操作

#You can lexicographically compare strings using the standard comparison operators:
標準的な比較演算子を使用して、辞書的に文字列を比較することができます。

```jldoctest
julia> "abracadabra" < "xylophone"
true

julia> "abracadabra" == "xylophone"
false

julia> "Hello, world." != "Goodbye, world."
true

julia> "1 + 2 = 3" == "1 + 2 = $(1 + 2)"
true
```

<!--- You can search for the index of a particular character using the [`search()`](@ref) function: -->
あなたは[`search()`](@ref)関数を使って特定の文字のインデックスを検索することができます：

```jldoctest
julia> search("xylophone", 'x')
1

julia> search("xylophone", 'p')
5

julia> search("xylophone", 'z')
0
```

<!--- You can start the search for a character at a given offset by providing a third argument: -->
3番目の引数を指定すると、指定したオフセットで文字の検索を開始できます。

```jldoctest
julia> search("xylophone", 'o')
4

julia> search("xylophone", 'o', 5)
7

julia> search("xylophone", 'o', 8)
0
```

<!--- You can use the [`contains()`](@ref) function to check if a substring is contained in a string: -->
文字列に部分文字列が含まれているかどうかを調べるには、[`contains()`](@ref)関数を使うことができます：

```jldoctest
julia> contains("Hello, world.", "world")
true

julia> contains("Xylophon", "o")
true

julia> contains("Xylophon", "a")
false

julia> contains("Xylophon", 'o')
ERROR: MethodError: no method matching contains(::String, ::Char)
Closest candidates are:
  contains(!Matched::Function, ::Any, !Matched::Any) at reduce.jl:664
  contains(::AbstractString, !Matched::AbstractString) at strings/search.jl:378
```

<!--- The last error is because `'o'` is a character literal, and [`contains()`](@ref) is a generic function that looks for subsequences.  -->
最後のエラーは `'o'`が文字リテラルであり、[`contains()`](@ref)が部分列を探す汎用関数だからです。
<!--- To look for an element in a sequence, you must use [`in()`](@ref) instead. -->
シーケンス内の要素を探すには、代わりに[`in()`](@ref)を使う必要があります。

<!--- Two other handy string functions are [`repeat()`](@ref) and [`join()`](@ref): -->
他の2つの便利な文字列関数は[`repeat()`](@ref)と[`join()`](@ref)です：

```jldoctest
julia> repeat(".:Z:.", 10)
".:Z:..:Z:..:Z:..:Z:..:Z:..:Z:..:Z:..:Z:..:Z:..:Z:."

julia> join(["apples", "bananas", "pineapples"], ", ", " and ")
"apples, bananas and pineapples"
```

<!--- Some other useful functions include: -->
その他の便利な機能には、

  * 
#    [`endof(str)`](@ref) gives the maximal (byte) index that can be used to index into `str`.
    [`endof(str)`](@ref)は、 `str`へのインデックス付けに使用できる最大(バイト)のインデックスを返します。
  * 
#    [`length(str)`](@ref) the number of characters in `str`.
    [`length(str)`](@ref) `str`の文字数です。
  * 
#    [`i = start(str)`](@ref start) gives the first valid index at which a character can be found in `str` (typically 1).
    [`i = start(str)`](@ref start)は `str`(通常は1)に文字がある最初の有効なインデックスを返します。
  * 
#    [`c, j = next(str,i)`](@ref next) returns next character at or after the index `i` and the next valid character index following that. 
    [`c、j = next(str、i)`](@ref next)は、インデックス `i`以降の次の文字とそれに続く有効な文字インデックスを返します。
#    With [`start()`](@ref) and [`endof()`](@ref), can be used to iterate through the characters in `str`.
    [`start()`](@ref)と[`endof()`](@ref)を使って `str`の文字を反復することができます。
  * 
#    [`ind2chr(str,i)`](@ref) gives the number of characters in `str` up to and including any at index `i`.
    [`ind2chr(str、i)`](@ref)は、 `str`の文字数をインデックス` i`の文字数まで与えます。
  * 
#    [`chr2ind(str,j)`](@ref) gives the index at which the `j`th character in `str` occurs.
    [`chr2ind(str、j)`](@ref) `str`の` j番目の文字が現れるインデックスを返します。

<!--- # [Non-Standard String Literals](@id non-standard-string-literals) -->
## [非標準文字列リテラル](@ id非標準文字列リテラル)

<!--- There are situations when you want to construct a string or use string semantics, but the behavior of the standard string construct is not quite what is needed.  -->
文字列を作成したり、文字列セマンティクスを使いたい場合がありますが、標準の文字列構造の動作は必要ではありません。
<!--- For these kinds of situations, Julia provides [non-standard string literals](@ref).  -->
このような場合、Juliaは[非標準文字列リテラル](@ref)を提供します。
<!--- A non-standard string literal looks like a regular double-quoted string literal, but is immediately prefixed by an identifier, and doesn't behave quite like a normal string literal.   -->
非標準の文字列リテラルは、二重引用符で囲まれた通常の文字列リテラルのように見えますが、すぐに識別子の前に置かれ、通常の文字列リテラルのようには動作しません。
<!--- Regular expressions, byte array literals and version number literals, as described below, are some examples of non-standard string literals.  -->
以下に説明する正規表現、バイト配列リテラルおよびバージョン番号リテラルは、非標準文字列リテラルのいくつかの例です。
<!--- Other examples are given in the [Metaprogramming](@ref) section. -->
その他の例は[Metaprogramming](@ref)セクションにあります。

<!--- ## Regular Expressions -->
## 正規表現

<!--- Julia has Perl-compatible regular expressions (regexes), as provided by the [PCRE](http://www.pcre.org/) library.  -->
Juliaには、[PCRE](http://www.pcre.org/)ライブラリが提供する、Perl互換の正規表現(正規表現)があります。
<!--- Regular expressions are related to strings in two ways: the obvious connection is that regular expressions are used to find regular patterns in strings; the other connection is that regular expressions are themselves input as strings, which are parsed into a state machine that can be used to efficiently search for patterns in strings.  -->
正規表現は、文字列に2つの点で関連しています。文字列内の正規のパターンを見つけるために正規表現が使用されるという明白な接続です; もう1つの関係は、正規表現自体が文字列として入力され、文字列のパターンを効率的に検索するために使用できるステートマシンに解析されるということです。
<!--- In Julia, regular expressions are input using non-standard string literals prefixed with various identifiers beginning with `r`.  -->
Juliaでは、正規表現は、 `r`で始まるさまざまな識別子の接頭辞が付けられた非標準文字列リテラルを使用して入力されます。
<!--- The most basic regular expression literal without any options turned on just uses `r"..."`: -->
オプションが設定されていない最も基本的な正規表現のリテラルは `r"... "`を使うだけです:

```jldoctest
julia> r"^\s*(?:#|$)"
r"^\s*(?:#|$)"

julia> typeof(ans)
Regex
```

<!--- To check if a regex matches a string, use [`ismatch()`](@ref): -->
正規表現が文字列にマッチするかどうかを調べるには、[`ismatch()`](@ref)を使います：

```jldoctest
julia> ismatch(r"^\s*(?:#|$)", "not a comment")
false

julia> ismatch(r"^\s*(?:#|$)", "# a comment")
true
```

<!--- As one can see here, [`ismatch()`](@ref) simply returns true or false, indicating whether the given regex matches the string or not.  -->
ここで見ることができるように、[`ismatch()`](@ref)は単に正規表現が文字列と一致するかどうかを示すtrueまたはfalseを返します。
<!--- Commonly, however, one wants to know not just whether a string matched, but also *how* it matched.  -->
しかし、一般的に、文字列が一致するかどうかだけでなく、どのように一致するかを知りたいと考えています。
<!--- To capture this information about a match, use the [`match()`](@ref) function instead: -->
一致に関するこの情報を取得するには、代わりに[`match()`](@ref)関数を使用します：

```jldoctest
julia> match(r"^\s*(?:#|$)", "not a comment")

julia> match(r"^\s*(?:#|$)", "# a comment")
RegexMatch("#")
```

<!--- If the regular expression does not match the given string, [`match()`](@ref) returns `nothing` -- a special value that does not print anything at the interactive prompt.  -->
正規表現が指定された文字列と一致しない場合、[`match()`](@ref)は `nothing`を返します。これは対話型プロンプトで何も出力しない特別な値です。
<!--- Other than not printing, it is a completely normal value and you can test for it programmatically: -->
印刷しない場合を除き、これは完全に正常な値であり、プログラムでテストすることができます。

```julia
m = match(r"^\s*(?:#|$)", line)
if m === nothing
    println("not a comment")
else
    println("blank or comment")
end
```

<!--- If a regular expression does match, the value returned by [`match()`](@ref) is a `RegexMatch` object.  -->
正規表現が一致する場合、[`match()`](@ref)によって返される値は `RegexMatch`オブジェクトです。
<!--- These objects record how the expression matches, including the substring that the pattern matches and any captured substrings, if there are any.  -->
これらのオブジェクトは、パターンが一致する部分文字列と取り込まれた部分文字列(存在する場合)を含む、式がどのように一致するかを記録します。
<!--- This example only captures the portion of the substring that matches, but perhaps we want to capture any non-blank text after the comment character.  -->
この例では、一致する部分文字列の部分だけをキャプチャしますが、コメント文字の後に空白以外のテキストをキャプチャしたい場合があります。
<!--- We could do the following: -->
次のように行うことができます：

```jldoctest
julia> m = match(r"^\s*(?:#\s*(.*?)\s*$|$)", "# a comment ")
RegexMatch("# a comment ", 1="a comment")
```

<!--- When calling [`match()`](@ref), you have the option to specify an index at which to start the search.  -->
[`match()`](@ref)を呼び出すとき、検索を開始するインデックスを指定するオプションがあります。
<!--- For example: -->
例えば：

```jldoctest
julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",1)
RegexMatch("1")

julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",6)
RegexMatch("2")

julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",11)
RegexMatch("3")
```

<!--- You can extract the following info from a `RegexMatch` object: -->
`RegexMatch`オブジェクトから以下の情報を抽出することができます：

  * 
#    the entire substring matched: `m.match`
    部分文字列全体が一致しました： `m.match`
  * 
#    the captured substrings as an array of strings: `m.captures`
    キャプチャされた部分文字列を文字列の配列として返します： `m.captures`
  * 
#    the offset at which the whole match begins: `m.offset`
    マッチ全体が始まるオフセット： `m.offset`
  * 
#    the offsets of the captured substrings as a vector: `m.offsets`
    取り込まれた部分文字列のオフセットをベクトルとして返します： `m.offsets`

<!--- For when a capture doesn't match, instead of a substring, `m.captures` contains `nothing` in that position, and `m.offsets` has a zero offset (recall that indices in Julia are 1-based, so a zero offset into a string is invalid).  -->
キャプチャが一致しない場合、部分文字列の代わりに `m.captures`はその位置に` nothing`を含み、 `m.offsets`はゼロオフセットを持ちます(Juliaのインデックスは1から始まるので、a 文字列へのゼロオフセットは無効です)。
<!--- Here is a pair of somewhat contrived examples: -->
ここには幾分考案された例のペアがあります：


```jldoctest acdmatch
julia> m = match(r"(a|b)(c)?(d)", "acd")
RegexMatch("acd", 1="a", 2="c", 3="d")

julia> m.match
"acd"

julia> m.captures
3-element Array{Union{SubString{String}, Void},1}:
 "a"
 "c"
 "d"

julia> m.offset
1

julia> m.offsets
3-element Array{Int64,1}:
 1
 2
 3

julia> m = match(r"(a|b)(c)?(d)", "ad")
RegexMatch("ad", 1="a", 2=nothing, 3="d")

julia> m.match
"ad"

julia> m.captures
3-element Array{Union{SubString{String}, Void},1}:
 "a"
 nothing
 "d"

julia> m.offset
1

julia> m.offsets
3-element Array{Int64,1}:
 1
 0
 2
```

<!--- It is convenient to have captures returned as an array so that one can use destructuring syntax to bind them to local variables: -->
キャプチャを配列として返すと、構造化構文を使用してローカル変数にバインドすることができます。

```jldoctest acdmatch
julia> first, second, third = m.captures; first
"a"
```

<!--- Captures can also be accessed by indexing the `RegexMatch` object with the number or name of the capture group: -->
キャプチャは、 `RegexMatch`オブジェクトをキャプチャグループの番号または名前で索引付けすることによってアクセスすることもできます。

```jldoctest
julia> m=match(r"(?<hour>\d+):(?<minute>\d+)","12:45")
RegexMatch("12:45", hour="12", minute="45")

julia> m[:minute]
"45"

julia> m[2]
"45"
```

<!--- Captures can be referenced in a substitution string when using [`replace()`](@ref) by using `\n` to refer to the nth capture group and prefixing the subsitution string with `s`.  -->
キャプチャは、n番目のキャプチャグループを参照するために `\n`を使用し、サブセット文字列に`s`を接頭辞として使用することで、[`replace()`](@ref)を使用するときに置換文字列で参照できます。
<!--- Capture group 0 refers to the entire match object.  -->
キャプチャ・グループ0はマッチ・オブジェクト全体を参照します。
<!--- Named capture groups can be referenced in the substitution with `g<groupname>`.  -->
名前付きキャプチャグループは、`g<groupname>`での置換で参照できます。
<!--- For example: -->
例えば：

```jldoctest
julia> replace("first second", r"(\w+) (?<agroup>\w+)", s"\g<agroup> \1")
"second first"
```

<!--- Numbered capture groups can also be referenced as `\g<n>` for disambiguation, as in: -->
番号付きのキャプチャグループは、次のように曖昧さ回避のために `\g<n>`として参照することもできます。

```jldoctest
julia> replace("a", r".", s"\g<0>1")
"a1"
```

<!--- You can modify the behavior of regular expressions by some combination of the flags `i`, `m`, `s`, and `x` after the closing double quote mark.  -->
閉じた二重引用符の後に、フラグ`i`、`m`、`s`、`x`の組み合わせによって正規表現の動作を変更することができます。
<!--- These flags have the same meaning as they do in Perl, as explained in this excerpt from the [perlre manpage](http://perldoc.perl.org/perlre.html#Modifiers): -->
これらのフラグは、[perlremanpage](http://perldoc.perl.org/perlre.html#Modifiers)からの抜粋で説明したように、Perlと同じ意味を持ちます。

<!--- 
 i   Do case-insensitive pattern matching.
    If locale matching rules are in effect, the case map is taken from the current locale for code points less than 255, and from Unicode rules for larger code points. 
    However, matches that would cross the Unicode rules/non-Unicode rules boundary (ords 255/256) will not succeed.
--->
```
i   大文字と小文字を区別しないパターンマッチングを行います。
    ロケール・マッチング・ルールが有効な場合、ケース・マップは、現在のロケールから255未満のコード・ポイント、およびより大きいコード・ポイントのUnicodeルールから取得されます。
    ただし、Unicode規則/非Unicode規則境界(ords 255/256)を超える一致は成功しません。

```
<!--- 
m   Treat string as multiple lines.  
    That is, change "^" and "$" from matching the start or end of the string to matching the start or end of any line anywhere within the string.
--->
```
m   文字列を複数の行として扱います。
    つまり、 "^"と "$"を、文字列の先頭または末尾が文字列内の任意の行の先頭または末尾に一致するように変更します。
```
<!--- 
s   Treat string as single line.  
    That is, change "." to match any character whatsoever, even a newline, which normally it would not match.
    Used together, as r""ms, they let the "." match any character whatsoever, while still allowing "^" and "$" to match, respectively, just after and just before newlines within the string.
--->
```
s   文字列を1行として扱います。
    つまり、 "."を変更します。どんな文字でも一致させることができます。改行は通常は一致しません。
    一緒に使用すると、r"" msのように "."文字列内の改行の直後と直前に、それぞれ "^"と "$"を一致させることができます。
```
<!--- 
x   
    Tells the regular expression parser to ignore most whitespace that is neither backslashed nor within a character class. 
    You can use this to break up your regular expression into (slightly) more readable parts. 
    The '#' character is also treated as a metacharacter introducing a comment, just as in ordinary code.
--->
```
X
    正規表現パーサーに、バックスラッシュでも文字クラス内でもない大部分の空白を無視するように指示します。
    これを使用して、正規表現を(少し)読みやすい部分に分割することができます。
    '＃'文字は、通常のコードと同様に、コメントを導入するメタキャラクタとしても扱われます。
```

<!--- For example, the following regex has all three flags turned on: -->
たとえば、次の正規表現には3つのフラグがすべてオンになっています。

```jldoctest
julia> r"a+.*b+.*?d$"ism
r"a+.*b+.*?d$"ims

julia> match(r"a+.*b+.*?d$"ism, "Goodbye,\nOh, angry,\nBad world\n")
RegexMatch("angry,\nBad world")
```

<!--- Triple-quoted regex strings, of the form `r"""..."""`, are also supported (and may be convenient for regular expressions containing quotation marks or newlines). -->
`r"""..."""`形式の三重引用符付き正規表現文字列もサポートされています(引用符や改行を含む正規表現には便利かもしれません)。

<!--- # [Byte Array Literals](@id man-byte-array-literals) -->
## [バイト配列リテラル](@id man-byte-array-literals)

<!--- Another useful non-standard string literal is the byte-array string literal: `b"..."`.  -->
もう一つの有用な非標準の文字列リテラルは、バイト配列の文字列リテラルです。 `b"..."`。
<!--- This form lets you use string notation to express literal byte arrays -- i.e. arrays of [`UInt8`](@ref) values.  -->
この形式では文字列表記を使用してリテラルバイト配列、つまり[`UInt8`](@ref)値の配列を表現できます。
<!--- The rules for byte array literals are the following: -->
バイト配列リテラルの規則は次のとおりです。

<!---  ASCII characters and ASCII escapes produce a single byte. -->
   * ASCII文字とASCIIエスケープは1バイトを生成します。
<!--- `\x` and octal escape sequences produce the *byte* corresponding to the escape value. -->
   * `\x`と8進エスケープシーケンスは、エスケープ値に対応する* byte *を生成します。
<!--- Unicode escape sequences produce a sequence of bytes encoding that code point in UTF-8. -->
   * Unicodeのエスケープシーケンスは、UTF-8でそのコードポイントをエンコードする一連のバイトを生成します。

<!--- There is some overlap between these rules since the behavior of `\x` and octal escapes less than 0x80 (128) are covered by both of the first two rules, but here these rules agree.  -->
`\x`の振る舞いと0x80(128)より小さい8進数のエスケープは最初の2つの規則の両方でカバーされるので、これらの規則の間にはいくつかの重複がありますが、ここではこれらの規則に同意します。
<!--- Together, these rules allow one to easily use ASCII characters, arbitrary byte values, and UTF-8 sequences to produce arrays of bytes.  -->
これらのルールを組み合わせることで、ASCII文字、任意のバイト値、UTF-8シーケンスを簡単に使用してバイト配列を生成することができます。
<!--- Here is an example using all three: -->
ここでは3つすべてを使用した例を示します。

```jldoctest
julia> b"DATA\xff\u2200"
8-element Array{UInt8,1}:
 0x44
 0x41
 0x54
 0x41
 0xff
 0xe2
 0x88
 0x80
```

<!--- The ASCII string "DATA" corresponds to the bytes 68, 65, 84, 65. `\xff` produces the single byte 255.  -->
ASCII文字列 "DATA"はバイト68,65,84,65に対応します。 `\xff`は1バイト255を生成します。
<!--- The Unicode escape `\u2200` is encoded in UTF-8 as the three bytes 226, 136, 128.  -->
Unicodeのエスケープ `\u2200`はUTF-8で3つのバイト226,136,128としてエンコードされます。
<!--- Note that the resulting byte array does not correspond to a valid UTF-8 string -- if you try to use this as a regular string literal, you will get a syntax error: -->
結果のバイト配列が有効なUTF-8文字列に対応しないことに注意してください。これを通常の文字列リテラルとして使用しようとすると、構文エラーが発生します：

```julia-repl
julia> "DATA\xff\u2200"
ERROR: syntax: invalid UTF-8 sequence
```

#Also observe the significant distinction between `\xff` and `\uff`: the former escape sequence encodes the *byte 255*, whereas the latter escape sequence represents the *code point 255*, which is encoded as two bytes in UTF-8:
前のエスケープシーケンスは*255*をエンコードしますが、後者のエスケープシーケンスはUTF-8で2バイトとしてエンコードされた*コードポイント255* を表しますが、`\xff`と`\uff`：

```jldoctest
julia> b"\xff"
1-element Array{UInt8,1}:
 0xff

julia> b"\uff"
2-element Array{UInt8,1}:
 0xc3
 0xbf
```
#In character literals, this distinction is glossed over and `\xff` is allowed to represent the code point 255, because characters *always* represent code points. 
文字リテラルでは、文字は*常に*コードポイントを表しているため、この区別はグロス表示され、`\xff`はコードポイント255を表すことができます。
#In strings, however, `\x` escapes always represent bytes, not code points, whereas `\u` and `\U` escapes always represent code points, which are encoded in one or more bytes. 
しかし、文字列では `\ x`エスケープは常にコードポイントではなくバイトを表し、` \ u`と `\ U`エスケープは常に1つ以上のバイトでエンコードされたコードポイントを表します。
#For code points less than `\u80`, it happens that the UTF-8 encoding of each code point is just the single byte produced by the corresponding `\x` escape, so the distinction can safely be ignored. 
`\ u80`より小さいコードポイントの場合、各コードポイントのUTF-8エンコーディングは、対応する` \ x`エスケープによって生成された1バイトだけであるため、区別は無視しても安全です。
#For the escapes `\x80` through `\xff` as compared to `\u80` through `\uff`, however, there is a major difference: the former escapes all encode single bytes, which -- unless followed by very specific continuation bytes -- do not form valid UTF-8 data, whereas the latter escapes all represent Unicode code points with two-byte encodings.
しかし、 `\ x80`から` \ xff`までの `\ u80`から` \ uff`までのエスケープでは、大きな違いがあります：前者はすべて単一バイトをエスケープします。バイト - 有効なUTF-8データを形成しませんが、後者はすべて2バイトのエンコードでUnicodeコードポイントを表します。

#If this is all extremely confusing, try reading ["The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets"](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/).
これが非常に混乱している場合は、[「すべてのソフトウェア開発者が絶対に最低限必要な、絶対にUnicodeと文字セットについて熟知している絶対最小」」(https://www.joelonsoftware.com/2003/10/08/the-absolute-ソフトウェア開発者が最低限必要とする、絶対に積極的に必要な、ユニコードとキャラクタセットについての言い訳ではありません。
#It's an excellent introduction to Unicode and UTF-8, and may help alleviate some confusion regarding the matter.
これは、UnicodeとUTF-8の優れた紹介であり、この問題に関するいくつかの混乱を緩和するのに役立ちます。

<!--- ## [Version Number Literals](@id man-version-number-literals) -->
## [バージョン番号リテラル](@ idの人バージョン番号リテラル)

<!--- Version numbers can easily be expressed with non-standard string literals of the form `v"..."`. -->
バージョン番号は `v"..."`形式の非標準文字列リテラルで簡単に表現できます。
#Version number literals create `VersionNumber` objects which follow the specifications of [semantic versioning](http://semver.org), and therefore are composed of major, minor and patch numeric values, followed by pre-release and build alpha-numeric annotations. 
バージョンナンバーリテラルは、[セマンティックバージョニング](http://semver.org)の仕様に従う `VersionNumber`オブジェクトを作成するため、メジャー、マイナー、パッチの数値、プレリリース、アルファベット注釈。
#For example, `v"0.2.1-rc1+win64"` is broken into major version `0`, minor version `2`, patch version `1`, pre-release `rc1` and build `win64`. 
たとえば、 `v" 0.2.1-rc1 + win64 "は、メジャーバージョン0、マイナーバージョン2、パッチバージョン1、リリース前rc1、ビルドwin64となっています。
#When entering a version literal, everything except the major version number is optional, therefore e.g.  `v"0.2"` is equivalent to `v"0.2.0"` (with empty pre-release/build annotations), `v"2"` is equivalent to `v"2.0.0"`, and so on.
バージョンリテラルを入力するとき、メジャーバージョン番号を除くすべてはオプションです。 `` 0.2 "`は `` v "0.2.0" `(リリース前/ビルドの注釈が空の状態)と同等で、` `2 '`は `` v "2.0.0" `と等価です。

#`VersionNumber` objects are mostly useful to easily and correctly compare two (or more) versions.
`VersionNumber`オブジェクトは、2つ(またはそれ以上)のバージョンを簡単かつ正確に比較するのに最も役立ちます。
#For example, the constant `VERSION` holds Julia version number as a `VersionNumber` object, and therefore one can define some version-specific behavior using simple statements as:
たとえば、定数 `VERSION`は、Juliaバージョン番号を` VersionNumber`オブジェクトとして保持します。したがって、単純な文を使用してバージョン固有の動作を定義できます。

```julia
if v"0.2" <= VERSION < v"0.3-"
    # do something specific to 0.2 release series
end
```

#Note that in the above example the non-standard version number `v"0.3-"` is used, with a trailing `-`: this notation is a Julia extension of the standard, and it's used to indicate a version which is lower than any `0.3` release, including all of its pre-releases. 
上記の例では、非標準バージョン番号 `v" 0.3- "`が使用され、末尾に `-`が付いています。この表記法は標準のJulia拡張であり、任意の「0.3」リリース(すべてのリリース前を含む)
#So in the above example the code would only run with stable `0.2` versions, and exclude such versions as `v"0.3.0-rc1"`. 
したがって、上の例では、コードは安定した `0.2`バージョンでのみ実行され、` v "0.3.0-rc1" `などのバージョンは除外されます。
#In order to also allow for unstable (i.e. pre-release) `0.2` versions, the lower bound check should be modified like this: `v"0.2-" <= VERSION`.
不安定(すなわちプレリリース)の「0.2」バージョンも可能にするために、下限チェックは以下のように変更されるべきである：「v」0.2-「<= VERSION」。

#Another non-standard version specification extension allows one to use a trailing `+` to express an upper limit on build versions, e.g.  `VERSION > v"0.2-rc1+"` can be used to mean any version above `0.2-rc1` and any of its builds: it will return `false` for version `v"0.2-rc1+win64"` and `true` for `v"0.2-rc2"`.
別の非標準バージョン仕様拡張では、ビルドバージョンの上限を表現するために後続の `+ 'を使用することができます。 `` 0.2-rc1 + '`は、` 0.2-rc1`とそのビルドのいずれかのバージョンを意味するために使用できます。バージョン `` v``に対して `false`を返す0.2-rc1 + win64" `と` true `` for `` 0.2-rc2 "`のようになります。

#It is good practice to use such special versions in comparisons (particularly, the trailing `-` should always be used on upper bounds unless there's a good reason not to), but they must not be used as the actual version number of anything, as they are invalid in the semantic versioning scheme.
このような特別なバージョンを比較に使うのは良い習慣です(特に、後続の ` - `は、そうしないと良い理由がある場合を除いて常に上限で使用するべきです)。しかし、実際のバージョン番号として使うことはできません。セマンティックバージョニングスキームでは無効です。

#Besides being used for the [`VERSION`](@ref) constant, `VersionNumber` objects are widely used in the `Pkg` module, to specify packages versions and their dependencies.
[`Version`](@ref)定数のために使用される以外に、 ` VersionNumber`オブジェクトは ` Pkg`モジュールで広く使われ、パッケージのバージョンとその依存関係を指定します。

<!--- ## [Raw String Literals](@id man-raw-string-literals) -->
## [生の文字列リテラル](@ id man-raw-string-literals)

<!--- Raw strings without interpolation or unescaping can be expressed with non-standard string literals of the form `raw"..."`.  -->
補間やアンエスケープのない生の文字列は、 `raw '..." `という形式の非標準文字列リテラルで表現できます。
<!--- Raw string literals create ordinary `String` objects which contain the enclosed contents exactly as entered with no interpolation or unescaping.  -->
生の文字列リテラルは、内包されていないか、エスケープされていない状態で入力されたものとまったく同じ内容を含む通常の `String`オブジェクトを作成します。
<!--- This is useful for strings which contain code or markup in other languages which use `$` or `\` as special characters.  -->
これは `$`や `\`を特殊文字として使う他の言語のコードやマークアップを含む文字列に便利です。
<!--- The exception is quotation marks that still must be escaped, e.g. `raw"\""` is equivalent to `"\""`. -->
例外は引き続きエスケープする必要がある引用符です。 `raw'\""`は `"\"" `と同じです。
